> 原文：[`dev.mysql.com/doc/refman/8.0/en/group-replication-tuning-recovery.html`](https://dev.mysql.com/doc/refman/8.0/en/group-replication-tuning-recovery.html)

#### 20.5.4.3 配置分布式恢复

Group Replication 的分布式恢复过程的几个方面可以配置以适应您的系统。

##### 连接尝试次数

对于从二进制日志进行状态传输，Group Replication 限制了加入成员在尝试连接到供体池中的供体时所做的尝试次数。如果达到连接重试限制而没有成功连接，则分布式恢复过程将以错误终止。请注意，此限制指定了加入成员尝试连接到供体的总次数。例如，如果有 2 个组成员是适当的供体，并且连接重试限制设置为 4，加入成员在达到限制之前对每个供体进行 2 次连接尝试。

默认连接重试限制为 10。您可以使用`group_replication_recovery_retry_count`系统变量来配置此设置。以下命令将最大连接尝试次数设置为 5：

```sql
mysql> SET GLOBAL group_replication_recovery_retry_count= 5;
```

对于远程克隆操作，不适用此限制。Group Replication 仅尝试与每个适当的供体进行一次连接，然后开始尝试从二进制日志进行状态传输。

##### 连接尝试的休眠间隔

对于从二进制日志进行状态传输，`group_replication_recovery_reconnect_interval`系统变量定义了分布式恢复过程在尝试连接到供体之间应该休眠多长时间。请注意，分布式恢复在每次供体连接尝试后不会休眠。由于加入成员正在连接到不同的服务器而不是重复连接到同一个服务器，它可以假设影响服务器 A 的问题不会影响服务器 B。因此，分布式恢复仅在经过所有可能的供体后暂停。一旦加入组的服务器对组中的每个适当的供体都尝试连接了一次，分布式恢复过程将休眠，时间由`group_replication_recovery_reconnect_interval`系统变量配置。例如，如果有 2 个组成员是适当的供体，并且连接重试限制设置为 4，加入成员对每个供体进行一次连接尝试，然后休眠连接重试间隔，然后对每个供体进行一次进一步的连接尝试，直到达到限制。

默认连接重试间隔为 60 秒，您可以动态更改此值。以下命令将分布式恢复捐赠者连接重试间隔设置为 120 秒：

```sql
mysql> SET GLOBAL group_replication_recovery_reconnect_interval= 120;
```

对于远程克隆操作，此间隔不适用。在开始尝试从二进制日志进行状态转移之前，Group Replication 仅对每个适当的捐赠者进行一次连接尝试。

##### 在线标记加入成员

当分布式恢复成功完成从捐赠者到加入成员的状态转移后，加入成员可以在组中标记为在线并准备参与。默认情况下，在加入成员接收并应用了其缺失的所有事务后才会执行此操作。可选地，您可以允许在加入成员接收并认证（即完成冲突检测）其缺失的所有事务后，但在应用它们之前将其标记为在线。如果您想这样做，请使用`group_replication_recovery_complete_at`系统变量来指定替代设置`TRANSACTIONS_CERTIFIED`。
