> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-transaction-summary-tables.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-transaction-summary-tables.html)

#### 29.12.20.5 事务摘要表

Performance Schema 维护用于收集当前和最近事务事件的表，并在摘要表中聚合该信息。第 29.12.7 节，“Performance Schema 事务表”描述了事务摘要的基础事件。请参阅该讨论以获取有关事务事件内容、当前和历史事务事件表以及如何控制事务事件收集（默认情况下已禁用）的信息。

事务事件摘要信息示例：

```sql
mysql> SELECT *
       FROM performance_schema.events_transactions_summary_global_by_event_name
       LIMIT 1\G
*************************** 1\. row ***************************
          EVENT_NAME: transaction
          COUNT_STAR: 5
      SUM_TIMER_WAIT: 19550092000
      MIN_TIMER_WAIT: 2954148000
      AVG_TIMER_WAIT: 3910018000
      MAX_TIMER_WAIT: 5486275000
    COUNT_READ_WRITE: 5
SUM_TIMER_READ_WRITE: 19550092000
MIN_TIMER_READ_WRITE: 2954148000
AVG_TIMER_READ_WRITE: 3910018000
MAX_TIMER_READ_WRITE: 5486275000
     COUNT_READ_ONLY: 0
 SUM_TIMER_READ_ONLY: 0
 MIN_TIMER_READ_ONLY: 0
 AVG_TIMER_READ_ONLY: 0
 MAX_TIMER_READ_ONLY: 0
```

每个事务摘要表都有一个或多个分组列，用于指示表如何聚合事件。事件名称指的是`setup_instruments`表中事件仪器的名称：

+   `events_transactions_summary_by_account_by_event_name`具有`USER`、`HOST`和`EVENT_NAME`列。每行总结了给定帐户（用户和主机组合）和事件名称的事件。

+   `events_transactions_summary_by_host_by_event_name`具有`HOST`和`EVENT_NAME`列。每行总结了给定主机和事件名称的事件。

+   `events_transactions_summary_by_thread_by_event_name`具有`THREAD_ID`和`EVENT_NAME`列。每行总结了给定线程和事件名称的事件。

+   `events_transactions_summary_by_user_by_event_name`具有`USER`和`EVENT_NAME`列。每行总结了给定用户和事件名称的事件。

+   `events_transactions_summary_global_by_event_name`具有一个`EVENT_NAME`列。每行总结了给定事件名称的事件。

每个事务摘要表都有这些包含聚合值的摘要列：

+   `COUNT_STAR`, `SUM_TIMER_WAIT`, `MIN_TIMER_WAIT`, `AVG_TIMER_WAIT`, `MAX_TIMER_WAIT`

    这些列类似于等待事件摘要表中同名列的列（参见第 29.12.20.1 节，“等待事件摘要表”），只是交易摘要表聚合了来自`events_transactions_current`而不是`events_waits_current`的事件。这些列总结了读写和只读事务。

+   `COUNT_READ_WRITE`, `SUM_TIMER_READ_WRITE`, `MIN_TIMER_READ_WRITE`, `AVG_TIMER_READ_WRITE`, `MAX_TIMER_READ_WRITE`

    这些类似于`COUNT_STAR`和`*`xxx`*_TIMER_WAIT`列，但仅总结了读写事务。事务访问模式指定事务是以读/写模式还是只读模式运行。

+   `COUNT_READ_ONLY`, `SUM_TIMER_READ_ONLY`, `MIN_TIMER_READ_ONLY`, `AVG_TIMER_READ_ONLY`, `MAX_TIMER_READ_ONLY`

    这些类似于`COUNT_STAR`和`*`xxx`*_TIMER_WAIT`列，但仅总结了只读事务。事务访问模式指定事务是以读/写模式还是只读模式运行。

交易摘要表具有以下索引：

+   `events_transactions_summary_by_account_by_event_name`:

    +   主键为(`USER`, `HOST`, `EVENT_NAME`)

+   `events_transactions_summary_by_host_by_event_name`:

    +   主键为(`HOST`, `EVENT_NAME`)

+   `events_transactions_summary_by_thread_by_event_name`:

    +   主键为(`THREAD_ID`, `EVENT_NAME`)

+   `events_transactions_summary_by_user_by_event_name`:

    +   主键为(`USER`, `EVENT_NAME`)

+   `events_transactions_summary_global_by_event_name`:

    +   主键为(`EVENT_NAME`)

`TRUNCATE TABLE`允许用于交易摘要表。它具有以下效果：

+   对于未按帐户、主机或用户聚合的摘要表，截断将摘要列重置为零，而不是删除行。

+   对于按帐户、主机或用户聚合的摘要表，截断将删除没有连接的帐户、主机或用户的行，并将剩余行的摘要列重置为零。

此外，按帐户、主机、用户或线程聚合的每个事务摘要表都会隐式地通过其依赖的连接表或`events_transactions_summary_global_by_event_name`的截断进行截断。详情请参见第 29.12.8 节“性能模式连接表”。

##### 事务聚合规则

事务事件收集不考虑隔离级别、访问模式或自动提交模式。

服务器发起的所有非中止事务，包括空事务，都会进行事务事件收集。

读写事务通常比只读事务更消耗资源，因此事务摘要表包括独立的读写和只读事务的聚合列。

资源需求也可能随着事务隔离级别的不同而变化。然而，假设每台服务器只使用一个隔离级别，就不提供按隔离级别聚合的功能。
