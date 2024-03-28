> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-replication-applier-status-by-worker-table.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-replication-applier-status-by-worker-table.html)

#### 29.12.11.8 复制 _applier_status_by_worker 表

此表提供了在副本或组复制组成员上由应用程序线程处理的事务的详细信息。对于单线程副本，显示副本的单个应用程序线程的数据。对于多线程副本，分别显示每个应用程序线程的数据。多线程副本上的应用程序线程有时称为工作者。副本或组复制组成员上的应用程序线程数量由`replica_parallel_workers`或`slave_parallel_workers`系统变量设置，对于单线程副本，该变量设置为零。多线程副本还有一个协调器线程来管理应用程序线程，并且此线程的状态显示在`replication_applier_status_by_coordinator`表中。

所有与错误相关的列中显示的错误代码和消息对应于服务器错误消息参考中列出的错误值。

当关闭性能模式时，不会收集本地时间信息，因此显示应用事务的开始和结束时间戳的字段为零。此表中的开始时间戳指的是工作者开始应用第一个事件的时间，结束时间戳指的是应用事务的最后一个事件的时间。

当副本通过`START REPLICA`语句重新启动时，以`APPLYING_TRANSACTION`开头的列将被重置。在 MySQL 8.0.13 之前，这些列在单线程模式下运��的副本上不会被重置，只有在多线程副本上才会被重置。

`replication_applier_status_by_worker`表具有以下列：

+   `CHANNEL_NAME`

    此行显示的复制通道。始终存在默认复制通道，并且可以添加更多复制通道。有关更多信息，请参见 Section 19.2.2, “Replication Channels”。

+   `WORKER_ID`

    工作者标识符（与`mysql.slave_worker_info`表中的`id`列相同）。在`STOP REPLICA`之后，`THREAD_ID`列变为`NULL`，但`WORKER_ID`值保持不变。

+   `THREAD_ID`

    工作者线程 ID。

+   `SERVICE_STATE`

    `ON`（线程存在且活动或空闲）或`OFF`（线程不再存在）。

+   `LAST_ERROR_NUMBER`，`LAST_ERROR_MESSAGE`

    导致工作线程停止的最近错误的错误编号和错误消息。错误编号为 0 且消息为空字符串表示“无错误”。如果`LAST_ERROR_MESSAGE`值不为空，则错误值也会出现在副本的错误日志中。

    发出`RESET MASTER`或`RESET REPLICA`将重置这些列中显示的值。

+   `LAST_ERROR_TIMESTAMP`

    一个时间戳，格式为`'*`YYYY-MM-DD hh:mm:ss`*[.*`fraction`*]'`，显示最近发生的工作程序错误的时间。

+   `LAST_APPLIED_TRANSACTION`

    此工作程序应用的最后一个事务的全局事务 ID（GTID）。

+   `LAST_APPLIED_TRANSACTION_ORIGINAL_COMMIT_TIMESTAMP`

    一个时间戳，格式为`'*`YYYY-MM-DD hh:mm:ss`*[.*`fraction`*]'`，显示此工作程序应用的最后一个事务在原始来源上提交的时间。

+   `LAST_APPLIED_TRANSACTION_IMMEDIATE_COMMIT_TIMESTAMP`

    一个时间戳，格式为`'*`YYYY-MM-DD hh:mm:ss`*[.*`fraction`*]'`，显示此工作程序应用的最后一个事务在即时来源上提交的时间。

+   `LAST_APPLIED_TRANSACTION_START_APPLY_TIMESTAMP`

    一个时间戳，格式为`'*`YYYY-MM-DD hh:mm:ss`*[.*`fraction`*]'`，显示此工作程序开始应用最后一个应用事务的时间。

+   `LAST_APPLIED_TRANSACTION_END_APPLY_TIMESTAMP`

    一个时间戳，格式为`'*`YYYY-MM-DD hh:mm:ss`*[.*`fraction`*]'`，显示此工作程序完成应用最后一个应用事务的时间。

+   `APPLYING_TRANSACTION`

    此工作程序当前应用的事务的全局事务 ID（GTID）。

+   `APPLYING_TRANSACTION_ORIGINAL_COMMIT_TIMESTAMP`

    一个时间戳，格式为`'*`YYYY-MM-DD hh:mm:ss`*[.*`fraction`*]'`，显示此工作程序当前应用的事务在原始来源上提交的时间。

+   `APPLYING_TRANSACTION_IMMEDIATE_COMMIT_TIMESTAMP`

    一个时间戳，格式为`'*`YYYY-MM-DD hh:mm:ss`*[.*`fraction`*]'`，显示此工作程序当前应用的事务在即时来源上提交的时间。

+   `APPLYING_TRANSACTION_START_APPLY_TIMESTAMP`

    一个时间戳，格式为`'*`YYYY-MM-DD hh:mm:ss`*[.*`fraction`*]'`，显示此工作程序开始尝试应用当前正在应用的事务的时间。在 MySQL 8.0.13 之前，由于瞬时错误导致事务重试时，此时间戳会刷新，因此显示了应用事务的最近尝试的时间戳。

+   `LAST_APPLIED_TRANSACTION_RETRIES_COUNT`

    最后一个应用事务在第一次尝试后由工作程序重试的次数。如果事务在第一次尝试时应用，则此数字为零。

+   `LAST_APPLIED_TRANSACTION_LAST_TRANSIENT_ERROR_NUMBER`

    最后一个瞬时错误的错误编号，导致事务需要重试。

+   `LAST_APPLIED_TRANSACTION_LAST_TRANSIENT_ERROR_MESSAGE`

    导致事务重试的最后一个瞬时错误的消息文本。

+   `LAST_APPLIED_TRANSACTION_LAST_TRANSIENT_ERROR_TIMESTAMP`

    以`'*`YYYY-MM-DD hh:mm:ss`*[.*`fraction`*]'`格式表示的最后一个导致事务重试的瞬时错误的时间戳。

+   `APPLYING_TRANSACTION_RETRIES_COUNT`

    当前正在应用的事务被重试的次数。如果事务第一次尝试时就被应用，这个数字为零。

+   `APPLYING_TRANSACTION_LAST_TRANSIENT_ERROR_NUMBER`

    导致当前事务重试的最后一个瞬时错误的错误编号。

+   `APPLYING_TRANSACTION_LAST_TRANSIENT_ERROR_MESSAGE`

    导致当前事务重试的最后一个瞬时错误的消息文本。

+   `APPLYING_TRANSACTION_LAST_TRANSIENT_ERROR_TIMESTAMP`

    以`'*`YYYY-MM-DD hh:mm:ss`*[.*`fraction`*]'`格式表示的最后一个导致当前事务重试的瞬时错误的时间戳。

`replication_applier_status_by_worker` 表具有以下索引：

+   在 (`CHANNEL_NAME`, `WORKER_ID`) 上的主键

+   在 (`THREAD_ID`) 上的索引

下表显示了`replication_applier_status_by_worker`列与`SHOW REPLICA STATUS`列之间的对应关系。

| `replication_applier_status_by_worker` 列 | `SHOW REPLICA STATUS` 列 |
| --- | --- |
| `WORKER_ID` | 无 |
| `THREAD_ID` | 无 |
| `SERVICE_STATE` | 无 |
| `LAST_ERROR_NUMBER` | `Last_SQL_Errno` |
| `LAST_ERROR_MESSAGE` | `Last_SQL_Error` |
| `LAST_ERROR_TIMESTAMP` | `Last_SQL_Error_Timestamp` |
