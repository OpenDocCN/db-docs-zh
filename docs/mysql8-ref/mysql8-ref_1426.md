# 19.4.11 延迟复制

> 原文：[`dev.mysql.com/doc/refman/8.0/en/replication-delayed.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-delayed.html)

MySQL 支持延迟复制，使得副本服务器故意比源服务器晚至少指定的时间执行事务。本节描述了如何在副本上配置复制延迟以及如何监视复制延迟。

在 MySQL 8.0 中，延迟复制的方法取决于两个时间戳，`immediate_commit_timestamp`和`original_commit_timestamp`（参见 Replication Delay Timestamps）。如果复制拓扑中的所有服务器都运行 MySQL 8.0 或更高版本，则使用这些时间戳来测量延迟复制。如果即时源或副本中有任何一个不使用这些时间戳，则使用 MySQL 5.7 中的延迟复制实现（参见 Delayed Replication）。本节描述了所有使用这些时间戳的服务器之间的延迟复制。

默认复制延迟为 0 秒。使用`CHANGE REPLICATION SOURCE TO SOURCE_DELAY=*`N`*`语句（从 MySQL 8.0.23 开始）或`CHANGE MASTER TO MASTER_DELAY=*`N`*`语句（MySQL 8.0.23 之前）将延迟设置为*`N`*秒。从源接收的事务直到比其在即时源上提交的时间晚至少*`N`*秒才执行。延迟是按事务发生的（而不是像以前的 MySQL 版本中那样按事件发生），实际延迟仅对`gtid_log_event`或`anonymous_gtid_log_event`施加。事务中的其他事件总是在这些事件之后立即执行，而不会对它们施加任何等待时间。

注意

`START REPLICA`和`STOP REPLICA`立即生效并忽略任何延迟。`RESET REPLICA`将延迟重置为 0。

`replication_applier_configuration`性能模式表包含`DESIRED_DELAY`列，显示使用`SOURCE_DELAY` | `MASTER_DELAY`选项配置的延迟。`replication_applier_status`性能模式表包含`REMAINING_DELAY`列，显示剩余的延迟秒数。

延迟复制可用于几个目的：

+   用于防止源上用户错误。通过延迟，您可以将延迟副本回滚到错误发生之前的时间。

+   为了测试系统在出现延迟时的行为。例如，在一个应用程序中，延迟可能是由于副本上的大量负载而引起的。然而，很难生成这种负载水平。延迟复制可以模拟延迟，而无需模拟负载。它还可以用于调试与滞后副本相关的条件。

+   为了查看数据库在过去的样子，而无需重新加载备份。例如，通过配置一个延迟一周的副本，如果您需要查看数据库在最近几天开发之前的样子，可以检查延迟副本。

#### 复制延迟时间戳

MySQL 8.0 提供了一种新的方法来测量复制拓扑中的延迟（也称为复制滞后），该方法依赖于写入二进制日志的每个事务（而不是每个事件）关联的 GTID 的以下时间戳。

+   `original_commit_timestamp`: 事务被写入（提交）到原始来源二进制日志时距离时代的微秒数。

+   `immediate_commit_timestamp`: 事务被写入（提交）到直接来源二进制日志时距离时代的微秒数。

**mysqlbinlog**的输出以两种格式显示这些时间戳，即从时代开始的微秒数和基于用户定义时区的`TIMESTAMP`格式，以便更易读。例如：

```sql
#170404 10:48:05 server id 1  end_log_pos 233 CRC32 0x016ce647     GTID    last_committed=0
\ sequence_number=1    original_committed_timestamp=1491299285661130    immediate_commit_timestamp=1491299285843771
# original_commit_timestamp=1491299285661130 (2017-04-04 10:48:05.661130 WEST)
# immediate_commit_timestamp=1491299285843771 (2017-04-04 10:48:05.843771 WEST)
 /*!80001 SET @@SESSION.original_commit_timestamp=1491299285661130*//*!*/;
   SET @@SESSION.GTID_NEXT= 'aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa:1'/*!*/;
# at 233
```

通常情况下，事务应用到的所有副本上的`original_commit_timestamp`始终相同。在源-副本复制中，事务在（原始）源二进制日志中的`original_commit_timestamp`始终与其`immediate_commit_timestamp`相同。在副本的中继日志中，事务的`original_commit_timestamp`和`immediate_commit_timestamp`与源二进制日志中的相同；而在其自己的二进制日志中，事务的`immediate_commit_timestamp`对应于副本提交事务的时间。

在群组复制设置中，当原始来源是群组的成员时，`original_commit_timestamp`在事务准备提交时生成。换句话说，当它在原始来源上执行完毕并且其写入集准备发送到群组的所有成员进行认证时。当原始来源是群组外的服务器时，`original_commit_timestamp`被保留。特定事务的相同`original_commit_timestamp`被复制到群组中的所有服务器，以及从成员复制的群组外的任何副本。从 MySQL 8.0.26 开始，事务的每个接收者还使用`immediate_commit_timestamp`在其二进制日志中存储本地提交时间。

视图更改事件，这是组复制独有的特殊情况。包含这些事件的交易由每个组成员生成，但共享相同的 GTID（因此，它们不是首先在源中执行，然后被复制到组中，而是组的所有成员执行并应用相同的交易）。在 MySQL 8.0.26 之前，这些交易的`original_commit_timestamp`设置为零，并且在可查看的输出中以这种方式显示。从 MySQL 8.0.26 开始，为了提高可观察性，组成员为与视图更改事件相关的交易设置本地时间戳值。

#### 监视复制延迟

在以前的 MySQL 版本中监视复制延迟（滞后）最常见的方法之一是依赖于`SHOW REPLICA STATUS`输出中的`Seconds_Behind_Master`字段。然而，在使用比传统源-副本设置更复杂的复制拓扑时，如组复制时，此度量标准并不适用。在 MySQL 8 中添加`immediate_commit_timestamp`和`original_commit_timestamp`提供了关于复制延迟的更精细信息。在支持这些时间戳的拓扑中监视复制延迟的推荐方法是使用以下性能模式表。

+   `replication_connection_status`：与源的连接的当前状态，提供连接线程排队到中继日志的最后一个和当前交易的信息。

+   `replication_applier_status_by_coordinator`：仅在使用多线程副本时显示协调器线程的当前状态，提供协调器线程缓冲到工作线程队列的最后一个交易的信息，以及它当前正在缓冲的交易。

+   `replication_applier_status_by_worker`：应用来自源的交易的线程（们）的当前状态，提供由复制 SQL 线程应用的交易的信息，或者在使用多线程副本时，每个工作线程应用的交易的信息。

使用这些表，您可以监视对应线程处理的最后一个交易和该线程当前正在处理的交易的信息。这些信息包括：

+   一个交易的 GTID

+   从副本的中继日志中检索的交易的`original_commit_timestamp`和`immediate_commit_timestamp`

+   线程开始处理交易的时间

+   对于最后处理的交易，线程完成处理的时间

除了性能模式表之外，`SHOW REPLICA STATUS`的输出还有三个字段显示：

+   `SQL_Delay`: 一个非负整数，表示使用`CHANGE REPLICATION SOURCE TO SOURCE_DELAY=*`N`*`（从 MySQL 8.0.23 开始）或`CHANGE MASTER TO MASTER_DELAY=N`（在 MySQL 8.0.23 之前）配置的复制延迟，以秒为单位。

+   `SQL_Remaining_Delay`: 当`Replica_SQL_Running_State`为`Waiting until MASTER_DELAY seconds after master executed event`时，此字段包含一个整数，表示延迟剩余的秒数。在其他时候，此字段为`NULL`。

+   `Replica_SQL_Running_State`: 表示 SQL 线程状态的字符串（类似于`Replica_IO_State`）。该值与由`SHOW PROCESSLIST`显示的 SQL 线程的`State`值相同。

当复制 SQL 线程在执行事件之前等待延迟时间时，`SHOW PROCESSLIST`将其`State`值显示为`Waiting until MASTER_DELAY seconds after master executed event`。
