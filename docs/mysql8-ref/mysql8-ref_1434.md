> 原文：[`dev.mysql.com/doc/refman/8.0/en/replication-features-create-if-not-exists.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-features-create-if-not-exists.html)

#### 19.5.1.6 复制`CREATE ... IF NOT EXISTS`语句

当复制各种`CREATE ... IF NOT EXISTS`语句时，MySQL 会应用以下规则：

+   每个`CREATE DATABASE IF NOT EXISTS`语句都会被复制，无论源数据库是否已存在。

+   类似地，每个不带`SELECT`的`CREATE TABLE IF NOT EXISTS`语句都会被复制，无论源端表是否已存在。这包括`CREATE TABLE IF NOT EXISTS ... LIKE`。`CREATE TABLE IF NOT EXISTS ... SELECT`的复制遵循稍有不同的规则；更多信息请参见第 19.5.1.7 节，“复制`CREATE TABLE ... SELECT`语句”。

+   `CREATE EVENT IF NOT EXISTS`始终会被复制，无论语句中指定的事件是否已存在于源端。

+   只有成功的`CREATE USER`语句才会被写入二进制日志。如果语句包含`IF NOT EXISTS`，则被视为成功，并在至少创建一个在语句中命名的用户时记录；在这种情况下，语句将被记录为原样；这包括对未创建的现有用户的引用。更多信息请参见创建用户二进制日志记录。

+   (*MySQL 8.0.29 及更高版本*:) `CREATE PROCEDURE IF NOT EXISTS`、`CREATE FUNCTION IF NOT EXISTS`或`CREATE TRIGGER IF NOT EXISTS`，如果成功，将完整写入二进制日志（包括`IF NOT EXISTS`子句），无论语句是否因对象（过程、函数或触发器）已存在而引发警告。
