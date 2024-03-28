> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-ps-setup-show-enabled-instruments.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-ps-setup-show-enabled-instruments.html)

#### 30.4.4.20 `ps_setup_show_enabled_instruments()` 过程

显示当前启用的所有性能模式仪器。这可能是一个很长的列表。

##### 参数

无。

##### 示例

```sql
mysql> CALL sys.ps_setup_show_enabled_instruments()\G
*************************** 1\. row ***************************
enabled_instruments: wait/io/file/sql/map
              timed: YES
*************************** 2\. row ***************************
enabled_instruments: wait/io/file/sql/binlog
              timed: YES
*************************** 3\. row ***************************
enabled_instruments: wait/io/file/sql/binlog_cache
              timed: YES
...
```
