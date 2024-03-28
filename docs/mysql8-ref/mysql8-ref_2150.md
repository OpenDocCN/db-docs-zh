> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-io-global-by-wait-by-bytes.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-io-global-by-wait-by-bytes.html)

#### 30.4.3.13 io_global_by_wait_by_bytes 和 x$io_global_by_wait_by_bytes 视图

这些视图总结了全局 I/O 消费者，显示了 I/O 量和等待 I/O 时间，按事件分组。默认情况下，按降序排列总 I/O（读取和写入的字节数）的行。

`io_global_by_wait_by_bytes` 和 `x$io_global_by_wait_by_bytes` 视图具有以下列：

+   `event_name`

    去除了`wait/io/file/`前缀的 I/O 事件名称。

+   `total`

    I/O 事件的总发生次数。

+   `total_latency`

    I/O 事件的定时发生的总等待时间。

+   `min_latency`

    I/O 事件的定时发生的最小单次等待时间。

+   `avg_latency`

    I/O 事件的每次定时发生的平均等待时间。

+   `max_latency`

    I/O 事件的定时发生的最大单次等待时间。

+   `count_read`

    I/O 事件的读取请求次数。

+   `total_read`

    I/O 事件的读取字节数。

+   `avg_read`

    I/O 事件的每次读取的平均字节数。

+   `count_write`

    I/O 事件的写入请求次数。

+   `total_written`

    I/O 事件的写入字节数。

+   `avg_written`

    I/O 事件的每次写入的平均字节数。

+   `total_requested`

    I/O 事件的总读取和写入字节数。
