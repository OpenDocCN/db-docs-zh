> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-table-exists.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-table-exists.html)

#### 30.4.4.26 table_exists() 过程

测试给定表是否存在为常规表、`TEMPORARY`表或视图。过程将表类型返回到一个`OUT`参数中。如果同名的临时表和永久表都存在，则返回`TEMPORARY`。

##### 参数

+   `in_db VARCHAR(64)`: 要检查表存在性的数据库名。

+   `in_table VARCHAR(64)`: 要检查存在性的表名。

+   `out_exists ENUM('', 'BASE TABLE', 'VIEW', 'TEMPORARY')`: 返回值。这是一个`OUT`参数，因此必须是一个可以存储表类型的变量。当过程返回时，变量具有以下值之一，指示表是否存在：

    +   `''`: 表名不存在为基本表、`TEMPORARY`表或视图。

    +   `BASE TABLE`: 表名存在为基本（永久）表。

    +   `VIEW`: 表名存在为视图。

    +   `TEMPORARY`: 表名存在为`TEMPORARY`表。

##### 示例

```sql
mysql> CREATE DATABASE db1;
Query OK, 1 row affected (0.01 sec)

mysql> USE db1;
Database changed

mysql> CREATE TABLE t1 (id INT PRIMARY KEY);
Query OK, 0 rows affected (0.03 sec)

mysql> CREATE TABLE t2 (id INT PRIMARY KEY);
Query OK, 0 rows affected (0.20 sec)

mysql> CREATE view v_t1 AS SELECT * FROM t1;
Query OK, 0 rows affected (0.02 sec)

mysql> CREATE TEMPORARY TABLE t1 (id INT PRIMARY KEY);
Query OK, 0 rows affected (0.00 sec)

mysql> CALL sys.table_exists('db1', 't1', @exists); SELECT @exists;
Query OK, 0 rows affected (0.01 sec)

+-----------+
| @exists   |
+-----------+
| TEMPORARY |
+-----------+
1 row in set (0.00 sec)

mysql> CALL sys.table_exists('db1', 't2', @exists); SELECT @exists;
Query OK, 0 rows affected (0.02 sec)

+------------+
| @exists    |
+------------+
| BASE TABLE |
+------------+
1 row in set (0.00 sec)

mysql> CALL sys.table_exists('db1', 'v_t1', @exists); SELECT @exists;
Query OK, 0 rows affected (0.02 sec)

+---------+
| @exists |
+---------+
| VIEW    |
+---------+
1 row in set (0.00 sec)

mysql> CALL sys.table_exists('db1', 't3', @exists); SELECT @exists;
Query OK, 0 rows affected (0.00 sec)

+---------+
| @exists |
+---------+
|         |
+---------+
1 row in set (0.00 sec)
```
