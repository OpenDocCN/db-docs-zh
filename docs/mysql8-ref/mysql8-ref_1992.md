# 29.3 性能模式启动配置

> [`dev.mysql.com/doc/refman/8.0/en/performance-schema-startup-configuration.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-startup-configuration.html)

要使用 MySQL 性能模式，必须在服务器启动时启用，以便进行事件收集。

性能模式默认启用。要显式启用或禁用它，请使用将服务器启动时`performance_schema`变量设置为适当值。例如，在服务器`my.cnf`文件中使用以下行：

```sql
[mysqld]
performance_schema=ON
```

如果服务器在性能模式初始化期间无法分配任何内部缓冲区，则性能模式会禁用自身，并将`performance_schema`设置为`OFF`，服务器将在没有仪器的情况下运行。

性能模式还允许在服务器启动时配置仪器和消费者。

要在服务器启动时控制仪器，请使用以下形式的选项：

```sql
--performance-schema-instrument='*instrument_name*=*value*'
```

这里，*`instrument_name`*是一个仪器名称，例如`wait/synch/mutex/sql/LOCK_open`，*`value`*是以下值之一：

+   `OFF`、`FALSE`或`0`：禁用仪器

+   `ON`、`TRUE`或`1`：启用并计时仪器

+   `COUNTED`：启用并计数（而不是计时）仪器

每个`--performance-schema-instrument`选项只能指定一个仪器名称，但可以给出多个选项的实例以配置多个仪器。此外，仪器名称中允许使用模式以配置与模式匹配的仪器。要将所有条件同步仪器配置为启用和计数，请使用此选项：

```sql
--performance-schema-instrument='wait/synch/cond/%=COUNTED'
```

要禁用所有仪器，请使用此选项：

```sql
--performance-schema-instrument='%=OFF'
```

例外：`memory/performance_schema/%`仪器是内置的，无法在启动时禁用。

较长的仪器名称字符串优先于较短的模式名称，无论顺序如何。有关指定模式以选择仪器的信息，请参阅第 29.4.9 节，“用于过滤操作的命名仪器或消费者”。

未识别的仪器名称将被忽略。后来安装的插件可能会创建该仪器，届时名称将被识别并配置。

要在服务器启动时控制消费者，请使用以下形式的选项：

```sql
--performance-schema-consumer-*consumer_name*=*value*
```

这里，*`consumer_name`*是一个消费者名称，例如`events_waits_history`，*`value`*是以下值之一：

+   `OFF`、`FALSE`或`0`：不为消费者收集事件

+   `ON`、`TRUE`或`1`：为消费者收集事件

例如，要启用`events_waits_history`消费者，请使用此选项：

```sql
--performance-schema-consumer-events-waits-history=ON
```

可以通过检查 `setup_consumers` 表找到允许的消费者名称。不允许使用模式。`setup_consumers` 表中的消费者名称使用下划线，但对于在启动时设置的消费者，名称中的破折号和下划线是等效的。

Performance Schema 包括几个提供配置信息的系统变量：

```sql
mysql> SHOW VARIABLES LIKE 'perf%';
+--------------------------------------------------------+---------+
| Variable_name                                          | Value   |
+--------------------------------------------------------+---------+
| performance_schema                                     | ON      |
| performance_schema_accounts_size                       | 100     |
| performance_schema_digests_size                        | 200     |
| performance_schema_events_stages_history_long_size     | 10000   |
| performance_schema_events_stages_history_size          | 10      |
| performance_schema_events_statements_history_long_size | 10000   |
| performance_schema_events_statements_history_size      | 10      |
| performance_schema_events_waits_history_long_size      | 10000   |
| performance_schema_events_waits_history_size           | 10      |
| performance_schema_hosts_size                          | 100     |
| performance_schema_max_cond_classes                    | 80      |
| performance_schema_max_cond_instances                  | 1000    |
...
```

`performance_schema` 变量为 `ON` 或 `OFF`，表示 Performance Schema 是否已启用或已禁用。其他变量表示表大小（行数）或内存分配值。

注意

启用 Performance Schema 后，Performance Schema 实例的数量会影响服务器的内存占用，可能在很大程度上。Performance Schema 会自动调整许多参数，仅在需要时使用内存；请参见 Section 29.17, “The Performance Schema Memory-Allocation Model”。

要更改 Performance Schema 系统变量的值，请在服务器启动时设置它们。例如，将以下行放入 `my.cnf` 文件中以更改等待事件的历史表大小：

```sql
[mysqld]
performance_schema
performance_schema_events_waits_history_size=20
performance_schema_events_waits_history_long_size=15000
```

Performance Schema 在服务器启动时会自动调整一些参数的值，如果它们没有被显式设置。例如，它会以这种方式调整控制事件等待表大小的参数。Performance Schema 会逐步分配内存，根据实际服务器负载来调整内存使用量，而不是在服务器启动时分配所有需要的内存。因此，许多调整参数根本不需要设置。要查看哪些参数是自动调整或自动缩放的，请使用 **mysqld --verbose --help** 并查看选项描述，或参见 Section 29.15, “Performance Schema System Variables”。

对于每个未在服务器启动时设置的自动调整参数，Performance Schema 根据以下系统值的值确定如何设置其值，这些值被视为关于如何配置 MySQL 服务器的“提示”：

```sql
max_connections
open_files_limit
table_definition_cache
table_open_cache
```

要覆盖给定参数的自动调整或自动缩放，请在启动时将其设置为非 -1 的值。在这种情况下，Performance Schema 将其分配为指定的值。

在运行时，`SHOW VARIABLES` 显示自动调整参数设置的实际值。自动缩放参数显示为 -1。

如果性能模式被禁用，其自动调整大小和自动缩放参数仍设置为−1，`SHOW VARIABLES`显示−1。
