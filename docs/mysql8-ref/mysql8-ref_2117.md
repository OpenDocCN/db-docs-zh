# 29.13 性能模式选项和变量参考

> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-option-variable-reference.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-option-variable-reference.html)

**表 29.18 性能模式变量参考**

| 名称 | 命令行 | 选项文件 | 系统变量 | 状态变量 | 变量范围 | 动态 |
| --- | --- | --- | --- | --- | --- | --- |
| 性能模式 | 是 | 是 | 是 |  | 全局 | 否 |
| 性能模式账户丢失 |  |  |  | 是 | 全局 | 否 |
| 性能模式账户大小 | 是 | 是 | 是 |  | 全局 | 否 |
| 性能模式条件类丢失 |  |  |  | 是 | 全局 | 否 |
| 性能模式条件实例丢失 |  |  |  | 是 | 全局 | 否 |
| performance-schema-consumer-events-stages-current | 是 | 是 |  |  |  |  |
| performance-schema-consumer-events-stages-history | 是 | 是 |  |  |  |  |
| performance-schema-consumer-events-stages-history-long | 是 | 是 |  |  |  |  |
| performance-schema-consumer-events-statements-cpu | 是 | 是 |  |  |  |  |
| performance-schema-consumer-events-statements-current | 是 | 是 |  |  |  |  |
| performance-schema-consumer-events-statements-history | 是 | 是 |  |  |  |  |
| performance-schema-consumer-events-statements-history-long | 是 | 是 |  |  |  |  |
| performance-schema-consumer-events-transactions-current | 是 | 是 |  |  |  |  |
| performance-schema-consumer-events-transactions-history | 是 | 是 |  |  |  |  |
| performance-schema-consumer-events-transactions-history-long | 是 | 是 |  |  |  |  |
| performance-schema-consumer-events-waits-current | 是 | 是 |  |  |  |  |
| performance-schema-consumer-events-waits-history | 是 | 是 |  |  |  |  |
| performance-schema-consumer-events-waits-history-long | 是 | 是 |  |  |  |  |
| performance-schema-consumer-global-instrumentation | 是 | 是 |  |  |  |  |
| performance-schema-consumer-statements-digest | 是 | 是 |  |  |  |  |
| performance-schema-consumer-thread-instrumentation | 是 | 是 |  |  |  |  |
| Performance_schema_digest_lost |  |  |  | 是 | 全局 | 否 |
| performance_schema_digests_size | 是 | 是 | 是 |  | 全局 | 否 |
| performance_schema_events_stages_history_long_size | 是 | 是 | 是 |  | 全局 | 否 |
| performance_schema_events_stages_history_size | 是 | 是 | 是 |  | 全局 | 否 |
| performance_schema_events_statements_history_long_size | 是 | 是 | 是 |  | 全局 | 否 |
| performance_schema_events_statements_history_size | 是 | 是 | 是 |  | 全局 | 否 |
| performance_schema_events_transactions_history_long_size | 是 | 是 | 是 |  | 全局 | 否 |
| performance_schema_events_transactions_history_size | 是 | 是 | 是 |  | 全局 | 否 |
| performance_schema_events_waits_history_long_size | 是 | 是 | 是 |  | 全局 | 否 |
| performance_schema_events_waits_history_size | 是 | 是 | 是 |  | 全局 | 否 |
| 性能模式文件类丢失 |  |  |  | 是 | 全局 | 否 |
| 性能模式文件句柄丢失 |  |  |  | 是 | 全局 | 否 |
| 性能模式文件实例丢失 |  |  |  | 是 | 全局 | 否 |
| 性能模式主机丢失 |  |  |  | 是 | 全局 | 否 |
| 性能模式主机大小 | 是 | 是 | 是 |  | 全局 | 否 |
| 性能模式仪器 | 是 | 是 |  |  |  |  |
| 性能模式锁丢失 |  |  |  | 是 | 全局 | 否 |
| 性能模式最大条件类 | 是 | 是 | 是 |  | 全局 | 否 |
| 性能模式最大条件实例数 | 是 | 是 | 是 |  | 全局 | 否 |
| 性能模式最大摘要长度 | 是 | 是 | 是 |  | 全局 | 否 |
| 性能模式最大文件类 | 是 | 是 | 是 |  | 全局 | 否 |
| 性能模式最大文件句柄数 | 是 | 是 | 是 |  | 全局 | 否 |
| 性能模式最大文件实例数 | 是 | 是 | 是 |  | 全局 | 否 |
| 性能模式最大内存类 | 是 | 是 | 是 |  | 全局 | 否 |
| 性能模式最大元数据锁数 | 是 | 是 | 是 |  | 全局 | 否 |
| 性能模式最大互斥体类 | 是 | 是 | 是 |  | 全局 | 否 |
| 性能模式最大互斥体实例数 | 是 | 是 | 是 |  | 全局 | 否 |
| 性能模式最大准备语句实例数 | 是 | 是 | 是 |  | 全局 | 否 |
| performance_schema_max_program_instances | 是 | 是 | 是 |  | 全局 | 否 |
| performance_schema_max_rwlock_classes | 是 | 是 | 是 |  | 全局 | 否 |
| performance_schema_max_rwlock_instances | 是 | 是 | 是 |  | 全局 | 否 |
| performance_schema_max_socket_classes | 是 | 是 | 是 |  | 全局 | 否 |
| performance_schema_max_socket_instances | 是 | 是 | 是 |  | 全局 | 否 |
| performance_schema_max_stage_classes | 是 | 是 | 是 |  | 全局 | 否 |
| performance_schema_max_statement_classes | 是 | 是 | 是 |  | 全局 | 否 |
| performance_schema_max_statement_stack | 是 | 是 | 是 |  | 全局 | 否 |
| performance_schema_max_table_handles | 是 | 是 | 是 |  | 全局 | 否 |
| performance_schema_max_table_instances | 是 | 是 | 是 |  | 全局 | 否 |
| performance_schema_max_thread_classes | 是 | 是 | 是 |  | 全局 | 否 |
| performance_schema_max_thread_instances | 是 | 是 | 是 |  | 全局 | 否 |
| 性能模式内存类丢失 |  |  |  | 是 | 全局 | 否 |
| 性能模式元数据锁丢失 |  |  |  | 是 | 全局 | 否 |
| 性能模式互斥类丢失 |  |  |  | 是 | 全局 | 否 |
| 性能模式互斥实例丢失 |  |  |  | 是 | 全局 | 否 |
| 性能模式嵌套语句丢失 |  |  |  | 是 | 全局 | 否 |
| 性能模式准备语句丢失 |  |  |  | 是 | 全局 | 否 |
| 性能模式程序丢失 |  |  |  | 是 | 全局 | 否 |
| 性能模式读写锁类丢失 |  |  |  | 是 | 全局 | 否 |
| 性能模式读写锁实例丢失 |  |  |  | 是 | 全局 | 否 |
| 性能模式会话连接属性丢失 |  |  |  | 是 | 全局 | 否 |
| performance_schema_session_connect_attrs_size | 是 | 是 | 是 |  | 全局 | 否 |
| performance_schema_setup_actors_size | 是 | 是 | 是 |  | 全局 | 否 |
| performance_schema_setup_objects_size | 是 | 是 | 是 |  | 全局 | 否 |
| 性能模式套接字类丢失 |  |  |  | 是 | 全局 | 否 |
| 性能模式套接字实例丢失 |  |  |  | 是 | 全局 | 否 |
| 性能模式阶段类丢失 |  |  |  | 是 | 全局 | 否 |
| 性能模式语句类丢失 |  |  |  | 是 | 全局 | 否 |
| 性能模式表句柄丢失 |  |  |  | 是 | 全局 | 否 |
| 性能模式表实例丢失 |  |  |  | 是 | 全局 | 否 |
| 性能模式线程类丢失 |  |  |  | 是 | 全局 | 否 |
| 性能模式线程实例丢失 |  |  |  | 是 | 全局 | 否 |
| 性能模式用户丢失 |  |  |  | 是 | 全局 | 否 |
| performance_schema_users_size | 是 | 是 | 是 |  | 全局 | 否 |
| 名称 | 命令行 | 选项文件 | 系统变量 | 状态变量 | 变量范围 | 动态 |
