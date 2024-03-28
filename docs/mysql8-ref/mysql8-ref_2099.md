> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-objects-summary-global-by-type-table.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-objects-summary-global-by-type-table.html)

#### 29.12.20.6 对象等待摘要表

性能模式维护`objects_summary_global_by_type`表以聚合对象等待事件。

示例对象等待事件摘要信息：

```sql
mysql> SELECT * FROM performance_schema.objects_summary_global_by_type\G
...
*************************** 3\. row ***************************
   OBJECT_TYPE: TABLE
 OBJECT_SCHEMA: test
   OBJECT_NAME: t
    COUNT_STAR: 3
SUM_TIMER_WAIT: 263126976
MIN_TIMER_WAIT: 1522272
AVG_TIMER_WAIT: 87708678
MAX_TIMER_WAIT: 258428280
...
*************************** 10\. row ***************************
   OBJECT_TYPE: TABLE
 OBJECT_SCHEMA: mysql
   OBJECT_NAME: user
    COUNT_STAR: 14
SUM_TIMER_WAIT: 365567592
MIN_TIMER_WAIT: 1141704
AVG_TIMER_WAIT: 26111769
MAX_TIMER_WAIT: 334783032
...
```

`objects_summary_global_by_type`表具有这些分组列，用于指示表如何聚合事件：`OBJECT_TYPE`，`OBJECT_SCHEMA`和`OBJECT_NAME`。每行总结了给定对象的事件。

`objects_summary_global_by_type`具有与`events_waits_summary_by_*`xxx`*`表相同的摘要列。请参阅第 29.12.20.1 节，“等待事件摘要表”。

`objects_summary_global_by_type`表具有以下索引：

+   主键为(`OBJECT_TYPE`, `OBJECT_SCHEMA`, `OBJECT_NAME`)

`TRUNCATE TABLE`允许用于对象摘要表。它将摘要列重置为零，而不是删除行。
