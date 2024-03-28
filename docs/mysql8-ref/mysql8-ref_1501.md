# 20.4.1 GTIDs 和组复制

> 原文：[`dev.mysql.com/doc/refman/8.0/en/group-replication-gtids.html`](https://dev.mysql.com/doc/refman/8.0/en/group-replication-gtids.html)

组复制使用 GTIDs（全局事务标识符）来精确跟踪每个服务器实例上已提交的事务。所有组成员都需要设置`gtid_mode=ON`和`enforce_gtid_consistency=ON`。从客户端接收的事务由接收它们的组成员分配一个 GTID。从组外部源服务器通过异步复制通道接收到的任何复制事务在到达组成员时保留它们的 GTID。

从客户端接收的事务分配的 GTID 使用`group_replication_group_name`系统变量指定的组名作为标识符的 UUID 部分，而不是接收事务的单个组成员的服务器 UUID。因此，所有直接由组接收的事务都可以被识别并分组在 GTID 集合中，不管最初接收它们的成员是谁。每个组成员都有一块连续的 GTID 保留供其使用，当这些 GTID 被消耗完时，它会保留更多。`group_replication_gtid_assignment_block_size`系统变量设置了块的大小，默认情况下每个块中有 100 万个 GTID。

当新成员加入时，由组自动生成的视图更改事件（`View_change_log_event`）在二进制日志中记录时被赋予 GTID。默认情况下，这些事件的 GTID 也使用`group_replication_group_name`系统变量指定的组名作为标识符的 UUID 部分。从 MySQL 8.0.26 开始，您可以设置 Group Replication 系统变量`group_replication_view_change_uuid`以在视图更改事件的 GTID 中使用替代 UUID，以便它们易于与从客户端接收的事务区分开来。如果您的设置允许在组之间进行故障切换，并且您需要识别和丢弃特定于备份组的事务，则这可能很有用。替代 UUID 必须与成员的服务器 UUID 不同。它还必须与使用`CHANGE REPLICATION SOURCE TO`语句的`ASSIGN_GTIDS_TO_ANONYMOUS_TRANSACTIONS`选项应用于匿名事务的 GTID 中的任何 UUID 不同。

从 MySQL 8.0.27 开始，设置 `GTID_ONLY=1`、`REQUIRE_ROW_FORMAT = 1` 和 `SOURCE_AUTO_POSITION = 1` 适用于 Group Replication 通道 `group_replication_applier` 和 `group_replication_recovery`。这些设置在创建 Group Replication 通道时自动设置，或者当复制组中的成员服务器升级到 8.0.27 或更高版本时自动设置。通常使用 `CHANGE REPLICATION SOURCE TO` 语句设置这些选项，但请注意，您无法为 Group Replication 通道禁用它们。设置了这些选项后，组成员不会在这些通道的复制元数据存储库中持久保存文件名和文件位置。在必要时，GTID 自动定位和 GTID 自动跳过用于定位正确的接收器和应用程序位置。

#### 额外事务

如果加入成员的 GTID 集中存在在组中现有成员中不存在的事务，则不允许完成分布式恢复过程，并且无法加入该组。如果进行了远程克隆操作，则这些事务将被删除和丢失，因为加入成员的数据目录被擦除。如果从捐赠者的二进制日志进行状态传输，则这些事务可能与组的事务发生冲突。

如果在 Group Replication 停止时在实例上执行管理事务，则可能会存在额外的事务。为了避免以这种方式引入新事务，请在发出管理语句之前始终将 `sql_log_bin` 系统变量的值设置为 `OFF`，然后在之后设置回 `ON`：

```sql
SET SQL_LOG_BIN=0;
<administrator action>
SET SQL_LOG_BIN=1;
```

将此系统变量设置为 `OFF` 意味着从那一点开始发生的事务不会写入二进制日志，并且不会分配 GTID。

如果加入成员存在额外的事务，请检查受影响服务器的二进制日志，查看额外事务的实际内容。将加入成员的数据和 GTID 集与当前组中的成员进行协调的最安全方法是使用 MySQL 的克隆功能，将内容从组中的一个服务器传输到受影响的服务器。有关如何执行此操作的说明，请参见 Section 7.6.7.3, “克隆远程数据”。如果需要该事务，请在成员成功重新加入后重新运行它。
