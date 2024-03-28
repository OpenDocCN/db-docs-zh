> 原文：[`dev.mysql.com/doc/refman/8.0/en/channels-commands-single-channel.html`](https://dev.mysql.com/doc/refman/8.0/en/channels-commands-single-channel.html)

#### 19.2.2.1 单通道操作命令

要使 MySQL 复制操作作用于单个复制通道，请在以下复制语句中使用`FOR CHANNEL *`channel`*`子句：

+   `CHANGE REPLICATION SOURCE TO`

+   `CHANGE MASTER TO`

+   `START REPLICA`（或在 MySQL 8.0.22 之前，`START SLAVE`）

+   `STOP REPLICA`（或在 MySQL 8.0.22 之前，`STOP SLAVE`）

+   `SHOW RELAYLOG EVENTS`

+   `FLUSH RELAY LOGS`

+   `SHOW REPLICA STATUS`（或在 MySQL 8.0.22 之前，`SHOW SLAVE STATUS`）

+   `RESET REPLICA`（或在 MySQL 8.0.22 之前，`RESET SLAVE`）

以下函数具有`channel`参数：

+   `MASTER_POS_WAIT()`

+   `SOURCE_POS_WAIT()`

以下语句不允许在`group_replication_recovery`通道上使用：

+   `START REPLICA`

+   `STOP REPLICA`

以下语句不允许在`group_replication_applier`通道上使用：

+   `START REPLICA`

+   `STOP REPLICA`

+   `SHOW REPLICA STATUS`

`FLUSH RELAY LOGS`现在允许在`group_replication_applier`通道上使用，但如果在应用事务时收到请求，则在事务结束后执行请求。请求者必须等待事务完成并进行旋转。此行为防止事务被拆分，这对于组复制是不允许的。
