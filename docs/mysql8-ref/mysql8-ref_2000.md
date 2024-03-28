# 29.4.7 按消费者进行预过滤

> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-consumer-filtering.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-consumer-filtering.html)

`setup_consumers`表列出了可用的消费者类型以及哪些已启用：

```sql
mysql> SELECT * FROM performance_schema.setup_consumers;
+----------------------------------+---------+
| NAME                             | ENABLED |
+----------------------------------+---------+
| events_stages_current            | NO      |
| events_stages_history            | NO      |
| events_stages_history_long       | NO      |
| events_statements_cpu            | NO      |
| events_statements_current        | YES     |
| events_statements_history        | YES     |
| events_statements_history_long   | NO      |
| events_transactions_current      | YES     |
| events_transactions_history      | YES     |
| events_transactions_history_long | NO      |
| events_waits_current             | NO      |
| events_waits_history             | NO      |
| events_waits_history_long        | NO      |
| global_instrumentation           | YES     |
| thread_instrumentation           | YES     |
| statements_digest                | YES     |
+----------------------------------+---------+
```

修改`setup_consumers`表以影响消费者阶段的预过滤，并确定事件发送的目的地。要启用或禁用一个消费者，将其`ENABLED`值设置为`YES`或`NO`。

对`setup_consumers`表的修改会立即影响监控。

如果禁用一个消费者，服务器将不再花时间维护该消费者的目的地。例如，如果你不关心历史事件信息，可以禁用历史消费者：

```sql
UPDATE performance_schema.setup_consumers
SET ENABLED = 'NO'
WHERE NAME LIKE '%history%';
```

`setup_consumers`表中的消费者设置形成从高级到低级的层次结构。以下原则适用：

+   与消费者关联的目的地只有在性能模式检查了消费者并且消费者已启用时才会接收事件。

+   一个消费者只有在它所依赖的所有消费者（如果有的话）都启用时才会被选中。

+   如果一个消费者未被选中，或者被选中但被禁用，依赖它的其他消费者也不会被选中。

+   依赖消费者可能有自己的依赖消费者。

+   如果一个事件不会被发送到任何目的地，性能模式就不会产生它。

以下列表描述了可用的消费者值。有关几种代表性消费者配置及其对仪表化的影响的讨论，请参见第 29.4.8 节，“示例消费者配置”。

+   全局和线程消费者

+   等待事件消费者

+   阶段事件消费者

+   语句事件消费者

+   事务事件消费者

+   语句摘要消费者

#### 全局和线程消费者

+   `global_instrumentation` 是最高级别的消费者。如果 `global_instrumentation` 为 `NO`，它会禁用全局仪器。所有其他设置都是较低级别的，不会被检查；它们设置为什么并不重要。不会维护全局或每个线程的信息，也不会在当前事件或事件历史表中收集任何个别事件。如果 `global_instrumentation` 为 `YES`，性能模式会维护全局状态的信息，并且还会检查 `thread_instrumentation` 消费者。

+   `thread_instrumentation` 只有在 `global_instrumentation` 为 `YES` 时才会被检查。否则，如果 `thread_instrumentation` 为 `NO`，它会禁用线程特定的仪器，并且所有更低级别的设置都会被忽略。不会为每个线程维护任何信息，也不会在当前事件或事件历史表中收集任何个别事件。如果 `thread_instrumentation` 为 `YES`，性能模式会维护线程特定信息，并且还会检查 `events_*`xxx`*_current` 消费者。

#### 等待事件消费者

这些消费者要求 `global_instrumentation` 和 `thread_instrumentation` 都为 `YES`，否则不会被检查。如果被检查，它们的作用如下：

+   如果 `events_waits_current` 为 `NO`，则会禁用在 `events_waits_current` 表中个别等待事件的收集。如果 `YES`，则会启用等待事件的收集，并且性能模式��检查 `events_waits_history` 和 `events_waits_history_long` 消费者。

+   如果 `event_waits_current` 为 `NO`，则不会检查 `events_waits_history`。否则，`events_waits_history` 的值为 `NO` 或 `YES` 会禁用或启用在 `events_waits_history` 表中等待事件的收集。

+   如果 `event_waits_current` 为 `NO`，则不会检查 `events_waits_history_long`。否则，`events_waits_history_long` 的值为 `NO` 或 `YES` 会禁用或启用在 `events_waits_history_long` 表中等待事件的收集。

#### 阶段事件消费者

这些消费者要求 `global_instrumentation` 和 `thread_instrumentation` 都为 `YES`，否则不会被检查。如果被检查，它们的作用如下：

+   如果 `events_stages_current` 为 `NO`，则会禁用在 `events_stages_current` 表中个别阶段事件的收集。如果 `YES`，则会启用阶段事件的收集，并且性能模式会检查 `events_stages_history` 和 `events_stages_history_long` 消费者。

+   如果`event_stages_current`为`NO`，则不会检查`events_stages_history`。否则，`events_stages_history`的值为`NO`或`YES`将禁用或启用在`events_stages_history`表中阶段事件的收集。

+   如果`event_stages_current`为`NO`，则不会检查`events_stages_history_long`。否则，`events_stages_history_long`的值为`NO`或`YES`将禁用或启用在`events_stages_history_long`表中阶段事件的收集。

#### 语句事件消费者

这些消费者需要`global_instrumentation`和`thread_instrumentation`都设置为`YES`，否则将不会被检查。如果被选中，它们的作用如下：

+   如果`events_statements_cpu`为`NO`，则禁用`CPU_TIME`的测量。如果为`YES`，并且启用了仪器和计时，将测量`CPU_TIME`。

+   如果`events_statements_current`为`NO`，则禁用在`events_statements_current`表中收集单个语句事件。如果为`YES`，则启用语句事件收集，并且性能模式将检查`events_statements_history`和`events_statements_history_long`消费者。

+   如果`events_statements_current`为`NO`，则不会检查`events_statements_history`。否则，`events_statements_history`的值为`NO`或`YES`将禁用或启用在`events_statements_history`表中语句事件的收集。

+   如果`events_statements_current`为`NO`，则不会检查`events_statements_history_long`。否则，`events_statements_history_long`的值为`NO`或`YES`将禁用或启用在`events_statements_history_long`表中语句事件的收集。

#### 事务事件消费者

这些消费者需要`global_instrumentation`和`thread_instrumentation`都设置为`YES`，否则将不会被检查。如果被选中，它们的作用如下：

+   如果`events_transactions_current`为`NO`，则禁用在`events_transactions_current`表中收集单个事务事件。如果为`YES`，则启用事务事件收集，并且性能模式将检查`events_transactions_history`和`events_transactions_history_long`消费者。

+   如果`events_transactions_current`为`NO`，则不会检查`events_transactions_history`。否则，`events_transactions_history`的值为`NO`或`YES`会禁用或启用`events_transactions_history`表中事务事件的收集。

+   如果`events_transactions_current`为`NO`，则不会检查`events_transactions_history_long`。否则，`events_transactions_history_long`的值为`NO`或`YES`会禁用或启用`events_transactions_history_long`表中事务事件的收集。

#### 语句摘要消费者

`statements_digest`消费者要求`global_instrumentation`为`YES`，否则不会进行检查。对语句事件消费者没有依赖，因此您可以在不必在`events_statements_current`中收集统计信息的情况下，获得每个摘要的统计信息，这在开销方面是有利的。相反，您可以在`events_statements_current`中获取详细的语句，而不需要摘要（在这种情况下，`DIGEST`和`DIGEST_TEXT`列为`NULL`）。

关于语句摘要的更多信息，请参见第 29.10 节，“性能模式语句摘要和抽样”。
