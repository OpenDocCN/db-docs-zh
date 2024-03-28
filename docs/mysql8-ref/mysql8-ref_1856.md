# 26.3.3 与表交换分区和子分区

> 原文：[`dev.mysql.com/doc/refman/8.0/en/partitioning-management-exchange.html`](https://dev.mysql.com/doc/refman/8.0/en/partitioning-management-exchange.html)

在 MySQL 8.0 中，可以使用`ALTER TABLE *`pt`* EXCHANGE PARTITION *`p`* WITH TABLE *`nt`*`来交换表分区或子分区与未分区表*`nt`*，其中*`pt`*是分区表，*`p`*是要与未分区表*`nt`*交换的*`pt`*的分区或子分区，前提是以下陈述为真：

1.  表*`nt`*本身没有分区。

1.  表*`nt`*不是临时表。

1.  表*`pt`*和*`nt`*的结构在其他方面是相同的。

1.  表`nt`不包含外键引用，也没有其他表有任何外键引用指向`nt`。

1.  *`nt`*中没有位于*`p`*的分区定义边界之外的行。如果使用`WITHOUT VALIDATION`，则不适用此条件。

1.  两个表必须使用相同的字符集和校对规则。

1.  对于`InnoDB`表，两个表必须使用相同的行格式。要确定`InnoDB`表的行格式，请查询`INFORMATION_SCHEMA.INNODB_TABLES`。

1.  任何分区级别的`MAX_ROWS`设置对于`p`必须与为`nt`设置的表级别`MAX_ROWS`值相同。对于`p`的任何分区级别的`MIN_ROWS`设置也必须与为`nt`设置的表级别`MIN_ROWS`值相同。

    无论`pt`是否具有显式的表级别`MAX_ROWS`或`MIN_ROWS`选项生效，这在任何情况下都是正确的。

1.  `AVG_ROW_LENGTH`在表`pt`和表`nt`之间不能有差异。

1.  表`pt`不能有任何使用`DATA DIRECTORY`选项的分区。这个限制在 MySQL 8.0.14 及更高版本中对`InnoDB`表解除。

1.  `INDEX DIRECTORY`在表和要与之交换的分区之间不能有差异。

1.  任何表或分区`TABLESPACE`选项都不能在任何表中使用。

除了通常需要的`ALTER`、`INSERT`和`CREATE`权限外，您必须具有`DROP`权限才能执行`ALTER TABLE ... EXCHANGE PARTITION`。

您还应该了解`ALTER TABLE ... EXCHANGE PARTITION`的以下影响：

+   执行`ALTER TABLE ... EXCHANGE PARTITION`不会触发分区表或要交换表上的任何触发器。

+   交换表中的任何`AUTO_INCREMENT`列都会被重置。

+   使用`ALTER TABLE ... EXCHANGE PARTITION`时，`IGNORE`关键字不起作用。

`ALTER TABLE ... EXCHANGE PARTITION`的语法如下，其中*`pt`*是分区表，*`p`*是要交换的分区（或子分区），*`nt`*是要与*`p`*交换的非分区表：

```sql
ALTER TABLE *pt*
    EXCHANGE PARTITION *p*
    WITH TABLE *nt*;
```

可选地，你可以附加`WITH VALIDATION`或`WITHOUT VALIDATION`。当指定`WITHOUT VALIDATION`时，`ALTER TABLE ... EXCHANGE PARTITION` 操作在交换分区到非分区表时不执行逐行验证，允许数据库管理员承担确保行在分区定义边界内的责任。`WITH VALIDATION`是默认选项。

在单个`ALTER TABLE EXCHANGE PARTITION`语句中，只能将一个分区或子分区与一个非分区表交换。要交换多个分区或子分区，请使用多个`ALTER TABLE EXCHANGE PARTITION`语句。`EXCHANGE PARTITION`不能与其他`ALTER TABLE`选项结合使用。分区表使用的分区和（如果适用）子分区可以是 MySQL 8.0 支持的任何类型。

#### 与非分区表交换分区

假设已经使用以下 SQL 语句创建和填充了分区表`e`：

```sql
CREATE TABLE e (
    id INT NOT NULL,
    fname VARCHAR(30),
    lname VARCHAR(30)
)
    PARTITION BY RANGE (id) (
        PARTITION p0 VALUES LESS THAN (50),
        PARTITION p1 VALUES LESS THAN (100),
        PARTITION p2 VALUES LESS THAN (150),
        PARTITION p3 VALUES LESS THAN (MAXVALUE)
);

INSERT INTO e VALUES
    (1669, "Jim", "Smith"),
    (337, "Mary", "Jones"),
    (16, "Frank", "White"),
    (2005, "Linda", "Black");
```

现在我们创建一个名为`e2`的非分区副本`e`。可以使用**mysql**客户端来完成，如下所示：

```sql
mysql> CREATE TABLE e2 LIKE e;
Query OK, 0 rows affected (0.04 sec)

mysql> ALTER TABLE e2 REMOVE PARTITIONING;
Query OK, 0 rows affected (0.07 sec)
Records: 0  Duplicates: 0  Warnings: 0
```

通过查询信息模式`PARTITIONS`表，你可以看到表`e`中包含行的分区，就像这样：

```sql
mysql> SELECT PARTITION_NAME, TABLE_ROWS
           FROM INFORMATION_SCHEMA.PARTITIONS
           WHERE TABLE_NAME = 'e';
+----------------+------------+
| PARTITION_NAME | TABLE_ROWS |
+----------------+------------+
| p0             |          1 |
| p1             |          0 |
| p2             |          0 |
| p3             |          3 |
+----------------+------------+
2 rows in set (0.00 sec)
```

注意

对于分区`InnoDB`表，信息模式`PARTITIONS`表中`TABLE_ROWS`列中给出的行数仅是 SQL 优化中使用的估计值，并不总是准确的。

要交换表`e`中的分区`p0`与表`e2`，可以使用`ALTER TABLE`，如下所示：

```sql
mysql> ALTER TABLE e EXCHANGE PARTITION p0 WITH TABLE e2;
Query OK, 0 rows affected (0.04 sec)
```

更准确地说，刚刚执行的语句导致在分区中找到的任何行与表中找到的行交换。你可以通过再次查询信息模式`PARTITIONS`表来观察这是如何发生的。之前在分区`p0`中找到的表行不再存在：

```sql
mysql> SELECT PARTITION_NAME, TABLE_ROWS
           FROM INFORMATION_SCHEMA.PARTITIONS
           WHERE TABLE_NAME = 'e';
+----------------+------------+
| PARTITION_NAME | TABLE_ROWS |
+----------------+------------+
| p0             |          0 |
| p1             |          0 |
| p2             |          0 |
| p3             |          3 |
+----------------+------------+
4 rows in set (0.00 sec)
```

如果查询表`e2`，你会发现“缺失”的行现在可以在那里找到：

```sql
mysql> SELECT * FROM e2;
+----+-------+-------+
| id | fname | lname |
+----+-------+-------+
| 16 | Frank | White |
+----+-------+-------+
1 row in set (0.00 sec)
```

与分区交换的表不一定要为空。为了演示这一点，我们首先向表`e`插入一行新数据，确保这行数据存储在分区`p0`中，方法是选择一个小于 50 的`id`列值，并在之后通过查询`PARTITIONS`表进行验证：

```sql
mysql> INSERT INTO e VALUES (41, "Michael", "Green");
Query OK, 1 row affected (0.05 sec)

mysql> SELECT PARTITION_NAME, TABLE_ROWS
           FROM INFORMATION_SCHEMA.PARTITIONS
           WHERE TABLE_NAME = 'e';            
+----------------+------------+
| PARTITION_NAME | TABLE_ROWS |
+----------------+------------+
| p0             |          1 |
| p1             |          0 |
| p2             |          0 |
| p3             |          3 |
+----------------+------------+
4 rows in set (0.00 sec)
```

现在我们再次使用与之前相同的`ALTER TABLE`语句交换分区`p0`与表`e2`：

```sql
mysql> ALTER TABLE e EXCHANGE PARTITION p0 WITH TABLE e2;
Query OK, 0 rows affected (0.28 sec)
```

以下查询的输出显示，在发出`ALTER TABLE`语句之前存储在分区`p0`中的表行和存储在表`e2`中的表行现在已经交换位置：

```sql
mysql> SELECT * FROM e;
+------+-------+-------+
| id   | fname | lname |
+------+-------+-------+
|   16 | Frank | White |
| 1669 | Jim   | Smith |
|  337 | Mary  | Jones |
| 2005 | Linda | Black |
+------+-------+-------+
4 rows in set (0.00 sec)

mysql> SELECT PARTITION_NAME, TABLE_ROWS
           FROM INFORMATION_SCHEMA.PARTITIONS
           WHERE TABLE_NAME = 'e';
+----------------+------------+
| PARTITION_NAME | TABLE_ROWS |
+----------------+------------+
| p0             |          1 |
| p1             |          0 |
| p2             |          0 |
| p3             |          3 |
+----------------+------------+
4 rows in set (0.00 sec)

mysql> SELECT * FROM e2;
+----+---------+-------+
| id | fname   | lname |
+----+---------+-------+
| 41 | Michael | Green |
+----+---------+-------+
1 row in set (0.00 sec)
```

#### 不匹配的行

请记住，在发出`ALTER TABLE ... EXCHANGE PARTITION`语句之前，在非分区表中找到的任何行必须满足存储在目标分区中的条件；否则，该语句将失败。为了看到这是如何发生的，首先向`e2`插入一行数据，该行数据超出了表`e`的分区`p0`的定义范围。例如，插入一个`id`列值过大的行；然后，再次尝试与分区交换表：

```sql
mysql> INSERT INTO e2 VALUES (51, "Ellen", "McDonald");
Query OK, 1 row affected (0.08 sec)

mysql> ALTER TABLE e EXCHANGE PARTITION p0 WITH TABLE e2;
ERROR 1707 (HY000): Found row that does not match the partition
```

只有`WITHOUT VALIDATION`选项才能使此操作成功：

```sql
mysql> ALTER TABLE e EXCHANGE PARTITION p0 WITH TABLE e2 WITHOUT VALIDATION;
Query OK, 0 rows affected (0.02 sec)
```

当将一个分区与包含不符合分区定义的行的表交换时，由数据库管理员负责修复不匹配的行，可以使用`REPAIR TABLE`或`ALTER TABLE ... REPAIR PARTITION`来执行。

#### 不需要逐行验证即可交换分区

当将一个分区与包含许多行的表交换时，为了避免耗时的验证，可以在`ALTER TABLE ... EXCHANGE PARTITION`语句中添加`WITHOUT VALIDATION`来跳过逐行验证步骤。

以下示例比较了与和不带验证时交换分区与非分区表的执行时间差异。分区表（表`e`）包含两个各有 100 万行的分区。表`e`的 p0 中的行被移除，并且 p0 与一个有 100 万行的非分区表交换。`WITH VALIDATION`操作耗时 0.74 秒。相比之下，`WITHOUT VALIDATION`操作只需 0.01 秒。

```sql
# Create a partitioned table with 1 million rows in each partition

CREATE TABLE e (
    id INT NOT NULL,
    fname VARCHAR(30),
    lname VARCHAR(30)
)
    PARTITION BY RANGE (id) (
        PARTITION p0 VALUES LESS THAN (1000001),
        PARTITION p1 VALUES LESS THAN (2000001),
);

mysql> SELECT COUNT(*) FROM e;
| COUNT(*) |
+----------+
|  2000000 |
+----------+
1 row in set (0.27 sec)

# View the rows in each partition

SELECT PARTITION_NAME, TABLE_ROWS FROM INFORMATION_SCHEMA.PARTITIONS WHERE TABLE_NAME = 'e';
+----------------+-------------+
| PARTITION_NAME | TABLE_ROWS  |
+----------------+-------------+
| p0             |     1000000 |
| p1             |     1000000 |
+----------------+-------------+
2 rows in set (0.00 sec)

# Create a nonpartitioned table of the same structure and populate it with 1 million rows

CREATE TABLE e2 (
    id INT NOT NULL,
    fname VARCHAR(30),
    lname VARCHAR(30)
);

mysql> SELECT COUNT(*) FROM e2;
+----------+
| COUNT(*) |
+----------+
|  1000000 |
+----------+
1 row in set (0.24 sec)

# Create another nonpartitioned table of the same structure and populate it with 1 million rows

CREATE TABLE e3 (
    id INT NOT NULL,
    fname VARCHAR(30),
    lname VARCHAR(30)
);

mysql> SELECT COUNT(*) FROM e3;
+----------+
| COUNT(*) |
+----------+
|  1000000 |
+----------+
1 row in set (0.25 sec)

# Drop the rows from p0 of table e

mysql> DELETE FROM e WHERE id < 1000001;
Query OK, 1000000 rows affected (5.55 sec)

# Confirm that there are no rows in partition p0

mysql> SELECT PARTITION_NAME, TABLE_ROWS FROM INFORMATION_SCHEMA.PARTITIONS WHERE TABLE_NAME = 'e';
+----------------+------------+
| PARTITION_NAME | TABLE_ROWS |
+----------------+------------+
| p0             |          0 |
| p1             |    1000000 |
+----------------+------------+
2 rows in set (0.00 sec)

# Exchange partition p0 of table e with the table e2 'WITH VALIDATION'

mysql> ALTER TABLE e EXCHANGE PARTITION p0 WITH TABLE e2 WITH VALIDATION;
Query OK, 0 rows affected (0.74 sec)

# Confirm that the partition was exchanged with table e2

mysql> SELECT PARTITION_NAME, TABLE_ROWS FROM INFORMATION_SCHEMA.PARTITIONS WHERE TABLE_NAME = 'e';
+----------------+------------+
| PARTITION_NAME | TABLE_ROWS |
+----------------+------------+
| p0             |    1000000 |
| p1             |    1000000 |
+----------------+------------+
2 rows in set (0.00 sec)

# Once again, drop the rows from p0 of table e

mysql> DELETE FROM e WHERE id < 1000001;
Query OK, 1000000 rows affected (5.55 sec)

# Confirm that there are no rows in partition p0

mysql> SELECT PARTITION_NAME, TABLE_ROWS FROM INFORMATION_SCHEMA.PARTITIONS WHERE TABLE_NAME = 'e';
+----------------+------------+
| PARTITION_NAME | TABLE_ROWS |
+----------------+------------+
| p0             |          0 |
| p1             |    1000000 |
+----------------+------------+
2 rows in set (0.00 sec)

# Exchange partition p0 of table e with the table e3 'WITHOUT VALIDATION'

mysql> ALTER TABLE e EXCHANGE PARTITION p0 WITH TABLE e3 WITHOUT VALIDATION;
Query OK, 0 rows affected (0.01 sec)

# Confirm that the partition was exchanged with table e3

mysql> SELECT PARTITION_NAME, TABLE_ROWS FROM INFORMATION_SCHEMA.PARTITIONS WHERE TABLE_NAME = 'e';
+----------------+------------+
| PARTITION_NAME | TABLE_ROWS |
+----------------+------------+
| p0             |    1000000 |
| p1             |    1000000 |
+----------------+------------+
2 rows in set (0.00 sec)
```

如果将一个分区与包含不符合分区定义的行的表进行交换，数据库管理员有责任修复不匹配的行，可以使用`REPAIR TABLE`或`ALTER TABLE ... REPAIR PARTITION`来执行此操作。

#### 用非分区表交换子分区

你也可以使用`ALTER TABLE ... EXCHANGE PARTITION`语句，将分区表的一个子分区（参见第 26.2.6 节，“子分区”）与非分区表进行交换。在下面的示例中，我们首先创建一个按`RANGE`分区并按`KEY`子分区的表`es`，像我们创建表`e`一样填充这个表，然后创建一个空的、非分区的副本`es2`，如下所示：

```sql
mysql> CREATE TABLE es (
 ->     id INT NOT NULL,
 ->     fname VARCHAR(30),
 ->     lname VARCHAR(30)
 -> )
 ->     PARTITION BY RANGE (id)
 ->     SUBPARTITION BY KEY (lname)
 ->     SUBPARTITIONS 2 (
 ->         PARTITION p0 VALUES LESS THAN (50),
 ->         PARTITION p1 VALUES LESS THAN (100),
 ->         PARTITION p2 VALUES LESS THAN (150),
 ->         PARTITION p3 VALUES LESS THAN (MAXVALUE)
 ->     );
Query OK, 0 rows affected (2.76 sec)

mysql> INSERT INTO es VALUES
 ->     (1669, "Jim", "Smith"),
 ->     (337, "Mary", "Jones"),
 ->     (16, "Frank", "White"),
 ->     (2005, "Linda", "Black");
Query OK, 4 rows affected (0.04 sec)
Records: 4  Duplicates: 0  Warnings: 0

mysql> CREATE TABLE es2 LIKE es;
Query OK, 0 rows affected (1.27 sec)

mysql> ALTER TABLE es2 REMOVE PARTITIONING;
Query OK, 0 rows affected (0.70 sec)
Records: 0  Duplicates: 0  Warnings: 0
```

虽然我们在创建表`es`时没有明确命名任何子分区，但我们可以通过在从`INFORMATION_SCHEMA`中的`PARTITIONS`表中选择时包含`SUBPARTITION_NAME`列来获取这些子分区的生成名称，如下所示：

```sql
mysql> SELECT PARTITION_NAME, SUBPARTITION_NAME, TABLE_ROWS
 ->     FROM INFORMATION_SCHEMA.PARTITIONS
 ->     WHERE TABLE_NAME = 'es';
+----------------+-------------------+------------+
| PARTITION_NAME | SUBPARTITION_NAME | TABLE_ROWS |
+----------------+-------------------+------------+
| p0             | p0sp0             |          1 |
| p0             | p0sp1             |          0 |
| p1             | p1sp0             |          0 |
| p1             | p1sp1             |          0 |
| p2             | p2sp0             |          0 |
| p2             | p2sp1             |          0 |
| p3             | p3sp0             |          3 |
| p3             | p3sp1             |          0 |
+----------------+-------------------+------------+
8 rows in set (0.00 sec)
```

以下`ALTER TABLE`语句将表`es`中的子分区`p3sp0`与非分区表`es2`进行交换：

```sql
mysql> ALTER TABLE es EXCHANGE PARTITION p3sp0 WITH TABLE es2;
Query OK, 0 rows affected (0.29 sec)
```

你可以通过发出以下查询来验证行是否已经交换：

```sql
mysql> SELECT PARTITION_NAME, SUBPARTITION_NAME, TABLE_ROWS
 ->     FROM INFORMATION_SCHEMA.PARTITIONS
 ->     WHERE TABLE_NAME = 'es';
+----------------+-------------------+------------+
| PARTITION_NAME | SUBPARTITION_NAME | TABLE_ROWS |
+----------------+-------------------+------------+
| p0             | p0sp0             |          1 |
| p0             | p0sp1             |          0 |
| p1             | p1sp0             |          0 |
| p1             | p1sp1             |          0 |
| p2             | p2sp0             |          0 |
| p2             | p2sp1             |          0 |
| p3             | p3sp0             |          0 |
| p3             | p3sp1             |          0 |
+----------------+-------------------+------------+
8 rows in set (0.00 sec)

mysql> SELECT * FROM es2;
+------+-------+-------+
| id   | fname | lname |
+------+-------+-------+
| 1669 | Jim   | Smith |
|  337 | Mary  | Jones |
| 2005 | Linda | Black |
+------+-------+-------+
3 rows in set (0.00 sec)
```

如果表被子分区，你只能交换表的一个子分区，而不是整个分区，如下所示：

```sql
mysql> ALTER TABLE es EXCHANGE PARTITION p3 WITH TABLE es2;
ERROR 1704 (HY000): Subpartitioned table, use subpartition instead of partition
```

表结构严格比较；分区表和非分区表的列和索引的数量、顺序、名称和类型必须完全匹配。此外，两个表必须使用相同的存储引擎：

```sql
mysql> CREATE TABLE es3 LIKE e;
Query OK, 0 rows affected (1.31 sec)

mysql> ALTER TABLE es3 REMOVE PARTITIONING;
Query OK, 0 rows affected (0.53 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> SHOW CREATE TABLE es3\G
*************************** 1\. row ***************************
       Table: es3
Create Table: CREATE TABLE `es3` (
  `id` int(11) NOT NULL,
  `fname` varchar(30) DEFAULT NULL,
  `lname` varchar(30) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci 1 row in set (0.00 sec)

mysql> ALTER TABLE es3 ENGINE = MyISAM;
Query OK, 0 rows affected (0.15 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> ALTER TABLE es EXCHANGE PARTITION p3sp0 WITH TABLE es3;
ERROR 1497 (HY000): The mix of handlers in the partitions is not allowed in this version of MySQL
```
