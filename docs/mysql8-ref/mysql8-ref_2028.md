> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-events-waits-history-long-table.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-events-waits-history-long-table.html)

#### 29.12.4.3 events_waits_history_long 表

`events_waits_history_long`表包含全局结束的*`N`*最近的等待事件，跨所有线程。等待事件直到结束后才会添加到表中。当表变满时，无论哪个线程生成了新行，最旧的行都会在添加新行时被丢弃。

性能模式在服务器启动期间自动调整*`N`*的值。要显式设置表大小，请在服务器启动时设置`performance_schema_events_waits_history_long_size`系统变量。

`events_waits_history_long`表与`events_waits_current`表具有相同的列。请参阅第 29.12.4.1 节，“events_waits_current 表”。与`events_waits_current`不同，`events_waits_history_long`没有索引。

`TRUNCATE TABLE`允许用于`events_waits_history_long`表。它会删除行。

有关三个等待事件表之间关系的更多信息，请参阅第 29.9 节，“当前和历史事件的性能模式表”。

关于配置是否收集等待事件的信息，请参阅第 29.12.4 节，“性能模式等待事件表”。
