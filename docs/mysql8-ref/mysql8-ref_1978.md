# 28.5.2 INFORMATION_SCHEMA TP_THREAD_GROUP_STATE 表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/information-schema-tp-thread-group-state-table.html`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-tp-thread-group-state-table.html)

注意

从 MySQL 8.0.14 开始，线程池`INFORMATION_SCHEMA`表也可以作为性能模式表使用。（参见 Section 29.12.16, “Performance Schema Thread Pool Tables”.）`INFORMATION_SCHEMA`表已被弃用；预计它们将在将来的 MySQL 版本中被移除。应用程序应该从旧表过渡到新表。例如，如果一个应用程序使用这个查询：

```sql
SELECT * FROM INFORMATION_SCHEMA.TP_THREAD_GROUP_STATE;
```

应用程序应该使用这个查询：

```sql
SELECT * FROM performance_schema.tp_thread_group_state;
```

`TP_THREAD_GROUP_STATE`表中每个线程组都有一行。每行提供有关组的当前状态的信息。

关于`INFORMATION_SCHEMA` `TP_THREAD_GROUP_STATE`表中列的描述，请参见 Section 29.12.16.1, “The tp_thread_group_state Table”. 性能模式`tp_thread_group_state`表具有相同的列。
