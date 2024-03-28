> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-replication-connection-status-table.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-replication-connection-status-table.html)

#### 29.12.11.2 replication_connection_status 表

此表显示处理副本连接到源的 I/O 线程的当前状态，排队在中继日志中的最后一个事务的信息，以及当前正在排队在中继日志中的事务的信息。

与`replication_connection_configuration`表相比，`replication_connection_status`更频繁更改。它包含在连接期间更改的值，而`replication_connection_configuration`包含定义副本连接到源的值，在连接期间保持不变。

`replication_connection_status`表具有以下列：

+   `CHANNEL_NAME`

    此行显示的复制通道。始终存在一个默认复制通道，并且可以添加更多复制通道。有关更多信息，请参见第 19.2.2 节，“复制通道”。

+   `GROUP_NAME`

    如果此服务器是某个组的成员，则显示服务器所属组的名称。

+   `SOURCE_UUID`

    源的`server_uuid`值。

+   `THREAD_ID`

    I/O 线程 ID。

+   `SERVICE_STATE`

    `ON`（线程存在且活动或空闲），`OFF`（线程不再存在），或`CONNECTING`（线程存在且正在连接到源）。

+   `RECEIVED_TRANSACTION_SET`

    对应于此副本接收的所有事务的全局事务 ID（GTID）集。如果未使用 GTID，则为空。有关更多信息，请参见 GTID 集。

+   `LAST_ERROR_NUMBER`，`LAST_ERROR_MESSAGE`

    导致 I/O 线程停止的最近错误的错误编号和错误消息。错误编号为 0，消息为空字符串表示“无错误”。如果`LAST_ERROR_MESSAGE`值不为空，则错误值也会出现在副本的错误日志中。

    发出`RESET MASTER`或`RESET REPLICA`会重置这些列中显示的值。

+   `LAST_ERROR_TIMESTAMP`

    以`'*`YYYY-MM-DD hh:mm:ss`*[.*`fraction`*]'`格式显示的时间戳，显示最近的 I/O 错误发生时间。

+   `LAST_HEARTBEAT_TIMESTAMP`

    一个时间戳，格式为`'*`YYYY-MM-DD hh:mm:ss`*[.*`fraction`*]'`，显示副本何时收到最近的心跳信号。

+   `COUNT_RECEIVED_HEARTBEATS`

    一个副本自上次重新启动或重置以来接收的心跳信号总数，或自上次发出`CHANGE REPLICATION SOURCE TO` | `CHANGE MASTER TO`语句以来。

+   `LAST_QUEUED_TRANSACTION`

    最后一个排队到中继日志的事务的全局事务 ID（GTID）。

+   `LAST_QUEUED_TRANSACTION_ORIGINAL_COMMIT_TIMESTAMP`

    一个时间戳，格式为`'*`YYYY-MM-DD hh:mm:ss`*[.*`fraction`*]'`，显示上一个在中继日志中排队的事务何时在原始源上提交。

+   `LAST_QUEUED_TRANSACTION_IMMEDIATE_COMMIT_TIMESTAMP`

    一个时间戳，格式为`'*`YYYY-MM-DD hh:mm:ss`*[.*`fraction`*]'`，显示上一个在中继日志中排队的事务何时在即时源上提交。

+   `LAST_QUEUED_TRANSACTION_START_QUEUE_TIMESTAMP`

    一个时间戳，格式为`'*`YYYY-MM-DD hh:mm:ss`*[.*`fraction`*]'`，显示上一个事务何时被此 I/O 线程放入中继日志队列中。

+   `LAST_QUEUED_TRANSACTION_END_QUEUE_TIMESTAMP`

    一个时间戳，格式为`'*`YYYY-MM-DD hh:mm:ss`*[.*`fraction`*]'`，显示上一个事务何时排队到中继日志文件中。

+   `QUEUEING_TRANSACTION`

    当前在中继日志中排队事务的全局事务 ID（GTID）。

+   `QUEUEING_TRANSACTION_ORIGINAL_COMMIT_TIMESTAMP`

    一个时间戳，格式为`'*`YYYY-MM-DD hh:mm:ss`*[.*`fraction`*]'`，显示当前排队事务何时在原始源上提交。

+   `QUEUEING_TRANSACTION_IMMEDIATE_COMMIT_TIMESTAMP`

    一个时间戳，格式为`'*`YYYY-MM-DD hh:mm:ss`*[.*`fraction`*]'`，显示当前排队事务何时在即时源上提交。

+   `QUEUEING_TRANSACTION_START_QUEUE_TIMESTAMP`

    一个时间戳，格式为`'*`YYYY-MM-DD hh:mm:ss`*[.*`fraction`*]'`，显示当前排队事务的第一个事件何时由此 I/O 线程写入中继日志。

当性能模式被禁用时，本地时间信息不会被收集，因此显示排队事务的开始和结束时间戳的字段为零。

`replication_connection_status`表具有以下索引：

+   主键在(`CHANNEL_NAME`)

+   在(`THREAD_ID`)上的索引

以下表显示了`replication_connection_status`列和`SHOW REPLICA STATUS`列之间的对应关系。

| `replication_connection_status`列 | `SHOW REPLICA STATUS`列 |
| --- | --- |
| `SOURCE_UUID` | `Master_UUID` |
| `THREAD_ID` | None |
| `SERVICE_STATE` | `Replica_IO_Running` |
| `RECEIVED_TRANSACTION_SET` | `Retrieved_Gtid_Set` |
| `LAST_ERROR_NUMBER` | `Last_IO_Errno` |
| `LAST_ERROR_MESSAGE` | `Last_IO_Error` |
| `LAST_ERROR_TIMESTAMP` | `Last_IO_Error_Timestamp` |
