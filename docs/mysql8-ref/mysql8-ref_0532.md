> 原文：[`dev.mysql.com/doc/refman/8.0/en/limit-optimization.html`](https://dev.mysql.com/doc/refman/8.0/en/limit-optimization.html)

#### 10.2.1.19 LIMIT 查询优化

如果只需要结果集中的指定数量的行，请在查询中使用`LIMIT`子句，而不是获取整个结果集并丢弃额外的数据。

MySQL 有时会优化具有`LIMIT *`row_count`*`子句而没有`HAVING`子句的查询：

+   如果你使用`LIMIT`只选择了少数几行，MySQL 在某些情况下会使用索引，而通常情况下它更倾向于进行全表扫描。

+   如果将`LIMIT *`row_count`*`与`ORDER BY`结合使用，MySQL 会在找到排序结果的前*`row_count`*行后停止排序，而不是对整个结果进行排序。如果使用索引进行排序，这是非常快的。如果必须进行文件排序，那么在找到不带`LIMIT`子句的查询的所有匹配行之前，将选择大多数或全部行，并对它们进行排序，然后才找到前*`row_count`*行。在找到初始行之后，MySQL 不会对结果集的任何剩余部分进行排序。

    这种行为的一种表现是，带有和不带有`LIMIT`的`ORDER BY`查询可能以不同的顺序返回行，如本节后面所述。

+   如果将`LIMIT *`row_count`*`与`DISTINCT`结合使用，MySQL 会在找到*`row_count`*个唯一行后停止。

+   在某些情况下，`GROUP BY`可以通过按顺序读取索引（或对索引进行排序），然后计算摘要直到索引值发生变化来解决。在这种情况下，`LIMIT *`row_count`*`不会计算任何不必要的`GROUP BY`值。

+   一旦 MySQL 向客户端发送了所需数量的行，除非你使用`SQL_CALC_FOUND_ROWS`，否则它会中止查询。在这种情况下，可以使用`SELECT FOUND_ROWS()`检索行数。参见第 14.15 节，“信息函数”。

+   `LIMIT 0`会快速返回一个空集。这对于检查查询的有效性很有用。它还可以用于在使用使结果集元数据可用的 MySQL API 的应用程序中获取结果列的类型。使用**mysql**客户端程序，你可以使用`--column-type-info`选项来显示结果列类型。

+   如果服务器使用临时表来解析查询，它会使用`LIMIT *`row_count`*`子句来计算所需的空间。

+   如果未对`ORDER BY`使用索引但同时存在`LIMIT`子句，优化器可能能够避免使用合并文件，并使用内存中的`filesort`操作在内存中对行进行排序。

如果多行在`ORDER BY`列中具有相同的值，服务器可以自由地以任何顺序返回这些行，并且可能根据整体执行计划而有所不同。换句话说，这些行的排序顺序对于非排序列是不确定的。

影响执行计划的一个因素是`LIMIT`，因此带有和不带有`LIMIT`的`ORDER BY`查询可能以不同的顺序返回行。考虑这个查询，它按`category`列排序，但对于`id`和`rating`列是不确定的：

```sql
mysql> SELECT * FROM ratings ORDER BY category;
+----+----------+--------+
| id | category | rating |
+----+----------+--------+
|  1 |        1 |    4.5 |
|  5 |        1 |    3.2 |
|  3 |        2 |    3.7 |
|  4 |        2 |    3.5 |
|  6 |        2 |    3.5 |
|  2 |        3 |    5.0 |
|  7 |        3 |    2.7 |
+----+----------+--------+
```

包含`LIMIT`可能会影响每个`category`值内行的顺序。例如，这是一个有效的查询结果：

```sql
mysql> SELECT * FROM ratings ORDER BY category LIMIT 5;
+----+----------+--------+
| id | category | rating |
+----+----------+--------+
|  1 |        1 |    4.5 |
|  5 |        1 |    3.2 |
|  4 |        2 |    3.5 |
|  3 |        2 |    3.7 |
|  6 |        2 |    3.5 |
+----+----------+--------+
```

在每种情况下，行都按照`ORDER BY`列排序，这是 SQL 标准要求的全部内容。

如果确保带有和不带有`LIMIT`时返回相同的行顺序很重要，请在`ORDER BY`子句中包含额外的列，以使顺序确定。例如，如果`id`值是唯一的，您可以通过像这样排序来使给定`category`值的行按`id`顺序显示：

```sql
mysql> SELECT * FROM ratings ORDER BY category, id;
+----+----------+--------+
| id | category | rating |
+----+----------+--------+
|  1 |        1 |    4.5 |
|  5 |        1 |    3.2 |
|  3 |        2 |    3.7 |
|  4 |        2 |    3.5 |
|  6 |        2 |    3.5 |
|  2 |        3 |    5.0 |
|  7 |        3 |    2.7 |
+----+----------+--------+

mysql> SELECT * FROM ratings ORDER BY category, id LIMIT 5;
+----+----------+--------+
| id | category | rating |
+----+----------+--------+
|  1 |        1 |    4.5 |
|  5 |        1 |    3.2 |
|  3 |        2 |    3.7 |
|  4 |        2 |    3.5 |
|  6 |        2 |    3.5 |
+----+----------+--------+
```

对于带有`ORDER BY`或`GROUP BY`以及`LIMIT`子句的查询，优化器会尝试默认选择有序索引，以加快查询执行速度。在 MySQL 8.0.21 之前，即使在使用其他优化可能更快的情况下，也无法覆盖此行为。从 MySQL 8.0.21 开始，可以通过将`optimizer_switch`系统变量的`prefer_ordering_index`标志设置为`off`来关闭此优化。

*示例*：首先我们创建并填充一个表`t`，如下所示：

```sql
# Create and populate a table t:

mysql> CREATE TABLE t (
 ->     id1 BIGINT NOT NULL,
 ->     id2 BIGINT NOT NULL,
 ->     c1 VARCHAR(50) NOT NULL,
 ->     c2 VARCHAR(50) NOT NULL,
 ->  PRIMARY KEY (id1),
 ->  INDEX i (id2, c1)
 -> );

# [Insert some rows into table t - not shown]
```

验证`prefer_ordering_index`标志是否已启用：

```sql
mysql> SELECT @@optimizer_switch LIKE '%prefer_ordering_index=on%';
+------------------------------------------------------+
| @@optimizer_switch LIKE '%prefer_ordering_index=on%' |
+------------------------------------------------------+
|                                                    1 |
+------------------------------------------------------+
```

由于以下查询有一个`LIMIT`子句，我们期望它使用有序索引，如果可能的话。在这种情况下，正如我们从`EXPLAIN`输出中看到的那样，它使用了表的主键。

```sql
mysql> EXPLAIN SELECT c2 FROM t
 ->     WHERE id2 > 3
 ->     ORDER BY id1 ASC LIMIT 2\G
*************************** 1\. row ***************************
           id: 1
  select_type: SIMPLE
        table: t
   partitions: NULL
         type: index
possible_keys: i
          key: PRIMARY
      key_len: 8
          ref: NULL
         rows: 2
     filtered: 70.00
        Extra: Using where
```

现在我们禁用`prefer_ordering_index`标志，并重新运行相同的查询；这次它使用索引`i`（其中包括`WHERE`子句中使用的`id2`列），以及一个文件排序：

```sql
mysql> SET optimizer_switch = "prefer_ordering_index=off";

mysql> EXPLAIN SELECT c2 FROM t
 ->     WHERE id2 > 3
 ->     ORDER BY id1 ASC LIMIT 2\G
*************************** 1\. row ***************************
           id: 1
  select_type: SIMPLE
        table: t
   partitions: NULL
         type: range
possible_keys: i
          key: i
      key_len: 8
          ref: NULL
         rows: 14
     filtered: 100.00
        Extra: Using index condition; Using filesort
```

参见第 10.9.2 节，“可切换优化”。
