> 原文：[`dev.mysql.com/doc/refman/8.0/en/reset-replica.html`](https://dev.mysql.com/doc/refman/8.0/en/reset-replica.html)

#### 15.4.2.4 RESET REPLICA Statement

```sql
RESET REPLICA [ALL] [*channel_option*]

*channel_option*:
    FOR CHANNEL *channel*
```

`RESET REPLICA`使副本忘记其在源二进制日志中的位置。从 MySQL 8.0.22 开始，请使用`RESET REPLICA`代替从该版本开始已弃用的`RESET SLAVE`。在 MySQL 8.0.22 之前的版本中，请使用`RESET SLAVE`。

此语句用于进行清理启动；它清除了复制元数据存储库，删除了所有中继日志文件，并启动了一个新的中继日志文件。它还将使用`CHANGE REPLICATION SOURCE TO`语句（从 MySQL 8.0.23 开始）或`CHANGE MASTER TO`语句（在 MySQL 8.0.23 之前）指定的复制延迟重置为 0。

注意

所有中继日志文件都将被删除，即使它们尚未完全被复制 SQL 线程执行。（这是一个可能存在的情况，如果您已经发出了`STOP REPLICA`语句或者如果副本负载很高。）

对于使用 GTIDs 的服务器（`gtid_mode`为`ON`），执行`RESET REPLICA`对 GTID 执行历史没有影响。该语句不会更改`gtid_executed`或`gtid_purged`的值，也不会更改`mysql.gtid_executed`表。如果需要重置 GTID 执行历史，请使用`RESET MASTER`，即使启用了 GTID 的服务器是一个禁用了二进制日志记录的副本。

`RESET REPLICA`需要`RELOAD`权限。

要使用`RESET REPLICA`，必须停止复制 SQL 线程和复制 I/O（接收器）线程，因此在运行中的副本上，在执行`RESET REPLICA`之前，请使用`STOP REPLICA`。要在 Group Replication 组成员上使用`RESET REPLICA`，成员状态必须为`OFFLINE`，表示插件已加载但成员当前不属于任何组。可以通过使用`STOP GROUP REPLICATION`语句将组成员下线。

可选的 `FOR CHANNEL *`channel`*` 子句使您可以命名语句适用于哪个复制通道。提供 `FOR CHANNEL *`channel`*` 子句将 `RESET REPLICA` 语句应用于特定的复制通道。将 `FOR CHANNEL *`channel`*` 子句与 `ALL` 选项结合使用会删除指定的通道。如果未命名通道且不存在额外通道，则该语句适用于默认通道。当存在多个复制通道时，发出不带 `FOR CHANNEL *`channel`*` 子句的 `RESET REPLICA ALL` 语句会删除 *所有* 复制通道，并仅重新创建默认通道。有关更多信息，请参见 Section 19.2.2, “Replication Channels”。

`RESET REPLICA` 不会更改任何复制连接参数，包括源主机名和端口、复制用户帐户及其密码、`PRIVILEGE_CHECKS_USER` 帐户、`REQUIRE_ROW_FORMAT` 选项、`REQUIRE_TABLE_PRIMARY_KEY_CHECK` 选项和 `ASSIGN_GTIDS_TO_ANONYMOUS_TRANSACTIONS` 选项。如果要更改任何复制连接参数，可以在服务器启动后使用 `CHANGE REPLICATION SOURCE TO` 语句（从 MySQL 8.0.23 开始）或 `CHANGE MASTER TO` 语句（MySQL 8.0.23 之前）来实现。如果要删除所有复制连接参数，请使用 `RESET REPLICA ALL`。`RESET REPLICA ALL` 还会清除由 `CHANGE REPLICATION SOURCE TO` | `CHANGE MASTER TO` 设置的 `IGNORE_SERVER_IDS` 列表。当使用了 `RESET REPLICA ALL` 后，如果要再次将实例用作复制品，则需要在服务器启动后发出 `CHANGE REPLICATION SOURCE TO` | `CHANGE MASTER TO` 语句以指定新的连接参数。

从 MySQL 8.0.27 开始，可以在 `CHANGE REPLICATION SOURCE TO` 语句中设置 `GTID_ONLY` 选项，以阻止复制通道在复制元数据存储库中持久化文件名和文件位置。当发出 `RESET REPLICA` 语句时，复制元数据存储库会同步。`RESET REPLICA ALL` 删除而不是更新存储库，因此它们会隐式同步。

在发出`RESET REPLICA`但在发出`START REPLICA`之前发生意外服务器退出或故意重启时，复制连接参数的保留取决于用于复制元数据的存储库：

+   当服务器上设置了`master_info_repository=TABLE`和`relay_log_info_repository=TABLE`时（这是 MySQL 8.0 的默认设置），复制连接参数将在`InnoDB`表`mysql.slave_master_info`和`mysql.slave_relay_log_info`中作为`RESET REPLICA`操作的一部分保留在崩溃安全的表中。它们也会保留在内存中。在发出`RESET REPLICA`但在发出`START REPLICA`之前发生意外服务器退出或故意重启时，复制连接参数将从表中检索并重新应用到通道上。这种情况适用于 MySQL 8.0.13 的连接元数据存储库，以及 MySQL 8.0.19 的应用程序元数据存储库。

+   如果服务器上设置了`master_info_repository=FILE`和`relay_log_info_repository=FILE`，这在 MySQL 8.0 之前已经被弃用，或者 MySQL 服务器版本早于上述版本，则复制连接参数仅保留在内存中。如果由于意外服务器退出或故意重启而在发出`RESET REPLICA`后立即重新启动副本**mysqld**，连接参数将丢失。在这种情况下，您必须在服务器启动后发出`CHANGE REPLICATION SOURCE TO`语句（从 MySQL 8.0.23 开始）或`CHANGE MASTER TO`语句（在 MySQL 8.0.23 之前）来重新指定连接参数，然后再发出`START REPLICA`。

`RESET REPLICA`不会更改任何受该语句影响的通道的复制过滤器设置（例如`--replicate-ignore-table`）。但是，`RESET REPLICA ALL`会删除该语句删除的通道上设置的复制过滤器。当删除的通道或通道重新创建时，将复制为副本指定的任何全局复制过滤器，并且不应用特定于通道的复制过滤器。有关更多信息，请参见第 19.2.5.4 节，“基于复制通道的过滤器”。

`RESET REPLICA`会导致正在进行的事务隐式提交。参见 Section 15.3.3, “Statements That Cause an Implicit Commit”。

如果复制 SQL 线程在停止时正在复制临时表，并且发出`RESET REPLICA`，则这些复制的临时表将在副本上被删除。

注意

当在 NDB 集群副本 SQL 节点上使用`RESET REPLICA`时，会清除`mysql.ndb_apply_status`表。在使用此语句时，应该记住`ndb_apply_status`使用`NDB`存储引擎，因此被附加到集群的所有 SQL 节点共享。

您可以通过在执行`RESET REPLICA`之前发出`SET` `GLOBAL @@``ndb_clear_apply_status=OFF`来覆盖此行为，这样可以防止副本在这种情况下清除`ndb_apply_status`表。
