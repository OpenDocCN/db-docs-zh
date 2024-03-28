> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-events-transactions-history-long-table.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-events-transactions-history-long-table.html)

#### 29.12.7.3 事件事务历史长表

`events_transactions_history_long` 表包含了全局结束的*`N`*最近的事务事件，跨所有线程。事务事件直到结束后才会添加到表中。当表满时，无论哪个线程生成了新行，最旧的行都会被丢弃。

性能模式在服务器启动时自动调整*`N`*的值。要显式设置表大小，请在服务器启动时设置`performance_schema_events_transactions_history_long_size`系统变量。

`events_transactions_history_long` 表与`events_transactions_current`具有相同的列。请参阅第 29.12.7.1 节，“事件事务当前表”。与`events_transactions_current`不同，`events_transactions_history_long`没有索引。

`TRUNCATE TABLE`允许用于`events_transactions_history_long`表。它会删除行。

有关三个事务事件表之间关系的更多信息，请参阅第 29.9 节，“当前和历史事件的性能模式表”。

有关配置是否收集事务事件的信息，请参阅第 29.12.7 节，“性能模式事务表”。
