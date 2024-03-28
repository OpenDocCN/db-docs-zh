# 15.2.12 REPLACE Statement

> 原文：[`dev.mysql.com/doc/refman/8.0/en/replace.html`](https://dev.mysql.com/doc/refman/8.0/en/replace.html)

```sql
REPLACE [LOW_PRIORITY | DELAYED]
    [INTO] *tbl_name*
    [PARTITION (*partition_name* [, *partition_name*] ...)]
    [(*col_name* [, *col_name*] ...)]
    { {VALUES | VALUE} (*value_list*) [, (*value_list*)] ...
      |
      VALUES *row_constructor_list*
    }

REPLACE [LOW_PRIORITY | DELAYED]
    [INTO] *tbl_name*
    [PARTITION (*partition_name* [, *partition_name*] ...)]
    SET *assignment_list*

REPLACE [LOW_PRIORITY | DELAYED]
    [INTO] *tbl_name*
    [PARTITION (*partition_name* [, *partition_name*] ...)]
    [(*col_name* [, *col_name*] ...)]
    {SELECT ... | TABLE *table_name*}

*value*:
    {*expr* | DEFAULT}

*value_list*:
    *value* [, *value*] ...

*row_constructor_list*:
    ROW(*value_list*)[, ROW(*value_list*)][, ...]

*assignment*:
    *col_name* = *value*

*assignment_list*:
    *assignment* [, *assignment*] ...
```

`REPLACE`的工作方式与`INSERT`完全相同，唯一的区别是，如果表中的旧行与`PRIMARY KEY`或`UNIQUE`索引的新行具有相同的值，则在插入新行之前会删除旧行。请参见第 15.2.7 节，“INSERT Statement”。

`REPLACE`是 MySQL 对 SQL 标准的扩展。它要么插入，要么*删除并插入*。对于另一个 MySQL 对标准 SQL 的扩展——要么插入，要么*更新*，请参见第 15.2.7.2 节，“INSERT ... ON DUPLICATE KEY UPDATE Statement”。

`DELAYED`插入和替换在 MySQL 5.6 中已弃用。在 MySQL 8.0 中，不再支持`DELAYED`。服务器会识别但忽略`DELAYED`关键字，将替换处理为非延迟替换，并生成一个`ER_WARN_LEGACY_SYNTAX_CONVERTED`警告：REPLACE DELAYED 不再受支持。该语句已转换为 REPLACE。`DELAYED`关键字计划在将来的版本中移除。

注意

`REPLACE`仅在表具有`PRIMARY KEY`或`UNIQUE`索引时才有意义。否则，它将等同于`INSERT`，因为没有索引可用于确定新行是否重复。

所有列的值都取自`REPLACE`语句中指定的值。任何缺失的列都将设置为它们的默认值，就像`INSERT`一样。您不能引用当前行的值并在新行中使用它们。如果您使用类似`SET *col_name* = *col_name* + 1`的赋值，右侧的列名引用将被视为`DEFAULT(*col_name*)`，因此该赋值等效于`SET *col_name* = DEFAULT(*col_name*) + 1`。

在 MySQL 8.0.19 及更高版本中，您可以使用`VALUES ROW()`指定`REPLACE`尝试插入的列值。

要使用`REPLACE`，您必须对表具有`INSERT`和`DELETE`权限。

如果一个生成的列被显式替换，唯一允许的值是`DEFAULT`。有关生成列的信息，请参见第 15.1.20.8 节，“CREATE TABLE and Generated Columns”。

`REPLACE`支持使用`PARTITION`子句显式选择分区，后面跟着逗号分隔的分区、子分区或两者名称列表。与`INSERT`一样，如果无法将新行插入这些分区或子分区中的任何一个，`REPLACE`语句将失败，并显示错误信息“Found a row not matching the given partition set”。有关更多信息和示例，请参见第 26.5 节，“分区选择”。

`REPLACE`语句返回一个计数，指示受影响的行数。这是删除和插入的行数之和。如果对于单行`REPLACE`，计数为 1，则插入了一行且未删除任何行。如果计数大于 1，则在插入新行之前删除了一个或多个旧行。如果表包含多个唯一索引，并且新行在不同唯一索引中重复值以替换一个以上的旧行是可能的。

受影响的行数计数使得很容易确定`REPLACE`是否仅添加了一行还是还替换了任何行：检查计数是否为 1（添加）或大于 1（替换）。

如果使用 C API，则可以使用`mysql_affected_rows()`函数获取受影响的行数计数。

不能在子查询中将新行替换到表中并从同一表中进行选择。

MySQL 使用以下算法进行`REPLACE`（以及`LOAD DATA ... REPLACE`）：

1.  尝试将新行插入表中

1.  当由于主键或唯一索引发生重复键错误而插入失败时：

    1.  从表中删除具有重复键值的冲突行

    1.  再次尝试将新行插入表中

在重复键错误的情况下，存储引擎可能将`REPLACE`作为更新而不是删除加插入来执行，但语义是相同的。除了存储引擎如何递增`Handler_*`xxx`*`状态变量可能有所不同外，没有其他用户可见的影响。

因为`REPLACE ... SELECT`语句的结果取决于从`SELECT`中的行的排序，而这种顺序并不能始终保证，所以在记录这些语句时，源和副本可能会发生分歧。因此，`REPLACE ... SELECT`语句被标记为不安全的基于语句的复制。当使用基于语句的模式时，这些语句在错误日志中产生警告，并在使用`MIXED`模式时以基于行的格式写入二进制日志。另请参阅 Section 19.2.1.1, “Advantages and Disadvantages of Statement-Based and Row-Based Replication”。

MySQL 8.0.19 及更高版本支持`TABLE`以及带有`REPLACE`的`SELECT`，就像它对`INSERT`一样。有关更多信息和示例，请参见 Section 15.2.7.1, “INSERT ... SELECT Statement”。

当修改一个现有的非分区表以适应分区，或者修改已经分区表的分区时，您可能考虑修改表的主键（参见 Section 26.6.1, “Partitioning Keys, Primary Keys, and Unique Keys”）。您应该意识到，如果这样做，`REPLACE`语句的结果可能会受到影响，就像您修改非分区表的主键时一样。考虑以下由`CREATE TABLE`语句创建的表：

```sql
CREATE TABLE test (
  id INT UNSIGNED NOT NULL AUTO_INCREMENT,
  data VARCHAR(64) DEFAULT NULL,
  ts TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (id)
);
```

当我们创建这个表并在 mysql 客户端中运行所示的语句时，结果如下：

```sql
mysql> REPLACE INTO test VALUES (1, 'Old', '2014-08-20 18:47:00');
Query OK, 1 row affected (0.04 sec)

mysql> REPLACE INTO test VALUES (1, 'New', '2014-08-20 18:47:42');
Query OK, 2 rows affected (0.04 sec)

mysql> SELECT * FROM test;
+----+------+---------------------+
| id | data | ts                  |
+----+------+---------------------+
|  1 | New  | 2014-08-20 18:47:42 |
+----+------+---------------------+
1 row in set (0.00 sec)
```

现在我们创建一个几乎与第一个表相同的第二个表，唯一不同的是主键现在涵盖了 2 列，如下所示（加粗文本）：

```sql
CREATE TABLE test2 (
  id INT UNSIGNED NOT NULL AUTO_INCREMENT,
  data VARCHAR(64) DEFAULT NULL,
  ts TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  *PRIMARY KEY (id, ts)* );
```

当我们在`test2`上运行与原始`test`表相同的两个`REPLACE`语句时，我们得到不同的结果：

```sql
mysql> REPLACE INTO test2 VALUES (1, 'Old', '2014-08-20 18:47:00');
Query OK, 1 row affected (0.05 sec)

mysql> REPLACE INTO test2 VALUES (1, 'New', '2014-08-20 18:47:42');
Query OK, 1 row affected (0.06 sec)

mysql> SELECT * FROM test2;
+----+------+---------------------+
| id | data | ts                  |
+----+------+---------------------+
|  1 | Old  | 2014-08-20 18:47:00 |
|  1 | New  | 2014-08-20 18:47:42 |
+----+------+---------------------+
2 rows in set (0.00 sec)
```

这是因为在`test2`上运行时，`id`和`ts`列的值必须与现有行的值匹配才能替换行；否则，将插入一行。
