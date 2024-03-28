> 原文：[`dev.mysql.com/doc/refman/8.0/en/purge-binary-logs.html`](https://dev.mysql.com/doc/refman/8.0/en/purge-binary-logs.html)

#### 15.4.1.1 PURGE BINARY LOGS Statement

```sql
PURGE { BINARY | MASTER } LOGS {
    TO '*log_name*'
  | BEFORE *datetime_expr*
}
```

二进制日志是一组文件，其中包含 MySQL 服务器进行的数据修改的信息。该日志由一组二进制日志文件和一个索引文件组成（参见 第 7.4.4 节，“二进制日志”）。

`PURGE BINARY LOGS` 语句删除日志索引文件中指定日志文件名或日期之前列出的所有二进制日志文件。`BINARY` 和 `MASTER` 是同义词。删除的日志文件也会从索引文件中记录的列表中移除，因此给定的日志文件将成为列表中的第一个。

运行 `PURGE BINARY LOGS` 需要 `BINLOG_ADMIN` 权限。如果服务器没有使用 `--log-bin` 选项启动以启用二进制日志记录，则此语句无效。

示例：

```sql
PURGE BINARY LOGS TO 'mysql-bin.010';
PURGE BINARY LOGS BEFORE '2019-04-02 22:46:26';
```

`BEFORE` 变体的 *`datetime_expr`* 参数应该评估为一个 `DATETIME` 值（一个以 `'*`YYYY-MM-DD hh:mm:ss`*'` 格式表示的值）。

当副本正在复制时，可以安全地运行 `PURGE BINARY LOGS`。你不需要停止它们。如果你有一个活动的副本，当前正在读取你尝试删除的日志文件之一，这个语句不会删除正在使用的日志文件或比它更晚的任何日志文件，但会删除任何更早的日志文件。在这种情况下会发出警告消息。然而，如果一个副本没有连接，并且你碰巧清除了它尚未读取的日志文件之一，那么在重新连接后，该副本无法复制。

在实例生效 `LOCK INSTANCE FOR BACKUP` 语句时，不应该执行 `PURGE BINARY LOGS`，因为这违反了备份锁的规则，会从服务器中删除文件。从 MySQL 8.0.28 开始，这是不允许的。

要安全地清除二进制日志文件，请按照以下步骤进行：

1.  在每个副本上，使用 `SHOW REPLICA STATUS` 来检查它正在读取哪个日志文件。

1.  使用 `SHOW BINARY LOGS` 在源上获取二进制日志文件的列表。

1.  确定所有副本中最早的日志文件。这是目标文件。如果所有副本都是最新的，那么这是列表中的最后一个日志文件。

1.  备份即将删除的所有日志文件。（此步骤是可选的，但始终建议执行。）

1.  清除所有日志文件，但不包括目标文件。

`PURGE BINARY LOGS TO` 和 `PURGE BINARY LOGS BEFORE` 在二进制日志文件列表中的文件已经通过其他方式（比如在 Linux 上使用**rm**）被移除系统时会出现错误（Bug #18199, Bug #18453）。为了处理这种错误，需要手动编辑`.index`文件（这是一个简单的文本文件），确保它只列出实际存在的二进制日志文件，然后重新运行失败的`PURGE BINARY LOGS`语句。

服务器的二进制日志到期后会自动删除二进制日志文件。文件的删除可以在启动时和二进制日志刷新时进行。默认的二进制日志到期时间为 30 天。您可以使用`binlog_expire_logs_seconds`系统变量指定替代的到期时间。如果您正在使用复制，应该指定一个到期时间，不低于从源头落后的最长时间。
