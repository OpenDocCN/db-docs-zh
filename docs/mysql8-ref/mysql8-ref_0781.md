# 13.4.10 创建空间索引

> 原文：[`dev.mysql.com/doc/refman/8.0/en/creating-spatial-indexes.html`](https://dev.mysql.com/doc/refman/8.0/en/creating-spatial-indexes.html)

对于`InnoDB`和`MyISAM`表，MySQL 可以使用类似于创建常规索引的语法来创建空间索引，但使用`SPATIAL`关键字。空间索引中的列必须声明为`NOT NULL`。以下示例演示了如何创建空间索引：

+   使用`CREATE TABLE`：

    ```sql
    CREATE TABLE geom (g GEOMETRY NOT NULL SRID 4326, SPATIAL INDEX(g));
    ```

+   使用`ALTER TABLE`：

    ```sql
    CREATE TABLE geom (g GEOMETRY NOT NULL SRID 4326);
    ALTER TABLE geom ADD SPATIAL INDEX(g);
    ```

+   使用`CREATE INDEX`：

    ```sql
    CREATE TABLE geom (g GEOMETRY NOT NULL SRID 4326);
    CREATE SPATIAL INDEX g ON geom (g);
    ```

`SPATIAL INDEX`创建一个 R 树索引。对于支持对空间列进行非空间索引的存储引擎，引擎会创建一个 B 树索引。对空间值创建 B 树索引对于精确值查找很有用，但不适用于范围扫描。

优化器可以使用在受 SRID 限制的列上定义的空间索引。有关更多信息，请参见第 13.4.1 节，“空间数据类型”和第 10.3.3 节，“空间索引优化”。

有关在空间列上创建索引的更多信息，请参见第 15.1.15 节，“CREATE INDEX Statement”。

要删除空间索引，请使用`ALTER TABLE`或`DROP INDEX`：

+   使用`ALTER TABLE`：

    ```sql
    ALTER TABLE geom DROP INDEX g;
    ```

+   使用`DROP INDEX`：

    ```sql
    DROP INDEX g ON geom;
    ```

例如：假设一个表`geom`包含超过 32,000 个几何图形，这些图形存储在类型为`GEOMETRY`的列`g`中。该表还有一个`AUTO_INCREMENT`列`fid`用于存储对象 ID 值。

```sql
mysql> DESCRIBE geom;
+-------+----------+------+-----+---------+----------------+
| Field | Type     | Null | Key | Default | Extra          |
+-------+----------+------+-----+---------+----------------+
| fid   | int(11)  |      | PRI | NULL    | auto_increment |
| g     | geometry |      |     |         |                |
+-------+----------+------+-----+---------+----------------+
2 rows in set (0.00 sec)

mysql> SELECT COUNT(*) FROM geom;
+----------+
| count(*) |
+----------+
|    32376 |
+----------+
1 row in set (0.00 sec)
```

要在列`g`上添加空间索引，请使用以下语句：

```sql
mysql> ALTER TABLE geom ADD SPATIAL INDEX(g);
Query OK, 32376 rows affected (4.05 sec)
Records: 32376  Duplicates: 0  Warnings: 0
```
