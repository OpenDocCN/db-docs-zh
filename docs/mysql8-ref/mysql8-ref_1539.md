# 20.7.6 XCom 缓存管理

> 原文：[`dev.mysql.com/doc/refman/8.0/en/group-replication-performance-xcom-cache.html`](https://dev.mysql.com/doc/refman/8.0/en/group-replication-performance-xcom-cache.html)

20.7.6.1 增加缓存大小

20.7.6.2 减少缓存大小

用于 Group Replication 的组通信引擎（XCom，一种 Paxos 变体）包括一个用于在共识协议的一部分中交换组成员之间的消息（及其元数据）的缓存。消息缓存除了其他功能外，还用于在成员在与其他组成员无法通信的一段时间后重新连接到组时恢复丢失的消息。

从 MySQL 8.0.16 开始，可以使用`group_replication_message_cache_size`系统变量为 XCom 的消息缓存设置缓存大小限制。如果达到缓存大小限制，XCom 会删除已经决定和传递的最旧条目。因为正在尝试重新连接的不可达成员会随机选择任何其他成员来恢复丢失的消息，所以所有组成员都应该设置相同的缓存大小限制。因此，每个成员的缓存中应该有相同的消息。

在 MySQL 8.0.16 之前，缓存大小为 1 GB，从 MySQL 8.0.16 开始，默认设置的缓存大小相同。请确保系统上有足够的内存可用于您选择的缓存大小限制，考虑到 MySQL Server 的其他缓存和对象池的大小。请注意，使用`group_replication_message_cache_size`设置的限制仅适用于缓存中存储的数据，缓存结构需要额外的 50 MB 内存。

在选择`group_replication_message_cache_size`设置时，应参考成员被驱逐之前的时间段内预期的消息量。这段时间的长度由`group_replication_member_expel_timeout`系统变量控制，该变量确定了等待期限（最长一小时），除了成员最初的 5 秒检测期外，允许成员返回到组中而不被驱逐。请注意，在 MySQL 8.0.21 之前，这段时间默认为成员变得不可用后的 5 秒，这只是在产生怀疑之前的检测期，因为`group_replication_member_expel_timeout`系统变量设置的额外驱逐超时默认为零。从 8.0.21 开始，驱逐超时默认为 5 秒，因此默认情况下，成员在至少离开 10 秒后才会被驱逐。
