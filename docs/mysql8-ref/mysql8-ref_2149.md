> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-io-global-by-file-by-latency.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-io-global-by-file-by-latency.html)

#### 30.4.3.12 io_global_by_file_by_latency 和 x$io_global_by_file_by_latency 视图

这些视图总结了全局 I/O 消费者，以显示按文件分组的 I/O 等待时间。默认情况下，按总等待时间降序排序行。

`io_global_by_file_by_latency` 和 `x$io_global_by_file_by_latency` 视图具有以下列：

+   `file`

    文件路径名。

+   `total`

    文件的 I/O 事件总数。

+   `total_latency`

    文件的定时 I/O 事件的总等待时间。

+   `count_read`

    文件的总读取 I/O 事件数。

+   `read_latency`

    文件的定时读取 I/O 事件的总等待时间。

+   `count_write`

    文件的写入 I/O 事件总数。

+   `write_latency`

    文件的定时写入 I/O 事件的总等待时间。

+   `count_misc`

    文件的其他 I/O 事件总数。

+   `misc_latency`

    文件的定时其他 I/O 事件的总等待时间。
