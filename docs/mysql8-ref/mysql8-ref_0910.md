> 原文：[`dev.mysql.com/doc/refman/8.0/en/create-table-select.html`](https://dev.mysql.com/doc/refman/8.0/en/create-table-select.html)

#### 15.1.20.4 创建表...选择语句

您可以通过在`CREATE TABLE`语句末尾添加一个`SELECT`语句来从另一个表创建一个表：

```sql
CREATE TABLE *new_tbl* [AS] SELECT * FROM *orig_tbl*;
```

MySQL 为`SELECT`中的所有元素创建新列。例如：

```sql
mysql> CREATE TABLE test (a INT NOT NULL AUTO_INCREMENT,
 ->        PRIMARY KEY (a), KEY(b))
 ->        ENGINE=InnoDB SELECT b,c FROM test2;
```

这将创建一个带有三列`a`、`b`和`c`的`InnoDB`表。`ENGINE`选项是`CREATE TABLE`语句的一部分，不应在`SELECT`之后使用；否则会导致语法错误。其他`CREATE TABLE`选项如`CHARSET`也是如此。

注意，`SELECT`语句中的列附加在表的右侧，而不是重叠在其上。看下面的例子：

```sql
mysql> SELECT * FROM foo;
+---+
| n |
+---+
| 1 |
+---+

mysql> CREATE TABLE bar (m INT) SELECT n FROM foo;
Query OK, 1 row affected (0.02 sec)
Records: 1  Duplicates: 0  Warnings: 0

mysql> SELECT * FROM bar;
+------+---+
| m    | n |
+------+---+
| NULL | 1 |
+------+---+
1 row in set (0.00 sec)
```

对于表`foo`中的每一行，在`bar`中插入一行，其值来自`foo`并且新列使用默认值。

在由`CREATE TABLE ... SELECT`生成的表中，仅在`CREATE TABLE`部分命名的列首先出现。在两个部分或仅在`SELECT`部分命名的列在其后出现。`SELECT`列的数据类型可以通过在`CREATE TABLE`部分中指定该列来覆盖。

如果在将数据复制到表时发生错误，则表会自动删除而不会创建。然而，在 MySQL 8.0.21 之前，当使用基于行的复制时，`CREATE TABLE ... SELECT`语句会在二进制日志中记录为两个事务，一个用于创建表，另一个用于插入数据。当从二进制日志应用语句时，在两个事务之间或在复制数据时发生故障可能导致复制空表。这个限制在 MySQL 8.0.21 中被移除。在支持原子 DDL 的存储引擎上，当使用基于行的复制时，`CREATE TABLE ... SELECT`现在被记录并应用为一个事务。更多信息，请参见第 15.1.1 节，“原子数据定义语句支持”。

截至 MySQL 8.0.21，在支持原子 DDL 和外键约束的存储引擎上，在使用基于行的复制时，不允许在`CREATE TABLE ... SELECT`语句中创建外键。可以稍后使用`ALTER TABLE`添加外键约束。

您可以在`SELECT`之前加上`IGNORE`或`REPLACE`来指示如何处理重复唯一键值的行。使用`IGNORE`，重复唯一键值的行将被丢弃。使用`REPLACE`，新行将替换具有相同唯一键值的行。如果未指定`IGNORE`或`REPLACE`，重复的唯一键值将导致错误。有关更多信息，请参见 IGNORE 对语句执行的影响。

在 MySQL 8.0.19 及更高版本中，您还可以在`CREATE TABLE ... SELECT`语句的`SELECT`部分使用`VALUES`语句；`VALUES`语句的部分必须包含使用`AS`子句的表别名。为了命名来自`VALUES`的列，使用表别名提供列别名；否则，默认列名`column_0`、`column_1`、`column_2`等将被使用。

否则，在创建的表中列的命名遵循本节中先前描述的相同规则。示例：

```sql
mysql> CREATE TABLE tv1
     >     SELECT * FROM (VALUES ROW(1,3,5), ROW(2,4,6)) AS v;
mysql> TABLE tv1;
+----------+----------+----------+
| column_0 | column_1 | column_2 |
+----------+----------+----------+
|        1 |        3 |        5 |
|        2 |        4 |        6 |
+----------+----------+----------+

mysql> CREATE TABLE tv2
     >     SELECT * FROM (VALUES ROW(1,3,5), ROW(2,4,6)) AS v(x,y,z);
mysql> TABLE tv2;
+---+---+---+
| x | y | z |
+---+---+---+
| 1 | 3 | 5 |
| 2 | 4 | 6 |
+---+---+---+

mysql> CREATE TABLE tv3 (a INT, b INT, c INT)
     >     SELECT * FROM (VALUES ROW(1,3,5), ROW(2,4,6)) AS v(x,y,z);
mysql> TABLE tv3;
+------+------+------+----------+----------+----------+
| a    | b    | c    |        x |        y |        z |
+------+------+------+----------+----------+----------+
| NULL | NULL | NULL |        1 |        3 |        5 |
| NULL | NULL | NULL |        2 |        4 |        6 |
+------+------+------+----------+----------+----------+

mysql> CREATE TABLE tv4 (a INT, b INT, c INT)
     >     SELECT * FROM (VALUES ROW(1,3,5), ROW(2,4,6)) AS v(x,y,z);
mysql> TABLE tv4;
+------+------+------+---+---+---+
| a    | b    | c    | x | y | z |
+------+------+------+---+---+---+
| NULL | NULL | NULL | 1 | 3 | 5 |
| NULL | NULL | NULL | 2 | 4 | 6 |
+------+------+------+---+---+---+

mysql> CREATE TABLE tv5 (a INT, b INT, c INT)
     >     SELECT * FROM (VALUES ROW(1,3,5), ROW(2,4,6)) AS v(a,b,c);
mysql> TABLE tv5;
+------+------+------+
| a    | b    | c    |
+------+------+------+
|    1 |    3 |    5 |
|    2 |    4 |    6 |
+------+------+------+
```

当选择所有列并使用默认列名时，可以省略`SELECT *`，因此刚刚用于创建表`tv1`的语句也可以写成如下所示：

```sql
mysql> CREATE TABLE tv1 VALUES ROW(1,3,5), ROW(2,4,6);
mysql> TABLE tv1;
+----------+----------+----------+
| column_0 | column_1 | column_2 |
+----------+----------+----------+
|        1 |        3 |        5 |
|        2 |        4 |        6 |
+----------+----------+----------+
```

当使用`VALUES`作为`SELECT`的源时，所有列始终被选入新表中，无法像从命名表中选择时那样选择单独的列；以下每个语句都会产生错误(`ER_OPERAND_COLUMNS`)：

```sql
CREATE TABLE tvx
    SELECT (x,z) FROM (VALUES ROW(1,3,5), ROW(2,4,6)) AS v(x,y,z);

CREATE TABLE tvx (a INT, c INT)
    SELECT (x,z) FROM (VALUES ROW(1,3,5), ROW(2,4,6)) AS v(x,y,z);
```

类似地，您可以在`SELECT`语句的位置使用`TABLE`语句。这遵循与`VALUES`相同的规则；源表的所有列及其在源表中的名称始终被插入到新表中。示例：

```sql
mysql> TABLE t1;
+----+----+
| a  | b  |
+----+----+
|  1 |  2 |
|  6 |  7 |
| 10 | -4 |
| 14 |  6 |
+----+----+

mysql> CREATE TABLE tt1 TABLE t1;
mysql> TABLE tt1;
+----+----+
| a  | b  |
+----+----+
|  1 |  2 |
|  6 |  7 |
| 10 | -4 |
| 14 |  6 |
+----+----+

mysql> CREATE TABLE tt2 (x INT) TABLE t1;
mysql> TABLE tt2;
+------+----+----+
| x    | a  | b  |
+------+----+----+
| NULL |  1 |  2 |
| NULL |  6 |  7 |
| NULL | 10 | -4 |
| NULL | 14 |  6 |
+------+----+----+
```

由于底层`SELECT`语句中行的排序不能总是确定，`CREATE TABLE ... IGNORE SELECT`和`CREATE TABLE ... REPLACE SELECT`语句被标记为不安全的基于语句的复制。在使用基于语句的模式时，这些语句在错误日志中产生警告，并在使用`MIXED`模式时以基于行的格式写入二进制日志。另请参阅第 19.2.1.1 节，“基于语句和基于行的复制的优缺点”。

`CREATE TABLE ... SELECT`不会为您自动创建任何索引。这是有意为之，以使语句尽可能灵活。如果您希望在创建的表中有索引，您应该在`SELECT`语句之前指定这些索引：

```sql
mysql> CREATE TABLE bar (UNIQUE (n)) SELECT n FROM foo;
```

对于`CREATE TABLE ... SELECT`，目标表不会保留所选自表中列是否为生成列的信息。语句的`SELECT`部分不能为目标表中的生成列分配值。

对于`CREATE TABLE ... SELECT`，目标表会保留原始表中的表达式默认值。

可能会发生一些数据类型的转换。例如，`AUTO_INCREMENT`属性不会被保留，`VARCHAR`列可能会变成`CHAR`列。保留的属性包括`NULL`（或`NOT NULL`）以及对于那些具有的列，`CHARACTER SET`，`COLLATION`，`COMMENT`和`DEFAULT`子句。

在使用`CREATE TABLE ... SELECT`创建表时，请确保为查询中的任何函数调用或表达式设置别名。如果不这样做，`CREATE`语句可能会失败或导致不良的列名称。

```sql
CREATE TABLE artists_and_works
  SELECT artist.name, COUNT(work.artist_id) AS number_of_works
  FROM artist LEFT JOIN work ON artist.id = work.artist_id
  GROUP BY artist.id;
```

您还可以明确指定创建表中列的数据类型：

```sql
CREATE TABLE foo (a TINYINT NOT NULL) SELECT b+1 AS a FROM bar;
```

对于`CREATE TABLE ... SELECT`，如果指定了`IF NOT EXISTS`并且目标表存在，则不会将任何内容插入目标表，并且该语句不会被记录。

为了确保二进制日志可以用于重新创建原始表，MySQL 不允许在 `CREATE TABLE ... SELECT` 过程中进行并发插入。然而，在 MySQL 8.0.21 之前，当使用基于行的复制时，从二进制日志应用 `CREATE TABLE ... SELECT` 操作时，允许在复制表上进行并发插入。这个限制在支持原子 DDL 的存储引擎上在 MySQL 8.0.21 中被移除。更多信息，请参见 Section 15.1.1, “Atomic Data Definition Statement Support”。

你不能在类似 `CREATE TABLE *`new_table`* SELECT ... FROM *`old_table`* ...` 的语句中将 `FOR UPDATE` 作为 `SELECT` 的一部分。如果尝试这样做，该语句将失败。

`CREATE TABLE ... SELECT` 操作仅对列应用 `ENGINE_ATTRIBUTE` 和 `SECONDARY_ENGINE_ATTRIBUTE` 值。除非明确指定，否则表和索引的 `ENGINE_ATTRIBUTE` 和 `SECONDARY_ENGINE_ATTRIBUTE` 值不会应用于新表。
