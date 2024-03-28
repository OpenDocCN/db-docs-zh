# 29.12.8 性能模式连接表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-connection-tables.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-connection-tables.html)

29.12.8.1 账户表

29.12.8.2 主机表

29.12.8.3 用户表

当客户端连接到 MySQL 服务器时，它会使用特定的用户名和特定的主机。性能模式提供关于这些连接的统计信息，分别跟踪每个账户（用户和主机组合）以及每个用户名和主机名，使用以下表：

+   `accounts`：每个客户端账户的连接统计

+   `hosts`：每个客户端主机名的连接统计

+   `users`：每个客户端用户名称的连接统计

连接表中“账户”的含义类似于 MySQL 授权表中`mysql`系统数据库中的含义，即该术语指的是用户和主机值的组合。它们的区别在于，对于授权表，账户的主机部分可以是模式，而对于性能模式表，主机值始终是特定的非模式主机名。

每个连接表都有`CURRENT_CONNECTIONS`和`TOTAL_CONNECTIONS`列，用于跟踪基于其统计数据的“跟踪值”上的当前连接数和总连接数。这些表在使用跟踪值方面有所不同。`accounts`表具有`USER`和`HOST`列，用于跟踪每个用户和主机组合的连接。`users`和`hosts`表分别具有`USER`和`HOST`列，用于跟踪每个用户名和主机名的连接。

性能模式还会统计内部线程和未能通过身份验证的用户会话线程，使用具有`USER`和`HOST`列值为`NULL`的行。

假设名为`user1`和`user2`的客户端分别从`hosta`和`hostb`连接一次。性能模式跟踪连接如下：

+   `accounts`表有四行，分别为`user1`/`hosta`、`user1`/`hostb`、`user2`/`hosta`和`user2`/`hostb`账户值，每行计算每个账户的一个连接。

+   `hosts`表有两行，分别为`hosta`和`hostb`，每行计算每个主机名的两个连接。

+   `users` 表有两行，分别为 `user1` 和 `user2`，每行计算每个用户名的两个连接。

当客户端连接时，性能模式确定每个连接表中适用的行，使用适用于每个表的跟踪值。如果没有这样的行，则添加一个。然后性能模式在该行中的 `CURRENT_CONNECTIONS` 和 `TOTAL_CONNECTIONS` 列中递增一。

当客户端断开连接时，性能模式将 `CURRENT_CONNECTIONS` 列中的值减一，并保持 `TOTAL_CONNECTIONS` 列不变。

`TRUNCATE TABLE` 允许连接表。它有以下效果：

+   对于没有当前连接的帐户、主机或用户（`CURRENT_CONNECTIONS = 0` 的行），将删除行。

+   未删除的行将被重置为仅计算当前连接数：对于 `CURRENT_CONNECTIONS > 0` 的行，`TOTAL_CONNECTIONS` 将被重置为 `CURRENT_CONNECTIONS`。

+   依赖连接表的汇总表将被隐式截断，如本节后面所述。

性能模式维护汇总表，按帐户、主机或用户聚合各种事件类型的连接统计信息。这些表的名称中包含 `_summary_by_account`、`_summary_by_host` 或 `_summary_by_user`。要识别它们，请使用以下查询：

```sql
mysql> SELECT TABLE_NAME FROM INFORMATION_SCHEMA.TABLES
       WHERE TABLE_SCHEMA = 'performance_schema'
       AND TABLE_NAME REGEXP '_summary_by_(account|host|user)'
       ORDER BY TABLE_NAME;
+------------------------------------------------------+
| TABLE_NAME                                           |
+------------------------------------------------------+
| events_errors_summary_by_account_by_error            |
| events_errors_summary_by_host_by_error               |
| events_errors_summary_by_user_by_error               |
| events_stages_summary_by_account_by_event_name       |
| events_stages_summary_by_host_by_event_name          |
| events_stages_summary_by_user_by_event_name          |
| events_statements_summary_by_account_by_event_name   |
| events_statements_summary_by_host_by_event_name      |
| events_statements_summary_by_user_by_event_name      |
| events_transactions_summary_by_account_by_event_name |
| events_transactions_summary_by_host_by_event_name    |
| events_transactions_summary_by_user_by_event_name    |
| events_waits_summary_by_account_by_event_name        |
| events_waits_summary_by_host_by_event_name           |
| events_waits_summary_by_user_by_event_name           |
| memory_summary_by_account_by_event_name              |
| memory_summary_by_host_by_event_name                 |
| memory_summary_by_user_by_event_name                 |
+------------------------------------------------------+
```

有关各个连接汇总表的详细信息，请参阅描述汇总事件类型表的部分：

+   等待事件汇总：第 29.12.20.1 节，“等待事件汇总表”

+   阶段事件汇总：第 29.12.20.2 节，“阶段汇总表”

+   语句事件汇总：第 29.12.20.3 节，“语句汇总表”

+   事务事件汇总：第 29.12.20.5 节，“事务汇总表”

+   内存事件汇总：第 29.12.20.10 节，“内存汇总表”

+   错误事件汇总：第 29.12.20.11 节，“错误汇总表”

允许对连接摘要表执行`TRUNCATE TABLE`。它会删除没有连接的帐户、主机或用户的行，并将剩余行的摘要列重置为零。此外，每个按帐户、主机、用户或线程汇总的摘要表在其依赖的连接表被截断时会被隐式截断。以下表格描述了连接表截断和隐式截断表之间的关系。

**表 29.2 连接表截断的隐式影响**

| 截断连接表 | 隐式截断的摘要表 |
| --- | --- |
| `accounts` | 表名包含`_summary_by_account`、`_summary_by_thread`的表 |
| `hosts` | 表名包含`_summary_by_account`、`_summary_by_host`、`_summary_by_thread`的表 |
| `users` | 表名包含`_summary_by_account`、`_summary_by_user`、`_summary_by_thread`的表 |

截断`_summary_global`摘要表也会隐式截断其对应的连接和线程摘要表。例如，截断`events_waits_summary_global_by_event_name`会隐式截断按帐户、主机、用户或线程汇总的等待事件摘要表。
