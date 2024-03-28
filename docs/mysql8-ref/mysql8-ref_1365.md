> 译文：[`dev.mysql.com/doc/refman/8.0/en/replication-multi-source-monitoring.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-multi-source-monitoring.html)

#### 19.1.5.8 监控多源复制

要监视复制通道的状态，存在以下选项：

+   使用复制性能模式表。这些表的第一列是`Channel_Name`。这使您可以基于`Channel_Name`编写复杂查询。请参见第 29.12.11 节，“性能模式复制表”。

+   使用`SHOW REPLICA STATUS FOR CHANNEL *`channel`*`。默认情况下，如果未使用`FOR CHANNEL *`channel`*`子句，则此语句将显示所有通道的复制状态，每个通道一行。标识符`Channel_name`将作为结果集中的一列添加。如果提供了`FOR CHANNEL *`channel`*`子句，则结果仅显示命名复制通道的状态。

注意

`SHOW VARIABLES`语句不适用于多个复制通道。通过这些变量可用的信息已迁移到复制性能表。在具有多个通道的拓扑中使用`SHOW VARIABLES`语句仅显示默认通道的状态。

启用多源复制时发出的错误代码和消息指定生成错误的通道。

##### 19.1.5.8.1 使用性能模式表监视通道

本节解释了如何使用复制性能模式表监视通道。您可以选择监视所有通道或现有通道的子集。

要监视所有通道的连接状态：

```sql
mysql> SELECT * FROM replication_connection_status\G;
*************************** 1\. row ***************************
CHANNEL_NAME: source_1
GROUP_NAME:
SOURCE_UUID: 046e41f8-a223-11e4-a975-0811960cc264
THREAD_ID: 24
SERVICE_STATE: ON
COUNT_RECEIVED_HEARTBEATS: 0
LAST_HEARTBEAT_TIMESTAMP: 0000-00-00 00:00:00
RECEIVED_TRANSACTION_SET: 046e41f8-a223-11e4-a975-0811960cc264:4-37
LAST_ERROR_NUMBER: 0
LAST_ERROR_MESSAGE:
LAST_ERROR_TIMESTAMP: 0000-00-00 00:00:00
*************************** 2\. row ***************************
CHANNEL_NAME: source_2
GROUP_NAME:
SOURCE_UUID: 7475e474-a223-11e4-a978-0811960cc264
THREAD_ID: 26
SERVICE_STATE: ON
COUNT_RECEIVED_HEARTBEATS: 0
LAST_HEARTBEAT_TIMESTAMP: 0000-00-00 00:00:00
RECEIVED_TRANSACTION_SET: 7475e474-a223-11e4-a978-0811960cc264:4-6
LAST_ERROR_NUMBER: 0
LAST_ERROR_MESSAGE:
LAST_ERROR_TIMESTAMP: 0000-00-00 00:00:00 2 rows in set (0.00 sec)
```

在上述输出中，有两个启用的通道，如`CHANNEL_NAME`字段所示，它们分别称为`source_1`和`source_2`。

添加`CHANNEL_NAME`字段使您能够查询特定通道的性能模式表。要监视命名通道的连接状态，请使用`WHERE CHANNEL_NAME=*`channel`*`子句：

```sql
mysql> SELECT * FROM replication_connection_status WHERE CHANNEL_NAME='source_1'\G
*************************** 1\. row ***************************
CHANNEL_NAME: source_1
GROUP_NAME:
SOURCE_UUID: 046e41f8-a223-11e4-a975-0811960cc264
THREAD_ID: 24
SERVICE_STATE: ON
COUNT_RECEIVED_HEARTBEATS: 0
LAST_HEARTBEAT_TIMESTAMP: 0000-00-00 00:00:00
RECEIVED_TRANSACTION_SET: 046e41f8-a223-11e4-a975-0811960cc264:4-37
LAST_ERROR_NUMBER: 0
LAST_ERROR_MESSAGE:
LAST_ERROR_TIMESTAMP: 0000-00-00 00:00:00 1 row in set (0.00 sec)
```

同样，`WHERE CHANNEL_NAME=*`channel`*`子句可用于监视其他复制性能模式表以获取特定通道的信息。有关更多信息，请参见第 29.12.11 节，“性能模式复制表”。
