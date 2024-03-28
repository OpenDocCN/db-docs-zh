# 20.7.3 单一共识领导者

> 原文：[`dev.mysql.com/doc/refman/8.0/en/group-replication-single-consensus-leader.html`](https://dev.mysql.com/doc/refman/8.0/en/group-replication-single-consensus-leader.html)

默认情况下，Group Replication 的组通信引擎（XCom，一种 Paxos 变体）使用复制组的每个成员作为领导者。从 MySQL 8.0.27 开始，当组处于单主模式时，组通信引擎可以使用单个领导者来驱动共识。在单主模式下使用单一共识领导者可以提高性能和韧性，特别是当组的某些次要成员当前无法访问时。

要使用单一共识领导者，组必须按以下方式配置：

+   组必须处于单主模式。

+   `group_replication_paxos_single_leader` 系统变量必须设置为 `ON`。默认设置为 `OFF` 时，该行为被禁用。您必须对复制组进行完全重启（引导）以使 Group Replication 生效对此设置的更改。

+   Group Replication 通信协议版本必须设置为 8.0.27 或更高。使用 `group_replication_get_communication_protocol()` 函数查看组的通信协议版本。如果使用较低版本，则组无法使用此行为。如果所有组成员都支持，您可以使用 `group_replication_set_communication_protocol()` 函数将组的通信协议设置为更高版本。MySQL InnoDB Cluster 会自动管理通信协议版本。有关更多信息，请参见 Section 20.5.1.4, “Setting a Group's Communication Protocol Version”。

当配置完成后，Group Replication 指示组通信引擎使用组的主节点作为单一领导者来驱动共识。当选举出新的主节点时，Group Replication 会告知组通信引擎使用新的主节点。如果主节点当前不健康，组通信引擎会使用其他成员作为共识领导者。性能模式表 `replication_group_communication_information` 显示当前首选和实际共识领导者，首选领导者是 Group Replication 的选择，实际领导者是组通信引擎选择的领导者。

如果组处于多主模式，具有较低的通信协议版本，或者行为被`group_replication_paxos_single_leader`设置禁用，则所有成员都被用作领导者来推动共识。在这种情况下，性能模式表`replication_group_communication_information`显示所有成员都是首选和实际领导者。

在性能模式表`replication_group_communication_information`中的字段`WRITE_CONSENSUS_SINGLE_LEADER_CAPABLE`显示组是否支持使用单个领导者，即使在查询的成员上当前设置为`OFF`的`group_replication_paxos_single_leader`。如果组是在启动时使用`group_replication_paxos_single_leader`设置为`ON`，并且其通信协议版本为 MySQL 8.0.27 或更高版本，则该字段设置为 1。此信息仅对处于`ONLINE`或`RECOVERING`状态的组成员返回。
