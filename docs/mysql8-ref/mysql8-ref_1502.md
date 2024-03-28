# 20.4.2 Group Replication Server States

> 原文：[`dev.mysql.com/doc/refman/8.0/en/group-replication-server-states.html`](https://dev.mysql.com/doc/refman/8.0/en/group-replication-server-states.html)

Group Replication 群组成员的状态显示其在群组中的当前角色。Performance Schema 表`replication_group_members`显示了群组中每个成员的状态。如果群组完全正常运行且所有成员正常通信，则所有成员对所有其他成员报告相同的状态。但是，已离开群组或处于网络分区的成员无法准确报告其他服务器的信息。在这种情况下，该成员不会尝试猜测其他服务器的状态，而是将它们报告为不可访问。

一个群组成员可以处于以下状态：

`ONLINE`

服务器是群组的活跃成员，并处于完全运行状态。其他群组成员可以连接到它，客户端（如果适用）也可以连接到它。只有当成员处于`ONLINE`状态时，它才会与群组完全同步，并参与其中。

`RECOVERING`

服务器已加入一个群组，并正在成为活跃成员的过程中。当前正在进行分布式恢复，成员正在接收来自捐赠者的状态传输，使用远程克隆操作或捐赠者的二进制日志。此状态为

更多信息，请参见 Section 20.5.4, “Distributed Recovery”。

`OFFLINE`

Group Replication 插件已加载，但成员不属于任何群组。在成员加入或重新加入群组时，可能会暂时出现此状态。

`ERROR`

该成员处于错误状态，并且作为群组成员无法正常运行。成员可能在应用事务期间或恢复阶段进入错误状态。处于此状态的成员不参与群组的事务。有关错误状态可能原因的更多信息，请参见 Section 20.7.7, “Responses to Failure Detection and Network Partitioning”。

根据`group_replication_exit_state_action`设置的退出操作，成员处于只读模式（`super_read_only=ON`），也可能处于离线模式（`offline_mode=ON`）。请注意，遵循`OFFLINE_MODE`退出操作的离线模式服务器显示为`ERROR`状态，而不是`OFFLINE`。采用`ABORT_SERVER`退出操作的服务器将关闭并从组的视图中移除。更多信息，请参见 Section 20.7.7.4, “退出操作”。

当成员加入或重新加入复制组时，在组完成兼容性检查并接受其为成员之前，其状态可能显示为`ERROR`。

`UNREACHABLE`

本地故障检测器怀疑无法联系到成员，因为组的消息超时。这可能发生在成员非自愿断开连接的情况下。如果您在其他服务器看到此状态，也可能意味着您查询此表的成员是分区的一部分，组的一部分服务器可以相互联系，但无法联系组中的其他服务器。更多信息，请参见 Section 20.7.8, “处理网络分区和失去法定人数”。

请参见 Section 20.4.3, “复制组成员表”，了解性能模式表内容的示例。
