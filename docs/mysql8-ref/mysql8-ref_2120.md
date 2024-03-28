# 29.16 性能模式状态变量

> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-status-variables.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-status-variables.html)

性能模式实现了几个状态变量，提供有关由于内存限制而无法加载或创建的仪器化信息：

```sql
mysql> SHOW STATUS LIKE 'perf%';
+-------------------------------------------+-------+
| Variable_name                             | Value |
+-------------------------------------------+-------+
| Performance_schema_accounts_lost          | 0     |
| Performance_schema_cond_classes_lost      | 0     |
| Performance_schema_cond_instances_lost    | 0     |
| Performance_schema_file_classes_lost      | 0     |
| Performance_schema_file_handles_lost      | 0     |
| Performance_schema_file_instances_lost    | 0     |
| Performance_schema_hosts_lost             | 0     |
| Performance_schema_locker_lost            | 0     |
| Performance_schema_mutex_classes_lost     | 0     |
| Performance_schema_mutex_instances_lost   | 0     |
| Performance_schema_rwlock_classes_lost    | 0     |
| Performance_schema_rwlock_instances_lost  | 0     |
| Performance_schema_socket_classes_lost    | 0     |
| Performance_schema_socket_instances_lost  | 0     |
| Performance_schema_stage_classes_lost     | 0     |
| Performance_schema_statement_classes_lost | 0     |
| Performance_schema_table_handles_lost     | 0     |
| Performance_schema_table_instances_lost   | 0     |
| Performance_schema_thread_classes_lost    | 0     |
| Performance_schema_thread_instances_lost  | 0     |
| Performance_schema_users_lost             | 0     |
+-------------------------------------------+-------+
```

有关使用这些变量检查性能模式状态的信息，请参见第 29.7 节“性能模式状态监控”。

性能模式状态变量具有以下含义：

+   `Performance_schema_accounts_lost`

    由于表已满，无法将行添加到`accounts`表中的次数。

+   `Performance_schema_cond_classes_lost`

    无法加载多少个条件仪器。

+   `Performance_schema_cond_instances_lost`

    无法创建多少个条件仪器实例。

+   `Performance_schema_digest_lost`

    无法在`events_statements_summary_by_digest`表中对摘要实例进行仪器化的次数。如果`performance_schema_digests_size`的值太小，则此值可能为非零。

+   `Performance_schema_file_classes_lost`

    无法加载多少个文件仪器。

+   `Performance_schema_file_handles_lost`

    无法打开多少个文件仪器实例。

+   `Performance_schema_file_instances_lost`

    无法创建多少个文件仪器实例。

+   `Performance_schema_hosts_lost`

    由于表已满，无法将行添加到`hosts`表中的次数。

+   `Performance_schema_index_stat_lost`

    丢失了多少个统计信息的索引。如果`performance_schema_max_index_stat`的值太小，则此值可能为非零。

+   `Performance_schema_locker_lost`

    由于以下条件，有多少事件“丢失”或未记录：

    +   事件是递归的（例如，等待 A 导致等待 B，导致等待 C）。

    +   嵌套事件堆栈的深度超过了实现所施加的限制。

    Performance Schema 记录的事件不是递归的，因此此变量应始终为 0。

+   `Performance_schema_memory_classes_lost`

    无法加载内存工具的次数。

+   `Performance_schema_metadata_lock_lost`

    在`metadata_locks`表中无法对元数据锁进行仪器化的数量。如果`performance_schema_max_metadata_locks`的值太小，则可能不为零。

+   `Performance_schema_mutex_classes_lost`

    有多少互斥锁工具无法加载。

+   `Performance_schema_mutex_instances_lost`

    有多少互斥锁工具实例无法创建。

+   `Performance_schema_nested_statement_lost`

    丢失统计信息的存储程序语句的数量。如果`performance_schema_max_statement_stack`的值太小，则可能不为零。

+   `Performance_schema_prepared_statements_lost`

    在`prepared_statements_instances`表中无法对准备语句进行仪器化的数量。如果`performance_schema_max_prepared_statements_instances`的值太小，则可能不为零��

+   `Performance_schema_program_lost`

    丢失统计信息的存储程序的数量。如果`performance_schema_max_program_instances`的值太小，则可能不为零。

+   `Performance_schema_rwlock_classes_lost`

    无法加载多少个 rwlock 仪器。

+   `Performance_schema_rwlock_instances_lost`

    无法创建多少个 rwlock 仪器实例。

+   `Performance_schema_session_connect_attrs_longest_seen`

    除了性能模式针对`performance_schema_session_connect_attrs_size`系统变量的值执行的连接属性大小限制检查外，服务器还执行了一个初步检查，对接受的连接属性数据的总大小施加了 64KB 的限制。如果客户端尝试发送超过 64KB 的属性数据，服务器将拒绝连接。否则，服务器将认为属性缓冲区有效，并跟踪最长缓冲区的大小，存储在`Performance_schema_session_connect_attrs_longest_seen`状态变量中。如果此值大于`performance_schema_session_connect_attrs_size`，DBA 可能希望增加后者的值，或者调查哪些客户端正在发送大量属性数据。

    关于连接属性的更多信息，请参见第 29.12.9 节，“性能模式连接属性表”。

+   `Performance_schema_session_connect_attrs_lost`

    发生连接属性截断的连接数。对于给定的连接，如果客户端发送的连接属性键值对的总大小大于`performance_schema_session_connect_attrs_size`系统变量允许的保留存储空间，性能模式将截断属性数据并增加`Performance_schema_session_connect_attrs_lost`。如果此值不为零，则可能希望将`performance_schema_session_connect_attrs_size`设置为较大的值。

    关于连接属性的更多信息，请参见第 29.12.9 节，“性能模式连接属性表”。

+   `Performance_schema_socket_classes_lost`

    有多少套接字工具无法加载。

+   `Performance_schema_socket_instances_lost`

    有多少套接字工具实例无法创建。

+   `Performance_schema_stage_classes_lost`

    有多少阶段工具无法加载。

+   `Performance_schema_statement_classes_lost`

    有多少语句工具无法加载。

+   `Performance_schema_table_handles_lost`

    有多少表工具实例无法打开。如果`performance_schema_max_table_handles`的值太小，则可能不为零。

+   `Performance_schema_table_instances_lost`

    有多少表工具实例无法创建。

+   `Performance_schema_table_lock_stat_lost`

    丢失锁统计信息的表数。如果`performance_schema_max_table_lock_stat`的值太小，则可能不为零。

+   `Performance_schema_thread_classes_lost`

    有多少线程工具无法加载。

+   `Performance_schema_thread_instances_lost`

    无法在`threads`表中对线程实例进行仪器化的次数。如果`performance_schema_max_thread_instances`的值太小，则可能不为零。

+   `Performance_schema_users_lost`

    由于表已满，无法将行添加到`users`表中的次数。
