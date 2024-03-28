# 30.4.1 sys 模式对象索引

> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-schema-object-index.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-schema-object-index.html)

下表列出了`sys`模式对象，并提供了每个对象的简短描述。

**表 30.1 sys 模式表和触发器**

| 表或触发器名称 | 描述 |
| --- | --- |
| `sys_config` | sys 模式配置选项表 |
| `sys_config_insert_set_user` | sys_config 插入触发器 |
| `sys_config_update_set_user` | sys_config 更新触发器 |

**表 30.2 sys 模式视图**

| 视图名称 | 描述 | 已弃用 |
| --- | --- | --- |
| `host_summary`, `x$host_summary` | 按主机分组的语句活动、文件 I/O 和连接 |  |
| `host_summary_by_file_io`, `x$host_summary_by_file_io` | 按主机分组的文件 I/O |  |
| `host_summary_by_file_io_type`, `x$host_summary_by_file_io_type` | 按主机和事件类型分组的文件 I/O |  |
| `host_summary_by_stages`, `x$host_summary_by_stages` | 按主机分组的语句阶段 |  |
| `host_summary_by_statement_latency`, `x$host_summary_by_statement_latency` | 按主机分组的语句统计信息 |  |
| `host_summary_by_statement_type`, `x$host_summary_by_statement_type` | 按主机和语句分组的执行语句 |  |
| `innodb_buffer_stats_by_schema`, `x$innodb_buffer_stats_by_schema` | 按模式分组的 InnoDB 缓冲区信息 |  |
| `innodb_buffer_stats_by_table`, `x$innodb_buffer_stats_by_table` | 按模式和表分组的 InnoDB 缓冲区信息 |  |
| `innodb_lock_waits`, `x$innodb_lock_waits` | InnoDB 锁信息 |  |
| `io_by_thread_by_latency`, `x$io_by_thread_by_latency` | 按线程分组的 I/O 消费者 |  |
| `io_global_by_file_by_bytes`, `x$io_global_by_file_by_bytes` | 按文件和字节分组的全局 I/O 消费者 |  |
| `io_global_by_file_by_latency`, `x$io_global_by_file_by_latency` | 按文��和延迟分组的全局 I/O 消费者 |  |
| `io_global_by_wait_by_bytes`, `x$io_global_by_wait_by_bytes` | 按字节分组的全局 I/O 消费者 |  |
| `io_global_by_wait_by_latency`, `x$io_global_by_wait_by_latency` | 按延迟分组的全局 I/O 消费者 |  |
| `latest_file_io`, `x$latest_file_io` | 最近的 I/O，按文件和线程分组 |  |
| `memory_by_host_by_current_bytes`, `x$memory_by_host_by_current_bytes` | 按主机分组的内存使用 |  |
| `memory_by_thread_by_current_bytes`, `x$memory_by_thread_by_current_bytes` | 按线程分组的内存使用 |  |
| `memory_by_user_by_current_bytes`, `x$memory_by_user_by_current_bytes` | 按用户分组的内存使用 |  |
| `memory_global_by_current_bytes`, `x$memory_global_by_current_bytes` | 按分配类型分组的内存使用 |  |
| `memory_global_total`, `x$memory_global_total` | 总内存使用 |  |
| `metrics` | 服务器指标 |  |
| `processlist`, `x$processlist` | 进程列表信息 |  |
| `ps_check_lost_instrumentation` | 已丢失仪器的变量 |  |
| `schema_auto_increment_columns` | AUTO_INCREMENT 列信息 |  |
| `schema_index_statistics`, `x$schema_index_statistics` | 索引统计信息 |  |
| `schema_object_overview` | 每个模式中对象的类型 |  |
| `schema_redundant_indexes` | 重复或冗余索引 |  |
| `schema_table_lock_waits`, `x$schema_table_lock_waits` | 等待元数据锁的会话 |  |
| `schema_table_statistics`, `x$schema_table_statistics` | 表统计信息 |  |
| `schema_table_statistics_with_buffer`, `x$schema_table_statistics_with_buffer` | 表统计信息，包括 InnoDB 缓冲池统计信息 |  |
| `schema_tables_with_full_table_scans`, `x$schema_tables_with_full_table_scans` | 使用全表扫描访问的表 |  |
| `schema_unused_indexes` | 未被使用的索引 |  |
| `session`, `x$session` | 用户会话的进程列表信息 |  |
| `session_ssl_status` | 连接 SSL 信息 |  |
| `statement_analysis`, `x$statement_analysis` | 语句聚合统计信息 |  |
| `statements_with_errors_or_warnings`, `x$statements_with_errors_or_warnings` | 产生错误或警告的语句 |  |
| `statements_with_full_table_scans`, `x$statements_with_full_table_scans` | 执行全表扫描的语句 |  |
| `statements_with_runtimes_in_95th_percentile`, `x$statements_with_runtimes_in_95th_percentile` | 平均运行时间最长的语句 |  |
| `statements_with_sorting`, `x$statements_with_sorting` | 执行排序操作的语句 |  |
| `statements_with_temp_tables`, `x$statements_with_temp_tables` | 使用临时表的语句 |  |
| `user_summary`, `x$user_summary` | 用户语��和连接活动 |  |
| `user_summary_by_file_io`, `x$user_summary_by_file_io` | 按用户分组的文件 I/O |  |
| `user_summary_by_file_io_type`, `x$user_summary_by_file_io_type` | 按用户和事件分组的文件 I/O |  |
| `user_summary_by_stages`, `x$user_summary_by_stages` | 按用户分组的阶段事件 |  |
| `user_summary_by_statement_latency`, `x$user_summary_by_statement_latency` | 按用户分组的语句统计信息 |  |
| `user_summary_by_statement_type`, `x$user_summary_by_statement_type` | 按用户和语句分组的执行语句 |  |
| `version` | 当前 sys 模式和 MySQL 服务器版本 | 8.0.18 |
| `wait_classes_global_by_avg_latency`, `x$wait_classes_global_by_avg_latency` | 按事件类别分组的等待类别平均延迟 |  |
| `wait_classes_global_by_latency`, `x$wait_classes_global_by_latency` | 按事件类别分组的等待类别总延迟 |  |
| `waits_by_host_by_latency`, `x$waits_by_host_by_latency` | 按主机和事件分组的等待事件 |  |
| `waits_by_user_by_latency`, `x$waits_by_user_by_latency` | 按用户和事件分组的等待事件 |  |
| `waits_global_by_latency`, `x$waits_global_by_latency` | 按事件分组的等待事件 |  |
| `x$ps_digest_95th_percentile_by_avg_us` | 95th-percentile 视图的辅助视图 |  |
| `x$ps_digest_avg_latency_distribution` | 95th-percentile 视图的辅助视图 |  |
| `x$ps_schema_table_statistics_io` | 表统计视图的辅助视图 |  |
| `x$schema_flattened_keys` | schema_redundant_indexes 的辅助视图 |  |
| 视图名称 | 描述 | 已弃用 |

**表 30.3 sys 模式存储过程**

| 过程名称 | 描述 |
| --- | --- |
| `create_synonym_db()` 过程") | 为模式创建同义词 |
| `diagnostics()` 过程") | 收集系统诊断信息 |
| `execute_prepared_stmt()` 过程") | 执行准备好的语句 |
| `ps_setup_disable_background_threads()` 过程") | 禁用后台线程仪器 |
| `ps_setup_disable_consumer()` 过程") | 禁用消费者 |
| `ps_setup_disable_instrument()` 过程") | 禁用仪器 |
| `ps_setup_disable_thread()` 过程") | 禁用线程的仪器 |
| `ps_setup_enable_background_threads()` 过程") | 启用后台线程仪器 |
| `ps_setup_enable_consumer()` 过程") | 启用消费者 |
| `ps_setup_enable_instrument()` 过程") | 启用仪器 |
| `ps_setup_enable_thread()` 过程") | 为线程启用仪器 |
| `ps_setup_reload_saved()` 过程") | 重新加载保存的性能模式配置 |
| `ps_setup_reset_to_default()` 过程") | 重置保存的性能模式配置 |
| `ps_setup_save()` 过程") | 保存性能模式配置 |
| `ps_setup_show_disabled()` 过程") | 显示已禁用的性能模式配置 |
| `ps_setup_show_disabled_consumers()` 过程") | 显示已禁用的性能模式消费者 |
| `ps_setup_show_disabled_instruments()` 过程") | 显示已禁用的性能模式工具 |
| `ps_setup_show_enabled()` 过程") | 显示已启用的性能模式配置 |
| `ps_setup_show_enabled_consumers()` 过程") | 显示已启用的性能模式消费者 |
| `ps_setup_show_enabled_instruments()` 过程") | 显示已启用的性能模式工具 |
| `ps_statement_avg_latency_histogram()` 过程") | 显示语句延迟直方图 |
| `ps_trace_statement_digest()` 过程") | 跟踪摘要的性能模式仪器 |
| `ps_trace_thread()` 过程") | 为线程转储性能模式数据 |
| `ps_truncate_all_tables()` 过程") | 截断性能模式摘要表 |
| `statement_performance_analyzer()` 过程") | 报告在服务器上运行的语句 |
| `table_exists()` 过程") | 检查表是否存在 |
| 过程名称 | 描述 |

**表 30.4 sys 模式存储函数**

| 函数名称 | 描述 | 已弃用 |
| --- | --- | --- |
| `extract_schema_from_file_name()` Function") | 从文件名中提取模式名部分 |  |
| `extract_table_from_file_name()` Function") | 从文件名中提取表名部分 |  |
| `format_bytes()` Function") | 将字节计数转换为带单位的值 | 8.0.16 |
| `format_path()` Function") | 用符号系统变量名替换路径名中的目录 |  |
| `format_statement()` Function") | 将长语句截断为固定长度 |  |
| `format_time()` Function") | 将皮秒时间转换为带单位的值 | 8.0.16 |
| `list_add()` Function") | 向列表添加项目 |  |
| `list_drop()` Function") | 从列表中移除项目 |  |
| `ps_is_account_enabled()` Function") | 是否启用帐户的性能模式仪表 |  |
| `ps_is_consumer_enabled()` Function") | 是否启用性能模式消费者 |  |
| `ps_is_instrument_default_enabled()` Function") | 是否默认启用性能模式仪表 |  |
| `ps_is_instrument_default_timed()` Function") | 是否默认启用性能模式计时仪表 |  |
| `ps_is_thread_instrumented()` Function") | 是否启用连接 ID 的性能模式仪表 |  |
| `ps_thread_account()` Function") | 与性能模式线程 ID 关联的帐户 |  |
| `ps_thread_id()` Function") | 与连接 ID 关联的性能模式线程 ID | 8.0.16 |
| `ps_thread_stack()` Function") | 连接 ID 的事件信息 |  |
| `ps_thread_trx_info()` Function") | 线程 ID 的事务信息 |  |
| `quote_identifier()` Function") | 将字符串引用为标识符 |  |
| `sys_get_config()` Function") | 系统模式配置选项值 |  |
| `version_major()` 函数") | MySQL 服务器主要版本号 |  |
| `version_minor()` 函数") | MySQL 服务器次要版本号 |  |
| `version_patch()` 函数") | MySQL 服务器补丁发布版本号 |  |
| 函数名称 | 描述 | 已弃用 |
