# 29.12.1 性能模式表参考

> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-table-reference.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-table-reference.html)

以下表格总结了所有可用的性能模式表。更详细的信息，请参阅各个表的描述。

**表 29.1 性能模式表**

| 表名 | 描述 | 引入版本 |
| --- | --- | --- |
| `accounts` | 每个客户端账户的连接统计 |  |
| `binary_log_transaction_compression_stats` | 二进制日志事务压缩 | 8.0.20 |
| `clone_progress` | 克隆操作进度 | 8.0.17 |
| `clone_status` | 克隆操作状态 | 8.0.17 |
| `component_scheduler_tasks` | 计划任务的状态 | 8.0.34 |
| `cond_instances` | 同步对象实例 |  |
| `data_lock_waits` | 数据锁等待关系 |  |
| `data_locks` | 持有和请求的数据锁 |  |
| `error_log` | 服务器错误日志最近条目 | 8.0.22 |
| `events_errors_summary_by_account_by_error` | 每个账户和错误代码的错误 |  |
| `events_errors_summary_by_host_by_error` | 每个主机和错误代码的错误 |  |
| `events_errors_summary_by_thread_by_error` | 每个线程和错误代码的错误 |  |
| `events_errors_summary_by_user_by_error` | 每个用户和错误代码的错误 |  |
| `events_errors_summary_global_by_error` | 每个错误代码的错误 |  |
| `events_stages_current` | 当前阶段事件 |  |
| `events_stages_history` | 每个线程的最近阶段事件 |  |
| `events_stages_history_long` | 最近的整体阶段事件 |  |
| `events_stages_summary_by_account_by_event_name` | 每个帐户和事件名称的阶段事件 |  |
| `events_stages_summary_by_host_by_event_name` | 每个主机名称和事件名称的阶段事件 |  |
| `events_stages_summary_by_thread_by_event_name` | 每个线程和事件名称的阶段等待 |  |
| `events_stages_summary_by_user_by_event_name` | 每个用户名称和事件名称的阶段事件 |  |
| `events_stages_summary_global_by_event_name` | 每个事件名称的阶段等待 |  |
| `events_statements_current` | 当前语句事件 |  |
| `events_statements_histogram_by_digest` | 每个模式和摘要值的语句直方图 |  |
| `events_statements_histogram_global` | 全局汇总的语句直方图 |  |
| `events_statements_history` | 每个线程的最近语句事件 |  |
| `events_statements_history_long` | 最近的整体语句事件 |  |
| `events_statements_summary_by_account_by_event_name` | 每个帐户和事件名称的语句事件 |  |
| `events_statements_summary_by_digest` | 每个模式和摘要值的语句事件 |  |
| `events_statements_summary_by_host_by_event_name` | 每个主机名称和事件名称的语句事件 |  |
| `events_statements_summary_by_program` | 每个存储程序的语句事件 |  |
| `events_statements_summary_by_thread_by_event_name` | 每个线程和事件名称的语句事件 |  |
| `events_statements_summary_by_user_by_event_name` | 每个用户名称和事件名称的语句事件 |  |
| `events_statements_summary_global_by_event_name` | 每个事件名称的语句事件 |  |
| `events_transactions_current` | 当前事务事件 |  |
| `events_transactions_history` | 每个线程的最近事务事件 |  |
| `events_transactions_history_long` | 最近的所有事务事件 |  |
| `events_transactions_summary_by_account_by_event_name` | 每个帐户和事件名称的事务事件 |  |
| `events_transactions_summary_by_host_by_event_name` | 每个主机名称和事件名称的事务事件 |  |
| `events_transactions_summary_by_thread_by_event_name` | 每个线程和事件名称的事务事件 |  |
| `events_transactions_summary_by_user_by_event_name` | 每个用户名称和事件名称的事务事件 |  |
| `events_transactions_summary_global_by_event_name` | 每个事件名称的事务事件 |  |
| `events_waits_current` | 当前等待事件 |  |
| `events_waits_history` | 每个线程的最近等待事件 |  |
| `events_waits_history_long` | 最近的所有等待事件 |  |
| `events_waits_summary_by_account_by_event_name` | 每个帐户和事件名称的等待事件 |  |
| `events_waits_summary_by_host_by_event_name` | 每个主机名称和事件名称的等待事件 |  |
| `events_waits_summary_by_instance` | 每个实例的等待事件 |  |
| `events_waits_summary_by_thread_by_event_name` | 按线程和事件名称的等待事件 |  |
| `events_waits_summary_by_user_by_event_name` | 按用户名称和事件名称的等待事件 |  |
| `events_waits_summary_global_by_event_name` | 按事件名称的等待事件 |  |
| `file_instances` | 文件实例 |  |
| `file_summary_by_event_name` | 按事件名称的文件事件 |  |
| `file_summary_by_instance` | 按文件实例的文件事件 |  |
| `firewall_group_allowlist` | 组配置允许列表的防火墙内存数据 | 8.0.23 |
| `firewall_groups` | 组配置的防火墙内存数据 | 8.0.23 |
| `firewall_membership` | 组配置成员的防火墙内存数据 | 8.0.23 |
| `global_status` | 全局状态变量 |  |
| `global_variables` | 全局系统变量 |  |
| `host_cache` | 内部主机缓存的信息 |  |
| `hosts` | 每个客户端主机名称的连接统计 |  |
| `keyring_component_status` | 已安装密钥环组件的状态信息 | 8.0.24 |
| `keyring_keys` | 密钥环键的元数据 | 8.0.16 |
| `log_status` | 用于备份目的的服务器日志信息 |  |
| `memory_summary_by_account_by_event_name` | 按账户和事件名称的内存操作 |  |
| `memory_summary_by_host_by_event_name` | 按主机和事件名称的内存操作 |  |
| `memory_summary_by_thread_by_event_name` | 按线程和事件名称内存操作 |  |
| `memory_summary_by_user_by_event_name` | 按用户和事件名称内存操作 |  |
| `memory_summary_global_by_event_name` | 按事件名称全局内存操作 |  |
| `metadata_locks` | 元数据锁和锁请求 |  |
| `mutex_instances` | 互斥同步对象实例 |  |
| `ndb_sync_excluded_objects` | 无法同步的 NDB 对象 | 8.0.21 |
| `ndb_sync_pending_objects` | 等待同步的 NDB 对象 | 8.0.21 |
| `objects_summary_global_by_type` | 对象摘要 |  |
| `performance_timers` | 可用的事件计时器 |  |
| `persisted_variables` | mysqld-auto.cnf 文件的内容 |  |
| `prepared_statements_instances` | 准备语句实例和统计信息 |  |
| `processlist` | 进程列表信息 | 8.0.22 |
| `replication_applier_configuration` | 复制应用程序在副本上的配置参数 |  |
| `replication_applier_filters` | 当前副本上的通道特定复制过滤器 |  |
| `replication_applier_global_filters` | 当前副本上的全局复制过滤器 |  |
| `replication_applier_status` | 复制应用程序在副本上的当前状态 |  |
| `replication_applier_status_by_coordinator` | SQL 或协调器线程应用程序状态 |  |
| `replication_applier_status_by_worker` | 工作线程应用程序状态 |  |
| `replication_asynchronous_connection_failover` | 异步连接故障转移机制的源列表 | 8.0.22 |
| `replication_asynchronous_connection_failover_managed` | 异步连接故障转移机制的受控源列表 | 8.0.23 |
| `replication_connection_configuration` | 连接到源的配置参数 |  |
| `replication_connection_status` | 与源的当前连接状态 |  |
| `replication_group_communication_information` | 复制组配置选项 | 8.0.27 |
| `replication_group_member_stats` | 复制组成员统计信息 |  |
| `replication_group_members` | 复制组成员网络和状态 |  |
| `rwlock_instances` | 锁同步对象实例 |  |
| `session_account_connect_attrs` | 当前会话的连接属性 |  |
| `session_connect_attrs` | 所有会话的连接属性 |  |
| `session_status` | 当前会话的状态变量 |  |
| `session_variables` | 当前会话的系统变量 |  |
| `setup_actors` | 如何为新前台线程初始化监视 |  |
| `setup_consumers` | 可存储事件信息的消费者 |  |
| `setup_instruments` | 可收集事件的已标记对象类别 |  |
| `setup_objects` | 应监视的对象 |  |
| `setup_threads` | 已标记线程名称和属性 |  |
| `socket_instances` | 活动连接实例 |  |
| `socket_summary_by_event_name` | 按事件名称的套接字等待和 I/O |  |
| `socket_summary_by_instance` | 按实例的套接字等待和 I/O |  |
| `status_by_account` | 每个账户的会话状态变量 |  |
| `status_by_host` | 每个主机名的会话状态变量 |  |
| `status_by_thread` | 每个会话的会话状态变量 |  |
| `status_by_user` | 每个用户名的会话状态变量 |  |
| `table_handles` | 表锁和锁请求 |  |
| `table_io_waits_summary_by_index_usage` | 按索引使用情况的表 I/O 等待 |  |
| `table_io_waits_summary_by_table` | 按表的表 I/O 等待 |  |
| `table_lock_waits_summary_by_table` | 按表的表锁等待 |  |
| `threads` | 服务器线程信息 |  |
| `tls_channel_status` | 每个连接接口的 TLS 状态 | 8.0.21 |
| `tp_thread_group_state` | 线程池线程组状态 | 8.0.14 |
| `tp_thread_group_stats` | 线程池线程组统计信息 | 8.0.14 |
| `tp_thread_state` | 线程池线程信息 | 8.0.14 |
| `user_defined_functions` | 注册的可加载函数 |  |
| `user_variables_by_thread` | 每个线程的用户定义变量 |  |
| `users` | 每个客户端用户名的连接统计 |  |
| `variables_by_thread` | 每个会话的会话系统变量 |  |
| `variables_info` | 最近设置的系统变量 |  |
| 表名 | 描述 | 引入版本 |
