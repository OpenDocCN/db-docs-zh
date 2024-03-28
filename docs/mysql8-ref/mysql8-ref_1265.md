# 17.16.2 使用性能模式监视 InnoDB 互斥锁等待

> 原文：[`dev.mysql.com/doc/refman/8.0/en/monitor-innodb-mutex-waits-performance-schema.html`](https://dev.mysql.com/doc/refman/8.0/en/monitor-innodb-mutex-waits-performance-schema.html)

互斥锁是代码中使用的一种同步机制，用于强制只有一个线程可以同时访问共享资源。当服务器中的两个或多个线程需要访问同一资源时，这些线程会相互竞争。第一个获得互斥锁的线程会导致其他线程等待，直到锁被释放。

对于被检测的`InnoDB`互斥锁，可以使用性能模式来监视互斥锁等待。在性能模式表中收集的等待事件数据可以帮助识别具有最多等待或最长总等待时间的互斥锁，例如。

以下示例演示了如何启用`InnoDB`互斥锁等待工具，如何启用相关的消费者，以及如何查询等待事件数据。

1.  要查看可用的`InnoDB`互斥锁等待工具，查询性能模式`setup_instruments`表。所有`InnoDB`互斥锁等待工具默认情况下是禁用的。

    ```sql
    mysql> SELECT *
           FROM performance_schema.setup_instruments
           WHERE NAME LIKE '%wait/synch/mutex/innodb%';
    +---------------------------------------------------------+---------+-------+
    | NAME                                                    | ENABLED | TIMED |
    +---------------------------------------------------------+---------+-------+
    | wait/synch/mutex/innodb/commit_cond_mutex               | NO      | NO    |
    | wait/synch/mutex/innodb/innobase_share_mutex            | NO      | NO    |
    | wait/synch/mutex/innodb/autoinc_mutex                   | NO      | NO    |
    | wait/synch/mutex/innodb/autoinc_persisted_mutex         | NO      | NO    |
    | wait/synch/mutex/innodb/buf_pool_flush_state_mutex      | NO      | NO    |
    | wait/synch/mutex/innodb/buf_pool_LRU_list_mutex         | NO      | NO    |
    | wait/synch/mutex/innodb/buf_pool_free_list_mutex        | NO      | NO    |
    | wait/synch/mutex/innodb/buf_pool_zip_free_mutex         | NO      | NO    |
    | wait/synch/mutex/innodb/buf_pool_zip_hash_mutex         | NO      | NO    |
    | wait/synch/mutex/innodb/buf_pool_zip_mutex              | NO      | NO    |
    | wait/synch/mutex/innodb/cache_last_read_mutex           | NO      | NO    |
    | wait/synch/mutex/innodb/dict_foreign_err_mutex          | NO      | NO    |
    | wait/synch/mutex/innodb/dict_persist_dirty_tables_mutex | NO      | NO    |
    | wait/synch/mutex/innodb/dict_sys_mutex                  | NO      | NO    |
    | wait/synch/mutex/innodb/recalc_pool_mutex               | NO      | NO    |
    | wait/synch/mutex/innodb/fil_system_mutex                | NO      | NO    |
    | wait/synch/mutex/innodb/flush_list_mutex                | NO      | NO    |
    | wait/synch/mutex/innodb/fts_bg_threads_mutex            | NO      | NO    |
    | wait/synch/mutex/innodb/fts_delete_mutex                | NO      | NO    |
    | wait/synch/mutex/innodb/fts_optimize_mutex              | NO      | NO    |
    | wait/synch/mutex/innodb/fts_doc_id_mutex                | NO      | NO    |
    | wait/synch/mutex/innodb/log_flush_order_mutex           | NO      | NO    |
    | wait/synch/mutex/innodb/hash_table_mutex                | NO      | NO    |
    | wait/synch/mutex/innodb/ibuf_bitmap_mutex               | NO      | NO    |
    | wait/synch/mutex/innodb/ibuf_mutex                      | NO      | NO    |
    | wait/synch/mutex/innodb/ibuf_pessimistic_insert_mutex   | NO      | NO    |
    | wait/synch/mutex/innodb/log_sys_mutex                   | NO      | NO    |
    | wait/synch/mutex/innodb/log_sys_write_mutex             | NO      | NO    |
    | wait/synch/mutex/innodb/mutex_list_mutex                | NO      | NO    |
    | wait/synch/mutex/innodb/page_zip_stat_per_index_mutex   | NO      | NO    |
    | wait/synch/mutex/innodb/purge_sys_pq_mutex              | NO      | NO    |
    | wait/synch/mutex/innodb/recv_sys_mutex                  | NO      | NO    |
    | wait/synch/mutex/innodb/recv_writer_mutex               | NO      | NO    |
    | wait/synch/mutex/innodb/redo_rseg_mutex                 | NO      | NO    |
    | wait/synch/mutex/innodb/noredo_rseg_mutex               | NO      | NO    |
    | wait/synch/mutex/innodb/rw_lock_list_mutex              | NO      | NO    |
    | wait/synch/mutex/innodb/rw_lock_mutex                   | NO      | NO    |
    | wait/synch/mutex/innodb/srv_dict_tmpfile_mutex          | NO      | NO    |
    | wait/synch/mutex/innodb/srv_innodb_monitor_mutex        | NO      | NO    |
    | wait/synch/mutex/innodb/srv_misc_tmpfile_mutex          | NO      | NO    |
    | wait/synch/mutex/innodb/srv_monitor_file_mutex          | NO      | NO    |
    | wait/synch/mutex/innodb/buf_dblwr_mutex                 | NO      | NO    |
    | wait/synch/mutex/innodb/trx_undo_mutex                  | NO      | NO    |
    | wait/synch/mutex/innodb/trx_pool_mutex                  | NO      | NO    |
    | wait/synch/mutex/innodb/trx_pool_manager_mutex          | NO      | NO    |
    | wait/synch/mutex/innodb/srv_sys_mutex                   | NO      | NO    |
    | wait/synch/mutex/innodb/lock_mutex                      | NO      | NO    |
    | wait/synch/mutex/innodb/lock_wait_mutex                 | NO      | NO    |
    | wait/synch/mutex/innodb/trx_mutex                       | NO      | NO    |
    | wait/synch/mutex/innodb/srv_threads_mutex               | NO      | NO    |
    | wait/synch/mutex/innodb/rtr_active_mutex                | NO      | NO    |
    | wait/synch/mutex/innodb/rtr_match_mutex                 | NO      | NO    |
    | wait/synch/mutex/innodb/rtr_path_mutex                  | NO      | NO    |
    | wait/synch/mutex/innodb/rtr_ssn_mutex                   | NO      | NO    |
    | wait/synch/mutex/innodb/trx_sys_mutex                   | NO      | NO    |
    | wait/synch/mutex/innodb/zip_pad_mutex                   | NO      | NO    |
    | wait/synch/mutex/innodb/master_key_id_mutex             | NO      | NO    |
    +---------------------------------------------------------+---------+-------+
    ```

1.  一些`InnoDB`互斥锁实例是在服务器启动时创建的，只有在服务器启动时还启用了相关的工具时才会被检测。为了确保所有`InnoDB`互斥锁实例都被检测和启用，将以下`performance-schema-instrument`规则添加到您的 MySQL 配置文件中：

    ```sql
    performance-schema-instrument='wait/synch/mutex/innodb/%=ON'
    ```

    如果您不需要所有`InnoDB`互斥锁的等待事件数据，可以通过向 MySQL 配置文件添加额外的`performance-schema-instrument`规则来禁用特定工具。例如，要禁用与全文搜索相关的`InnoDB`互斥锁等待事件工具，添加以下规则：

    ```sql
    performance-schema-instrument='wait/synch/mutex/innodb/fts%=OFF'
    ```

    注意

    前缀较长的规则（例如`wait/synch/mutex/innodb/fts%`）优先于前缀较短的规则（例如`wait/synch/mutex/innodb/%`）。

    在将`performance-schema-instrument`规则添加到配置文件后，重新启动服务器。除了与全文搜索相关的`InnoDB`互斥锁之外，所有互斥锁都已启用。要验证，请查询`setup_instruments`表。对于您启用的工具，`ENABLED`和`TIMED`列应设置为`YES`。

    ```sql
    mysql> SELECT *
           FROM performance_schema.setup_instruments
           WHERE NAME LIKE '%wait/synch/mutex/innodb%';
    +-------------------------------------------------------+---------+-------+
    | NAME                                                  | ENABLED | TIMED |
    +-------------------------------------------------------+---------+-------+
    | wait/synch/mutex/innodb/commit_cond_mutex             | YES     | YES   |
    | wait/synch/mutex/innodb/innobase_share_mutex          | YES     | YES   |
    | wait/synch/mutex/innodb/autoinc_mutex                 | YES     | YES   |
    ...
    | wait/synch/mutex/innodb/master_key_id_mutex           | YES     | YES   |
    +-------------------------------------------------------+---------+-------+
    49 rows in set (0.00 sec)
    ```

1.  通过更新`setup_consumers`表来启用等待事件消费者。等待事件消费者默认情况下是禁用的。

    ```sql
    mysql> UPDATE performance_schema.setup_consumers
           SET enabled = 'YES'
           WHERE name like 'events_waits%';
    Query OK, 3 rows affected (0.00 sec)
    Rows matched: 3  Changed: 3  Warnings: 0
    ```

    您可以通过查询`setup_consumers`表来验证等待事件消费者是否已启用。`events_waits_current`、`events_waits_history`和`events_waits_history_long`消费者应该已启用。

    ```sql
    mysql> SELECT * FROM performance_schema.setup_consumers;
    +----------------------------------+---------+
    | NAME                             | ENABLED |
    +----------------------------------+---------+
    | events_stages_current            | NO      |
    | events_stages_history            | NO      |
    | events_stages_history_long       | NO      |
    | events_statements_current        | YES     |
    | events_statements_history        | YES     |
    | events_statements_history_long   | NO      |
    | events_transactions_current      | YES     |
    | events_transactions_history      | YES     |
    | events_transactions_history_long | NO      |
    | events_waits_current             | YES     |
    | events_waits_history             | YES     |
    | events_waits_history_long        | YES     |
    | global_instrumentation           | YES     |
    | thread_instrumentation           | YES     |
    | statements_digest                | YES     |
    +----------------------------------+---------+
    15 rows in set (0.00 sec)
    ```

1.  一旦仪器和消费者已启用，请运行您想要监视的工作负载。在本示例中，使用**mysqlslap**负载仿真客户端来模拟工作负载。

    ```sql
    $> ./mysqlslap --auto-generate-sql --concurrency=100 --iterations=10 
           --number-of-queries=1000 --number-char-cols=6 --number-int-cols=6;
    ```

1.  查询等待事件数据。在本示例中，等待事件数据是从`events_waits_summary_global_by_event_name`表中查询的，该表汇总了`events_waits_current`、`events_waits_history`和`events_waits_history_long`表中的数据。数据按事件名称（`EVENT_NAME`）进行汇总，这是产生事件的仪器的名称。汇总的数据包括：

    +   `COUNT_STAR`

        总结的等待事件数量。

    +   `SUM_TIMER_WAIT`

        总结的定时等待事件的总等待时间。

    +   `MIN_TIMER_WAIT`

        总结的定时等待事件的最小等待时间。

    +   `AVG_TIMER_WAIT`

        总结的定时等待事件的平均等待时间。

    +   `MAX_TIMER_WAIT`

        总结的定时等待事件的最大等待时间。

    以下查询返回仪器名称（`EVENT_NAME`）、等待事件数量（`COUNT_STAR`）以及该仪器事件的总等待时间（`SUM_TIMER_WAIT`）。由于默认情况下等待时间以皮秒（万亿分之一秒）计时，等待时间除以 1000000000 以显示以毫秒为单位的等待时间。数据按照总结的等待事件数量（`COUNT_STAR`）降序呈现。您可以调整`ORDER BY`子句以按照总等待时间对数据进行排序。

    ```sql
    mysql> SELECT EVENT_NAME, COUNT_STAR, SUM_TIMER_WAIT/1000000000 SUM_TIMER_WAIT_MS
           FROM performance_schema.events_waits_summary_global_by_event_name
           WHERE SUM_TIMER_WAIT > 0 AND EVENT_NAME LIKE 'wait/synch/mutex/innodb/%'
           ORDER BY COUNT_STAR DESC;
    +---------------------------------------------------------+------------+-------------------+
    | EVENT_NAME                                              | COUNT_STAR | SUM_TIMER_WAIT_MS |
    +---------------------------------------------------------+------------+-------------------+
    | wait/synch/mutex/innodb/trx_mutex                       |     201111 |           23.4719 |
    | wait/synch/mutex/innodb/fil_system_mutex                |      62244 |            9.6426 |
    | wait/synch/mutex/innodb/redo_rseg_mutex                 |      48238 |            3.1135 |
    | wait/synch/mutex/innodb/log_sys_mutex                   |      46113 |            2.0434 |
    | wait/synch/mutex/innodb/trx_sys_mutex                   |      35134 |         1068.1588 |
    | wait/synch/mutex/innodb/lock_mutex                      |      34872 |         1039.2589 |
    | wait/synch/mutex/innodb/log_sys_write_mutex             |      17805 |         1526.0490 |
    | wait/synch/mutex/innodb/dict_sys_mutex                  |      14912 |         1606.7348 |
    | wait/synch/mutex/innodb/trx_undo_mutex                  |      10634 |            1.1424 |
    | wait/synch/mutex/innodb/rw_lock_list_mutex              |       8538 |            0.1960 |
    | wait/synch/mutex/innodb/buf_pool_free_list_mutex        |       5961 |            0.6473 |
    | wait/synch/mutex/innodb/trx_pool_mutex                  |       4885 |         8821.7496 |
    | wait/synch/mutex/innodb/buf_pool_LRU_list_mutex         |       4364 |            0.2077 |
    | wait/synch/mutex/innodb/innobase_share_mutex            |       3212 |            0.2650 |
    | wait/synch/mutex/innodb/flush_list_mutex                |       3178 |            0.2349 |
    | wait/synch/mutex/innodb/trx_pool_manager_mutex          |       2495 |            0.1310 |
    | wait/synch/mutex/innodb/buf_pool_flush_state_mutex      |       1318 |            0.2161 |
    | wait/synch/mutex/innodb/log_flush_order_mutex           |       1250 |            0.0893 |
    | wait/synch/mutex/innodb/buf_dblwr_mutex                 |        951 |            0.0918 |
    | wait/synch/mutex/innodb/recalc_pool_mutex               |        670 |            0.0942 |
    | wait/synch/mutex/innodb/dict_persist_dirty_tables_mutex |        345 |            0.0414 |
    | wait/synch/mutex/innodb/lock_wait_mutex                 |        303 |            0.1565 |
    | wait/synch/mutex/innodb/autoinc_mutex                   |        196 |            0.0213 |
    | wait/synch/mutex/innodb/autoinc_persisted_mutex         |        196 |            0.0175 |
    | wait/synch/mutex/innodb/purge_sys_pq_mutex              |        117 |            0.0308 |
    | wait/synch/mutex/innodb/srv_sys_mutex                   |         94 |            0.0077 |
    | wait/synch/mutex/innodb/ibuf_mutex                      |         22 |            0.0086 |
    | wait/synch/mutex/innodb/recv_sys_mutex                  |         12 |            0.0008 |
    | wait/synch/mutex/innodb/srv_innodb_monitor_mutex        |          4 |            0.0009 |
    | wait/synch/mutex/innodb/recv_writer_mutex               |          1 |            0.0005 |
    +---------------------------------------------------------+------------+-------------------+
    ```

    注意

    前面的结果集包括在启动过程中产生的等待事件数据。为了排除这些数据，您可以在启动后立即截断`events_waits_summary_global_by_event_name`表，然后再运行您的工作负载。但是，��断操作本身可能会产生微不足道的等待事件数据。

    ```sql
    mysql> TRUNCATE performance_schema.events_waits_summary_global_by_event_name;
    ```
