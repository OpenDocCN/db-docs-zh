> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-ps-setup-disable-thread.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-ps-setup-disable-thread.html)

#### 30.4.4.7 `ps_setup_disable_thread()` 过程

给定连接 ID，禁用线程的性能模式仪表。生成一个结果集，指示禁用了多少个线程。已禁用的线程不计入其中。

##### 参数

+   `in_connection_id BIGINT`：连接 ID。这是性能模式 `threads` 表中 `PROCESSLIST_ID` 列或 `SHOW PROCESSLIST` 输出中 `Id` 列中给定类型的值。

##### 示例

通过连接 ID 禁用特定连接：

```sql
mysql> CALL sys.ps_setup_disable_thread(225);
+-------------------+
| summary           |
+-------------------+
| Disabled 1 thread |
+-------------------+
```

禁用当前连接：

```sql
mysql> CALL sys.ps_setup_disable_thread(CONNECTION_ID());
+-------------------+
| summary           |
+-------------------+
| Disabled 1 thread |
+-------------------+
```
