> 原文：[`dev.mysql.com/doc/refman/8.0/en/replication-features-flush.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-features-flush.html)

#### 19.5.1.13 复制和 FLUSH

一些形式的`FLUSH`语句不会被记录，因为如果被复制到副本中可能会引起问题：`FLUSH LOGS`和`FLUSH TABLES WITH READ LOCK`。有关语法示例，请参见 Section 15.7.8.3, “FLUSH Statement”。`FLUSH TABLES`、`ANALYZE TABLE`、`OPTIMIZE TABLE`和`REPAIR TABLE`语句会被写入二进制日志，因此会被复制到副本中。通常这不是问题，因为这些语句不会修改表数据。

然而，在某些情况下，这种行为可能会引起困难。如果在`mysql`数据库中复制权限表并直接更新这些表而不使用`GRANT`，则必须在副本上发出`FLUSH PRIVILEGES`以使新权限生效。此外，如果在重命名作为`MERGE`表的`MyISAM`表时使用`FLUSH TABLES`，则必须在副本上手动发出`FLUSH TABLES`。除非指定`NO_WRITE_TO_BINLOG`或其别名`LOCAL`，否则这些语句将被写入二进制日志。
