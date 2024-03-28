> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-ps-setup-show-disabled.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-ps-setup-show-disabled.html)

#### 30.4.4.15 `ps_setup_show_disabled()` 过程

显示当前所有已禁用的性能模式配置。

##### 参数

+   `in_show_instruments BOOLEAN`: 是否显示已禁用的工具。这可能是一个很长的列表。

+   `in_show_threads BOOLEAN`: 是否显示已禁用的线程。

##### 示例

```sql
mysql> CALL sys.ps_setup_show_disabled(TRUE, TRUE);
+----------------------------+
| performance_schema_enabled |
+----------------------------+
|                          1 |
+----------------------------+

+---------------+
| enabled_users |
+---------------+
| '%'@'%'       |
+---------------+

+-------------+----------------------+---------+-------+
| object_type | objects              | enabled | timed |
+-------------+----------------------+---------+-------+
| EVENT       | mysql.%              | NO      | NO    |
| EVENT       | performance_schema.% | NO      | NO    |
| EVENT       | information_schema.% | NO      | NO    |
| FUNCTION    | mysql.%              | NO      | NO    |
| FUNCTION    | performance_schema.% | NO      | NO    |
| FUNCTION    | information_schema.% | NO      | NO    |
| PROCEDURE   | mysql.%              | NO      | NO    |
| PROCEDURE   | performance_schema.% | NO      | NO    |
| PROCEDURE   | information_schema.% | NO      | NO    |
| TABLE       | mysql.%              | NO      | NO    |
| TABLE       | performance_schema.% | NO      | NO    |
| TABLE       | information_schema.% | NO      | NO    |
| TRIGGER     | mysql.%              | NO      | NO    |
| TRIGGER     | performance_schema.% | NO      | NO    |
| TRIGGER     | information_schema.% | NO      | NO    |
+-------------+----------------------+---------+-------+

...
```
