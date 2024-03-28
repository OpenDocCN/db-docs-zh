> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-setup-consumers-table.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-setup-consumers-table.html)

#### 29.12.2.2 The setup_consumers Table

`setup_consumers` 表列出了可以存储事件信息并启用的消费者类型：

```sql
mysql> SELECT * FROM performance_schema.setup_consumers;
+----------------------------------+---------+
| NAME                             | ENABLED |
+----------------------------------+---------+
| events_stages_current            | NO      |
| events_stages_history            | NO      |
| events_stages_history_long       | NO      |
| events_statements_current        | YES     |
| events_statements_history        | YES     |
| events_statements_history_long   | NO      |
| events_transactions_current      | YES     |
| events_transactions_history      | YES     |
| events_transactions_history_long | NO      |
| events_waits_current             | NO      |
| events_waits_history             | NO      |
| events_waits_history_long        | NO      |
| global_instrumentation           | YES     |
| thread_instrumentation           | YES     |
| statements_digest                | YES     |
+----------------------------------+---------+
```

`setup_consumers` 表中的消费者设置形成了从高级到低级的层次结构。有关启用不同消费者的影响的详细信息，请参见 Section 29.4.7, “Pre-Filtering by Consumer”。

对 `setup_consumers` 表的修改立即影响监视。

`setup_consumers` 表具有以下列：

+   `NAME`

    消费者名称。

+   `ENABLED`

    指示消费者是否已启用。值为 `YES` 或 `NO`。此列可修改。如果禁用消费者，则服务器不会花时间将事件信息添加到其中。

`setup_consumers` 表具有以下索引：

+   主键为 (`NAME`)

不允许对 `setup_consumers` 表执行 `TRUNCATE TABLE`。
