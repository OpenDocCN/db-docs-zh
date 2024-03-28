> 译文：[`dev.mysql.com/doc/refman/8.0/en/replication-features-transaction-inconsistencies.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-features-transaction-inconsistencies.html)

#### 19.5.1.34 复制和事务不一致性

根据您的复制配置，从中继日志中执行的事务序列可能存在不一致性。本节解释了如何避免不一致性并解决其引起的任何问题。

可能存在以下类型的不一致性：

+   *未完成的事务*。更新非事务表的事务已应用部分但不是全部更改。

+   *间隙*。在外部化事务集中，当给定一个有序的事务序列时，后续序列中的事务在先前序列中的某些其他事务之前应用时，会出现间隙。只有在使用多线程副本时才会出现间隙。

    要避免多线程复制中出现的间隙，请设置`replica_preserve_commit_order=ON`（从 MySQL 8.0.26 开始）或`slave_preserve_commit_order=ON`（在 MySQL 8.0.26 之前）。从 MySQL 8.0.27 开始，默认情况下设置此选项，因为从该版本开始，默认情况下所有副本都是多线程的。

    直到 MySQL 8.0.18，保留提交顺序需要启用二进制日志（`log_bin`）和副本更新日志（`log_replica_updates`或`log_slave_updates`），这是从 MySQL 8.0 开始的默认设置。从 MySQL 8.0.19 开始，在副本上设置`replica_preserve_commit_order=ON`或`slave_preserve_commit_order=ON`不需要启用二进制日志和副本更新日志，并且如果需要，可以禁用。

    在所有版本中，设置`replica_preserve_commit_order=ON`或`slave_preserve_commit_order=ON`需要设置`replica_parallel_type`（从 MySQL 8.0.26 开始）或`slave_parallel_type`（在 MySQL 8.0.26 之前）为`LOGICAL_CLOCK`。从 MySQL 8.0.27 开始（但不适用于早期版本），这是默认设置。

    在某些特定情况下，如`replica_preserve_commit_order`和`slave_preserve_commit_order`的描述中列出的情况，设置`replica_preserve_commit_order=ON`或`slave_preserve_commit_order=ON`无法在复制品上保留提交顺序，因此在这些情况下，序列中仍可能出现间隙。

    设置`replica_preserve_commit_order=ON`或`slave_preserve_commit_order=ON`不会阻止源二进制日志位置滞后。

+   *源二进制日志位置滞后*。即使没有间隙，也有可能在`Exec_master_log_pos`之后应用了事务。也就是说，直到点`N`之前的所有事务都已应用，但在点`N`之后没有应用任何事务，但`Exec_master_log_pos`的值小于`N`。在这种情况下，`Exec_master_log_pos`是已应用事务的“低水位标记”，落后于最近应用事务的位置。这只会发生在多线程复制中。启用`replica_preserve_commit_order`或`slave_preserve_commit_order`不会阻止源二进制日志位置滞后。

以下情景与部分应用事务、间隙和源二进制日志位置滞后有关：

1.  在复制线程运行时，可能会出现间隙和部分应用事务。

1.  **mysqld**关闭。无论是干净的还是不干净的关闭都会中止正在进行的事务，并可能留下间隙和部分应用事务。

1.  复制线程的`KILL`（在使用单线程复制时为 SQL 线程，在使用多线程复制时为协调器线程）。这会中止正在进行的事务，并可能留下间隙和部分应用事务。

1.  在应用程序线程中发生错误。这可能会留下间隙。如果错误发生在混合事务中，则该事务将被部分应用。在使用多线程复制时，未收到错误的工作线程会完成它们的队列，因此可能需要一些时间来停止所有线程。

1.  `STOP REPLICA` 在使用多线程复制时使用。发出 `STOP REPLICA` 后，复制品等待任何间隙被填充，然后更新 `Exec_master_log_pos`。这确保它永远不会留下间隙或源二进制日志位置滞后，除非上述任何情况适用，换句话说，在 `STOP REPLICA` 完成之前，要么发生错误，要么另一个线程发出 `KILL`，要么服务器重新启动。在这些情况下，`STOP REPLICA` 返回成功。

1.  如果中继日志中的最后一个事务仅被部分接收，并且多线程复制的协调器线程已开始将事务调度给工作线程，则 `STOP REPLICA` 最多等待 60 秒以接收事务。超时后，协调器放弃并中止事务。如果事务是混合的，则可能会留下未完成的部分。

1.  当正在进行的事务仅更新事务表时，`STOP REPLICA` 立即回滚并停止。如果正在进行的事务是混合的，`STOP REPLICA` 最多等待 60 秒以完成事务。超时后，它会中止事务，因此可能会留下未完成的部分。

系统变量 `rpl_stop_replica_timeout`（从 MySQL 8.0.26 开始）或 `rpl_stop_slave_timeout`（在 MySQL 8.0.26 之前）的全局设置与停止复制线程的过程无关。它只是使发出 `STOP REPLICA` 的客户端返回给客户端，但���制线程继续尝试停止。

如果复制通道存在间隙，会产生以下后果：

1.  复制品数据库处于可能从未存在于源上的状态。

1.  `SHOW REPLICA STATUS` 中的 `Exec_master_log_pos` 字段仅是“低水位标记”。换句话说，出现在该位置之前的事务已经提交，但在该位置之后的事务可能已经提交，也可能没有。

1.  对于该通道，`CHANGE REPLICATION SOURCE TO` 和 `CHANGE MASTER TO` 语句将因错误而失败，除非应用程序线程正在运行且语句仅设置接收器选项。

1.  如果使用`--relay-log-recovery`启动**mysqld**，则不会为该通道执行恢复，并会打印警告。

1.  如果使用`--dump-replica`或`--dump-slave`来使用**mysqldump**，它不会记录间隙的存在；因此，它会打印`CHANGE REPLICATION SOURCE TO` | `CHANGE MASTER TO`，并将`RELAY_LOG_POS`设置为`Exec_master_log_pos`中的“低水位”位置。

    在另一台服务器上应用转储并启动复制线程后，出现在该位置之后的事务将再次被复制。请注意，如果启用了 GTID（但在这种情况下不建议使用`--dump-replica`或`--dump-slave`），这是无害的。

如果复制通道存在源二进制日志位置滞后但没有间隙，则适用于上述情况 2 至 5，但不适用于情况 1。

源二进制日志位置信息以二进制格式持久化存储在内部表`mysql.slave_worker_info`中。[`START REPLICA [SQL_THREAD]`](start-replica.html "15.4.2.6 START REPLICA Statement")始终会查阅此信息，以便仅应用正确的事务。即使在`START REPLICA`之前将`replica_parallel_workers`或`slave_parallel_workers`更改为 0，甚至在使用`UNTIL`子句的情况下使用`START REPLICA`，这仍然有效。`START REPLICA UNTIL SQL_AFTER_MTS_GAPS`仅应用所需数量的事务以填补间隙。如果在消耗所有间隙之前告诉`START REPLICA`停止的`UNTIL`子句，则会留下剩余的间隙。

警告

`RESET REPLICA`会删除中继日志并重置复制位置。因此，在具有间隙的多线程复制中发出`RESET REPLICA`意味着复制丢失了有关间隙的任何信息，而没有纠正间隙。在这种情况下，如果使用基于二进制日志位置的复制，则恢复过程将失败。

当使用基于 GTID 的复制时（`GTID_MODE=ON`），并且使用`CHANGE REPLICATION SOURCE TO`语句为复制通道设置了`SOURCE_AUTO_POSITION`时，旧的中继日志在恢复过程中不再需要。相反，复制品可以使用 GTID 自动定位来计算与源相比缺少的事务。从 MySQL 8.0.26 开始，在使用基于 GTID 的复制时，用于在多线程复制品上解决间隙的基于二进制日志位置的过程将完全跳过。当跳过该过程时，`START REPLICA UNTIL SQL_AFTER_MTS_GAPS`语句的行为会有所不同，并且不会尝试检查事务序列中的间隙。您还可以发出`CHANGE REPLICATION SOURCE TO`语句，在非 GTID 复制品上不允许存在间隙的情况下。
