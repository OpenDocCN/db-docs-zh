> 原文：[`dev.mysql.com/doc/refman/8.0/en/replication-semisync-monitoring.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-semisync-monitoring.html)

#### 19.4.10.3 半同步复制监控

半同步复制的插件公开了许多状态变量，使您能够监视其操作。要检查状态变量的当前值，请使用 `SHOW STATUS`：

```sql
mysql> SHOW STATUS LIKE 'Rpl_semi_sync%';
```

从 MySQL 8.0.26 开始，提供了新版本的源和副本插件，这些插件在系统变量和状态变量中用“source”和“replica”替换了“master”和“slave”这些术语。如果安装了新的 `rpl_semi_sync_source` 和 `rpl_semi_sync_replica` 插件，则新的系统变量和状态变量可用，而旧的则不可用。如果安装了旧的 `rpl_semi_sync_master` 和 `rpl_semi_sync_slave` 插件，则旧的系统变量和状态变量可用，而新的则不可用。在一个实例中不能同时安装相关插件的新旧版本。

所有 `Rpl_semi_sync_*`xxx`*` 状态变量在 第 7.1.10 节，“服务器状态变量” 中有描述。一些示例包括：

+   `Rpl_semi_sync_source_clients` 或 `Rpl_semi_sync_master_clients`

    连接到源服务器的半同步副本数量。

+   `Rpl_semi_sync_source_status` 或 `Rpl_semi_sync_master_status`

    半同步复制当前是否在源服务器上运行。如果插件已启用且提交确认尚未发生，则值为 1。如果插件未启用或源由于提交确认超时而回退到异步复制，则值为 0。

+   `Rpl_semi_sync_source_no_tx` 或 `Rpl_semi_sync_master_no_tx`

    未被副本成功确认的提交数量。

+   `Rpl_semi_sync_source_yes_tx` 或 `Rpl_semi_sync_master_yes_tx`

    被副本成功确认的提交数量。

+   `Rpl_semi_sync_replica_status` 或 `Rpl_semi_sync_slave_status`

    半同步复制当前是否在副本上运行。如果插件已启用且复制 I/O（接收器）线程正在运行，则为 1，否则为 0。

当源端由于提交阻塞超时或副本追赶而在异步或半同步复制之间切换时，它会适当地设置`Rpl_semi_sync_source_status`或`Rpl_semi_sync_master_status`状态变量的值。源端从半同步自动回退到异步复制意味着即使半同步复制实际上当前不可用，`rpl_semi_sync_source_enabled`或`rpl_semi_sync_master_enabled`系统变量在源端可能仍然具有值 1。您可以监视`Rpl_semi_sync_source_status`或`Rpl_semi_sync_master_status`状态变量来确定源端当前是在使用异步还是半同步复制。
