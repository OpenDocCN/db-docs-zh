# 29.17 性能模式内存分配模型

> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-memory-model.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-memory-model.html)

性能模式使用此内存分配模型：

+   可能在服务器启动时分配内存。

+   可能在服务器运行期间分配额外的内存。

+   永远不要在服务器运行期间释放内存（尽管它可能被回收）。

+   在关闭时释放所有使用的内存。

结果是放宽内存约束，以便性能模式可以在较少的配置下使用，并减少内存占用量，使消耗随服务器负载而扩展。内存使用取决于实际观察到的负载，而不是估计的负载或明确配置的负载。

几个性能模式大小参数是自动缩放的，除非您想要在内存分配上建立明确的限制，否则无需显式配置：

```sql
performance_schema_accounts_size
performance_schema_hosts_size
performance_schema_max_cond_instances
performance_schema_max_file_instances
performance_schema_max_index_stat
performance_schema_max_metadata_locks
performance_schema_max_mutex_instances
performance_schema_max_prepared_statements_instances
performance_schema_max_program_instances
performance_schema_max_rwlock_instances
performance_schema_max_socket_instances
performance_schema_max_table_handles
performance_schema_max_table_instances
performance_schema_max_table_lock_stat
performance_schema_max_thread_instances
performance_schema_users_size
```

对于自动缩放的参数，配置如下：

+   当值设置为-1（默认值）时，参数会自动缩放：

    +   相应的内部缓冲区最初为空，不会分配任何内存。

    +   当性能模式收集数据时，内存会在相应的缓冲区中分配。缓冲区大小是无限的，并且可能随着负载而增长。

+   当值设置为 0 时：

    +   相应的内部缓冲区最初为空，不会分配任何内存。

+   当值设置为*`N`* > 0 时：

    +   相应的内部缓冲区最初为空，不会分配任何内存。

    +   当性能模式收集数据时，内存会在相应的缓冲区中分配，直到缓冲区大小达到*`N`*。

    +   一旦缓冲区大小达到*`N`*，就不会再分配更多内存。性能模式为此缓冲区收集的数据将丢失，并且任何相应的“丢失实例”计数器都会递增。

要查看性能模式使用了多少内存，请检查为此目的设计的工具。性能模式在内部分配内存，并将每个缓冲区与专用工具相关联，以便可以将内存消耗跟踪到各个缓冲区。以`memory/performance_schema/`为前缀命名的工具公开了为这些内部缓冲区分配了多少内存。这些缓冲区是全局的，因此这些工具仅在`memory_summary_global_by_event_name`表中显示，并不在其他`memory_summary_by_*`xxx`*_by_event_name`表中显示。

此查询显示与内存工具相关的信息：

```sql
SELECT * FROM performance_schema.memory_summary_global_by_event_name
WHERE EVENT_NAME LIKE 'memory/performance_schema/%';
```
