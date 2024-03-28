# 10.9.4 索引提示

> 原文：[`dev.mysql.com/doc/refman/8.0/en/index-hints.html`](https://dev.mysql.com/doc/refman/8.0/en/index-hints.html)

索引提示为优化器提供有关在查询处理期间如何选择索引的信息。这里描述的索引提示与第 10.9.3 节“优化器提示”中描述的优化器提示不同。索引提示和优化器提示可以单独或一起使用。

索引提示适用于`SELECT`和`UPDATE`语句。它们还适用于多表`DELETE`语句，但不适用于单表`DELETE`，如本节后面所示。

索引提示在表名后指定。（有关在`SELECT`语句中指定表的一般语法，请参见第 15.2.13.2 节“JOIN 子句”。）指定单个表的语法，包括索引提示，如下所示：

```sql
*tbl_name* [[AS] *alias*] [*index_hint_list*]

*index_hint_list*:
    *index_hint* [*index_hint*] ...

*index_hint*:
    USE {INDEX|KEY}
      [FOR {JOIN|ORDER BY|GROUP BY}] ([*index_list*])
  | {IGNORE|FORCE} {INDEX|KEY}
      [FOR {JOIN|ORDER BY|GROUP BY}] (*index_list*)

*index_list*:
    *index_name* [, *index_name*] ...
```

`USE INDEX (*`index_list`*)`提示告诉 MySQL 仅使用命名索引之一来查找表中的行。另一种语法`IGNORE INDEX (*`index_list`*)`告诉 MySQL 不使用某些特定的索引。如果`EXPLAIN`显示 MySQL 正在使用列表中错误的索引，则这些提示���有用。

`FORCE INDEX`提示类似于`USE INDEX (*`index_list`*)`，但额外假定表扫描是*非常*昂贵的。换句话说，只有在无法使用命名索引之一来查找表中的行时，才会使用表扫描。

注意

截至 MySQL 8.0.20，服务器支持索引级别的优化器提示`JOIN_INDEX`、`GROUP_INDEX`、`ORDER_INDEX`和`INDEX`，它们相当于并打算取代`FORCE INDEX`索引提示，以及`NO_JOIN_INDEX`、`NO_GROUP_INDEX`、`NO_ORDER_INDEX`和`NO_INDEX`优化器提示，它们相当于并打算取代`IGNORE INDEX`索引提示。因此，您应该期望`USE INDEX`、`FORCE INDEX`和`IGNORE INDEX`在将来的 MySQL 版本中被弃用，并在此后的某个时间被完全移除。

这些索引级优化器提示支持单表和多表`DELETE`语句。

更多信息，请参见 Index-Level Optimizer Hints。

每个提示都需要索引名称，而不是列名称。要引用主键，请使用名称`PRIMARY`。要查看表的索引名称，请使用`SHOW INDEX`语句或 Information Schema `STATISTICS`表。

*`index_name`*值不必是完整的索引名称。它可以是索引名称的明确前缀。如果前缀不明确，则会出现错误。

示例：

```sql
SELECT * FROM table1 USE INDEX (col1_index,col2_index)
  WHERE col1=1 AND col2=2 AND col3=3;

SELECT * FROM table1 IGNORE INDEX (col3_index)
  WHERE col1=1 AND col2=2 AND col3=3;
```

索引提示的语法具有以下特点：

+   对于`USE INDEX`，省略*`index_list`*在语法上是有效的，这意味着“不使用索引”。对于`FORCE INDEX`或`IGNORE INDEX`省略*`index_list`*是语法错误。

+   通过在提示中添加`FOR`子句，您可以指定索引提示的范围。这可以更精细地控制优化器在查询处理的各个阶段选择执行计划。要仅影响 MySQL 在决定如何在表中查找行以及如何处理连接时使用的索引，请使用`FOR JOIN`。要影响用于对行进行排序或分组的索引使用，请使用`FOR ORDER BY`或`FOR GROUP BY`。

+   您可以指定多个索引提示：

    ```sql
    SELECT * FROM t1 USE INDEX (i1) IGNORE INDEX FOR ORDER BY (i2) ORDER BY a;
    ```

    在几个提示中命名相同的索引不是错误的（即使在同一个提示中）：

    ```sql
    SELECT * FROM t1 USE INDEX (i1) USE INDEX (i1,i1);
    ```

    但是，对于同一表混合使用`USE INDEX`和`FORCE INDEX`是错误的：

    ```sql
    SELECT * FROM t1 USE INDEX FOR JOIN (i1) FORCE INDEX FOR JOIN (i2);
    ```

如果索引提示不包含`FOR`子句，则提示的范围是应用于语句的所有部分。例如，这个提示：

```sql
IGNORE INDEX (i1)
```

等同于以下提示的组合：

```sql
IGNORE INDEX FOR JOIN (i1)
IGNORE INDEX FOR ORDER BY (i1)
IGNORE INDEX FOR GROUP BY (i1)
```

在 MySQL 5.0 中，没有`FOR`子句的提示范围仅适用于行检索。要在没有`FOR`子句的情况下使服务器使用这种较旧的行为，请在服务器启动时启用`old`系统变量。在复制设置中启用此变量时要小心。使用基于语句的二进制日志记录，源和副本之间具有不同模式可能会导致复制错误。

处理索引提示时，它们按类型（`USE`，`FORCE`，`IGNORE`）和范围（`FOR JOIN`，`FOR ORDER BY`，`FOR GROUP BY`）被收集到单个列表中。例如：

```sql
SELECT * FROM t1
  USE INDEX () IGNORE INDEX (i2) USE INDEX (i1) USE INDEX (i2);
```

等同于：

```sql
SELECT * FROM t1
   USE INDEX (i1,i2) IGNORE INDEX (i2);
```

然后，索引提示按以下顺序应用于每个范围：

1.  如果存在，则应用`{USE|FORCE} INDEX`。（如果不存在，则使用优化器确定的索引集。）

1.  `IGNORE INDEX`应用于上一步的结果。例如，以下两个查询是等效的：

    ```sql
    SELECT * FROM t1 USE INDEX (i1) IGNORE INDEX (i2) USE INDEX (i2);

    SELECT * FROM t1 USE INDEX (i1);
    ```

对于`FULLTEXT`搜索，索引提示的工作方式如下：

+   对于自然语言模式搜索，索引提示会被静默忽略。例如，`IGNORE INDEX(i1)`会被忽略而不会有警告，索引仍然会被使用。

+   对于布尔模式搜索，带有`FOR ORDER BY`或`FOR GROUP BY`的索引提示会被静默忽略。带有`FOR JOIN`或没有`FOR`修饰符的索引提示会被应用。与非`FULLTEXT`搜索的提示应用方式相反，该提示用于查询执行的所有阶段（查找行和检索、分组和排序）。即使为非`FULLTEXT`索引提供提示，也是如此。

    例如，以下两个查询是等效的：

    ```sql
    SELECT * FROM t
      USE INDEX (index1)
      IGNORE INDEX FOR ORDER BY (index1)
      IGNORE INDEX FOR GROUP BY (index1)
      WHERE ... IN BOOLEAN MODE ... ;

    SELECT * FROM t
      USE INDEX (index1)
      WHERE ... IN BOOLEAN MODE ... ;
    ```

索引提示适用于`DELETE`语句，但仅当您使用多表`DELETE`语法时，如下所示：

```sql
mysql> EXPLAIN DELETE FROM t1 USE INDEX(col2) 
 -> WHERE col1 BETWEEN 1 AND 100 AND COL2 BETWEEN 1 AND 100\G
ERROR 1064 (42000): You have an error in your SQL syntax; check the manual that
corresponds to your MySQL server version for the right syntax to use near 'use
index(col2) where col1 between 1 and 100 and col2 between 1 and 100' at line 1 
mysql> EXPLAIN DELETE t1.* FROM t1 USE INDEX(col2) 
 -> WHERE col1 BETWEEN 1 AND 100 AND COL2 BETWEEN 1 AND 100\G
*************************** 1\. row ***************************
           id: 1
  select_type: DELETE
        table: t1
   partitions: NULL
         type: range
possible_keys: col2
          key: col2
      key_len: 5
          ref: NULL
         rows: 72
     filtered: 11.11
        Extra: Using where 1 row in set, 1 warning (0.00 sec)
```
