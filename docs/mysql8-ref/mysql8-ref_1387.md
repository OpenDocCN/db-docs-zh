> 原文：[`dev.mysql.com/doc/refman/8.0/en/replication-threads-monitor-main.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-threads-monitor-main.html)

#### 19.2.3.1 监控复制主线程

`SHOW PROCESSLIST`语句提供的信息告诉您关于复制在源和副本上发生的情况。有关源状态的信息，请参见第 10.14.4 节，“复制源线程状态”。有关副本状态，请参见第 10.14.5 节，“复制 I/O（接收器）线程状态” Thread States")，以及第 10.14.6 节，“复制 SQL 线程状态”。

以下示例说明了三个主要的复制线程，即二进制日志转储线程、复制 I/O（接收器）线程和复制 SQL（应用程序）线程，在`SHOW PROCESSLIST`的输出中显示。

在源服务器上，`SHOW PROCESSLIST`的输出如下：

```sql
mysql> SHOW PROCESSLIST\G
*************************** 1\. row ***************************
     Id: 2
   User: root
   Host: localhost:32931
     db: NULL
Command: Binlog Dump
   Time: 94
  State: Has sent all binlog to slave; waiting for binlog to
         be updated
   Info: NULL
```

在这里，线程 2 是服务于连接的副本的`Binlog Dump`线程。`State`信息表明所有未完成的更新已发送到副本，并且源正在等待更多更新发生。如果在源服务器上看不到`Binlog Dump`线程，则表示复制未运行；也就是说，当前没有副本连接。

在副本服务器上，`SHOW PROCESSLIST`的输出如下：

```sql
mysql> SHOW PROCESSLIST\G
*************************** 1\. row ***************************
     Id: 10
   User: system user
   Host:
     db: NULL
Command: Connect
   Time: 11
  State: Waiting for master to send event
   Info: NULL
*************************** 2\. row ***************************
     Id: 11
   User: system user
   Host:
     db: NULL
Command: Connect
   Time: 11
  State: Has read all relay log; waiting for the slave I/O
         thread to update it
   Info: NULL
```

`State`信息表明线程 10 是与源服务器通信的复制 I/O（接收器）线程，线程 11 是处理中继日志中存储的更新的复制 SQL（应用程序）线程。在运行`SHOW PROCESSLIST`时，这两个线程都处于空闲状态，等待进一步的更新。

`Time`列中的值可以显示副本相对于源的延迟情况。参见第 A.14 节，“MySQL 8.0 FAQ：复制”。如果源端在`Binlog Dump`线程上没有活动发生足够的时间，源会确定副本已不再连接。对于任何其他客户端连接，这些超时取决于`net_write_timeout`和`net_retry_count`的值；有关这些值的更多信息，请参见第 7.1.8 节，“服务器系统变量”。

`显示副本状态` 语句提供有关副本服务器上复制处理的额外信息。参见 第 19.1.7.1 节，“检查复制状态”。
