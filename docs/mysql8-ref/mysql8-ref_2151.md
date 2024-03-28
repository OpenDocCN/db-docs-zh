> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-io-global-by-wait-by-latency.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-io-global-by-wait-by-latency.html)

#### 30.4.3.14 io_global_by_wait_by_latency 和 x$io_global_by_wait_by_latency 视图

这些视图总结了全局 I/O 消费者，显示了 I/O 量和等待 I/O 的时间，按事件分组。默认情况下，行按总延迟降序排序。

`io_global_by_wait_by_latency` 和 `x$io_global_by_wait_by_latency` 视图具有以下列：

+   `event_name`

    I/O 事件名称，去除`wait/io/file/`前缀。

+   `total`

    I/O 事件的总发生次数。

+   `total_latency`

    I/O 事件的定时发生的总等待时间。

+   `avg_latency`

    I/O 事件的每次定时发生的平均等待时间。

+   `max_latency`

    I/O 事件的定时发生的最大单次等待时间。

+   `read_latency`

    I/O 事件的读取定时发生的总等待时间。

+   `write_latency`

    I/O 事件的写入定时发生的总等待时间。

+   `misc_latency`

    I/O 事件的其他定时发生的总等待时间。

+   `count_read`

    I/O 事件的读取请求数。

+   `total_read`

    I/O 事件的读取字节数。

+   `avg_read`

    I/O 事件的每次读取的平均字节数。

+   `count_write`

    I/O 事件的写入请求数。

+   `total_written`

    I/O 事件的写入字节数。

+   `avg_written`

    I/O 事件的每次写入的平均字节数。
