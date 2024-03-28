> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-ps-is-consumer-enabled.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-ps-is-consumer-enabled.html)

#### 30.4.5.10 `ps_is_consumer_enabled()` 函数

返回`YES`或`NO`以指示给定的性能模式消费者是否已启用，如果参数为`NULL`，则返回`NULL`。如果参数不是有效的消费者名称，则会发生错误。（在 MySQL 8.0.18 之前，如果参数不是有效的消费者名称，则函数将返回`NULL`。）

该函数考虑了消费者层次结构，因此除非所有依赖的消费者也处于启用状态，否则不会被视为已启用。有关消费者层次结构的信息，请参见第 29.4.7 节，“按消费者进行预过滤”。

##### 参数

+   `in_consumer VARCHAR(64)`: 要检查的消费者的名称。

##### 返回值

一个`ENUM('YES','NO')`值。

##### 示例

```sql
mysql> SELECT sys.ps_is_consumer_enabled('thread_instrumentation');
+------------------------------------------------------+
| sys.ps_is_consumer_enabled('thread_instrumentation') |
+------------------------------------------------------+
| YES                                                  |
+------------------------------------------------------+
```
