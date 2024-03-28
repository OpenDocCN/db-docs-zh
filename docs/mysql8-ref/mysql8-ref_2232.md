> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-ps-thread-id.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-ps-thread-id.html)

#### 30.4.5.15 `ps_thread_id()`函数

注意

截至 MySQL 8.0.16，`ps_thread_id()` Function")已被弃用，并将在未来的 MySQL 版本中移除。使用它的应用程序应该迁移到使用内置的`PS_THREAD_ID()`和`PS_CURRENT_THREAD_ID()`函数。参见第 14.21 节，“性能模式函数”

返回给定连接 ID 分配的性能模式线程 ID，如果连接 ID 为`NULL`，则返回当前连接的线程 ID。

##### 参数

+   `in_connection_id BIGINT UNSIGNED`: 要返回线程 ID 的连接的 ID。这是性能模式`threads`表中`PROCESSLIST_ID`列中给定类型的值，或`SHOW PROCESSLIST`输出中的`Id`列。

##### 返回值

一个`BIGINT UNSIGNED`值。

##### 示例

```sql
mysql> SELECT sys.ps_thread_id(260);
+-----------------------+
| sys.ps_thread_id(260) |
+-----------------------+
|                   285 |
+-----------------------+
```
