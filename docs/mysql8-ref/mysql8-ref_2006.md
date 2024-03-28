# 29.7 性能模式状态监控

> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-status-monitoring.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-status-monitoring.html)

与性能模式相关的几个状态变量有：

```sql
mysql> SHOW STATUS LIKE 'perf%';
+-----------------------------------------------+-------+
| Variable_name                                 | Value |
+-----------------------------------------------+-------+
| Performance_schema_accounts_lost              | 0     |
| Performance_schema_cond_classes_lost          | 0     |
| Performance_schema_cond_instances_lost        | 0     |
| Performance_schema_digest_lost                | 0     |
| Performance_schema_file_classes_lost          | 0     |
| Performance_schema_file_handles_lost          | 0     |
| Performance_schema_file_instances_lost        | 0     |
| Performance_schema_hosts_lost                 | 0     |
| Performance_schema_locker_lost                | 0     |
| Performance_schema_memory_classes_lost        | 0     |
| Performance_schema_metadata_lock_lost         | 0     |
| Performance_schema_mutex_classes_lost         | 0     |
| Performance_schema_mutex_instances_lost       | 0     |
| Performance_schema_nested_statement_lost      | 0     |
| Performance_schema_program_lost               | 0     |
| Performance_schema_rwlock_classes_lost        | 0     |
| Performance_schema_rwlock_instances_lost      | 0     |
| Performance_schema_session_connect_attrs_lost | 0     |
| Performance_schema_socket_classes_lost        | 0     |
| Performance_schema_socket_instances_lost      | 0     |
| Performance_schema_stage_classes_lost         | 0     |
| Performance_schema_statement_classes_lost     | 0     |
| Performance_schema_table_handles_lost         | 0     |
| Performance_schema_table_instances_lost       | 0     |
| Performance_schema_thread_classes_lost        | 0     |
| Performance_schema_thread_instances_lost      | 0     |
| Performance_schema_users_lost                 | 0     |
+-----------------------------------------------+-------+
```

性能模式状态变量提供了有关由于内存限制而无法加载或创建的仪器化信息。这些变量的名称有几种形式：

+   `Performance_schema_*`xxx`*_classes_lost` 表示无法加载类型为 *`xxx`* 的仪器数量。

+   `Performance_schema_*`xxx`*_instances_lost` 表示无法创建对象类型为 *`xxx`* 的实例数量。

+   `Performance_schema_*`xxx`*_handles_lost` 表示无法打开对象类型为 *`xxx`* 的实例数量。

+   `Performance_schema_locker_lost` 表示有多少事件“丢失”或未记录。

例如，如果在服务器源代码中为互斥仪器进行了仪器化，但服务器在运行时无法为仪器分配内存，则会增加 `Performance_schema_mutex_classes_lost`。互斥仍然作为同步对象运行（即，服务器继续正常运行），但不会收集其性能数据。如果可以分配仪器，则可以用于初始化仪器化的互斥实例。对于全局互斥体这样的单例互斥体，只有一个实例。其他互斥体每个连接或每个页面在各种缓存和数据缓冲区中有一个实例，因此实例数量随时间变化。增加最大连接数或某些缓冲区的最大大小会增加一次可能分配的实例的最大数量。如果服务器无法创建给定的仪器化互斥实例，则会增加 `Performance_schema_mutex_instances_lost`。

假设以下条件成立：

+   服务器使用 `--performance_schema_max_mutex_classes=200` 选项启动，因此有 200 个互斥仪器的空间。

+   已加载了 150 个互斥仪器。

+   名为 `plugin_a` 的插件包含 40 个互斥仪器。

+   名为 `plugin_b` 的插件包含 20 个互斥仪器。

服务器根据插件需要的数量和可用数量为插件分配互斥仪器，如下面的语句序列所示：

```sql
INSTALL PLUGIN plugin_a
```

服务器现在有 150+40 = 190 个互斥仪器。

```sql
UNINSTALL PLUGIN plugin_a;
```

服务器仍然有 190 个仪器。插件代码生成的所有历史数据仍然可用，但不会收集仪器的新事件。

```sql
INSTALL PLUGIN plugin_a;
```

服务器检测到已经定义了 40 个仪器，因此不会创建新的仪器，并且先前分配的内部内存缓冲区将被重用。服务器仍然有 190 个仪器。

```sql
INSTALL PLUGIN plugin_b;
```

服务器有 200-190 = 10 个仪器的空间（在本例中是互斥类），并且发现插件包含 20 个新仪器。加载了 10 个仪器，而 10 个被丢弃或“丢失”。`Performance_schema_mutex_classes_lost` 指示了丢失的仪器（互斥类）的数量：

```sql
mysql> SHOW STATUS LIKE "perf%mutex_classes_lost";
+---------------------------------------+-------+
| Variable_name                         | Value |
+---------------------------------------+-------+
| Performance_schema_mutex_classes_lost | 10    |
+---------------------------------------+-------+
1 row in set (0.10 sec)
```

仪器仍在工作并为 `plugin_b` 收集（部分）数据。

当服务器无法创建互斥仪器时，会出现以下结果：

+   仪器未插入 `setup_instruments` 表中。

+   `Performance_schema_mutex_classes_lost` 增加了 1。

+   `Performance_schema_mutex_instances_lost` 不会改变。（当互斥仪器未创建时，无法用于稍后创建仪器化的互斥实例。）

刚才描述的模式适用于所有类型的仪器，而不仅仅是互斥体。

`Performance_schema_mutex_classes_lost` 大于 0 的值可能出现在两种情况下：

+   为了节省一些内存空间，您可以使用 `--performance_schema_max_mutex_classes=*`N`*` 启动服务器，其中 *`N`* 小于默认值。默认值被选择为足以加载 MySQL 发行版中提供的所有插件，但如果某些插件从未加载，则可以减少此值。例如，您可能选择不加载发行版中的某些存储引擎。

+   您加载了一个为性能模式进行了仪器化的第三方插件，但在启动服务器时未考虑插件的仪器化内存需求。由于来自第三方，因此此引擎的仪器内存消耗不计入为 `performance_schema_max_mutex_classes` 选择的默认值。

    如果服务器对插件的仪器资源不足，并且您未明确使用 `--performance_schema_max_mutex_classes=*`N`*` 分配更多资源，则加载插件会导致仪器资源匮乏。

如果为`performance_schema_max_mutex_classes`选择的值太小，错误日志中不会报告任何错误，并且在运行时不会出现故障。然而，`performance_schema`数据库中的表内容会缺少事件。`Performance_schema_mutex_classes_lost`状态变量是唯一可见的迹象，表明由于无法创建仪器而导致一些事件被丢弃。

如果一个仪器没有丢失，那么性能模式就会知道它，并在实例化时使用。例如，`wait/synch/mutex/sql/LOCK_delete`是`setup_instruments`表中一个互斥体仪器的名称。这个单一仪器在代码中创建互斥体时使用（在`THD::LOCK_delete`中），然而服务器运行时需要多个互斥体实例。在这种情况下，`LOCK_delete`是一个每个连接（`THD`）的互斥体，所以如果服务器有 1000 个连接，就有 1000 个线程，以及 1000 个被标记的`LOCK_delete`互斥体实例（`THD::LOCK_delete`）。

如果服务器没有足够的空间来容纳这 1000 个被标记的互斥体（实例），一些互斥体会被创建并标记，而另一些则会被创建但不被标记。如果服务器只能创建 800 个实例，那么就会丢失 200 个实例。服务器继续运行，但会将`Performance_schema_mutex_instances_lost`增加 200，以表示无法创建实例。

如果`Performance_schema_mutex_instances_lost`的值大于 0，可能是因为代码在运行时初始化的互斥体比为`--performance_schema_max_mutex_instances=*`N`*`分配的数量多。

关键是，如果`SHOW STATUS LIKE 'perf%'`显示没有丢失任何内容（所有值都为零），那么性能模式数据是准确的，可以信赖。如果有内容丢失，数据就是不完整的，性能模式无法记录所有内容，因为给定的内存量不足。在这种情况下，特定的`Performance_schema_*`xxx`*_lost`变量指示了问题区域。

在某些情况下，有意造成仪器匮乏可能是合适的。例如，如果您不关心文件 I/O 的性能数据，可以将服务器启动时与文件 I/O 相关的所有性能模式参数设置为 0。不会为与文件相关的类、实例或句柄分配任何内存，并且所有文件事件都会丢失。

使用`SHOW ENGINE PERFORMANCE_SCHEMA STATUS`来检查性能模式代码的内部操作：

```sql
mysql> SHOW ENGINE PERFORMANCE_SCHEMA STATUS\G
...
*************************** 3\. row ***************************
  Type: performance_schema
  Name: events_waits_history.size
Status: 76
*************************** 4\. row ***************************
  Type: performance_schema
  Name: events_waits_history.count
Status: 10000
*************************** 5\. row ***************************
  Type: performance_schema
  Name: events_waits_history.memory
Status: 760000
...
*************************** 57\. row ***************************
  Type: performance_schema
  Name: performance_schema.memory
Status: 26459600
...
```

这个语句旨在帮助数据库管理员理解不同性能模式选项对内存需求的影响。有关字段含义的描述，请参阅第 15.7.7.15 节，“SHOW ENGINE Statement”。
