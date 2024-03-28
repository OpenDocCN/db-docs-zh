> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-ps-setup-show-enabled-consumers.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-ps-setup-show-enabled-consumers.html)

#### 30.4.4.19 ps_setup_show_enabled_consumers() 过程

显示当前所有已启用的性能模式消费者。

##### 参数

无。

##### 示例

```sql
mysql> CALL sys.ps_setup_show_enabled_consumers();
+-----------------------------+
| enabled_consumers           |
+-----------------------------+
| events_statements_current   |
| events_statements_history   |
| events_transactions_current |
| events_transactions_history |
| global_instrumentation      |
| statements_digest           |
| thread_instrumentation      |
+-----------------------------+
```
