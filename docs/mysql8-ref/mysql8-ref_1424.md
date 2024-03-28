> 原文：[`dev.mysql.com/doc/refman/8.0/en/replication-semisync-interface.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-semisync-interface.html)

#### 19.4.10.2 配置半同步复制

当您为半同步复制安装源和复制品插件时（参见 Section 19.4.10.1，“安装半同步复制”），系统变量变得可用以控制插件行为。

要检查半同步复制状态变量的当前值，请使用 `SHOW VARIABLES`：

```sql
mysql> SHOW VARIABLES LIKE 'rpl_semi_sync%';
```

从 MySQL 8.0.26 开始，提供了源和复制品插件的新版本，这些版本在系统变量和状态变量中将“master”和“slave”替换为“source”和“replica”。如果安装新的 `rpl_semi_sync_source` 和 `rpl_semi_sync_replica` 插件，则新的系统变量和状态变量可用，但旧的则不可用。如果安装旧的 `rpl_semi_sync_master` 和 `rpl_semi_sync_slave` 插件，则旧的系统变量和状态变量可用，但新的则不可用。在一个实例中不能同时安装相关插件的新旧版本。

所有 `rpl_semi_sync_*`xxx`*` 系统变量在 Section 19.1.6.2，“复制源选项和变量”和 Section 19.1.6.3，“复制品服务器选项和变量”中有描述。一些关键的系统变量包括：

`rpl_semi_sync_source_enabled` 或 `rpl_semi_sync_master_enabled`

控制源服务器上是否启用半同步复制。要启用或禁用插件，请将此变量分别设置为 1 或 0。默认值为 0（关闭）。

`rpl_semi_sync_replica_enabled` 或 `rpl_semi_sync_slave_enabled`

控制复制品上是否启用半同步复制。

`rpl_semi_sync_source_timeout` 或 `rpl_semi_sync_master_timeout`

以毫秒为单位的值，控制源在提交时等待来自复制品的确认的超时时间，超时后将回滚到异步复制。默认值为 10000（10 秒）。

`rpl_semi_sync_source_wait_for_replica_count` 或 `rpl_semi_sync_master_wait_for_slave_count`

控制源在返回会话之前必须接收的每个事务的副本确认数。默认值为 1，意味着源只等待一个副本确认接收事务事件。

`rpl_semi_sync_source_wait_point` 或 `rpl_semi_sync_master_wait_point` 系统变量控制半同步源服务器在返回事务提交状态给客户端之前等待副本确认事务接收的时间点。这些值是允许的：

+   `AFTER_SYNC`（默认）：源将每个事务写入其二进制日志和副本，并将二进制日志同步到磁盘。源在同步后等待副本确认事务接收。在接收到确认后，源将事务提交到存储引擎并返回结果给客户端，然后客户端可以继续。

+   `AFTER_COMMIT`：源将每个事务写入其二进制日志和副本，同步二进制日志，并提交事务到存储引擎。源在提交后等待副本确认事务接收。在接收到确认后，源返回结果给客户端，然后客户端可以继续。

这些设置的复制特性如下所示：

+   使用 `AFTER_SYNC`，所有客户端在同一时间看到已提交的事务，即在副本确认并在源上提交到存储引擎之后。因此，所有客户端在源上看到相同的数据。

    在源故障的情况下，所有在源上提交的事务已被复制到副本（保存到其中继日志）。源的意外退出并故障转移到副本是无损的，因为副本是最新的。如上所述，在故障转移后不应再重用源。

+   使用 `AFTER_COMMIT`，发起事务的客户端只有在服务器提交到存储引擎并接收到副本确认后才会收到返回状态。在提交后和副本确认前，其他客户端可能在提交客户端之前看到已提交的事务。

    如果出现问题导致副本未处理事务，那么在源意外退出并故障转移到副本的情况下，这些客户端可能会看到相对于在源上看到的数据有所丢失。

从 MySQL 8.0.23 开始，您可以通过启用系统变量`replication_sender_observe_commit_only`和`replication_optimize_for_static_plugin_config`来提高半同步复制的性能，前者限制回调，后者添加共享锁并避免不必要的锁获取。这些设置在副本数量增加时非常有帮助，因为锁争用可能会降低性能。半同步复制源服务器还可以通过启用这些系统变量获得性能优势，因为它们使用与副本相同的锁定机制。
