> 原文：[`dev.mysql.com/doc/refman/8.0/en/binary-log-transaction-compression.html`](https://dev.mysql.com/doc/refman/8.0/en/binary-log-transaction-compression.html)

#### 7.4.4.5 二进制日志事务压缩

从 MySQL 8.0.20 开始，您可以在 MySQL 服务器实例上启用二进制日志事务压缩。启用二进制日志事务压缩后，事务负载将使用 zstd 算法进行压缩，然后作为单个事件（`Transaction_payload_event`）写入服务器的二进制日志文件。

在将压缩的事务负载发送到副本、其他组复制组成员或客户端（如**mysqlbinlog**）时，压缩的事务负载保持压缩状态。接收线程不会对其进行解压缩，并且仍以压缩状态写入中继日志。因此，二进制日志事务压缩在事务发起者和接收者（以及它们的备份）上节省了存储空间，并在服务器实例之间发送事务时节省了网络带宽。

当需要检查其中包含的各个事件时，压缩的事务负载将被解压缩。例如，`Transaction_payload_event`由应用程序线程解压缩，以便在接收端应用其中包含的事件。在恢复期间，通过**mysqlbinlog**重放事务时，以及通过`SHOW BINLOG EVENTS`和`SHOW RELAYLOG EVENTS`语句进行解压缩。

你可以使用`binlog_transaction_compression`系统变量在 MySQL 服务器实例上启用二进制日志事务压缩，该变量默认为`OFF`。您还可以使用`binlog_transaction_compression_level_zstd`系统变量设置用于压缩的 zstd 算法的级别。该值确定了压缩的努力程度，从 1（最低努力）到 22（最高努力）。随着压缩级别的增加，压缩比例增加，从而减少了事务负载所需的存储空间和网络带宽。然而，数据压缩所需的努力也增加，消耗了源服务器上的时间、CPU 和内存资源。压缩努力的增加与压缩比例的增加之间没有线性关系。

注意

*NDB 8.0.31 之前*：可以在 NDB Cluster 中启用二进制日志事务压缩，但仅在使用--binlog-transaction-compression 选项（可能还包括--binlog-transaction-compression-level-zstd）启动服务器时；在运行时更改`binlog_transaction_compression`和`binlog_transaction_compression_level_zstd`系统变量的值对`NDB`表的日志记录没有影响。

*NDB 8.0.31 及更高版本*：您可以在运行时使用该版本引入的`ndb_log_transaction_compression`系统变量启用对使用`NDB`存储引擎的表的压缩事务的二进制日志记录，并使用`ndb_log_transaction_compression_level_zstd`控制压缩级别。在命令行或`my.cnf`文件中使用`--binlog-transaction-compression`启动**mysqld**会自动启用`ndb_log_transaction_compression`，并忽略`--ndb-log-transaction-compression`选项的任何设置；要仅为`NDB`存储引擎禁用二进制日志事务压缩，需在启动**mysqld**后在客户端会话中设置`ndb_log_transaction_compression=OFF`。

以下类型的事件不包括在二进制日志事务压缩中，因此始终以未压缩形式写入二进制日志：

+   与事务的 GTID 相关的事件（包括匿名 GTID 事件）。

+   其他类型的控制事件，例如视图更改事件和心跳事件。

+   事故事件和包含它们的任何事务的全部内容。

+   非事务事件和包含它们的任何事务的全部内容。涉及非事务性和事务性存储引擎混合的事务不会对其有效负载进行压缩。

+   使用基于语句的二进制日志记录的事件。二进制日志事务压缩仅适用于基于行的二进制日志格式。

可以在包含压缩事务的二进制日志文件上使用二进制日志加密。

##### 7.4.4.5.1 启用二进制日志事务压缩时的行为

具有压缩有效负载的事务可以像任何其他事务一样回滚，并且也可以通过常规过滤选项在副本中过滤掉。二进制日志事务压缩可以应用于 XA 事务。

当启用二进制日志事务压缩时，服务器的`max_allowed_packet`和`replica_max_allowed_packet`或`slave_max_allowed_packet`限制仍然适用，并且是基于`Transaction_payload_event`的压缩大小加上事件头使用的字节进行计量。

重要

压缩事务负载作为一个单独的数据包发送，而不是在未使用二进制日志事务压缩时，将事务的每个事件作为单独的数据包发送。如果您的复制拓扑处理大型事务，请注意，当使用二进制日志事务压缩时，一个在未使用二进制日志事务压缩时可以成功复制的大型事务，可能由于其大小而停止复制。

对于多线程工作者，每个事务（包括其 GTID 事件和`Transaction_payload_event`）被分配给一个工作线程。工作线程解压缩事务负载，并逐个应用其中的各个事件。如果在`Transaction_payload_event`中应用任何事件时发现错误，则将报告整个事务失败给协调者。当`replica_parallel_type`或`slave_parallel_type`设置为`DATABASE`时，事务调度之前将受事务影响的所有数据库映射。与未压缩事务相比，使用二进制日志事务压缩与`DATABASE`策略可以减少并行性，后者将每个事件映射并调度。

对于半同步复制（参见 Section 19.4.10, “Semisynchronous Replication”)，当完整的`Transaction_payload_event`被接收时，副本确认事务。

当启用二进制日志校验和（这是默认设置）时，复制源服务器不会为压缩事务负载中的单个事件写入校验和。相反，为完整的`Transaction_payload_event`写入校验和，并为未压缩的任何事件写入单独的校验和，例如与 GTID 相关的事件。

对于`SHOW BINLOG EVENTS`和`SHOW RELAYLOG EVENTS`语句，`Transaction_payload_event`首先作为一个单元打印，然后解压缩并打印其中的每个事件。

对于引用事件结束位置的操作，比如`START REPLICA`（或在 MySQL 8.0.22 之前的版本中，`START SLAVE`）带有`UNTIL`子句，`SOURCE_POS_WAIT()`或`MASTER_POS_WAIT()`，以及`sql_replica_skip_counter`或`sql_slave_skip_counter`，您必须指定压缩事务有效载荷（`Transaction_payload_event`）的结束位置。当使用`sql_replica_skip_counter`或`sql_slave_skip_counter`跳过事件时，压缩的事务有效载荷被计为单个计数器值，因此其中的所有事件都作为一个单元被跳过。

##### 7.4.4.5.2 组合压缩和未压缩的事务有效载荷

支持二进制日志事务压缩的 MySQL 服务器版本可以处理压缩和未压缩的事务有效载荷的混合。

+   与二进制日志事务压缩相关的系统变量不需要在所有组复制组成员上设置相同，并且不会从源复制到副本在复制拓扑中。您可以决定是否对具有二进制日志的每个 MySQL 服务器实例启用二进制日志事务压缩。

+   如果启用了事务压缩，然后在服务器上禁用了压缩，则未来在该服务器上发起的事务不会应用压缩，但已经压缩的事务有效载荷仍然可以被处理和显示。

+   如果通过设置`binlog_transaction_compression`的会话值为个别会话指定了事务压缩，则二进制日志可以包含压缩和未压缩的事务有效载荷的混合。

当复制拓扑中的源和其副本都启用了二进制日志事务压缩时，副本接收压缩的事务有效载荷并将其压缩写入其中继日志。它解压事务有效载荷以应用事务，然后在应用后再次压缩以写入其二进制日志。任何下游副本都会接收压缩的事务有效载荷。

当复制拓扑中的源具有启用二进制日志事务压缩，但其副本没有启用时，副本接收压缩的事务有效载荷并将其压缩写入其中继日志。它解压事务有效载荷以应用事务，然后将其未压缩写入其自己的二进制日志（如果有）。任何下游副本接收未压缩的事务有效载荷。

当复制拓扑中的源没有启用二进制日志事务压缩，但其副本启用时，如果副本具有二进制日志，则在应用事务后压缩事务有效载荷，并将压缩的事务有效载荷写入其二进制日志。任何下游副本接收压缩的事务有效载荷。

当 MySQL 服务器实例没有二进制日志时，如果它是来自 MySQL 8.0.20 的版本，则无论其对`binlog_transaction_compression`的值如何，它都可以接收、处理和显示压缩的事务有效载荷。这些服务器实例接收的压缩事务有效载荷以其压缩状态写入中继日志，因此它们间接受益于复制拓扑中其他服务器执行的压缩。

MySQL 8.0.20 之前的版本的副本无法从启用二进制日志事务压缩的源进行复制。MySQL 8.0.20 或更高版本的副本可以从不支持二进制日志事务压缩的早期版本的源进行复制，并在将其写入自己的二进制日志时对从该源接收的事务进行自己的压缩。

##### 7.4.4.5.3 监视二进制日志事务压缩

您可以使用性能模式表`binary_log_transaction_compression_stats`监视二进制日志事务压缩的影响。统计数据包括监控期间的数据压缩比率，您还可以查看压缩对服务器上最后一个事务的影响。您可以通过截断表来重置统计信息。二进制日志和中继日志的统计数据是分开的，因此您可以看到每种日志类型的压缩影响。MySQL 服务器实例必须具有二进制日志才能生成这些统计信息。

Performance Schema 表 `events_stages_current` 显示事务何时处于解压缩或压缩阶段，以及显示该阶段的进度。压缩是由处理事务的工作线程在事务提交之前执行的，前提是在最终捕获缓存中没有排除事务的事件（例如，事故事件）。当需要解压缩时，会逐个事件从有效载荷中进行解压缩。

**mysqlbinlog** 使用 `--verbose` 选项时，会包含注释，说明压缩事务有效载荷的压缩大小和未压缩大小，以及所使用的压缩算法。

您可以在复制连接的协议级别启用连接压缩，使用 `CHANGE REPLICATION SOURCE TO` 语句（从 MySQL 8.0.23 开始）或 `CHANGE MASTER TO` 语句（在 MySQL 8.0.23 之前），或 `replica_compressed_protocol` 或 `slave_compressed_protocol` 系统变量的 `SOURCE_COMPRESSION_ALGORITHMS` | `MASTER_COMPRESSION_ALGORITHMS` 和 `SOURCE_ZSTD_COMPRESSION_LEVEL` | `MASTER_ZSTD_COMPRESSION_LEVEL` 选项。如果在启用连接压缩的系统中启用了二进制日志事务压缩，连接压缩的影响会减小，因为可能很少有机会进一步压缩已经压缩的事务有效载荷。但是，连接压缩仍然可以操作未压缩的事件和消息头。如果您需要节省存储空间以及网络带宽，可以在启用连接压缩的情况下启用二进制日志事务压缩。有关复制连接的连接压缩的更多信息，请参见 Section 6.2.8, “Connection Compression Control”。

对于组复制，当消息超过`group_replication_compression_threshold`系统变量设置的阈值时，默认启用压缩。您还可以通过使用`group_replication_recovery_compression_algorithms`和`group_replication_recovery_zstd_compression_level`系统变量，为从捐赠者的二进制日志进行状态传输的消息配置压缩。如果在配置了这些内容的系统中启用了二进制日志事务压缩，组复制的消息压缩仍然可以在未压缩事件和消息头上运行，但其影响会减小。有关组复制消息压缩的更多信息，请参见第 20.7.4 节，“消息压缩”。
