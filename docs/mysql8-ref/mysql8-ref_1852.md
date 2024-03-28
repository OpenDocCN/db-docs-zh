# 26.2.7 MySQL 分区如何处理 NULL

> 原文：[`dev.mysql.com/doc/refman/8.0/en/partitioning-handling-nulls.html`](https://dev.mysql.com/doc/refman/8.0/en/partitioning-handling-nulls.html)

MySQL 中的分区对于将`NULL`作为分区表达式的值（无论是列值还是用户提供的表达式的值）并不做任何限制。即使允许将`NULL`用作必须产生整数的表达式的值，但重要的是要记住`NULL`不是一个数字。MySQL 的分区实现将`NULL`视为小于任何非`NULL`值，就像`ORDER BY`一样。

这意味着`NULL`的处理在不同类型的分区之间有所不同，并且如果您没有为此做好准备，可能会产生您意想不到的行为。在这种情况下，我们在本节中讨论了每种 MySQL 分区类型在确定应将行存储在哪个分区时如何处理`NULL`值，并为每种情况提供了示例。

**使用 RANGE 分区处理 NULL。** 如果您向由`RANGE`分区的表插入一行，使得用于确定分区的列值为`NULL`，则该行将插入到最低的分区中。考虑以下在名为`p`的数据库中创建的两个表：

```sql
mysql> CREATE TABLE t1 (
 ->     c1 INT,
 ->     c2 VARCHAR(20)
 -> )
 -> PARTITION BY RANGE(c1) (
 ->     PARTITION p0 VALUES LESS THAN (0),
 ->     PARTITION p1 VALUES LESS THAN (10),
 ->     PARTITION p2 VALUES LESS THAN MAXVALUE
 -> );
Query OK, 0 rows affected (0.09 sec)

mysql> CREATE TABLE t2 (
 ->     c1 INT,
 ->     c2 VARCHAR(20)
 -> )
 -> PARTITION BY RANGE(c1) (
 ->     PARTITION p0 VALUES LESS THAN (-5),
 ->     PARTITION p1 VALUES LESS THAN (0),
 ->     PARTITION p2 VALUES LESS THAN (10),
 ->     PARTITION p3 VALUES LESS THAN MAXVALUE
 -> );
Query OK, 0 rows affected (0.09 sec)
```

通过以下查询`INFORMATION_SCHEMA`数据库中的`PARTITIONS`表，您可以看到这两个`CREATE TABLE`语句创建的分区：

```sql
mysql> SELECT TABLE_NAME, PARTITION_NAME, TABLE_ROWS, AVG_ROW_LENGTH, DATA_LENGTH
     >   FROM INFORMATION_SCHEMA.PARTITIONS
     >   WHERE TABLE_SCHEMA = 'p' AND TABLE_NAME LIKE 't_';
+------------+----------------+------------+----------------+-------------+
| TABLE_NAME | PARTITION_NAME | TABLE_ROWS | AVG_ROW_LENGTH | DATA_LENGTH |
+------------+----------------+------------+----------------+-------------+
| t1         | p0             |          0 |              0 |           0 |
| t1         | p1             |          0 |              0 |           0 |
| t1         | p2             |          0 |              0 |           0 |
| t2         | p0             |          0 |              0 |           0 |
| t2         | p1             |          0 |              0 |           0 |
| t2         | p2             |          0 |              0 |           0 |
| t2         | p3             |          0 |              0 |           0 |
+------------+----------------+------------+----------------+-------------+
7 rows in set (0.00 sec)
```

（有关此表的更多信息，请参见第 28.3.21 节，“The INFORMATION_SCHEMA PARTITIONS Table”。）现在让我们用包含在用作分区键的列中的`NULL`的单行填充这些表，并验证使用一对`SELECT`语句插入了这些行：

```sql
mysql> INSERT INTO t1 VALUES (NULL, 'mothra');
Query OK, 1 row affected (0.00 sec)

mysql> INSERT INTO t2 VALUES (NULL, 'mothra');
Query OK, 1 row affected (0.00 sec)

mysql> SELECT * FROM t1;
+------+--------+
| id   | name   |
+------+--------+
| NULL | mothra |
+------+--------+
1 row in set (0.00 sec)

mysql> SELECT * FROM t2;
+------+--------+
| id   | name   |
+------+--------+
| NULL | mothra |
+------+--------+
1 row in set (0.00 sec)
```

您可以通过重新运行针对`INFORMATION_SCHEMA.PARTITIONS`的上一个查询并检查输出来查看用于存储插入行的分区：

```sql
mysql> SELECT TABLE_NAME, PARTITION_NAME, TABLE_ROWS, AVG_ROW_LENGTH, DATA_LENGTH
     >   FROM INFORMATION_SCHEMA.PARTITIONS
     >   WHERE TABLE_SCHEMA = 'p' AND TABLE_NAME LIKE 't_';
+------------+----------------+------------+----------------+-------------+
| TABLE_NAME | PARTITION_NAME | TABLE_ROWS | AVG_ROW_LENGTH | DATA_LENGTH |
+------------+----------------+------------+----------------+-------------+
*| t1         | p0             |          1 |             20 |          20 |*
| t1         | p1             |          0 |              0 |           0 |
| t1         | p2             |          0 |              0 |           0 |
*| t2         | p0             |          1 |             20 |          20 |*
| t2         | p1             |          0 |              0 |           0 |
| t2         | p2             |          0 |              0 |           0 |
| t2         | p3             |          0 |              0 |           0 |
+------------+----------------+------------+----------------+-------------+
7 rows in set (0.01 sec)
```

您还可以通过删除这些分区，然后重新运行`SELECT`语句来演示这些行存储在每个表的最低编号分区中：

```sql
mysql> ALTER TABLE t1 DROP PARTITION p0;
Query OK, 0 rows affected (0.16 sec)

mysql> ALTER TABLE t2 DROP PARTITION p0;
Query OK, 0 rows affected (0.16 sec)

mysql> SELECT * FROM t1;
Empty set (0.00 sec)

mysql> SELECT * FROM t2;
Empty set (0.00 sec)
```

（有关`ALTER TABLE ... DROP PARTITION`的更多信息，请参见第 15.1.9 节，“ALTER TABLE Statement”。）

对于使用 SQL 函数的分区表达式，`NULL`也以这种方式处理。假设我们使用类似于以下的`CREATE TABLE`语句定义一个表：

```sql
CREATE TABLE tndate (
    id INT,
    dt DATE
)
PARTITION BY RANGE( YEAR(dt) ) (
    PARTITION p0 VALUES LESS THAN (1990),
    PARTITION p1 VALUES LESS THAN (2000),
    PARTITION p2 VALUES LESS THAN MAXVALUE
);
```

与其他 MySQL 函数一样，`YEAR(NULL)`返回`NULL`。具有`NULL`值的`dt`列行被视为分区表达式评估为低于任何其他值的值，因此被插入到分区`p0`中。

**使用 LIST 分区处理 NULL 值。** 通过`LIST`分区的表仅在其中一个分区使用包含`NULL`的值列表定义时才允许`NULL`值。相反，通过`LIST`分区的表如果在值列表中没有明确使用`NULL`，则拒绝导致分区表达式产生`NULL`值的行，如下例所示：

```sql
mysql> CREATE TABLE ts1 (
 ->     c1 INT,
 ->     c2 VARCHAR(20)
 -> )
 -> PARTITION BY LIST(c1) (
 ->     PARTITION p0 VALUES IN (0, 3, 6),
 ->     PARTITION p1 VALUES IN (1, 4, 7),
 ->     PARTITION p2 VALUES IN (2, 5, 8)
 -> );
Query OK, 0 rows affected (0.01 sec)

mysql> INSERT INTO ts1 VALUES (9, 'mothra');
ERROR 1504 (HY000): Table has no partition for value 9 
mysql> INSERT INTO ts1 VALUES (NULL, 'mothra');
ERROR 1504 (HY000): Table has no partition for value NULL
```

只有`c1`值在`0`和`8`之间（包括 0 和 8）的行才能插入到`ts1`中。`NULL`不在此范围内，就像数字`9`一样。我们可以创建包含`NULL`值列表的`ts2`和`ts3`表，如下所示：

```sql
mysql> CREATE TABLE ts2 (
 ->     c1 INT,
 ->     c2 VARCHAR(20)
 -> )
 -> PARTITION BY LIST(c1) (
 ->     PARTITION p0 VALUES IN (0, 3, 6),
 ->     PARTITION p1 VALUES IN (1, 4, 7),
 ->     PARTITION p2 VALUES IN (2, 5, 8),
 ->     PARTITION p3 VALUES IN (NULL)
 -> );
Query OK, 0 rows affected (0.01 sec)

mysql> CREATE TABLE ts3 (
 ->     c1 INT,
 ->     c2 VARCHAR(20)
 -> )
 -> PARTITION BY LIST(c1) (
 ->     PARTITION p0 VALUES IN (0, 3, 6),
 ->     PARTITION p1 VALUES IN (1, 4, 7, NULL),
 ->     PARTITION p2 VALUES IN (2, 5, 8)
 -> );
Query OK, 0 rows affected (0.01 sec)
```

在为分区定义值列表时，您可以（也应该）将`NULL`视为任何其他值一样对待。例如，`VALUES IN (NULL)`和`VALUES IN (1, 4, 7, NULL)`都是有效的，就像`VALUES IN (1, NULL, 4, 7)`，`VALUES IN (NULL, 1, 4, 7)`等一样。您可以将具有`c1`列为`NULL`的行插入到`ts2`和`ts3`中：

```sql
mysql> INSERT INTO ts2 VALUES (NULL, 'mothra');
Query OK, 1 row affected (0.00 sec)

mysql> INSERT INTO ts3 VALUES (NULL, 'mothra');
Query OK, 1 row affected (0.00 sec)
```

通过针对`INFORMATION_SCHEMA.PARTITIONS`发出适当的查询，您可以确定刚刚插入的行使用了哪些分区进行存储（我们假设，与前面的示例一样，分区表是在`p`数据库中创建的）：

```sql
mysql> SELECT TABLE_NAME, PARTITION_NAME, TABLE_ROWS, AVG_ROW_LENGTH, DATA_LENGTH
     >   FROM INFORMATION_SCHEMA.PARTITIONS
     >   WHERE TABLE_SCHEMA = 'p' AND TABLE_NAME LIKE 'ts_';
+------------+----------------+------------+----------------+-------------+
| TABLE_NAME | PARTITION_NAME | TABLE_ROWS | AVG_ROW_LENGTH | DATA_LENGTH |
+------------+----------------+------------+----------------+-------------+
| ts2        | p0             |          0 |              0 |           0 |
| ts2        | p1             |          0 |              0 |           0 |
| ts2        | p2             |          0 |              0 |           0 |
*| ts2        | p3             |          1 |             20 |          20 |*
| ts3        | p0             |          0 |              0 |           0 |
*| ts3        | p1             |          1 |             20 |          20 |*
| ts3        | p2             |          0 |              0 |           0 |
+------------+----------------+------------+----------------+-------------+
7 rows in set (0.01 sec)
```

正如本节前面所示，您还可以通过删除这些分区并执行`SELECT`来验证用于存储行的分区。

**使用 HASH 和 KEY 分区处理 NULL 值。** 对于使用`HASH`或`KEY`分区的表，`NULL`的处理略有不同。在这些情况下，任何产生`NULL`值的分区表达式都被视为其返回值为零。我们可以通过创建一个使用适当值的记录的`HASH`分区表并查看其对文件系统的影响来验证这种行为。假设您使用以下语句创建了一个名为`th`的表（也在`p`数据库中）：

```sql
mysql> CREATE TABLE th (
 ->     c1 INT,
 ->     c2 VARCHAR(20)
 -> )
 -> PARTITION BY HASH(c1)
 -> PARTITIONS 2;
Query OK, 0 rows affected (0.00 sec)
```

可以使用以下查询查看属于该表的分区：

```sql
mysql> SELECT TABLE_NAME,PARTITION_NAME,TABLE_ROWS,AVG_ROW_LENGTH,DATA_LENGTH
     >   FROM INFORMATION_SCHEMA.PARTITIONS
     >   WHERE TABLE_SCHEMA = 'p' AND TABLE_NAME ='th';
+------------+----------------+------------+----------------+-------------+
| TABLE_NAME | PARTITION_NAME | TABLE_ROWS | AVG_ROW_LENGTH | DATA_LENGTH |
+------------+----------------+------------+----------------+-------------+
| th         | p0             |          0 |              0 |           0 |
| th         | p1             |          0 |              0 |           0 |
+------------+----------------+------------+----------------+-------------+
2 rows in set (0.00 sec)
```

每个分区的`TABLE_ROWS`为 0。现在向`th`插入两行，这两行的`c1`列值分别为`NULL`和 0，并验证这些行是否已插入，如下所示：

```sql
mysql> INSERT INTO th VALUES (NULL, 'mothra'), (0, 'gigan');
Query OK, 1 row affected (0.00 sec)

mysql> SELECT * FROM th;
+------+---------+
| c1   | c2      |
+------+---------+
| NULL | mothra  |
+------+---------+
|    0 | gigan   |
+------+---------+
2 rows in set (0.01 sec)
```

对于任意整数*`N`*，`NULL MOD *`N`*`的值始终为`NULL`。对于按`HASH`或`KEY`分区的表，此结果被视为确定正确分区的`0`。再次检查信息模式`PARTITIONS`表，我们可以看到两行都被插入到分区`p0`中：

```sql
mysql> SELECT TABLE_NAME, PARTITION_NAME, TABLE_ROWS, AVG_ROW_LENGTH, DATA_LENGTH
     >   FROM INFORMATION_SCHEMA.PARTITIONS
     >   WHERE TABLE_SCHEMA = 'p' AND TABLE_NAME ='th';
+------------+----------------+------------+----------------+-------------+
| TABLE_NAME | PARTITION_NAME | TABLE_ROWS | AVG_ROW_LENGTH | DATA_LENGTH |
+------------+----------------+------------+----------------+-------------+
*| th         | p0             |          2 |             20 |          20 |*
| th         | p1             |          0 |              0 |           0 |
+------------+----------------+------------+----------------+-------------+
2 rows in set (0.00 sec)
```

通过在表的定义中使用`PARTITION BY KEY`替换`PARTITION BY HASH`来重复上一个示例，您可以验证对于这种类型的分区，`NULL`也被视为 0。
