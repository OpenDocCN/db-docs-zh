> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-replication-asynchronous-connection-failover-managed-table.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-replication-asynchronous-connection-failover-managed-table.html)

#### 29.12.11.4 复制异步连接故障转移管理表

此表保存由副本的异步连接故障转移机制使用的配置信息，用于处理受管理组，包括 Group Replication 拓扑。

当您将组成员添加到源列表并将其定义为受管理组的一部分时，异步连接故障转移机制会更新源列表，以使其与成员变化保持一致，自动添加和删除组成员，随着它们加入或离开。当为由 Group Replication 管理的一组副本启用异步连接故障转移时，当它们加入时，源列表会广播到所有组成员，并且当列表发生变化时也会广播。

异步连接故障转移机制会在源列表中另一个可用服务器具有更高优先级（权重）设置时切换连接。对于受管理的组，源的权重根据其是主服务器还是从服务器而分配。因此，假设您设置了受管理组以给主服务器分配更高的权重，给从服务器分配较低的权重，当主服务器更改时，新主服务器被分配更高的权重，因此副本切换到新主服务器的连接。异步连接故障转移机制还会在当前连接的受管理源服务器离开受管理组或不再是受管理组中的大多数时更改连接。有关更多信息，请参见第 19.4.9 节，“使用异步连接故障转移切换源和副本”。

`replication_asynchronous_connection_failover_managed` 表包含以下列：

+   `CHANNEL_NAME`

    用于操作此受管理组的复制通道。

+   `MANAGED_NAME`

    受管理组的标识符。对于`GroupReplication`受管理服务，标识符是`group_replication_group_name`系统变量的值。

+   `MANAGED_TYPE`

    异步连接故障转移机制为该组提供的受管理服务类型。目前唯一可用的值是`GroupReplication`。

+   `CONFIGURATION`

    此托管组的配置信息。对于`GroupReplication`托管服务，配置显示分配给组的主服务器和组的辅助服务器的权重。例如：`{"Primary_weight": 80, "Secondary_weight": 60}`

    +   `Primary_weight`: 介于 0 和 100 之间的整数。默认值为 80。

    +   `Secondary_weight`: 介于 0 和 100 之间的整数。默认值为 60。

`replication_asynchronous_connection_failover_managed` 表具有以下索引：

+   主键为(`CHANNEL_NAME, MANAGED_NAME`)。

`TRUNCATE TABLE` 不允许用于`replication_asynchronous_connection_failover_managed` 表。
