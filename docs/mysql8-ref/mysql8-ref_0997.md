> 原文：[`dev.mysql.com/doc/refman/8.0/en/start-replica.html`](https://dev.mysql.com/doc/refman/8.0/en/start-replica.html)

#### 15.4.2.6 启动 REPLICA 语句

```sql
START REPLICA [*thread_types*] [*until_option*] [*connection_options*] [*channel_option*]

*thread_types*:
    [*thread_type* [, *thread_type*] ... ]

*thread_type*:
    IO_THREAD | SQL_THREAD

*until_option*:
    UNTIL {   {SQL_BEFORE_GTIDS | SQL_AFTER_GTIDS} = *gtid_set*
          |   MASTER_LOG_FILE = '*log_name*', MASTER_LOG_POS = *log_pos*
          |   SOURCE_LOG_FILE = '*log_name*', SOURCE_LOG_POS = *log_pos*
          |   RELAY_LOG_FILE = '*log_name*', RELAY_LOG_POS = *log_pos*
          |   SQL_AFTER_MTS_GAPS  }

*connection_options*:
    [USER='*user_name*'] [PASSWORD='*user_pass*'] [DEFAULT_AUTH='*plugin_name*'] [PLUGIN_DIR='*plugin_dir*']

*channel_option*:
    FOR CHANNEL *channel*

*gtid_set*:
    *uuid_set* [, *uuid_set*] ...
    | ''

*uuid_set*:
    *uuid*:*interval*[:*interval*]...

*uuid*:
    *hhhhhhhh*-*hhhh*-*hhhh*-*hhhh*-*hhhhhhhhhhhh*

*h*:
    [0-9,A-F]

*interval*:
    *n*[-*n*]

    (*n* >= 1)
```

`START REPLICA`启动复制线程，可以一起启动或分开启动。从 MySQL 8.0.22 开始，使用`START REPLICA`代替从该版本开始弃用的`START SLAVE`。在 MySQL 8.0.22 之前的版本中，请使用`START SLAVE`。

`START REPLICA`需要`REPLICATION_SLAVE_ADMIN`权限（或已弃用的`SUPER`权限）。`START REPLICA`导致正在进行的事务隐式提交。请参阅第 15.3.3 节，“导致隐式提交的语句”。

对于线程类型选项，您可以指定`IO_THREAD`，`SQL_THREAD`，这两者，或者都不指定。只有启动的线程受语句影响。

+   没有线程类型选项的`START REPLICA`启动所有复制线程，同时具有线程类型选项的`START REPLICA`也是如此。

+   `IO_THREAD`启动复制接收器线程，从源服务器读取事件并将其存储在中继日志中。

+   `SQL_THREAD`启动复制应用程序线程，从中继日志中读取事件并执行它们。多线程复制（使用`replica_parallel_workers`或`slave_parallel_workers`>0）使用协调器线程和多个应用程序线程应用事务，而`SQL_THREAD`启动所有这些线程。

重要提示

`START REPLICA`在所有复制线程启动后向用户发送确认。然而，复制接收线程可能尚未成功连接到源，或者应用程序线程可能在启动后立即停止应用事件。`START REPLICA`在启动线程后不会继续监视线程，因此如果线程随后停止或无法连接，则不会警告您。您必须检查复制的错误日志以查看复制线程生成的错误消息，或者使用`SHOW REPLICA STATUS`检查它们是否正常运行。成功的`START REPLICA`语句会导致`SHOW REPLICA STATUS`显示`Replica_SQL_Running=Yes`，但可能会或可能不会显示`Replica_IO_Running=Yes`，因为只有在接收线程同时运行和连接时才会显示`Replica_IO_Running=Yes`。有关更多信息，请参见第 19.1.7.1 节，“检查复制状态”。

可选的`FOR CHANNEL *`channel`*`子句使您能够命名语句适用于哪个复制通道。提供`FOR CHANNEL *`channel`*`子句将`START REPLICA`语句应用于特定的复制通道。如果未命名任何子句且没有额外的通道存在，则该语句适用于默认通道。如果`START REPLICA`语句在使用多个通道时未定义通道，则此语句将为所有通道启动指定的线程。有关更多信息，请参见第 19.2.2 节，“复制通道”。

Group Replication 的复制通道（`group_replication_applier`和`group_replication_recovery`）由服务器实例自动管理。`START REPLICA`不能与`group_replication_recovery`通道一起使用，只能在 Group Replication 未运行时才能与`group_replication_applier`通道一起使用。`group_replication_applier`通道只有一个应用程序线程，没有接收线程，因此如果需要，可以使用`SQL_THREAD`选项启动它，而不使用`IO_THREAD`选项。

`START REPLICA`支持可插拔的用户密码身份验证（参见第 8.2.17 节，“可插拔身份验证”），使用`USER`、`PASSWORD`、`DEFAULT_AUTH`和`PLUGIN_DIR`选项，如下列表所述。当您使用这些选项时，必须启动接收线程（`IO_THREAD`选项）或所有复制线程；不能仅启动复制应用程序线程（`SQL_THREAD`选项）。

`USER`

用户账户的用户名。如果使用`PASSWORD`，则必须设置此选项。该选项不能设置为空或 null 字符串。

`PASSWORD`

指定用户账户的密码。

`DEFAULT_AUTH`

认证插件的名称。默认为 MySQL 本机认证。

`PLUGIN_DIR`

认证插件的位置。

重要

使用`START REPLICA`设置的密码在写入 MySQL Server 的日志、性能模式表和`显示进程列表`语句时会被掩码。但是，在传输过程中，它以明文形式发送到副本服务器实例。为了保护传输中的密码，使用 SSL/TLS 加密、SSH 隧道或其他方法保护连接，以防止未经授权的查看，用于发出`START REPLICA`的客户端与副本服务器实例之间的连接。

`UNTIL`子句使副本开始复制，然后处理事务直到您在`UNTIL`子句中指定的点，然后再次停止。`UNTIL`子句可用于使副本继续进行，直到您想要跳过不需要的事务的点之前，然后如第 19.1.7.3 节，“跳过事务”中所述跳过事务。要识别事务，您可以使用源二进制日志或副本中继日志的**mysqlbinlog**，或使用`显示二进制日志事件`语句。

您还可以使用`UNTIL`子句来通过逐个处理事务或分段处理事务来调试复制。如果您使用`UNTIL`子句来执行此操作，请使用`--skip-slave-start`选项启动副本，或从 MySQL 8.0.24 开始，使用`skip_slave_start`系统变量，以防止副本服务器启动时运行 SQL 线程。在过程完成后，删除选项或系统变量设置，以防止在意外服务器重新启动时被遗忘。

`显示副本状态`语句包括显示`UNTIL`条件当前值的输出字段。`UNTIL`条件持续时间取决于受影响线程是否仍在运行，并在它们停止时被移除。

`UNTIL`子句作用于复制应用程序线程（`SQL_THREAD`选项）。您可以使用`SQL_THREAD`选项或让副本默认启动两个线程。如果仅使用`IO_THREAD`选项，则`UNTIL`子句将被忽略，因为未启动应用程序线程。

您在`UNTIL`子句中指定的点可以是以下选项中的任何一个（且仅一个）：

`SOURCE_LOG_FILE` 和 `SOURCE_LOG_POS`（来自 MySQL 8.0.23），或 `MASTER_LOG_FILE` 和 `MASTER_LOG_POS`（到 MySQL 8.0.22）

这些选项使得复制应用程序处理事务直到其中继日志中的位置，该位置由源服务器上二进制日志中相应点的文件名和文件位置标识。应用程序线程在指定位置之后或之后找到最近的事务边界，完成应用事务，并在那里停止。对于压缩的事务有效载荷，请指定压缩的`Transaction_payload_event`的结束位置。

当在`CHANGE REPLICATION SOURCE TO`语句上设置了`GTID_ONLY`选项以阻止复制通道在复制元数据存储库中持久化文件名和文件位置时，仍然可以使用这些选项。文件名和文件位置在内存中被跟踪。

`RELAY_LOG_FILE` 和 `RELAY_LOG_POS`

这些选项使得复制应用程序处理事务直到副本的中继日志中的位置，该位置由中继日志文件名和该文件中的位置标识。应用程序线程在指定位置之后或之后找到最近的事务边界，完成应用事务，并在那里停止。对于压缩的事务有效载荷，请指定压缩的`Transaction_payload_event`的结束位置。

当在`CHANGE REPLICATION SOURCE TO`语句上设置了`GTID_ONLY`选项以阻止复制通道在复制元数据存储库中持久化文件名和文件位置时，仍然可以使用这些选项。文件名和文件位置在内存中被跟踪。

`SQL_BEFORE_GTIDS`

此选项使得复制应用程序开始处理事务，并在遇到任何位于指定 GTID 集中的事务时停止。来自 GTID 集的遇到的事务不会被应用，也不会应用 GTID 集中的任何其他事务。该选项接受包含一个或多个全局事务标识符的 GTID 集作为参数（参见 GTID Sets）。GTID 集中的事务不一定按照其 GTID 的顺序出现在复制流中，因此应用程序停止之前的事务不一定是最早的。

`SQL_AFTER_GTIDS`

此选项使得复制应用程序开始处理事务，并在处理完指定 GTID 集中的所有事务时停止。该选项接受包含一个或多个全局事务标识符的 GTID 集作为参数（参见 GTID Sets）。

使用 `SQL_AFTER_GTIDS` 后，复制线程在处理完 GTID 集中的所有事务后停止。事务按接收顺序处理，因此可能包括不属于 GTID 集的事务，但在集合中的所有事务提交之前接收（并处理）。例如，执行 `START REPLICA UNTIL SQL_AFTER_GTIDS = 3E11FA47-71CA-11E1-9E33-C80AA9429562:11-56` 会导致复制获取（并处理）源中的所有事务，直到处理完序列号为 11 到 56 的所有事务，然后在达到该点后停止，不再处理任何额外的事务。

`SQL_AFTER_GTIDS` 与多线程应用程序不兼容。如果在多线程应用程序中使用此选项，将会引发警告，并且复制将切换到单线程模式。根据用例，可能可以使用 `START REPLICA UNTIL MASTER_LOG_POS` 或 `START REPLICA UNTIL SQL_BEFORE_GTIDS`。您还可以使用 `WAIT_UNTIL_SQL_THREAD_AFTER_GTIDS()`，它会等待到达正确位置，但不会停止应用程序线程。

`SQL_AFTER_MTS_GAPS`

对于仅多线程复制（具有`replica_parallel_workers`或`slave_parallel_workers` > 0）的情况，此选项使复制进程处理事务，直到从中继日志执行的事务序列中不再存在间隙为止。在使用多线程复制时，以下情况可能会导致间隙出现：

+   协调器线程停止。

+   应用程序线程发生错误。

+   **mysqld** 意外关闭。

当复制通道存在间隙时，复制的数据库处于可能从未存在于源上的状态。复制在内部跟踪间隙并禁止执行会删除间隙信息的 `CHANGE REPLICATION SOURCE TO` 语句。

在 MySQL 8.0.26 之前，在具有从中继日志执行的事务序列中存在间隙的多线程复制上发出 `START REPLICA` 会生成警告。要纠正这种情况，解决方案是使用 `START REPLICA UNTIL SQL_AFTER_MTS_GAPS`。有关更多信息，请参见 Section 19.5.1.34, “Replication and Transaction Inconsistencies”。

从 MySQL 8.0.26 开始，当使用基于 GTID 的复制和 GTID 自动定位（`SOURCE_AUTO_POSITION=1`）用于通道时，完全跳过检查事务序列中的间隙的过程，因为可以使用 GTID 自动定位来解决事务中的间隙。在这种情况下，`START REPLICA UNTIL SQL_AFTER_MTS_GAPS`只是在找到要执行的第一个事务时停止应用程序线程，并且不尝试检查事务序列中的间隙。您也可以像往常一样继续使用`CHANGE REPLICATION SOURCE TO`语句，并且通道可以进行中继日志恢复。

从 MySQL 8.0.27 开始，默认情况下所有副本都是多线程的。当`replica_preserve_commit_order=ON`或`slave_preserve_commit_order=ON`为副本设置时，这也是从 MySQL 8.0.27 开始的默认设置，除了在`replica_preserve_commit_order`和`slave_preserve_commit_order`的描述中列出的特定情况外，不应该出现间隙。如果为副本设置了`replica_preserve_commit_order=OFF`或`slave_preserve_commit_order=OFF`，这是在 MySQL 8.0.27 之前的默认设置，事务的提交顺序不会被保留，因此出现间隙的可能性要大得多。

如果没有使用 GTIDs，并且您需要将失败的多线程副本更改为单线程模式，您可以按照以下顺序发出以下一系列语句：

```sql
START SLAVE UNTIL SQL_AFTER_MTS_GAPS;
SET @@GLOBAL.slave_parallel_workers = 0;
START SLAVE SQL_THREAD;

Or from MySQL 8.0.26:
START REPLICA UNTIL SQL_AFTER_MTS_GAPS;
SET @@GLOBAL.replica_parallel_workers = 0;
START REPLICA SQL_THREAD;
```
