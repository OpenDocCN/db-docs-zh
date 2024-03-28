> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-events-stages-history-long-table.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-events-stages-history-long-table.html)

#### 29.12.5.3 events_stages_history_long 表

`events_stages_history_long`表包含全局结束的*`N`*最近的阶段事件，跨所有线程。直到阶段事件结束后才将其添加到表中。当表已满时，添加新行时会丢弃最旧的行，无论哪个线程生成了这两行。

性能模式在服务器启动期间自动调整*`N`*的值。要显式设置表大小，请在服务器启动时设置`performance_schema_events_stages_history_long_size`系统变量。

`events_stages_history_long`表与`events_stages_current`表具有相同的列。请参阅第 29.12.5.1 节，“events_stages_current 表”。与`events_stages_current`表不同，`events_stages_history_long`表没有索引。

`TRUNCATE TABLE`允许用于`events_stages_history_long`表。它会删除行。

有关三个阶段事件表之间关系的更多信息，请参阅第 29.9 节，“当前和历史事件的性能模式表”。

关于配置是否收集阶段事件的信息，请参阅第 29.12.5 节，“性能模式阶段事件表”。
