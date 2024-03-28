> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-user-summary-by-file-io.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-user-summary-by-file-io.html)

#### 30.4.3.42 `user_summary_by_file_io`和`x$user_summary_by_file_io`视图

这些视图按用户分组总结文件 I/O。默认情况下，按降序总文件 I/O 延迟排序行。

`user_summary_by_file_io`和`x$user_summary_by_file_io`视图具有以下列：

+   `user`

    客户端用户名。假定底层性能模式表中`USER`列为`NULL`的行是用于后台线程，并且以主机名`background`报告。

+   `ios`

    用户的文件 I/O 事件总数。

+   `io_latency`

    用户的定时文件 I/O 事件的总等待时间。
