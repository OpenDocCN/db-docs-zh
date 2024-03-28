# 28.5.3 INFORMATION_SCHEMA TP_THREAD_GROUP_STATS 表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/information-schema-tp-thread-group-stats-table.html`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-tp-thread-group-stats-table.html)

注意

从 MySQL 8.0.14 开始，线程池`INFORMATION_SCHEMA`表也可以作为性能模式表使用。（参见 Section 29.12.16, “性能模式线程池表”.）`INFORMATION_SCHEMA`表已被弃用；预计它们将在 MySQL 的未来版本中被移除。应用程序应该从旧表过渡到新表。例如，如果一个应用程序使用这个查询：

```sql
SELECT * FROM INFORMATION_SCHEMA.TP_THREAD_GROUP_STATS;
```

应用程序应该改用这个查询：

```sql
SELECT * FROM performance_schema.tp_thread_group_stats;
```

`TP_THREAD_GROUP_STATS`表报告每个线程组的统计信息。每个组有一行。

要了解`INFORMATION_SCHEMA` `TP_THREAD_GROUP_STATS`表中的列的描述，请参阅 Section 29.12.16.2, “tp_thread_group_stats 表”.性能模式`tp_thread_group_stats`表具有相同的列。
