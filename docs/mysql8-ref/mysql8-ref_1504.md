# 20.4.4 The replication_group_member_stats Table

> 译文：[`dev.mysql.com/doc/refman/8.0/en/group-replication-replication-group-member-stats.html`](https://dev.mysql.com/doc/refman/8.0/en/group-replication-replication-group-member-stats.html)

每个复制组中的成员都会对组接收的事务进行认证和应用。关于认证者和应用者过程的统计信息对于了解应用程序队列的增长情况、发现了多少冲突、检查了多少事务、哪些事务在所有地方都已提交等方面非常有用。

`performance_schema.replication_group_member_stats` 表提供了与认证过程相关的组级信息，以及每个复制组成员接收和发起的事务的统计信息。这些信息在所有作为复制组成员的服务器实例之间共享，因此可以从任何成员查询所有组成员的信息。请注意，远程成员的统计信息刷新由`group_replication_flow_control_period` 选项中指定的消息周期控制，因此这些统计信息可能与在进行查询的成员本地收集的统计信息略有不同。要使用此表监视 Group Replication 成员，请执行以下语句：

```sql
mysql> SELECT * FROM performance_schema.replication_group_member_stats\G
```

从 MySQL 8.0.19 开始，您还可以使用以下语句：

```sql
mysql> TABLE performance_schema.replication_group_member_stats\G
```

这些列对于监视组中连接的成员的性能非常重要。假设组中的一个成员总是报告其队列中的事务数量比其他成员多。这意味着该成员延迟了，并且无法跟上组中其他成员的步伐。根据这些信息，您可以决定是将该成员从组中移除，还是延迟其他组成员的事务处理，以减少排队事务的数量。这些信息还可以帮助您决定如何调整 Group Replication 插件的流量控制，请参阅 Section 20.7.2, “Flow Control”。
