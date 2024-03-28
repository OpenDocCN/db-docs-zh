> [`dev.mysql.com/doc/refman/8.0/en/performance-schema-error-summary-tables.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-error-summary-tables.html)

#### 29.12.20.11 错误摘要表

性能模式维护了用于聚合关于服务器错误（和警告）的统计信息的摘要表。有关服务器错误的列表，请参见服务器错误消息参考。

错误信息的收集由默认启用的 `error` 仪器控制。不收集时间信息。

每个错误摘要表都有三列用于标识错误：

+   `ERROR_NUMBER` 是数字错误值。该值是唯一的。

+   `ERROR_NAME` 是与 `ERROR_NUMBER` 值对应的符号错误名称。该值是唯一的。

+   `SQLSTATE` 是与 `ERROR_NUMBER` 值对应的 SQLSTATE 值。该值不一定是唯一的。

例如，如果 `ERROR_NUMBER` 是 1050，`ERROR_NAME` 是 `ER_TABLE_EXISTS_ERROR`，`SQLSTATE` 是 `42S01`。

示例错误事件摘要信息：

```sql
mysql> SELECT *
       FROM performance_schema.events_errors_summary_global_by_error
       WHERE SUM_ERROR_RAISED <> 0\G
*************************** 1\. row ***************************
     ERROR_NUMBER: 1064
       ERROR_NAME: ER_PARSE_ERROR
        SQL_STATE: 42000
 SUM_ERROR_RAISED: 1
SUM_ERROR_HANDLED: 0
       FIRST_SEEN: 2016-06-28 07:34:02
        LAST_SEEN: 2016-06-28 07:34:02
*************************** 2\. row ***************************
     ERROR_NUMBER: 1146
       ERROR_NAME: ER_NO_SUCH_TABLE
        SQL_STATE: 42S02
 SUM_ERROR_RAISED: 2
SUM_ERROR_HANDLED: 0
       FIRST_SEEN: 2016-06-28 07:34:05
        LAST_SEEN: 2016-06-28 07:36:18
*************************** 3\. row ***************************
     ERROR_NUMBER: 1317
       ERROR_NAME: ER_QUERY_INTERRUPTED
        SQL_STATE: 70100
 SUM_ERROR_RAISED: 1
SUM_ERROR_HANDLED: 0
       FIRST_SEEN: 2016-06-28 11:01:49
        LAST_SEEN: 2016-06-28 11:01:49
```

每个错误摘要表都有一个或多个分组列，指示表如何聚合错误：

+   `events_errors_summary_by_account_by_error` 有 `USER`、`HOST` 和 `ERROR_NUMBER` 列。每行总结了给定帐户（用户和主机组合）和错误的事件。

+   `events_errors_summary_by_host_by_error` 有 `HOST` 和 `ERROR_NUMBER` 列。每行总结了给定主机和错误的事件。

+   `events_errors_summary_by_thread_by_error` 有 `THREAD_ID` 和 `ERROR_NUMBER` 列。每行总结了给定线程和错误的事件。

+   `events_errors_summary_by_user_by_error` 有 `USER` 和 `ERROR_NUMBER` 列。每行总结了给定用户和错误的事件。

+   `events_errors_summary_global_by_error` 有一个 `ERROR_NUMBER` 列。每行总结了给定错误的事件。

每个错误摘要表都有包含聚合值的这些摘要列：

+   `SUM_ERROR_RAISED`

    此列聚合错误发生的次数。

+   `SUM_ERROR_HANDLED`

    此列聚合了通过 SQL 异常处理程序处理的错误次数。

+   `FIRST_SEEN`、`LAST_SEEN`

    时间戳指示错误首次出现和最近出现的时间。

每个错误摘要表中的`NULL`行用于聚合超出仪表化错误范围的所有错误的统计信息。例如，如果 MySQL Server 错误范围从*`M`*到*`N`*，并且引发了一个不在该范围内的编号为*`Q`*的错误，则该错误将在`NULL`行中聚合。`NULL`行是具有`ERROR_NUMBER=0`，`ERROR_NAME=NULL`和`SQLSTATE=NULL`的行。

错误摘要表具有以下索引：

+   `events_errors_summary_by_account_by_error`:

    +   主键为 (`USER`, `HOST`, `ERROR_NUMBER`)

+   `events_errors_summary_by_host_by_error`:

    +   主键为 (`HOST`, `ERROR_NUMBER`)

+   `events_errors_summary_by_thread_by_error`:

    +   主键为 (`THREAD_ID`, `ERROR_NUMBER`)

+   `events_errors_summary_by_user_by_error`:

    +   主键为 (`USER`, `ERROR_NUMBER`)

+   `events_errors_summary_global_by_error`:

    +   主键为 (`ERROR_NUMBER`)

允许对错误摘要表执行`TRUNCATE TABLE`。它具有以下效果：

+   对于未按帐户、主机或用户聚合的摘要表，截断将摘要列重置为零或`NULL`，而不是删除行。

+   对于按帐户、主机或用户聚合的摘要表，截断将删除没有连接的帐户、主机或用户的行，并将剩余行的摘要列重置为零或`NULL`。

此外，通过截断依赖的连接表或截断`events_errors_summary_global_by_error`来隐式截断每个按帐户、主机、用户或线程聚合的错误摘要表。有关详细信息，请参见第 29.12.8 节“性能模式连接表”。
