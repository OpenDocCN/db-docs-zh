> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-wait-summary-tables.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-wait-summary-tables.html)

#### 29.12.20.1 等待事件摘要表

性能模式维护表用于收集当前和最近的等待事件，并在摘要表中汇总这些信息。第 29.12.4 节，“性能模式等待事件表”描述了等待摘要的基础事件。请参阅该讨论以获取有关等待事件内容、当前和最近的等待事件表以及如何控制等待事件收集（默认情况下已禁用）的信息。

示例等待事件摘要信息：

```sql
mysql> SELECT *
       FROM performance_schema.events_waits_summary_global_by_event_name\G
...
*************************** 6\. row ***************************
    EVENT_NAME: wait/synch/mutex/sql/BINARY_LOG::LOCK_index
    COUNT_STAR: 8
SUM_TIMER_WAIT: 2119302
MIN_TIMER_WAIT: 196092
AVG_TIMER_WAIT: 264912
MAX_TIMER_WAIT: 569421
...
*************************** 9\. row ***************************
    EVENT_NAME: wait/synch/mutex/sql/hash_filo::lock
    COUNT_STAR: 69
SUM_TIMER_WAIT: 16848828
MIN_TIMER_WAIT: 0
AVG_TIMER_WAIT: 244185
MAX_TIMER_WAIT: 735345
...
```

每个等待事件摘要表都有一个或多个分组列，用于指示表如何聚合事件。事件名称指的是`setup_instruments`表中事件工具的名称：

+   `events_waits_summary_by_account_by_event_name`具有`EVENT_NAME`、`USER`和`HOST`列。每行总结了给定帐户（用户和主机组合）和事件名称的事件。

+   `events_waits_summary_by_host_by_event_name`具有`EVENT_NAME`和`HOST`列。每行总结了给定主机和事件名称的事件。

+   `events_waits_summary_by_instance`具有`EVENT_NAME`和`OBJECT_INSTANCE_BEGIN`列。每行总结了给定事件名称和对象的事件。如果一个工具用于创建多个实例，则每个实例都有一个唯一的`OBJECT_INSTANCE_BEGIN`值，并在此表中单独进行总结。

+   `events_waits_summary_by_thread_by_event_name`具有`THREAD_ID`和`EVENT_NAME`列。每行总结了给定线程和事件名称的事件。

+   `events_waits_summary_by_user_by_event_name`具有`EVENT_NAME`和`USER`列。每行总结了给定用户和事件名称的事件。

+   `events_waits_summary_global_by_event_name`具有一个`EVENT_NAME`列。每行总结了给定事件名称的事件。一个工具可能用于创建被检测对象的多个实例。例如，如果为每个连接创建一个互斥体的工具，则实例的数量与连接数量相同。工具的摘要行总结了所有这些实例。

每个等待事件摘要表都有这些包含聚合值的摘要列：

+   `COUNT_STAR`

    汇总事件的数量。此值包括所有事件，无论是定时的还是非定时的。

+   `SUM_TIMER_WAIT`

    汇总定时事件的总等待时间。此值仅针对定时事件计算，因为非定时事件的等待时间为 `NULL`。对于其他 `*`xxx`*_TIMER_WAIT` 值也是如此。

+   `MIN_TIMER_WAIT`

    汇总定时事件的最小等待时间。

+   `AVG_TIMER_WAIT`

    汇总定时事件的平均等待时间。

+   `MAX_TIMER_WAIT`

    汇总定时事件的最大等待时间。

等待事件汇总表具有以下索引：

+   `events_waits_summary_by_account_by_event_name`:

    +   主键在 (`USER`, `HOST`, `EVENT_NAME`)

+   `events_waits_summary_by_host_by_event_name`:

    +   主键在 (`HOST`, `EVENT_NAME`)

+   `events_waits_summary_by_instance`:

    +   主键在 (`OBJECT_INSTANCE_BEGIN`)

    +   在 (`EVENT_NAME`) 上的索引

+   `events_waits_summary_by_thread_by_event_name`:

    +   主键在 (`THREAD_ID`, `EVENT_NAME`)

+   `events_waits_summary_by_user_by_event_name`:

    +   主键在 (`USER`, `EVENT_NAME`)

+   `events_waits_summary_global_by_event_name`:

    +   主键在 (`EVENT_NAME`)

`TRUNCATE TABLE` 允许用于等待汇总表。它具有以下效果：

+   对于不按帐户、主机或用户汇总的汇总表，截断会将汇总列重置为零，而不是删除行。

+   对于按帐户、主机或用户汇总的汇总表，截断会删除没有连接的帐户、主机或用户的行，并将剩余行的汇总列重置为零。

此外，每个按帐户、主机、用户或线程汇总的等待汇总表在其依赖的连接表被截断或 `events_waits_summary_global_by_event_name` 被截断时会被隐式截断。有关详细信息，请参见 第 29.12.8 节，“性能模式连接表”。
