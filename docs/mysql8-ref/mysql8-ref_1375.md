> 原文：[`dev.mysql.com/doc/refman/8.0/en/replication-administration-skip.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-administration-skip.html)

#### 19.1.7.3 跳过事务

如果由于复制事务中的事件问题而导致复制停止，则可以通过在副本上跳过失败的事务来恢复复制。在跳过事务之前，请确保复制 I/O（接收器）线程以及 SQL（应用程序）线程已停止。

首先，您需要确定导致错误的复制事件。错误的详细信息和最后成功应用的事务记录在性能模式表`replication_applier_status_by_worker`中。您可以使用**mysqlbinlog**检索和显示围绕错误发生时记录的事件。有关如何执行此操作的说明，请参阅第 9.5 节，“时间点（增量）恢复”。或者，您可以在副本上发出`SHOW RELAYLOG EVENTS`或在源上发出`SHOW BINLOG EVENTS`。

在跳过事务并重新启动副本之前，请检查以下几点：

+   停止复制的事务是否来自未知或不受信任的来源？如果是，请调查原因，以确定是否有任何安全考虑因素表明不应重新启动副本。

+   停止复制的事务是否需要在副本上应用？如果是，请进行适当的更正并重新应用该事务，或者在副本上手动协调数据。

+   停止复制的事务是否需要在源上应用？如果不需要，请在原始发生地的服务器上手动撤消该事务。

要跳过事务，请根据情况选择以下方法之一：

+   当使用 GTIDs 时（`gtid_mode`为`ON`），请参阅第 19.1.7.3.1 节，“跳过带有 GTIDs 的事务”。

+   当不使用 GTIDs 或正在逐步引入 GTIDs 时（`gtid_mode`为`OFF`、`OFF_PERMISSIVE`或`ON_PERMISSIVE`），请参阅第 19.1.7.3.2 节，“跳过不带 GTIDs 的事务”。

+   如果您已经使用`CHANGE REPLICATION SOURCE TO`或`CHANGE MASTER TO`语句的`ASSIGN_GTIDS_TO_ANONYMOUS_TRANSACTIONS`选项在复制通道上启用了 GTID 分配，请参阅第 19.1.7.3.2 节，“跳过没有 GTIDs 的事务”。在复制通道上使用`ASSIGN_GTIDS_TO_ANONYMOUS_TRANSACTIONS`与为通道引入基于 GTID 的复制不同，您不能在这些通道上使用基于 GTID 的复制的事务跳过方法。

要在跳过事务后重新启动复制，请发出`START REPLICA`命令，如果副本是多源副本，则带有`FOR CHANNEL`子句。

##### 19.1.7.3.1 跳过带有 GTIDs 的事务

当使用 GTIDs 时（`gtid_mode`为`ON`），即使事务内容被过滤掉，已提交事务的 GTID 也会在副本上持久化。这个特性可以防止副本在使用 GTID 自动定位重新连接到源时检索先前被过滤的事务。它还可以用于在副本上跳过一个事务，方法是提交一个空事务来代替失败的事务。

当您已经使用`CHANGE REPLICATION SOURCE TO`语句的`ASSIGN_GTIDS_TO_ANONYMOUS_TRANSACTIONS`选项在复制通道上启用了 GTID 分配时，这种跳过事务的方法不适用。

如果失败的事务在工作线程中生成了错误，您可以直接从性能模式表`replication_applier_status_by_worker`中的`APPLYING_TRANSACTION`字段获取其 GTID。要查看该事务是什么，请在副本上发出`SHOW RELAYLOG EVENTS`或在源上发出`SHOW BINLOG EVENTS`，并搜索输出以找到由该 GTID 引导的事务。

当您已经评估了失败的事务是否需要采取其他适当的操作（如安全考虑）后，要跳过它，请在副本上提交一个与失败事务具有相同 GTID 的空事务。例如：

```sql
SET GTID_NEXT='aaa-bbb-ccc-ddd:N';
BEGIN;
COMMIT;
SET GTID_NEXT='AUTOMATIC';
```

在副本上存在这个空事务意味着当您发出`START REPLICA`语句重新启动复制时，副本将使用自动跳过功能来忽略失败的事务，因为它看到具有该 GTID 的事务已经被应用。如果副本是多源副本，则在提交空事务时不需要指定通道名称，但在发出`START REPLICA`时需要指定通道名称。

请注意，如果此副本正在使用二进制日志记录，则在将来该副本成为源或主时，空事务将进入复制流。如果您需要避免这种可能性，请考虑刷新和清除副本的二进制日志，就像这个例子一样：

```sql
FLUSH LOGS;
PURGE BINARY LOGS TO 'binlog.000146';
```

空事务的 GTID 是持久的，但事务本身通过清除二进制日志文件而被移除。

##### 19.1.7.3.2 跳过没有 GTID 的事务

当 GTID 未被使用或正在逐步引入时（`gtid_mode`为`OFF`、`OFF_PERMISSIVE`或`ON_PERMISSIVE`），您可以通过发出`SET GLOBAL sql_replica_skip_counter`语句（从 MySQL 8.0.26 开始）或`SET GLOBAL sql_slave_skip_counter`语句来跳过指定数量的事件。或者，您可以通过发出`CHANGE REPLICATION SOURCE TO`或`CHANGE MASTER TO`语句来跳过一个或多个事件，以将源二进制日志位置向前移动。

当您在复制通道上使用`ASSIGN_GTIDS_TO_ANONYMOUS_TRANSACTIONS`选项的`CHANGE REPLICATION SOURCE TO`或`CHANGE MASTER TO`语句启用 GTID 分配时，这些方法也是适用的。

当您使用这些方法时，重要的是要理解，您并不一定是跳过一个完整的事务，就像之前描述的基于 GTID 的方法总是这样。这些非 GTID 方法并不知道事务的存在，而是操作事件。二进制日志被组织为一系列称为事件组的组，每个事件组由一系列事件组成。

+   对于事务性表，一个事件组对应一个事务。

+   对于非事务性表，一个事件组对应一个单独的 SQL 语句。

一个单独的事务可以包含对事务性和非事务性表的更改。

当您使用`SET GLOBAL sql_replica_skip_counter`或`SET GLOBAL sql_slave_skip_counter`语句跳过事件，并且结果位置位于事件组的中间时，复制品会继续跳过事件，直到达到该组的末尾。然后执行将从下一个事件组开始。`CHANGE REPLICATION SOURCE TO`或`CHANGE MASTER TO`语句没有此功能，因此您必须小心确定重新启动复制的正确位置，即事件组的开头。但是，使用`CHANGE REPLICATION SOURCE TO`或`CHANGE MASTER TO`意味着您无需计算需要跳过的事件，就像使用`SET GLOBAL sql_replica_skip_counter`或`SET GLOBAL sql_slave_skip_counter`一样，而是可以直接指定重新启动的位置。

###### 19.1.7.3.2.1 使用`SET GLOBAL sql_slave_skip_counter`跳过事务

当您已经评估了失败的事务以及之前描述的任何其他适当操作（如安全考虑）后，计算您需要跳过的事件数。一个事件通常对应于二进制日志中的一个 SQL 语句，但请注意，使用`AUTO_INCREMENT`或`LAST_INSERT_ID()`的语句在二进制日志中计为两个事件。当使用二进制日志事务压缩时，压缩的事务负载（`Transaction_payload_event`）被计为单个计数值，因此其中的所有事件都作为一个单元跳过。

如果您想跳过完整的事务，可以计算事件直到事务结束，或者只是跳过相关的事件组。请记住，使用`SET GLOBAL sql_replica_skip_counter`或`SET GLOBAL sql_slave_skip_counter`时，复制品会继续跳过到事件组的末尾。确保您不要跳得太远，进入下一个事件组或事务，以免也被跳过。

发出以下`SET`语句，其中*`N`*是要从源跳过的事件数：

```sql
SET GLOBAL sql_slave_skip_counter = *N*

Or from MySQL 8.0.26:
SET GLOBAL sql_replica_skip_counter = *N*
```

如果设置了`gtid_mode=ON`，或者复制 I/O（接收器）和 SQL（应用程序）线程正在运行，则无法发出此语句。

`SET GLOBAL sql_replica_skip_counter` 或 `SET GLOBAL sql_slave_skip_counter` 语句没有立即生效。当你在这个 `SET` 语句之后再次发出 `START REPLICA` 语句时，系统变量 `sql_replica_skip_counter` 或 `sql_slave_skip_counter` 的新值会生效，并且事件会被跳过。那个 `START REPLICA` 语句也会自动将系统变量的值设置回 0。如果复制品是多源复制品，则在发出那个 `START REPLICA` 语句时，`FOR CHANNEL` 子句是必需的。确保你命名了正确的通道，否则事件会在错误的通道上被跳过。

###### 19.1.7.3.2.2 跳过事务使用 `CHANGE MASTER TO`

当你评估了失败的事务以及之前描述的任何其他适当操作（如安全考虑）后，确定源二进制日志中表示适当位置以重新启动复制的坐标（文件和位置）。这可以是导致问题的事件后的事件组的开始，或者下一个事务的开始。复制 I/O（接收器）线程在下次启动时从这些坐标开始读取源，跳过失败的事件。确保你准确地确定了位置，因为这个语句不考虑事件组。

发出以下 `CHANGE REPLICATION SOURCE TO` 或 `CHANGE MASTER TO` 语句，其中 *`source_log_name`* 是包含重新启动位置的二进制日志文件，*`source_log_pos`* 是表示二进制日志文件中重新启动位置的数字：

```sql
CHANGE MASTER TO MASTER_LOG_FILE='*source_log_name*', MASTER_LOG_POS=*source_log_pos*;

Or from MySQL 8.0.24:
CHANGE REPLICATION SOURCE TO SOURCE_LOG_FILE='*source_log_name*', SOURCE_LOG_POS=*source_log_pos*;
```

如果复制品是多源复制品，则必须在 `CHANGE REPLICATION SOURCE TO` 或 `CHANGE MASTER TO` 语句中使用 `FOR CHANNEL` 子句来命名适当的通道。

如果 `SOURCE_AUTO_POSITION=1` 或 `MASTER_AUTO_POSITION=1` 被设置，或者复制 I/O（接收器）和 SQL（应用程序）线程正在运行，则无法发出此语句。如果在通常设置为 `SOURCE_AUTO_POSITION=1` 或 `MASTER_AUTO_POSITION=1` 时需要使用此跳过事务的方法，你可以在发出语句时将设置更改为 `SOURCE_AUTO_POSITION=0` 或 `MASTER_AUTO_POSITION=0`，然后之后再将其改回。例如：

```sql
CHANGE MASTER TO MASTER_AUTO_POSITION=0, MASTER_LOG_FILE='binlog.000145', MASTER_LOG_POS=235;
CHANGE MASTER TO MASTER_AUTO_POSITION=1;

Or from MySQL 8.0.24:

CHANGE REPLICATION SOURCE TO SOURCE_AUTO_POSITION=0, SOURCE_LOG_FILE='binlog.000145', SOURCE_LOG_POS=235;
CHANGE REPLICATION SOURCE TO SOURCE_AUTO_POSITION=1;
```
