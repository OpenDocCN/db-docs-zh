> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-status-variable-summary-tables.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-status-variable-summary-tables.html)

#### 29.12.20.12 状态变量摘要表

Performance Schema 使状态变量信息在第 29.12.15 节“Performance Schema 状态变量表”中描述的表中可用。它还使聚合状态变量信息在此处描述的摘要表中可用。每个状态变量摘要表都有一个或多个分组列，用于指示表如何聚合状态值：

+   [`status_by_account`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-status-variable-summary-tables.html "29.12.20.12 状态变量摘要表")具有`USER`，`HOST`和`VARIABLE_NAME`列，用于按帐户总结状态变量。

+   [`status_by_host`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-status-variable-summary-tables.html "29.12.20.12 状态变量摘要表")具有`HOST`和`VARIABLE_NAME`列，用于按客户端连接的主机总结状态变量。

+   [`status_by_user`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-status-variable-summary-tables.html "29.12.20.12 状态变量摘要表")具有`USER`和`VARIABLE_NAME`列，用于按客户端用户名称总结状态变量。

每个状态变量摘要表都有包含聚合值的此摘要列：

+   `VARIABLE_VALUE`

    活��和终止会话的聚合状态变量值。

状态变量摘要表具有以下索引：

+   [`status_by_account`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-status-variable-summary-tables.html "29.12.20.12 状态变量摘要表"):

    +   主键为(`USER`, `HOST`, `VARIABLE_NAME`)

+   [`status_by_host`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-status-variable-summary-tables.html "29.12.20.12 状态变量摘要表"):

    +   主键为(`HOST`, `VARIABLE_NAME`)

+   [`status_by_user`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-status-variable-summary-tables.html "29.12.20.12 状态变量摘要表"):

    +   主键为(`USER`, `VARIABLE_NAME`)

这些表中“帐户”的含义类似于 MySQL 授予表中`mysql`系统数据库中的含义，即该术语指的是用户和主机值的组合。它们的区别在于，对于授予表，帐户的主机部分可以是模式，而对于 Performance Schema 表，主机值始终是特定的非模式主机名。

当会话终止时，会收集帐户状态。会话状态计数器将添加到全局状态计数器和相应的帐户状态计数器中。如果未收集帐户统计信息，则会话状态将添加到主机和用户状态中，如果已收集主机和用户状态。

如果分别将`performance_schema_accounts_size`、`performance_schema_hosts_size`和`performance_schema_users_size`系统变量设置为 0，则不会收集帐户、主机和用户统计信息。

性能模式支持以下状态变量摘要表的`TRUNCATE TABLE`；在所有情况下，活动会话的状态不受影响：

+   `status_by_account`: 将来自已终止会话的帐户状态聚合到用户和主机状态中，然后重置帐户状态。

+   `status_by_host`: 重置来自已终止会话的主机状态的聚合状态。

+   `status_by_user`: 重置来自已终止会话的用户状态。

`FLUSH STATUS` 将所有活动会话的会话状态添加到全局状态变量中，重置所有活动会话的状态，并重置从断开会话中聚合的帐户、主机和用户状态值。
