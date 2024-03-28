> 原文：[`dev.mysql.com/doc/refman/8.0/en/replication-asynchronous-connection-failover-replica.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-asynchronous-connection-failover-replica.html)

#### 19.4.9.2 副本的异步连接故障转移

在 MySQL 8.0.27 及更高版本中，当你在复制通道上设置`SOURCE_CONNECTION_AUTO_FAILOVER=1`时，副本的异步连接故障转移将自动激活于 Group Replication 主服务器上。该功能旨在设计一个发送者组和一个接收者组，即使一些成员暂时不可用，它们也能保持同步。当该功能处于活动状态并正确配置时，如果正在复制的主服务器离线或进入错误状态，新的主服务器在选举时将在同一通道上开始复制。新的主服务器使用通道的源列表来选择具有最高优先级（权重）设置的源，这可能与原始源不同。

要配置此功能，必须在复制组中的所有成员服务器以及任何新加入的成员上设置复制通道和通道的复制用户帐户和密码。确保`SOURCE_RETRY_COUNT`和`SOURCE_CONNECT_RETRY`设置为允许少量重试尝试的最小数字，例如 3 和 10。您可以使用`CHANGE REPLICATION SOURCE TO`设置复制通道，或者如果新服务器是使用 MySQL 的克隆功能进行配置的，则所有这些都会自动发生。当新成员加入时，主服务器会向组成员广播通道的`SOURCE_CONNECTION_AUTO_FAILOVER`设置。如果您稍后在主服务器上禁用通道的`SOURCE_CONNECTION_AUTO_FAILOVER`，这也会广播到辅助服务器，并且它们会更改通道的状态以匹配。

注意

在单主模式下参与组的服务器必须使用`--skip-replica-start=ON`启动。否则，服务器无法作为辅助服务器加入组。

副本的异步连接故障转移是通过 Group Replication 成员操作`mysql_start_failover_channels_if_primary`来激活和停用的，默认情况下启用。您可以通过在主服务器上禁用该成员操作来为整个组禁用它，使用`group_replication_disable_member_action`函数，如下例所示：

```sql
mysql> SELECT group_replication_disable_member_action("mysql_start_failover_channels_if_primary", "AFTER_PRIMARY_ELECTION");
```

该功能只能在主服务器上更改，并且必须为整个组启用或禁用，因此不能让一些成员提供故障转移，而其他成员不提供。当禁用`mysql_start_failover_channels_if_primary`成员操作时，次要成员不需要配置频道，但如果主服务器下线或进入错误状态，则该频道的复制将停止。请注意，如果有多个具有`SOURCE_CONNECTION_AUTO_FAILOVER=1`的频道，则成员操作将覆盖所有频道，因此它们不能通过该方法单独启用或禁用。在主服务器上将`SOURCE_CONNECTION_AUTO_FAILOVER=0`设置为禁用单个频道。

当频道具有`SOURCE_CONNECTION_AUTO_FAILOVER=1`时，源列表会在所有组成员加入时广播，并在更改时也会广播。无论这些源是否是一个受管理的组，其成员资格是否会自动更新，或者是使用`asynchronous_connection_failover_add_source()`，`asynchronous_connection_failover_delete_source()`，`asynchronous_connection_failover_add_managed()`，或`asynchronous_connection_failover_delete_managed()`手动添加或更改。所有组成员都会收到当前源列表，记录在`mysql.replication_asynchronous_connection_failover`和`mysql.replication_asynchronous_connection_failover_managed`表中。因为源不必在受管理的组中，所以您可以设置该功能以将一组接收器与一个或多个备用独立发件人同步，甚至是单个发件人。不属于复制组的独立副本无法使用此功能。
