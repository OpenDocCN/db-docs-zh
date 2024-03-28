> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-host-summary.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-host-summary.html)

#### 30.4.3.1 主机摘要和 x$host_summary 视图

这些视图按主机分组总结语句活动、文件 I/O 和连接。

`host_summary` 和 `x$host_summary` 视图具有以下列：

+   `host`

    客户端连接的主机。假定底层性能模式表中的`HOST`列为`NULL`的行是用于后台线程的，并报告为`background`主机名。

+   `statements`

    主机的总语句数。

+   `statement_latency`

    主机的定时语句的总等待时间。

+   `statement_avg_latency`

    主机的定时语句的平均等待时间。

+   `table_scans`

    主机的总表扫描次数。

+   `file_ios`

    主机的总文件 I/O 事件数。

+   `file_io_latency`

    主机的定时文件 I/O 事件的总等待时间。

+   `current_connections`

    主机的当前连接数。

+   `total_connections`

    主机的总连接数。

+   `unique_users`

    主机的不同用户数。

+   `current_memory`

    主机的当前分配内存量。

+   `total_memory_allocated`

    主机的总分配内存量。
