> 原文：[`dev.mysql.com/doc/refman/8.0/en/index-merge-optimization.html`](https://dev.mysql.com/doc/refman/8.0/en/index-merge-optimization.html)

#### 10.2.1.3 索引合并优化

索引合并访问方法通过多个`range`扫描检索行，并将它们的结果合并为一个。此访问方法仅合并来自单个表的索引扫描，而不是跨多个表的扫描。合并可以生成底层扫描的并集、交集或并集的交集。

可能使用索引合并的示例查询：

```sql
SELECT * FROM *tbl_name* WHERE *key1* = 10 OR *key2* = 20;

SELECT * FROM *tbl_name*
  WHERE (*key1* = 10 OR *key2* = 20) AND *non_key* = 30;

SELECT * FROM t1, t2
  WHERE (t1.*key1* IN (1,2) OR t1.*key2* LIKE '*value*%')
  AND t2.*key1* = t1.*some_col*;

SELECT * FROM t1, t2
  WHERE t1.*key1* = 1
  AND (t2.*key1* = t1.*some_col* OR t2.*key2* = t1.*some_col2*);
```

注意

索引合并优化算法具有以下已知限制：

+   如果您的查询具有复杂的`WHERE`子句，带有深层次的`AND`/`OR`嵌套，并且 MySQL 没有选择最佳计划，请尝试使用以下标识变换来分发术语：

    ```sql
    (*x* AND *y*) OR *z* => (*x* OR *z*) AND (*y* OR *z*)
    (*x* OR *y*) AND *z* => (*x* AND *z*) OR (*y* AND *z*)
    ```

+   索引合并不适用于全文索引。

在`EXPLAIN`输出中，索引合并方法显示为`type`列中的`index_merge`。在这种情况下，`key`列包含使用的索引列表，`key_len`包含这些索引的最长键部分的列表。

索引合并访问方法有几种算法，这些算法显示在`EXPLAIN`输出的`Extra`字段中：

+   `使用 intersect(...)`

+   `使用 union(...)`

+   `使用 sort_union(...)`

以下部分更详细地描述了这些算法。优化器根据各种可用选项的成本估算，在不同可能的索引合并算法和其他访问方法之间进行选择。

+   索引合并交集访问算法

+   索引合并并集访问算法

+   索引合并排序-并集访问算法

+   影响索引合并优化

##### 索引合并交集访问算法

当`WHERE`子句转换为与`AND`组合的不同键上的几个范围条件时，并且每个条件是以下之一时，可以应用此访问算法：

+   这种形式的*`N`*部分表达式，其中索引恰好有*`N`*部分（即，所有索引部分都被覆盖）：

    ```sql
    *key_part1* = *const1* AND *key_part2* = *const2* ... AND *key_partN* = *constN*
    ```

+   `InnoDB`表的主键上的任何范围条件。

示例：

```sql
SELECT * FROM *innodb_table*
  WHERE *primary_key* < 10 AND *key_col1* = 20;

SELECT * FROM *tbl_name*
  WHERE *key1_part1* = 1 AND *key1_part2* = 2 AND *key2* = 2;
```

索引合并交集算法对所有使用的索引执行同时扫描，并生成从合并索引扫描接收到的行序列的交集。

如果查询中使用的所有列都由使用的索引覆盖，则不会检索完整的表行（在这种情况下，`EXPLAIN`输出在`Extra`字段中包含`Using index`）。以下是这种查询的示例：

```sql
SELECT COUNT(*) FROM t1 WHERE key1 = 1 AND key2 = 1;
```

如果使用的索引未覆盖查询中使用的所有列，则仅在满足所有使用键的范围条件时才检索完整行。

如果合并条件之一是`InnoDB`表的主键条件，则不用于检索行，而是用于过滤使用其他条件检索的行。

##### 索引合并联合访问算法

该算法的标准与索引合并交集算法的标准类似。当表的`WHERE`子句转换为与`OR`组合的不同键上的多个范围条件，并且每个条件是以下条件之一时，该算法适用：

+   这种形式的*`N`*部分表达式，其中索引恰好有*`N`*部分（即所有索引部分都被覆盖）：

    ```sql
    *key_part1* = *const1* OR *key_part2* = *const2* ... OR *key_partN* = *constN*
    ```

+   任何`InnoDB`表的主键上的范围条件。

+   适用于索引合并交集算法的条件。

例子：

```sql
SELECT * FROM t1
  WHERE *key1* = 1 OR *key2* = 2 OR *key3* = 3;

SELECT * FROM *innodb_table*
  WHERE (*key1* = 1 AND *key2* = 2)
     OR (*key3* = 'foo' AND *key4* = 'bar') AND *key5* = 5;
```

##### 索引合并排序联合访问算法

当`WHERE`子句转换为由`OR`组合的多个范围条件时，此访问算法适用，但索引合并联合算法不适用。

例子：

```sql
SELECT * FROM *tbl_name*
  WHERE *key_col1* < 10 OR *key_col2* < 20;

SELECT * FROM *tbl_name*
  WHERE (*key_col1* > 10 OR *key_col2* = 20) AND *nonkey_col* = 30;
```

排序联合算法和联合算法之间的区别在于排序联合算法必须首先获取所有行的行 ID 并对其进行排序，然后才能返回任何行。

##### 影响索引合并优化

使用索引合并取决于`index_merge`、`index_merge_intersection`、`index_merge_union`和`index_merge_sort_union`标志的`optimizer_switch`系统变量的值。请参阅第 10.9.2 节，“可切换优化”。默认情况下，所有这些标志都是`on`。要仅启用某些算法，请将`index_merge`设置为`off`，并仅启用应允许的其他算法。

除了使用`optimizer_switch`系统变量来全局控制 MySQL 对索引合并算法的使用之外，MySQL 还支持优化器提示以影响每个语句的优化器。请参阅第 10.9.3 节，“优化器提示”。
