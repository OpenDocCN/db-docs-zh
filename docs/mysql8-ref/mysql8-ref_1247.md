# 17.14 InnoDB 启动选项和系统变量

> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html)

+   可以通过命名启用或使用`--skip-`前缀禁用在服务器启动时为真或假的系统变量。例如，要启用或禁用`InnoDB`自适应哈希索引，可以在命令行上使用`--innodb-adaptive-hash-index`或`--skip-innodb-adaptive-hash-index`，或在选项文件中使用`innodb_adaptive_hash_index`或`skip_innodb_adaptive_hash_index`。

+   一些变量描述涉及“启用”或“禁用”变量。这些变量可以通过将它们设置为`ON`或`1`来使用`SET`语句启用，或通过将它们设置为`OFF`或`0`来禁用。布尔变量可以在启动时设置为`ON`、`TRUE`、`OFF`和`FALSE`（不区分大小写），以及`1`和`0`的值。参见第 6.2.2.4 节，“程序选项修饰符”。

+   ���受数值的系统变量���以在命令行上指定为`--*`var_name`*=*`value`*`，也可以在选项文件中指定为`*`var_name`*=*`value`*`。

+   许多系统变量可以在运行时更改（参见第 7.1.9.2 节，“动态系统变量”）。

+   有关`GLOBAL`和`SESSION`变量范围修饰符的信息，请参考`SET`语句文档。

+   某些选项控制`InnoDB`数据文件的位置和布局。第 17.8.1 节，“InnoDB 启动配置”解释了如何使用这些选项。

+   一些选项，可能最初不会使用，有助于根据机器容量和数据库工作负载调整`InnoDB`性能特征。

+   有关指定选项和系统变量的更多信息，请参见第 6.2.2 节，“指定程序选项”。

**表格 17.24 InnoDB 选项和变量参考**

| 名称 | 命令行 | 选项文件 | 系统变量 | 状态变量 | 变量范围 | 动态 |
| --- | --- | --- | --- | --- | --- | --- |
| daemon_memcached_enable_binlog | 是 | 是 | 是 |  | 全局 | 否 |
| daemon_memcached_engine_lib_name | 是 | 是 | 是 |  | 全局 | 否 |
| daemon_memcached_engine_lib_path | 是 | 是 | 是 |  | 全局 | 否 |
| daemon_memcached_option | 是 | 是 | 是 |  | 全局 | 否 |
| daemon_memcached_r_batch_size | 是 | 是 | 是 |  | 全局 | 否 |
| daemon_memcached_w_batch_size | 是 | 是 | 是 |  | 全局 | 否 |
| foreign_key_checks |  |  | 是 |  | 两者 | 是 |
| innodb | 是 | 是 |  |  |  |  |
| innodb_adaptive_flushing | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_adaptive_flushing_lwm | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_adaptive_hash_index | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_adaptive_hash_index_parts | 是 | 是 | 是 |  | 全局 | 否 |
| innodb_adaptive_max_sleep_delay | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_api_bk_commit_interval | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_api_disable_rowlock | 是 | 是 | 是 |  | 全局 | 否 |
| innodb_api_enable_binlog | 是 | 是 | 是 |  | 全局 | 否 |
| innodb_api_enable_mdl | 是 | 是 | 是 |  | 全局 | 否 |
| innodb_api_trx_level | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_autoextend_increment | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_autoinc_lock_mode | 是 | 是 | 是 |  | 全局 | 否 |
| innodb_background_drop_list_empty | 是 | 是 | 是 |  | 全局 | 是 |
| Innodb_buffer_pool_bytes_data |  |  |  | 是 | 全局 | 否 |
| Innodb_buffer_pool_bytes_dirty |  |  |  | 是 | 全局 | 否 |
| innodb_buffer_pool_chunk_size | 是 | 是 | 是 |  | 全局 | 否 |
| innodb_buffer_pool_debug | 是 | 是 | 是 |  | 全局 | 否 |
| innodb_buffer_pool_dump_at_shutdown | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_buffer_pool_dump_now | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_buffer_pool_dump_pct | 是 | 是 | 是 |  | 全局 | 是 |
| Innodb_buffer_pool_dump_status |  |  |  | 是 | 全局 | 否 |
| innodb_buffer_pool_filename | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_buffer_pool_in_core_file | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_buffer_pool_instances | 是 | 是 | 是 |  | 全局 | 否 |
| innodb_buffer_pool_load_abort | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_buffer_pool_load_at_startup | 是 | 是 | 是 |  | 全局 | 否 |
| innodb_buffer_pool_load_now | 是 | 是 | 是 |  | 全局 | 是 |
| Innodb_buffer_pool_load_status |  |  |  | 是 | 全局 | 否 |
| Innodb_buffer_pool_pages_data |  |  |  | 是 | 全局 | 否 |
| Innodb_buffer_pool_pages_dirty |  |  |  | 是 | 全局 | 否 |
| Innodb_buffer_pool_pages_flushed |  |  |  | 是 | 全局 | 否 |
| Innodb_buffer_pool_pages_free |  |  |  | 是 | 全局 | 否 |
| Innodb_buffer_pool_pages_latched |  |  |  | 是 | 全局 | 否 |
| Innodb_buffer_pool_pages_misc |  |  |  | 是 | 全局 | 否 |
| Innodb_buffer_pool_pages_total |  |  |  | 是 | 全局 | 否 |
| Innodb_buffer_pool_read_ahead |  |  |  | 是 | 全局 | 否 |
| Innodb_buffer_pool_read_ahead_evicted |  |  |  | 是 | 全局 | 否 |
| Innodb_buffer_pool_read_ahead_rnd |  |  |  | 是 | 全局 | 否 |
| Innodb_buffer_pool_read_requests |  |  |  | 是 | 全局 | 否 |
| Innodb_buffer_pool_reads |  |  |  | 是 | 全局 | 否 |
| Innodb_buffer_pool_resize_status |  |  |  | 是 | 全局 | 否 |
| innodb_buffer_pool_size | 是 | 是 | 是 |  | 全局 | 是 |
| Innodb_buffer_pool_wait_free |  |  |  | 是 | 全局 | 否 |
| Innodb_buffer_pool_write_requests |  |  |  | 是 | 全局 | 否 |
| innodb_change_buffer_max_size | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_change_buffering | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_change_buffering_debug | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_checkpoint_disabled | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_checksum_algorithm | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_cmp_per_index_enabled | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_commit_concurrency | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_compress_debug | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_compression_failure_threshold_pct | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_compression_level | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_compression_pad_pct_max | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_concurrency_tickets | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_data_file_path | 是 | 是 | 是 |  | 全局 | 否 |
| Innodb_data_fsyncs |  |  |  | 是 | 全局 | 否 |
| innodb_data_home_dir | 是 | 是 | 是 |  | 全局 | 否 |
| Innodb_data_pending_fsyncs |  |  |  | 是 | 全局 | 否 |
| Innodb_data_pending_reads |  |  |  | 是 | 全局 | 否 |
| Innodb_data_pending_writes |  |  |  | 是 | 全局 | 否 |
| Innodb_data_read |  |  |  | 是 | 全局 | 否 |
| Innodb_data_reads |  |  |  | 是 | 全局 | 否 |
| Innodb_data_writes |  |  |  | 是 | 全局 | 否 |
| Innodb_data_written |  |  |  | 是 | 全局 | 否 |
| Innodb_dblwr_pages_written |  |  |  | 是 | 全局 | 否 |
| Innodb_dblwr_writes |  |  |  | 是 | 全局 | 否 |
| innodb_ddl_buffer_size | 是 | 是 | 是 |  | 两者 | 是 |
| innodb_ddl_log_crash_reset_debug | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_ddl_threads | 是 | 是 | 是 |  | 两者 | 是 |
| innodb_deadlock_detect | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_dedicated_server | 是 | 是 | 是 |  | 全局 | 否 |
| innodb_default_row_format | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_directories | 是 | 是 | 是 |  | 全局 | 否 |
| innodb_disable_sort_file_cache | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_doublewrite | 是 | 是 | 是 |  | 全局 | 不定 |
| innodb_doublewrite_batch_size | 是 | 是 | 是 |  | 全局 | 否 |
| innodb_doublewrite_dir | 是 | 是 | 是 |  | 全局 | 否 |
| innodb_doublewrite_files | 是 | 是 | 是 |  | 全局 | 否 |
| innodb_doublewrite_pages | 是 | 是 | 是 |  | 全局 | 否 |
| innodb_fast_shutdown | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_fil_make_page_dirty_debug | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_file_per_table | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_fill_factor | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_flush_log_at_timeout | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_flush_log_at_trx_commit | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_flush_method | 是 | 是 | 是 |  | 全局 | 否 |
| innodb_flush_neighbors | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_flush_sync | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_flushing_avg_loops | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_force_load_corrupted | 是 | 是 | 是 |  | 全局 | 否 |
| innodb_force_recovery | 是 | 是 | 是 |  | 全局 | 否 |
| innodb_fsync_threshold | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_ft_aux_table |  |  | 是 |  | 全局 | 是 |
| innodb_ft_cache_size | 是 | 是 | 是 |  | 全局 | 否 |
| innodb_ft_enable_diag_print | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_ft_enable_stopword | 是 | 是 | 是 |  | 两者 | 是 |
| innodb_ft_max_token_size | 是 | 是 | 是 |  | 全局 | 否 |
| innodb_ft_min_token_size | 是 | 是 | 是 |  | 全局 | 否 |
| innodb_ft_num_word_optimize | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_ft_result_cache_limit | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_ft_server_stopword_table | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_ft_sort_pll_degree | 是 | 是 | 是 |  | 全局 | 否 |
| innodb_ft_total_cache_size | 是 | 是 | 是 |  | 全局 | 否 |
| innodb_ft_user_stopword_table | 是 | 是 | 是 |  | 两者 | 是 |
| Innodb_have_atomic_builtins |  |  |  | 是 | 全局 | 否 |
| innodb_idle_flush_pct | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_io_capacity | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_io_capacity_max | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_limit_optimistic_insert_debug | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_lock_wait_timeout | 是 | 是 | 是 |  | 两者 | 是 |
| innodb_log_buffer_size | 是 | 是 | 是 |  | 全局 | 变化 |
| innodb_log_checkpoint_fuzzy_now | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_log_checkpoint_now | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_log_checksums | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_log_compressed_pages | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_log_file_size | 是 | 是 | 是 |  | 全局 | 否 |
| innodb_log_files_in_group | 是 | 是 | 是 |  | 全局 | 否 |
| innodb_log_group_home_dir | 是 | 是 | 是 |  | 全局 | 否 |
| innodb_log_spin_cpu_abs_lwm | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_log_spin_cpu_pct_hwm | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_log_wait_for_flush_spin_hwm | 是 | 是 | 是 |  | 全局 | 是 |
| Innodb_log_waits |  |  |  | 是 | 全局 | 否 |
| innodb_log_write_ahead_size | 是 | 是 | 是 |  | 全局 | 是 |
| Innodb_log_write_requests |  |  |  | 是 | 全局 | 否 |
| innodb_log_writer_threads | 是 | 是 | 是 |  | 全局 | 是 |
| Innodb_log_writes |  |  |  | 是 | 全局 | 否 |
| innodb_lru_scan_depth | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_max_dirty_pages_pct | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_max_dirty_pages_pct_lwm | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_max_purge_lag | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_max_purge_lag_delay | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_max_undo_log_size | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_merge_threshold_set_all_debug | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_monitor_disable | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_monitor_enable | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_monitor_reset | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_monitor_reset_all | 是 | 是 | 是 |  | 全局 | 是 |
| Innodb_num_open_files |  |  |  | 是 | 全局 | 否 |
| innodb_numa_interleave | 是 | 是 | 是 |  | 全局 | 否 |
| innodb_old_blocks_pct | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_old_blocks_time | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_online_alter_log_max_size | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_open_files | 是 | 是 | 是 |  | 全局 | 不定 |
| innodb_optimize_fulltext_only | 是 | 是 | 是 |  | 全局 | 是 |
| Innodb_os_log_fsyncs |  |  |  | 是 | 全局 | 否 |
| Innodb_os_log_pending_fsyncs |  |  |  | 是 | 全局 | 否 |
| Innodb_os_log_pending_writes |  |  |  | 是 | 全局 | 否 |
| Innodb_os_log_written |  |  |  | 是 | 全局 | 否 |
| innodb_page_cleaners | 是 | 是 | 是 |  | 全局 | 否 |
| Innodb_page_size |  |  |  | 是 | 全局 | 否 |
| innodb_page_size | 是 | 是 | 是 |  | 全局 | 否 |
| Innodb_pages_created |  |  |  | 是 | 全局 | 否 |
| Innodb_pages_read |  |  |  | 是 | 全局 | 否 |
| Innodb_pages_written |  |  |  | 是 | 全局 | 否 |
| innodb_parallel_read_threads | 是 | 是 | 是 |  | 会话 | 是 |
| innodb_print_all_deadlocks | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_print_ddl_logs | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_purge_batch_size | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_purge_rseg_truncate_frequency | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_purge_threads | 是 | 是 | 是 |  | 全局 | 否 |
| innodb_random_read_ahead | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_read_ahead_threshold | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_read_io_threads | 是 | 是 | 是 |  | 全局 | 否 |
| innodb_read_only | 是 | 是 | 是 |  | 全局 | 否 |
| innodb_redo_log_archive_dirs | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_redo_log_capacity | 是 | 是 | 是 |  | 全局 | 是 |
| Innodb_redo_log_capacity_resized |  |  |  | 是 | 全局 | 否 |
| Innodb_redo_log_checkpoint_lsn |  |  |  | 是 | 全局 | 否 |
| Innodb_redo_log_current_lsn |  |  |  | 是 | 全局 | 否 |
| Innodb_redo_log_enabled |  |  |  | 是 | 全局 | 否 |
| innodb_redo_log_encrypt | 是 | 是 | 是 |  | 全局 | 是 |
| Innodb_redo_log_flushed_to_disk_lsn |  |  |  | 是 | 全局 | 否 |
| Innodb_redo_log_logical_size |  |  |  | 是 | 全局 | 否 |
| Innodb_redo_log_physical_size |  |  |  | 是 | 全局 | 否 |
| Innodb_redo_log_read_only |  |  |  | 是 | 全局 | 否 |
| Innodb_redo_log_resize_status |  |  |  | 是 | 全局 | 否 |
| Innodb_redo_log_uuid |  |  |  | 是 | 全局 | 否 |
| innodb_replication_delay | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_rollback_on_timeout | 是 | 是 | 是 |  | 全局 | 否 |
| innodb_rollback_segments | 是 | 是 | 是 |  | 全局 | 是 |
| Innodb_row_lock_current_waits |  |  |  | 是 | 全局 | 否 |
| Innodb_row_lock_time |  |  |  | 是 | 全局 | 否 |
| Innodb_row_lock_time_avg |  |  |  | 是 | 全局 | 否 |
| Innodb_row_lock_time_max |  |  |  | 是 | 全局 | 否 |
| Innodb_row_lock_waits |  |  |  | 是 | 全局 | 否 |
| Innodb_rows_deleted |  |  |  | 是 | 全局 | 否 |
| Innodb_rows_inserted |  |  |  | 是 | 全局 | 否 |
| Innodb_rows_read |  |  |  | 是 | 全局 | 否 |
| Innodb_rows_updated |  |  |  | 是 | 全局 | 否 |
| innodb_saved_page_number_debug | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_segment_reserve_factor | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_sort_buffer_size | 是 | 是 | 是 |  | 全局 | 否 |
| innodb_spin_wait_delay | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_spin_wait_pause_multiplier | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_stats_auto_recalc | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_stats_include_delete_marked | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_stats_method | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_stats_on_metadata | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_stats_persistent | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_stats_persistent_sample_pages | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_stats_transient_sample_pages | 是 | 是 | 是 |  | 全局 | 是 |
| innodb-status-file | 是 | 是 |  |  |  |  |
| innodb_status_output | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_status_output_locks | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_strict_mode | 是 | 是 | 是 |  | 两者 | 是 |
| innodb_sync_array_size | 是 | 是 | 是 |  | 全局 | 否 |
| innodb_sync_debug | 是 | 是 | 是 |  | 全局 | 否 |
| innodb_sync_spin_loops | 是 | 是 | 是 |  | 全局 | 是 |
| Innodb_system_rows_deleted |  |  |  | 是 | 全局 | 否 |
| Innodb_system_rows_inserted |  |  |  | 是 | 全局 | 否 |
| Innodb_system_rows_read |  |  |  | 是 | 全局 | 否 |
| innodb_table_locks | 是 | 是 | 是 |  | 两者 | 是 |
| innodb_temp_data_file_path | 是 | 是 | 是 |  | 全局 | 否 |
| innodb_temp_tablespaces_dir | 是 | 是 | 是 |  | 全局 | 否 |
| innodb_thread_concurrency | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_thread_sleep_delay | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_tmpdir | 是 | 是 | 是 |  | 两者 | 是 |
| Innodb_truncated_status_writes |  |  |  | 是 | 全局 | 否 |
| innodb_trx_purge_view_update_only_debug | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_trx_rseg_n_slots_debug | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_undo_directory | 是 | 是 | 是 |  | 全局 | 否 |
| innodb_undo_log_encrypt | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_undo_log_truncate | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_undo_tablespaces | 是 | 是 | 是 |  | 全局 | 不定 |
| Innodb_undo_tablespaces_active |  |  |  | 是 | 全局 | 否 |
| Innodb_undo_tablespaces_explicit |  |  |  | 是 | 全局 | 否 |
| Innodb_undo_tablespaces_implicit |  |  |  | 是 | 全局 | 否 |
| Innodb_undo_tablespaces_total |  |  |  | 是 | 全局 | 否 |
| innodb_use_fdatasync | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_use_native_aio | 是 | 是 | 是 |  | 全局 | 否 |
| innodb_validate_tablespace_paths | 是 | 是 | 是 |  | 全局 | 否 |
| innodb_version |  |  | 是 |  | 全局 | 否 |
| innodb_write_io_threads | 是 | 是 | 是 |  | 全局 | 否 |
| unique_checks |  |  | 是 |  | 两者 | 是 |
| 名称 | 命令行 | 选项文件 | 系统变量 | 状态变量 | 变量范围 | 动态 |

### InnoDB 命令选项

+   [`--innodb[=*`value`*]`](innodb-parameters.html#option_mysqld_innodb)

    | 命令行格式 | `--innodb[=value]` |
    | --- | --- |
    | 弃用 | 是 |
    | 类型 | 枚举 |
    | 默认值 | `ON` |
    | 有效值 | `OFF``ON``FORCE` |

    控制 `InnoDB` 存储引擎的加载，如果服务器编译时支持 `InnoDB`。此选项具有三态格式，可能的值为 `OFF`、`ON` 或 `FORCE`。请参阅 第 7.6.1 节，“安装和卸载插件”。

    要禁用 `InnoDB`，请使用 `--innodb=OFF` 或 `--skip-innodb`。在这种情况下，由于默认存储引擎是 `InnoDB`，除非还使用 `--default-storage-engine` 和 `--default-tmp-storage-engine` 将默认值设置为其他引擎，否则服务器不会启动，用于永久表和 `TEMPORARY` 表。

    `InnoDB` 存储引擎不再可以被禁用，`--innodb=OFF` 和 `--skip-innodb` 选项已被弃用且无效。使用它们会产生警告。预计这些选项将在未来的 MySQL 版本中被移除。

+   `--innodb-status-file`

    | 命令行格式 | `--innodb-status-file[={OFF&#124;ON}]` |
    | --- | --- |
    | 类型 | 布尔 |
    | 默认值 | `OFF` |

    `--innodb-status-file` 启动选项控制 `InnoDB` 是否在数据目录中创建一个名为 `innodb_status.*`pid`*` 的文件，并每隔约 15 秒将 `SHOW ENGINE INNODB STATUS` 输出到其中。

    默认情况下不会创建 `innodb_status.*`pid`*` 文件。要创建该文件，请使用 `--innodb-status-file` 选项启动 **mysqld**。`InnoDB` 在服务器正常关闭时会删除该文件。如果发生异常关闭，则可能需要手动删除状态文件。

    `--innodb-status-file`选项仅供临时使用，因为`SHOW ENGINE INNODB STATUS`输出生成可能会影响性能，并且随着时间的推移，`innodb_status.*pid`*`文件可能会变得非常大。

    有关相关信息，请参见第 17.17.2 节，“启用 InnoDB 监视器”。

+   `--skip-innodb`

    禁用`InnoDB`存储引擎。请参阅`--innodb`的描述。

### InnoDB 系统变量

+   `daemon_memcached_enable_binlog`

    | 命令行格式 | `--daemon-memcached-enable-binlog[={OFF&#124;ON}]` |
    | --- | --- |
    | 已弃用 | 8.0.22 |
    | 系统变量 | `daemon_memcached_enable_binlog` |
    | 作用范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    在源服务器上启用此选项以使用 MySQL 的`InnoDB` **memcached**插件（`daemon_memcached`）与 MySQL 的二进制日志。此选项只能在服务器启动时设置。您还必须使用`--log-bin`选项在源服务器上启用 MySQL 二进制日志。

    有关更多信息，请参见第 17.20.7 节，“InnoDB memcached 插件和复制”。

+   `daemon_memcached_engine_lib_name`

    | 命令行格式 | `--daemon-memcached-engine-lib-name=file_name` |
    | --- | --- |
    | 已弃用 | 8.0.22 |
    | 系统变量 | `daemon_memcached_engine_lib_name` |
    | 作用范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 文件名 |
    | 默认值 | `innodb_engine.so` |

    指定实现`InnoDB` **memcached**插件的共享库。

    有关更多信息，请参见第 17.20.3 节，“设置 InnoDB memcached 插件”。

+   `daemon_memcached_engine_lib_path`

    | 命令行格式 | `--daemon-memcached-engine-lib-path=dir_name` |
    | --- | --- |
    | 已弃用 | 8.0.22 |
    | 系统变量 | `daemon_memcached_engine_lib_path` |
    | 作用范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 目录名 |
    | 默认值 | `NULL` |

    包含实现`InnoDB` **memcached**插件的共享库的目录路径。默认值为 NULL，表示 MySQL 插件目录。除非指定位于 MySQL 插件目录之外的不同存储引擎的`memcached`插件，否则您不应该需要修改此参数。

    更多信息，请参阅第 17.20.3 节，“设置 InnoDB memcached 插件”。

+   `daemon_memcached_option`

    | 命令行格式 | `--daemon-memcached-option=options` |
    | --- | --- |
    | 已弃用 | 8.0.22 |
    | 系统变量 | `daemon_memcached_option` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 字符串 |
    | 默认值 |  |

    用于在启动时将空格分隔的**memcached**选项传递给底层**memcached**内存对象缓存守护程序。例如，您可以更改**memcached**侦听的端口，减少最大同时连接数，更改键值对的最大内存大小，或者为错误日志启用调试消息。

    有关使用详细信息，请参阅第 17.20.3 节，“设置 InnoDB memcached 插件”。有关**memcached**选项的信息，请参考**memcached**手册页。

+   `daemon_memcached_r_batch_size`

    | 命令行格式 | `--daemon-memcached-r-batch-size=#` |
    | --- | --- |
    | 已弃用 | 8.0.22 |
    | 系统变量 | `daemon_memcached_r_batch_size` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `1` |
    | 最小值 | `1` |
    | 最大值 | `1073741824` |

    指定在启动新事务之前执行多少个**memcached**读取操作（`get`操作）。与`daemon_memcached_w_batch_size`相对应。

    默认值为 1，因此通过 SQL 语句对表进行的任何更改都会立即对**memcached**操作可见。您可以增加它以减少在仅通过**memcached**接口访问底层表的系统上频繁提交的开销。如果将值设置得太大，则撤消或重做数据量可能会对存储造成一些开销，就像任何长时间运行的事务一样。

    更多信息，请参阅 第 17.20.3 节，“设置 InnoDB memcached 插件”。

+   `daemon_memcached_w_batch_size`

    | 命令行格式 | `--daemon-memcached-w-batch-size=#` |
    | --- | --- |
    | 已弃用 | 8.0.22 |
    | 系统变量 | `daemon_memcached_w_batch_size` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `1` |
    | 最小值 | `1` |
    | 最大值 | `1048576` |

    指定在启动新事务之前执行多少个**memcached**写操作，例如`add`、`set`和`incr`。与`daemon_memcached_r_batch_size`相对应。

    默认情况下，此值设置为 1，假设存储的数据在发生故障时很重要并且应立即提交。当存储非关键数据时，您可以增加此值以减少频繁提交的开销；但是如果发生意外退出，则最后*`N`*-1 个未提交的写操作可能会丢失。

    更多信息，请参阅 第 17.20.3 节，“设置 InnoDB memcached 插件”。

+   `innodb_adaptive_flushing`

    | 命令行格式 | `--innodb-adaptive-flushing[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `innodb_adaptive_flushing` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `ON` |

    指定是否根据工作负载动态调整`InnoDB`缓冲池中脏页的刷新速率。动态调整刷新速率旨在避免 I/O 活动的突发发生。此设置默认启用。有关更多信息，请参阅 第 17.8.3.5 节，“配置缓冲池刷新”。有关一般 I/O 调优建议，请参阅 第 10.5.8 节，“优化 InnoDB 磁盘 I/O”。

+   `innodb_adaptive_flushing_lwm`

    | 命令行格式 | `--innodb-adaptive-flushing-lwm=#` |
    | --- | --- |
    | 系统变量 | `innodb_adaptive_flushing_lwm` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `10` |
    | 最小值 | `0` |
    | 最大值 | `70` |

    定义表示重做日志容量百分比的低水位标记，在此百分比下启用自适应刷新。有关更多信息，请参阅第 17.8.3.5 节，“配置缓冲池刷新”。

+   `innodb_adaptive_hash_index`

    | 命令行格式 | `--innodb-adaptive-hash-index[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `innodb_adaptive_hash_index` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `ON` |

    `InnoDB` 自适应哈希索引是否启用或禁用。根据您的工作负载，动态启用或禁用自适应哈希索引可能是有益的，以提高查询性能。由于自适应哈希索引可能并非适用于所有工作负载，因此请使用真实工作负载分别启用和禁用它进行基准测试。有关详细信息，请参阅第 17.5.3 节，“自适应哈希索引”。

    此变量默认启用。您可以使用`SET GLOBAL`语句修改此参数，无需重新启动服务器。在运行时更改设置需要具有足够权限设置全局系统变量。请参阅第 7.1.9.1 节，“系统变量权限”。您也可以在服务器启动时使用`--skip-innodb-adaptive-hash-index`来禁用它。

    禁用自适应哈希索引会立即清空哈希表。在清空哈希表的同时，正常操作可以继续进行，并且执行使用哈希表的查询将直接访问索引 B 树。重新启用自适应哈希索引时，在正常操作期间哈希表将再次填充。

+   `innodb_adaptive_hash_index_parts`

    | 命令行格式 | `--innodb-adaptive-hash-index-parts=#` |
    | --- | --- |
    | 系统变量 | `innodb_adaptive_hash_index_parts` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 数值 |
    | 默认值 | `8` |
    | 最小值 | `1` |
    | 最大值 | `512` |

    对自适应哈希索引搜索系统进行分区。每个索引绑定到特定分区，每个分区由单独的闩保护。

    自适应哈希索引搜索系统默认分为 8 部分。最大设置为 512。

    有关相关信息，请参见第 17.5.3 节，“自适应哈希索引”。

+   `innodb_adaptive_max_sleep_delay`

    | 命令行格式 | `--innodb-adaptive-max-sleep-delay=#` |
    | --- | --- |
    | 系统变量 | `innodb_adaptive_max_sleep_delay` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `150000` |
    | 最小值 | `0` |
    | 最大值 | `1000000` |
    | 单位 | 微秒 |

    允许`InnoDB`根据当前工作负载自动调整`innodb_thread_sleep_delay`的值。任何非零值都会启用`innodb_thread_sleep_delay`值的自动动态调整，最高值不超过`innodb_adaptive_max_sleep_delay`选项中指定的最大值。该值表示微秒数。此选项在繁忙系统中非常有用，具有超过 16 个`InnoDB`线程。（实际上，对于具有数百或数千个同时连接的 MySQL 系统来说，这是最有价值的。）

    有关更多信息，请参见第 17.8.4 节，“配置 InnoDB 的线程并发性”。

+   `innodb_api_bk_commit_interval`

    | 命令行格式 | `--innodb-api-bk-commit-interval=#` |
    | --- | --- |
    | 已弃用 | 8.0.22 |
    | 系统变量 | `innodb_api_bk_commit_interval` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `5` |
    | 最小值 | `1` |
    | 最大值 | `1073741824` |
    | 单位 | 秒 |

    自动提交使用`InnoDB` **memcached**接口的空闲连接的频率，单位为秒。更多信息，请参见第 17.20.6.4 节，“控制 InnoDB memcached 插件的事务行为”。

+   `innodb_api_disable_rowlock`

    | 命令行格式 | `--innodb-api-disable-rowlock[={OFF&#124;ON}]` |
    | --- | --- |
    | 已弃用 | 8.0.22 |
    | 系统变量 | `innodb_api_disable_rowlock` |
    | 作用范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    使用此选项禁用 `InnoDB` **memcached** 执行 DML 操作时的行锁。默认情况下，`innodb_api_disable_rowlock` 处于禁用状态，这意味着 **memcached** 请求行锁用于 `get` 和 `set` 操作。当启用 `innodb_api_disable_rowlock` 时，**memcached** 请求表锁而不是行锁。

    `innodb_api_disable_rowlock` 不是动态的。必须在**mysqld**命令行上指定，或者输入到 MySQL 配置文件中。配置在插件安装时生效，插件安装发生在 MySQL 服务器启动时。

    更多信息，请参见 Section 17.20.6.4, “控制 InnoDB memcached 插件的事务行为”。

+   `innodb_api_enable_binlog`

    | 命令行格式 | `--innodb-api-enable-binlog[={OFF&#124;ON}]` |
    | --- | --- |
    | 已弃用 | 8.0.22 |
    | 系统变量 | `innodb_api_enable_binlog` |
    | 作用范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    允许您使用 MySQL 二进制日志 的 `InnoDB` **memcached** 插件。更多信息，请参见 启用 InnoDB memcached 二进制日志。

+   `innodb_api_enable_mdl`

    | 命令行格式 | `--innodb-api-enable-mdl[={OFF&#124;ON}]` |
    | --- | --- |
    | ���弃用 | 8.0.22 |
    | 系统变量 | `innodb_api_enable_mdl` |
    | 作用范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    锁定`InnoDB` **memcached** 插件使用的表，以防止通过 SQL 接口的 DDL 删除或更改。更多信息，请参见 Section 17.20.6.4, “控制 InnoDB memcached 插件的事务行为”。

+   `innodb_api_trx_level`

    | 命令行格式 | `--innodb-api-trx-level=#` |
    | --- | --- |
    | 已弃用 | 8.0.22 |
    | 系统变量 | `innodb_api_trx_level` |
    | 作用域 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `0` |
    | 最小值 | `0` |
    | 最大值 | `3` |

    控制由**memcached**接口处理的查询的事务隔离级别。对应于熟悉名称的常量为：

    +   0 = `READ UNCOMMITTED`

    +   1 = `READ COMMITTED`

    +   2 = `REPEATABLE READ`

    +   3 = `SERIALIZABLE`

    有关更多信息，请参阅第 17.20.6.4 节，“控制 InnoDB memcached 插件的事务行为”。

+   `innodb_autoextend_increment`

    | 命令行格式 | `--innodb-autoextend-increment=#` |
    | --- | --- |
    | 系统变量 | `innodb_autoextend_increment` |
    | 作用域 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `64` |
    | 最小值 | `1` |
    | 最大值 | `1000` |
    | 单位 | 兆字节 |

    当`InnoDB` 系统表空间文件满时，自动扩展大小的增量大小（以兆字节为单位）。默认值为 64。有关相关信息，请参阅系统表空间数据文件配置和调整系统表空间大小。

    `innodb_autoextend_increment` 设置不影响 file-per-table 表空间文件或 general tablespace 文件。这些文件会自动扩展，不受`innodb_autoextend_increment`设置的影响。初始扩展量较小，之后每次扩展增加 4MB。

+   `innodb_autoinc_lock_mode`

    | 命令行格式 | `--innodb-autoinc-lock-mode=#` |
    | --- | --- |
    | 系统变量 | `innodb_autoinc_lock_mode` |
    | 作用范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `2` |
    | 有效值 | `0``1``2` |

    用于生成自增值的锁定模式。允许的值为 0、1 或 2，分别表示传统、连续或交错。

    截至 MySQL 8.0，默认设置为 2（交错），之前为 1（连续）。从语句为基础的复制变为行为基础的复制作为默认复制类型的变化反映在默认设置为交错锁定模式上，这发生在 MySQL 5.7 中。语句为基础的复制需要连续的自增锁定模式，以确保自增值按照给定 SQL 语句序列的可预测和可重复的顺序分配，而行为基础的复制不受 SQL 语句执行顺序的影响。

    有关每种锁定模式的特性，请参阅 InnoDB AUTO_INCREMENT Lock Modes。 

+   `innodb_background_drop_list_empty`

    | 命令行格式 | `--innodb-background-drop-list-empty[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `innodb_background_drop_list_empty` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    启用`innodb_background_drop_list_empty`调试选项有助于避免测试用例失败，延迟表的创建直到后台删除列表为空。例如，如果测试用例 A 将表`t1`放在后台删除列表中，测试用例 B 将等待直到后台删除列表为空才创建表`t1`。

+   `innodb_buffer_pool_chunk_size`

    | 命令行格式 | `--innodb-buffer-pool-chunk-size=#` |
    | --- | --- |
    | 系统变量 | `innodb_buffer_pool_chunk_size` |
    | 作用范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `134217728` |
    | 最小值 | `1048576` |
    | 最大值 | `innodb_buffer_pool_size / innodb_buffer_pool_instances` |
    | 单位 | 字节 |

    `innodb_buffer_pool_chunk_size`定义了`InnoDB`缓冲池调整大小操作的块大小。

    为避免在调整大小操作期间复制所有缓冲池页面，操作是以“块”进行的。默认情况下，`innodb_buffer_pool_chunk_size`为 128MB（134217728 字节）。块中包含的页面数量取决于`innodb_page_size`的值。`innodb_buffer_pool_chunk_size`可以以 1MB（1048576 字节）的单位增加或减少。

    更改`innodb_buffer_pool_chunk_size`值时应满足以下条件：

    +   如果在初始化缓冲池时，`innodb_buffer_pool_chunk_size` * `innodb_buffer_pool_instances`大于当前缓冲池大小，则`innodb_buffer_pool_chunk_size`会被截断为`innodb_buffer_pool_size` / `innodb_buffer_pool_instances`。

    +   缓冲池大小必须始终等于或是`innodb_buffer_pool_chunk_size` * `innodb_buffer_pool_instances`的倍数。如果更改`innodb_buffer_pool_chunk_size`，`innodb_buffer_pool_size`会自动舍入为等于或是`innodb_buffer_pool_chunk_size` * `innodb_buffer_pool_instances`的值。此调整发生在初始化缓冲池时。

    重要

    更改`innodb_buffer_pool_chunk_size`时需谨慎，因为更改此值可能会自动增加缓冲池的大小。在更改`innodb_buffer_pool_chunk_size`之前，计算其对`innodb_buffer_pool_size`的影响，以确保生成的缓冲池大小是可接受的。

    为避免潜在的性能问题，块的数量(`innodb_buffer_pool_size` / `innodb_buffer_pool_chunk_size`)不应超过 1000。

    `innodb_buffer_pool_size` 变量是动态的，允许在服务器在线时调整缓冲池的大小。然而，缓冲池的大小必须等于或是 `innodb_buffer_pool_chunk_size` * `innodb_buffer_pool_instances` 的倍数，改变这两个变量设置中的任何一个都需要重新启动服务器。

    更多信息请参见 第 17.8.3.1 节，“配置 InnoDB 缓冲池大小”。

+   `innodb_buffer_pool_debug`

    | 命令行格式 | `--innodb-buffer-pool-debug[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `innodb_buffer_pool_debug` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    启用此选项允许在缓冲池小于 1GB 时存在多个缓冲池实例，忽略对 `innodb_buffer_pool_instances` 强加的 1GB 最小缓冲池大小约束。只有在使用 `WITH_DEBUG` **CMake** 选项编译时才可用 `innodb_buffer_pool_debug` 选项。

+   `innodb_buffer_pool_dump_at_shutdown`

    | 命令行格式 | `--innodb-buffer-pool-dump-at-shutdown[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `innodb_buffer_pool_dump_at_shutdown` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `ON` |

    指定在 MySQL 服务器关闭时记录缓存在 `InnoDB` 缓冲池 中的页面，以缩短下次重启时的 预热 过程。通常与 `innodb_buffer_pool_load_at_startup` 结合使用。`innodb_buffer_pool_dump_pct` 选项定义要转储的最近使用的缓冲池页面的百分比。

    `innodb_buffer_pool_dump_at_shutdown` 和 `innodb_buffer_pool_load_at_startup` 默认启用。

    更多信息，请参阅 Section 17.8.3.6，“保存和恢复缓冲池状态”。

+   `innodb_buffer_pool_dump_now`

    | 命令行格式 | `--innodb-buffer-pool-dump-now[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `innodb_buffer_pool_dump_now` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    立即记录在`InnoDB`缓冲池中缓存的页面。通常与`innodb_buffer_pool_load_now`结合使用。

    启用`innodb_buffer_pool_dump_now`会触发记录操作，但不会改变变量设置，该设置始终保持`OFF`或`0`。要在触发转储后查看缓冲池转储状态，请查询`Innodb_buffer_pool_dump_status`变量。

    启用`innodb_buffer_pool_dump_now`会触发转储操作，但不会改变变量设置，该设置始终保持`OFF`或`0`。要在触发转储后查看缓冲池转储状态，请查询`Innodb_buffer_pool_dump_status`变量。

    更多信息，请参阅 Section 17.8.3.6，“保存和恢复缓冲池状态”。

+   `innodb_buffer_pool_dump_pct`

    | 命令行格式 | `--innodb-buffer-pool-dump-pct=#` |
    | --- | --- |
    | 系统变量 | `innodb_buffer_pool_dump_pct` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `25` |
    | 最小值 | `1` |
    | ��大值 | `100` |

    指定每个缓冲池中最近使用的页面的百分比以读取并转储。范围为 1 到 100。默认值为 25。例如，如果有 4 个每个有 100 页的缓冲池，并且`innodb_buffer_pool_dump_pct`设置为 25，则从每个缓冲池中转储最近使用的 25 页。

+   `innodb_buffer_pool_filename`

    | 命令行格式 | `--innodb-buffer-pool-filename=file_name` |
    | --- | --- |
    | 系统变量 | `innodb_buffer_pool_filename` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 文件名 |
    | 默认值 | `ib_buffer_pool` |

    指定保存由`innodb_buffer_pool_dump_at_shutdown`或`innodb_buffer_pool_dump_now`生成的表空间 ID 和页面 ID 列表的文件的名称。表空间 ID 和页面 ID 以以下格式保存：`space, page_id`。默认情况下，文件名为`ib_buffer_pool`，位于`InnoDB`数据目录中。必须相对于数据目录指定非默认位置。

    可以在运行时使用`SET`语句指定文件名：

    ```sql
    SET GLOBAL innodb_buffer_pool_filename=*'file_name'*;
    ```

    您还可以在启动时指定文件名，在启动字符串或 MySQL 配置文件中。在启动时指定文件名时，文件必须存在，否则`InnoDB`会返回启动错误，指示没有这样的文件或目录。

    有关更多信息，请参见第 17.8.3.6 节，“保存和恢复缓冲池状态”。

+   `innodb_buffer_pool_in_core_file`

    | 命令行格式 | `--innodb-buffer-pool-in-core-file[={OFF&#124;ON}]` |
    | --- | --- |
    | 引入版本 | 8.0.14 |
    | 系统变量 | `innodb_buffer_pool_in_core_file` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `ON` |

    禁用`innodb_buffer_pool_in_core_file`变量可通过排除`InnoDB`缓冲池页面来减小核心文件的大小。要使用此变量，必须启用`core_file`变量，并且操作系统必须支持`MADV_DONTDUMP`非 POSIX 扩展到`madvise()`，该扩展在 Linux 3.4 及更高版本中受支持。有关更多信息，请参见第 17.8.3.7 节，“从核心文件中排除缓冲池页面”。

+   `innodb_buffer_pool_instances`

    | 命令行格式 | `--innodb-buffer-pool-instances=#` |
    | --- | --- |
    | 系统变量 | `innodb_buffer_pool_instances` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值（Windows，32 位平台） | `(autosized)` |
    | 默认值（其他） | `8（如果 innodb_buffer_pool_size < 1GB，则为 1）` |
    | 最小值 | `1` |
    | 最大值 | `64` |

    `InnoDB` 缓冲池 分成的区域数量。对于具有多个千兆字节范围的缓冲池的系统，将缓冲池分成单独的实例可以通过减少不同线程读取和写入缓存页面时的争用来提高并发性。存储在缓冲池中或从缓冲池中读取的每个页面都会随机分配给缓冲池实例之一，使用哈希函数。每个缓冲池管理自己的空闲列表、刷新列表、LRU 以及与缓冲池相关的所有其他数据结构，并由自己的缓冲池互斥锁保护。

    当将`innodb_buffer_pool_size`设置为 1GB 或更高时，此选项才会生效。总缓冲池大小将分配给所有缓冲池。为了达到最佳效率，请指定`innodb_buffer_pool_instances`和`innodb_buffer_pool_size`的组合，以便每个缓冲池实例至少为 1GB。

    在 32 位 Windows 系统上的默认值取决于`innodb_buffer_pool_size`的值，如下所述：

    +   如果`innodb_buffer_pool_size`大于 1.3GB，则`innodb_buffer_pool_instances`的默认值为`innodb_buffer_pool_size`/128MB，每个块的内存分配请求。选择 1.3GB 作为边界是因为在此处，32 位 Windows 无法为单个缓冲池分配所需的连续地址空间存在显著风险。

    +   否则，默认值为 1。

    在所有其他平台上，当`innodb_buffer_pool_size`大于或等于 1GB 时，默认值为 8。否则，默认值为 1。

    有关相关信息，请参阅第 17.8.3.1 节，“配置 InnoDB 缓冲池大小”。

+   `innodb_buffer_pool_load_abort`

    | 命令行格式 | `--innodb-buffer-pool-load-abort[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `innodb_buffer_pool_load_abort` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    中断由`innodb_buffer_pool_load_at_startup`或`innodb_buffer_pool_load_now`触发的`InnoDB`缓冲池内容恢复过程。

    启用`innodb_buffer_pool_load_abort`会触发中止操作，但不会改变变量设置，其始终保持为`OFF`或`0`。要在触发中止操作后查看缓冲池加载状态，请查询`Innodb_buffer_pool_load_status`变量。

    有关更多信息，请参见第 17.8.3.6 节，“保存和恢复缓冲池状态”。

+   `innodb_buffer_pool_load_at_startup`

    | 命令行格式 | `--innodb-buffer-pool-load-at-startup[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `innodb_buffer_pool_load_at_startup` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `ON` |

    指定在 MySQL 服务器启动时，`InnoDB`缓冲池会自动通过加载先前保存的页面来预热。通常与`innodb_buffer_pool_dump_at_shutdown`一起使用。

    `innodb_buffer_pool_dump_at_shutdown`和`innodb_buffer_pool_load_at_startup`默认启用。

    有关更多信息，请参见第 17.8.3.6 节，“保存和恢复缓冲池状态”。

+   `innodb_buffer_pool_load_now`

    | 命令行格式 | `--innodb-buffer-pool-load-now[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `innodb_buffer_pool_load_now` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    通过加载数据页立即 预热 `InnoDB` 缓冲池，无需等待服务器重新启动。在进行基准测试期间，或在运行报告或维护查询后准备 MySQL 服务器恢复其正常工作负载时，这可能很有用。

    启用 `innodb_buffer_pool_load_now` 触发加载操作，但不会改变变量设置，变量始终保持 `OFF` 或 `0`。在触发加载后查看缓冲池加载进度，请查询 `Innodb_buffer_pool_load_status` 变量。

    更多信息，请参阅 Section 17.8.3.6, “保存和恢复缓冲池状态”。

+   `innodb_buffer_pool_size`

    | 命令行格式 | `--innodb-buffer-pool-size=#` |
    | --- | --- |
    | 系统变量 | `innodb_buffer_pool_size` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `134217728` |
    | 最小值 | `5242880` |
    | 最大值（64 位平台） | `2**64-1` |
    | 最大值（32 位平台） | `2**32-1` |
    | 单位 | 字�� |

    缓冲池的大小（以字节为单位），即 `InnoDB` 缓存表和索引数据的内存区域。默认值为 134217728 字节（128MB）。最大值取决于 CPU 架构；32 位系统上的最大值为 4294967295（2³²-1），64 位系统上的最大值为 18446744073709551615（2⁶⁴-1）。在 32 位系统上，CPU 架构和操作系统可能会强加一个比规定最大值更低的实际最大值。当缓冲池的大小大于 1GB 时，将 `innodb_buffer_pool_instances` 设置为大于 1 的值可以提高繁忙服务器的可伸缩性。

    更大的缓冲池需要更少的磁盘 I/O 来多次访问相同的表数据。在专用数据库服务器上，您可能会将缓冲池大小设置为机器物理内存大小的 80%。在配置缓冲池大小时，请注意以下潜在问题，并准备根据需要缩小缓冲池的大小。

    +   物理内存的竞争可能导致操作系统进行分页。

    +   `InnoDB` 为缓冲区和控制结构保留额外的内存，因此总分配空间大约比指定的缓冲池大小大约多 10%。

    +   缓冲池的地址空间必须是连续的，在 Windows 系统中，DLL 可能会加载到特定地址，这可能会成为一个问题。

    +   初始化缓冲池的时间大致与其大小成正比。在具有大型缓冲池的实例上，初始化时间可能很显著。为了缩短初始化时间，您可以在服务器关闭时保存缓冲池状态，并在服务器启动时恢复它。参见第 17.8.3.6 节，“保存和恢复缓冲池状态”。

    当您增加或减少缓冲池大小时，操作是以块为单位执行的。块大小由`innodb_buffer_pool_chunk_size`变量定义，默认值为 128 MB。

    缓冲池大小必须始终等于或是`innodb_buffer_pool_chunk_size` * `innodb_buffer_pool_instances`的倍数。如果您将缓冲池大小更改为不等于或不是`innodb_buffer_pool_chunk_size` * `innodb_buffer_pool_instances`的倍数的值，则缓冲池大小会自动调整为等于或是`innodb_buffer_pool_chunk_size` * `innodb_buffer_pool_instances`的倍数的值。

    `innodb_buffer_pool_size`可以动态设置，这允许您在不重启服务器的情况下调整缓冲池大小。`Innodb_buffer_pool_resize_status`状态变量报告在线缓冲池调整操作的状态。有关更多信息，请参见第 17.8.3.1 节，“配置 InnoDB 缓冲池大小”。

    如果启用了`innodb_dedicated_server`，并且未明确定义`innodb_buffer_pool_size`的值，则该值会自动配置。有关更多信息，请参见第 17.8.12 节，“为专用 MySQL 服务器启用自动配置”��

+   `innodb_change_buffer_max_size`

    | 命令行格式 | `--innodb-change-buffer-max-size=#` |
    | --- | --- |
    | 系统变量 | `innodb_change_buffer_max_size` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `25` |
    | 最小值 | `0` |
    | 最大值 | `50` |

    `InnoDB` 更改缓冲区的最大大小，作为缓冲池总大小的百分比。您可能会增加此值，用于具有大量插入、更新和删除活动的 MySQL 服务器，或者减少用于报告中使用的数据保持不变的 MySQL 服务器。有关更多信息，请参见第 17.5.2 节，“更改缓冲区”。有关一般 I/O 调优建议，请参见第 10.5.8 节，“优化 InnoDB 磁盘 I/O”。

+   `innodb_change_buffering`

    | 命令行格式 | `--innodb-change-buffering=value` |
    | --- | --- |
    | 系统变量 | `innodb_change_buffering` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 枚举 |
    | 默认值 | `all` |
    | 有效值 | `none``inserts``deletes``changes``purges``all` |

    `InnoDB` 是否执行更改缓冲，这是一种优化，延迟写入操作到次要索引，以便 I/O 操作可以按顺序执行。允许的值在下表中描述。值也可以用数字指定。

    **表 17.25 innodb_change_buffering 允许的值**

    | 值 | 数值 | 描述 |
    | --- | --- | --- |
    | `none` | `0` | 不缓冲任何操作。 |
    | `inserts` | `1` | 缓冲插入操作。 |
    | `deletes` | `2` | 缓冲删除标记操作；严格来说，标记索引记录以便稍后在清除操作期间删除。 |
    | `changes` | `3` | 缓冲插入和删除标记操作。 |
    | `purges` | `4` | 缓冲后台中发生的物理删除操作。 |
    | `all` | `5` | 默认值。缓冲插入、删除标记操作和清除。 |

    有关更多信息，请参见第 17.5.2 节，“更改缓冲区”。有关一般 I/O 调优建议，请参见第 10.5.8 节，“优化 InnoDB 磁盘 I/O”。

+   `innodb_change_buffering_debug`

    | 命令行格式 | `--innodb-change-buffering-debug=#` |
    | --- | --- |
    | 系统变量 | `innodb_change_buffering_debug` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `0` |
    | 最小值 | `0` |
    | 最大值 | `2` |

    设置`InnoDB`更改缓冲区的调试标志。值为 1 会强制所有更改进入更改缓冲区。值为 2 会在合并时导致意外退出。默认值为 0 表示更改缓冲区调试标志未设置。此选项仅在使用`WITH_DEBUG` **CMake** 选项编译调试支持时才可用。

+   `innodb_checkpoint_disabled`

    | 命令行格式 | `--innodb-checkpoint-disabled[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `innodb_checkpoint_disabled` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    这是一个仅供专家调试使用的调试选项。它禁用了检查点，以便有意的服务器退出始终会启动`InnoDB`恢复。通常在运行写入重做日志条目的 DML 操作之前，应该仅启用它一小段时间。此选项仅在使用`WITH_DEBUG` **CMake** 选项编译调试支持时才可用。

+   `innodb_checksum_algorithm`

    | 命令行格式 | `--innodb-checksum-algorithm=value` |
    | --- | --- |
    | 系统变量 | `innodb_checksum_algorithm` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 枚举 |
    | 默认值 | `crc32` |
    | 有效值 | `crc32``strict_crc32``innodb``strict_innodb``none``strict_none` |

    指定如何生成和验证`InnoDB`表空间中磁盘块中存储的校验和。`innodb_checksum_algorithm`的默认值为`crc32`。

    MySQL 企业备份版本直到 3.8.0 不支持备份使用 CRC32 校验和的表空间。MySQL 企业备份在 3.8.1 中添加了 CRC32 校验和支持，但有一些限制。有关更多信息，请参考 MySQL 企业备份 3.8.1 变更历史。

    值`innodb`向后兼容早期版本的 MySQL。值`crc32`使用一种更快的算法来计算每个修改块的校验和，并检查每个磁盘读取的校验和。它每次扫描 64 位块，比每次扫描 8 位块的`innodb`校验算法更快。值`none`在校验字段中写入一个常数值，而不是基于块数据计算值。表空间中的块可以使用旧、新和无校验和值的混合，随着数据逐渐修改而逐步更新；一旦表空间中的块被修改为使用`crc32`算法，相关表将无法被早期版本的 MySQL 读取。

    校验算法的严格形式在表空间中遇到有效但不匹配的校验和值时会报错。建议您只在新实例中使用严格设置，首次设置表空间。严格设置略快，因为在磁盘读取期间不需要计算所有校验和值。

    以下表格显示了`none`、`innodb`和`crc32`选项值及其严格对应项之间的区别。`none`、`innodb`和`crc32`将指定类型的校验和值写入每个数据块，但在验证读取操作期间的块时，也接受其他校验和值以确保兼容性。严格设置还接受有效的校验和值，但在遇到有效但不匹配的校验和值时会打印错误消息。如果实例中的所有`InnoDB`数据文件都是在相同的`innodb_checksum_algorithm`值下创建的，则使用严格形式可以使验证更快。

    **表 17.26 允许的 innodb_checksum_algorithm 值**

    | 值 | 生成的校验和（写入时） | 允许的校验和（读取时） |
    | --- | --- | --- |
    | none | 一个常数值。 | 由`none`、`innodb`或`crc32`生成的任何校验和。 |
    | innodb | 使用`InnoDB`原始算法在软件中计算的校验和。 | 由`none`、`innodb`或`crc32`生成的任何校验和。 |
    | crc32 | 使用`crc32`算法计算的校验和，可能使用硬件辅助完成。 | 由`none`、`innodb`或`crc32`生成的任何校验和。 |
    | strict_none | 一个常数值 | 由`none`、`innodb`或`crc32`生成的任何校验和。如果遇到有效但不匹配的校验和，`InnoDB`会打印错误消息。 |
    | strict_innodb | 使用`InnoDB`原始算法在软件中计算的校验和。 | 由`none`、`innodb`或`crc32`生成的任何校验和。如果遇到有效但不匹配的校验和，`InnoDB`会打印错误消息。 |
    | strict_crc32 | 使用`crc32`算法计算的校验和，可能使用硬件辅助完成。 | 由`none`、`innodb`或`crc32`生成的任何校验和。如果遇到有效但不匹配的校验和，`InnoDB`会打印错误消息。 |

+   `innodb_cmp_per_index_enabled`

    | 命令行格式 | `--innodb-cmp-per-index-enabled[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `innodb_cmp_per_index_enabled` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    启用信息模式`INNODB_CMP_PER_INDEX`表中每个索引的压缩相关统计信息。由于这些统计信息可能很昂贵，只在与`InnoDB` 压缩表相关的性能调优期间的开发、测试或副本实例上启用此选项。

    更多信息，请参阅第 28.4.8 节，“INFORMATION_SCHEMA INNODB_CMP_PER_INDEX 和 INNODB_CMP_PER_INDEX_RESET 表”，以及第 17.9.1.4 节，“运行时监视 InnoDB 表压缩”。

+   `innodb_commit_concurrency`

    | 命令行格式 | `--innodb-commit-concurrency=#` |
    | --- | --- |
    | 系统变量 | `innodb_commit_concurrency` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `0` |
    | 最小值 | `0` |
    | 最大值 | `1000` |

    可以同时提交的线程数量。值为 0（默认值）允许任意数量的事务同时提交。

    `innodb_commit_concurrency`的值不能在运行时从零更改为非零或反之。可以从一个非零值更改为另一个非零值。

+   `innodb_compress_debug`

    | 命令行格式 | `--innodb-compress-debug=value` |
    | --- | --- |
    | 系统变量 | `innodb_compress_debug` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 枚举 |
    | 默认值 | `none` |
    | 有效值 | `none``zlib``lz4``lz4hc` |

    使用指定的压缩算法压缩所有表，而无需为每个表定义`COMPRESSION`属性。此选项仅在使用`WITH_DEBUG` **CMake** 选项编译调试支持时才可用。

    有关更多信息，请参阅第 17.9.2 节，“InnoDB 页面压缩”。

+   `innodb_compression_failure_threshold_pct`

    | 命令行格式 | `--innodb-compression-failure-threshold-pct=#` |
    | --- | --- |
    | 系统变量 | `innodb_compression_failure_threshold_pct` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `5` |
    | 最小值 | `0` |
    | 最大值 | `100` |

    定义表的压缩失败率阈值，以百分比表示，在此阈值之上，MySQL 开始在压缩页面内添加填充，以避免昂贵的压缩失败。当超过此阈值时，MySQL 开始在每个新的压缩页面内留下额外的空闲空间，动态调整空闲空间的量，直到达到由`innodb_compression_pad_pct_max`指定的页面大小百分比。值为零会禁用监视压缩效率并动态调整填充量的机制。

    有关更多信息，请参阅第 17.9.1.6 节，“OLTP 工作负载的压缩”。

+   `innodb_compression_level`

    | 命令行格式 | `--innodb-compression-level=#` |
    | --- | --- |
    | 系统变量 | `innodb_compression_level` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `6` |
    | 最小值 | `0` |
    | 最大值 | `9` |

    指定用于`InnoDB`压缩表和索引的 zlib 压缩级别。较高的值可以让您将更多数据放入存储设备，但会增加压缩时的 CPU 开销。较低的值可以减少 CPU 开销，当存储空间不是关键问题，或者您预计数据不太容易压缩时使用。

    更多信息，请参阅第 17.9.1.6 节，“OLTP 工作负载的压缩”。

+   `innodb_compression_pad_pct_max`

    | 命令行格式 | `--innodb-compression-pad-pct-max=#` |
    | --- | --- |
    | 系统变量 | `innodb_compression_pad_pct_max` |
    | 作用域 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `50` |
    | 最小值 | `0` |
    | 最大值 | `75` |

    指定每个压缩页内可保留为自由空间的最大百分比，以便在更新压缩表或索引时重新组织数据和修改日志，并在数据可能被重新压缩时为页内留出空间。仅在`innodb_compression_failure_threshold_pct`设置为非零值，并且压缩失败的速率超过截止点时才适用。

    更多信息，请参阅第 17.9.1.6 节，“OLTP 工作负载的压缩”。

+   `innodb_concurrency_tickets`

    | 命令行格式 | `--innodb-concurrency-tickets=#` |
    | --- | --- |
    | 系统变量 | `innodb_concurrency_tickets` |
    | 作用域 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `5000` |
    | 最小值 | `1` |
    | 最大值 | `4294967295` |

    确定可以同时进入`InnoDB`的线程数量。当尝试进入`InnoDB`的线程数已达到并发限制时，线程将被放置在队列中。当线程被允许进入`InnoDB`时，它将获得与`innodb_concurrency_tickets`值相等的“票”，并且线程可以自由进入和离开`InnoDB`直到使用完票为止。在那之后，线程再次成为并发检查的对象（可能排队），下次尝试进入`InnoDB`时。默认值为 5000。

    具有较小的`innodb_concurrency_tickets`值时，只需要处理少量行的小事务与处理许多行的大事务公平竞争。较小的`innodb_concurrency_tickets`值的缺点是，大型事务必须多次遍历队列才能完成，这会延长完成任务所需的时间。

    具有较大的`innodb_concurrency_tickets`值，大型事务等待队列末尾位置的时间较短（由`innodb_thread_concurrency`控制），更多时间用于检索行。大型事务还需要较少的队列遍历次数才能完成任务。较大的`innodb_concurrency_tickets`值的缺点是，同时运行太多大型事务可能会通过使它们等待更长时间来执行，使较小事务饥饿。

    具有非零的`innodb_thread_concurrency`值时，您可能需要调整`innodb_concurrency_tickets`值，以找到较大和较小事务之间的最佳平衡。`SHOW ENGINE INNODB STATUS`报告显示了当前通过队列执行事务时剩余的票数。此数据也可以从信息模式`INNODB_TRX`表的`TRX_CONCURRENCY_TICKETS`列中获取。

    有关更多信息，请参见第 17.8.4 节，“配置 InnoDB 的线程并发性”。

+   `innodb_data_file_path`

    | 命令行格式 | `--innodb-data-file-path=file_name` |
    | --- | --- |
    | 系统变量 | `innodb_data_file_path` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 字符串 |
    | 默认值 | `ibdata1:12M:autoextend` |

    定义`InnoDB`系统表空间数据文件的名称、大小和属性。如果未为`innodb_data_file_path`指定值，则默认行为是创建一个稍大于 12MB 的单个自动扩展数据文件，命名为`ibdata1`。

    数据文件规范的完整语法包括文件名、文件大小、`autoextend`属性和`max`属性：

    ```sql
    *file_name*:*file_size*[:autoextend[:max:*max_file_size*]]
    ```

    文件大小通过在大小值后附加`K`、`M`或`G`来指定为千字节、兆字节或千兆字节。如果以千字节指定数据文件大小，请以 1024 的倍数进行。否则，KB 值将四舍五入到最近的兆字节（MB）边界。文件大小之和必须至少略大于 12MB。

    有关其他配置信息，请参阅系统表空间数据文件配置。有关调整大小的说明，请参阅调整系统表空间大小。

+   `innodb_data_home_dir`

    | 命令行格式 | `--innodb-data-home-dir=dir_name` |
    | --- | --- |
    | 系统变量 | `innodb_data_home_dir` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 目录名称 |

    `InnoDB`系统表空间数据文件的目录路径的公共部分。默认值为 MySQL 的`data`目录。该设置与`innodb_data_file_path`设置连接在一起，除非该设置使用绝对路径定义。

    在为`innodb_data_home_dir`指定值时，需要添加尾随斜杠。例如：

    ```sql
    [mysqld]
    innodb_data_home_dir = /path/to/myibdata/
    ```

    此设置不影响 file-per-table 表空间的位置。

    有关信息，请参阅第 17.8.1 节，“InnoDB 启动配置”。

+   `innodb_ddl_buffer_size`

    | 命令行格式 | `--innodb-ddl-buffer-size=#` |
    | --- | --- |
    | 引入版本 | 8.0.27 |
    | 系统变量 | `innodb_ddl_buffer_size` |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `1048576` |
    | 最小值 | `65536` |
    | 最大值 | `4294967295` |
    | 单位 | 字节 |

    定义了 DDL 操作的最大缓冲区大小。默认设置为 1048576 字节（约 1 MB）。适用于创建或重建二级索引的在线 DDL 操作。请参见第 17.12.4 节，“在线 DDL 内存管理”。每个 DDL 线程的最大缓冲区大小是最大缓冲区大小除以 DDL 线程数（`innodb_ddl_buffer_size`/`innodb_ddl_threads`）。

+   `innodb_ddl_log_crash_reset_debug`

    | 命令行格式 | `--innodb-ddl-log-crash-reset-debug[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `innodb_ddl_log_crash_reset_debug` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` Hint Applies | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    启用此调试选项可将 DDL 日志崩溃注入计数器重置为 1。此选项仅在使用`WITH_DEBUG` **CMake** 选项编译时才可用。

+   `innodb_ddl_threads`

    | 命令行格式 | `--innodb-ddl-threads=#` |
    | --- | --- |
    | 引入版本 | 8.0.27 |
    | 系统变量 | `innodb_ddl_threads` |
    | 作用范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR` Hint Applies | 否 |
    | 类型 | 整数 |
    | 默认值 | `4` |
    | 最小值 | `1` |
    | 最大值 | `64` |

    定义了索引创建的排序和构建阶段的最大并行线程数。适用于创建或重建二级索引的在线 DDL 操作。有关更多信息，请参见第 17.12.5 节，“配置在线 DDL 操作的并行线程”和第 17.12.4 节，“在线 DDL 内存管理”。

+   `innodb_deadlock_detect`

    | 命令行格式 | `--innodb-deadlock-detect[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `innodb_deadlock_detect` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` Hint Applies | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `ON` |

    此选项用于禁用死锁检测。在高并发系统中，死锁检测可能导致大量线程等待同一锁时减速。有时，更有效的做法是禁用死锁检测，并依赖于`innodb_lock_wait_timeout`设置在死锁发生时进行事务回滚。

    有关更多信息，请参阅第 17.7.5.2 节，“死锁检测”。

+   `innodb_dedicated_server`

    | 命令行格式 | `--innodb-dedicated-server[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `innodb_dedicated_server` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    当启用`innodb_dedicated_server`时，`InnoDB`会自动配置以下变量：

    +   `innodb_buffer_pool_size`

    +   `innodb_redo_log_capacity`或在 MySQL 8.0.30 之前，`innodb_log_file_size`和`innodb_log_files_in_group`。

        注意

        `innodb_log_file_size`和`innodb_log_files_in_group`在 MySQL 8.0.30 中已弃用。这些变量已被`innodb_redo_log_capacity`取代。有关更多信息，请参阅第 17.6.5 节，“重做日志”。

    +   `innodb_flush_method`

    只有在 MySQL 实例位于可以使用所有可用系统资源的专用服务器上时，才考虑启用`innodb_dedicated_server`。如果 MySQL 实例与其他应用程序共享系统资源，则不建议启用`innodb_dedicated_server`。

    更多信息，请参阅第 17.8.12 节，“为专用 MySQL 服务器启用自动配置”。

+   `innodb_default_row_format`

    | 命令行格式 | `--innodb-default-row-format=value` |
    | --- | --- |
    | 系统变量 | `innodb_default_row_format` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 枚举 |
    | 默认值 | `DYNAMIC` |
    | 有效值 | `REDUNDANT``COMPACT``DYNAMIC` |

    `innodb_default_row_format`选项定义了`InnoDB`表和用户创建的临时表的默认行格式。默认设置为`DYNAMIC`。其他允许的值为`COMPACT`和`REDUNDANT`。不支持在系统表空间中使用的`COMPRESSED`行格式不能定义为默认值。

    新创建的表在未明确指定`ROW_FORMAT`选项或使用`ROW_FORMAT=DEFAULT`时，会使用由`innodb_default_row_format`定义的行格式。

    当未明确指定`ROW_FORMAT`选项或使用`ROW_FORMAT=DEFAULT`时，任何重建表的操作都会悄无声息地将表的行格式更改为由`innodb_default_row_format`定义的格式。更多信息，请参阅定义表的行格式。

    服务器为处理查询而创建的内部`InnoDB`临时表使用`DYNAMIC`行格式，不受`innodb_default_row_format`设置的影响。

+   `innodb_directories`

    | 命令行格式 | `--innodb-directories=dir_name` |
    | --- | --- |
    | 系统变量 | `innodb_directories` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 目录名称 |
    | 默认值 | `NULL` |

    定义在启动时扫描用于表空间文件的目录。在服务器离线时移动或恢复表空间文件到新位置时使用此选项。还用于指定使用绝对路径创建或位于数据目录之外的表空间文件的目录。

    在崩溃恢复期间，表空间的发现依赖于`innodb_directories`设置来识别重做日志中引用的表空间。更多信息，请参阅崩溃恢复期间的表空间发现。

    默认值为 NULL，但由`innodb_data_home_dir`、`innodb_undo_directory`和`datadir`定义的目录始终会在`InnoDB`在启动时构建要扫描的目录列表时附加到`innodb_directories`参数值。这些目录会被附加，无论是否明确指定了`innodb_directories`设置。

    `innodb_directories`可以作为启动命令中的选项或 MySQL 选项文件中的选项指定。引号包围参数值，否则某些命令解释器会将分号（`;`）解释为特殊字符。（例如，Unix shell 将其视为命令终止符。）

    启动命令：

    ```sql
    mysqld --innodb-directories="*directory_path_1*;*directory_path_2*"
    ```

    MySQL 选项文件：

    ```sql
    [mysqld]
    innodb_directories="*directory_path_1*;*directory_path_2*"
    ```

    不能使用通配符表达式来指定目录。

    `innodb_directories`扫描还会遍历指���目录的子目录。重复的目录和子目录将从要扫描的目录列表中丢弃。

    有关更多信息，请参见第 17.6.3.6 节，“服务器离线时移动表空间文件”。

+   `innodb_disable_sort_file_cache`

    | 命令行格式 | `--innodb-disable-sort-file-cache[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `innodb_disable_sort_file_cache` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    禁用合并排序临时文件的操作系统文件系统缓存。效果是以`O_DIRECT`的等效方式打开这些文件。

+   `innodb_doublewrite`

    | 命令行格式 | `--innodb-doublewrite=value`（≥ 8.0.30）`--innodb-doublewrite[={OFF&#124;ON}]`（≤ 8.0.29） |
    | --- | --- |
    | 系统变量 | `innodb_doublewrite` |
    | 作用范围 | 全局 |
    | 动态（≥ 8.0.30） | 是 |
    | 动态（≤ 8.0.29） | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型（≥ 8.0.30） | 枚举 |
    | 类型（≤ 8.0.29） | 布尔值 |
    | 默认值 | `ON` |
    | 有效值 | `ON``OFF``DETECT_AND_RECOVER``DETECT_ONLY` |

    `innodb_doublewrite`变量控制双写缓冲。在大多数情况下，默认情况下启用双写缓冲。

    在 MySQL 8.0.30 之前，您可以在启动服务器时将`innodb_doublewrite`设置为`ON`或`OFF`以分别启用或禁用双写缓冲，从 MySQL 8.0.30 开始，`innodb_doublewrite`还支持`DETECT_AND_RECOVER`和`DETECT_ONLY`设置。

    `DETECT_AND_RECOVER`设置与`ON`设置相同。使用此设置，双写缓冲区完全启用，数据库页面内容被写入双写缓冲区，在恢复过程中访问以修复不完整的页面写入。

    使用`DETECT_ONLY`设置时，只有元数据被写入双写缓冲区。数据库页面内容不会被写入双写缓冲区，并且恢复不使用双写缓冲区来修复不完整的页面写入。此轻量级设置仅用于检测不完整的页面写入。

    MySQL 8.0.30 及更高版本支持动态更改`innodb_doublewrite`设置，可以在`ON`、`DETECT_AND_RECOVER`和`DETECT_ONLY`之间启用双写缓冲区。MySQL 不支持在启用双写缓冲区和`OFF`之间进行动态更改。

    如果双写缓冲区位于支持原子写入的 Fusion-io 设备上，则双写缓冲区将自动禁用，并且数据文件写入将使用 Fusion-io 原子写入。但是，请注意`innodb_doublewrite`设置是全局的。当双写缓冲区被禁用时，所有数据文件都被禁用，包括那些不位于 Fusion-io 硬件上的文件。此功能仅在 Fusion-io 硬件上受支持，并且仅在 Linux 上为 Fusion-io NVMFS 启用。为了充分利用此功能，建议将`innodb_flush_method`设置为`O_DIRECT`。

    有关相关信息，请参见第 17.6.4 节，“双写缓冲区”。

+   `innodb_doublewrite_batch_size`

    | 命令行格式 | `--innodb-doublewrite-batch-size=#` |
    | --- | --- |
    | 引入版本 | 8.0.20 |
    | 系统变量 | `innodb_doublewrite_batch_size` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `0` |
    | 最小值 | `0` |
    | 最大值 | `256` |

    定义要批量写入的双写页面数。

    有关更多信息，请参见第 17.6.4 节，“双写缓冲区”。

+   `innodb_doublewrite_dir`

    | 命令行格式 | `--innodb-doublewrite-dir=dir_name` |
    | --- | --- |
    | 引入版本 | 8.0.20 |
    | 系统变量 | `innodb_doublewrite_dir` |
    | 作用范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 目录名称 |

    定义双写文件的目录。如果未指定目录，则双写文件将在`innodb_data_home_dir`目录中创建，默认情况下为数据目录（如果未指定）。

    更多信息，请参见第 17.6.4 节，“双写缓冲区”。

+   `innodb_doublewrite_files`

    | 命令行格式 | `--innodb-doublewrite-files=#` |
    | --- | --- |
    | 引入版本 | 8.0.20 |
    | 系统变量 | `innodb_doublewrite_files` |
    | 作用范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `innodb_buffer_pool_instances * 2` |
    | 最小值 | `2` |
    | 最大值 | `256` |

    定义双写文件的数量。默认情况下，为每个缓冲池实例创建两个双写文件。

    至少有两个双写文件。双写文件的最大数量是缓冲池实例数量的两倍。（缓冲池实例数量由`innodb_buffer_pool_instances`变量控制。）

    更多信息，请参见第 17.6.4 节，“双写缓冲区”。

+   `innodb_doublewrite_pages`

    | 命令行格式 | `--innodb-doublewrite-pages=#` |
    | --- | --- |
    | 引入版本 | 8.0.20 |
    | 系统变量 | `innodb_doublewrite_pages` |
    | 作用范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `innodb_write_io_threads value` |
    | 最小值 | `innodb_write_io_threads value` |
    | 最大值 | `512` |

    定义每个线程的批量写入的双写页面的最大数量。如果未指定值，则`innodb_doublewrite_pages`设置为`innodb_write_io_threads`的值。

    更多信息，请参见第 17.6.4 节，“双写缓冲区”。

+   `innodb_extend_and_initialize`

    | 命令行格式 | `--innodb=extend-and-initialize[={OFF&#124;ON}]` |
    | --- | --- |
    | 引入版本 | 8.0.22 |
    | 系统变量 | `innodb_extend_and_initialize` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `ON` |

    控制在 Linux 系统上为每个表和通用表空间分配空间的方式。

    启用时，`InnoDB`会将 NULL 写入新分配的页面。禁用时，空间是通过`posix_fallocate()`调用进行分配的，该调用保留空间而不实际写入 NULL。

    欲了解更多信息，请参阅第 17.6.3.8 节，“优化 Linux 上的表空间空间分配”。

+   `innodb_fast_shutdown`

    | 命令行格式 | `--innodb-fast-shutdown=#` |
    | --- | --- |
    | 系统变量 | `innodb_fast_shutdown` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `1` |
    | 有效值 | `0``1``2` |

    `InnoDB` 关闭模式。如果值为 0，则`InnoDB`在关闭之前执行慢关闭、完全清理和更改缓冲区合并。如果值为 1（默认值），`InnoDB`在关闭时跳过这些操作，这个过程称为快速关闭。如果值为 2，则`InnoDB`刷新其日志并冷静���闭，就像 MySQL 崩溃了一样；没有提交的事务会丢失，但崩溃恢复操作会使下一次启动时间变长。

    慢关闭可能需要几分钟，甚至在极端情况下，仍有大量数据缓冲时可能需要几个小时。在升级或降级 MySQL 主要版本之前，请使用慢关闭技术，以便在升级过程中更新文件格式时，所有数据文件都已准备就绪。

    在紧急情况或故障排除情况下使用`innodb_fast_shutdown=2`，以获得绝对最快的关闭速度，如果数据有损坏风险。

+   `innodb_fil_make_page_dirty_debug`

    | 命令行格式 | `--innodb-fil-make-page-dirty-debug=#` |
    | --- | --- |
    | 系统变量 | `innodb_fil_make_page_dirty_debug` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `0` |
    | 最小值 | `0` |
    | 最大值 | `2**32-1` |

    默认情况下，将 `innodb_fil_make_page_dirty_debug` 设置为表空间的 ID 会立即使表空间的第一页变脏。如果 `innodb_saved_page_number_debug` 设置为非默认值，则设置 `innodb_fil_make_page_dirty_debug` 会使指定页变脏。只有在使用 `WITH_DEBUG` **CMake** 选项编译时，才能使用 `innodb_fil_make_page_dirty_debug` 选项。

+   `innodb_file_per_table`

    | 命令行格式 | `--innodb-file-per-table[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `innodb_file_per_table` |
    | 作用域 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `ON` |

    当启用 `innodb_file_per_table` 时，默认情况下表会在文件表空间中创建。当禁用时，默认情况下表会在系统表空间中创建。有关文件表空间的信息，请参阅 第 17.6.3.2 节，“文件表空间”。有关 `InnoDB` 系统表空间的信息，请参阅 第 17.6.3.1 节，“系统表空间”。

    `innodb_file_per_table` 变量可以通过 `SET GLOBAL` 语句在运行时配置，在启动时在命令行上指定，或在选项文件中指定。在运行时配置需要足够的权限来设置全局系统变量（参见 第 7.1.9.1 节，“系统变量权限”），并立即影响所有连接的操作。

    当位于文件表空间中的表被截断或删除时，释放的空间会返回给操作系统。截断或删除位于系统表空间中的表只会释放系统表空间中的空间。系统表空间中释放的空间可以再次用于 `InnoDB` 数据，但不会返回给操作系统，因为系统表空间数据文件永远不会收缩。

    `innodb_file_per-table` 设置不影响临时表的创建。从 MySQL 8.0.14 开始，临时表在会话临时表空间中创建，在此之前是在全局临时表空间中创建。请参阅 第 17.6.3.5 节，“临时表空间”。

+   `innodb_fill_factor`

    | 命令行格式 | `--innodb-fill-factor=#` |
    | --- | --- |
    | 系统变量 | `innodb_fill_factor` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `100` |
    | 最小值 | `10` |
    | 最大值 | `100` |

    `InnoDB` 在创建或重建索引时执行批量加载。这种索引创建方法被称为“排序索引构建”。

    `innodb_fill_factor` 定义了在排序索引构建期间填充在每个 B 树页上的空间百分比，剩余空间保留用于未来的索引增长���例如，将 `innodb_fill_factor` 设置为 80，将在每个 B 树页上保留 20% 的空间用于未来的索引增长。实际百分比可能有所不同。`innodb_fill_factor` 设置被解释为提示而不是硬限制。

    将 `innodb_fill_factor` 设置为 100，将聚簇索引页中的 1/16 空间留给未来的索引增长。

    `innodb_fill_factor` 适用于 B 树叶子页和非叶子页。它不适用于用于 `TEXT` 或 `BLOB` 条目的外部页。

    欲了解更多信息，请参阅 第 17.6.2.3 节，“排序索引构建”。

+   `innodb_flush_log_at_timeout`

    | 命令行格式 | `--innodb-flush-log-at-timeout=#` |
    | --- | --- |
    | 系统变量 | `innodb_flush_log_at_timeout` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `1` |
    | 最小值 | `1` |
    | 最大值 | `2700` |
    | 单位 | 秒 |

    每隔*`N`*秒写入并刷新日志。`innodb_flush_log_at_timeout`允许增加刷新之间的超时时间，以减少刷新并避免影响二进制日志组提交的性能。`innodb_flush_log_at_timeout`的默认设置是每秒一次。

+   `innodb_flush_log_at_trx_commit`

    | 命令行格式 | `--innodb-flush-log-at-trx-commit=#` |
    | --- | --- |
    | 系统变量 | `innodb_flush_log_at_trx_commit` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 枚举 |
    | 默认值 | `1` |
    | 有效值 | `0``1``2` |

    控制严格的 ACID 合规性和在重新排列和批量执行与提交相关的 I/O 操作时可能实现的更高性能之间的平衡。通过更改默认值，您可以获得更好的性能，但在崩溃时可能会丢失事务。

    +   默认设置为 1 是为了完全符合 ACID 要求。日志在每次事务提交时被写入并刷新到磁盘。

    +   设置为 0 时，日志每秒写入并刷新到磁盘一次。在崩溃时可能会丢失未刷新日志的事务。

    +   设置为 2 时，日志在每次事务提交后写入并每秒刷新到磁盘一次。在崩溃时可能会丢失未刷新日志的事务。

    +   对于设置为 0 和 2 的情况，每秒刷新并不是 100%保证的。由于 DDL 更改和其他内部`InnoDB`活动可能导致日志独立于`innodb_flush_log_at_trx_commit`设置而更频繁地刷新，有时由于调度问题而更少地刷新。如果日志每秒刷新一次，在崩溃时可能会丢失最多一秒钟的事务。如果日志的刷新频率高于或低于每秒一次，可能会相应地丢失不同数量的事务。

    +   日志刷新频率由`innodb_flush_log_at_timeout`控制，允许您将日志刷新频率设置为*`N`*秒（其中*`N`*为`1 ... 2700`，默认值为 1）。然而，任何意外的**mysqld**进程退出可能会擦除最多*`N`*秒的事务。

    +   DDL 更改和其他内部`InnoDB`活动会独立于`innodb_flush_log_at_trx_commit`设置刷新日志。

    +   `InnoDB` 崩溃恢复无论`innodb_flush_log_at_trx_commit`设置如何都能正常工作。事务要么完全应用，要么完全擦除。

    对于使用带有事务的`InnoDB`的复制设置中的耐用性和一致性：

    +   如果启用了二进制日志记录，请设置`sync_binlog=1`。

    +   始终设置`innodb_flush_log_at_trx_commit=1`。

    对于在复制品上组合设置的信息，以使其对意外停机最具弹性，请参阅第 19.4.2 节，“处理复制品意外停机”。

    警告

    许多操作系统和一些磁盘硬件欺骗了刷新到磁盘的操作。它们可能会告诉**mysqld**刷新已经完成，尽管实际上并没有。在这种情况下，即使使用推荐的设置，事务的持久性也无法得到保证，最坏的情况下，断电可能会损坏`InnoDB`数据。在 SCSI 磁盘控制器或磁盘本身中使用带电池后备的磁盘缓存可以加快文件刷新速度，并使操作更安全。您还可以尝试禁用硬件缓存中的磁盘写入缓存。

+   `innodb_flush_method`

    | 命令行格式 | `--innodb-flush-method=value` |
    | --- | --- |
    | 系统变量 | `innodb_flush_method` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 字符串 |
    | 默认值（Unix） | `fsync` |
    | 默认值（Windows） | `unbuffered` |
    | 有效值（Unix） | `fsync``O_DSYNC``littlesync``nosync``O_DIRECT``O_DIRECT_NO_FSYNC` |
    | 有效值（Windows） | `unbuffered``normal` |

    定义用于将数据刷新到`InnoDB`数据文件和日志文件的方法，这可能会影响 I/O 吞吐量。

    在类 Unix 系统上，默认值为`fsync`。在 Windows 上，默认值为`unbuffered`。

    注意

    在 MySQL 8.0 中，可以通过数字方式指定`innodb_flush_method`选项。

    适用于类 Unix 系统的`innodb_flush_method`选项包括：

    +   `fsync`或`0`：`InnoDB`使用`fsync()`系统调用来刷新数据和日志文件。`fsync`是默认设置。

    +   `O_DSYNC`或`1`：`InnoDB`使用`O_SYNC`来打开和刷新日志文件，并使用`fsync()`来刷新数据文件。`InnoDB`不直接使用`O_DSYNC`，因为在许多 Unix 变种上存在问题。

    +   `littlesync`或`2`：此选项用于内部性能测试，目前不受支持。请自行承担风险。

    +   `nosync`或`3`：此选项用于内部性能测试，目前不受支持。请自行承担风险。

    +   `O_DIRECT`或`4`：`InnoDB`使用`O_DIRECT`（或 Solaris 上的`directio()`）来打开数据文件，并使用`fsync()`来刷新数据和日志文件。此选项适用于某些 GNU/Linux 版本、FreeBSD 和 Solaris。

    +   `O_DIRECT_NO_FSYNC`：`InnoDB`在刷新 I/O 时使用`O_DIRECT`，但在每次写操作后跳过`fsync()`系统调用。

        在 MySQL 8.0.14 之前，此设置不适用于需要`fsync()`系统调用同步文件系统元数据更改的文件系统，如 XFS 和 EXT4。如果不确定您的文件系统是否需要`fsync()`系统调用来同步文件系统元数据更改，请改用`O_DIRECT`。

        自 MySQL 8.0.14 起，在创建新文件、增加文件大小和关闭文件后，会调用`fsync()`以确保文件系统元数据更改被同步。每次写操作后仍会跳过`fsync()`系统调用。

        如果重做日志文件和数据文件位于不同存储设备上，并且在数据文件写入未从非带电池备份的设备缓存中刷新时发生意外退出，则可能会发生数据丢失。如果您使用或打算使用不同的存储设备用于重做日志文件和数据文件，并且您的数据文件位于没有带电池备份的缓存的设备上，请改用`O_DIRECT`。

    在支持`fdatasync()`系统调用的平台上，MySQL 8.0.26 中引入的`innodb_use_fdatasync`变量允许使用`fsync()`的`innodb_flush_method`选项来代替`fdatasync()`。`fdatasync()`系统调用不会刷新文件元数据，除非需要用于后续数据检索，从而提供潜在的性能优势。

    Windows 系统的`innodb_flush_method`选项包括：

    +   `unbuffered`或`0`：`InnoDB`使用模拟异步 I/O 和非缓冲 I/O。

    +   `normal`或`1`：`InnoDB`使用模拟异步 I/O 和缓冲 I/O。

    每个设置如何影响性能取决于硬件配置和工作负载。基准测试您的特定配置以决定使用哪个设置，或者是否保留默认设置。检查`Innodb_data_fsyncs`状态变量，查看每个设置的`fsync()`调用总数（如果启用了`innodb_use_fdatasync`，则为`fdatasync()`调用）。工作负载中读取和写入操作的混合可能会影响设置的性能。例如，在具有硬件 RAID 控制器和带电池后备写缓存的系统上，`O_DIRECT`可以帮助避免`InnoDB`缓冲池和操作系统文件系统缓存之间的双重缓冲。在一些`InnoDB`数据和日志文件位于 SAN 上的系统中，默认值或`O_DSYNC`可能对大部分为`SELECT`语句的读取密集型工作负载更快。始终使用反映生产环境的硬件和工作负载测试此参数。有关一般 I/O 调优建议，请参见第 10.5.8 节，“优化 InnoDB 磁盘 I/O”。

    如果启用了`innodb_dedicated_server`，则如果未明确定义，`innodb_flush_method`值将自动配置。有关更多信息，请参见第 17.8.12 节，“为专用 MySQL 服务器启用自动配置”。

+   `innodb_flush_neighbors`

    | 命令行格式 | `--innodb-flush-neighbors=#` |
    | --- | --- |
    | 系统变量 | `innodb_flush_neighbors` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 枚举 |
    | 默认值 | `0` |
    | 有效值 | `0``1``2` |

    指定从`InnoDB`缓冲池刷新页面时是否也刷新同一范围中的其他脏页。

    +   设置为 0 会禁用`innodb_flush_neighbors`。同一范围内的脏页不会被刷新。

    +   设置为 1 会刷新同一范围内连续的脏页。

    +   设置为 2 会刷新同一范围内的脏页。

    当表数据存储在传统的 HDD 存储设备上时，一次刷新这样的相邻页减少了 I/O 开销（主要是磁盘寻道操作）与在不同时间刷新单个页相比。对于存储在 SSD 上的表数据，寻道时间不是一个重要因素，您可以将此选项设置为 0 以分散写操作。有关相关信息，请参见第 17.8.3.5 节，“配置缓冲池刷新”。

+   `innodb_flush_sync`

    | 命令行格式 | `--innodb-flush-sync[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `innodb_flush_sync` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔 |
    | 默认值 | `ON` |

    默认情况下启用的`innodb_flush_sync`变量导致在检查点发生 I/O 活动突发时忽略`innodb_io_capacity`设置。要遵守由`innodb_io_capacity`设置定义的 I/O 速率，在 I/O 活动突发时禁用`innodb_flush_sync`。

    有关配置`innodb_flush_sync`变量的信息，请参见第 17.8.7 节，“配置 InnoDB I/O 容量”。

+   `innodb_flushing_avg_loops`

    | 命令行格式 | `--innodb-flushing-avg-loops=#` |
    | --- | --- |
    | 系统变量 | `innodb_flushing_avg_loops` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `30` |
    | 最小值 | `1` |
    | 最大值 | `1000` |

    `InnoDB`保留先前计算的刷新状态快照的迭代次数，控制自适应刷新对变化的工作负载作出响应的速度。增加该值使得刷新操作的速率在工作负载变化时平稳逐渐变化。减少该值使得自适应刷新快速调整到工作负载变化，这可能导致刷新活动在工作负载突然增加和减少时出现波动。

    有关相关信息，请参阅第 17.8.3.5 节，“配置缓冲池刷新”。

+   `innodb_force_load_corrupted`

    | 命令行格式 | `--innodb-force-load-corrupted[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `innodb_force_load_corrupted` |
    | 作用范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    允许`InnoDB`在启动时加载标记为损坏的表格。仅在故障排除期间使用，以恢复其他无法访问的数据。故障排除完成后，请禁用此设置并重新启动服务器。

+   `innodb_force_recovery`

    | 命令行格式 | `--innodb-force-recovery=#` |
    | --- | --- |
    | 系统变量 | `innodb_force_recovery` |
    | 作用范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `0` |
    | 最小值 | `0` |
    | 最大值 | `6` |

    崩溃恢复模式，通常仅在严重故障排除情况下更改。可能的值为 0 到 6。有关这些值的含义以及关于`innodb_force_recovery`的重要信息，请参阅第 17.21.3 节，“强制 InnoDB 恢复”。

    警告

    仅在紧急情况下将此变量设置为大于 0 的值，以便您可以启动`InnoDB`并转储表格。作为安全措施，当`innodb_force_recovery`大于 0 时，`InnoDB`会阻止`INSERT`、`UPDATE`或`DELETE`操作。`innodb_force_recovery`设置为 4 或更高时，将`InnoDB`置于只读模式。

    这些限制可能导致复制管理命令失败并显示错误，因为复制将副本状态日志存储在`InnoDB`表中。

+   `innodb_fsync_threshold`

    | 命令行格式 | `--innodb-fsync-threshold=#` |
    | --- | --- |
    | 引入版本 | 8.0.13 |
    | 系统变量 | `innodb_fsync_threshold` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `0` |
    | 最小值 | `0` |
    | 最大值 | `2**64-1` |

    默认情况下，当`InnoDB`创建新的数据文件，例如新的日志文件或表空间文件时，文件在刷新到磁盘之前会完全写入操作系统缓存，这可能导致大量的磁盘写入活动一次性发生。为了强制从操作系统缓存中定期刷新较小的数据块，您可以使用`innodb_fsync_threshold`变量定义一个阈值值，以字节为单位。当达到字节阈值时，操作系统缓存的内容会刷新到磁盘。默认值为 0，强制默认行为，即仅在文件完全写入缓存后才将数据刷新到磁盘。

    指定阈值以强制较小的定期刷新可能有益于多个 MySQL 实例使用相同存储设备的情况。例如，创建新的 MySQL 实例及其关联的数据文件可能导致大量的磁盘写入活动激增，影响使用相同存储设备的其他 MySQL 实例的性能。配置阈值有助于避免此类写入活动激增。

+   `innodb_ft_aux_table`

    | 系统变量 | `innodb_ft_aux_table` |
    | --- | --- |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 字符串 |

    指定包含`FULLTEXT`索引的`InnoDB`表的限定名称。此变量仅用于诊断目的，并且只能在运行时设置。例如：

    ```sql
    SET GLOBAL innodb_ft_aux_table = 'test/t1';
    ```

    将此变量设置为格式为`*`db_name`*/*`table_name`*`的名称后，`INFORMATION_SCHEMA`表`INNODB_FT_INDEX_TABLE`、`INNODB_FT_INDEX_CACHE`、`INNODB_FT_CONFIG`、`INNODB_FT_DELETED`和`INNODB_FT_BEING_DELETED`显示关于指定表的搜索索引的信息。

    更多信息，请参见 第 17.15.4 节，“InnoDB INFORMATION_SCHEMA FULLTEXT Index Tables”。

+   `innodb_ft_cache_size`

    | 命令行格式 | `--innodb-ft-cache-size=#` |
    | --- | --- |
    | 系统变量 | `innodb_ft_cache_size` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `8000000` |
    | 最小值 | `1600000` |
    | 最大值 | `80000000` |
    | 单位 | 字节 |

    为 `InnoDB` `FULLTEXT` 搜索索引缓存分配的内存量（以字节为单位），在创建 `InnoDB` `FULLTEXT` 索引时，该缓存在内存中保存解析的文档。仅当达到 `innodb_ft_cache_size` 大小限制时，索引插入和更新才会提交到磁盘。`innodb_ft_cache_size` 定义了每个表的缓存大小。要为所有表设置全局限制，请参阅 `innodb_ft_total_cache_size`。

    欲了解更多信息，请参阅 InnoDB 全文索引缓存。

+   `innodb_ft_enable_diag_print`

    | 命令行格式 | `--innodb-ft-enable-diag-print[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `innodb_ft_enable_diag_print` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    是否启用额外的全文搜索（FTS）诊断输出。此选项主要用于高级 FTS 调试，对大多数用户不感兴趣。输出打印到错误日志中，包括以下信息：

    +   FTS 索引同步进度（当达到 FTS 缓存限制时）。例如：

        ```sql
        FTS SYNC for table test, deleted count: 100 size: 10000 bytes
        SYNC words: 100
        ```

    +   FTS 优化进度。例如：

        ```sql
        FTS start optimize test
        FTS_OPTIMIZE: optimize "mysql"
        FTS_OPTIMIZE: processed "mysql"
        ```

    +   FTS 索引构建进度。例如：

        ```sql
        Number of doc processed: 1000
        ```

    +   对于 FTS 查询，会打印查询解析树、词权重、查询处理时间和内存使用情况。例如：

        ```sql
        FTS Search Processing time: 1 secs: 100 millisec: row(s) 10000
        Full Search Memory: 245666 (bytes),  Row: 10000
        ```

+   `innodb_ft_enable_stopword`

    | 命令行格式 | `--innodb-ft-enable-stopword[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `innodb_ft_enable_stopword` |
    | 作用范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `ON` |

    指定在创建 `InnoDB` `FULLTEXT` 索引时，与之关联的一组停用词。如果设置了 `innodb_ft_user_stopword_table` 选项，则从该表中获取停用词。否则，如果设置了 `innodb_ft_server_stopword_table` 选项，则从该表中获取停用词。否则，将使用内置的默认停用词集。

    欲了解更多信息，请参阅 第 14.9.4 节，“全文搜索停用词”。

+   `innodb_ft_max_token_size`

    | 命令行格式 | `--innodb-ft-max-token-size=#` |
    | --- | --- |
    | 系统变量 | `innodb_ft_max_token_size` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `84` |
    | 最小值 | `10` |
    | 最大值 | `84` |

    存储在 `InnoDB` `FULLTEXT` 索引中的单词的最大字符长度。设置此值的限制会减小索引的大小，从而加快查询速度，通过省略长关键词或不是真实单词且不太可能是搜索词的任意字母集合。

    欲了解更多信息，请参阅 第 14.9.6 节，“调整 MySQL 全文搜索”。

+   `innodb_ft_min_token_size`

    | 命令行格式 | `--innodb-ft-min-token-size=#` |
    | --- | --- |
    | 系统变量 | `innodb_ft_min_token_size` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `3` |
    | 最小值 | `0` |
    | 最大值 | `16` |

    存储在 `InnoDB` `FULLTEXT` 索引中的单词的最小长度。增加此值会减小索引的大小，从而加快查询速度，通过省略在搜索上下文中不太可能重要的常见单词，例如英语单词“a”和“to”。对于使用 CJK（中文、日文、韩文）字符集的内容，请指定值为 1。

    欲了解更多信息，请参阅 第 14.9.6 节，“调整 MySQL 全文搜索”。

+   `innodb_ft_num_word_optimize`

    | 命令行格式 | `--innodb-ft-num-word-optimize=#` |
    | --- | --- |
    | 系统变量 | `innodb_ft_num_word_optimize` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `2000` |
    | 最小值 | `1000` |
    | 最大值 | `10000` |

    在每次对 `InnoDB` `FULLTEXT` 索引执行 `OPTIMIZE TABLE` 操作期间要处理的单词数。因为对包含全文搜索索引的表进行大量插入或更新操作可能需要大量的索引维护来合并所有更改，您可能需要执行一系列 `OPTIMIZE TABLE` 语句，每个语句从上一个语句结束的地方开始。

    更多信息，请参见 第 14.9.6 节，“调整 MySQL 全文搜索”。

+   `innodb_ft_result_cache_limit`

    | 命令行格式 | `--innodb-ft-result-cache-limit=#` |
    | --- | --- |
    | 系统变量 | `innodb_ft_result_cache_limit` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提��适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `2000000000` |
    | 最小值 | `1000000` |
    | 最大值 | `2**32-1` |
    | 单位 | 字节 |

    每个全文搜索查询或每个线程的 `InnoDB` 全文搜索查询结果缓存限制（以字节为单位）。中间和最终的 `InnoDB` 全文搜索查询结果在内存中处理。使用 `innodb_ft_result_cache_limit` 来对全文搜索查询结果缓存设置大小限制，以避免在出现非常大的 `InnoDB` 全文搜索查询结果（例如数百万或数亿行）时导致过多的内存消耗。在处理全文搜索查询时根据需要分配内存。如果达到结果缓存大小限制，将返回错误，指示查询超过允许的最大内存。

    所有平台类型和位数的 `innodb_ft_result_cache_limit` 的最大值为 2**32-1。

+   `innodb_ft_server_stopword_table`

    | 命令行格式 | `--innodb-ft-server-stopword-table=db_name/table_name` |
    | --- | --- |
    | 系统变量 | `innodb_ft_server_stopword_table` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 字符串 |
    | 默认值 | `NULL` |

    此选项用于为所有 `InnoDB` 表指定自己的 `InnoDB` `FULLTEXT` 索引停用词列表。要为特定的 `InnoDB` 表配置自己的停用词列表，请使用 `innodb_ft_user_stopword_table`。

    将`innodb_ft_server_stopword_table`设置为包含停用词列表的表的名称，格式为`*`db_name`*/*`table_name`*`。

    在配置`innodb_ft_server_stopword_table`之前，停用词表必须存在。在创建`FULLTEXT`索引之前，必须启用`innodb_ft_enable_stopword`，并配置`innodb_ft_server_stopword_table`选项。

    停用词表必须是一个`InnoDB`表，包含一个名为`value`的单个`VARCHAR`列。

    更多信息，请参见第 14.9.4 节，“全文停用词”。

+   `innodb_ft_sort_pll_degree`

    | 命令行格式 | `--innodb-ft-sort-pll-degree=#` |
    | --- | --- |
    | 系统变量 | `innodb_ft_sort_pll_degree` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `2` |
    | 最小值 | `1` |
    | 最大值 | `16` |

    在构建搜索索引时，用于并行索引和标记`InnoDB` `FULLTEXT`索引中文本的线程数。

    有关信息，请参见第 17.6.2.4 节，“InnoDB 全文索引”，以及`innodb_sort_buffer_size`。

+   `innodb_ft_total_cache_size`

    | 命令行格式 | `--innodb-ft-total-cache-size=#` |
    | --- | --- |
    | 系统变量 | `innodb_ft_total_cache_size` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `640000000` |
    | 最小值 | `32000000` |
    | 最大值 | `1600000000` |
    | 单位 | 字节 |

    为所有表的`InnoDB`全文搜索索引缓存分配的总内存，以字节为单位。创建多个具有`FULLTEXT`搜索索引的表可能会消耗大量可用内存。`innodb_ft_total_cache_size`定义了所有全文搜索索引的全局内存限制，以帮助避免过度内存消耗。如果索引操作达到全局限制，将触发强制同步。

    更多信息，请参见 InnoDB 全文索引缓存。

+   `innodb_ft_user_stopword_table`

    | 命令行格式 | `--innodb-ft-user-stopword-table=db_name/table_name` |
    | --- | --- |
    | 系统变量 | `innodb_ft_user_stopword_table` |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 字符串 |
    | 默认值 | `NULL` |

    此选项用于在特定表上指定自己的 `InnoDB` `FULLTEXT` 索引停用词列表。要为所有 `InnoDB` 表配置自己的停用词列表，请使用 `innodb_ft_server_stopword_table`。

    将 `innodb_ft_user_stopword_table` 设置为包含停用词列表的表的名称，格式为 `*`db_name`*/*`table_name`*`。

    在配置 `innodb_ft_user_stopword_table` 之前，停用词表必须存在。在创建 `FULLTEXT` 索引之前，必须启用 `innodb_ft_enable_stopword` 并配置 `innodb_ft_user_stopword_table`。

    停用词表必须是一个 `InnoDB` 表，包含一个名为 `value` 的单个 `VARCHAR` 列。

    更多信息，请参阅 第 14.9.4 节，“全文本停用词”。

+   `innodb_idle_flush_pct`

    | 命令行格式 | `--innodb-idle-flush-pct=#` |
    | --- | --- |
    | 引入版本 | 8.0.18 |
    | 系统变量 | `innodb_idle_flush_pct` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `100` |
    | 最小值 | `0` |
    | 最大值 | `100` |

    当 `InnoDB` 空闲时限制页面刷新。`innodb_idle_flush_pct` 值是 `innodb_io_capacity` 设置的百分比，该设置定义了 `InnoDB` 每秒可用的 I/O 操作数。更多信息，请参阅 在空闲时期限制缓冲区刷新。

+   `innodb_io_capacity`

    | 命令行格式 | `--innodb-io-capacity=#` |
    | --- | --- |
    | 系统变量 | `innodb_io_capacity` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `200` |
    | 最小值 | `100` |
    | 最大值（64 位平台） | `2**64-1` |
    | 最大值（32 位平台） | `2**32-1` |

    `innodb_io_capacity`变量定义了`InnoDB`后台任务每秒（IOPS）可用的 I/O 操作数量，例如从缓冲池刷新页面和从更改缓冲区合并数据。

    有关配置`innodb_io_capacity`变量的信息，请参见第 17.8.7 节，“配置 InnoDB I/O 容量”。

+   `innodb_io_capacity_max`

    | 命令行格式 | `--innodb-io-capacity-max=#` |
    | --- | --- |
    | 系统变量 | `innodb_io_capacity_max` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `见描述` |
    | 最小值 | `100` |
    | 最大值（32 位平台） | `2**32-1` |
    | 最大值（Unix，64 位平台，≥ 8.0.29） | `2**32-1` |
    | 最大值（Unix，64 位平台，≤ 8.0.28） | `2**64-1` |
    | 最大值（Windows，64 位平台） | `2**32-1` |

    如果刷新活动落后，`InnoDB`可以以比`innodb_io_capacity`变量定义的更高的 I/O 操作每秒（IOPS）速率更积极地刷新。`innodb_io_capacity_max`变量定义了在这种情况下`InnoDB`后台任务执行的最大 IOPS 数量。

    有关配置`innodb_io_capacity_max`变量的信息，请参见第 17.8.7 节，“配置 InnoDB I/O 容量”。

+   `innodb_limit_optimistic_insert_debug`

    | 命令行格式 | `--innodb-limit-optimistic-insert-debug=#` |
    | --- | --- |
    | 系统变量 | `innodb_limit_optimistic_insert_debug` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `0` |
    | 最小值 | `0` |
    | 最大值 | `2**32-1` |

    限制每个 B 树页面的记录数。默认值为 0 表示不施加限制。此选项仅在使用`WITH_DEBUG` **CMake** 选项编译调试支持时才可用。

+   `innodb_lock_wait_timeout`

    | 命令行格式 | `--innodb-lock-wait-timeout=#` |
    | --- | --- |
    | 系统变量 | `innodb_lock_wait_timeout` |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `50` |
    | 最小值 | `1` |
    | 最大值 | `1073741824` |
    | 单位 | 秒 |

    在`InnoDB` 事务在放弃之前等待行锁的时间长度（以秒为单位）。默认值为 50 秒。尝试访问被另一个`InnoDB`事务锁定的行的事务在发出以下错误之前最多等待这么多秒以获得对行的写访问：

    ```sql
    ERROR 1205 (HY000): Lock wait timeout exceeded; try restarting transaction
    ```

    当发生锁等待超时时，当前语句会被回滚（而不是整个事务）。要使整个事务回滚，请使用`--innodb-rollback-on-timeout`选项启动服务器。另请参见第 17.21.5 节，“InnoDB 错误处理”。

    对于高度交互式应用程序或 OLTP 系统，您可以减少此值，以便快速显示用户反馈或将更新放入队列以供稍后处理。对于长时间运行的后端操作，例如数据仓库中等待其他大型插入或更新操作完成的转换步骤，您可以增加此值。

    `innodb_lock_wait_timeout` 适用于 `InnoDB` 行锁。MySQL 的表锁不会发生在 `InnoDB` 中，因此此超时不适用于等待表锁。

    当启用（默认）`innodb_deadlock_detect` 时，锁等待超时值不适用于死锁，因为 `InnoDB` 立即检测到死锁并回滚其中一个死锁事务。当禁用`innodb_deadlock_detect` 时，`InnoDB` 依赖于`innodb_lock_wait_timeout` 在发生死锁时进行事务回滚。参见第 17.7.5.2 节，“死锁检测”。

    `innodb_lock_wait_timeout`可以使用`SET GLOBAL`或`SET SESSION`语句在运行时设置。更改`GLOBAL`设置需要足够权限来设置全局系统变量（请参阅第 7.1.9.1 节，“系统变量权限”），并影响随后连接的所有客户端的操作。任何客户端都可以更改`innodb_lock_wait_timeout`的`SESSION`设置，这仅影响该客户端。

+   `innodb_log_buffer_size`

    | 命令行格式 | `--innodb-log-buffer-size=#` |
    | --- | --- |
    | 系统变量 | `innodb_log_buffer_size` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `16777216` |
    | 最小值 | `1048576` |
    | 最大值 | `4294967295` |

    `InnoDB`用于在磁盘上写入日志文件的缓冲区大小（以字节为单位）。默认值为 16MB。较大的日志缓冲区使得大型事务可以在提交之前无需将日志写入磁盘。因此，如果您有更新、插入或删除多行的事务，增大日志缓冲区可以节省磁盘 I/O。有关相关信息，请参阅内存配置和第 10.5.4 节，“优化 InnoDB 重做日志记录”。有关一般 I/O 调优建议，请参阅第 10.5.8 节，“优化 InnoDB 磁盘 I/O”。

+   `innodb_log_checkpoint_fuzzy_now`

    | 命令行格式 | `--innodb-log-checkpoint-fuzzy-now[={OFF&#124;ON}]` |
    | --- | --- |
    | 引入版本 | 8.0.13 |
    | 系统变量 | `innodb_log_checkpoint_fuzzy_now` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    启用此调试选项以强制`InnoDB`执行模糊检查点。此选项仅在使用`WITH_DEBUG` **CMake** 选项编译时才可用。

+   `innodb_log_checkpoint_now`

    | 命令行格式 | `--innodb-log-checkpoint-now[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `innodb_log_checkpoint_now` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    启用此调试选项以强制`InnoDB`写入检查点。此选项仅在使用`WITH_DEBUG` **CMake**选项编译时才可用。

+   `innodb_log_checksums`

    | 命令行格式 | `--innodb-log-checksums[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `innodb_log_checksums` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `ON` |

    启用或禁用重做日志页面的校验和。

    `innodb_log_checksums=ON`启用`CRC-32C`校验算法用于重做日志页面。当禁用`innodb_log_checksums`时，重做日志页面校验字段的内容将被忽略。

    重做日志头页面和重做日志检查点页面上的校验和永远不会被禁用。

+   `innodb_log_compressed_pages`

    | 命令行格式 | `--innodb-log-compressed-pages[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `innodb_log_compressed_pages` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `ON` |

    指定是否将重新压缩的页面图像写入重做日志。当对压缩数据进行更改时可能会发生重新压缩。

    `innodb_log_compressed_pages`默认启用，以防止在恢复期间使用不同版本的`zlib`压缩算法时可能发生的损坏。如果您确定`zlib`版本不会更改，可以禁用`innodb_log_compressed_pages`以减少修改压缩数据的工作负载的重做日志生成。

    要衡量启用或禁用`innodb_log_compressed_pages`的影响，请比较相同工作负载下两种设置的重做日志生成情况。衡量重做日志生成的选项包括观察`SHOW ENGINE INNODB STATUS`输出中`LOG`部分中的`Log sequence number`（LSN），或监视`Innodb_os_log_written`状态，查看写入重做日志文件的字节数。

    有关相关信息，请参阅第 17.9.1.6 节，“OLTP 工作负载的压缩”。

+   `innodb_log_file_size`

    | 命令行格式 | `--innodb-log-file-size=#` |
    | --- | --- |
    | 已弃用 | 8.0.30 |
    | 系统变量 | `innodb_log_file_size` |
    | 作用范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `50331648` |
    | 最小值 | `4194304` |
    | 最大值 | `512GB / innodb_log_files_in_group` |
    | 单位 | 字节 |

    注意

    `innodb_log_file_size`和`innodb_log_files_in_group`在 MySQL 8.0.30 中已弃用。这些变量已被`innodb_redo_log_capacity`取代。有关更多信息，请参阅第 17.6.5 节，“重做日志”。

    每个日志组中每个日志文件的大小（`innodb_log_file_size` * `innodb_log_files_in_group`）。日志文件的组合大小（`innodb_log_file_size` * `innodb_log_files_in_group`）不能超过略小于 512GB 的最大值。例如，一对 255GB 的日志文件接近限制但不超过。默认值为 48MB。

    通常，日志文件的组合大小应足够大，以便服务器可以平滑处理工作负载活动的峰值和谷值，这通常意味着有足够的重做日志空间来处理超过一个小时的写入活动。数值越大，在缓冲池中需要的检查点刷新活动就越少，从而节省磁盘 I/O。更大的日志文件也会使崩溃恢复变慢。

    `innodb_log_file_size`的最小值为 4MB。

    有关更多信息，请参阅重做日志配置。有关一般 I/O 调优建议，请参阅第 10.5.8 节，“优化 InnoDB 磁盘 I/O”。

    如果启用了`innodb_dedicated_server`，并且未明确定义，则`innodb_log_file_size`的值将自动配置。有关更多信息，请参阅第 17.8.12 节，“为专用 MySQL 服务器启用自动配置”。

+   `innodb_log_files_in_group`

    | 命令行格式 | `--innodb-log-files-in-group=#` |
    | --- | --- |
    | 已弃用 | 8.0.30 |
    | 系统变量 | `innodb_log_files_in_group` |
    | 作用域 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `2` |
    | 最小值 | `2` |
    | 最大值 | `100` |

    注意

    `innodb_log_file_size`和`innodb_log_files_in_group`在 MySQL 8.0.30 中已弃用。这些变量已被`innodb_redo_log_capacity`取代。有关更多信息，请参阅第 17.6.5 节，“重做日志”。

    日志文件在日志组中的数量。`InnoDB`以循环方式写入这些文件。默认（也是推荐的）值为 2。文件的位置由`innodb_log_group_home_dir`指定。日志文件的组合大小（`innodb_log_file_size` * `innodb_log_files_in_group`）最多可达 512GB。

    有关更多信息，请参阅重做日志配置。

    如果启用了`innodb_dedicated_server`，并且未明确定义，则`innodb_log_files_in_group`将自动配置。有关更多信息，请参阅第 17.8.12 节，“为专用 MySQL 服务器启用自动配置”。

+   `innodb_log_group_home_dir`

    | 命令行格式 | `--innodb-log-group-home-dir=dir_name` |
    | --- | --- |
    | 系统变量 | `innodb_log_group_home_dir` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 目录名称 |

    `InnoDB` 重做日志文件的目录路径。

    有关信息，请参阅重做日志配置。

+   `innodb_log_spin_cpu_abs_lwm`

    | 命令行格式 | `--innodb-log-spin-cpu-abs-lwm=#` |
    | --- | --- |
    | 系统变量 | `innodb_log_spin_cpu_abs_lwm` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `80` |
    | 最小值 | `0` |
    | 最大值 | `4294967295` |

    定义了用户线程在等待刷新的重做时不再自旋的最小 CPU 使用率。该值表示为 CPU 核心使用率的总和。例如，80 的默认值是单个 CPU 核心的 80%。在具有多核处理器的系统上，值为 150 表示一个 CPU 核心的 100%使用率加上第二个 CPU 核心的 50%使用率。

    有关信息，请参阅第 10.5.4 节，“优化 InnoDB 重做日志”。

+   `innodb_log_spin_cpu_pct_hwm`

    | 命令行格式 | `--innodb-log-spin-cpu-pct-hwm=#` |
    | --- | --- |
    | 系统变量 | `innodb_log_spin_cpu_pct_hwm` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `50` |
    | 最小值 | `0` |
    | 最大值 | `100` |

    定义了用户线程在等待刷新的重做时不再自旋的最大 CPU 使用率。该值表示为所有 CPU 核心的总处理能力的百分比。默认值为 50%。例如，对于具有四个 CPU 核心的服务器，两个 CPU 核心的 100%使用率是组合 CPU 处理能力的 50%。

    `innodb_log_spin_cpu_pct_hwm`变量遵守处理器亲和性。例如，如果服务器有 48 个核心，但**mysqld**进程只固定在四个 CPU 核心上，则其他 44 个 CPU 核心将被忽略。

    有关信息，请参阅第 10.5.4 节，“优化 InnoDB 重做日志”。

+   `innodb_log_wait_for_flush_spin_hwm`

    | 命令行格式 | `--innodb-log-wait-for-flush-spin-hwm=#` |
    | --- | --- |
    | 系统变量 | `innodb_log_wait_for_flush_spin_hwm` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `400` |
    | 最小值 | `0` |
    | 最大值（64 位平台） | `2**64-1` |
    | 最大值（32 位平台） | `2**32-1` |
    | 单位 | 微秒 |

    定义了超过最大平均日志刷新时间的值，用户线程在等待刷新的重做时不再旋转。默认值为 400 微秒。

    有关信息，请参见 Section 10.5.4, “Optimizing InnoDB Redo Logging”。

+   `innodb_log_write_ahead_size`

    | 命令行格式 | `--innodb-log-write-ahead-size=#` |
    | --- | --- |
    | 系统变量 | `innodb_log_write_ahead_size` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `8192` |
    | 最小值 | `512（日志文件块大小）` |
    | 最大值 | `等于 innodb_page_size` |
    | 单位 | 字节 |

    定义了重做日志的预写块大小，以字节为单位。为避免“读写”，将`innodb_log_write_ahead_size`设置为与操作系统或文件系统缓存块大小相匹配。默认设置为 8192 字节。当由于重做日志块与操作系统或文件系统的缓存块大小不匹配而导致重做日志块未完全缓存在操作系统或文件系统中时，就会发生读写。

    `innodb_log_write_ahead_size`的有效值是`InnoDB`日志文件块大小（2^n）的倍数。最小值是`InnoDB`日志文件块大小（512）。当指定最小值时，不会发生预写。最大值等于`innodb_page_size`值。如果为`innodb_log_write_ahead_size`指定的值大于`innodb_page_size`值，则`innodb_log_write_ahead_size`设置将被截断为`innodb_page_size`值。

    如果`innodb_log_write_ahead_size`的值相对于操作系统或文件系统缓存块大小设置得太低，会导致“写入时读取”。如果值设置得太高，可能会对`fsync`性能产生轻微影响，因为多个块一次被写入。

    有关更多信息，请参阅第 10.5.4 节，“优化 InnoDB 重做日志记录”。

+   `innodb_log_writer_threads`

    | 命令行格式 | `--innodb-log-writer-threads[={OFF&#124;ON}]` |
    | --- | --- |
    | 引入版本 | 8.0.22 |
    | 系统变量 | `innodb_log_writer_threads` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `ON` |

    启用专用日志写入线程，用于将重做日志记录从日志缓冲区写入系统缓冲区并将系统缓冲区刷新到重做日志文件。专用日志写入线程可以提高高并发系统的性能，但对于低并发系统，禁用专用日志写入线程可以提供更好的性能。

    有关更多信息，请参阅第 10.5.4 节，“优化 InnoDB 重做日志记录”。

+   `innodb_lru_scan_depth`

    | 命令行格式 | `--innodb-lru-scan-depth=#` |
    | --- | --- |
    | 系统变量 | `innodb_lru_scan_depth` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `1024` |
    | 最小值 | `100` |
    | 最大值（64 位平台） | `2**64-1` |
    | 最大值（32 位平台） | `2**32-1` |

    影响`InnoDB`缓冲池的刷新操作算法和启发式的参数。主要关注性能专家调整 I/O 密集型工作负载。它指定了每个缓冲池实例，页面清理线程在 LRU 页面列表中扫描多深以查找要刷新的脏页。这是一个每秒执行一次的后台操作。

    一般来说，比默认值小的设置对大多数工作负载都适用。如果值远高于必要值，可能会影响性能。只有在典型工作负载下有多余 I/O 容量时才考虑增加该值。相反，如果写入密集型工作负载使 I/O 容量饱和，降低该值，尤其是在有大缓冲池的情况下。

    在调整`innodb_lru_scan_depth`时，从一个较低值开始，并将设置向上配置，目标是很少看到零空闲页。此外，考虑在更改缓冲池实例数时调整`innodb_lru_scan_depth`，因为`innodb_lru_scan_depth` * `innodb_buffer_pool_instances`定义了页面清理线程每秒执行的工作量。

    有关相关信息，请参见第 17.8.3.5 节，“配置缓冲池刷新”。有关一般 I/O 调优建议，请参见第 10.5.8 节，“优化 InnoDB 磁盘 I/O”。

+   `innodb_max_dirty_pages_pct`

    | 命令行格式 | `--innodb-max-dirty-pages-pct=#` |
    | --- | --- |
    | 系统变量 | `innodb_max_dirty_pages_pct` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 数值 |
    | 默认值 | `90` |
    | 最小值 | `0` |
    | 最大值 | `99.999` |

    `InnoDB`尝试从缓冲池中刷新数据，以使脏页的百分比不超过此值。

    `innodb_max_dirty_pages_pct` 设置了刷新活动的目标，不影响刷新速率。有关管理刷新速率的信息，请参见第 17.8.3.5 节，“配置缓冲池刷新”。

    有关相关信息，请参见第 17.8.3.5 节，“配置缓冲池刷新”。有关一般 I/O 调优建议，请参见第 10.5.8 节，“优化 InnoDB 磁盘 I/O”。

+   `innodb_max_dirty_pages_pct_lwm`

    | 命令行格式 | `--innodb-max-dirty-pages-pct-lwm=#` |
    | --- | --- |
    | 系统变量 | `innodb_max_dirty_pages_pct_lwm` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 数值 |
    | 默认值 | `10` |
    | 最小值 | `0` |
    | 最大值 | `99.999` |

    定义了表示脏页百分比的低水位，当预刷写启用以控制脏页比率时。值为 0 会完全禁用预刷写行为。配置的值应始终低于`innodb_max_dirty_pages_pct`的值。更多信息，请参见第 17.8.3.5 节，“配置缓冲池刷新”。

+   `innodb_max_purge_lag`

    | 命令行格式 | `--innodb-max-purge-lag=#` |
    | --- | --- |
    | 系统变量 | `innodb_max_purge_lag` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `0` |
    | 最小值 | `0` |
    | 最大值 | `4294967295` |

    定义了期望的最大清除延迟。如果超过此值，将对`INSERT`、`UPDATE`和`DELETE`操作施加延迟，以便清除赶上。默认值为 0，这意味着没有最大清除延迟和没有延迟。

    更多信息，请参见第 17.8.9 节，“清除配置”。

+   `innodb_max_purge_lag_delay`

    | 命令行格式 | `--innodb-max-purge-lag-delay=#` |
    | --- | --- |
    | 系统变量 | `innodb_max_purge_lag_delay` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `0` |
    | 最小值 | `0` |
    | 最大值 | `10000000` |
    | 单位 | 微秒 |

    指定了当超过`innodb_max_purge_lag`阈值时施加的延迟的最大延迟时间（以微秒为单位）。指定的`innodb_max_purge_lag_delay`值是由`innodb_max_purge_lag`公式计算的延迟期限的上限。

    更多信息，请参见第 17.8.9 节，“清除配置”。

+   `innodb_max_undo_log_size`

    | 命令行格式 | `--innodb-max-undo-log-size=#` |
    | --- | --- |
    | 系统变量 | `innodb_max_undo_log_size` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `1073741824` |
    | 最小值 | `10485760` |
    | 最大值 | `2**64-1` |
    | 单位 | 字节 |

    定义回滚表空间的阈值大小。如果回滚表空间超过阈值，则在启用`innodb_undo_log_truncate`时可以标记为截断。默认值为 1073741824 字节（1024 MiB）。

    更多信息，请参见截断回滚表空间。

+   `innodb_merge_threshold_set_all_debug`

    | 命令行格式 | `--innodb-merge-threshold-set-all-debug=#` |
    | --- | --- |
    | 系统变量 | `innodb_merge_threshold_set_all_debug` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `50` |
    | 最小值 | `1` |
    | 最大值 | `50` |

    定义索引页的页面满百分比值，该值覆盖当前字典缓存中当前所有索引的`MERGE_THRESHOLD`设置。仅当使用`WITH_DEBUG` **CMake**选项编译调试支持时才可用此选项。有关相关信息，请参见第 17.8.11 节，“配置索引页合并阈值”。

+   `innodb_monitor_disable`

    | 命令行格式 | `--innodb-monitor-disable={counter&#124;module&#124;pattern&#124;all}` |
    | --- | --- |
    | 系统变量 | `innodb_monitor_disable` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 字符串 |

    此变量充当开关，禁用`InnoDB`度量计数器。可以使用信息模式`INNODB_METRICS`表查询计数器数据。有关使用信息，请参见第 17.15.6 节，“InnoDB INFORMATION_SCHEMA 度量表”。

    `innodb_monitor_disable='latch'` 禁用 `SHOW ENGINE INNODB MUTEX` 的统计信息收集。更多信息，请参见 第 15.7.7.15 节，“SHOW ENGINE 语句”。

+   `innodb_monitor_enable`

    | 命令行格式 | `--innodb-monitor-enable={counter&#124;module&#124;pattern&#124;all}` |
    | --- | --- |
    | 系统变量 | `innodb_monitor_enable` |
    | 作用域 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 字符串 |

    该变量充当开关，启用 `InnoDB` 的 度量计数器。可以使用信息模式 `INNODB_METRICS` 表查询计数器数据。有关使用信息，请参见 第 17.15.6 节，“InnoDB INFORMATION_SCHEMA Metrics Table”。

    `innodb_monitor_enable='latch'` 启用统计信息收集，用于 `SHOW ENGINE INNODB MUTEX`。更多信息，请参见 第 15.7.7.15 节，“SHOW ENGINE 语句”。

+   `innodb_monitor_reset`

    | 命令行格式 | `--innodb-monitor-reset={counter&#124;module&#124;pattern&#124;all}` |
    | --- | --- |
    | 系统变量 | `innodb_monitor_reset` |
    | 作用域 | ��局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 枚举 |
    | 默认值 | `NULL` |
    | 有效值 | `counter``module``pattern``all` |

    该变量充当开关，将 `InnoDB` 的 度量计数器 的计数值重置为零。可以使用信息模式 `INNODB_METRICS` 表查询计数器数据。有关使用信息，请参见 第 17.15.6 节，“InnoDB INFORMATION_SCHEMA Metrics Table”。

    `innodb_monitor_reset='latch'` 重置由 `SHOW ENGINE INNODB MUTEX` 报告的统计信息。更多信息，请参见 第 15.7.7.15 节，“SHOW ENGINE 语句”。

+   `innodb_monitor_reset_all`

    | 命令行格式 | `--innodb-monitor-reset-all={counter&#124;module&#124;pattern&#124;all}` |
    | --- | --- |
    | 系统变量 | `innodb_monitor_reset_all` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 枚举 |
    | 默认值 | `NULL` |
    | 有效值 | `counter``module``pattern``all` |

    此变量充当开关，重置所有`InnoDB`度量计数器的值（最小值、最大值等）。可以使用信息模式`INNODB_METRICS`表查询计数器数据。有关使用信息，请参见第 17.15.6 节，“InnoDB INFORMATION_SCHEMA 度量表”。

+   `innodb_numa_interleave`

    | 命令行格式 | `--innodb-numa-interleave[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `innodb_numa_interleave` |
    | 作用范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔 |
    | 默认值 | `OFF` |

    启用 NUMA 交错内存策略以分配`InnoDB`缓冲池。当启用`innodb_numa_interleave`时，NUMA 内存策略设置为`MPOL_INTERLEAVE`用于**mysqld**进程。分配`InnoDB`缓冲池后，NUMA 内存策略将恢复为`MPOL_DEFAULT`。要使`innodb_numa_interleave`选项可用，必须在启用 NUMA 的 Linux 系统上编译 MySQL。

    **CMake**根据当前平台是否支持`NUMA`，设置默认`WITH_NUMA`值。有关更多信息，请参见第 2.8.7 节，“MySQL 源配置选项”。

+   `innodb_old_blocks_pct`

    | 命令行格式 | `--innodb-old-blocks-pct=#` |
    | --- | --- |
    | 系统变量 | `innodb_old_blocks_pct` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `37` |
    | 最小值 | `5` |
    | 最大值 | `95` |

    指定用于旧块 sublist 的`InnoDB`缓冲池的近似百分比。值的范围为 5 到 95。默认值为 37（即池的 3/8）。通常与`innodb_old_blocks_time`结合使用。

    更多信息，请参阅第 17.8.3.3 节，“使缓冲池具有扫描抵抗力”。有关缓冲池管理、LRU 算法和驱逐策略的信息，请参阅第 17.5.1 节，“缓冲池”。

+   `innodb_old_blocks_time`

    | 命令行格式 | `--innodb-old-blocks-time=#` |
    | --- | --- |
    | 系统变量 | `innodb_old_blocks_time` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `1000` |
    | 最小值 | `0` |
    | 最大值 | `2**32-1` |
    | 单位 | 毫秒 |

    非零值可防止缓冲池被仅在短时间内引用的数据填满，例如在全表扫描期间。增加此值可提供更多保护，防止全表扫描干扰缓冲池中缓存的数据。

    指定一个块插入到旧 sublist 后，在第一次访问之后必须在那里停留多长时间（以毫秒为单位），然后才能移动到新的子列表。如果值为 0，则插入到旧子列表的块在第一次访问时立即移动到新子列表，无论插入后多久发生访问。如果值大于 0，则块保留在旧子列表中，直到第一次访问后至少经过那么多毫秒。例如，值为 1000 会导致块在第一次访问后在旧子列表中停留 1 秒，然后才能移动到新子列表。

    默认值为 1000。

    这个变量通常与`innodb_old_blocks_pct`结合使用。更多信息，请参阅第 17.8.3.3 节，“使缓冲池具有扫描抵抗力”。有关缓冲池管理、LRU 算法和驱逐策略的信息，请参阅第 17.5.1 节，“缓冲池”。

+   `innodb_online_alter_log_max_size`

    | 命令行格式 | `--innodb-online-alter-log-max-size=#` |
    | --- | --- |
    | 系统变量 | `innodb_online_alter_log_max_size` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `134217728` |
    | 最小值 | `65536` |
    | 最大值 | `2**64-1` |
    | 单位 | 字节 |

    指定`InnoDB`表的在线 DDL 操作期间使用的临时日志文件大小上限（以字节为单位）。每个正在创建的索引或正在更改的表都有一个这样的日志文件。此日志文件存储在 DDL 操作期间插入、更新或删除的表中的数据。临时日志文件在需要时通过`innodb_sort_buffer_size`的值扩展，最多扩展到`innodb_online_alter_log_max_size`指定的最大值。如果临时日志文件超过上限大小，`ALTER TABLE`操作将失败，并且所有未提交的并发 DML 操作都将被回滚。因此，此选项的较大值允许在在线 DDL 操作期间发生更多的 DML 操作，但也会延长 DDL 操作结束时表被锁定以应用日志中数据的时间。

+   `innodb_open_files`

    | 命令行格式 | `--innodb-open-files=#` |
    | --- | --- |
    | 系统变量 | `innodb_open_files` |
    | 范围 | 全局 |
    | 动态 (≥ 8.0.28) | 是 |
    | 动态 (≤ 8.0.27) | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `-1`（表示自动调整大小；不要分配此字面值） |
    | 最小值 | `10` |
    | 最大值 | `2147483647` |

    指定`InnoDB`在同一时间内可以打开的文件的最大数量。最小值为 10。如果禁用了`innodb_file_per_table`，默认值为 300；否则，默认值为 300 或`table_open_cache`设置中的较高值。

    截至 MySQL 8.0.28，可以使用`SELECT innodb_set_open_files_limit(*N*)`语句在运行时设置`innodb_open_files`限制，其中*N*是所需的`innodb_open_files`限制；例如：

    ```sql
    mysql> SELECT innodb_set_open_files_limit(1000);
    ```

    该语句执行一个存储过程，设置新的限制。如果过程成功，则返回新设置限制的值；否则，返回失败消息。

    不允许使用`SET`语句设置`innodb_open_files`。要在运行时设置`innodb_open_files`，请使用上述描述的`SELECT innodb_set_open_files_limit(*`N`*)`语句。

    设置`innodb_open_files=default`不受支持。只允许整数值。

    从 MySQL 8.0.28 开始，为防止非 LRU 管理文件占用整个`innodb_open_files`限制，非 LRU 管理文件限制为`innodb_open_files`限制的 90%，这样就为 LRU 管理文件保留了`innodb_open_files`限制的 10%。

    从 MySQL 8.0.24 到 MySQL 8.0.27，临时表空间文件不计入`innodb_open_files`限制。

+   `innodb_optimize_fulltext_only`

    | 命令行格式 | `--innodb-optimize-fulltext-only[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `innodb_optimize_fulltext_only` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    改变了`OPTIMIZE TABLE`在`InnoDB`表上的操作方式。旨在在具有`FULLTEXT`索引的`InnoDB`表的维护操作期间暂时启用。

    默认情况下，`OPTIMIZE TABLE` 重新组织表的聚簇索引中的数据。当启用此选项时，`OPTIMIZE TABLE` 将跳过表数据的重新组织，而是处理`InnoDB` `FULLTEXT`索引中新添加、删除和更新的标记数据。更多信息，请参见优化 InnoDB 全文索引。

+   `innodb_page_cleaners`

    | 命令行格式 | `--innodb-page-cleaners=#` |
    | --- | --- |
    | 系统变量 | `innodb_page_cleaners` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `4` |
    | 最小值 | `1` |
    | 最大值 | `64` |

    从缓冲池实例刷新脏页的页面清理线程数量。页面清理线程执行刷新列表和 LRU 刷新。当存在多个页面清理线程时，每个缓冲池实例的缓冲池刷新任务将分派给空闲的页面清理线程。`innodb_page_cleaners` 的默认值为 4。如果页面清理线程的数量超过缓冲池实例的数量，则 `innodb_page_cleaners` 会自动设置为与 `innodb_buffer_pool_instances` 相同的值。

    如果在将脏页从缓冲池实例刷新到数据文件时，您的工作负载受写入 IO 限制，并且系统硬件有可用容量，则增加页面清理线程的数量可能有助于提高写入 IO 吞吐量。

    多线程页面清理支持扩展到关闭和恢复阶段。

    `setpriority()` 系统调用在支持的 Linux 平台上使用，在 **mysqld** 执行用户被授权为帮助页面刷新跟上当前工作负载而给 `page_cleaner` 线程优先级高于其他 MySQL 和 `InnoDB` 线程的情况下。`setpriority()` 支持由此 `InnoDB` 启动消息指示：

    ```sql
    [Note] InnoDB: If the mysqld execution user is authorized, page cleaner
    thread priority can be changed. See the man page of setpriority().
    ```

    对于不由 systemd 管理服务器启动和关闭的系统，可以在 `/etc/security/limits.conf` 中配置 **mysqld** 执行用户授权。例如，如果 **mysqld** 在 `mysql` 用户下运行，则可以通过将以下行添加到 `/etc/security/limits.conf` 来授权 `mysql` 用户：

    ```sql
    mysql              hard    nice       -20
    mysql              soft    nice       -20
    ```

    对于由 systemd 管理的系统，可以通过在本地化 systemd 配置文件中指定 `LimitNICE=-20` 来实现相同的效果。例如，在 `/etc/systemd/system/mysqld.service.d/override.conf` 中创建一个名为 `override.conf` 的文件，并添加以下条目：

    ```sql
    [Service]
    LimitNICE=-20
    ```

    创建或更改 `override.conf` 后，重新加载 systemd 配置，然后告诉 systemd 重新启动 MySQL 服务：

    ```sql
    systemctl daemon-reload
    systemctl restart mysqld  # RPM platforms
    systemctl restart mysql   # Debian platforms
    ```

    有关使用本地化 systemd 配置文件的更多信息，请参见为 MySQL 配置 systemd。

    授权 **mysqld** 执行用户后，使用 **cat** 命令验证 **mysqld** 进程配置的 `Nice` 限制：

    ```sql
    $> cat /proc/*mysqld_pid*/limits | grep nice
    Max nice priority         18446744073709551596 18446744073709551596
    ```

+   `innodb_page_size`

    | 命令行格式 | `--innodb-page-size=#` |
    | --- | --- |
    | 系统变量 | `innodb_page_size` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 枚举 |
    | 默认值 | `16384` |
    | 有效值 | `4096``8192``16384``32768``65536` |

    指定`InnoDB`表空间的页面大小。值可以以字节或千字节为单位指定。例如，可以将 16KB 页面大小值指定为 16384、16KB 或 16k。

    `innodb_page_size`只能在初始化 MySQL 实例之前配置，之后不能更改。如果未指定任何值，则实例将使用默认页面大小进行初始化。请参阅第 17.8.1 节，“InnoDB 启动配置”。

    对于 32KB 和 64KB 页面大小，最大行长度约为 16000 字节。当`innodb_page_size`设置为 32KB 或 64KB 时，不支持`ROW_FORMAT=COMPRESSED`。对于`innodb_page_size=32KB`，扩展大小为 2MB。对于`innodb_page_size=64KB`，扩展大小为 4MB。在使用 32KB 或 64KB 页面大小时，`innodb_log_buffer_size`应至少设置为 16M（默认值）。

    默认的 16KB 页面大小或更大适用于各种工作负载，特别是涉及表扫描和涉及大量更新的 DML 操作的查询。较小的页面大小可能对涉及许多小写入的 OLTP 工作负载更有效，当单个页面包含许多行时，争用可能是一个问题。较小的页面在通常使用小块大小的 SSD 存储设备上也可能更有效。保持`InnoDB`页面大小接近存储设备块大小可以最大程度地减少被重写到磁盘的未更改数据量。

    第一个系统表空间数据文件（`ibdata1`）的最小文件大小取决于`innodb_page_size`值。有关更多信息，请参阅`innodb_data_file_path`选项描述。

    使用特定`InnoDB`页面大小的 MySQL 实例不能使用来自使用不同页面大小的实例的数据文件或日志文件。

    有关一般 I/O 调优建议，请参阅第 10.5.8 节，“优化 InnoDB 磁盘 I/O”。

+   `innodb_parallel_read_threads`

    | 命令行格式 | `--innodb-parallel-read-threads=#` |
    | --- | --- |
    | 引入 | 8.0.14 |
    | 系统变量 | `innodb_parallel_read_threads` |
    | 作用范围 | 会话 |
    | 动态 | 是 |
    | `SET_VAR` Hint Applies | 否 |
    | 类型 | 整数 |
    | 默认值 | `4` |
    | 最小值 | `1` |
    | 最大值 | `256` |

    定义可用于并行集群索引读取的线程数。截至 MySQL 8.0.17 版本，支持对分区进行并行扫描。并行读取线程可以提高`CHECK TABLE`性能。`InnoDB`在`CHECK TABLE`操作期间两次读取聚簇索引。第二次读取可以并行执行。此功能不适用于二级索引扫描。必须将`innodb_parallel_read_threads`会话变量设置为大于 1 的值，才能进行并行集群索引读取。执行并行集群索引读取的实际线程数由`innodb_parallel_read_threads`设置或要扫描的索引子树数量决定，取两者中较小的值。扫描期间读取到缓冲池的页面保持在缓冲池 LRU 列表的尾部，以便在需要空闲缓冲池页面时可以快速丢弃它们。

    截至 MySQL 8.0.17 版本，最大并行读取线程数（256）是所有客户端连接的总线程数。如果达到线程限制，连接将回退到使用单个线程。

+   `innodb_print_all_deadlocks`

    | 命令行格式 | `--innodb-print-all-deadlocks[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `innodb_print_all_deadlocks` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` Hint Applies | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    当启用此选项时，`InnoDB` 用户事务中所有 死锁 的信息都记录在 `mysqld` 错误日志 中。否则，您只会看到关于最后一个死锁的信息，使用 `SHOW ENGINE INNODB STATUS` 命令。偶尔的 `InnoDB` 死锁并不一定是问题，因为 `InnoDB` 立即检测到条件并自动回滚其中一个事务。如果应用程序没有适当的错误处理逻辑来检测回滚并重试其操作，您可能会使用此选项来排除死锁发生的原因。大量的死锁可能表明需要重新构造为多个表发出 DML 或 `SELECT ... FOR UPDATE` 语句的事务，以便每个事务以相同顺序访问表，从而避免死锁条件。

    有关更多信息，请参见 第 17.7.5 节，“InnoDB 中的死锁”。

+   `innodb_print_ddl_logs`

    | 命令行格式 | `--innodb-print-ddl-logs[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `innodb_print_ddl_logs` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    启用此选项会导致 MySQL 将 DDL 日志写入 `stderr`。更多信息，请参见 查看 DDL 日志。

+   `innodb_purge_batch_size`

    | 命令行格式 | `--innodb-purge-batch-size=#` |
    | --- | --- |
    | 系统变量 | `innodb_purge_batch_size` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `300` |
    | 最小值 | `1` |
    | 最大值 | `5000` |

    定义了一批从 历史列表 中解析和处理的撤销日志页数。在多线程清除配置中，协调员清除线程将 `innodb_purge_batch_size` 除以 `innodb_purge_threads`，并将该数量的页分配给每个清除线程。`innodb_purge_batch_size` 变量还定义了清除在通过撤销日志的每 128 次迭代后释放的撤销日志页数。

    `innodb_purge_batch_size` 选项旨在与 `innodb_purge_threads` 设置结合进行高级性能调优。大多数用户不需要更改 `innodb_purge_batch_size` 的默认值。

    有关更多信息，请参见 第 17.8.9 节，“清除配置”。

+   `innodb_purge_threads` 

    | 命令行格式 | `--innodb-purge-threads=#` |
    | --- | --- |
    | 系统变量 | `innodb_purge_threads` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `4` |
    | 最小值 | `1` |
    | 最大值 | `32` |

    专用于 `InnoDB` 清除 操作的后台线程数。增加该值会创建额外的清除线程，可以提高在执行多个表的 DML 操作的系统上的效率。

    有关更多信息，请参见 第 17.8.9 节，“清除配置”。

+   `innodb_purge_rseg_truncate_frequency`

    | 命令行格式 | `--innodb-purge-rseg-truncate-frequency=#` |
    | --- | --- |
    | 系统变量 | `innodb_purge_rseg_truncate_frequency` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `128` |
    | 最小值 | `1` |
    | 最大值 | `128` |

    定义清除系统释放回滚段的频率，以调用清除的次数来衡量。在回滚段被释放之前，无法截断撤消表空间。通常，清除系统每调用 128 次就会释放一次回滚段。默认值为 128。减少此值会增加清除线程释放回滚段的频率。

    `innodb_purge_rseg_truncate_frequency` 旨在与 `innodb_undo_log_truncate` 一起使用。有关更多信息，请参见 截断撤消表空间。

+   `innodb_random_read_ahead`

    | 命令行格式 | `--innodb-random-read-ahead[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `innodb_random_read_ahead` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `关闭` |

    启用随机预读技术，优化`InnoDB` I/O。

    有关不同类型预读请求的性能考虑的详细信息，请参见第 17.8.3.4 节，“配置 InnoDB 缓冲池预取（预读）”。有关一般 I/O 调优建议，请参见第 10.5.8 节，“优化 InnoDB 磁盘 I/O”。

+   `innodb_read_ahead_threshold`

    | 命令行格式 | `--innodb-read-ahead-threshold=#` |
    | --- | --- |
    | 系统变量 | `innodb_read_ahead_threshold` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `56` |
    | 最小值 | `0` |
    | 最大值 | `64` |

    控制`InnoDB`用于预取页面到缓冲池的线性预读的灵敏度。如果`InnoDB`按顺序从一个 extent（64 页）中至少读取`innodb_read_ahead_threshold`页，它将启动对整个后续 extent 的异步读取。允许的值范围是 0 到 64。值为 0 表示禁用预读。对于默认值 56，`InnoDB`必须按顺序从一个 extent 中至少读取 56 页，才能启动对后续 extent 的异步读取。

    了解通过预读机制读取了多少页面，以及这些页面中有多少被从缓冲池中驱逐而从未被访问，对于微调`innodb_read_ahead_threshold`设置可能是有用的。`SHOW ENGINE INNODB STATUS`输出显示了来自`Innodb_buffer_pool_read_ahead`和`Innodb_buffer_pool_read_ahead_evicted`全局状态变量的计数器信息，这些变量报告了通过预读请求带入缓冲池的页面数，以及这些页面从未被访问而从缓冲池中驱逐的页面数。这些状态变量报告自上次服务器重启以来的全局值。

    `SHOW ENGINE INNODB STATUS`还显示了预读页面的读取速率以及这些页面被驱逐而没有被访问的速率。每秒平均值基于自上次调用`SHOW ENGINE INNODB STATUS`以来收集的统计数据，并显示在`SHOW ENGINE INNODB STATUS`输出的`BUFFER POOL AND MEMORY`部分。

    更多信息，请参见第 17.8.3.4 节，“配置 InnoDB 缓冲池预取（预读）”。有关一般 I/O 调优建议，请参见第 10.5.8 节，“优化 InnoDB 磁盘 I/O”。

+   `innodb_read_io_threads`

    | 命令行格式 | `--innodb-read-io-threads=#` |
    | --- | --- |
    | 系统变量 | `innodb_read_io_threads` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `4` |
    | 最小值 | `1` |
    | 最大值 | `64` |

    用于`InnoDB`读操作的 I/O 线程数。其写线程的对应项是`innodb_write_io_threads`。有关更多信息，请参见第 17.8.5 节，“配置后台 InnoDB I/O 线程数”。有关一般 I/O 调优建议，请参见第 10.5.8 节，“优化 InnoDB 磁盘 I/O”。

    注意

    在 Linux 系统上，使用默认设置的`innodb_read_io_threads`、`innodb_write_io_threads`和 Linux 的`aio-max-nr`设置运行多个 MySQL 服务器（通常超过 12 个）可能超出系统限制。理想情况下，增加`aio-max-nr`设置；作为解决方法，您可以减少一个或两个 MySQL 变量的设置。

+   `innodb_read_only`

    | 命令行格式 | `--innodb-read-only[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `innodb_read_only` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    以只读模式启动`InnoDB`。用于在只读媒体上分发数据库应用程序或数据集。也可用于数据仓库，在多个实例之间共享相同的数据目录。有关更多信息，请参见第 17.8.2 节，“配置 InnoDB 进行只读操作”。

    以前，启用`innodb_read_only`系统变量仅阻止了`InnoDB`存储引擎的表的创建和删除。从 MySQL 8.0 开始，启用`innodb_read_only`将阻止所有存储引擎的这些操作。任何存储引擎的表创建和删除操作都会修改`mysql`系统数据库中的数据字典表，但这些表使用`InnoDB`存储引擎，在启用`innodb_read_only`时无法修改。同样的原则也适用于其他需要修改数据字典表的表操作。例如：

    +   如果启用了`innodb_read_only`系统变量，`ANALYZE TABLE`可能会失败，因为它无法更新使用`InnoDB`的数据字典中的统计表。对于更新键分布的`ANALYZE TABLE`操作，即使操作更新了表本身（例如，如果它是一个`MyISAM`表），也可能会发生失败。要获取更新后的分布统计信息，请设置`information_schema_stats_expiry=0`。

    +   `ALTER TABLE *`tbl_name`* ENGINE=*`engine_name`*` 失败，因为它更新了存储引擎的指定，这些信息存储在数据字典中。

    此外，MySQL 8.0 中 `mysql` 系统数据库中的其他表使用 `InnoDB` 存储引擎。将这些表设置为只读会导致修改它们的操作受到限制。例如：

    +   帐户管理语句，如 `CREATE USER` 和 `GRANT` 失败，因为授权表使用了 `InnoDB`。

    +   `INSTALL PLUGIN` 和 `UNINSTALL PLUGIN` 插件管理语句失败，因为 `mysql.plugin` 系统表使用了 `InnoDB`。

    +   `CREATE FUNCTION` 和 `DROP FUNCTION` 可加载函数管理语句失败，因为 `mysql.func` 系统表使用了 `InnoDB`。

+   `innodb_redo_log_archive_dirs`

    | 命令行格式 | `--innodb-redo-log-archive-dirs` |
    | --- | --- |
    | 引入 | 8.0.17 |
    | 系统变量 | `innodb_redo_log_archive_dirs` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 字符串 |
    | 默认值 | `NULL` |

    定义标记目录，可以在其中创建重做日志归档文件。您可以在分号分隔的列表中定义多个标记目录。例如：

    ```sql
    innodb_redo_log_archive_dirs='label1:/backups1;label2:/backups2'
    ```

    标签可以是任意字符的字符串，但不允许使用冒号（:）。空标签也是允许的，但在这种情况下仍然需要冒号(:)。

    必须指定路径，并且目录必须存在。路径可以包含冒号（':'），但不允许使用分号（;）。

+   `innodb_redo_log_capacity`

    | 命令行格式 | `--innodb-redo-log-capacity=#` |
    | --- | --- |
    | 引入 | 8.0.30 |
    | 系统变量 | `innodb_redo_log_capacity` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `104857600` |
    | 最小值 | `8388608` |
    | 最大值 (≥ 8.0.34) | `549755813888` |
    | 最大值 (≥ 8.0.30, ≤ 8.0.33) | `137438953472` |
    | 单位 | 字节 |

    定义重做日志文件占用的磁盘空间量。

    此变量取代了`innodb_log_files_in_group`和`innodb_log_file_size`变量。当定义了`innodb_redo_log_capacity`设置时，`innodb_log_files_in_group`和`innodb_log_file_size`设置将被忽略；否则，这些设置将用于计算`innodb_redo_log_capacity`设置（`innodb_log_files_in_group` * `innodb_log_file_size` = `innodb_redo_log_capacity`）。如果没有设置这些变量中的任何一个，重做日志容量将设置为`innodb_redo_log_capacity`的默认值。

    有关更多信息，请参见第 17.6.5 节，“重做日志”。

+   `innodb_redo_log_encrypt`

    | 命令行格式 | `--innodb-redo-log-encrypt[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `innodb_redo_log_encrypt` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    控制使用`InnoDB`数据静态加密功能加密的表的重做日志数据的加密。默认情况下，重做日志数据的加密是禁用的。有关更多信息，请参见重做日志加密。

+   `innodb_replication_delay`

    | 命令行格式 | `--innodb-replication-delay=#` |
    | --- | --- |
    | 系统变量 | `innodb_replication_delay` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `0` |
    | 最小值 | `0` |
    | 最大值 | `4294967295` |
    | 单位 | 毫秒 |

    如果达到`innodb_thread_concurrency`时，在副本服务器上的复制线程延迟���以毫秒为单位）。

+   `innodb_rollback_on_timeout`

    | 命令行格式 | `--innodb-rollback-on-timeout[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `innodb_rollback_on_timeout` |
    | 作用范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    `InnoDB` 默认仅在事务超时时回滚最后一个语句。如果指定了 `--innodb-rollback-on-timeout`，事务超时会导致 `InnoDB` 中止并回滚整个事务。

    更多信息，请参见 第 17.21.5 节，“InnoDB 错误处理”。

+   `innodb_rollback_segments`

    | 命令行格式 | `--innodb-rollback-segments=#` |
    | --- | --- |
    | 系统变量 | `innodb_rollback_segments` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `128` |
    | 最小值 | `1` |
    | 最大值 | `128` |

    `innodb_rollback_segments` 定义了分配给每个撤消表空间和生成撤消记录的全局临时表空间的撤消段数量。每个撤消段支持的事务数量取决于 `InnoDB` 页大小和分配给每个事务的撤消日志数量。更多信息，请参见 第 17.6.6 节，“撤消日志”。

    相关信息，请参见 第 17.3 节，“InnoDB 多版本”。有关撤消表空间的信息，请参见 第 17.6.3.4 节，“撤消表空间”。

+   `innodb_saved_page_number_debug`

    | 命令行格式 | `--innodb-saved-page-number-debug=#` |
    | --- | --- |
    | 系统变量 | `innodb_saved_page_number_debug` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `0` |
    | 最小值 | `0` |
    | 最大值 | `2**32-1` |

    保存一个页面编号。设置 `innodb_fil_make_page_dirty_debug` 选项会使由 `innodb_saved_page_number_debug` 定义的页面变脏。只有在使用 `WITH_DEBUG` **CMake** 选项编译支持调试时，才能使用 `innodb_saved_page_number_debug` 选项。

+   `innodb_segment_reserve_factor`

    | 命令行格式 | `--innodb-segment-reserve-factor=#` |
    | --- | --- |
    | 引入版本 | 8.0.26 |
    | 系统变量 | `innodb_segment_reserve_factor` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 数值 |
    | 默认值 | `12.5` |
    | 最小值 | `0.03` |
    | 最大值 | `40` |

    定义表空间文件段页面保留为空页面的百分比。该设置适用于每表文件和通用表空间。`innodb_segment_reserve_factor`默认设置为 12.5%，这与之前的 MySQL 版本中保留的页面百分比相同。

    更多信息，请参阅配置保留文件段页面的百分比。

+   `innodb_sort_buffer_size`

    | 命令行格式 | `--innodb-sort-buffer-size=#` |
    | --- | --- |
    | 系统变量 | `innodb_sort_buffer_size` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `1048576` |
    | 最小值 | `65536` |
    | 最大值 | `67108864` |
    | 单位 | 字节 |

    此变量定义：

    +   用于创建或重建二级索引的在线 DDL 操作的排序缓冲区大小。然而，从 MySQL 8.0.27 开始，这一责任被`innodb_ddl_buffer_size`变量所取代。

    +   在在线 DDL 操作期间记录并发 DML 时，临时日志文件扩展的量，以及临时日志文件读缓冲区和写缓冲区的大小。

    有关更多信息，请参阅第 17.12.3 节，“在线 DDL 空间要求”。

+   `innodb_spin_wait_delay`

    | 命令行格式 | `--innodb-spin-wait-delay=#` |
    | --- | --- |
    | 系统变量 | `innodb_spin_wait_delay` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `6` |
    | 最小值 | `0` |
    | 最大值（64 位平台，≤ 8.0.13） | `2**64-1` |
    | 最大值（32 位平台，≤ 8.0.13） | `2**32-1` |
    | 最大值（≥ 8.0.14） | `1000` |

    自旋锁之间轮询的最大延迟。此机制的底层实现因硬件和操作系统的组合而异，因此延迟不对应固定的时间间隔。

    可与`innodb_spin_wait_pause_multiplier`变量结合使用，以更好地控制自旋锁轮询延迟的持续时间。

    更多信息，请参阅��17.8.8 节，“配置自旋锁轮询”。

+   `innodb_spin_wait_pause_multiplier`

    | 命令行格式 | `--innodb-spin-wait-pause-multiplier=#` |
    | --- | --- |
    | 引入版本 | 8.0.16 |
    | 系统变量 | `innodb_spin_wait_pause_multiplier` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `50` |
    | 最小值 | `0` |
    | 最大值 | `100` |

    定义一个乘数值，用于确定线程在等待获取互斥锁或读写锁时发生自旋等待循环中的 PAUSE 指令数量。

    更多信息，请参阅第 17.8.8 节，“配置自旋锁轮询”。

+   `innodb_stats_auto_recalc`

    | 命令行格式 | `--innodb-stats-auto-recalc[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `innodb_stats_auto_recalc` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `ON` |

    导致`InnoDB`在表中的数据发生重大更改后自动重新计算持久性统计信息。阈值为表中行数的 10%。此设置适用于启用`innodb_stats_persistent`选项时创建的表。还可以通过在`CREATE TABLE`或`ALTER TABLE`语句中指定`STATS_PERSISTENT=1`来配置自动统计信息重新计算。用于生成统计信息的采样数据量由`innodb_stats_persistent_sample_pages`变量控制。

    更多信息，请参阅第 17.8.10.1 节，“配置持久性优化器统计参数”。

+   `innodb_stats_include_delete_marked`

    | 命令行格式 | `--innodb-stats-include-delete-marked[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `innodb_stats_include_delete_marked` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    默认情况下，`InnoDB`在计算统计信息时读取未提交的数据。在未提交事务删除表中的行的情况下，`InnoDB`在计算行估计和索引统计信息时排除了被标记为删除的记录，这可能导致其他使用事务隔离级别为`READ UNCOMMITTED`并发操作表的事务执行计划不佳。为避免这种情况，可以启用`innodb_stats_include_delete_marked`以确保`InnoDB`在计算持久优化器统计信息时包括被标记为删除的记录。

    当启用`innodb_stats_include_delete_marked`时，`ANALYZE TABLE`在重新计算统计信息时考虑了删除标记记录。

    `innodb_stats_include_delete_marked`是一个影响所有`InnoDB`表的全局设置。仅适用于持久优化器统计信息。

    有关相关信息，请参阅第 17.8.10.1 节，“配置持久优化器统计参数”。

+   `innodb_stats_method`

    | 命令行格式 | `--innodb-stats-method=value` |
    | --- | --- |
    | 系统变量 | `innodb_stats_method` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 枚举 |
    | 默认值 | `nulls_equal` |
    | 有效取值 | `nulls_equal``nulls_unequal``nulls_ignored` |

    服务器在收集关于`InnoDB`表索引值分布的统计信息时如何处理`NULL`值。允许的取值为`nulls_equal`、`nulls_unequal`和`nulls_ignored`。对于`nulls_equal`，所有`NULL`索引值被视为相等，并形成一个大小等于`NULL`值数量的值组。对于`nulls_unequal`，`NULL`值被视为不相等，每个`NULL`形成一个大小为 1 的独立值组。对于`nulls_ignored`，`NULL`值被忽略。

    生成表统计信息的方法会影响优化器选择查询执行时使用的索引，详见第 10.3.8 节，“InnoDB 和 MyISAM 索引统计信息收集”。

+   `innodb_stats_on_metadata`

    | 命令行格式 | `--innodb-stats-on-metadata[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `innodb_stats_on_metadata` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    此选项仅在优化器统计信息配置为非持久性时适用。当禁用`innodb_stats_persistent`或使用`STATS_PERSISTENT=0`创建或更改单个表时，优化器统计信息不会持久保存到磁盘。有关更多信息，请参阅第 17.8.10.2 节，“配置非持久性优化器统计参数”。

    当启用`innodb_stats_on_metadata`时，`InnoDB`在元数据语句（如`SHOW TABLE STATUS`）或访问信息模式`TABLES`或`STATISTICS`表时更新非持久性统计信息。（这些更新类似于`ANALYZE TABLE`的操作。）禁用时，`InnoDB`在这些操作期间不会更新统计信息。保持禁用设置可以提高对具有大量表或索引的模式的访问速度。它还可以改善涉及`InnoDB`表的查询的执行计划的稳定性。

    要更改设置，请发出语句`SET GLOBAL innodb_stats_on_metadata=*`mode`*`，其中`*`mode`*`为`ON`或`OFF`（或`1`或`0`）。更改设置需要足够设置全局系统变量的权限（请参阅第 7.1.9.1 节，“系统变量权限”），并立即影响所有连接的操作。

+   `innodb_stats_persistent`

    | 命令行格式 | `--innodb-stats-persistent[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `innodb_stats_persistent` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔 |
    | 默认值 | `ON` |

    指定`InnoDB`索引统计信息是否持久化到磁盘。否则，统计信息可能会经常重新计算，这可能导致查询执行计划的变化。此设置在创建表时与每个表一起存储。您可以在创建表之前在全局级别设置`innodb_stats_persistent`，或者使用`CREATE TABLE`和`ALTER TABLE`语句的`STATS_PERSISTENT`子句覆盖系统级设置，并为单个表配置持久性统计信息。

    更多信息，请参阅 第 17.8.10.1 节，“配置持久性优化器统计参数”。

+   `innodb_stats_persistent_sample_pages`

    | 命令行格式 | `--innodb-stats-persistent-sample-pages=#` |
    | --- | --- |
    | 系统变量 | `innodb_stats_persistent_sample_pages` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `20` |
    | 最小值 | `1` |
    | 最大值 | `18446744073709551615` |

    用于估算索引列的基数和其他统计信息的索引页的数量，例如`ANALYZE TABLE`计算的那些。增加该值可以提高索引统计信息的准确性，从而可以改善查询执行计划，但会增加执行`ANALYZE TABLE`时`InnoDB`表的 I/O 开销。更多信息，请参阅 第 17.8.10.1 节，“配置持久性优化器统计参数”。

    注意

    为 `innodb_stats_persistent_sample_pages` 设置一个较高的值可能导致 `ANALYZE TABLE` 执行时间过长。要估算 `ANALYZE TABLE` 访问的数据库页数，请参见 Section 17.8.10.3, “Estimating ANALYZE TABLE Complexity for InnoDB Tables”。

    当为表启用 `innodb_stats_persistent` 时，`innodb_stats_persistent_sample_pages` 才适用；当禁用 `innodb_stats_persistent` 时，将使用 `innodb_stats_transient_sample_pages`。

+   `innodb_stats_transient_sample_pages`

    | 命令行格式 | `--innodb-stats-transient-sample-pages=#` |
    | --- | --- |
    | 系统变量 | `innodb_stats_transient_sample_pages` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `8` |
    | 最小值 | `1` |
    | 最大值 | `18446744073709551615` |

    在估算索引列的基数和其他统计信息时要采样的索引页数，例如通过 `ANALYZE TABLE` 计算的那些。默认值为 8。增加该值可以提高索引统计信息的准确性，从而改善查询执行计划，但会增加打开 `InnoDB` 表或重新计算统计信息时的 I/O。有关更多信息，请参见 Section 17.8.10.2, “Configuring Non-Persistent Optimizer Statistics Parameters”。

    注意

    为 `innodb_stats_transient_sample_pages` 设置一个较高的值可能导致 `ANALYZE TABLE` 执行时间过长。要估算 `ANALYZE TABLE` 访问的数据库页数，请参见 Section 17.8.10.3, “Estimating ANALYZE TABLE Complexity for InnoDB Tables”。

    `innodb_stats_transient_sample_pages`仅在为表禁用`innodb_stats_persistent`时适用；当启用`innodb_stats_persistent`时，将使用`innodb_stats_persistent_sample_pages`。取代`innodb_stats_sample_pages`。更多信息，请参见第 17.8.10.2 节，“配置非持久性优化器统计参数”。

+   `innodb_status_output`

    | 命令行格式 | `--innodb-status-output[={OFF | ON}]` |
    | --- | --- | --- |
    | 系统变量 | `innodb_status_output` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔 |
    | 默认值 | `OFF` |

    启用或禁用标准`InnoDB`监视器的定期输出。也与`innodb_status_output_locks`结合使用，以启用或禁用`InnoDB`锁监视器的定期输出。更多信息，请参见第 17.17.2 节，“启用 InnoDB 监视器”。

+   `innodb_status_output_locks`

    | 命令行格式 | `--innodb-status-output-locks[={OFF | ON}]` |
    | --- | --- | --- |
    | 系统变量 | `innodb_status_output_locks` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔 |
    | 默认值 | `OFF` |

    启用或禁用`InnoDB`锁监视器。启用后，`InnoDB`锁监视器会在`SHOW ENGINE INNODB STATUS`输出和定期输出中打印有关锁的附加信息，并在 MySQL 错误日志中打印。`InnoDB`锁监视器的定期输出作为标准`InnoDB`监视器输出的一部分打印。因此，必须启用标准`InnoDB`监视器，`InnoDB`锁监视器才能定期将数据打印到 MySQL 错误日志中。更多信息，请参见第 17.17.2 节，“启用 InnoDB 监视器”。

+   `innodb_strict_mode`

    | 命令行格式 | `--innodb-strict-mode[={OFF | ON}]` |
    | --- | --- | --- |
    | 系统变量 | `innodb_strict_mode` |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `ON` |

    当启用`innodb_strict_mode`时，`InnoDB`在检查无效或不兼容表选项时返回错误而不是警告。

    它检查`KEY_BLOCK_SIZE`、`ROW_FORMAT`、`DATA DIRECTORY`、`TEMPORARY`和`TABLESPACE`选项是否与彼此和其他设置兼容。

    `innodb_strict_mode=ON`还在创建或更改表时启用行大小检查，以防止由于记录对所选页面大小过大而导致`INSERT`或`UPDATE`失败。

    `您可以在启动`mysqld`时在命令行上启用或禁用`innodb_strict_mode`，或者在 MySQL 配置文件中启用或禁用`innodb_strict_mode`。您还可以使用语句`SET [GLOBAL|SESSION] innodb_strict_mode=*`mode`*`在运行时启用或禁用`innodb_strict_mode`，其中`*`mode`*`为`ON`或`OFF`。更改`GLOBAL`设置需要具有足够权限设置全局系统变量的权限（请参阅 Section 7.1.9.1, “系统变量权限”），并影响随后连接的所有客户端的操作。任何客户端都可以更改`innodb_strict_mode`的`SESSION`设置，该设置仅影响该客户端。`

    `从 MySQL 8.0.26 开始，设置此系统变量的会话值是受限制的操作。会话用户必须具有足够的权限来设置受限制的会话变量。请参阅 Section 7.1.9.1, “系统变量权限”。`

+   ``innodb_sync_array_size``

    `| 命令行格式 | `--innodb-sync-array-size=#` |

    | 系统变量 | `innodb_sync_array_size` |
    | --- | --- |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `1` |
    | 最小值 | `1` |
    | 最大值 | `1024` |

    定义互斥锁/锁等待数组的大小。增加该值会分割用于协调线程的内部数据结构，以提高在具有大量等待线程的工作负载中的并发性。必须在 MySQL 实例启动时配置此设置，之后无法更改。建议增加该值以适应频繁产生大量等待线程的工作负载，通常大于 768 个。`

+   ``innodb_sync_spin_loops``

    `| 命令行格式 | `--innodb-sync-spin-loops=#` |

    | System Variable | `innodb_sync_spin_loops` |
    | --- | --- |
    | Scope | Global |
    | Dynamic | Yes |
    | `SET_VAR` Hint Applies | No |
    | Type | Integer |
    | Default Value | `30` |
    | Minimum Value | `0` |
    | Maximum Value | `4294967295` |

    线程等待`InnoDB`互斥锁被释放的次数，超过后线程将被挂起。

+   ``innodb_sync_debug``

    `| Command-Line Format | `--innodb-sync-debug[={OFF&#124;ON}]` |

    | System Variable | `innodb_sync_debug` |
    | --- | --- |
    | Scope | Global |
    | Dynamic | No |
    | `SET_VAR` Hint Applies | No |
    | Type | Boolean |
    | Default Value | `OFF` |

    启用`InnoDB`存储引擎的同步调试检查。此选项仅在使用`WITH_DEBUG` **CMake**选项编译时才可用。

+   ``innodb_table_locks``

    `| Command-Line Format | `--innodb-table-locks[={OFF&#124;ON}]` |

    | System Variable | `innodb_table_locks` |
    | --- | --- |
    | Scope | Global, Session |
    | Dynamic | Yes |
    | `SET_VAR` Hint Applies | No |
    | Type | Boolean |
    | Default Value | `ON` |

    如果`autocommit = 0`，`InnoDB`会遵守`LOCK TABLES`；MySQL 在`LOCK TABLES ... WRITE`直到所有其他线程都释放对表的锁之前不会返回。`innodb_table_locks`的默认值为 1，这意味着`LOCK TABLES`会导致 InnoDB 在`autocommit = 0`时内部锁定表。

    `innodb_table_locks = 0`对使用`LOCK TABLES ... WRITE`显式锁定的表没有影响。但对通过触发器隐式锁定的表或通过`LOCK TABLES ... READ`读写锁定的表有影响。

    有关更多信息，请参见第 17.7 节，“InnoDB 锁定和事务模型”。

+   ``innodb_temp_data_file_path``

    `| Command-Line Format | `--innodb-temp-data-file-path=file_name` |

    | 系统变量 | `innodb_temp_data_file_path` |
    | --- | --- |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 字符串 |
    | 默认值 | `ibtmp1:12M:autoextend` |

    定义全局临时表空间数据文件的相对路径、名称、大小和属性。全局临时表空间存储对用户创建的临时表所做更改的回滚段。

    如果未为 `innodb_temp_data_file_path` 指定任何值，则默认行为是在 `innodb_data_home_dir` 目录中创建一个名为 `ibtmp1` 的单个自动扩展数据文件。初始文件大小略大于 12MB。

    全局临时表空间数据文件规范的语法包括文件名、文件大小以及 `autoextend` 和 `max` 属性���

    ```sql
    *file_name*:*file_size*[:autoextend[:max:*max_file_size*]]
    ```

    全局临时表空间数据文件的名称不能与另一个 `InnoDB` 数据文件相同。任何无法创建全局临时表空间数据文件或出现错误的情况都被视为致命，服务器启动将被拒绝。

    文件大小通过在大小值后附加 `K`、`M` 或 `G` 来指定为 KB、MB 或 GB。文件大小之和必须略大于 12MB。

    单个文件的大小限制由操作系统确定。在支持大文件的操作系统上，文件大小可以超过 4GB。不支持使用原始磁盘分区用于全局临时表空间数据文件。

    `autoextend` 和 `max` 属性只能用于在 `innodb_temp_data_file_path` 设置中指定的最后一个数据文件。例如：

    ```sql
    [mysqld]
    innodb_temp_data_file_path=ibtmp1:50M;ibtmp2:12M:autoextend:max:500M
    ```

    `autoextend` 选项会在数据文件用尽空间时自动增加大小。`autoextend` 增量默认为 64MB。要修改增量，请更改 `innodb_autoextend_increment` 变量设置。

    全局临时表空间数据文件的目录路径是通过连接由 `innodb_data_home_dir` 和 `innodb_temp_data_file_path` 定义的路径形成的。

    在以只读模式运行 `InnoDB` 之前，将 `innodb_temp_data_file_path` 设置为数据目录之外的位置。路径必须相对于数据目录。例如：

    ```sql
    --innodb-temp-data-file-path=../../../tmp/ibtmp1:12M:autoextend
    ```

    有关更多信息，请参阅 全局临时表空间。`

+   `innodb_temp_tablespaces_dir`

    `| 命令行格式 | `--innodb-temp-tablespaces-dir=dir_name` |

    | 引入版本 | 8.0.13 |
    | --- | --- |
    | 系统变量 | `innodb_temp_tablespaces_dir` |
    | 范围 | 全局 |
    | Dynamic | No |
    | `SET_VAR` Hint Applies | No |
    | 类型 | 目录名称 |
    | Default Value | `#innodb_temp` |

    定义`InnoDB`在启动时创建会话临时表空间池的位置。默认位置是数据目录中的`#innodb_temp`目录。允许使用完全限定路径或相对于数据目录的路径。

    截至 MySQL 8.0.16，会话临时表空间始终存储用户创建的临时表和使用`InnoDB`创建的优化器内部临时表。（以前，内部临时表的磁盘存储引擎由不再支持的`internal_tmp_disk_storage_engine`系统变量确定。请参阅磁盘内部临时表的存储引擎。）

    更多信息，请参阅会话临时表空间。

+   ``innodb_thread_concurrency``

    `| 命令行格式 | `--innodb-thread-concurrency=#` |

    | 系统变量 | `innodb_thread_concurrency` |
    | --- | --- |
    | Scope | Global |
    | 动态 | 是 |
    | `SET_VAR` Hint Applies | No |
    | 类型 | 整数 |
    | Default Value | `0` |
    | 最小值 | `0` |
    | 最大值 | `1000` |

    定义`InnoDB`内允许的最大线程数。值为 0（默认值）被解释为无限并发（无限制）。此变量旨在用于高并发系统的性能调优。

    `InnoDB`试图保持`InnoDB`内部线程的数量小于或等于`innodb_thread_concurrency`限制。等待锁的线程不计入并发执行线程数。

    正确的设置取决于工作负载和计算环境。如果你的 MySQL 实例与其他应用程序共享 CPU 资源，或者你的工作负载或并发用户数量正在增加，考虑设置这个变量。测试一系列值以确定提供最佳性能的设置。`innodb_thread_concurrency`是一个动态变量，允许在实时测试系统上尝试不同的设置。如果某个设置表现不佳，你可以快速将`innodb_thread_concurrency`设置回 0。

    使用以下准则来帮助找到并保持适当的设置：

    +   如果工作负载的并发用户线程数量始终很少且不影响性能，设置`innodb_thread_concurrency=0`（无限制）。

    +   如果你的工作负载始终很重或偶尔会出现峰值，设置一个`innodb_thread_concurrency`值，并调整直到找到提供最佳性能的线程数量。例如，假设你的系统通常有 40 到 50 个用户，但有时用户数量增加到 60、70 或更多。通过测试，你发现限制为 80 个并发用户时性能基本稳定。在这种情况下，将`innodb_thread_concurrency`设置为 80。

    +   如果你不希望`InnoDB`为用户线程使用超过一定数量的虚拟 CPU（例如 20 个虚拟 CPU），将`innodb_thread_concurrency`设置为这个数字（或根据性能测试可能更低）。如果你的目标是将 MySQL 与其他应用程序隔离开来，考虑将`mysqld`进程专门绑定到虚拟 CPU。然而，请注意，独占绑定可能导致硬件使用不佳，如果`mysqld`进程没有持续繁忙。在这种情况下，你可以将`mysqld`进程绑定到虚拟 CPU，但允许其他应用程序使用部分或全部虚拟 CPU。

        注意

        从操作系统的角度来看，使用资源管理解决方案来管理 CPU 时间在应用程序之间的共享可能比绑定`mysqld`进程更可取。例如，你可以在其他关键进程*不运行*时将 90% 的虚拟 CPU 时间分配给特定应用程序，而在其他关键进程*运行*时将该值缩减到 40%。

    +   在某些情况下，最佳的`innodb_thread_concurrency`设置可能小于虚拟 CPU 的数量。

    +   如果`innodb_thread_concurrency`值过高，可能会导致性能下降，因为系统内部和资源的争用增加。

    +   定期监视和分析您的系统。工作负载、用户数量或计算环境的变化可能需要您调整`innodb_thread_concurrency`设置。

    值为 0 会禁用`SHOW ENGINE INNODB STATUS`输出中`ROW OPERATIONS`部分的`InnoDB`内部查询和队列中查询计数器。

    有关更多信息，请参阅第 17.8.4 节，“配置 InnoDB 的线程并发性”。

+   ``innodb_thread_sleep_delay``

    `| 命令行格式 | `--innodb-thread-sleep-delay=#` |

    | 系统变量 | `innodb_thread_sleep_delay` |
    | --- | --- |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` Hint Applies | 否 |
    | 类型 | 整数 |
    | 默认值 | `10000` |
    | 最小值 | `0` |
    | 最大值 | `1000000` |
    | 单位 | 微秒 |

    `InnoDB`线程在加入`InnoDB`队列之前睡眠的时间，单位为微秒。默认值为 10000。值为 0 会禁用睡眠。您可以将`innodb_adaptive_max_sleep_delay`设置为允许的最高值，以供`innodb_thread_sleep_delay`使用，`InnoDB`会根据当前线程调度活动自动调整`innodb_thread_sleep_delay`的值，以适应系统轻载或接近满负荷运行时的情况，这种动态调整有助于线程调度机制在系统轻载或接近满负荷运行时平稳工作。

    有关更多信息，请参阅第 17.8.4 节，“配置 InnoDB 的线程并发性”。

+   ``innodb_tmpdir``

    `| 命令行格式 | `--innodb-tmpdir=dir_name` |

    | 系统变量 | `innodb_tmpdir` |
    | --- | --- |
    | 作用范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR` Hint Applies | 否 |
    | 类型 | 目录名称 |
    | 默认值 | `NULL` |

    用于定义在线`ALTER TABLE`操作重建表时创建的临时排序文件的替代目录。

    在线`ALTER TABLE`操作重建表时，还会在与原始表相同目录中创建一个*中间*表文件。`innodb_tmpdir`选项不适用于中间表文件。

    有效值是除 MySQL 数据目录路径之外的任何目录路径。如果值为 NULL（默认值），则临时文件将在 MySQL 临时目录（Unix 上的`$TMPDIR`，Windows 上的`%TEMP%`，或由`--tmpdir`配置选项指定的目录）中创建。如果指定了目录，则仅在使用`SET`语句配置`innodb_tmpdir`时才检查目录的存在和权限。如果在目录字符串中提供了符号链接，则符号链接将被解析并存储为绝对路径。路径不应超过 512 字节。如果将`innodb_tmpdir`设置为无效目录，则在线`ALTER TABLE`操作将报告错误。`innodb_tmpdir`覆盖了 MySQL 的`tmpdir`设置，但仅适用于在线`ALTER TABLE`操作。

    配置`innodb_tmpdir`需要`FILE`权限。

    `innodb_tmpdir` 选项的引入是为了帮助避免在`tmpfs`文件系统上的临时文件目录溢出。这种溢出可能是由在线`ALTER TABLE`操作中创建的大型临时排序文件导致的，这些操作会重建表。

    在复制环境中，只有当所有服务器具有相同的操作系统环境时，才考虑复制`innodb_tmpdir`设置。否则，在运行重建表的在线`ALTER TABLE`操作时，复制`innodb_tmpdir`设置可能导致复制失败。如果服务器操作环境不同，建议在每台服务器上单独配置`innodb_tmpdir`。

    更多信息，请参见第 17.12.3 节，“在线 DDL 空间要求”。有关在线`ALTER TABLE`操作的信息，请参见第 17.12 节，“InnoDB 和在线 DDL”。

+   ``innodb_trx_purge_view_update_only_debug``

    `| 命令行格式 | `--innodb-trx-purge-view-update-only-debug[={OFF&#124;ON}]` |

    | 系统变量 | `innodb_trx_purge_view_update_only_debug` |
    | --- | --- |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    暂停删除标记记录的清除，同时允许更新清除视图。此选项人为地创建了一种情况，即清除视图已更新但尚未执行清除。此选项仅在使用`WITH_DEBUG` **CMake**选项编译时才可用。

+   ``innodb_trx_rseg_n_slots_debug``

    | 命令行格式 | `--innodb-trx-rseg-n-slots-debug=#` |

    | 系统变量 | `innodb_trx_rseg_n_slots_debug` |
    | --- | --- |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `0` |
    | 最小值 | `0` |
    | 最大值 | `1024` |

    设置一个调试标志，将`TRX_RSEG_N_SLOTS`限制为`trx_rsegf_undo_find_free`函数中查找撤销日志段的空闲槽位的给定值。此选项仅在使用`WITH_DEBUG` **CMake**选项编译时才可用。

+   ``innodb_undo_directory``

    | 命令行格式 | `--innodb-undo-directory=dir_name` |

    | 系统变量 | `innodb_undo_directory` |
    | --- | --- |
    | 作用范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 目录名称 |

    `InnoDB`创建撤销表空间的路径。通常用于将撤销表空间放置在不同的存储设备上。

    没有默认值（为 NULL）。如果未定义`innodb_undo_directory`变量，则撤销表空间将在数据目录中创建。

    MySQL 实例初始化时创建的默认撤销表空间（`innodb_undo_001`和`innodb_undo_002`）始终驻留在由`innodb_undo_directory`变量定义的目录中。

    使用`CREATE UNDO TABLESPACE`语法创建的撤销表空间将在由`innodb_undo_directory`变量定义的目录中创建，如果没有指定不同的路径。

    更多信息，请参阅第 17.6.3.4 节，“撤销表空间”。

+   ``innodb_undo_log_encrypt``

    | 命令行格式 | `--innodb-undo-log-encrypt[={OFF&#124;ON}]` |

    | 系统变量 | `innodb_undo_log_encrypt` |
    | --- | --- |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    控制使用 `InnoDB` 数据静态加密功能加密的表的撤消日志数据的加密。仅适用于驻留在单独的撤消表空间中的撤消日志。请参阅第 17.6.3.4 节，“撤消表空间”。不支持对驻留在系统表空间中的撤消日志数据进行加密。有关更多信息，请参阅撤消日志加密。

+   ``innodb_undo_log_truncate``

    `| 命令行格式 | `--innodb-undo-log-truncate[={OFF&#124;ON}]` |

    | 系统变量 | `innodb_undo_log_truncate` |
    | --- | --- |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `ON` |

    启用后，超过由 `innodb_max_undo_log_size` 定义的阈值的撤消表空间将被标记为截断。只能截断撤消表空间。不支持截断驻留在系统表空间中的撤消日志。要进行截断，必须至少有两个撤消表空间。

    `innodb_purge_rseg_truncate_frequency` 变量可用于加快截断撤消表空间的速度。

    更多信息，请参阅截断撤消表空间。

+   ``innodb_undo_tablespaces``

    `| 命令行格式 | `--innodb-undo-tablespaces=#` |

    | 已弃用 | 是 |
    | --- | --- |
    | 系统变量 | `innodb_undo_tablespaces` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `2` |
    | 最小值 | `2` |
    | 最大值 | `127` |

    定义 `InnoDB` 使用的撤消表空间的数量。默认值和最小值均为 2。

    注意

    `innodb_undo_tablespaces` 变量已被弃用，并且自 MySQL 8.0.14 版本起不再可配置。预计在未来的版本中将被移除。

    有关更多信息，请参见第 17.6.3.4 节，“撤销表空间”。`

+   ``innodb_use_fdatasync``

    | 命令行格式 | `--innodb-use-fdatasync[={OFF&#124;ON}]` |

    | 引入版本 | 8.0.26 |
    | --- | --- |
    | 系统变量 | `innodb_use_fdatasync` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    在支持`fdatasync()`系统调用的平台上，启用`innodb_use_fdatasync`变量允许使用`fdatasync()`而不是`fsync()`系统调用进行操作系统刷新。`fdatasync()`调用不会刷新文件元数据，除非需要进行后续数据检索，从而提供潜在的性能优势。

    一些`innodb_flush_method`设置的子集，如`fsync`、`O_DSYNC`和`O_DIRECT`使用`fsync()`系统调用。在使用这些设置时，`innodb_use_fdatasync`变量是适用的。`

+   ``innodb_use_native_aio``

    | 命令行格式 | `--innodb-use-native-aio[={OFF&#124;ON}]` |

    | 系统变量 | `innodb_use_native_aio` |
    | --- | --- |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `ON` |

    指定是否使用 Linux 异步 I/O 子系统。此变量仅适用于 Linux 系统，并且不能在服务器运行时更改。通常情况下，您不需要配置此选项，因为它默认启用。

    `InnoDB` 在 Windows 系统上具有的异步 I/O 功能也适用于 Linux 系统。（其他类 Unix 系统继续使用同步 I/O 调用。）此功能提高了通常在`SHOW ENGINE INNODB STATUS\G`输出中显示许多待处理读取/写入的 I/O 密集型系统的可伸缩性。

    使用大量`InnoDB` I/O 线程运行，尤其是在同一台服务器上运行多个这样的实例，可能会超出 Linux 系统的容量限制。在这种情况下，您可能会收到以下错误：

    ```sql
    EAGAIN: The specified maxevents exceeds the user's limit of available events.
    ```

    通常情况下，您可以通过将更高的限制写入`/proc/sys/fs/aio-max-nr`来解决此错误。

    然而，如果操作系统中异步 I/O 子系统出现问题导致 `InnoDB` 无法启动，您可以使用 `innodb_use_native_aio=0` 启动服务器。在启动过程中，如果 `InnoDB` 检测到潜在问题，例如 `tmpdir` 位置、`tmpfs` 文件系统和不支持 `tmpfs` 上的 AIO 的 Linux 内核的组合，此选项也可能会被自动禁用。

    欲了解更多信息，请参阅 第 17.8.6 节，“在 Linux 上使用异步 I/O”。

+   ``innodb_validate_tablespace_paths``

    `| 命令行格式 | `--innodb-validate-tablespace-paths[={OFF&#124;ON}]` |

    | 引入版本 | 8.0.21 |
    | --- | --- |
    | 系统变量 | `innodb_validate_tablespace_paths` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔 |
    | 默认值 | `ON` |

    控制表空间文件路径验证。在启动时，`InnoDB` 验证已知表空间文件的路径与数据字典中存储的表空间文件路径是否匹配，以防表空间文件已移至其他位置。`innodb_validate_tablespace_paths` 变量允许禁用表空间路径验证。此功能适用于表空间文件未移动的环境。禁用路径验证可提高在具有大量表空间文件的系统上的启动时间。

    警告

    在移动表空间文件后以禁用表空间路径验证启动服务器可能导致未定义行为。

    欲了解更多信息，请参阅 第 17.6.3.7 节，“禁用表空间路径验证”。

+   ``innodb_version``

    `InnoDB` 版本号。在 MySQL 8.0 中，`InnoDB` 的单独版本编号不适用，此值与服务器的 `version` 编号相同。`

+   ``innodb_write_io_threads``

    `| 命令行格式 | `--innodb-write-io-threads=#` |

    | 系统变量 | `innodb_write_io_threads` |
    | --- | --- |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `4` |
    | 最小值 | `1` |
    | 最大值 | `64` |

    `InnoDB`写操作的 I/O 线程数。默认值为 4。读线程的对应值是`innodb_read_io_threads`。更多信息，请参见第 17.8.5 节，“配置后台 InnoDB I/O 线程数”。有关一般 I/O 调优建议，请参见第 10.5.8 节，“优化 InnoDB 磁盘 I/O”。

    注意

    在 Linux 系统上，使用默认设置运行多个 MySQL 服务器（通常超过 12 个），`innodb_read_io_threads`，`innodb_write_io_threads`，以及 Linux `aio-max-nr`设置可能超出系统限制。理想情况下，增加`aio-max-nr`设置；作为解决方法，您可以减少一个或两个 MySQL 变量的设置。

    考虑到`sync_binlog`的价值，它控制着二进制日志与磁盘的同步。

    有关一般 I/O 调优建议，请参见第 10.5.8 节，“优化 InnoDB 磁盘 I/O”。
