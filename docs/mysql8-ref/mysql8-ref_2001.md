# 29.4.8 示例消费者配置

> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-consumer-configurations.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-consumer-configurations.html)

`setup_consumers`表中的消费者设置形成从高级到低级的层次结构。以下讨论描述了消费者如何工作，展示了特定配置及其效果，随着从高到低逐渐启用消费者设置。所示的消费者值是代表性的。这里描述的一般原则适用于可能可用的其他消费者值。

配置描述按功能和开销递增的顺序出现。如果不需要启用较低级别设置提供的信息，则禁用它们，以便性能模式在您的代表执行更少的代码，并且需要筛选的信息更少。

`setup_consumers`表包含以下值的层次结构：

```sql
global_instrumentation
 thread_instrumentation
   events_waits_current
     events_waits_history
     events_waits_history_long
   events_stages_current
     events_stages_history
     events_stages_history_long
   events_statements_current
     events_statements_history
     events_statements_history_long
   events_transactions_current
     events_transactions_history
     events_transactions_history_long
 statements_digest
```

注意

在消费者层次结构中，等待、阶段、语句和事务的消费者都处于同一级别。这与事件嵌套层次结构不同，等待事件嵌套在阶段事件内，阶段事件嵌套在语句事件内，语句事件嵌套在事务事件内。

如果给定的消费者设置为`NO`，性能模式将禁用与该消费者相关联的仪器，并忽略所有较低级别的设置。如果给定的设置为`YES`，性能模式将启用与之相关联的仪器，并检查下一个较低级别的设置。有关每个消费者的规则描述，请参见第 29.4.7 节“按消费者进行预过滤”。

例如，如果启用了`global_instrumentation`，则会检查`thread_instrumentation`。如果启用了`thread_instrumentation`，则会检查`events_*`xxx`*_current`消费者。如果其中的`events_waits_current`已启用，则会检查`events_waits_history`和`events_waits_history_long`。

以下每个配置描述指示性能模式检查哪些设置元素，并维护哪些输出表（即，为哪些表收集信息）。

+   无仪器

+   仅全局仪器

+   仅全局和线程仪器

+   全局、线程和当前事件仪器

+   全局、线程、当前事件和事件历史仪器

#### 无仪器

服务器配置状态：

```sql
mysql> SELECT * FROM performance_schema.setup_consumers;
+---------------------------+---------+
| NAME                      | ENABLED |
+---------------------------+---------+
| global_instrumentation    | NO      |
...
+---------------------------+---------+
```

在这种配置中，没有任何仪器。

检查的设置元素：

+   表 `setup_consumers`, 消费者 `global_instrumentation`

维护的输出表：

+   无

#### 仅全局仪器

服务器配置状态：

```sql
mysql> SELECT * FROM performance_schema.setup_consumers;
+---------------------------+---------+
| NAME                      | ENABLED |
+---------------------------+---------+
| global_instrumentation    | YES     |
| thread_instrumentation    | NO      |
...
+---------------------------+---------+
```

在这种配置中，仪器仅用于全局状态。每个线程的仪器被禁用。

相对于前述配置，检查的额外设置元素：

+   表 `setup_consumers`, 消费者 `thread_instrumentation`

+   表 `setup_instruments`

+   表 `setup_objects`

相对于前述配置，维护的额外输出表：

+   `mutex_instances`

+   `rwlock_instances`

+   `cond_instances`

+   `file_instances`

+   `users`

+   `hosts`

+   `accounts`

+   `socket_summary_by_event_name`

+   `file_summary_by_instance`

+   `file_summary_by_event_name`

+   `objects_summary_global_by_type`

+   `memory_summary_global_by_event_name`

+   `table_lock_waits_summary_by_table`

+   `table_io_waits_summary_by_index_usage`

+   `table_io_waits_summary_by_table`

+   `events_waits_summary_by_instance`

+   `events_waits_summary_global_by_event_name`

+   `events_stages_summary_global_by_event_name`

+   `events_statements_summary_global_by_event_name`

+   `events_transactions_summary_global_by_event_name`

#### 仅全局和线程仪器

服务器配置状态：

```sql
mysql> SELECT * FROM performance_schema.setup_consumers;
+----------------------------------+---------+
| NAME                             | ENABLED |
+----------------------------------+---------+
| global_instrumentation           | YES     |
| thread_instrumentation           | YES     |
| events_waits_current             | NO      |
...
| events_stages_current            | NO      |
...
| events_statements_current        | NO      |
...
| events_transactions_current      | NO      |
...
+----------------------------------+---------+
```

在这种配置下，仪器仪表是全局和每个线程都维护的。当前事件或事件历史表中不收集任何个别事件。

相对于前述配置，检查了额外的设置元素：

+   表`setup_consumers`，消费者`events_*`xxx`*_current`，其中*`xxx`*是`waits`、`stages`、`statements`、`transactions`

+   表`setup_actors`

+   列`threads.instrumented`

相对于前述配置，维护了额外的输出表：

+   `events_*`xxx`*_summary_by_*`yyy`*_by_event_name`，其中*`xxx`*是`waits`、`stages`、`statements`、`transactions`；*`yyy`*是`thread`、`user`、`host`、`account`

#### 全局、线程和当前事件仪器

服务器配置状态：

```sql
mysql> SELECT * FROM performance_schema.setup_consumers;
+----------------------------------+---------+
| NAME                             | ENABLED |
+----------------------------------+---------+
| global_instrumentation           | YES     |
| thread_instrumentation           | YES     |
| events_waits_current             | YES     |
| events_waits_history             | NO      |
| events_waits_history_long        | NO      |
| events_stages_current            | YES     |
| events_stages_history            | NO      |
| events_stages_history_long       | NO      |
| events_statements_current        | YES     |
| events_statements_history        | NO      |
| events_statements_history_long   | NO      |
| events_transactions_current      | YES     |
| events_transactions_history      | NO      |
| events_transactions_history_long | NO      |
...
+----------------------------------+---------+
```

在这种配置下，仪器仪表是全局和每个线程都维护的。个别事件在当前事件表中收集，但不在事件历史表中。

相对于前述配置，检查了额外的设置元素：

+   消费者`events_*`xxx`*_history`，其中*`xxx`*是`waits`、`stages`、`statements`、`transactions`

+   消费者`events_*`xxx`*_history_long`，其中*`xxx`*是`waits`、`stages`、`statements`、`transactions`

相对于前述配置，维护了额外的输出表：

+   `events_*`xxx`*_current`，其中*`xxx`*是`waits`、`stages`、`statements`、`transactions`

#### 全局、线程、当前事件和事件历史仪器

前述配置未收集任何事件历史，因为`events_*`xxx`*_history`和`events_*`xxx`*_history_long`消费者已禁用。这些消费者可以单独或一起启用，以便按线程、全局或两者同时收集事件历史。

此配置按线程收集事件历史，但不全局：

```sql
mysql> SELECT * FROM performance_schema.setup_consumers;
+----------------------------------+---------+
| NAME                             | ENABLED |
+----------------------------------+---------+
| global_instrumentation           | YES     |
| thread_instrumentation           | YES     |
| events_waits_current             | YES     |
| events_waits_history             | YES     |
| events_waits_history_long        | NO      |
| events_stages_current            | YES     |
| events_stages_history            | YES     |
| events_stages_history_long       | NO      |
| events_statements_current        | YES     |
| events_statements_history        | YES     |
| events_statements_history_long   | NO      |
| events_transactions_current      | YES     |
| events_transactions_history      | YES     |
| events_transactions_history_long | NO      |
...
+----------------------------------+---------+
```

为此配置维护的事件历史表：

+   `events_*`xxx`*_history`，其中*`xxx`*是`waits`、`stages`、`statements`、`transactions`

此配置全局收集事件历史，但不按线程：

```sql
mysql> SELECT * FROM performance_schema.setup_consumers;
+----------------------------------+---------+
| NAME                             | ENABLED |
+----------------------------------+---------+
| global_instrumentation           | YES     |
| thread_instrumentation           | YES     |
| events_waits_current             | YES     |
| events_waits_history             | NO      |
| events_waits_history_long        | YES     |
| events_stages_current            | YES     |
| events_stages_history            | NO      |
| events_stages_history_long       | YES     |
| events_statements_current        | YES     |
| events_statements_history        | NO      |
| events_statements_history_long   | YES     |
| events_transactions_current      | YES     |
| events_transactions_history      | NO      |
| events_transactions_history_long | YES     |
...
+----------------------------------+---------+
```

为此配置维护的事件历史表：

+   `events_*`xxx`*_history_long`，其中*`xxx`*是`waits`、`stages`、`statements`、`transactions`

此配置按线程和全局收集事件历史：

```sql
mysql> SELECT * FROM performance_schema.setup_consumers;
+----------------------------------+---------+
| NAME                             | ENABLED |
+----------------------------------+---------+
| global_instrumentation           | YES     |
| thread_instrumentation           | YES     |
| events_waits_current             | YES     |
| events_waits_history             | YES     |
| events_waits_history_long        | YES     |
| events_stages_current            | YES     |
| events_stages_history            | YES     |
| events_stages_history_long       | YES     |
| events_statements_current        | YES     |
| events_statements_history        | YES     |
| events_statements_history_long   | YES     |
| events_transactions_current      | YES     |
| events_transactions_history      | YES     |
| events_transactions_history_long | YES     |
...
+----------------------------------+---------+
```

为此配置维护的事件历史表：

+   `events_*`xxx`*_history`，其中*`xxx`*是`waits`、`stages`、`statements`、`transactions`

+   `events_*`xxx`*_history_long`，其中*`xxx`*是`waits`、`stages`、`statements`、`transactions`
