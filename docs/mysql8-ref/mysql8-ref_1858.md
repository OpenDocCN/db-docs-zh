# 26.3.5 获取有关分区的信息

> 原文：[`dev.mysql.com/doc/refman/8.0/en/partitioning-info.html`](https://dev.mysql.com/doc/refman/8.0/en/partitioning-info.html)

本节讨论获取有关现有分区的信息，可以通过多种方式进行。获取此类信息的方法包括以下内容：

+   使用`SHOW CREATE TABLE`语句查看创建分区表时使用的分区子句。

+   使用`SHOW TABLE STATUS`语句来确定表是否被分区。

+   查询信息模式`PARTITIONS`表。

+   使用语句`EXPLAIN SELECT`来查看给定`SELECT`使用的分区。

从 MySQL 8.0.16 开始，当对分区表进行插入、删除或更新操作时，二进制日志记录有关发生行事件的分区和（如果有）子分区的信息。即使涉及的表相同，也为发生在不同分区或子分区中的修改创建新的行事件。因此，如果一个事务涉及三个分区或子分区，将生成三个行事件。对于更新事件，分区信息被记录在“之前”图像和“之后”图像中。如果在使用**mysqlbinlog**查看二进制日志时指定了`-v`或`--verbose`选项，则会显示分区信息。仅当使用基于行的日志记录时（`binlog_format=ROW`）才记录分区信息。

如本章其他地方所讨论的，`SHOW CREATE TABLE`在其输出中包含用于创建分区表的`PARTITION BY`子句。例如：

```sql
mysql> SHOW CREATE TABLE trb3\G
*************************** 1\. row ***************************
       Table: trb3
Create Table: CREATE TABLE `trb3` (
  `id` int(11) DEFAULT NULL,
  `name` varchar(50) DEFAULT NULL,
  `purchased` date DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4
/*!50100 PARTITION BY RANGE (YEAR(purchased))
(PARTITION p0 VALUES LESS THAN (1990) ENGINE = InnoDB,
 PARTITION p1 VALUES LESS THAN (1995) ENGINE = InnoDB,
 PARTITION p2 VALUES LESS THAN (2000) ENGINE = InnoDB,
 PARTITION p3 VALUES LESS THAN (2005) ENGINE = InnoDB) */ 0 row in set (0.00 sec)
```

对于分区表，`SHOW TABLE STATUS`的输出与非分区表相同，只是`Create_options`列包含字符串`partitioned`。`Engine`列包含表的所有分区使用的存储引擎的名称。（有关此语句的更多信息，请参见第 15.7.7.38 节，“SHOW TABLE STATUS Statement”。）

您还可以从`INFORMATION_SCHEMA`中获取有关分区的信息，其中包含一个`PARTITIONS`表。请参阅第 28.3.21 节，“INFORMATION_SCHEMA PARTITIONS 表”。

可以通过使用`解释`来确定分区表中哪些分区涉及给定的`SELECT`查询。`解释`输出中的`partitions`列列出了查询将匹配的记录的分区。

假设创建并填充了一个名为`trb1`的表如下：

```sql
CREATE TABLE trb1 (id INT, name VARCHAR(50), purchased DATE)
    PARTITION BY RANGE(id)
    (
        PARTITION p0 VALUES LESS THAN (3),
        PARTITION p1 VALUES LESS THAN (7),
        PARTITION p2 VALUES LESS THAN (9),
        PARTITION p3 VALUES LESS THAN (11)
    );

INSERT INTO trb1 VALUES
    (1, 'desk organiser', '2003-10-15'),
    (2, 'CD player', '1993-11-05'),
    (3, 'TV set', '1996-03-10'),
    (4, 'bookcase', '1982-01-10'),
    (5, 'exercise bike', '2004-05-09'),
    (6, 'sofa', '1987-06-05'),
    (7, 'popcorn maker', '2001-11-22'),
    (8, 'aquarium', '1992-08-04'),
    (9, 'study desk', '1984-09-16'),
    (10, 'lava lamp', '1998-12-25');
```

你可以看到在查询中使用了哪些分区，比如`SELECT * FROM trb1;`，如下所示：

```sql
mysql> EXPLAIN SELECT * FROM trb1\G
*************************** 1\. row ***************************
           id: 1
  select_type: SIMPLE
        table: trb1
   partitions: p0,p1,p2,p3
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 10
        Extra: Using filesort
```

在这种情况下，所有四个分区都被搜索。然而，当在查询中添加使用分区键的限制条件时，您可以看到只有包含匹配值的分区被扫描，如下所示：

```sql
mysql> EXPLAIN SELECT * FROM trb1 WHERE id < 5\G
*************************** 1\. row ***************************
           id: 1
  select_type: SIMPLE
        table: trb1
   partitions: p0,p1
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 10
        Extra: Using where
```

`解释`还提供了有关使用的键和可能的键的信息：

```sql
mysql> ALTER TABLE trb1 ADD PRIMARY KEY (id);
Query OK, 10 rows affected (0.03 sec)
Records: 10  Duplicates: 0  Warnings: 0

mysql> EXPLAIN SELECT * FROM trb1 WHERE id < 5\G
*************************** 1\. row ***************************
           id: 1
  select_type: SIMPLE
        table: trb1
   partitions: p0,p1
         type: range
possible_keys: PRIMARY
          key: PRIMARY
      key_len: 4
          ref: NULL
         rows: 7
        Extra: Using where
```

如果使用`解释`来检查针对非分区表的查询，不会产生错误，但`partitions`列的值始终为`NULL`。

`解释`输出的`rows`列显示表中的总行数。

另请参阅第 15.8.2 节，“解释语句”。
