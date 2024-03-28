> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-ps-thread-account.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-ps-thread-account.html)

#### 30.4.5.14 `ps_thread_account()`函数

给定一个性能模式线程 ID，返回与该线程关联的`*`user_name`*@*`host_name`*`账户。

##### 参数

+   `in_thread_id BIGINT UNSIGNED`：要返回账户的线程 ID。该值应与某些性能模式`threads`表行的`THREAD_ID`列匹配。

##### 返回值

一个`TEXT`值。

##### 示例

```sql
mysql> SELECT sys.ps_thread_account(sys.ps_thread_id(CONNECTION_ID()));
+----------------------------------------------------------+
| sys.ps_thread_account(sys.ps_thread_id(CONNECTION_ID())) |
+----------------------------------------------------------+
| root@localhost                                           |
+----------------------------------------------------------+
```
