# 17.8.6 在 Linux 上使用异步 I/O

> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-linux-native-aio.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-linux-native-aio.html)

`InnoDB`在 Linux 上使用异步 I/O 子系统（本机 AIO）执行数据文件页的预读和写请求。此行为由`innodb_use_native_aio`配置选项控制，仅适用于 Linux 系统，并且默认启用。在其他类 Unix 系统上，`InnoDB`仅使用同步 I/O。从历史上看，`InnoDB`仅在 Windows 系统上使用异步 I/O。在 Linux 上使用异步 I/O 子系统需要`libaio`库。

使用同步 I/O 时，查询线程排队 I/O 请求，`InnoDB`后台线程逐个检索排队的请求，为每个请求发出同步 I/O 调用。当 I/O 请求完成并且 I/O 调用返回时，处理请求的`InnoDB`后台线程调用 I/O 完成例程并返回以处理下一个请求。可以并行处理的请求数量为*`n`*，其中*`n`*是`InnoDB`后台线程的数量。`InnoDB`后台线程的数量由`innodb_read_io_threads`和`innodb_write_io_threads`控制。参见第 17.8.5 节，“配置后台 InnoDB I/O 线程数量”。

使用本机 AIO，查询线程直接将 I/O 请求分派给操作系统，从而消除了后台线程数量的限制。`InnoDB`后台线程等待 I/O 事件来标志完成的请求。当请求完成时，后台线程调用 I/O 完成例程并恢复等待 I/O 事件。

本机 AIO 的优势在于对于通常在`SHOW ENGINE INNODB STATUS\G`输出中显示许多待处理读/写操作的重度 I/O 绑定系统的可伸缩性。使用本机 AIO 时的并行处理增加意味着 I/O 性能受到 I/O 调度程序类型或磁盘阵列控制器属性的更大影响。

对于重度 I/O 绑定系统，本机 AIO 的一个潜在缺点是无法控制一次性发送到操作系统的 I/O 写请求数量。在某些情况下，一次性发送太多 I/O 写请求到操作系统进行并行处理可能导致 I/O 读取饥饿，这取决于 I/O 活动量和系统能力。

如果操作系统中异步 I/O 子系统出现问题导致`InnoDB`无法启动，您可以使用`innodb_use_native_aio=0`选项启动服务器。在启动过程中，如果`InnoDB`检测到潜在问题，比如`tmpdir`位置、`tmpfs`文件系统和 Linux 内核的组合不支持在`tmpfs`上进行异步 I/O，该选项也可能会被自动禁用。
