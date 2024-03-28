# 1.6.2 MySQL 与标准 SQL 的差异

> 原文：[`dev.mysql.com/doc/refman/8.0/en/differences-from-ansi.html`](https://dev.mysql.com/doc/refman/8.0/en/differences-from-ansi.html)

1.6.2.1 SELECT INTO TABLE 差异

1.6.2.2 UPDATE 差异

1.6.2.3 外键约束的差异

1.6.2.4 '--' 作为注释的起始

我们努力使 MySQL Server 遵循 ANSI SQL 标准和 ODBC SQL 标准，但在某些情况下，MySQL Server 的操作方式有所不同：

+   MySQL 和标准 SQL 权限系统之间存在几个不同之处。例如，在 MySQL 中，当你删除一个表时，该表的权限不会自动被撤销。你必须明确地发出一个`REVOKE`语句来撤销表的权限。更多信息，请参见 Section 15.7.1.8, “REVOKE Statement”。

+   `CAST()`函数不支持转换为`REAL` - FLOAT, DOUBLE")或`BIGINT` - INTEGER, INT, SMALLINT, TINYINT, MEDIUMINT, BIGINT")。请参见 Section 14.10, “Cast Functions and Operators”。
