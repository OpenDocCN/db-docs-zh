> 原文：[`dev.mysql.com/doc/refman/8.0/en/group-replication-performance-xcom-cache-reduce.html`](https://dev.mysql.com/doc/refman/8.0/en/group-replication-performance-xcom-cache-reduce.html)

#### 20.7.6.2 减小缓存大小

XCom 消息缓存大小的最小设置是 1 GB，直到 MySQL 8.0.20 版本。从 MySQL 8.0.21 开始，最小设置为 134217728 字节（128 MB），这使得可以在可用内存量受限的主机上部署。如果主机处于不稳定网络上，不建议将`group_replication_message_cache_size`设置得非常低，因为较小的消息缓存会使组成员在短暂失去连接后更难重新连接。

如果重新连接的成员无法从 XCom 消息缓存中检索到所需的所有消息，则该成员必须离开组并重新加入，以从另一个成员的二进制日志中使用分布式恢复检索缺失的事务。从 MySQL 8.0.21 开始，默认情况下，离开组的成员会进行三次自动重新加入尝试，因此重新加入组的过程可以在没有操作员干预的情况下进行。然而，使用分布式恢复重新加入是一个明显更长且更复杂的过程，比从 XCom 消息缓存中检索消息需要更长时间，因此成员需要更长时间才能变得可用，组的性能可能会受到影响。在稳定网络上，可以最小化成员短暂失去连接的频率和持续时间，因此这种情况发生的频率也应该最小化，因此组可能能够容忍较小的 XCom 消息缓存大小而不会对其性能产生显著影响。

如果您考虑减小缓存大小限制，可以使用以下语句查询 Performance Schema 表`memory_summary_global_by_event_name`：

```sql
SELECT * FROM performance_schema.memory_summary_global_by_event_name
  WHERE EVENT_NAME LIKE 'memory/group_rpl/GCS_XCom::xcom_cache';
```

这返回消息缓存的内存使用统计，包括当前缓存条目数和当前缓存大小。如果您减小缓存大小限制，XCom 会删除已经决定和传递的最旧条目，直到当前大小低于限制。在此移除过程进行时，XCom 可能暂时超过缓存大小限制。
