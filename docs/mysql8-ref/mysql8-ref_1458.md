> 原文：[`dev.mysql.com/doc/refman/8.0/en/replication-features-sql-mode.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-features-sql-mode.html)

#### 19.5.1.30 复制和服务器 SQL 模式

在源数据库和副本数据库上使用不同的服务器 SQL 模式设置可能导致相同的`INSERT`语句在源数据库和副本数据库上处理方式不同，导致源数据库和副本数据库分歧。为了获得最佳结果，您应该始终在源数据库和副本数据库上使用相同的服务器 SQL 模式。无论您使用基于语句还是基于行的复制，这些建议都适用。

如果您正在复制分区表，并且在源数据库和副本数据库上使用不同的 SQL 模式，可能会导致问题。至少，这可能导致数据在源数据库和副本数据库中的分区分布不同。这也可能导致在源数据库上成功插入分区表的数据，在副本数据库上失败。

更多信息，请参见 Section 7.1.11, “Server SQL Modes”。
