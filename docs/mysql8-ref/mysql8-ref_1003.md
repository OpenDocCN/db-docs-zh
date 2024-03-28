> 原文：[`dev.mysql.com/doc/refman/8.0/en/stop-group-replication.html`](https://dev.mysql.com/doc/refman/8.0/en/stop-group-replication.html)

#### 15.4.3.2 停止 GROUP_REPLICATION 语句

```sql
STOP GROUP_REPLICATION
```

停止 Group Replication。此语句需要 `GROUP_REPLICATION_ADMIN` 权限（或已弃用的 `SUPER` 权限）。一旦您发出 `STOP GROUP_REPLICATION`，该成员将被设置为 `super_read_only=ON`，这确保在 Group Replication 停止时无法对成员进行写入。该成员上运行的任何其他异步复制通道也将停止。在启动此成员上的 Group Replication 时在 `START GROUP_REPLICATION` 语句中指定的任何用户凭据将从内存中删除，并且在再次启动 Group Replication 时必须提供。

警告

使用此语句时要非常小心，因为它会将服务器实例从组中移除，这意味着它不再受到 Group Replication 一致性保证机制的保护。为了完全安全起见，在发出此语句之前，请确保您的应用程序无法连接到该实例，以避免任何陈旧读取的可能性。

`STOP GROUP_REPLICATION` 语句会停止组成员上的异步复制通道，但不像 `STOP REPLICA` 那样隐式提交正在进行的事务。这是因为在 Group Replication 组成员上，关闭操作期间提交的其他事务会使成员与组不一致，并导致重新加入时出现问题。为了避免在停止 Group Replication 时正在进行的事务失败提交，从 MySQL 8.0.28 开始，当将 GTID 分配为 `gtid_next` 系统变量的值时，不能发出 `STOP GROUP_REPLICATION` 语句。

`group_replication_components_stop_timeout` 系统变量指定了组复制在发出此语句后等待其各个模块完成正在进行的进程的时间。超时用于解决组复制组件无法正常停止的情况，这可能发生在成员在错误状态下被驱逐出组，或者在诸如 MySQL Enterprise Backup 正在持有成员表的全局锁时。在这种情况下，成员无法停止应用程序线程或完成分布式恢复过程以重新加入。`STOP GROUP_REPLICATION` 不会完成，直到情况得到解决（例如，通过释放锁），或者组件超时到期并且模块无论其状态如何都会关闭。在 MySQL 8.0.27 之前，默认组件超时为 31536000 秒，即 365 天。使用此设置，组件超时对于刚才描述的情况并不起作用，因此建议在这些 MySQL 8.0 版本中使用较低的设置。从 MySQL 8.0.27 开始，默认值为 300 秒；这意味着如果在此之前未解决情况，则在 5 分钟后停止组复制组件，从而允许成员重新启动并重新加入。
