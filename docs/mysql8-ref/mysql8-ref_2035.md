> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-events-statements-history-table.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-events-statements-history-table.html)

#### 29.12.6.2 events_statements_history 表

`events_statements_history`表包含每个线程已结束的*`N`*个最近的语句事件。语句事件直到结束后才会添加到表中。当表包含给定线程的最大行数时，当为该线程添加新行时，最旧的线程行将被丢弃。当线程结束时，其所有行都将被丢弃。

性能模式在服务器启动时自动调整*`N`*的值。要显式设置每个线程的行数，请在服务器启动时设置`performance_schema_events_statements_history_size`系统变量。

`events_statements_history`表具有与`events_statements_current`相同的列和索引。请参阅 Section 29.12.6.1, “events_statements_current 表”。

`TRUNCATE TABLE`允许用于`events_statements_history`表。它会删除行。

要了解三个`events_statements_*`xxx`*`事件表之间的关系，请参阅 Section 29.9, “当前和历史事件的性能模式表”。

有关配置是否收集语句事件的信息，请参阅 Section 29.12.6, “性能模式语句事件表”。
