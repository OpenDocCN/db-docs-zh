> 原文：[`dev.mysql.com/doc/refman/8.0/en/group-replication-performance-xcom-cache-increase.html`](https://dev.mysql.com/doc/refman/8.0/en/group-replication-performance-xcom-cache-increase.html)

#### 20.7.6.1 增加缓存大小

如果一个成员缺席的时间不够长，以至于还没有被从组中驱逐，它可以重新连接并从另一个成员的 XCom 消息缓存中检索丢失的事务，然后再次参与组。然而，如果在成员缺席期间发生的事务已经被从其他成员的 XCom 消息缓存中删除，因为它们达到了最大大小限制，那么该成员无法通过这种方式重新连接。

Group Replication 的组通信系统（GCS）通过警告消息向您发出警告，当一个消息被从消息缓存中删除时，这个消息可能会被当前不可达的成员用于恢复。这个警告消息被记录在所有活动组成员上（对于每个不可达成员只记录一次）。尽管组成员无法确定不可达成员看到的最后一条消息是什么，但警告消息表明缓存大小可能不足以支持您选择的成员被驱逐之前的等待时间。

在这种情况下，考虑根据`group_replication_member_expel_timeout`系统变量指定的时间段内预期的消息量，再加上 5 秒的检测期，来增加`group_replication_message_cache_size`限制，以便缓存包含所有成员成功返回所需的所有丢失消息。如果预计某个成员将在异常时间内变得不可达，还可以考虑暂时增加缓存大小限制。
