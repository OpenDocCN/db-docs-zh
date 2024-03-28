> 原文：[`dev.mysql.com/doc/refman/8.0/en/replication-features-max-allowed-packet.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-features-max-allowed-packet.html)

#### 19.5.1.20 复制和 max_allowed_packet

`max_allowed_packet`设置了 MySQL 服务器和客户端之间的任何单个消息的大小上限，包括副本。如果在源端复制大型列值（例如可能在`TEXT`或`BLOB`列中找到的值），而`max_allowed_packet`在源端设置过小，源端将会出现错误，并且副本会关闭复制 I/O（接收器）线程。如果副本端的`max_allowed_packet`设置过小，也会导致副本停止 I/O 线程。

基于行的复制从源端向副本发送更新行的所有列和列值，包括实际上未被更新的列的值。这意味着，当您使用基于行的复制复制大型列值时，您必须确保将`max_allowed_packet`设置得足够大，以容纳要复制的任何表中最大行的大小，即使您只复制更新，或者只插入相对较小的值。

在多线程复制（具有`replica_parallel_workers > 0`或`slave_parallel_workers > 0`）中，确保系统变量`replica_pending_jobs_size_max`或`slave_pending_jobs_size_max`的设置值等于或大于源端`max_allowed_packet`系统变量的设置值。`replica_pending_jobs_size_max`或`slave_pending_jobs_size_max`的默认设置为 128M，是`max_allowed_packet`系统变量的默认设置值 64M 的两倍。`max_allowed_packet`限制了源端可以发送的数据包大小，但添加事件头可能会产生超过此大小的二进制日志事件。此外，在基于行的复制中，单个事件可能比`max_allowed_packet`大小显著更大，因为`max_allowed_packet`的值仅限制表的每列。

实际上，复制端接受的数据包上限由其`replica_max_allowed_packet`或`slave_max_allowed_packet`设置确定，默认设置为最大设置值 1GB，以防止由于大数据包而导致复制失败。然而，`replica_pending_jobs_size_max`或`slave_pending_jobs_size_max`的值控制了复制端可用于保存传入数据包的内存。指定的内存是在所有复制工作队列之间共享的。

`replica_pending_jobs_size_max`或`slave_pending_jobs_size_max`的值是一个软限制，如果一个异常大的事件（由一个或多个数据包组成）超过了这个大小，事务将被暂停，直到所有副本工作者的队列为空，然后再处理。所有后续事务都将被暂停，直到大事务完成。因此，虽然大于`replica_pending_jobs_size_max`或`slave_pending_jobs_size_max`的异常事件可以被处理，但清空所有副本工作者队列和等待排队后续事务的延迟可能导致副本延迟和副本工作者并发性降低。因此，`replica_pending_jobs_size_max`或`slave_pending_jobs_size_max`应设置为足够高，以容纳大多数预期事件大小。
