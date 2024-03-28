# 28.5.4 `INFORMATION_SCHEMA TP_THREAD_STATE` 表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/information-schema-tp-thread-state-table.html`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-tp-thread-state-table.html)

注意

从 MySQL 8.0.14 开始，线程池 `INFORMATION_SCHEMA` 表也可以作为性能模式表使用。（参见 第 29.12.16 节，“性能模式线程池表”。）`INFORMATION_SCHEMA` 表已被弃用；预计它们将在未来的 MySQL 版本中被移除。应用程序应该从旧表过渡到新表。例如，如果一个应用程序使用以下查询：

```sql
SELECT * FROM INFORMATION_SCHEMA.TP_THREAD_STATE;
```

应用程序应该使用以下查询：

```sql
SELECT * FROM performance_schema.tp_thread_state;
```

`TP_THREAD_STATE` 表每个由线程池创建的用于处理连接的线程都有一行。

要了解 `INFORMATION_SCHEMA TP_THREAD_STATE` 表中的列的描述，请参阅 第 29.12.16.3 节，“tp_thread_state 表”。性能模式 `tp_thread_state` 表具有相同的列。
