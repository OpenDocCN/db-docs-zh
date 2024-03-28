# 20.7.1 优化组通信线程

> 原文：[`dev.mysql.com/doc/refman/8.0/en/group-replication-fine-tuning-the-group-communication-thread.html`](https://dev.mysql.com/doc/refman/8.0/en/group-replication-fine-tuning-the-group-communication-thread.html)

当 Group Replication 插件加载时，组通信线程（GCT）在循环中运行。GCT 从组和插件接收消息，处理关于法定人数和故障检测的任务，发送一些保持活动的消息，并处理来自/发送到服务器/组的传入和传出事务。GCT 在队列中等待传入消息。当没有消息时，GCT 会等待。在实际进入睡眠之前，通过将这种等待时间设置得稍长一些（进行主动等待）可能在某些情况下证明是有益的。这是因为另一种选择是操作系统将 GCT 从处理器中切换出来并进行上下文切换。

要强制 GCT 进行主动等待，使用 `group_replication_poll_spin_loops` 选项，使 GCT 在实际轮询队列获取下一条消息之前，循环执行无关紧要的操作，达到配置的循环次数。

例如：

```sql
mysql> SET GLOBAL group_replication_poll_spin_loops= 10000;
```
