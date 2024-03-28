> 原文：[`dev.mysql.com/doc/refman/8.0/en/replication-gtids-assign-anon.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-gtids-assign-anon.html)

#### 19.1.3.6 从没有 GTIDs 的源到具有 GTIDs 的复制品的复制

从 MySQL 8.0.23 开始，您可以设置复制通道以为尚未具有 GTID 的复制事务分配 GTID。此功能使得可以从未启用 GTIDs 并且不使用基于 GTID 的复制的源服务器复制到启用了 GTIDs 的复制品。如果可以在复制源服务器上启用 GTIDs，如第 19.1.4 节“在线更改 GTID 模式”中所述，请使用该方法。此功能适用于无法启用 GTIDs 的复制源服务器。请注意，与 MySQL 复制的标准相同，此功能不支持从 MySQL 源服务器复制到先前发布系列之前的 MySQL 源服务器，因此 MySQL 5.7 是 MySQL 8.0 复制品的最早支持源。

您可以使用`CHANGE REPLICATION SOURCE TO`语句的`ASSIGN_GTIDS_TO_ANONYMOUS_TRANSACTIONS`选项在复制通道上启用 GTID 分配。`LOCAL`分配一个包括复制品自身 UUID（`server_uuid`设置）的 GTID。`*`uuid`*`分配一个包括指定 UUID 的 GTID，例如复制源服务器的`server_uuid`设置。使用非本地 UUID 可以区分在复制品上发起的事务和在源上发起的事务，以及在多源复制品上，区分在不同源上发起的事务。如果源发送的任何事务已经有 GTID，则保留该 GTID。

重要提示

在任何通道上设置了`ASSIGN_GTIDS_TO_ANONYMOUS_TRANSACTIONS`的复制品无法在需要故障转移时晋升为替换复制源服务器，并且无法使用从复制品备份的备份来恢复复制源服务器。替换或恢复其他使用任何通道上的`ASSIGN_GTIDS_TO_ANONYMOUS_TRANSACTIONS`的复制品也适用相同的限制。

复制品必须设置`gtid_mode=ON`，且此后不能更改，除非删除`ASSIGN_GTIDS_TO_ANONYMOUS_TRANSACTIONS=ON`设置。如果复制品服务器在未启用 GTIDs 的情况下启动，并为任何复制通道设置了`ASSIGN_GTIDS_TO_ANONYMOUS_TRANSACTIONS`，则设置不会更改，但会向错误日志写入警告消息，解释如何更改情况。

对于多源复制品，您可以混合使用使用`ASSIGN_GTIDS_TO_ANONYMOUS_TRANSACTIONS`和不使用的通道。专用于组复制的通道不能使用`ASSIGN_GTIDS_TO_ANONYMOUS_TRANSACTIONS`，但是在作为组复制组成员的服务器实例上的另一个源的异步复制通道可以这样做。对于组复制组成员上的通道，请不要将组复制组名称指定为创建 GTID 的 UUID。

在复制通道上使用`ASSIGN_GTIDS_TO_ANONYMOUS_TRANSACTIONS`并不等同于为通道引入基于 GTID 的复制。使用`ASSIGN_GTIDS_TO_ANONYMOUS_TRANSACTIONS`设置的复制品的 GTID 集合（`gtid_executed`）不应传输到另一台服务器或与另一台服务器的`gtid_executed`集合进行比较。分配给匿名事务的 GTID 以及您为其选择的 UUID 仅对该复制品自身的使用具有意义。唯一的例外是启用了`ASSIGN_GTIDS_TO_ANONYMOUS_TRANSACTIONS`的复制品的任何下游复制品，以及从该复制品的备份创建的任何服务器。

如果您设置了任何下游复制品，这些服务器不会启用`ASSIGN_GTIDS_TO_ANONYMOUS_TRANSACTIONS`。只有直接从非 GTID 源服务器接收事务的复制品需要在相关复制通道上设置`ASSIGN_GTIDS_TO_ANONYMOUS_TRANSACTIONS`。在该复制品及其下游复制品之间，您可以比较 GTID 集合，从一个复制品故障转移至另一个复制品，并使用备份创建额外的复制品，就像在任何基于 GTID 的复制拓扑中一样。`ASSIGN_GTIDS_TO_ANONYMOUS_TRANSACTIONS`用于从此组外部的非 GTID 服务器接收事务的情况。

使用`ASSIGN_GTIDS_TO_ANONYMOUS_TRANSACTIONS`的复制通道与基于 GTID 的复制有以下行为差异：

+   当应用复制的事务时，将为其分配 GTID（除非它们已经有 GTID）。通常，当事务提交时，GTID 会在复制源服务器上分配，并与事务一起发送到复制品。在多线程复制品上，这意味着 GTID 的顺序不一定与事务的顺序匹配，即使设置了`slave-preserve-commit-order=1`。

+   `CHANGE REPLICATION SOURCE TO`语句的`SOURCE_LOG_FILE`和`SOURCE_LOG_POS`选项用于定位复制 I/O（接收器）线程，而不是`SOURCE_AUTO_POSITION`选项。

+   使用`SET GLOBAL sql_replica_skip_counter`或`SET GLOBAL sql_slave_skip_counter`语句跳过使用`ASSIGN_GTIDS_TO_ANONYMOUS_TRANSACTIONS`设置的复制通道上的事务，而不是提交空事务的方法。有关说明，请参见 Section 19.1.7.3, “Skipping Transactions”。

+   `START REPLICA`语句的`UNTIL SQL_BEFORE_GTIDS`和`UNTIL_SQL_AFTER_GTIDS`选项不能用于通道。

+   函数`WAIT_UNTIL_SQL_THREAD_AFTER_GTIDS()`在 MySQL 8.0.18 中已被弃用，不能与通道一起使用。它的替代品`WAIT_FOR_EXECUTED_GTID_SET()`可以在整个服务器上工作，可用于等待启用了`ASSIGN_GTIDS_TO_ANONYMOUS_TRANSACTIONS`的服务器的任何下游复制品。要等待启用了`ASSIGN_GTIDS_TO_ANONYMOUS_TRANSACTIONS`的通道赶上不使用 GTIDs 的源，请使用`SOURCE_POS_WAIT()`函数（从 MySQL 8.0.26 开始）或`MASTER_POS_WAIT()`函数。

Performance Schema `replication_applier_configuration` 表显示复制通道上是否为匿名事务分配了 GTIDs，UUID 是什么，以及它是副本服务器的 UUID（`LOCAL`）还是用户指定的 UUID（`UUID`）。这些信息也记录在 applier 元数据存储库中。`RESET REPLICA ALL`语句会重置`ASSIGN_GTIDS_TO_ANONYMOUS_TRANSACTIONS`设置，但`RESET REPLICA`语句不会。
