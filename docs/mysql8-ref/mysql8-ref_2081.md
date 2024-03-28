> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-tp-thread-group-stats-table.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-tp-thread-group-stats-table.html)

#### 29.12.16.2 The tp_thread_group_stats Table

注意

此处描述的性能模式表在 MySQL 8.0.14 版本中可用。在 MySQL 8.0.14 之前，请改用相应的 `INFORMATION_SCHEMA` 表；请参阅第 28.5.3 节，“INFORMATION_SCHEMA TP_THREAD_GROUP_STATS 表”。

`tp_thread_group_stats` 表报告每个线程组的统计信息。每个组有一行。

`tp_thread_group_stats` 表具有以下列：

+   `TP_GROUP_ID`

    线程组 ID。这是表内的唯一键。

+   `CONNECTIONS_STARTED`

    开始的连接数量。

+   `CONNECTIONS_CLOSED`

    连接关闭的次数。

+   `QUERIES_EXECUTED`

    执行的语句数量。当语句开始执行时，此数字会递增，而不是在语句执行完成时。

+   `QUERIES_QUEUED`

    接收到的排队等待执行的语句数量。这不包括线程组能够立即开始执行而无需排队的语句，这可能发生在第 7.6.3.3 节，“线程池操作”中描述的条件下。

+   `THREADS_STARTED`

    启动的线程数量。

+   `PRIO_KICKUPS`

    根据 `thread_pool_prio_kickup_timer` 系统变量的值，已从低优先级队列移至高优先级队列的语句数量。如果此数字快速增加，请考虑增加该变量的值。快速增加的计数器意味着优先级系统未能阻止事务过早启动。对于 `InnoDB`，这很可能意味着由于太多并发事务而导致性能下降。

+   `STALLED_QUERIES_EXECUTED`

    由于执行时间超过 `thread_pool_stall_limit` 系统变量的值而被定义为停滞的语句数量。

+   `BECOME_CONSUMER_THREAD`

    已分配消费者线程角色的次数。

+   `BECOME_RESERVE_THREAD`

    已分配保留线程角色的次数。

+   `BECOME_WAITING_THREAD`

    线程被分配等待线程角色的次数。当语句被排队时，即使在正常操作中，这种情况也经常发生，因此在负载高的系统中，语句排队的情况下，这个值的快速增加是正常的。

+   `WAKE_THREAD_STALL_CHECKER`

    检查线程决定唤醒或创建线程以处理某些语句或处理等待线程角色的次数。

+   `SLEEP_WAITS`

    `THD_WAIT_SLEEP`等待的次数。当线程进入睡眠状态时（例如，通过调用`SLEEP()`函数）会发生这种情况。

+   `DISK_IO_WAITS`

    `THD_WAIT_DISKIO`等待的次数。当线程执行磁盘 I/O 时，很可能不会命中文件系统缓存。这种等待发生在缓冲池读取和写入数据到磁盘时，而不是正常的文件读取和写入时。

+   `ROW_LOCK_WAITS`

    `THD_WAIT_ROW_LOCK`等待另一个事务释放行锁的次数。

+   `GLOBAL_LOCK_WAITS`

    `THD_WAIT_GLOBAL_LOCK`等待全局锁释放的次数。

+   `META_DATA_LOCK_WAITS`

    `THD_WAIT_META_DATA_LOCK`等待元数据锁释放的次数。

+   `TABLE_LOCK_WAITS`

    `THD_WAIT_TABLE_LOCK`等待表被解锁以便语句访问的次数。

+   `USER_LOCK_WAITS`

    `THD_WAIT_USER_LOCK`等待用户线程构造的特殊锁的次数。

+   `BINLOG_WAITS`

    `THD_WAIT_BINLOG_WAITS`等待二进制日志释放的次数。

+   `GROUP_COMMIT_WAITS`

    `THD_WAIT_GROUP_COMMIT`等待的次数。当组提交必须等待其他方完成事务的一部分时会发生这种情况。

+   `FSYNC_WAITS`

    `THD_WAIT_SYNC`等待文件同步操作的次数。

`tp_thread_group_stats`表具有以下索引：

+   在(`TP_GROUP_ID`)上的唯一索引

`TRUNCATE TABLE`不允许用于`tp_thread_group_stats`表。
