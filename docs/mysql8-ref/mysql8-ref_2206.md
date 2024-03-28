> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-ps-setup-show-disabled-consumers.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-ps-setup-show-disabled-consumers.html)

#### 30.4.4.16 ps_setup_show_disabled_consumers() 过程

显示当前所有已禁用的性能模式消费者。

##### 参数

无。

##### 示例

```sql
mysql> CALL sys.ps_setup_show_disabled_consumers();
+----------------------------------+
| disabled_consumers               |
+----------------------------------+
| events_stages_current            |
| events_stages_history            |
| events_stages_history_long       |
| events_statements_history        |
| events_statements_history_long   |
| events_transactions_history      |
| events_transactions_history_long |
| events_waits_current             |
| events_waits_history             |
| events_waits_history_long        |
+----------------------------------+
```
