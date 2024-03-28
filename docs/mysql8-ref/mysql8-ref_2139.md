> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-host-summary-by-file-io.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-host-summary-by-file-io.html)

#### 30.4.3.2 `host_summary_by_file_io` 和 `x$host_summary_by_file_io` 视图

这些视图按主机分组总结文件 I/O，默认情况下按降序总文件 I/O 延迟排序。

`host_summary_by_file_io` 和 `x$host_summary_by_file_io` 视图具有以下列：

+   `主机`

    客户端连接的主机。假设基础性能模式表中的`HOST`列为`NULL`的行是用于后台线程的，并且以`background`作为主机名报告。

+   `ios`

    主机的文件 I/O 事件总数。

+   `io_latency`

    主机的定时文件 I/O 事件的总等待时间。
