# 19.4.9 利用异步连接故障转移切换源和副本

> 原文：[`dev.mysql.com/doc/refman/8.0/en/replication-asynchronous-connection-failover.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-asynchronous-connection-failover.html)

19.4.9.1 源的异步连接故障转移

19.4.9.2 副本的异步连接故障转移

从 MySQL 8.0.22 开始，您可以使用异步连接故障转移机制在现有副本到源的连接失败后自动建立与新源的异步（源到副本）复制连接。异步连接故障转移机制可用于使副本与共享数据的多个 MySQL 服务器或服务器组保持同步。潜在源服务器列表存储在副本上，在连接失败时，根据您设置的加权优先级从列表中选择新源。

从 MySQL 8.0.23 开始，异步连接故障转移机制还支持组复制拓扑结构，通过自动监视组成员变化并区分主服务器和次要服务器。当您将组成员添加到源列表并将其定义为受控组的一部分时，异步连接故障转移机制会更新源列表以使其与成员变化保持一致，随着成员加入或离开自动添加和删除组成员。仅使用在线占多数的组成员进行连接和获取状态。即使最后剩下的受控组成员离开组，也不会自动删除，以保持受控组的配置。但是，如果不再需要，可以手动删除受控组。

从 MySQL 8.0.27 开始，异步连接故障转移机制还可以使作为受控复制组一部分的副本在当前接收者（组的主节点）失败时自动重新连接到发送者。此功能与组复制一起使用，在配置为单主模式的组上工作，其中组的主节点是使用该机制的副本的复制通道。该功能旨在使一组发送者和一组接收者即使某些成员暂时不可用也保持同步。它还将一组接收者与一个或多个不属于受控组的发送者同步。不属于复制组的副本无法使用此功能。

使用异步连接故障转移机制的要求如下：

+   GTIDs 必须在源端和副本端使用 (`gtid_mode=ON`)，并且必须在副本端启用 `CHANGE REPLICATION SOURCE TO` | `CHANGE MASTER TO` 语句的 `SOURCE_AUTO_POSITION` | `MASTER_AUTO_POSITION` 选项，以便使用 GTID 自动定位连接到源端。

+   在通道的源列表中，所有源服务器上必须存在相同的复制用户帐户和密码。此帐户用于连接到每个源端。您可以为不同的通道设置不同的帐户。

+   复制用户帐户必须在 Performance Schema 表上具有 `SELECT` 权限，例如，通过发出 `GRANT SELECT ON performance_schema.* TO '*`repl_user`*';`。

+   由于需要在备用源端的自动重启中使用，因此不能在用于启动复制的语句中指定复制用户帐户和密码。必须在副本端使用 `CHANGE REPLICATION SOURCE TO` | `CHANGE MASTER TO` 语句为通道设置这些信息，并记录在复制元数据存储库中。

+   如果异步连接故障转移机制正在使用的通道位于 Group Replication 单主模式组的主服务器上，在 MySQL 8.0.27 中，默认情况下也会激活副本之间的异步连接故障转移。在这种情况下，必须在复制组中的所有次要服务器以及任何新加入的成员上设置复制通道和复制用户帐户和密码。如果使用 MySQL 的克隆功能提供新服务器，则所有这些都会自动发生。

    重要

    如果您不希望在这种情况下副本之间发生异步连接故障转移，请通过使用 `group_replication_disable_member_action` 函数为组禁用成员操作 `mysql_start_failover_channels_if_primary` 来禁用它。当禁用该功能时，您无需在次要组成员上配置复制通道，但如果主服务器下线或进入错误状态，则通道的复制将停止。

从 MySQL Shell 8.0.27 和 MySQL 8.0.27 开始，MySQL InnoDB ClusterSet 可用于为 InnoDB Cluster 部署提供灾难容忍性，方法是将主 InnoDB Cluster 与其在不同位置（如不同数据中心）的一个或多个副本链接起来。考虑使用这种解决方案来简化新的多组部署的设置，用于复制、故障切换和灾难恢复。您可以将现有的 Group Replication 部署作为 InnoDB Cluster 采用。

InnoDB ClusterSet 和 InnoDB Cluster 旨在抽象和简化设置、管理、监控、恢复和修复复制组的过程。InnoDB ClusterSet 自动管理从主集群到副本集群的复制，使用专用的 ClusterSet 复制通道。您可以使用管理员命令在主集群不正常运行时触发受控切换或紧急故障转移。在需求发生变化时，可以在初始设置后轻松地向 InnoDB ClusterSet 部署添加或删除服务器和组。有关更多信息，请参阅 MySQL InnoDB ClusterSet。
