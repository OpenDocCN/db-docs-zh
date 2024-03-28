# 30.4.3 sys 模式视图

> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-schema-views.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-schema-views.html)

30.4.3.1 主机 host_summary 和 x$host_summary 视图

30.4.3.2 主机 host_summary_by_file_io 和 x$host_summary_by_file_io 视图

30.4.3.3 主机 host_summary_by_file_io_type 和 x$host_summary_by_file_io_type 视图

30.4.3.4 主机 host_summary_by_stages 和 x$host_summary_by_stages 视图

30.4.3.5 主机 host_summary_by_statement_latency 和 x$host_summary_by_statement_latency 视图

30.4.3.6 主机 host_summary_by_statement_type 和 x$host_summary_by_statement_type 视图

30.4.3.7 innodb_buffer_stats_by_schema 和 x$innodb_buffer_stats_by_schema 视图

30.4.3.8 innodb_buffer_stats_by_table 和 x$innodb_buffer_stats_by_table 视图

30.4.3.9 innodb_lock_waits 和 x$innodb_lock_waits 视图

30.4.3.10 IO 线程 io_by_thread_by_latency 和 x$io_by_thread_by_latency 视图

30.4.3.11 IO 全局 io_global_by_file_by_bytes 和 x$io_global_by_file_by_bytes 视图

30.4.3.12 IO 全局 io_global_by_file_by_latency 和 x$io_global_by_file_by_latency 视图

30.4.3.13 IO 全局 io_global_by_wait_by_bytes 和 x$io_global_by_wait_by_bytes 视图

30.4.3.14 IO 全局 io_global_by_wait_by_latency 和 x$io_global_by_wait_by_latency 视图

30.4.3.15 最新的 latest_file_io 和 x$latest_file_io 视图

30.4.3.16 主机内存 memory_by_host_by_current_bytes 和 x$memory_by_host_by_current_bytes 视图

30.4.3.17 内存 memory_by_thread_by_current_bytes 和 x$memory_by_thread_by_current_bytes 视图

30.4.3.18 用户内存 memory_by_user_by_current_bytes 和 x$memory_by_user_by_current_bytes 视图

30.4.3.19 全局内存 memory_global_by_current_bytes 和 x$memory_global_by_current_bytes 视图

30.4.3.20 全局内存 memory_global_total 和 x$memory_global_total 视图

30.4.3.21 指标 metrics 视图

30.4.3.22 进程列表 processlist 和 x$processlist 视图

30.4.3.23 ps_check_lost_instrumentation 视图

30.4.3.24 模式自增列 schema_auto_increment_columns 视图

30.4.3.25 模式索引统计 schema_index_statistics 和 x$schema_index_statistics 视图

30.4.3.26 模式对象概览 schema_object_overview 视图

30.4.3.27 schema_redundant_indexes 和 x$schema_flattened_keys 视图

30.4.3.28 schema_table_lock_waits 和 x$schema_table_lock_waits 视图

30.4.3.29 schema_table_statistics 和 x$schema_table_statistics 视图

30.4.3.30 schema_table_statistics_with_buffer 和 x$schema_table_statistics_with_buffer 视图

30.4.3.31 schema_tables_with_full_table_scans 和 x$schema_tables_with_full_table_scans 视图

30.4.3.32 schema_unused_indexes 视图

30.4.3.33 session 和 x$session 视图

30.4.3.34 session_ssl_status 视图

30.4.3.35 statement_analysis 和 x$statement_analysis 视图

30.4.3.36 statements_with_errors_or_warnings 和 x$statements_with_errors_or_warnings 视图

30.4.3.37 statements_with_full_table_scans 和 x$statements_with_full_table_scans 视图

30.4.3.38 statements_with_runtimes_in_95th_percentile 和 x$statements_with_runtimes_in_95th_percentile 视图

30.4.3.39 statements_with_sorting 和 x$statements_with_sorting 视图

30.4.3.40 statements_with_temp_tables 和 x$statements_with_temp_tables 视图

30.4.3.41 user_summary 和 x$user_summary 视图

30.4.3.42 user_summary_by_file_io 和 x$user_summary_by_file_io 视图

30.4.3.43 user_summary_by_file_io_type 和 x$user_summary_by_file_io_type 视图

30.4.3.44 user_summary_by_stages 和 x$user_summary_by_stages 视图

30.4.3.45 user_summary_by_statement_latency 和 x$user_summary_by_statement_latency 视图

30.4.3.46 user_summary_by_statement_type 和 x$user_summary_by_statement_type 视图

30.4.3.47 version 视图

30.4.3.48 wait_classes_global_by_avg_latency 和 x$wait_classes_global_by_avg_latency 视图

30.4.3.49 wait_classes_global_by_latency 和 x$wait_classes_global_by_latency 视图

30.4.3.50 waits_by_host_by_latency 和 x$waits_by_host_by_latency 视图

30.4.3.51 waits_by_user_by_latency 和 x$waits_by_user_by_latency 视图

30.4.3.52 waits_global_by_latency 和 x$waits_global_by_latency 视图

以下部分描述了`sys`模式视图。

`sys`模式包含许多视图，以各种方式总结性能模式表。其中大多数视图都是成对出现的，其中一对的名称与另一对成员的名称相同，只是多了一个`x$`前缀。例如，`host_summary_by_file_io`视图按主机分组总结文件 I/O，并显示从皮秒转换为更易读值（带单位）的延迟；

```sql
mysql> SELECT * FROM sys.host_summary_by_file_io;
+------------+-------+------------+
| host       | ios   | io_latency |
+------------+-------+------------+
| localhost  | 67570 | 5.38 s     |
| background |  3468 | 4.18 s     |
+------------+-------+------------+
```

`x$host_summary_by_file_io`视图总结了相同的数据，但显示了未格式化的皮秒延迟：

```sql
mysql> SELECT * FROM sys.x$host_summary_by_file_io;
+------------+-------+---------------+
| host       | ios   | io_latency    |
+------------+-------+---------------+
| localhost  | 67574 | 5380678125144 |
| background |  3474 | 4758696829416 |
+------------+-------+---------------+
```

没有`x$`前缀的视图旨在提供更用户友好且更易读的输出。带有`x$`前缀的视图显示相同的原始值，更适用于其他对数据进行自己处理的工具。

没有`x$`前缀的视图与相应的带有`x$`前缀的视图有以下不同：

+   字节计数使用`format_bytes()` Function")进行大小单位格式化。

+   时间值使用`format_time()` Function")进行时间单位格式化。

+   SQL 语句通过`format_statement()` Function")进行最大显示宽度截断。

+   路径名使用`format_path()` Function")进行缩短。
