> 原文：[`dev.mysql.com/doc/refman/8.0/en/monitor-mysql-memory-use.html`](https://dev.mysql.com/doc/refman/8.0/en/monitor-mysql-memory-use.html)

#### 10.12.3.2 监控 MySQL 内存使用情况

以下示例演示了如何使用性能模式和 sys 模式来监控 MySQL 内存使用情况。

大多数性能模式内存工具默认处于禁用状态。可以通过更新性能模式`setup_instruments`表的`ENABLED`列来启用工具。内存工具的名称形式为`memory/*`code_area`*/*`instrument_name`*`，其中*`code_area`*是诸如`sql`或`innodb`之类的值，*`instrument_name`*是工具的详细信息。

1.  要查看可用的 MySQL 内存工具，请查询性能模式`setup_instruments`表。以下查询返回所有代码区域的数百个内存工具。

    ```sql
    mysql> SELECT * FROM performance_schema.setup_instruments
           WHERE NAME LIKE '%memory%';
    ```

    通过指定代码区域，您可以缩小结果范围。例如，您可以通过指定`innodb`作为代码区域来限制结果为`InnoDB`内存工具。

    ```sql
    mysql> SELECT * FROM performance_schema.setup_instruments
           WHERE NAME LIKE '%memory/innodb%';
    +-------------------------------------------+---------+-------+
    | NAME                                      | ENABLED | TIMED |
    +-------------------------------------------+---------+-------+
    | memory/innodb/adaptive hash index         | NO      | NO    |
    | memory/innodb/buf_buf_pool                | NO      | NO    |
    | memory/innodb/dict_stats_bg_recalc_pool_t | NO      | NO    |
    | memory/innodb/dict_stats_index_map_t      | NO      | NO    |
    | memory/innodb/dict_stats_n_diff_on_level  | NO      | NO    |
    | memory/innodb/other                       | NO      | NO    |
    | memory/innodb/row_log_buf                 | NO      | NO    |
    | memory/innodb/row_merge_sort              | NO      | NO    |
    | memory/innodb/std                         | NO      | NO    |
    | memory/innodb/trx_sys_t::rw_trx_ids       | NO      | NO    |
    ...
    ```

    根据您的 MySQL 安装，代码区域可能包括`performance_schema`、`sql`、`client`、`innodb`、`myisam`、`csv`、`memory`、`blackhole`、`archive`、`partition`等。

1.  要启用内存工具，请向您的 MySQL 配置文件添加`performance-schema-instrument`规则。例如，要启用所有内存工具，请将此规则添加到您的配置文件中并重新启动服务器：

    ```sql
    performance-schema-instrument='memory/%=COUNTED'
    ```

    注意

    在启动时启用内存工具可确保计算启动时发生的内存分配。

    重新启动服务器后，性能模式`setup_instruments`表的`ENABLED`列应报告您启用的内存工具为`YES`。性能模式`setup_instruments`表中的`TIMED`列对于内存工具是被忽略的，因为内存操作不计时。

    ```sql
    mysql> SELECT * FROM performance_schema.setup_instruments
           WHERE NAME LIKE '%memory/innodb%';
    +-------------------------------------------+---------+-------+
    | NAME                                      | ENABLED | TIMED |
    +-------------------------------------------+---------+-------+
    | memory/innodb/adaptive hash index         | NO      | NO    |
    | memory/innodb/buf_buf_pool                | NO      | NO    |
    | memory/innodb/dict_stats_bg_recalc_pool_t | NO      | NO    |
    | memory/innodb/dict_stats_index_map_t      | NO      | NO    |
    | memory/innodb/dict_stats_n_diff_on_level  | NO      | NO    |
    | memory/innodb/other                       | NO      | NO    |
    | memory/innodb/row_log_buf                 | NO      | NO    |
    | memory/innodb/row_merge_sort              | NO      | NO    |
    | memory/innodb/std                         | NO      | NO    |
    | memory/innodb/trx_sys_t::rw_trx_ids       | NO      | NO    |
    ...
    ```

1.  查询内存工具数据。在此示例中，内存工具数据在性能模式`memory_summary_global_by_event_name`表中查询，该表按`EVENT_NAME`汇总数据。`EVENT_NAME`是工具的名称。

    以下查询返回`InnoDB`缓冲池的内存数据。有关列描述，请参阅第 29.12.20.10 节，“内存摘要表”。

    ```sql
    mysql> SELECT * FROM performance_schema.memory_summary_global_by_event_name
           WHERE EVENT_NAME LIKE 'memory/innodb/buf_buf_pool'\G
                      EVENT_NAME: memory/innodb/buf_buf_pool
                     COUNT_ALLOC: 1
                      COUNT_FREE: 0
       SUM_NUMBER_OF_BYTES_ALLOC: 137428992
        SUM_NUMBER_OF_BYTES_FREE: 0
                  LOW_COUNT_USED: 0
              CURRENT_COUNT_USED: 1
                 HIGH_COUNT_USED: 1
        LOW_NUMBER_OF_BYTES_USED: 0
    CURRENT_NUMBER_OF_BYTES_USED: 137428992
       HIGH_NUMBER_OF_BYTES_USED: 137428992
    ```

    可以使用 `sys` schema 的 `memory_global_by_current_bytes` 表查询相同的基础数据，该表显示了服务器全局范围内当前内存使用情况，按分配类型进行了拆分。

    ```sql
    mysql> SELECT * FROM sys.memory_global_by_current_bytes
           WHERE event_name LIKE 'memory/innodb/buf_buf_pool'\G
    *************************** 1\. row ***************************
           event_name: memory/innodb/buf_buf_pool
        current_count: 1
        current_alloc: 131.06 MiB
    current_avg_alloc: 131.06 MiB
           high_count: 1
           high_alloc: 131.06 MiB
       high_avg_alloc: 131.06 MiB
    ```

    此 `sys` schema 查询通过代码区域对当前分配的内存（`current_alloc`）进行聚合：

    ```sql
    mysql> SELECT SUBSTRING_INDEX(event_name,'/',2) AS
           code_area, FORMAT_BYTES(SUM(current_alloc))
           AS current_alloc
           FROM sys.x$memory_global_by_current_bytes
           GROUP BY SUBSTRING_INDEX(event_name,'/',2)
           ORDER BY SUM(current_alloc) DESC;
    +---------------------------+---------------+
    | code_area                 | current_alloc |
    +---------------------------+---------------+
    | memory/innodb             | 843.24 MiB    |
    | memory/performance_schema | 81.29 MiB     |
    | memory/mysys              | 8.20 MiB      |
    | memory/sql                | 2.47 MiB      |
    | memory/memory             | 174.01 KiB    |
    | memory/myisam             | 46.53 KiB     |
    | memory/blackhole          | 512 bytes     |
    | memory/federated          | 512 bytes     |
    | memory/csv                | 512 bytes     |
    | memory/vio                | 496 bytes     |
    +---------------------------+---------------+
    ```

    注意

    在 MySQL 8.0.16 之前，使用 `sys.format_bytes()` 来进行 `FORMAT_BYTES()`。

    有关 `sys` schema 的更多信息，请参阅 第三十章，*MySQL sys Schema*。
