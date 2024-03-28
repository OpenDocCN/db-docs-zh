> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-replication-applier-status-table.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-replication-applier-status-table.html)

#### 29.12.11.6 复制 _applier_status 表

该表显示副本上当前的一般事务执行状态。该表提供有关事务应用程序状态的一般方面的信息，这些信息与涉及的任何线程无关。线程特定的状态信息可在`replication_applier_status_by_coordinator`表中找到（如果副本是多线程的，则在`replication_applier_status_by_worker`中也有）。

`replication_applier_status`表具有以下列：

+   `CHANNEL_NAME`

    显示此行所显示的复制通道。始终存在默认复制通道，并且可以添加更多复制通道。有关更多信息，请参见 Section 19.2.2，“复制通道”。

+   `SERVICE_STATE`

    当复制通道的应用程序线程处于活动或空闲状态时显示`ON`，`OFF`表示应用程序线程不活动。

+   `REMAINING_DELAY`

    如果副本正在等待`DESIRED_DELAY`秒，以便自源应用事务以来经过，此字段包含剩余延迟秒数。在其他时间，此字段为`NULL`。（`DESIRED_DELAY`值存储在`replication_applier_configuration`表中。）有关更多信息，请参见 Section 19.4.11，“延迟复制”。

+   `COUNT_TRANSACTIONS_RETRIES`

    显示由于复制 SQL 线程未能应用事务而进行的重试次数。给定事务的最大重试次数由系统变量`replica_transaction_retries`和`slave_transaction_retries`设置。`replication_applier_status_by_worker`表显示了单线程或多线程副本的事务重试的详细信息。

`replication_applier_status`表具有以下索引：

+   (`CHANNEL_NAME`)上的主键

`TRUNCATE TABLE` 不允许用于`replication_applier_status`表。

以下表格显示了`replication_applier_status`列与`SHOW REPLICA STATUS`列之间的对应关系。

| `replication_applier_status` 列 | `SHOW REPLICA STATUS` 列 |
| --- | --- |
| `SERVICE_STATE` | 无 |
| `REMAINING_DELAY` | `SQL_Remaining_Delay` |
