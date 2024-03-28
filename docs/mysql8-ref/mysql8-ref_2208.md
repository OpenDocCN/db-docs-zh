> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-ps-setup-show-enabled.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-ps-setup-show-enabled.html)

#### 30.4.4.18 `ps_setup_show_enabled()` 过程

显示当前所有已启用的性能模式配置。

##### 参数

+   `in_show_instruments BOOLEAN`: 是否显示已启用的仪器。这可能是一个很长的列表。

+   `in_show_threads BOOLEAN`: 是否显示已启用的线程。

##### 示例

```sql
mysql> CALL sys.ps_setup_show_enabled(FALSE, FALSE);
+----------------------------+
| performance_schema_enabled |
+----------------------------+
|                          1 |
+----------------------------+
1 row in set (0.01 sec)

+---------------+
| enabled_users |
+---------------+
| '%'@'%'       |
+---------------+
1 row in set (0.01 sec)

+-------------+---------+---------+-------+
| object_type | objects | enabled | timed |
+-------------+---------+---------+-------+
| EVENT       | %.%     | YES     | YES   |
| FUNCTION    | %.%     | YES     | YES   |
| PROCEDURE   | %.%     | YES     | YES   |
| TABLE       | %.%     | YES     | YES   |
| TRIGGER     | %.%     | YES     | YES   |
+-------------+---------+---------+-------+
5 rows in set (0.02 sec)

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
