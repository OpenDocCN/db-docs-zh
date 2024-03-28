> 原文：[`dev.mysql.com/doc/refman/8.0/en/clone-plugin-stop.html`](https://dev.mysql.com/doc/refman/8.0/en/clone-plugin-stop.html)

#### 7.6.7.11 停止克隆操作

如有必要，您可以使用`KILL QUERY *`processlist_id`*`语句停止克隆操作。

在接收端的 MySQL 服务器实例中，您可以从`clone_status`表的`PID`列中检索克隆操作的进程列表标识符（PID）。

```sql
mysql> SELECT * FROM performance_schema.clone_status\G
*************************** 1\. row ***************************
             ID: 1
            PID: 8
          STATE: In Progress
     BEGIN_TIME: 2019-07-15 11:58:36.767
       END_TIME: NULL
         SOURCE: LOCAL INSTANCE
    DESTINATION: /*path/to/clone_dir*/
       ERROR_NO: 0
  ERROR_MESSAGE:
    BINLOG_FILE:
BINLOG_POSITION: 0
  GTID_EXECUTED:
```

您还可以从`INFORMATION_SCHEMA` `PROCESSLIST`表的`ID`列，`SHOW PROCESSLIST`输出的`Id`列，或性能模式`threads`表的`PROCESSLIST_ID`列中检索进程列表标识符。这些获取 PID 信息的方法可用于捐赠者或接收者的 MySQL 服务器实例。
