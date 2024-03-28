> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-stage-summary-tables.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-stage-summary-tables.html)

#### 29.12.20.2 阶段摘要表

性能模式维护用于收集当前和最近阶段事件的表，并在摘要表中聚合该信息。Section 29.12.5, “性能模式阶段事件表”描述了阶段摘要的基础事件。请参阅该讨论以获取有关阶段事件内容、当前和历史阶段事件表以及如何控制阶段事件收集（默认情况下已禁用）的信息。

示例阶段事件摘要信息：

```sql
mysql> SELECT *
       FROM performance_schema.events_stages_summary_global_by_event_name\G
...
*************************** 5\. row ***************************
    EVENT_NAME: stage/sql/checking permissions
    COUNT_STAR: 57
SUM_TIMER_WAIT: 26501888880
MIN_TIMER_WAIT: 7317456
AVG_TIMER_WAIT: 464945295
MAX_TIMER_WAIT: 12858936792
...
*************************** 9\. row ***************************
    EVENT_NAME: stage/sql/closing tables
    COUNT_STAR: 37
SUM_TIMER_WAIT: 662606568
MIN_TIMER_WAIT: 1593864
AVG_TIMER_WAIT: 17907891
MAX_TIMER_WAIT: 437977248
...
```

每个阶段摘要表都有一个或多个分组列，用于指示表如何聚合事件。事件名称指的是`setup_instruments`表中事件工具的名称：

+   `events_stages_summary_by_account_by_event_name`具有`EVENT_NAME`，`USER`和`HOST`列。每行总结了给定帐户（用户和主机组合）和事件名称的事件。

+   `events_stages_summary_by_host_by_event_name`具有`EVENT_NAME`和`HOST`列。每行总结了给定主机和事件名称的事件。

+   `events_stages_summary_by_thread_by_event_name`具有`THREAD_ID`和`EVENT_NAME`列。每行总结了给定线程和事件名称的事件。

+   `events_stages_summary_by_user_by_event_name`具有`EVENT_NAME`和`USER`列。每行总结了给定用户和事件名称的事件。

+   `events_stages_summary_global_by_event_name`具有一个`EVENT_NAME`列。每行总结了给定事件名称的事件。

每个阶段摘要表都有这些包含聚合值的摘要列：`COUNT_STAR`，`SUM_TIMER_WAIT`，`MIN_TIMER_WAIT`，`AVG_TIMER_WAIT`和`MAX_TIMER_WAIT`。这些列类似于等待事件摘要表中同名列的列（参见 Section 29.12.20.1, “等待事件摘要表”)，不同之处在于阶段摘要表聚合了来自`events_stages_current`而不是`events_waits_current`的事件。

阶段摘要表具有以下索引：

+   `events_stages_summary_by_account_by_event_name`：

    +   主键为 (`USER`, `HOST`, `EVENT_NAME`)

+   `events_stages_summary_by_host_by_event_name`：

    +   主键为 (`HOST`, `EVENT_NAME`)

+   `events_stages_summary_by_thread_by_event_name`：

    +   主键为 (`THREAD_ID`, `EVENT_NAME`)

+   `events_stages_summary_by_user_by_event_name`：

    +   主键为 (`USER`, `EVENT_NAME`)

+   `events_stages_summary_global_by_event_name`：

    +   主键为 (`EVENT_NAME`)

`TRUNCATE TABLE` 可用于阶段汇总表。它具有以下效果：

+   对于不按帐户、主机或用户聚合的汇总表，截断会将汇总列重置为零，而不是删除行。

+   对于按帐户、主机或用户聚合的汇总表，截断会删除没有连接的帐户、主机或用户的行，并将剩余行的汇总列重置为零。

此外，每个按帐户、主机、用户或线程聚合的阶段汇总表在其依赖的连接表被截断或截断 `events_stages_summary_global_by_event_name` 时会被隐式截断。有关详细信息，请参见 Section 29.12.8, “性能模式连接表”。
