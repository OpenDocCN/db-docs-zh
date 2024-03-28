# 25.6.19 快速参考：NDB 集群 SQL 语句

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-sql-statements.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-sql-statements.html)

本节讨论了几个 SQL 语句，这些语句在管理和监控连接到 NDB 集群的 MySQL 服务器时可能会有用，并在某些情况下提供有关集群本身的信息。

+   `SHOW ENGINE NDB STATUS`，`SHOW ENGINE NDBCLUSTER STATUS`

    此语句的输出包含有关服务器与集群的连接、NDB 集群对象的创建和使用，以及 NDB 集群复制的二进制日志记录的信息。

    有关用法示例和更详细信息，请参见第 15.7.7.15 节，“SHOW ENGINE 语句”。

+   `SHOW ENGINES`

    此语句可用于确定 MySQL 服务器中是否启用了集群支持，以及如果是，则是否处于活动状态。

    有关更详细信息，请参见第 15.7.7.16 节，“SHOW ENGINES 语句”。

    注意

    此语句不支持`LIKE`子句。但是，您可以使用`LIKE`来过滤针对 Information Schema `ENGINES`表的查询，如下一项所述。

+   `SELECT * FROM INFORMATION_SCHEMA.ENGINES [WHERE ENGINE LIKE 'NDB%']`

    这相当于`SHOW ENGINES`，但使用`INFORMATION_SCHEMA`数据库的`ENGINES`表。与`SHOW ENGINES`语句不同，可以使用`LIKE`子句来过滤结果，并选择特定列以获取在脚本中可能有用的信息。例如，以下查询显示服务器是否构建有`NDB`支持，并且如果是，则是否已启用：

    ```sql
    mysql> SELECT ENGINE, SUPPORT FROM INFORMATION_SCHEMA.ENGINES
     ->   WHERE ENGINE LIKE 'NDB%';
    +------------+---------+
    | ENGINE     | SUPPORT |
    +------------+---------+
    | ndbcluster | YES     |
    | ndbinfo    | YES     |
    +------------+---------+
    ```

    如果未启用`NDB`支持，则上述查询将返回一个空集。有关更多信息，请参见第 28.3.13 节，“INFORMATION_SCHEMA ENGINES 表”。

+   `SHOW VARIABLES LIKE 'NDB%'`

    此语句提供了与`NDB`存储引擎相关的大多数服务器系统变量的列表及其值，如下所示：

    ```sql
    mysql> SHOW VARIABLES LIKE 'NDB%';
    +--------------------------------------+---------------------------------------+
    | Variable_name                        | Value                                 |
    +--------------------------------------+---------------------------------------+
    | ndb_allow_copying_alter_table        | ON                                    |
    | ndb_autoincrement_prefetch_sz        | 512                                   |
    | ndb_batch_size                       | 32768                                 |
    | ndb_blob_read_batch_bytes            | 65536                                 |
    | ndb_blob_write_batch_bytes           | 65536                                 |
    | ndb_clear_apply_status               | ON                                    |
    | ndb_cluster_connection_pool          | 1                                     |
    | ndb_cluster_connection_pool_nodeids  |                                       |
    | ndb_connectstring                    | 127.0.0.1                             |
    | ndb_data_node_neighbour              | 0                                     |
    | ndb_default_column_format            | FIXED                                 |
    | ndb_deferred_constraints             | 0                                     |
    | ndb_distribution                     | KEYHASH                               |
    | ndb_eventbuffer_free_percent         | 20                                    |
    | ndb_eventbuffer_max_alloc            | 0                                     |
    | ndb_extra_logging                    | 1                                     |
    | ndb_force_send                       | ON                                    |
    | ndb_fully_replicated                 | OFF                                   |
    | ndb_index_stat_enable                | ON                                    |
    | ndb_index_stat_option                | loop_enable=1000ms,loop_idle=1000ms,
    loop_busy=100ms,update_batch=1,read_batch=4,idle_batch=32,check_batch=8,
    check_delay=10m,delete_batch=8,clean_delay=1m,error_batch=4,error_delay=1m,
    evict_batch=8,evict_delay=1m,cache_limit=32M,cache_lowpct=90,zero_total=0      |
    | ndb_join_pushdown                    | ON                                    |
    | ndb_log_apply_status                 | OFF                                   |
    | ndb_log_bin                          | OFF                                   |
    | ndb_log_binlog_index                 | ON                                    |
    | ndb_log_empty_epochs                 | OFF                                   |
    | ndb_log_empty_update                 | OFF                                   |
    | ndb_log_exclusive_reads              | OFF                                   |
    | ndb_log_orig                         | OFF                                   |
    | ndb_log_transaction_id               | OFF                                   |
    | ndb_log_update_as_write              | ON                                    |
    | ndb_log_update_minimal               | OFF                                   |
    | ndb_log_updated_only                 | ON                                    |
    | ndb_metadata_check                   | ON                                    |
    | ndb_metadata_check_interval          | 60                                    |
    | ndb_metadata_sync                    | OFF                                   |
    | ndb_mgmd_host                        | 127.0.0.1                             |
    | ndb_nodeid                           | 0                                     |
    | ndb_optimization_delay               | 10                                    |
    | ndb_optimized_node_selection         | 3                                     |
    | ndb_read_backup                      | ON                                    |
    | ndb_recv_thread_activation_threshold | 8                                     |
    | ndb_recv_thread_cpu_mask             |                                       |
    | ndb_report_thresh_binlog_epoch_slip  | 10                                    |
    | ndb_report_thresh_binlog_mem_usage   | 10                                    |
    | ndb_row_checksum                     | 1                                     |
    | ndb_schema_dist_lock_wait_timeout    | 30                                    |
    | ndb_schema_dist_timeout              | 120                                   |
    | ndb_schema_dist_upgrade_allowed      | ON                                    |
    | ndb_show_foreign_key_mock_tables     | OFF                                   |
    | ndb_slave_conflict_role              | NONE                                  |
    | ndb_table_no_logging                 | OFF                                   |
    | ndb_table_temporary                  | OFF                                   |
    | ndb_use_copying_alter_table          | OFF                                   |
    | ndb_use_exact_count                  | OFF                                   |
    | ndb_use_transactions                 | ON                                    |
    | ndb_version                          | 524308                                |
    | ndb_version_string                   | ndb-8.0.35                            |
    | ndb_wait_connected                   | 30                                    |
    | ndb_wait_setup                       | 30                                    |
    | ndbinfo_database                     | ndbinfo                               |
    | ndbinfo_max_bytes                    | 0                                     |
    | ndbinfo_max_rows                     | 10                                    |
    | ndbinfo_offline                      | OFF                                   |
    | ndbinfo_show_hidden                  | OFF                                   |
    | ndbinfo_table_prefix                 | ndb$                                  |
    | ndbinfo_version                      | 524308                                |
    +--------------------------------------+---------------------------------------+
    ```

    查看更多信息，请参见第 7.1.8 节，“服务器系统变量”。

+   `SELECT * FROM performance_schema.global_variables WHERE VARIABLE_NAME LIKE 'NDB%'`

    这个语句相当于之前描述的`SHOW VARIABLES`语句，并提供几乎相同的输出，如下所示：

    ```sql
    mysql> SELECT * FROM performance_schema.global_variables
     ->   WHERE VARIABLE_NAME LIKE 'NDB%';
    +--------------------------------------+---------------------------------------+
    | VARIABLE_NAME                        | VARIABLE_VALUE                        |
    +--------------------------------------+---------------------------------------+
    | ndb_allow_copying_alter_table        | ON                                    |
    | ndb_autoincrement_prefetch_sz        | 512                                   |
    | ndb_batch_size                       | 32768                                 |
    | ndb_blob_read_batch_bytes            | 65536                                 |
    | ndb_blob_write_batch_bytes           | 65536                                 |
    | ndb_clear_apply_status               | ON                                    |
    | ndb_cluster_connection_pool          | 1                                     |
    | ndb_cluster_connection_pool_nodeids  |                                       |
    | ndb_connectstring                    | 127.0.0.1                             |
    | ndb_data_node_neighbour              | 0                                     |
    | ndb_default_column_format            | FIXED                                 |
    | ndb_deferred_constraints             | 0                                     |
    | ndb_distribution                     | KEYHASH                               |
    | ndb_eventbuffer_free_percent         | 20                                    |
    | ndb_eventbuffer_max_alloc            | 0                                     |
    | ndb_extra_logging                    | 1                                     |
    | ndb_force_send                       | ON                                    |
    | ndb_fully_replicated                 | OFF                                   |
    | ndb_index_stat_enable                | ON                                    |
    | ndb_index_stat_option                | loop_enable=1000ms,loop_idle=1000ms,
    loop_busy=100ms,update_batch=1,read_batch=4,idle_batch=32,check_batch=8,
    check_delay=10m,delete_batch=8,clean_delay=1m,error_batch=4,error_delay=1m,
    evict_batch=8,evict_delay=1m,cache_limit=32M,cache_lowpct=90,zero_total=0      |
    | ndb_join_pushdown                    | ON                                    |
    | ndb_log_apply_status                 | OFF                                   |
    | ndb_log_bin                          | OFF                                   |
    | ndb_log_binlog_index                 | ON                                    |
    | ndb_log_empty_epochs                 | OFF                                   |
    | ndb_log_empty_update                 | OFF                                   |
    | ndb_log_exclusive_reads              | OFF                                   |
    | ndb_log_orig                         | OFF                                   |
    | ndb_log_transaction_id               | OFF                                   |
    | ndb_log_update_as_write              | ON                                    |
    | ndb_log_update_minimal               | OFF                                   |
    | ndb_log_updated_only                 | ON                                    |
    | ndb_metadata_check                   | ON                                    |
    | ndb_metadata_check_interval          | 60                                    |
    | ndb_metadata_sync                    | OFF                                   |
    | ndb_mgmd_host                        | 127.0.0.1                             |
    | ndb_nodeid                           | 0                                     |
    | ndb_optimization_delay               | 10                                    |
    | ndb_optimized_node_selection         | 3                                     |
    | ndb_read_backup                      | ON                                    |
    | ndb_recv_thread_activation_threshold | 8                                     |
    | ndb_recv_thread_cpu_mask             |                                       |
    | ndb_report_thresh_binlog_epoch_slip  | 10                                    |
    | ndb_report_thresh_binlog_mem_usage   | 10                                    |
    | ndb_row_checksum                     | 1                                     |
    | ndb_schema_dist_lock_wait_timeout    | 30                                    |
    | ndb_schema_dist_timeout              | 120                                   |
    | ndb_schema_dist_upgrade_allowed      | ON                                    |
    | ndb_show_foreign_key_mock_tables     | OFF                                   |
    | ndb_slave_conflict_role              | NONE                                  |
    | ndb_table_no_logging                 | OFF                                   |
    | ndb_table_temporary                  | OFF                                   |
    | ndb_use_copying_alter_table          | OFF                                   |
    | ndb_use_exact_count                  | OFF                                   |
    | ndb_use_transactions                 | ON                                    |
    | ndb_version                          | 524308                                |
    | ndb_version_string                   | ndb-8.0.35                            |
    | ndb_wait_connected                   | 30                                    |
    | ndb_wait_setup                       | 30                                    |
    | ndbinfo_database                     | ndbinfo                               |
    | ndbinfo_max_bytes                    | 0                                     |
    | ndbinfo_max_rows                     | 10                                    |
    | ndbinfo_offline                      | OFF                                   |
    | ndbinfo_show_hidden                  | OFF                                   |
    | ndbinfo_table_prefix                 | ndb$                                  |
    | ndbinfo_version                      | 524308                                |
    +--------------------------------------+---------------------------------------+
    ```

    与`SHOW VARIABLES`语句不同，可以选择单独的列。例如：

    ```sql
    mysql> SELECT VARIABLE_VALUE 
     ->   FROM performance_schema.global_variables
     ->   WHERE VARIABLE_NAME = 'ndb_force_send';
    +----------------+
    | VARIABLE_VALUE |
    +----------------+
    | ON             |
    +----------------+
    ```

    一个更有用的查询如下所示：

    ```sql
    mysql> SELECT VARIABLE_NAME AS Name, VARIABLE_VALUE AS Value
         >   FROM performance_schema.global_variables
         >   WHERE VARIABLE_NAME
         >     IN ('version', 'ndb_version',
         >       'ndb_version_string', 'ndbinfo_version');

    +--------------------+----------------+
    | Name               | Value          |
    +--------------------+----------------+
    | ndb_version        | 524317         |
    | ndb_version_string | ndb-8.0.29     |
    | ndbinfo_version    | 524317         |
    | version            | 8.0.29-cluster |
    +--------------------+----------------+
    4 rows in set (0.00 sec)
    ```

    更多信息，请参见第 29.12.15 节，“性能模式状态变量表”，以及第 7.1.8 节，“服务器系统变量”。

+   `SHOW STATUS LIKE 'NDB%'`

    这个语句一目了然地显示了 MySQL 服务器是否作为集群 SQL 节点运行，如果是的话，它提供了 MySQL 服务器的集群节点 ID、连接的集群管理服务器的主机名和端口，以及集群中的数据节点数量，如下所示：

    ```sql
    mysql> SHOW STATUS LIKE 'NDB%';
    +----------------------------------------------+-------------------------------+
    | Variable_name                                | Value                         |
    +----------------------------------------------+-------------------------------+
    | Ndb_metadata_detected_count                  | 0                             |
    | Ndb_cluster_node_id                          | 100                           |
    | Ndb_config_from_host                         | 127.0.0.1                     |
    | Ndb_config_from_port                         | 1186                          |
    | Ndb_number_of_data_nodes                     | 2                             |
    | Ndb_number_of_ready_data_nodes               | 2                             |
    | Ndb_connect_count                            | 0                             |
    | Ndb_execute_count                            | 0                             |
    | Ndb_scan_count                               | 0                             |
    | Ndb_pruned_scan_count                        | 0                             |
    | Ndb_schema_locks_count                       | 0                             |
    | Ndb_api_wait_exec_complete_count_session     | 0                             |
    | Ndb_api_wait_scan_result_count_session       | 0                             |
    | Ndb_api_wait_meta_request_count_session      | 1                             |
    | Ndb_api_wait_nanos_count_session             | 163446                        |
    | Ndb_api_bytes_sent_count_session             | 60                            |
    | Ndb_api_bytes_received_count_session         | 28                            |
    | Ndb_api_trans_start_count_session            | 0                             |
    | Ndb_api_trans_commit_count_session           | 0                             |
    | Ndb_api_trans_abort_count_session            | 0                             |
    | Ndb_api_trans_close_count_session            | 0                             |
    | Ndb_api_pk_op_count_session                  | 0                             |
    | Ndb_api_uk_op_count_session                  | 0                             |
    | Ndb_api_table_scan_count_session             | 0                             |
    | Ndb_api_range_scan_count_session             | 0                             |
    | Ndb_api_pruned_scan_count_session            | 0                             |
    | Ndb_api_scan_batch_count_session             | 0                             |
    | Ndb_api_read_row_count_session               | 0                             |
    | Ndb_api_trans_local_read_row_count_session   | 0                             |
    | Ndb_api_adaptive_send_forced_count_session   | 0                             |
    | Ndb_api_adaptive_send_unforced_count_session | 0                             |
    | Ndb_api_adaptive_send_deferred_count_session | 0                             |
    | Ndb_trans_hint_count_session                 | 0                             |
    | Ndb_sorted_scan_count                        | 0                             |
    | Ndb_pushed_queries_defined                   | 0                             |
    | Ndb_pushed_queries_dropped                   | 0                             |
    | Ndb_pushed_queries_executed                  | 0                             |
    | Ndb_pushed_reads                             | 0                             |
    | Ndb_last_commit_epoch_server                 | 37632503447571                |
    | Ndb_last_commit_epoch_session                | 0                             |
    | Ndb_system_name                              | MC_20191126162038             |
    | Ndb_api_event_data_count_injector            | 0                             |
    | Ndb_api_event_nondata_count_injector         | 0                             |
    | Ndb_api_event_bytes_count_injector           | 0                             |
    | Ndb_api_wait_exec_complete_count_slave       | 0                             |
    | Ndb_api_wait_scan_result_count_slave         | 0                             |
    | Ndb_api_wait_meta_request_count_slave        | 0                             |
    | Ndb_api_wait_nanos_count_slave               | 0                             |
    | Ndb_api_bytes_sent_count_slave               | 0                             |
    | Ndb_api_bytes_received_count_slave           | 0                             |
    | Ndb_api_trans_start_count_slave              | 0                             |
    | Ndb_api_trans_commit_count_slave             | 0                             |
    | Ndb_api_trans_abort_count_slave              | 0                             |
    | Ndb_api_trans_close_count_slave              | 0                             |
    | Ndb_api_pk_op_count_slave                    | 0                             |
    | Ndb_api_uk_op_count_slave                    | 0                             |
    | Ndb_api_table_scan_count_slave               | 0                             |
    | Ndb_api_range_scan_count_slave               | 0                             |
    | Ndb_api_pruned_scan_count_slave              | 0                             |
    | Ndb_api_scan_batch_count_slave               | 0                             |
    | Ndb_api_read_row_count_slave                 | 0                             |
    | Ndb_api_trans_local_read_row_count_slave     | 0                             |
    | Ndb_api_adaptive_send_forced_count_slave     | 0                             |
    | Ndb_api_adaptive_send_unforced_count_slave   | 0                             |
    | Ndb_api_adaptive_send_deferred_count_slave   | 0                             |
    | Ndb_slave_max_replicated_epoch               | 0                             |
    | Ndb_api_wait_exec_complete_count             | 4                             |
    | Ndb_api_wait_scan_result_count               | 7                             |
    | Ndb_api_wait_meta_request_count              | 172                           |
    | Ndb_api_wait_nanos_count                     | 1083548094028                 |
    | Ndb_api_bytes_sent_count                     | 4640                          |
    | Ndb_api_bytes_received_count                 | 109356                        |
    | Ndb_api_trans_start_count                    | 4                             |
    | Ndb_api_trans_commit_count                   | 1                             |
    | Ndb_api_trans_abort_count                    | 1                             |
    | Ndb_api_trans_close_count                    | 4                             |
    | Ndb_api_pk_op_count                          | 2                             |
    | Ndb_api_uk_op_count                          | 0                             |
    | Ndb_api_table_scan_count                     | 1                             |
    | Ndb_api_range_scan_count                     | 1                             |
    | Ndb_api_pruned_scan_count                    | 0                             |
    | Ndb_api_scan_batch_count                     | 1                             |
    | Ndb_api_read_row_count                       | 3                             |
    | Ndb_api_trans_local_read_row_count           | 2                             |
    | Ndb_api_adaptive_send_forced_count           | 1                             |
    | Ndb_api_adaptive_send_unforced_count         | 5                             |
    | Ndb_api_adaptive_send_deferred_count         | 0                             |
    | Ndb_api_event_data_count                     | 0                             |
    | Ndb_api_event_nondata_count                  | 0                             |
    | Ndb_api_event_bytes_count                    | 0                             |
    | Ndb_metadata_excluded_count                  | 0                             |
    | Ndb_metadata_synced_count                    | 0                             |
    | Ndb_conflict_fn_max                          | 0                             |
    | Ndb_conflict_fn_old                          | 0                             |
    | Ndb_conflict_fn_max_del_win                  | 0                             |
    | Ndb_conflict_fn_epoch                        | 0                             |
    | Ndb_conflict_fn_epoch_trans                  | 0                             |
    | Ndb_conflict_fn_epoch2                       | 0                             |
    | Ndb_conflict_fn_epoch2_trans                 | 0                             |
    | Ndb_conflict_trans_row_conflict_count        | 0                             |
    | Ndb_conflict_trans_row_reject_count          | 0                             |
    | Ndb_conflict_trans_reject_count              | 0                             |
    | Ndb_conflict_trans_detect_iter_count         | 0                             |
    | Ndb_conflict_trans_conflict_commit_count     | 0                             |
    | Ndb_conflict_epoch_delete_delete_count       | 0                             |
    | Ndb_conflict_reflected_op_prepare_count      | 0                             |
    | Ndb_conflict_reflected_op_discard_count      | 0                             |
    | Ndb_conflict_refresh_op_count                | 0                             |
    | Ndb_conflict_last_conflict_epoch             | 0                             |
    | Ndb_conflict_last_stable_epoch               | 0                             |
    | Ndb_index_stat_status                        | allow:1,enable:1,busy:0,
    loop:1000,list:(new:0,update:0,read:0,idle:0,check:0,delete:0,error:0,total:0),
    analyze:(queue:0,wait:0),stats:(nostats:0,wait:0),total:(analyze:(all:0,error:0),
    query:(all:0,nostats:0,error:0),event:(act:0,skip:0,miss:0),cache:(refresh:0,
    clean:0,pinned:0,drop:0,evict:0)),cache:(query:0,clean:0,drop:0,evict:0,
    usedpct:0.00,highpct:0.00)                                                     |
    | Ndb_index_stat_cache_query                   | 0                             |
    | Ndb_index_stat_cache_clean                   | 0                             |
    +----------------------------------------------+-------------------------------+
    ```

    如果 MySQL 服务器是使用`NDB`支持构建的，但当前未连接到集群，那么此语句的输出中的每一行都包含`Value`列的零或空字符串。

    参见第 15.7.7.37 节，“SHOW STATUS Statement”。

+   `SELECT * FROM performance_schema.global_status WHERE VARIABLE_NAME LIKE 'NDB%'`

    这个语句提供了类似于之前讨论的`SHOW STATUS`语句的输出。与`SHOW STATUS`不同的是，可以使用`SELECT`语句提取 SQL 中的数值，用于监控和自动化脚本。

    更多信息，请参见第 29.12.15 节，“性能模式状态变量表”。

+   `SELECT * FROM INFORMATION_SCHEMA.PLUGINS WHERE PLUGIN_NAME LIKE 'NDB%'`

    这个语句显示了与 NDB Cluster 相关的插件的信息，如版本、作者和许可证，如下所示：

    ```sql
    mysql> SELECT * FROM INFORMATION_SCHEMA.PLUGINS
         >     WHERE PLUGIN_NAME LIKE 'NDB%'\G
    *************************** 1\. row ***************************
               PLUGIN_NAME: ndbcluster
            PLUGIN_VERSION: 1.0
             PLUGIN_STATUS: ACTIVE
               PLUGIN_TYPE: STORAGE ENGINE
       PLUGIN_TYPE_VERSION: 80036.0
            PLUGIN_LIBRARY: NULL
    PLUGIN_LIBRARY_VERSION: NULL
             PLUGIN_AUTHOR: Oracle Corporation
        PLUGIN_DESCRIPTION: Clustered, fault-tolerant tables
            PLUGIN_LICENSE: GPL
               LOAD_OPTION: ON
    *************************** 2\. row ***************************
               PLUGIN_NAME: ndbinfo
            PLUGIN_VERSION: 0.1
             PLUGIN_STATUS: ACTIVE
               PLUGIN_TYPE: STORAGE ENGINE
       PLUGIN_TYPE_VERSION: 80036.0
            PLUGIN_LIBRARY: NULL
    PLUGIN_LIBRARY_VERSION: NULL
             PLUGIN_AUTHOR: Oracle Corporation
        PLUGIN_DESCRIPTION: MySQL Cluster system information storage engine
            PLUGIN_LICENSE: GPL
               LOAD_OPTION: ON
    *************************** 3\. row ***************************
               PLUGIN_NAME: ndb_transid_mysql_connection_map
            PLUGIN_VERSION: 0.1
             PLUGIN_STATUS: ACTIVE
               PLUGIN_TYPE: INFORMATION SCHEMA
       PLUGIN_TYPE_VERSION: 80036.0
            PLUGIN_LIBRARY: NULL
    PLUGIN_LIBRARY_VERSION: NULL
             PLUGIN_AUTHOR: Oracle Corporation
        PLUGIN_DESCRIPTION: Map between MySQL connection ID and NDB transaction ID
            PLUGIN_LICENSE: GPL
               LOAD_OPTION: ON
    ```

    你也可以使用`SHOW PLUGINS`语句来显示这些信息，但该语句的输出不容易过滤。另请参阅 MySQL 插件 API，其中描述了`PLUGINS`表中信息的获取位置和方式。

你还可以查询`ndbinfo`信息数据库中的表，获取关于许多 NDB 集群操作的实时数据。参见 Section 25.6.16, “ndbinfo: The NDB Cluster Information Database”。
