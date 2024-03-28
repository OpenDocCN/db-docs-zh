> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-ps-is-instrument-default-enabled.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-ps-is-instrument-default-enabled.html)

#### 30.4.5.11 `ps_is_instrument_default_enabled()`函数

返回`YES`或`NO`，指示给定的性能模式工具是否默认启用。

##### 参数

+   `in_instrument VARCHAR(128)`: 要检查的工具的名称。

##### 返回值

一个`ENUM('YES','NO')`值。

##### 示例

```sql
mysql> SELECT sys.ps_is_instrument_default_enabled('memory/innodb/row_log_buf');
+-------------------------------------------------------------------+
| sys.ps_is_instrument_default_enabled('memory/innodb/row_log_buf') |
+-------------------------------------------------------------------+
| NO                                                                |
+-------------------------------------------------------------------+
mysql> SELECT sys.ps_is_instrument_default_enabled('statement/sql/alter_user');
+------------------------------------------------------------------+
| sys.ps_is_instrument_default_enabled('statement/sql/alter_user') |
+------------------------------------------------------------------+
| YES                                                              |
+------------------------------------------------------------------+
```
