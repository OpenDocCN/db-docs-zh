> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-ps-setup-enable-thread.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-ps-setup-enable-thread.html)

#### 30.4.4.11 `ps_setup_enable_thread()` 过程

给定连接 ID，为线程启用性能模式仪器。生成一个结果集，指示启用了多少个线程。已经启用的线程不计入其中。

##### 参数

+   `in_connection_id BIGINT`: 连接 ID。这是性能模式`threads`表中`PROCESSLIST_ID`列或`SHOW PROCESSLIST`输出中的`Id`列中给定类型的值。

##### 示例

通过连接 ID 启用特定连接：

```sql
mysql> CALL sys.ps_setup_enable_thread(225);
+------------------+
| summary          |
+------------------+
| Enabled 1 thread |
+------------------+
```

启用当前连接：

```sql
mysql> CALL sys.ps_setup_enable_thread(CONNECTION_ID());
+------------------+
| summary          |
+------------------+
| Enabled 1 thread |
+------------------+
```
