# 13.9 使用其他数据库引擎的数据类型

> 原文：[`dev.mysql.com/doc/refman/8.0/en/other-vendor-data-types.html`](https://dev.mysql.com/doc/refman/8.0/en/other-vendor-data-types.html)

为了方便使用来自其他供应商的 SQL 实现编写的代码，MySQL 将数据类型映射如下表所示。这些映射使得更容易将其他数据库系统的表定义导入到 MySQL 中。

| 其他供应商类型 | MySQL 类型 |
| --- | --- |
| `BOOL` | `TINYINT` |
| `BOOLEAN` | `TINYINT` |
| `CHARACTER VARYING(*`M`*)` | `VARCHAR(*`M`*)` |
| `FIXED` | `DECIMAL` |
| `FLOAT4` | `FLOAT` |
| `FLOAT8` | `DOUBLE` |
| `INT1` | `TINYINT` |
| `INT2` | `SMALLINT` |
| `INT3` | `MEDIUMINT` |
| `INT4` | `INT` |
| `INT8` | `BIGINT` |
| `LONG VARBINARY` | `MEDIUMBLOB` |
| `LONG VARCHAR` | `MEDIUMTEXT` |
| `LONG` | `MEDIUMTEXT` |
| `MIDDLEINT` | `MEDIUMINT` |
| `NUMERIC` | `DECIMAL` |
| 其他供应商类型 | MySQL 类型 |

数据类型映射发生在表创建时，之后原始类型规范被丢弃。如果你创建了一个使用其他供应商类型的表，然后发出一个`DESCRIBE *tbl_name*`语句，MySQL 会使用等效的 MySQL 类型报告表结构。例如：

```sql
mysql> CREATE TABLE t (a BOOL, b FLOAT8, c LONG VARCHAR, d NUMERIC);
Query OK, 0 rows affected (0.00 sec)

mysql> DESCRIBE t;
+-------+---------------+------+-----+---------+-------+
| Field | Type          | Null | Key | Default | Extra |
+-------+---------------+------+-----+---------+-------+
| a     | tinyint(1)    | YES  |     | NULL    |       |
| b     | double        | YES  |     | NULL    |       |
| c     | mediumtext    | YES  |     | NULL    |       |
| d     | decimal(10,0) | YES  |     | NULL    |       |
+-------+---------------+------+-----+---------+-------+
4 rows in set (0.01 sec)
```
