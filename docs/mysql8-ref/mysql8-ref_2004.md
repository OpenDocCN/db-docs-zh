# 29.5 性能模式查询

> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-queries.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-queries.html)

预过滤限制了收集的事件信息，与任何特定用户无关。相比之下，后过滤是由个别用户通过使用带有适当`WHERE`子句的查询执行的，这些子句限制了在应用预过滤后可用的事件中选择哪些事件信息。

在第 29.4.3 节，“事件预过滤”中，一个示例展示了如何为文件工具进行预过滤。如果事件表中既包含文件信息又包含非文件信息，后过滤是另一种仅查看文件事件信息的方法。向查询添加`WHERE`子句以适当限制事件选择：

```sql
mysql> SELECT THREAD_ID, NUMBER_OF_BYTES
       FROM performance_schema.events_waits_history
       WHERE EVENT_NAME LIKE 'wait/io/file/%'
       AND NUMBER_OF_BYTES IS NOT NULL;
+-----------+-----------------+
| THREAD_ID | NUMBER_OF_BYTES |
+-----------+-----------------+
|        11 |              66 |
|        11 |              47 |
|        11 |             139 |
|         5 |              24 |
|         5 |             834 |
+-----------+-----------------+
```

大多数性能模式表都有索引，这使得优化器可以访问除全表扫描之外的执行计划。这些索引还提高了与使用这些表的`sys`模式视图等相关对象的性能。有关更多信息，请参见第 10.2.4 节，“优化性能模式查询”。
