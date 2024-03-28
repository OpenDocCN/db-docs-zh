# 29.12 性能模式表描述

> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-table-descriptions.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-table-descriptions.html)

29.12.1 性能模式表参考

29.12.2 性能模式设置表

29.12.3 性能模式实例表

29.12.4 性能模式等待事件表

29.12.5 性能模式阶段事件表

29.12.6 性能模式语句事件表

29.12.7 性能模式事务表

29.12.8 性能模式连接表

29.12.9 性能模式连接属性表

29.12.10 性能模式用户定义变量表

29.12.11 性能模式复制表

29.12.12 性能模式 NDB 集群表

29.12.13 性能模式锁表

29.12.14 性能模式系统变量表

29.12.15 性能模式状态变量表

29.12.16 性能模式线程池表

29.12.17 性能模式防火墙表

29.12.18 性能模式密钥环表

29.12.19 性能模式克隆表

29.12.20 性能模式摘要表

29.12.21 性能模式杂项表

`performance_schema` 数据库中的表可以分为以下几类：

+   设置表。这些表用于配置和显示监控特性。

+   当前事件表。`events_waits_current` 表包含每个线程的最新事件。其他类似的表包含不同层次事件的当前事件：`events_stages_current` 用于阶段事件，`events_statements_current` 用于语句事件，以及`events_transactions_current` 用于事务事件。

+   历史表。这些表与当前事件表具有相同的结构，但包含更多行。例如，对于等待事件，`events_waits_history` 表包含每个线程的最新 10 个事件。`events_waits_history_long` 包含最近的 10,000 个事件。其他类似的表存在于阶段、语句和事务历史中。

    要更改历史表的大小，请在服务器启动时设置适当的系统变量。例如，要设置等待事件历史表的大小，请设置`performance_schema_events_waits_history_size` 和`performance_schema_events_waits_history_long_size`。

+   汇总表。这些表包含对事件组进行聚合的信息，包括已从历史表中丢弃的事件。

+   实例表。这些表记录了被检测对象的类型。当服务器使用被检测对象时，会产生一个事件。这些表提供事件名称、解释说明或状态信息。

+   杂项表。这些表不属于任何其他表组。
