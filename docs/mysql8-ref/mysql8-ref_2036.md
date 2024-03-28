> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-events-statements-history-long-table.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-events-statements-history-long-table.html)

#### 29.12.6.3 `events_statements_history_long` 表

`events_statements_history_long` 表包含全局结束的 *`N`* 最近语句事件，跨所有线程。语句事件直到结束后才会添加到表中。当表满时，当添加新行时，最旧的行将被丢弃，无论哪个线程生成了这两行。

*`N`* 的值在服务器启动时会自动调整大小。要显式设置表大小，请在服务器启动时设置`performance_schema_events_statements_history_long_size`系统变量。

`events_statements_history_long` 表与 `events_statements_current` 表具有相同的列。请参见第 29.12.6.1 节，“events_statements_current 表”。与 `events_statements_current` 不同，`events_statements_history_long` 没有索引。

`TRUNCATE TABLE` 允许用于 `events_statements_history_long` 表。它会删除行。

关于三个 `events_statements_*`xxx`*` 事件表之间的关系的更多信息，请参见第 29.9 节，“当前和历史事件的性能模式表”。

有关配置是否收集语句事件的信息，请参见第 29.12.6 节，“性能模式语句事件表”。
