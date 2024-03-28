> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-replication-group-communication-information-table.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-replication-group-communication-information-table.html)

#### 29.12.11.15 replication_group_communication_information 表

此表显示整个复制组的组配置选项。仅在安装了 Group Replication 时才可用。

`replication_group_communication_information`表具有以下列：

+   `WRITE_CONCURRENCY`

    组可以并行执行的最大共识实例数。默认值为 10。请参阅 20.5.1.3 节，“使用 Group Replication 组写共识”。

+   `PROTOCOL_VERSION`

    Group Replication 通信协议版本，确定使用的消息传递能力。这是为了适应您希望组支持的最旧 MySQL Server 版本而设置的。请参阅 20.5.1.4 节，“设置组的通信协议版本”。

+   `WRITE_CONSENSUS_LEADERS_PREFERRED`

    Group Replication 已指示组通信引擎使用的领导者或领导者来驱动共识。对于使用`group_replication_paxos_single_leader`系统变量设置为`ON`且通信协议版本设置为 8.0.27 或更高版本的单主模式组，单一共识领导者是组的主。否则，所有组成员都被用作领导者，因此它们都在这里显示。请参阅 20.7.3 节，“单一共识领导者”。

+   `WRITE_CONSENSUS_LEADERS_ACTUAL`

    组通信引擎正在使用的实际领导者或领导者来驱动共识。如果组使用单一共识领导者，并且主目前不健康，组通信会选择替代共识领导者。在这种情况下，此处指定的组成员可能与首选组成员不同。

+   `WRITE_CONSENSUS_SINGLE_LEADER_CAPABLE`

    复制组是否能够使用单一一致性领导者。1 表示该组是在启用单一领导者的情况下启动的（`group_replication_paxos_single_leader = ON`），即使此后在该组成员上更改了 `group_replication_paxos_single_leader` 的值，仍然显示为 1。0 表示该组是在禁用单一领导者模式（`group_replication_paxos_single_leader = OFF`）启动的，或者具有不支持使用单一一致性领导者的 Group Replication 通信协议版本（低于 8.0.27）。此信息仅对处于 `ONLINE` 或 `RECOVERING` 状态的组成员返回。

`replication_group_communication_information` 表没有索引。

不允许对 `replication_group_communication_information` 表使用 `TRUNCATE TABLE`。
