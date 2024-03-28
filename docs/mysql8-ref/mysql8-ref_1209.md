# 17.8.5 配置后台 InnoDB I/O 线程的数量

> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-performance-multiple_io_threads.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-performance-multiple_io_threads.html)

`InnoDB`使用后台线程来处理各种类型的 I/O 请求。您可以使用`innodb_read_io_threads`和`innodb_write_io_threads`配置参数来配置服务数据页读取和写入 I/O 的后台线程数量。这些参数分别表示用于读取和写入请求的后台线程数量。它们在所有支持的平台上都有效。您可以在 MySQL 选项文件（`my.cnf`或`my.ini`）中为这些参数设置值；您不能动态更改值。这些参数的默认值为`4`，允许的值范围为`1-64`。

这些配置选项的目的是使`InnoDB`在高端系统上更具可扩展性。每个后台线程可以处理多达 256 个待处理的 I/O 请求。后台 I/O 的一个主要来源是预读取请求。`InnoDB`试图平衡传入请求的负载，使大多数后台线程平均分担工作。`InnoDB`还尝试将来自同一范围的读取请求分配给同一线程，以增加合并请求的机会。如果您拥有高端 I/O 子系统，并且在`SHOW ENGINE INNODB STATUS`输出中看到超过 64 × `innodb_read_io_threads`个待处理读取请求，您可以通过增加`innodb_read_io_threads`的值来提高性能。

在 Linux 系统上，默认情况下，`InnoDB`使用异步 I/O 子系统执行数据文件页的预读取和写入请求，这改变了`InnoDB`后台线程处理这些类型 I/O 请求的方式。有关更多信息，请参阅第 17.8.6 节，“在 Linux 上使用异步 I/O”。

有关`InnoDB` I/O 性能的更多信息，请参阅第 10.5.8 节，“优化 InnoDB 磁盘 I/O”。
