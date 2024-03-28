> [`dev.mysql.com/doc/refman/8.0/en/sys-ps-setup-show-disabled-instruments.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-ps-setup-show-disabled-instruments.html)

#### 30.4.4.17 `ps_setup_show_disabled_instruments()` 过程

显示当前所有禁用的性能模式仪器。这可能是一个很长的列表。

##### 参数

无。

##### 示例

```sql
mysql> CALL sys.ps_setup_show_disabled_instruments()\G
*************************** 1\. row ***************************
disabled_instruments: wait/synch/mutex/sql/TC_LOG_MMAP::LOCK_tc
               timed: NO
*************************** 2\. row ***************************
disabled_instruments: wait/synch/mutex/sql/THD::LOCK_query_plan
               timed: NO
*************************** 3\. row ***************************
disabled_instruments: wait/synch/mutex/sql/MYSQL_BIN_LOG::LOCK_commit
               timed: NO
...
```
