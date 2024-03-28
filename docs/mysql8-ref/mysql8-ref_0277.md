> 原文：[`dev.mysql.com/doc/refman/8.0/en/binary-log-mysql-database.html`](https://dev.mysql.com/doc/refman/8.0/en/binary-log-mysql-database.html)

#### 7.4.4.4 更改`mysql`数据库表的日志格式

`mysql`数据库中授权表的内容可以直接（例如，使用`INSERT`或`DELETE`）或间接（例如，使用`GRANT`或`CREATE USER`）进行修改。影响`mysql`数据库表的语句将根据以下规则写入二进制日志：

+   直接更改`mysql`数据库表中数据的数据操作语句将根据`binlog_format`系统变量的设置进行记录。这包括诸如`INSERT`、`UPDATE`、`DELETE`、`REPLACE`、`DO`、`LOAD DATA`、`SELECT`和`TRUNCATE TABLE`等语句。

+   间接更改`mysql`数据库的语句将作为语句记录，不受`binlog_format`值的影响。这包括诸如`GRANT`、`REVOKE`、`SET PASSWORD`、`RENAME USER`、`CREATE`（除`CREATE TABLE ... SELECT`之外的所有形式）、`ALTER`（所有形式）和`DROP`（所有形式）等语句。

`CREATE TABLE ... SELECT`是数据定义和数据操作的组合。`CREATE TABLE`部分使用语句格式记录，而`SELECT`部分根据`binlog_format`的值进行记录。
