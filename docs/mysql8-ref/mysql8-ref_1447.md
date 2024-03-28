> 原文：[`dev.mysql.com/doc/refman/8.0/en/replication-features-load-data.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-features-load-data.html)

#### 19.5.1.19 复制和 LOAD DATA

`LOAD DATA` 被认为在基于语句的日志记录中是不安全的（参见 Section 19.2.1.3, “二进制日志中安全和不安全语句的确定”）。当设置 `binlog_format=MIXED` 时，该语句以行格式记录。当设置 `binlog_format=STATEMENT` 时，请注意，与其他不安全语句不同，`LOAD DATA` 不会生成警告。

如果您使用 `LOAD DATA` 与 `binlog_format=STATEMENT`，每个要应用更改的副本都会创建一个包含数据的临时文件。然后，副本使用 `LOAD DATA` 语句来应用更改。即使源上启用了二进制日志加密，此临时文件也不会被加密。如果需要加密，请改用基于行或混合的二进制日志格式，副本不会创建临时文件。

如果使用 `PRIVILEGE_CHECKS_USER` 帐户来帮助保护复制通道（参见 Section 19.3.3, “复制权限检查”），强烈建议您使用基于行的二进制日志记录（`binlog_format=ROW`）记录 `LOAD DATA` 操作。如果为通道设置了 `REQUIRE_ROW_FORMAT`，则需要基于行的二进制日志记录。使用此日志格式，执行事件不需要 `FILE` 权限，因此不要给 `PRIVILEGE_CHECKS_USER` 帐户此权限。如果需要从以语句格式记录的 `LOAD DATA INFILE` 操作的复制错误中恢复，并且复制的事件是可信的，则可以暂时授予 `PRIVILEGE_CHECKS_USER` 帐户 `FILE` 权限，在应用复制的事件后再删除该权限。

当**mysqlbinlog**读取以语句为基础格式记录的`LOAD DATA`语句的日志事件时，会在临时目录中创建一个生成的本地文件。这些临时文件不会被**mysqlbinlog**或任何其他 MySQL 程序自动删除。如果您确实使用了以语句为基础的二进制日志记录的`LOAD DATA`语句，那么在不再需要语句日志后，您应该自行删除这些临时文件。更多信息，请参阅 Section 6.6.9, “mysqlbinlog — 用于处理二进制日志文件的实用程序”。
