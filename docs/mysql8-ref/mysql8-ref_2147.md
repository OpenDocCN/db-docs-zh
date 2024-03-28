> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-io-by-thread-by-latency.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-io-by-thread-by-latency.html)

#### 30.4.3.10 `io_by_thread_by_latency`和`x$io_by_thread_by_latency`视图

这些视图总结了 I/O 消费者的信息，显示了等待 I/O 的时间，按线程分组。默认情况下，按照总 I/O 延迟降序排序行。

`io_by_thread_by_latency`和`x$io_by_thread_by_latency`视图具有以下列：

+   `user`

    对于前台线程，与线程关联的账户。对于后台线程，线程名称。

+   `total`

    线程的 I/O 事件总数。

+   `total_latency`

    线程的定时 I/O 事件的总等待时间。

+   `min_latency`

    线程的定时 I/O 事件的最小单次等待时间。

+   `avg_latency`

    平均每个线程的定时 I/O 事件等待时间。

+   `max_latency`

    线程的定时 I/O 事件的最大单次等待时间。

+   `thread_id`

    线程 ID。

+   `processlist_id`

    对于前台线程，线程的进程列表 ID。对于后台线程，`NULL`。
