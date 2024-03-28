# 17.16 InnoDB 与 MySQL Performance Schema 的集成

> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-performance-schema.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-performance-schema.html)

17.16.1 使用 Performance Schema 监控 InnoDB 表的 ALTER TABLE 进度

17.16.2 使用 Performance Schema 监控 InnoDB 互斥等待

本节简要介绍了 `InnoDB` 与 Performance Schema 的集成。有关全面的 Performance Schema 文档，请参阅 第二十九章，*MySQL Performance Schema*。

您可以使用 MySQL Performance Schema 功能 对某些内部 `InnoDB` 操作进行分析。这种调优主要面向专家用户，他们评估优化策略以克服性能瓶颈。数据库管理员也可以使用此功能进行容量规划，以查看其典型工作负载是否遇到特定 CPU、RAM 和磁盘存储组合的性能瓶颈；如果是，可以判断是否通过增加系统某部分的容量来改善性能。

要使用此功能检查 `InnoDB` 性能：

+   您必须对如何使用 Performance Schema 功能 有一般了解。例如，您应该知道如何启用仪器和消费者，以及如何查询 `performance_schema` 表以检索数据。有关简介概述，请参阅 第 29.1 节，“Performance Schema 快速入门”。

+   您应该熟悉可用于 `InnoDB` 的 Performance Schema 仪器。要查看与 `InnoDB` 相关的仪器，您可以查询 `setup_instruments` 表，以查找包含 '`innodb`' 的仪器名称。

    ```sql
    mysql> SELECT *
           FROM performance_schema.setup_instruments
           WHERE NAME LIKE '%innodb%';
    +-------------------------------------------------------+---------+-------+
    | NAME                                                  | ENABLED | TIMED |
    +-------------------------------------------------------+---------+-------+
    | wait/synch/mutex/innodb/commit_cond_mutex             | NO      | NO    |
    | wait/synch/mutex/innodb/innobase_share_mutex          | NO      | NO    |
    | wait/synch/mutex/innodb/autoinc_mutex                 | NO      | NO    |
    | wait/synch/mutex/innodb/buf_pool_mutex                | NO      | NO    |
    | wait/synch/mutex/innodb/buf_pool_zip_mutex            | NO      | NO    |
    | wait/synch/mutex/innodb/cache_last_read_mutex         | NO      | NO    |
    | wait/synch/mutex/innodb/dict_foreign_err_mutex        | NO      | NO    |
    | wait/synch/mutex/innodb/dict_sys_mutex                | NO      | NO    |
    | wait/synch/mutex/innodb/recalc_pool_mutex             | NO      | NO    |
    ...
    | wait/io/file/innodb/innodb_data_file                  | YES     | YES   |
    | wait/io/file/innodb/innodb_log_file                   | YES     | YES   |
    | wait/io/file/innodb/innodb_temp_file                  | YES     | YES   |
    | stage/innodb/alter table (end)                        | YES     | YES   |
    | stage/innodb/alter table (flush)                      | YES     | YES   |
    | stage/innodb/alter table (insert)                     | YES     | YES   |
    | stage/innodb/alter table (log apply index)            | YES     | YES   |
    | stage/innodb/alter table (log apply table)            | YES     | YES   |
    | stage/innodb/alter table (merge sort)                 | YES     | YES   |
    | stage/innodb/alter table (read PK and internal sort)  | YES     | YES   |
    | stage/innodb/buffer pool load                         | YES     | YES   |
    | memory/innodb/buf_buf_pool                            | NO      | NO    |
    | memory/innodb/dict_stats_bg_recalc_pool_t             | NO      | NO    |
    | memory/innodb/dict_stats_index_map_t                  | NO      | NO    |
    | memory/innodb/dict_stats_n_diff_on_level              | NO      | NO    |
    | memory/innodb/other                                   | NO      | NO    |
    | memory/innodb/row_log_buf                             | NO      | NO    |
    | memory/innodb/row_merge_sort                          | NO      | NO    |
    | memory/innodb/std                                     | NO      | NO    |
    | memory/innodb/sync_debug_latches                      | NO      | NO    |
    | memory/innodb/trx_sys_t::rw_trx_ids                   | NO      | NO    |
    ...
    +-------------------------------------------------------+---------+-------+
    155 rows in set (0.00 sec)
    ```

    有关已仪器化的 `InnoDB` 对象的其他信息，您可以查询 Performance Schema 实例表，这些表提供有关已仪器化对象的其他信息。与 `InnoDB` 相关的实例表包括：

    +   `mutex_instances` 表

    +   `rwlock_instances` 表

    +   `cond_instances` 表

    +   `file_instances` 表

    注意

    与`InnoDB`缓冲池相关的互斥锁和读写锁不包括在此范围内；相同的情况也适用于`SHOW ENGINE INNODB MUTEX`命令的输出。

    例如，要查看执行文件 I/O 仪表化时性能模式看到的`InnoDB`文件对象的信息，您可以发出以下查询：

    ```sql
    mysql> SELECT *
           FROM performance_schema.file_instances
           WHERE EVENT_NAME LIKE '%innodb%'\G
    *************************** 1\. row ***************************
     FILE_NAME: /home/dtprice/mysql-8.0/data/ibdata1
    EVENT_NAME: wait/io/file/innodb/innodb_data_file
    OPEN_COUNT: 3
    *************************** 2\. row ***************************
     FILE_NAME: /home/dtprice/mysql-8.0/data/#ib_16384_0.dblwr
    EVENT_NAME: wait/io/file/innodb/innodb_dblwr_file
    OPEN_COUNT: 2
    *************************** 3\. row ***************************
     FILE_NAME: /home/dtprice/mysql-8.0/data/#ib_16384_1.dblwr
    EVENT_NAME: wait/io/file/mysql-8.0/innodb_dblwr_file
    OPEN_COUNT: 2
    ...
    ```

+   您应该熟悉存储`InnoDB`事件数据的`performance_schema`表。与`InnoDB`相关事件相关的表包括：

    +   等待事件 表，存储等待事件。

    +   摘要 表，为随时间终止的事件提供聚合信息。摘要表包括 file I/O 摘要表，其中聚合了有关 I/O 操作的信息。

    +   阶段事件 表，存储了`InnoDB`的`ALTER TABLE`和缓冲池加载操作的事件数据。更多信息，请参阅 17.16.1 节，“使用性能模式监控 InnoDB 表的 ALTER TABLE 进度”，以及使用性能模式监控缓冲池加载进度。

    如果您只对与`InnoDB`相关的对象感兴趣，在查询这些表时，请使用子句`WHERE EVENT_NAME LIKE '%innodb%'`或`WHERE NAME LIKE '%innodb%'`（根据需要）。
