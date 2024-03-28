> 原文：[`dev.mysql.com/doc/refman/8.0/en/channels-with-prev-replication.html`](https://dev.mysql.com/doc/refman/8.0/en/channels-with-prev-replication.html)

#### 19.2.2.2 与先前复制语句的兼容性

当一个复制品有多个通道且未指定 `FOR CHANNEL *`channel`*` 选项时，一个有效的语句通常会作用于所有可用通道，但也有一些特定的例外情况。

例如，以下语句对所有通道都能正常工作，除了某些 Group Replication 通道：

+   `START REPLICA` 启动所有通道的复制线程，除了 `group_replication_recovery` 和 `group_replication_applier` 通道。

+   `STOP REPLICA` 停止所有通道的复制线程，除了 `group_replication_recovery` 和 `group_replication_applier` 通道。

+   `SHOW REPLICA STATUS` 报告所有通道的状态，除了 `group_replication_applier` 通道。

+   `RESET REPLICA` 重置所有通道。

警告

谨慎使用 `RESET REPLICA`，因为此语句会删除所有现有通道，清除它们的中继日志文件，并仅重新创建默认通道。

一些复制语句无法在所有通道上操作。在这种情况下，会生成错误 1964 复制品上存在多个通道。请提供通道名称作为参数。以下语句和函数在多源复制拓扑中使用时会生成此错误，并且未使用 `FOR CHANNEL *`channel`*` 选项来指定要操作的通道：

+   `SHOW RELAYLOG EVENTS`

+   `CHANGE REPLICATION SOURCE TO`

+   `CHANGE MASTER TO`

+   `MASTER_POS_WAIT()`

+   `SOURCE_POS_WAIT()`

请注意，在单源复制拓扑中始终存在一个默认通道，其中语句和函数的行为与 MySQL 先前版本中的行为相同。
