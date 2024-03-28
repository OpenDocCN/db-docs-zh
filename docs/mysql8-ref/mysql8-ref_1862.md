# 26.6.1 分区键、主键和唯一键

> 原文：[`dev.mysql.com/doc/refman/8.0/en/partitioning-limitations-partitioning-keys-unique-keys.html`](https://dev.mysql.com/doc/refman/8.0/en/partitioning-limitations-partitioning-keys-unique-keys.html)

本节讨论了分区键与主键和唯一键的关系。规定这种关系的规则可以表述如下：用于分区表的分区表达式中的所有列必须是表可能具有的每个唯一键的一部分。

换句话说，*表上的每个唯一键都必须使用表的分区表达式中的每一列*。（这也包括表的主键，因为根据定义，它是一个唯一键。这种特殊情况稍后在本节中讨论。）例如，以下每个表创建语句都是无效的：

```sql
CREATE TABLE t1 (
    col1 INT NOT NULL,
    col2 DATE NOT NULL,
    col3 INT NOT NULL,
    col4 INT NOT NULL,
    UNIQUE KEY (col1, col2)
)
PARTITION BY HASH(col3)
PARTITIONS 4;

CREATE TABLE t2 (
    col1 INT NOT NULL,
    col2 DATE NOT NULL,
    col3 INT NOT NULL,
    col4 INT NOT NULL,
    UNIQUE KEY (col1),
    UNIQUE KEY (col3)
)
PARTITION BY HASH(col1 + col3)
PARTITIONS 4;
```

在每种情况下，建议的表至少应该有一个不包括分区表达式中使用的所有列的唯一键。

每个以下语句都是有效的，并代表了相应的无效表创建语句可以正常工作的一种方式：

```sql
CREATE TABLE t1 (
    col1 INT NOT NULL,
    col2 DATE NOT NULL,
    col3 INT NOT NULL,
    col4 INT NOT NULL,
    UNIQUE KEY (col1, col2, col3)
)
PARTITION BY HASH(col3)
PARTITIONS 4;

CREATE TABLE t2 (
    col1 INT NOT NULL,
    col2 DATE NOT NULL,
    col3 INT NOT NULL,
    col4 INT NOT NULL,
    UNIQUE KEY (col1, col3)
)
PARTITION BY HASH(col1 + col3)
PARTITIONS 4;
```

这个例子展示了这种情况下产生的错误：

```sql
mysql> CREATE TABLE t3 (
 ->     col1 INT NOT NULL,
 ->     col2 DATE NOT NULL,
 ->     col3 INT NOT NULL,
 ->     col4 INT NOT NULL,
 ->     UNIQUE KEY (col1, col2),
 ->     UNIQUE KEY (col3)
 -> )
 -> PARTITION BY HASH(col1 + col3)
 -> PARTITIONS 4;
ERROR 1491 (HY000): A PRIMARY KEY must include all columns in the table's partitioning function
```

`CREATE TABLE` 语句失败，因为`col1`和`col3`都包含在建议的分区键中，但这两列都不是表上的两个唯一键的一部分。这显示了无效表定义的一种可能修复方法：

```sql
mysql> CREATE TABLE t3 (
 ->     col1 INT NOT NULL,
 ->     col2 DATE NOT NULL,
 ->     col3 INT NOT NULL,
 ->     col4 INT NOT NULL,
 ->     UNIQUE KEY (col1, col2, col3),
 ->     UNIQUE KEY (col3)
 -> )
 -> PARTITION BY HASH(col3)
 -> PARTITIONS 4;
Query OK, 0 rows affected (0.05 sec)
```

在这种情况下，建议的分区键`col3`是两个唯一键的一部分，表创建语句成功。

以下表根本无法分区，因为无法在分区键中包含任何同时属于唯一键的列：

```sql
CREATE TABLE t4 (
    col1 INT NOT NULL,
    col2 INT NOT NULL,
    col3 INT NOT NULL,
    col4 INT NOT NULL,
    UNIQUE KEY (col1, col3),
    UNIQUE KEY (col2, col4)
);
```

由于每个主键根据定义都是唯一键，如果表有主键，这个限制也包括表的主键。例如，下面的两个语句是无效的：

```sql
CREATE TABLE t5 (
    col1 INT NOT NULL,
    col2 DATE NOT NULL,
    col3 INT NOT NULL,
    col4 INT NOT NULL,
    PRIMARY KEY(col1, col2)
)
PARTITION BY HASH(col3)
PARTITIONS 4;

CREATE TABLE t6 (
    col1 INT NOT NULL,
    col2 DATE NOT NULL,
    col3 INT NOT NULL,
    col4 INT NOT NULL,
    PRIMARY KEY(col1, col3),
    UNIQUE KEY(col2)
)
PARTITION BY HASH( YEAR(col2) )
PARTITIONS 4;
```

在这两种情况下，主键都不包括分区表达式中引用的所有列。然而，下面的两个语句都是有效的：

```sql
CREATE TABLE t7 (
    col1 INT NOT NULL,
    col2 DATE NOT NULL,
    col3 INT NOT NULL,
    col4 INT NOT NULL,
    PRIMARY KEY(col1, col2)
)
PARTITION BY HASH(col1 + YEAR(col2))
PARTITIONS 4;

CREATE TABLE t8 (
    col1 INT NOT NULL,
    col2 DATE NOT NULL,
    col3 INT NOT NULL,
    col4 INT NOT NULL,
    PRIMARY KEY(col1, col2, col4),
    UNIQUE KEY(col2, col1)
)
PARTITION BY HASH(col1 + YEAR(col2))
PARTITIONS 4;
```

如果一个表没有唯一键——这包括没有主键——那么这个限制就不适用，你可以使用分区表达式中的任何列或多列，只要列的类型与分区类型兼容。

出于同样的原因，除非该键包含表的分区表达式中使用的所有列，否则您不能随后向分区表添加唯一键。考虑在此处创建的分区表：

```sql
mysql> CREATE TABLE t_no_pk (c1 INT, c2 INT)
 ->     PARTITION BY RANGE(c1) (
 ->         PARTITION p0 VALUES LESS THAN (10),
 ->         PARTITION p1 VALUES LESS THAN (20),
 ->         PARTITION p2 VALUES LESS THAN (30),
 ->         PARTITION p3 VALUES LESS THAN (40)
 ->     );
Query OK, 0 rows affected (0.12 sec)
```

可以使用以下任一`ALTER TABLE`语句向`t_no_pk`添加主键：

```sql
#  possible PK
mysql> ALTER TABLE t_no_pk ADD PRIMARY KEY(c1);
Query OK, 0 rows affected (0.13 sec)
Records: 0  Duplicates: 0  Warnings: 0

# drop this PK
mysql> ALTER TABLE t_no_pk DROP PRIMARY KEY;
Query OK, 0 rows affected (0.10 sec)
Records: 0  Duplicates: 0  Warnings: 0

#  use another possible PK
mysql> ALTER TABLE t_no_pk ADD PRIMARY KEY(c1, c2);
Query OK, 0 rows affected (0.12 sec)
Records: 0  Duplicates: 0  Warnings: 0

# drop this PK
mysql> ALTER TABLE t_no_pk DROP PRIMARY KEY;
Query OK, 0 rows affected (0.09 sec)
Records: 0  Duplicates: 0  Warnings: 0
```

然而，下一个语句失败，因为`c1`是分区键的一部分，但不是建议的主键的一部分：

```sql
#  fails with error 1503
mysql> ALTER TABLE t_no_pk ADD PRIMARY KEY(c2);
ERROR 1503 (HY000): A PRIMARY KEY must include all columns in the table's partitioning function
```

由于`t_no_pk`只有`c1`在其分区表达式中，尝试仅在`c2`上添加唯一键将失败。但是，您可以添加一个使用`c1`和`c2`的唯一键。

这些规则也适用于您希望使用`ALTER TABLE ... PARTITION BY`对现有的非分区表进行分区的情况。考虑一个如下所示创建的表`np_pk`：

```sql
mysql> CREATE TABLE np_pk (
 ->     id INT NOT NULL AUTO_INCREMENT,
 ->     name VARCHAR(50),
 ->     added DATE,
 ->     PRIMARY KEY (id)
 -> );
Query OK, 0 rows affected (0.08 sec)
```

以下`ALTER TABLE`语句将因为`added`列不是表中任何唯一键的一部分而失败：

```sql
mysql> ALTER TABLE np_pk
 ->     PARTITION BY HASH( TO_DAYS(added) )
 ->     PARTITIONS 4;
ERROR 1503 (HY000): A PRIMARY KEY must include all columns in the table's partitioning function
```

然而，使用`id`列作为分区列的语句是有效的，如下所示：

```sql
mysql> ALTER TABLE np_pk
 ->     PARTITION BY HASH(id)
 ->     PARTITIONS 4;
Query OK, 0 rows affected (0.11 sec)
Records: 0  Duplicates: 0  Warnings: 0
```

对于`np_pk`，唯一可以用作分区表达式的列是`id`；如果您希望使用分区表达式中的任何其他列对该表进行分区，您必须首先修改表，可以通过将所需的列添加到主键中或完全删除主键。
