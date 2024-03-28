> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-events-waits-history-table.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-events-waits-history-table.html)

#### 29.12.4.2 事件等待历史表

`events_waits_history`表包含每个线程已结束的*`N`*个最近的等待事件。直到等待事件结束后才将其添加到表中。当表包含给定线程的最大行数时，当为该线程添加新行时，最旧的线程行将被丢弃。当线程结束时，其所有行都将被丢弃。

性能模式在服务器启动期间自动调整*`N`*的值。要显式设置每个线程的行数，请在服务器启动时设置`performance_schema_events_waits_history_size`系统变量。

`events_waits_history`表具有与`events_waits_current`相同的列和索引。请参阅第 29.12.4.1 节，“事件等待当前表”。

`TRUNCATE TABLE`允许用于`events_waits_history`表。它会删除行。

有关三个等待事件表之间关系的更多信息，请参阅第 29.9 节，“当前和历史事件的性能模式表”。

有关配置是否收集等待事件的信息，请参阅第 29.12.4 节，“性能模式等待事件表”。
