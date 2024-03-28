> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-replication-asynchronous-connection-failover-table.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-replication-asynchronous-connection-failover-table.html)

#### 29.12.11.3 复制异步连接故障转移表

此表为异步连接故障转移机制的每个复制通道保存副本的源列表。异步连接故障转移机制在副本到源的连接失败后，自动从适当的列表中向新源建立异步（源到副本）复制连接。当为由 Group Replication 管理的一组副本启用异步连接故障转移时，当它们加入时，源列表会广播到所有组成员，并且当列表发生变化时也会广播。

您可以使用`asynchronous_connection_failover_add_source`和`asynchronous_connection_failover_delete_source`函数设置和管理源列表，以添加和删除复制源服务器到复制通道的源列表。要添加和删除受管理的服务器组，请改用`asynchronous_connection_failover_add_managed`和`asynchronous_connection_failover_delete_managed`函数。

有关更多信息，请参见第 19.4.9 节，“使用异步连接故障转移切换源和副本”。

`replication_asynchronous_connection_failover` 表具有以下列：

+   `通道名称`

    此复制源服务器所属源列表的复制通道。如果此通道与当前源的连接失败，则此复制源服务器是其潜在的新源之一。

+   `主机`

    此复制源服务器的主机名。

+   `端口`

    此复制源服务器的端口号。

+   `网络命名空间`

    复制源服务器的网络命名空间。如果此值为空，则连接使用默认（全局）命名空间。

+   `权重`

    在复制通道源列表中，此复制源服务器的优先级。权重范围为 1 到 100，100 为最高，50 为默认值。当异步连接故障转移机制激活时，通道源列表中备选源中设置最高权重的源将被选择用于第一次连接尝试。如果此尝试不起作用，则副本将按权重降序尝试所有列出的源，然后再从最高权重源开始。如果多个源具有相同的权重，则副本会随机排序它们。

+   `MANAGED_NAME`

    服务器所属的受管理组的标识符。对于`GroupReplication`受管理服务，该标识符是`group_replication_group_name`系统变量的值。

`replication_asynchronous_connection_failover` 表具有以下索引：

+   主键为 (`CHANNEL_NAME, HOST, PORT, NETWORK_NAMESPACE, MANAGED_NAME`)

`TRUNCATE TABLE` 不允许用于 `replication_asynchronous_connection_failover` 表。
