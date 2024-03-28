# 17.5.4 日志缓冲区

> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-redo-log-buffer.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-redo-log-buffer.html)

日志缓冲区是保存待写入磁盘上日志文件的数据的内存区域。日志缓冲区大小由`innodb_log_buffer_size`变量定义。默认大小为 16MB。日志缓冲区的内容定期刷新到磁盘。较大的日志缓冲区使得大型事务可以在提交之前无需将重做日志数据写入磁盘。因此，如果您有更新、插入或删除许多行的事务，增加日志缓冲区的大小可以节省磁盘 I/O。

`innodb_flush_log_at_trx_commit`变量控制日志缓冲区的内容如何写入和刷新到磁盘。`innodb_flush_log_at_timeout`变量控制日志刷新频率。

有关相关信息，请参阅内存配置，以及第 10.5.4 节，“优化 InnoDB 重做日志记录”。
