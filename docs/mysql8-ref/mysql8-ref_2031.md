> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-events-stages-history-table.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-events-stages-history-table.html)

#### 29.12.5.2 `events_stages_history` 表

`events_stages_history` 表包含每个线程中最近结束的 *`N`* 个阶段事件。阶段事件直到结束后才会添加到表中。当表中包含给定线程的最大行数时，当为该线程添加新行时，最旧的线程行将被丢弃。当线程结束时，其所有行都将被丢弃。

在服务器启动期间，性能模式会自动调整 *`N`* 的值。要显式设置每个线程的行数，请在服务器启动时设置 `performance_schema_events_stages_history_size` 系统变量。

`events_stages_history` 表具有与 `events_stages_current` 相同的列和索引。请参阅 Section 29.12.5.1, “`events_stages_current` 表”。

允许对 `events_stages_history` 表执行 `TRUNCATE TABLE`。它会删除行。

有关三个阶段事件表之间关系的更多信息，请参阅 Section 29.9, “当前和历史事件的性能模式表”。

有关配置是否收集阶段事件的信息，请参阅 Section 29.12.5, “性能模式阶段事件表”。
