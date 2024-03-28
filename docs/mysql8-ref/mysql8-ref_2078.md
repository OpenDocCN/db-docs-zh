# 29.12.15 性能模式状态变量表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-status-variable-tables.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-status-variable-tables.html)

MySQL 服务器维护许多状态变量，提供关于其操作的信息（请参阅第 7.1.10 节，“服务器状态变量”）。状态变量信息在这些性能模式表中可用：

+   `global_status`：全局状态变量。只想要全局值的应用程序应该使用这个表。

+   `session_status`：当前会话的状态变量。想要获取自己会话的所有状态变量值的应用程序应该使用这个表。它包括会话变量以及没有会话对应的全局变量的值。

+   `status_by_thread`：每个活动会话的会话状态变量。想要了解特定会话的会话变量值的应用程序应该使用这个表。它仅包含会话变量，通过线程 ID 进行标识。

还有汇总表，提供按帐户、主机名和用户名聚合的状态变量信息。请参阅第 29.12.20.12 节，“状态变量汇总表”。

会话变量表(`session_status`, `status_by_thread`)仅包含活动会话的信息，而不包括已终止的会话。

性能模式仅为`threads`中`INSTRUMENTED`值为`YES`的线程收集全局状态变量的统计信息。会话状态变量的统计信息始终会被收集，不受`INSTRUMENTED`值的影响。

性能模式不会在状态变量表中收集`Com_*`xxx`*`状态变量的统计信息。要获取全局和每个会话的语句执行计数，请分别使用`events_statements_summary_global_by_event_name`和`events_statements_summary_by_thread_by_event_name`表。例如：

```sql
SELECT EVENT_NAME, COUNT_STAR
FROM performance_schema.events_statements_summary_global_by_event_name
WHERE EVENT_NAME LIKE 'statement/sql/%';
```

`global_status`和`session_status`表具有以下列：

+   `VARIABLE_NAME`

    状态变量名称。

+   `VARIABLE_VALUE`

    状态变量值。对于`global_status`，此列包含全局值。对于`session_status`，此列包含当前会话的变量值。

`global_status`和`session_status`表具有以下索引：

+   (`VARIABLE_NAME`)上的主键

`status_by_thread`表包含每个活动线程的状态。它有以下列：

+   `THREAD_ID`

    定义状态变量的会话的线程标识符。

+   `VARIABLE_NAME`

    状态变量名称。

+   `VARIABLE_VALUE`

    由`THREAD_ID`列命名的会话的会话变量值。

`status_by_thread`表具有以下索引：

+   (`THREAD_ID`, `VARIABLE_NAME`)上的主键

`status_by_thread`表仅包含关于前台线程的状态变量信息。如果`performance_schema_max_thread_instances`系统变量不是自动缩放（值为-1）且允许的最大仪表化线程对象数不大于后台线程数，则表为空。

性能模式支持`TRUNCATE TABLE`用于状态变量表如下：

+   `global_status`：重置线程、账户、主机和用户状态。重置服务器从不重置的全局状态变量。

+   `session_status`：不支持。

+   `status_by_thread`：将所有线程的状态聚合到全局状态和账户状态中，然后重置线程状态。如果未收集账户统计信息，则将会话状态添加到主机和用户状态中，如果已收集主机和用户状态。

    如果分别将系统变量`performance_schema_accounts_size`、`performance_schema_hosts_size`和`performance_schema_users_size`设置为 0，则不会收集账户、主机和用户统计信息。

`FLUSH STATUS` 将所有活动会话的状态添加到全局状态变量中，重置所有活动会话的状态，并重置从断开的会话中聚合的账户、主机和用户状态值。
