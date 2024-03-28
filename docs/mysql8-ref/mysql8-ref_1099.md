> 原文：[`dev.mysql.com/doc/refman/8.0/en/show-master-status.html`](https://dev.mysql.com/doc/refman/8.0/en/show-master-status.html)

#### 15.7.7.23 SHOW MASTER STATUS Statement

```sql
SHOW MASTER STATUS
```

这个语句提供了关于源服务器的二进制日志文件的状态信息。它需要`REPLICATION CLIENT`权限（或已弃用的`SUPER`权限）。

示例：

```sql
mysql> SHOW MASTER STATUS\G
*************************** 1\. row ***************************
             File: source-bin.000002
         Position: 1307
     Binlog_Do_DB: test
 Binlog_Ignore_DB: manual, mysql
Executed_Gtid_Set: 3E11FA47-71CA-11E1-9E33-C80AA9429562:1-5 1 row in set (0.00 sec)
```

当全局事务 ID 被使用时，`Executed_Gtid_Set`显示了在源服务器上已执行的事务的 GTID 集合。这与此服务器上的`gtid_executed`系统变量的值相同，以及在此服务器上`SHOW REPLICA STATUS`输出中的`Executed_Gtid_Set`的值（或在 MySQL 8.0.22 之前，在此服务器上`SHOW SLAVE STATUS`中的值）。
