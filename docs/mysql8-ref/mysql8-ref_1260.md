# 17.15.6 InnoDB INFORMATION_SCHEMA Metrics 表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-information-schema-metrics-table.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-information-schema-metrics-table.html)

`INNODB_METRICS` 表提供关于 `InnoDB` 性能和资源相关计数器的信息。

`INNODB_METRICS` 表列如下。有关列描述，请参见 Section 28.4.21, “INFORMATION_SCHEMA INNODB_METRICS 表”。

```sql
mysql> SELECT * FROM INFORMATION_SCHEMA.INNODB_METRICS WHERE NAME="dml_inserts" \G
*************************** 1\. row ***************************
           NAME: dml_inserts
      SUBSYSTEM: dml
          COUNT: 46273
      MAX_COUNT: 46273
      MIN_COUNT: NULL
      AVG_COUNT: 492.2659574468085
    COUNT_RESET: 46273
MAX_COUNT_RESET: 46273
MIN_COUNT_RESET: NULL
AVG_COUNT_RESET: NULL
   TIME_ENABLED: 2014-11-28 16:07:53
  TIME_DISABLED: NULL
   TIME_ELAPSED: 94
     TIME_RESET: NULL
         STATUS: enabled
           TYPE: status_counter
        COMMENT: Number of rows inserted
```

#### 启用、禁用和重置计数器

您可以使用以下变量启用、禁用和重置计数器：

+   `innodb_monitor_enable`: 启用计数器。

    ```sql
    SET GLOBAL innodb_monitor_enable = [counter-name|module_name|pattern|all];
    ```

+   `innodb_monitor_disable`: 禁用计数器。

    ```sql
    SET GLOBAL innodb_monitor_disable = [counter-name|module_name|pattern|all];
    ```

+   `innodb_monitor_reset`: 将计数器值重置为零。

    ```sql
    SET GLOBAL innodb_monitor_reset = [counter-name|module_name|pattern|all];
    ```

+   `innodb_monitor_reset_all`: 重置所有计数器值。在使用 `innodb_monitor_reset_all` 之前，必须禁用计数器。

    ```sql
    SET GLOBAL innodb_monitor_reset_all = [counter-name|module_name|pattern|all];
    ```

计数器和计数器模块也可以在启动时使用 MySQL 服务器配置文件启用。例如，要启用 `log` 模块，`metadata_table_handles_opened` 和 `metadata_table_handles_closed` 计数器，在 MySQL 服务器配置文件的 `[mysqld]` 部分中输入以下行。

```sql
[mysqld]
innodb_monitor_enable = log,metadata_table_handles_opened,metadata_table_handles_closed
```

在配置文件中启用多个计数器或模块时，请指定 `innodb_monitor_enable` 变量，后跟以逗号分隔的计数器和模块名称，如上所示。只能在配置文件中使用 `innodb_monitor_enable` 变量。`innodb_monitor_disable` 和 `innodb_monitor_reset` 变量仅在命令行上受支持。

注意

由于每个计数器都会增加运行时开销，建议在生产服务器上谨慎使用计数器来诊断特定问题或监视特定功能。建议在测试或开发服务器上更广泛地使用计数器。

#### 计数器

可用计数器列表可能会更改。查询信息模式 `INNODB_METRICS` 表，以获��您的 MySQL 服务器版本中可用的计数器。

默认启用的计数器对应于 `SHOW ENGINE INNODB STATUS` 输出中显示的计数器。在 `SHOW ENGINE INNODB STATUS` 输出中显示的计数器始终在系统级别启用，但可以在 `INNODB_METRICS` 表中禁用。计数器状态不是持久的。除非另有配置，否则在服务器重新启动时，计数器会恢复到它们的默认启用或禁用状态。

如果您运行的程序会受到计数器的添加或移除影响，建议您查看发布说明并查询 `INNODB_METRICS` 表，以识别这些更改作为升级过程的一部分。

```sql
mysql> SELECT name, subsystem, status FROM INFORMATION_SCHEMA.INNODB_METRICS ORDER BY NAME;
+---------------------------------------------+---------------------+----------+
| name                                        | subsystem           | status   |
+---------------------------------------------+---------------------+----------+
| adaptive_hash_pages_added                   | adaptive_hash_index | disabled |
| adaptive_hash_pages_removed                 | adaptive_hash_index | disabled |
| adaptive_hash_rows_added                    | adaptive_hash_index | disabled |
| adaptive_hash_rows_deleted_no_hash_entry    | adaptive_hash_index | disabled |
| adaptive_hash_rows_removed                  | adaptive_hash_index | disabled |
| adaptive_hash_rows_updated                  | adaptive_hash_index | disabled |
| adaptive_hash_searches                      | adaptive_hash_index | enabled  |
| adaptive_hash_searches_btree                | adaptive_hash_index | enabled  |
| buffer_data_reads                           | buffer              | enabled  |
| buffer_data_written                         | buffer              | enabled  |
| buffer_flush_adaptive                       | buffer              | disabled |
| buffer_flush_adaptive_avg_pass              | buffer              | disabled |
| buffer_flush_adaptive_avg_time_est          | buffer              | disabled |
| buffer_flush_adaptive_avg_time_slot         | buffer              | disabled |
| buffer_flush_adaptive_avg_time_thread       | buffer              | disabled |
| buffer_flush_adaptive_pages                 | buffer              | disabled |
| buffer_flush_adaptive_total_pages           | buffer              | disabled |
| buffer_flush_avg_page_rate                  | buffer              | disabled |
| buffer_flush_avg_pass                       | buffer              | disabled |
| buffer_flush_avg_time                       | buffer              | disabled |
| buffer_flush_background                     | buffer              | disabled |
| buffer_flush_background_pages               | buffer              | disabled |
| buffer_flush_background_total_pages         | buffer              | disabled |
| buffer_flush_batches                        | buffer              | disabled |
| buffer_flush_batch_num_scan                 | buffer              | disabled |
| buffer_flush_batch_pages                    | buffer              | disabled |
| buffer_flush_batch_scanned                  | buffer              | disabled |
| buffer_flush_batch_scanned_per_call         | buffer              | disabled |
| buffer_flush_batch_total_pages              | buffer              | disabled |
| buffer_flush_lsn_avg_rate                   | buffer              | disabled |
| buffer_flush_neighbor                       | buffer              | disabled |
| buffer_flush_neighbor_pages                 | buffer              | disabled |
| buffer_flush_neighbor_total_pages           | buffer              | disabled |
| buffer_flush_n_to_flush_by_age              | buffer              | disabled |
| buffer_flush_n_to_flush_by_dirty_page       | buffer              | disabled |
| buffer_flush_n_to_flush_requested           | buffer              | disabled |
| buffer_flush_pct_for_dirty                  | buffer              | disabled |
| buffer_flush_pct_for_lsn                    | buffer              | disabled |
| buffer_flush_sync                           | buffer              | disabled |
| buffer_flush_sync_pages                     | buffer              | disabled |
| buffer_flush_sync_total_pages               | buffer              | disabled |
| buffer_flush_sync_waits                     | buffer              | disabled |
| buffer_LRU_batches_evict                    | buffer              | disabled |
| buffer_LRU_batches_flush                    | buffer              | disabled |
| buffer_LRU_batch_evict_pages                | buffer              | disabled |
| buffer_LRU_batch_evict_total_pages          | buffer              | disabled |
| buffer_LRU_batch_flush_avg_pass             | buffer              | disabled |
| buffer_LRU_batch_flush_avg_time_est         | buffer              | disabled |
| buffer_LRU_batch_flush_avg_time_slot        | buffer              | disabled |
| buffer_LRU_batch_flush_avg_time_thread      | buffer              | disabled |
| buffer_LRU_batch_flush_pages                | buffer              | disabled |
| buffer_LRU_batch_flush_total_pages          | buffer              | disabled |
| buffer_LRU_batch_num_scan                   | buffer              | disabled |
| buffer_LRU_batch_scanned                    | buffer              | disabled |
| buffer_LRU_batch_scanned_per_call           | buffer              | disabled |
| buffer_LRU_get_free_loops                   | buffer              | disabled |
| buffer_LRU_get_free_search                  | Buffer              | disabled |
| buffer_LRU_get_free_waits                   | buffer              | disabled |
| buffer_LRU_search_num_scan                  | buffer              | disabled |
| buffer_LRU_search_scanned                   | buffer              | disabled |
| buffer_LRU_search_scanned_per_call          | buffer              | disabled |
| buffer_LRU_single_flush_failure_count       | Buffer              | disabled |
| buffer_LRU_single_flush_num_scan            | buffer              | disabled |
| buffer_LRU_single_flush_scanned             | buffer              | disabled |
| buffer_LRU_single_flush_scanned_per_call    | buffer              | disabled |
| buffer_LRU_unzip_search_num_scan            | buffer              | disabled |
| buffer_LRU_unzip_search_scanned             | buffer              | disabled |
| buffer_LRU_unzip_search_scanned_per_call    | buffer              | disabled |
| buffer_pages_created                        | buffer              | enabled  |
| buffer_pages_read                           | buffer              | enabled  |
| buffer_pages_written                        | buffer              | enabled  |
| buffer_page_read_blob                       | buffer_page_io      | disabled |
| buffer_page_read_fsp_hdr                    | buffer_page_io      | disabled |
| buffer_page_read_ibuf_bitmap                | buffer_page_io      | disabled |
| buffer_page_read_ibuf_free_list             | buffer_page_io      | disabled |
| buffer_page_read_index_ibuf_leaf            | buffer_page_io      | disabled |
| buffer_page_read_index_ibuf_non_leaf        | buffer_page_io      | disabled |
| buffer_page_read_index_inode                | buffer_page_io      | disabled |
| buffer_page_read_index_leaf                 | buffer_page_io      | disabled |
| buffer_page_read_index_non_leaf             | buffer_page_io      | disabled |
| buffer_page_read_other                      | buffer_page_io      | disabled |
| buffer_page_read_rseg_array                 | buffer_page_io      | disabled |
| buffer_page_read_system_page                | buffer_page_io      | disabled |
| buffer_page_read_trx_system                 | buffer_page_io      | disabled |
| buffer_page_read_undo_log                   | buffer_page_io      | disabled |
| buffer_page_read_xdes                       | buffer_page_io      | disabled |
| buffer_page_read_zblob                      | buffer_page_io      | disabled |
| buffer_page_read_zblob2                     | buffer_page_io      | disabled |
| buffer_page_written_blob                    | buffer_page_io      | disabled |
| buffer_page_written_fsp_hdr                 | buffer_page_io      | disabled |
| buffer_page_written_ibuf_bitmap             | buffer_page_io      | disabled |
| buffer_page_written_ibuf_free_list          | buffer_page_io      | disabled |
| buffer_page_written_index_ibuf_leaf         | buffer_page_io      | disabled |
| buffer_page_written_index_ibuf_non_leaf     | buffer_page_io      | disabled |
| buffer_page_written_index_inode             | buffer_page_io      | disabled |
| buffer_page_written_index_leaf              | buffer_page_io      | disabled |
| buffer_page_written_index_non_leaf          | buffer_page_io      | disabled |
| buffer_page_written_on_log_no_waits         | buffer_page_io      | disabled |
| buffer_page_written_on_log_waits            | buffer_page_io      | disabled |
| buffer_page_written_on_log_wait_loops       | buffer_page_io      | disabled |
| buffer_page_written_other                   | buffer_page_io      | disabled |
| buffer_page_written_rseg_array              | buffer_page_io      | disabled |
| buffer_page_written_system_page             | buffer_page_io      | disabled |
| buffer_page_written_trx_system              | buffer_page_io      | disabled |
| buffer_page_written_undo_log                | buffer_page_io      | disabled |
| buffer_page_written_xdes                    | buffer_page_io      | disabled |
| buffer_page_written_zblob                   | buffer_page_io      | disabled |
| buffer_page_written_zblob2                  | buffer_page_io      | disabled |
| buffer_pool_bytes_data                      | buffer              | enabled  |
| buffer_pool_bytes_dirty                     | buffer              | enabled  |
| buffer_pool_pages_data                      | buffer              | enabled  |
| buffer_pool_pages_dirty                     | buffer              | enabled  |
| buffer_pool_pages_free                      | buffer              | enabled  |
| buffer_pool_pages_misc                      | buffer              | enabled  |
| buffer_pool_pages_total                     | buffer              | enabled  |
| buffer_pool_reads                           | buffer              | enabled  |
| buffer_pool_read_ahead                      | buffer              | enabled  |
| buffer_pool_read_ahead_evicted              | buffer              | enabled  |
| buffer_pool_read_requests                   | buffer              | enabled  |
| buffer_pool_size                            | server              | enabled  |
| buffer_pool_wait_free                       | buffer              | enabled  |
| buffer_pool_write_requests                  | buffer              | enabled  |
| compression_pad_decrements                  | compression         | disabled |
| compression_pad_increments                  | compression         | disabled |
| compress_pages_compressed                   | compression         | disabled |
| compress_pages_decompressed                 | compression         | disabled |
| cpu_n                                       | cpu                 | disabled |
| cpu_stime_abs                               | cpu                 | disabled |
| cpu_stime_pct                               | cpu                 | disabled |
| cpu_utime_abs                               | cpu                 | disabled |
| cpu_utime_pct                               | cpu                 | disabled |
| dblwr_async_requests                        | dblwr               | disabled |
| dblwr_flush_requests                        | dblwr               | disabled |
| dblwr_flush_wait_events                     | dblwr               | disabled |
| dblwr_sync_requests                         | dblwr               | disabled |
| ddl_background_drop_tables                  | ddl                 | disabled |
| ddl_log_file_alter_table                    | ddl                 | disabled |
| ddl_online_create_index                     | ddl                 | disabled |
| ddl_pending_alter_table                     | ddl                 | disabled |
| ddl_sort_file_alter_table                   | ddl                 | disabled |
| dml_deletes                                 | dml                 | enabled  |
| dml_inserts                                 | dml                 | enabled  |
| dml_reads                                   | dml                 | disabled |
| dml_system_deletes                          | dml                 | enabled  |
| dml_system_inserts                          | dml                 | enabled  |
| dml_system_reads                            | dml                 | enabled  |
| dml_system_updates                          | dml                 | enabled  |
| dml_updates                                 | dml                 | enabled  |
| file_num_open_files                         | file_system         | enabled  |
| ibuf_merges                                 | change_buffer       | enabled  |
| ibuf_merges_delete                          | change_buffer       | enabled  |
| ibuf_merges_delete_mark                     | change_buffer       | enabled  |
| ibuf_merges_discard_delete                  | change_buffer       | enabled  |
| ibuf_merges_discard_delete_mark             | change_buffer       | enabled  |
| ibuf_merges_discard_insert                  | change_buffer       | enabled  |
| ibuf_merges_insert                          | change_buffer       | enabled  |
| ibuf_size                                   | change_buffer       | enabled  |
| icp_attempts                                | icp                 | disabled |
| icp_match                                   | icp                 | disabled |
| icp_no_match                                | icp                 | disabled |
| icp_out_of_range                            | icp                 | disabled |
| index_page_discards                         | index               | disabled |
| index_page_merge_attempts                   | index               | disabled |
| index_page_merge_successful                 | index               | disabled |
| index_page_reorg_attempts                   | index               | disabled |
| index_page_reorg_successful                 | index               | disabled |
| index_page_splits                           | index               | disabled |
| innodb_activity_count                       | server              | enabled  |
| innodb_background_drop_table_usec           | server              | disabled |
| innodb_dblwr_pages_written                  | server              | enabled  |
| innodb_dblwr_writes                         | server              | enabled  |
| innodb_dict_lru_count                       | server              | disabled |
| innodb_dict_lru_usec                        | server              | disabled |
| innodb_ibuf_merge_usec                      | server              | disabled |
| innodb_master_active_loops                  | server              | disabled |
| innodb_master_idle_loops                    | server              | disabled |
| innodb_master_purge_usec                    | server              | disabled |
| innodb_master_thread_sleeps                 | server              | disabled |
| innodb_mem_validate_usec                    | server              | disabled |
| innodb_page_size                            | server              | enabled  |
| innodb_rwlock_sx_os_waits                   | server              | enabled  |
| innodb_rwlock_sx_spin_rounds                | server              | enabled  |
| innodb_rwlock_sx_spin_waits                 | server              | enabled  |
| innodb_rwlock_s_os_waits                    | server              | enabled  |
| innodb_rwlock_s_spin_rounds                 | server              | enabled  |
| innodb_rwlock_s_spin_waits                  | server              | enabled  |
| innodb_rwlock_x_os_waits                    | server              | enabled  |
| innodb_rwlock_x_spin_rounds                 | server              | enabled  |
| innodb_rwlock_x_spin_waits                  | server              | enabled  |
| lock_deadlocks                              | lock                | enabled  |
| lock_deadlock_false_positives               | lock                | enabled  |
| lock_deadlock_rounds                        | lock                | enabled  |
| lock_rec_grant_attempts                     | lock                | enabled  |
| lock_rec_locks                              | lock                | disabled |
| lock_rec_lock_created                       | lock                | disabled |
| lock_rec_lock_removed                       | lock                | disabled |
| lock_rec_lock_requests                      | lock                | disabled |
| lock_rec_lock_waits                         | lock                | disabled |
| lock_rec_release_attempts                   | lock                | enabled  |
| lock_row_lock_current_waits                 | lock                | enabled  |
| lock_row_lock_time                          | lock                | enabled  |
| lock_row_lock_time_avg                      | lock                | enabled  |
| lock_row_lock_time_max                      | lock                | enabled  |
| lock_row_lock_waits                         | lock                | enabled  |
| lock_schedule_refreshes                     | lock                | enabled  |
| lock_table_locks                            | lock                | disabled |
| lock_table_lock_created                     | lock                | disabled |
| lock_table_lock_removed                     | lock                | disabled |
| lock_table_lock_waits                       | lock                | disabled |
| lock_threads_waiting                        | lock                | enabled  |
| lock_timeouts                               | lock                | enabled  |
| log_checkpoints                             | log                 | disabled |
| log_concurrency_margin                      | log                 | disabled |
| log_flusher_no_waits                        | log                 | disabled |
| log_flusher_waits                           | log                 | disabled |
| log_flusher_wait_loops                      | log                 | disabled |
| log_flush_avg_time                          | log                 | disabled |
| log_flush_lsn_avg_rate                      | log                 | disabled |
| log_flush_max_time                          | log                 | disabled |
| log_flush_notifier_no_waits                 | log                 | disabled |
| log_flush_notifier_waits                    | log                 | disabled |
| log_flush_notifier_wait_loops               | log                 | disabled |
| log_flush_total_time                        | log                 | disabled |
| log_free_space                              | log                 | disabled |
| log_full_block_writes                       | log                 | disabled |
| log_lsn_archived                            | log                 | disabled |
| log_lsn_buf_dirty_pages_added               | log                 | disabled |
| log_lsn_buf_pool_oldest_approx              | log                 | disabled |
| log_lsn_buf_pool_oldest_lwm                 | log                 | disabled |
| log_lsn_checkpoint_age                      | log                 | disabled |
| log_lsn_current                             | log                 | disabled |
| log_lsn_last_checkpoint                     | log                 | disabled |
| log_lsn_last_flush                          | log                 | disabled |
| log_max_modified_age_async                  | log                 | disabled |
| log_max_modified_age_sync                   | log                 | disabled |
| log_next_file                               | log                 | disabled |
| log_on_buffer_space_no_waits                | log                 | disabled |
| log_on_buffer_space_waits                   | log                 | disabled |
| log_on_buffer_space_wait_loops              | log                 | disabled |
| log_on_file_space_no_waits                  | log                 | disabled |
| log_on_file_space_waits                     | log                 | disabled |
| log_on_file_space_wait_loops                | log                 | disabled |
| log_on_flush_no_waits                       | log                 | disabled |
| log_on_flush_waits                          | log                 | disabled |
| log_on_flush_wait_loops                     | log                 | disabled |
| log_on_recent_closed_wait_loops             | log                 | disabled |
| log_on_recent_written_wait_loops            | log                 | disabled |
| log_on_write_no_waits                       | log                 | disabled |
| log_on_write_waits                          | log                 | disabled |
| log_on_write_wait_loops                     | log                 | disabled |
| log_padded                                  | log                 | disabled |
| log_partial_block_writes                    | log                 | disabled |
| log_waits                                   | log                 | enabled  |
| log_writer_no_waits                         | log                 | disabled |
| log_writer_on_archiver_waits                | log                 | disabled |
| log_writer_on_file_space_waits              | log                 | disabled |
| log_writer_waits                            | log                 | disabled |
| log_writer_wait_loops                       | log                 | disabled |
| log_writes                                  | log                 | enabled  |
| log_write_notifier_no_waits                 | log                 | disabled |
| log_write_notifier_waits                    | log                 | disabled |
| log_write_notifier_wait_loops               | log                 | disabled |
| log_write_requests                          | log                 | enabled  |
| log_write_to_file_requests_interval         | log                 | disabled |
| metadata_table_handles_closed               | metadata            | disabled |
| metadata_table_handles_opened               | metadata            | disabled |
| metadata_table_reference_count              | metadata            | disabled |
| module_cpu                                  | cpu                 | disabled |
| module_dblwr                                | dblwr               | disabled |
| module_page_track                           | page_track          | disabled |
| os_data_fsyncs                              | os                  | enabled  |
| os_data_reads                               | os                  | enabled  |
| os_data_writes                              | os                  | enabled  |
| os_log_bytes_written                        | os                  | enabled  |
| os_log_fsyncs                               | os                  | enabled  |
| os_log_pending_fsyncs                       | os                  | enabled  |
| os_log_pending_writes                       | os                  | enabled  |
| os_pending_reads                            | os                  | disabled |
| os_pending_writes                           | os                  | disabled |
| page_track_checkpoint_partial_flush_request | page_track          | disabled |
| page_track_full_block_writes                | page_track          | disabled |
| page_track_partial_block_writes             | page_track          | disabled |
| page_track_resets                           | page_track          | disabled |
| purge_del_mark_records                      | purge               | disabled |
| purge_dml_delay_usec                        | purge               | disabled |
| purge_invoked                               | purge               | disabled |
| purge_resume_count                          | purge               | disabled |
| purge_stop_count                            | purge               | disabled |
| purge_truncate_history_count                | purge               | disabled |
| purge_truncate_history_usec                 | purge               | disabled |
| purge_undo_log_pages                        | purge               | disabled |
| purge_upd_exist_or_extern_records           | purge               | disabled |
| sampled_pages_read                          | sampling            | disabled |
| sampled_pages_skipped                       | sampling            | disabled |
| trx_active_transactions                     | transaction         | disabled |
| trx_allocations                             | transaction         | disabled |
| trx_commits_insert_update                   | transaction         | disabled |
| trx_nl_ro_commits                           | transaction         | disabled |
| trx_on_log_no_waits                         | transaction         | disabled |
| trx_on_log_waits                            | transaction         | disabled |
| trx_on_log_wait_loops                       | transaction         | disabled |
| trx_rollbacks                               | transaction         | disabled |
| trx_rollbacks_savepoint                     | transaction         | disabled |
| trx_rollback_active                         | transaction         | disabled |
| trx_ro_commits                              | transaction         | disabled |
| trx_rseg_current_size                       | transaction         | disabled |
| trx_rseg_history_len                        | transaction         | enabled  |
| trx_rw_commits                              | transaction         | disabled |
| trx_undo_slots_cached                       | transaction         | disabled |
| trx_undo_slots_used                         | transaction         | disabled |
| undo_truncate_count                         | undo                | disabled |
| undo_truncate_done_logging_count            | undo                | disabled |
| undo_truncate_start_logging_count           | undo                | disabled |
| undo_truncate_usec                          | undo                | disabled |
+---------------------------------------------+---------------------+----------+
314 rows in set (0.00 sec)
```

#### 计数器模块

每个计数器与特定模块相关联。模块名称可用于启用、禁用或重置特定子系统的所有计数器。例如，使用 `module_dml` 可以启��与 `dml` 子系统相关的所有计数器。

```sql
mysql> SET GLOBAL innodb_monitor_enable = module_dml;

mysql> SELECT name, subsystem, status FROM INFORMATION_SCHEMA.INNODB_METRICS
       WHERE subsystem ='dml';
+-------------+-----------+---------+
| name        | subsystem | status  |
+-------------+-----------+---------+
| dml_reads   | dml       | enabled |
| dml_inserts | dml       | enabled |
| dml_deletes | dml       | enabled |
| dml_updates | dml       | enabled |
+-------------+-----------+---------+
```

模块名称可与 `innodb_monitor_enable` 和相关变量一起使用。

模块名称及其对应的 `SUBSYSTEM` 名称如下。

+   `module_adaptive_hash` (subsystem = `adaptive_hash_index`)

+   `module_buffer` (subsystem = `buffer`)

+   `module_buffer_page` (subsystem = `buffer_page_io`)

+   `module_compress` (subsystem = `compression`)

+   `module_ddl` (subsystem = `ddl`)

+   `module_dml` (subsystem = `dml`)

+   `module_file` (subsystem = `file_system`)

+   `module_ibuf_system` (subsystem = `change_buffer`)

+   `module_icp` (subsystem = `icp`)

+   `module_index` (subsystem = `index`)

+   `module_innodb` (subsystem = `innodb`)

+   `module_lock` (subsystem = `lock`)

+   `module_log` (subsystem = `log`)

+   `module_metadata` (subsystem = `metadata`)

+   `module_os` (subsystem = `os`)

+   `module_purge` (subsystem = `purge`)

+   `module_trx` (subsystem = `transaction`)

+   `module_undo` (subsystem = `undo`)

**示例 17.11 使用 INNODB_METRICS 表计数器**

本示例演示了启用、禁用和重置计数器，并在 `INNODB_METRICS` 表中查询计数器数据。

1.  创建一个简单的 `InnoDB` 表：

    ```sql
    mysql> USE test;
    Database changed

    mysql> CREATE TABLE t1 (c1 INT) ENGINE=INNODB;
    Query OK, 0 rows affected (0.02 sec)
    ```

1.  启用 `dml_inserts` 计数器。

    ```sql
    mysql> SET GLOBAL innodb_monitor_enable = dml_inserts;
    Query OK, 0 rows affected (0.01 sec)
    ```

    在 `INNODB_METRICS` 表的 `COMMENT` 列中可以找到 `dml_inserts` 计数器的描述：

    ```sql
    mysql> SELECT NAME, COMMENT FROM INFORMATION_SCHEMA.INNODB_METRICS WHERE NAME="dml_inserts";
    +-------------+-------------------------+
    | NAME        | COMMENT                 |
    +-------------+-------------------------+
    | dml_inserts | Number of rows inserted |
    +-------------+-------------------------+
    ```

1.  查询 `dml_inserts` 计数器数据的 `INNODB_METRICS` 表。因为没有执行 DML 操作，计数器值为零或 NULL。`TIME_ENABLED` 和 `TIME_ELAPSED` 值指示计数器上次启用的时间以及自那时经过的秒数。

    ```sql
    mysql>  SELECT * FROM INFORMATION_SCHEMA.INNODB_METRICS WHERE NAME="dml_inserts" \G
    *************************** 1\. row ***************************
               NAME: dml_inserts
          SUBSYSTEM: dml
              COUNT: 0
          MAX_COUNT: 0
          MIN_COUNT: NULL
          AVG_COUNT: 0
        COUNT_RESET: 0
    MAX_COUNT_RESET: 0
    MIN_COUNT_RESET: NULL
    AVG_COUNT_RESET: NULL
       TIME_ENABLED: 2014-12-04 14:18:28
      TIME_DISABLED: NULL
       TIME_ELAPSED: 28
         TIME_RESET: NULL
             STATUS: enabled
               TYPE: status_counter
            COMMENT: Number of rows inserted
    ```

1.  向表中插入三行数据。

    ```sql
    mysql> INSERT INTO t1 values(1);
    Query OK, 1 row affected (0.00 sec)

    mysql> INSERT INTO t1 values(2);
    Query OK, 1 row affected (0.00 sec)

    mysql> INSERT INTO t1 values(3);
    Query OK, 1 row affected (0.00 sec)
    ```

1.  再次查询`INNODB_METRICS`表以获取`dml_inserts`计数器数据。现在已经增加了许多计数器值，包括`COUNT`、`MAX_COUNT`、`AVG_COUNT`和`COUNT_RESET`。请参考`INNODB_METRICS`表定义，了解这些值的描述。

    ```sql
    mysql>  SELECT * FROM INFORMATION_SCHEMA.INNODB_METRICS WHERE NAME="dml_inserts"\G
    *************************** 1\. row ***************************
               NAME: dml_inserts
          SUBSYSTEM: dml
              COUNT: 3
          MAX_COUNT: 3
          MIN_COUNT: NULL
          AVG_COUNT: 0.046153846153846156
        COUNT_RESET: 3
    MAX_COUNT_RESET: 3
    MIN_COUNT_RESET: NULL
    AVG_COUNT_RESET: NULL
       TIME_ENABLED: 2014-12-04 14:18:28
      TIME_DISABLED: NULL
       TIME_ELAPSED: 65
         TIME_RESET: NULL
             STATUS: enabled
               TYPE: status_counter
            COMMENT: Number of rows inserted
    ```

1.  重置`dml_inserts`计数器，并再次查询`INNODB_METRICS`表以获取`dml_inserts`计数器数据。先前报告的`%_RESET`值，如`COUNT_RESET`和`MAX_RESET`，将被设置为零。像`COUNT`、`MAX_COUNT`和`AVG_COUNT`这样的值，从启用计数器开始累积收集数据，不受重置影响。

    ```sql
    mysql> SET GLOBAL innodb_monitor_reset = dml_inserts;
    Query OK, 0 rows affected (0.00 sec)

    mysql> SELECT * FROM INFORMATION_SCHEMA.INNODB_METRICS WHERE NAME="dml_inserts"\G
    *************************** 1\. row ***************************
               NAME: dml_inserts
          SUBSYSTEM: dml
              COUNT: 3
          MAX_COUNT: 3
          MIN_COUNT: NULL
          AVG_COUNT: 0.03529411764705882
        COUNT_RESET: 0
    MAX_COUNT_RESET: 0
    MIN_COUNT_RESET: NULL
    AVG_COUNT_RESET: 0
       TIME_ENABLED: 2014-12-04 14:18:28
      TIME_DISABLED: NULL
       TIME_ELAPSED: 85
         TIME_RESET: 2014-12-04 14:19:44
             STATUS: enabled
               TYPE: status_counter
            COMMENT: Number of rows inserted
    ```

1.  要重置所有计数器值，必须先禁用计数器。禁用计数器会将`STATUS`值设置为`disabled`。

    ```sql
    mysql> SET GLOBAL innodb_monitor_disable = dml_inserts;
    Query OK, 0 rows affected (0.00 sec)

    mysql> SELECT * FROM INFORMATION_SCHEMA.INNODB_METRICS WHERE NAME="dml_inserts"\G
    *************************** 1\. row ***************************
               NAME: dml_inserts
          SUBSYSTEM: dml
              COUNT: 3
          MAX_COUNT: 3
          MIN_COUNT: NULL
          AVG_COUNT: 0.030612244897959183
        COUNT_RESET: 0
    MAX_COUNT_RESET: 0
    MIN_COUNT_RESET: NULL
    AVG_COUNT_RESET: 0
       TIME_ENABLED: 2014-12-04 14:18:28
      TIME_DISABLED: 2014-12-04 14:20:06
       TIME_ELAPSED: 98
         TIME_RESET: NULL
             STATUS: disabled
               TYPE: status_counter
            COMMENT: Number of rows inserted
    ```

    注意

    计数器和模块名称支持通配符匹配。例如，可以指定`dml_i%`而不是完整的`dml_inserts`计数器名称。还可以使用通配符匹配一次性启用、禁用或重置多个计数器或模块。例如，指定`dml_%`以启用、禁用或重置所有以`dml_`开头的计数器。

1.  在禁用计数器后，可以使用`innodb_monitor_reset_all`选项重置所有计数器值。所有值都设置为零或 NULL。

    ```sql
    mysql> SET GLOBAL innodb_monitor_reset_all = dml_inserts;
    Query OK, 0 rows affected (0.00 sec)

    mysql> SELECT * FROM INFORMATION_SCHEMA.INNODB_METRICS WHERE NAME="dml_inserts"\G
    *************************** 1\. row ***************************
               NAME: dml_inserts
          SUBSYSTEM: dml
              COUNT: 0
          MAX_COUNT: NULL
          MIN_COUNT: NULL
          AVG_COUNT: NULL
        COUNT_RESET: 0
    MAX_COUNT_RESET: NULL
    MIN_COUNT_RESET: NULL
    AVG_COUNT_RESET: NULL
       TIME_ENABLED: NULL
      TIME_DISABLED: NULL
       TIME_ELAPSED: NULL
         TIME_RESET: NULL
             STATUS: disabled
               TYPE: status_counter
            COMMENT: Number of rows inserted
    ```
