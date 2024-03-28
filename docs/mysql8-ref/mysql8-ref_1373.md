> 原文：[`dev.mysql.com/doc/refman/8.0/en/replication-administration-status.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-administration-status.html)

#### 19.1.7.1 检查复制状态

管理复制过程时最常见的任务是确保复制正在进行，并且源端和副本之间没有错误发生。

您必须在每个副本上执行的`SHOW REPLICA STATUS`语句提供有关副本服务器和源服务器之间连接的配置和状态的信息。从 MySQL 8.0.22 开始，`SHOW SLAVE STATUS`已被弃用，可以使用`SHOW REPLICA STATUS`代替。性能模式具有提供此信息的复制表，以更易于访问的形式呈现。请参阅第 29.12.11 节，“性能模式复制表”。

在性能模式复制表中显示的复制心跳信息可以让您检查复制连接是否活动，即使源端最近没有向副本发送事件。如果二进制日志中的更新或未发送事件的时间超过心跳间隔时间，源端会向副本发送心跳信号。源端上的`MASTER_HEARTBEAT_PERIOD`设置（由`CHANGE MASTER TO`语句设置）指定了心跳的频率，默认为副本的连接超时时间的一半（由系统变量`replica_net_timeout`或`slave_net_timeout`指定）。性能模式表`replication_connection_status`显示了副本最近接收到心跳信号的时间，以及接收到的心跳信号数量。

如果您使用`SHOW REPLICA STATUS`语句来检查单个副本的状态，则该语句提供以下信息：

```sql
mysql> SHOW REPLICA STATUS\G
*************************** 1\. row ***************************
             Replica_IO_State: Waiting for source to send event
                  Source_Host: 127.0.0.1
                  Source_User: root
                  Source_Port: 13000
                Connect_Retry: 1
              Source_Log_File: master-bin.000001
          Read_Source_Log_Pos: 927
               Relay_Log_File: slave-relay-bin.000002
                Relay_Log_Pos: 1145
        Relay_Source_Log_File: master-bin.000001
           Replica_IO_Running: Yes
          Replica_SQL_Running: Yes
              Replicate_Do_DB:
          Replicate_Ignore_DB:
           Replicate_Do_Table:
       Replicate_Ignore_Table:
      Replicate_Wild_Do_Table:
  Replicate_Wild_Ignore_Table:
                   Last_Errno: 0
                   Last_Error:
                 Skip_Counter: 0
          Exec_Source_Log_Pos: 927
              Relay_Log_Space: 1355
              Until_Condition: None
               Until_Log_File:
                Until_Log_Pos: 0
           Source_SSL_Allowed: No
           Source_SSL_CA_File:
           Source_SSL_CA_Path:
              Source_SSL_Cert:
            Source_SSL_Cipher:
               Source_SSL_Key:
        Seconds_Behind_Source: 0
Source_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error:
               Last_SQL_Errno: 0
               Last_SQL_Error:
  Replicate_Ignore_Server_Ids:
             Source_Server_Id: 1
                  Source_UUID: 73f86016-978b-11ee-ade5-8d2a2a562feb
             Source_Info_File: mysql.slave_master_info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
    Replica_SQL_Running_State: Replica has read all relay log; waiting for more updates
           Source_Retry_Count: 10
                  Source_Bind:
      Last_IO_Error_Timestamp:
     Last_SQL_Error_Timestamp:
               Source_SSL_Crl:
           Source_SSL_Crlpath:
           Retrieved_Gtid_Set: 73f86016-978b-11ee-ade5-8d2a2a562feb:1-3
            Executed_Gtid_Set: 73f86016-978b-11ee-ade5-8d2a2a562feb:1-3
                Auto_Position: 1
         Replicate_Rewrite_DB:
                 Channel_Name:
           Source_TLS_Version:
       Source_public_key_path:
        Get_Source_public_key: 0
            Network_Namespace:
```

要检查状态报告中的关键字段，请查看：

+   `Replica_IO_State`：副本的当前状态。有关更多信息，请参阅第 10.14.5 节，“复制 I/O（接收器）线程状态” Thread States")和第 10.14.6 节，“复制 SQL 线程状态”。

+   `Replica_IO_Running`：用于读取源头二进制日志的 I/O（接收器）线程是否正在运行。通常情况下，你希望这个值为 `Yes`，除非你尚未开始复制或已经使用 `STOP REPLICA` 明确停止了复制。

+   `Replica_SQL_Running`：用于执行中继日志中事件的 SQL 线程是否正在运行。与 I/O 线程一样，这通常应该是 `Yes`。

+   `Last_IO_Error`、`Last_SQL_Error`：处理中继日志时 I/O（接收器）和 SQL（应用程序）线程注册的最后错误。理想情况下，这些应该为空，表示没有错误。

+   `Seconds_Behind_Source`：复制 SQL（应用程序）线程落后处理源头二进制日志的秒数。较高的数字（或递增的数字）可能表明复制无法及时处理源头的事件。

    `Seconds_Behind_Source` 为 0 的值通常可以解释为复制已经追赶上源头，但也有一些情况并非严格如此。例如，如果源头和复制之间的网络连接中断，但复制 I/O（接收器）线程尚未注意到这一点；也就是说，`replica_net_timeout` 或 `slave_net_timeout` 设置的时间段尚未过去。

    `Seconds_Behind_Source` 的瞬时值可能无法准确反映当前情况。当复制 SQL（应用程序）线程已经追赶上 I/O 时，`Seconds_Behind_Source` 显示为 0；但当复制 I/O（接收器）线程仍在排队新事件时，`Seconds_Behind_Source` 可能会显示一个较大的值，直到复制应用程序线程执行完新事件。特别是在事件具有旧时间戳的情况下，如果在相对较短的时间内多次执行 `SHOW REPLICA STATUS`，你可能会看到这个值在 0 和一个相对较大的值之间反复变化。

几对字段提供有关复制从源头二进制日志中读取事件并在中继日志中处理事件的进度信息：

+   (`Master_Log_file`, `Read_Master_Log_Pos`)：源头二进制日志中指示复制 I/O（接收器）线程已读取事件的坐标。

+   (`Relay_Master_Log_File`, `Exec_Master_Log_Pos`)：源头二进制日志中指示复制 SQL（应用程序）线程已执行从该日志接收的事件的坐标。

+   (`Relay_Log_File`, `Relay_Log_Pos`)：副本中继日志中的坐标，指示复制 SQL（应用程序）线程执行中继日志的进度。这些坐标对应于前面的坐标，但是以副本中继日志坐标表示，而不是源二进制日志坐标。

在源端，您可以使用`SHOW PROCESSLIST`检查连接的副本的状态，以查看正在运行的进程列表。副本连接在`Command`字段中具有`Binlog Dump`：

```sql
mysql> SHOW PROCESSLIST \G;
*************************** 4\. row ***************************
     Id: 10
   User: root
   Host: replica1:58371
     db: NULL
Command: Binlog Dump
   Time: 777
  State: Has sent all binlog to slave; waiting for binlog to be updated
   Info: NULL
```

因为是副本驱动复制过程，所以在此报告中几乎没有可用的信息。

对于使用`--report-host`选项启动并连接到源端的副本，源端上的`SHOW REPLICAS`（或在 MySQL 8.0.22 之前，`SHOW SLAVE HOSTS`）语句显示有关副本的基本信息。输出包括副本服务器的 ID，`--report-host`选项的值，连接端口和源 ID：

```sql
mysql> SHOW REPLICAS;
+-----------+----------+------+-------------------+-----------+
| Server_id | Host     | Port | Rpl_recovery_rank | Source_id |
+-----------+----------+------+-------------------+-----------+
|        10 | replica1 | 3306 |                 0 |         1 |
+-----------+----------+------+-------------------+-----------+
1 row in set (0.00 sec)
```
