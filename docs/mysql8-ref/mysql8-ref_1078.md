> 原文：[`dev.mysql.com/doc/refman/8.0/en/show-binlog-events.html`](https://dev.mysql.com/doc/refman/8.0/en/show-binlog-events.html)

#### 15.7.7.2 SHOW BINLOG EVENTS Statement

```sql
SHOW BINLOG EVENTS
   [IN '*log_name*']
   [FROM *pos*]
   [LIMIT [*offset*,] *row_count*]
```

显示二进制日志中的事件。如果不指定 `'*`log_name`*'`，则显示第一个二进制日志。`SHOW BINLOG EVENTS` 需要 `REPLICATION SLAVE` 权限。

`LIMIT` 子句的语法与 `SELECT` 语句相同。请参见 Section 15.2.13, “SELECT Statement”。

注意

发出不带 `LIMIT` 子句的 `SHOW BINLOG EVENTS` 可能会启动一个非常耗时和资源消耗的过程，因为服务器会将二进制日志的完整内容（包括服务器执行的修改数据的所有语句）返回给客户端。作为 `SHOW BINLOG EVENTS` 的替代方案，可以使用 **mysqlbinlog** 实用程序将二进制日志保存到文本文件以供以后检查和分析。请参见 Section 6.6.9, “mysqlbinlog — Utility for Processing Binary Log Files”。

`SHOW BINLOG EVENTS` 显示二进制日志中每个事件的以下字段：

+   `Log_name`

    正在列出的文件的名称。

+   `Pos`

    事件发生的位置。

+   `Event_type`

    描述事件类型的标识符。

+   `Server_id`

    事件发生的服务器的服务器 ID。

+   `End_log_pos`

    下一个事件开始的位置，等于 `Pos` 加上事件的大小。

+   `Info`

    有关事件类型的更详细信息。此信息的格式取决于事件类型。

对于压缩的事务负载，`Transaction_payload_event` 首先作为单个单元打印，然后解压缩并打印其中的每个事件。

与用户和系统变量设置相关的一些事件不包含在 `SHOW BINLOG EVENTS` 的输出中。要完整覆盖二进制日志中的事件，请使用 **mysqlbinlog**。

`SHOW BINLOG EVENTS` 无法与中继日志文件一起使用。您可以使用 `SHOW RELAYLOG EVENTS` 来实现此目的。
