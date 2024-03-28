# 26.2.2 LIST 分区

> 译文：[`dev.mysql.com/doc/refman/8.0/en/partitioning-list.html`](https://dev.mysql.com/doc/refman/8.0/en/partitioning-list.html)

MySQL 中的列表分区在许多方面类似于范围分区。与按`RANGE`分区一样，每个分区必须明确定义。两种分区类型之间的主要区别在于，在列表分区中，每个分区是根据列值在一组值列表中的成员资格而定义和选择的，而不是在一组连续值范围中的一个。这是通过使用`PARTITION BY LIST(*`expr`*)`来完成的，其中*`expr`*是一个列值或基于列值并返回整数值的表达式，然后通过`VALUES IN (*`value_list`*)`来定义每个分区，其中*`value_list`*是一个逗号分隔的整数列表。

注意

在 MySQL 8.0 中，当按`LIST`分区时，只能匹配一组整数（可能包括`NULL`—请参见第 26.2.7 节，“MySQL 分区如何处理 NULL”）。

但是，在使用`LIST COLUMN`分区时，可以在值列表中使用其他列类型，该分区稍后在本节中描述。

与按范围定义的分区不同，列表分区不需要按任何特定顺序声明。有关更详细的语法信息，请参见第 15.1.20 节，“CREATE TABLE Statement”。

对于接下来的示例，我们假设要分区的表的基本定义由此处显示的`CREATE TABLE`语句提供：

```sql
CREATE TABLE employees (
    id INT NOT NULL,
    fname VARCHAR(30),
    lname VARCHAR(30),
    hired DATE NOT NULL DEFAULT '1970-01-01',
    separated DATE NOT NULL DEFAULT '9999-12-31',
    job_code INT,
    store_id INT
);
```

（这是用作第 26.2.1 节，“RANGE 分区”示例基础的相同表。与其他分区示例一样，我们假设`default_storage_engine`为`InnoDB`。）

假设有 20 家视频商店分布在 4 个特许经营店中，如下表所示。

| 地区 | 商店 ID 号码 |
| --- | --- |
| 北部 | 3, 5, 6, 9, 17 |
| 东部 | 1, 2, 10, 11, 19, 20 |
| 西部 | 4, 12, 13, 14, 18 |
| 中部 | 7, 8, 15, 16 |

要将此表分区，使属于同一地区的商店行存储在同一分区中，您可以使用此处显示的`CREATE TABLE`语句：

```sql
CREATE TABLE employees (
    id INT NOT NULL,
    fname VARCHAR(30),
    lname VARCHAR(30),
    hired DATE NOT NULL DEFAULT '1970-01-01',
    separated DATE NOT NULL DEFAULT '9999-12-31',
    job_code INT,
    store_id INT
)
PARTITION BY LIST(store_id) (
    PARTITION pNorth VALUES IN (3,5,6,9,17),
    PARTITION pEast VALUES IN (1,2,10,11,19,20),
    PARTITION pWest VALUES IN (4,12,13,14,18),
    PARTITION pCentral VALUES IN (7,8,15,16)
);
```

这使得很容易向表中添加或删除与特定区域相关的员工记录。例如，假设西区的所有商店都被卖给另一家公司。在 MySQL 8.0 中，可以使用查询`ALTER TABLE employees TRUNCATE PARTITION pWest`删除与该区域商店工作的员工相关的所有行，这比等效的`DELETE`语句`DELETE FROM employees WHERE store_id IN (4,12,13,14,18);`执行效率更高。（使用`ALTER TABLE employees DROP PARTITION pWest`也会删除所有这些行，但也会从表的定义中删除分区`pWest`；您需要使用`ALTER TABLE ... ADD PARTITION`语句来恢复表的原始分区方案。）

与`RANGE`分区一样，可以将`LIST`分区与哈希或键分区组合以生成复合分区（子分区）。请参见第 26.2.6 节，“子分区”。

与`RANGE`分区不同，没有像`MAXVALUE`这样的“捕获所有”；分区表达式的所有预期值应该在`PARTITION ... VALUES IN (...)`子句中涵盖。包含不匹配的分区列值的`INSERT`语句会失败并显示错误，如下例所示：

```sql
mysql> CREATE TABLE h2 (
 ->   c1 INT,
 ->   c2 INT
 -> )
 -> PARTITION BY LIST(c1) (
 ->   PARTITION p0 VALUES IN (1, 4, 7),
 ->   PARTITION p1 VALUES IN (2, 5, 8)
 -> );
Query OK, 0 rows affected (0.11 sec)

mysql> INSERT INTO h2 VALUES (3, 5);
ERROR 1525 (HY000): Table has no partition for value 3
```

当使用单个`INSERT`语句将多行插入单个`InnoDB`表时，`InnoDB`将该语句视为单个事务，因此任何不匹配的值的存在都会导致该语句完全失败，因此不会插入任何行。

你可以通过使用`IGNORE`关键字来忽略这种类型的错误，尽管对于每一行包含不匹配的分区列值的情况会发出警告，如下所示。

```sql
mysql> TRUNCATE h2;
Query OK, 1 row affected (0.00 sec)

mysql> TABLE h2;
Empty set (0.00 sec)

mysql> INSERT IGNORE INTO h2 VALUES (2, 5), (6, 10), (7, 5), (3, 1), (1, 9);
Query OK, 3 rows affected, 2 warnings (0.01 sec)
Records: 5  Duplicates: 2  Warnings: 2

mysql> SHOW WARNINGS;
+---------+------+------------------------------------+
| Level   | Code | Message                            |
+---------+------+------------------------------------+
| Warning | 1526 | Table has no partition for value 6 |
| Warning | 1526 | Table has no partition for value 3 |
+---------+------+------------------------------------+
2 rows in set (0.00 sec)
```

您可以在以下`TABLE`语句的输出中看到，包含不匹配的分区列值的行被静默拒绝，而不包含不匹配值的行被插入到表中：

```sql
mysql> TABLE h2;
+------+------+
| c1   | c2   |
+------+------+
|    7 |    5 |
|    1 |    9 |
|    2 |    5 |
+------+------+
3 rows in set (0.00 sec)
```

MySQL 还支持`LIST COLUMNS`分区，这是`LIST`分区的一种变体，允许您使用除整数以外的其他类型的列作为分区列，并使用多个列作为分区键。有关更多信息，请参见第 26.2.3.2 节，“LIST COLUMNS 分区”。
