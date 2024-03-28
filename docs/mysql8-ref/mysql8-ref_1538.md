# 20.7.5 消息分段

> 原文：[`dev.mysql.com/doc/refman/8.0/en/group-replication-performance-message-fragmentation.html`](https://dev.mysql.com/doc/refman/8.0/en/group-replication-performance-message-fragmentation.html)

当在 Group Replication 组成员之间发送异常大的消息时，可能导致一些组成员被报告为失败并从组中驱逐。这是因为 Group Replication 的组通信引擎（XCom，一种 Paxos 变体）使用的单个线程在处理消息时占用时间过长，因此一些组成员可能会将接收者报告为失败。从 MySQL 8.0.16 开始，默认情况下，大消息会自动分割成单独发送并由接收方重新组装的片段。

系统变量`group_replication_communication_max_message_size`指定了 Group Replication 通信的最大消息大小，超过该大小的消息将被分段。默认的最大消息大小为 10485760 字节（10 MiB）。允许的最大值与`replica_max_allowed_packet`和`slave_max_allowed_packet`系统变量的最大值相同，即 1073741824 字节（1 GB）。`group_replication_communication_max_message_size`的设置必须小于`replica_max_allowed_packet`或`slave_max_allowed_packet`的设置，因为应用程序线程无法处理大于最大允许数据包大小的消息片段。要关闭分段，为`group_replication_communication_max_message_size`指定零值。

与大多数其他 Group Replication 系统变量一样，必须重新启动 Group Replication 插件才能使更改生效。例如：

```sql
STOP GROUP_REPLICATION;
SET GLOBAL group_replication_communication_max_message_size= 5242880;
START GROUP_REPLICATION;
```

分段消息的消息传递在所有组成员接收并重新组装消息的所有片段后被视为完成。分段消息在其标头中包含信息，使得在消息传输过程中加入的成员能够恢复其加入之前发送的早期片段。如果加入的成员未能恢复片段，则会自行从组中退出。

为了使复制组使用分片，所有组成员必须在 MySQL 8.0.16 或更高版本，并且组使用的 Group Replication 通信协议版本必须允许分片。您可以使用`group_replication_get_communication_protocol()`函数检查组使用的通信协议，该函数返回组支持的最旧的 MySQL Server 版本。从 MySQL 5.7.14 版本开始允许消息压缩，从 MySQL 8.0.16 版本开始还允许消息分片。如果所有组成员都在 MySQL 8.0.16 或更高版本，并且没有要求允许早期版本的成员加入，您可以使用`group_replication_set_communication_protocol()`函数将通信协议版本设置为 MySQL 8.0.16 或更高版本，以允许分片。有关更多信息，请参见第 20.5.1.4 节，“设置组的通信协议版本”。

如果一个复制组由于某些成员不支持而无法使用分片，系统变量`group_replication_transaction_size_limit`可以用来限制组接受的事务的最大大小。在 MySQL 8.0 中，默认设置约为 143 MB。超过此大小的事务将被回滚。您还可以使用系统变量`group_replication_member_expel_timeout`来允许额外的时间（最长一小时），在怀疑已经失败的成员被从组中驱逐之前。
