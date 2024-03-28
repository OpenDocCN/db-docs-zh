# 19.2.3 复制线程

> 原文：[`dev.mysql.com/doc/refman/8.0/en/replication-threads.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-threads.html)

19.2.3.1 监控复制主线程

19.2.3.2 监控复制应用程序工作线程

MySQL 复制功能使用以下类型的线程实现：

+   **二进制日志转储线程。** 源码在复制品连接时创建一个线程，将二进制日志内容发送到复制品。在源码的`SHOW PROCESSLIST`输出中，可以将此线程标识为`Binlog Dump`线程。

+   **复制 I/O 接收线程。** 当在复制品服务器上发出`START REPLICA`语句时，复制品将创建一个 I/O（接收器）线程，该线程连接到源并要求其发送其二进制日志中记录的更新。

    复制接收器线程读取源的`Binlog Dump`线程发送的更新（请参见前一项），并将其复制到组成复制品中继日志的本地文件中。

    此线程的状态在`SHOW REPLICA STATUS`的输出中显示为`Slave_IO_running`。

+   **复制 SQL 应用程序线程。** 当`replica_parallel_workers`（在 MySQL 8.0.26 及更早版本中，请使用`slave_parallel_workers`）等于 0 时，复制品将创建一个 SQL（应用程序）线程来读取由复制接收器线程写入的中继日志，并执行其中包含的事务。当`replica_parallel_workers`为`*`N`* >= 1`时，存在*N*个应用程序线程和一个协调器线程，该协调器线程按顺序从中继日志中读取事务，并安排工作线程应用它们。每个工作线程应用协调器分配给它的事务。

通过将系统变量`replica_parallel_workers`（MySQL 8.0.26 或更高版本）或`slave_parallel_workers`（MySQL 8.0.26 之前）设置为大于 0 的值，可以进一步并行化复制品上的任务。这样做后，复制品将创建指定数量的工作线程来应用事务，以及一个协调器线程，该线程从中继日志中读取事务并将其分配给工作线程。将`replica_parallel_workers`（`slave_parallel_workers`）设置为大于 0 的值的复制品称为多线程复制品。如果使用多个复制通道，则每个通道都使用此变量指定的线程数。

注意

多线程复制在 NDB 集群中从 NDB 8.0.33 版本开始得到支持。（之前，`NDB` 会默默地忽略 `replica_parallel_workers` 的任何设置。）详细信息请参见 第 25.7.11 节，“使用多线程应用程序的 NDB 集群复制”。
