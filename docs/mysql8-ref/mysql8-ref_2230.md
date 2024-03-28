> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-ps-is-thread-instrumented.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-ps-is-thread-instrumented.html)

#### 30.4.5.13 `ps_is_thread_instrumented()`函数

返回`YES`或`NO`以指示给定连接 ID 的性能模式仪器是否启用，如果 ID 未知则返回`UNKNOWN`，如果 ID 为`NULL`则返回`NULL`。

##### 参数

+   `in_connection_id BIGINT UNSIGNED`：连接 ID。这是性能模式`threads`表中`PROCESSLIST_ID`列或`SHOW PROCESSLIST`输出中`Id`列中给定类型的值。

##### 返回值

一个`ENUM('YES','NO','UNKNOWN')`值。

##### 示例

```sql
mysql> SELECT sys.ps_is_thread_instrumented(43);
+-----------------------------------+
| sys.ps_is_thread_instrumented(43) |
+-----------------------------------+
| UNKNOWN                           |
+-----------------------------------+
mysql> SELECT sys.ps_is_thread_instrumented(CONNECTION_ID());
+------------------------------------------------+
| sys.ps_is_thread_instrumented(CONNECTION_ID()) |
+------------------------------------------------+
| YES                                            |
+------------------------------------------------+
```
