> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-host-summary-by-file-io-type.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-host-summary-by-file-io-type.html)

#### 30.4.3.3 主机按文件 I/O 类型和 x$host_summary_by_file_io_type 视图

这些视图按主机和事件类型分组汇总文件 I/O。默认情况下，行按主机和总 I/O 延迟降序排序。

`host_summary_by_file_io_type` 和 `x$host_summary_by_file_io_type` 视图具有以下列：

+   `host`

    客户端连接的主机。在底层性能模式表中 `HOST` 列为 `NULL` 的行被假定为后台线程，并以 `background` 作为主机名报告。

+   `event_name`

    文件 I/O 事件名称。

+   `total`

    主机上文件 I/O 事件发生的总次数。

+   `total_latency`

    主机上文件 I/O 事件的定时发生的总等待时间。

+   `max_latency`

    主机上文件 I/O 事件的单个最大等待时间。
