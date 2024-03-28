> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-io-global-by-file-by-bytes.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-io-global-by-file-by-bytes.html)

#### 30.4.3.11 io_global_by_file_by_bytes 和 x$io_global_by_file_by_bytes 视图

这些视图总结了全局 I/O 消费者，以显示按文件分组的 I/O 量。默认情况下，按降序排列总 I/O（读取和写入的字节数）。

`io_global_by_file_by_bytes` 和 `x$io_global_by_file_by_bytes` 视图具有以下列：

+   `file`

    文件路径名。

+   `count_read`

    文件的总读取事件数。

+   `total_read`

    从文件中读取的总字节数。

+   `avg_read`

    每次从文件读取的平均字节数。

+   `count_write`

    文件的总写入事件数。

+   `total_written`

    向文件写入的总字节数。

+   `avg_write`

    每次写入文件的平均字节数。

+   `total`

    文件的总读取和写入字节数。

+   `write_pct`

    I/O 总字节数中写入的百分比。
