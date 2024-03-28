> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-replication-applier-status-by-coordinator-table.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-replication-applier-status-by-coordinator-table.html)

#### 29.12.11.7 复制 _applier_status_by_coordinator 表

对于多线程复制，复制使用多个工作线程和一个协调线程来管理它们，此表显示协调线程的状态。对于单线程复制，此表为空。对于多线程复制，`replication_applier_status_by_worker` 表显示工作线程的状态。此表提供有关由协调线程缓冲到工作线程队列的最后一个事务的信息，以及它当前正在缓冲的事务。开始时间戳指的是此线程从中继日志读取事务的第一个事件并将其缓冲到工作线程队列的时间，而结束时间戳指的是最后一个事件完成缓冲到工作线程队列的时间。

`replication_applier_status_by_coordinator` 表具有以下列：

+   `CHANNEL_NAME`

    此行显示的复制通道。始终存在默认复制通道，并且可以添加更多复制通道。有关更多信息，请参见 Section 19.2.2, “Replication Channels”。

+   `THREAD_ID`

    SQL/协调线程 ID。

+   `SERVICE_STATE`

    `ON`（线程存在且活动或空闲）或`OFF`（线程已不存在）。

+   `LAST_ERROR_NUMBER`, `LAST_ERROR_MESSAGE`

    导致 SQL/协调线程停止的最近错误的错误编号和错误消息。错误编号为 0，消息为空字符串表示“无错误”。如果`LAST_ERROR_MESSAGE`值不为空，则错误值也会出现在复制的错误日志中。

    发出`RESET MASTER`或`RESET REPLICA`将重置这些列中显示的值。

    所有显示在`LAST_ERROR_NUMBER`和`LAST_ERROR_MESSAGE`列中的错误代码和消息对应于服务器错误消息参考中列出的错误值。

+   `LAST_ERROR_TIMESTAMP`

    以`'*`YYYY-MM-DD hh:mm:ss`*[.*`fraction`*]'`格式的时间戳，显示最近的 SQL/协调器错误发生的时间。

+   `LAST_PROCESSED_TRANSACTION`

    此协调器处理的最后一个事务的全局事务 ID（GTID）。

+   `LAST_PROCESSED_TRANSACTION_ORIGINAL_COMMIT_TIMESTAMP`

    一个时间戳，采用`'*`YYYY-MM-DD hh:mm:ss`*[.*`fraction`*]'`格式，显示此协调器处理的最后一个事务在原始源上提交的时间。

+   `LAST_PROCESSED_TRANSACTION_IMMEDIATE_COMMIT_TIMESTAMP`

    一个时间戳，采用`'*`YYYY-MM-DD hh:mm:ss`*[.*`fraction`*]'`格式，显示此协调器处理的最后一个事务在即时源上提交的时间。

+   `LAST_PROCESSED_TRANSACTION_START_BUFFER_TIMESTAMP`

    一个时间戳，采用`'*`YYYY-MM-DD hh:mm:ss`*[.*`fraction`*]'`格式，显示此协调器线程开始将最后一个事务写入工作线程的缓冲区的时间。

+   `LAST_PROCESSED_TRANSACTION_END_BUFFER_TIMESTAMP`

    一个时间戳，采用`'*`YYYY-MM-DD hh:mm:ss`*[.*`fraction`*]'`格式，显示此协调器线程将最后一个事务写入工作线程的缓冲区的时间。

+   `PROCESSING_TRANSACTION`

    此协调器线程当前正在处理的事务的全局事务 ID（GTID）。

+   `PROCESSING_TRANSACTION_ORIGINAL_COMMIT_TIMESTAMP`

    一个时间戳，采用`'*`YYYY-MM-DD hh:mm:ss`*[.*`fraction`*]'`格式，显示当前处理事务在原始源上提交的时间。

+   `PROCESSING_TRANSACTION_IMMEDIATE_COMMIT_TIMESTAMP`

    一个时间戳，采用`'*`YYYY-MM-DD hh:mm:ss`*[.*`fraction`*]'`格式，显示当前处理事务在即时源上提交的时间。

+   `PROCESSING_TRANSACTION_START_BUFFER_TIMESTAMP`

    一个时间戳，采用`'*`YYYY-MM-DD hh:mm:ss`*[.*`fraction`*]'`格式，显示此协调器线程开始将当前处理事务写入工作线程的缓冲区的时间。

当性能模式被禁用时，本地时间信息不会被收集，因此显示缓冲事务的开始和结束时间戳的字段为零。

`replication_applier_status_by_coordinator`表具有以下索引：

+   在(`CHANNEL_NAME`)上的主键

+   在(`THREAD_ID`)上的索引

以下表显示`replication_applier_status_by_coordinator`列与`SHOW REPLICA STATUS`列之间的对应关系。

| `replication_applier_status_by_coordinator`列 | `SHOW REPLICA STATUS`列 |
| --- | --- |
| `THREAD_ID` | None |
| `SERVICE_STATE` | `Replica_SQL_Running` |
| `LAST_ERROR_NUMBER` | `Last_SQL_Errno` |
| `LAST_ERROR_MESSAGE` | `Last_SQL_Error` |
| `LAST_ERROR_TIMESTAMP` | `Last_SQL_Error_Timestamp` |
