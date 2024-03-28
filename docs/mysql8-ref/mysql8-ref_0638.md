# 10.14 检查服务器线程（进程）信息

> 原文：[`dev.mysql.com/doc/refman/8.0/en/thread-information.html`](https://dev.mysql.com/doc/refman/8.0/en/thread-information.html)

10.14.1 访问进程列表

10.14.2 线程命令值

10.14.3 通用线程状态

10.14.4 复制源线程状态

10.14.5 复制 I/O（接收器）线程状态

10.14.6 复制 SQL 线程状态

10.14.7 复制连接线程状态

10.14.8 NDB 集群线程状态

10.14.9 事件调度器线程状态

要了解你的 MySQL 服务器正在做什么，检查进程列表可能会有所帮助，该列表显示了服务器内执行的一组线程当前正在执行的操作。例如：

```sql
mysql> SHOW PROCESSLIST\G
*************************** 1\. row ***************************
     Id: 5
   User: event_scheduler
   Host: localhost
     db: NULL
Command: Daemon
   Time: 2756681
  State: Waiting on empty queue
   Info: NULL
*************************** 2\. row ***************************
     Id: 20
   User: me
   Host: localhost:52943
     db: test
Command: Query
   Time: 0
  State: starting
   Info: SHOW PROCESSLIST
```

可以使用 `KILL` 语句终止线程。参见 第 15.7.8.4 节，“KILL 语句”。
