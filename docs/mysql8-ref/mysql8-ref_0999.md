> 原文：[`dev.mysql.com/doc/refman/8.0/en/stop-replica.html`](https://dev.mysql.com/doc/refman/8.0/en/stop-replica.html)

#### 15.4.2.8 STOP REPLICA 语句

```sql
STOP REPLICA [*thread_types*] [*channel_option*]

*thread_types*:
    [*thread_type* [, *thread_type*] ... ]

*thread_type*: IO_THREAD | SQL_THREAD

*channel_option*:
    FOR CHANNEL *channel*
```

停止复制线程。从 MySQL 8.0.22 开始，使用 `STOP REPLICA` 替代已经废弃的 `STOP SLAVE`。在 MySQL 8.0.22 之前的版本中，请使用 `STOP SLAVE`。

`STOP REPLICA` 需要 `REPLICATION_SLAVE_ADMIN` 权限（或已废弃的 `SUPER` 权限）。推荐的最佳实践是在停止复制服务器之前在副本上执行 `STOP REPLICA`（有关更多信息，请参见 Section 7.1.19, “The Server Shutdown Process”）。

与 `START REPLICA` 类似，此语句可以与 `IO_THREAD` 和 `SQL_THREAD` 选项一起使用，以命名要停止的复制线程。请注意，Group Replication 应用程序通道（`group_replication_applier`）没有复制 I/O（接收器）线程，只有一个复制 SQL（应用程序）线程。因此，使用 `SQL_THREAD` 选项会完全停止此通道。

`STOP REPLICA` 导致正在进行的事务隐式提交。请参见 Section 15.3.3, “Statements That Cause an Implicit Commit”。

`gtid_next` 在执行此语句之前必须设置为 `AUTOMATIC`。

您可以通过设置系统变量 `rpl_stop_replica_timeout`（从 MySQL 8.0.26 开始）或 `rpl_stop_slave_timeout`（在 MySQL 8.0.26 之前）来控制 `STOP REPLICA` 在超时之前等待的时间。这可以用于避免 `STOP REPLICA` 与使用不同客户端连接到副本的其他 SQL 语句之间的死锁。当达到超时值时，发出命令的客户端会返回错误消息并停止等待，但 `STOP REPLICA` 指令仍然有效。一旦复制线程不再忙碌，`STOP REPLICA` 语句就会执行，副本就会停止。

在副本运行时允许一些`CHANGE REPLICATION SOURCE TO` | `CHANGE MASTER TO`语句，具体取决于复制线程的状态。但是，在这种情况下，在执行`CHANGE REPLICATION SOURCE TO` | `CHANGE MASTER TO`语句之前使用`STOP REPLICA`仍然受支持。有关更多信息，请参见第 15.4.2.3 节，“CHANGE REPLICATION SOURCE TO Statement”，第 15.4.2.1 节，“CHANGE MASTER TO Statement”和第 19.4.8 节，“故障切换期间切换源”。

可选的`FOR CHANNEL *`channel`*`子句使您可以命名语句适用于哪个复制通道。提供`FOR CHANNEL *`channel`*`子句将`STOP REPLICA`语句应用于特定的复制通道。如果未命名通道且没有额外通道存在，则该语句适用于默认通道。如果`STOP REPLICA`语句在使用多个通道时未命名通道，则此语句将停止所有通道的指定线程。有关更多信息，请参见第 19.2.2 节，“复制通道”。

Group Replication 的复制通道（`group_replication_applier`和`group_replication_recovery`）由服务器实例自动管理。`STOP REPLICA`不能与`group_replication_recovery`通道一起使用，并且仅在 Group Replication 未运行时才应与`group_replication_applier`通道一起使用。`group_replication_applier`通道仅具有一个应用程序线程，没有接收器线程，因此可以通过使用`SQL_THREAD`选项而不使用`IO_THREAD`选项来停止它。

当复制品是多线程的（`replica_parallel_workers` 或 `slave_parallel_workers` 的值不为零），从中继日志执行的事务序列中的任何间隙都将作为停止工作线程的一部分而关闭。如果复制品意外停止（例如由于工作线程中的错误或其他线程发出 `KILL`），而在执行 `STOP REPLICA` 语句时，从中继日志执行的事务序列可能变得不一致。有关更多信息，请参见 第 19.5.1.34 节，“复制和事务不一致性”。

当源使用基于行的二进制日志格式时，如果正在复制使用非事务性存储引擎的任何表，则应在关闭复制品服务器之前在复制品上执行 `STOP REPLICA` 或 `STOP REPLICA SQL_THREAD`。如果当前的复制事件组已修改了一个或多个非事务性表，`STOP REPLICA` 将等待最多 60 秒以完成事件组，除非您为复制 SQL 线程发出 `KILL QUERY` 或 `KILL CONNECTION` 语句。如果超时后事件组仍未完成，将记录错误消息。

当源使用基于语句的二进制日志格式时，在源具有打开临时表时更改源是潜在不安全的。这是为什么不建议使用基于语句的复制临时表的原因之一。您可以通过检查 `Replica_open_temp_tables` 或 `Slave_open_temp_tables` 的值来查看复制品上是否有任何临时表。在使用基于语句的复制时，在执行 `CHANGE REPLICATION SOURCE TO` | `CHANGE MASTER TO` 之前，此值应为 0。如果在复制品上有任何临时表打开，在发出 `STOP REPLICA` 后再发出 `CHANGE REPLICATION SOURCE TO` | `CHANGE MASTER TO` 语句会导致一个 `ER_WARN_OPEN_TEMP_TABLES_MUST_BE_ZERO` 警告。
