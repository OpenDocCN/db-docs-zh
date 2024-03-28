> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-user-summary.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-user-summary.html)

#### 30.4.3.41 用户摘要和 x$user_summary 视图

这些视图总结了语句活动、文件 I/O 和连接，按用户分组。默认情况下，按总延迟降序排序行。

`user_summary` 和 `x$user_summary` 视图具有以下列：

+   `user`

    客户端用户名。在底层性能模式表中`USER`列为`NULL`的行被认为是后台线程，并以`background`主机名报告。

+   `statements`

    用户的语句总数。

+   `statement_latency`

    用户的定时语句的总等待时间。

+   `statement_avg_latency`

    用户的每个定时语句的平均等待时间。

+   `table_scans`

    用户的表扫描总数。

+   `file_ios`

    用户的文件 I/O 事件总数。

+   `file_io_latency`

    用户的定时文件 I/O 事件的总等待时间。

+   `current_connections`

    用户的当前连接数。

+   `total_connections`

    用户的总连接数。

+   `unique_hosts`

    用户发起连接的不同主机数量。

+   `current_memory`

    用户的当前分配内存量。

+   `total_memory_allocated`

    用户的总分配内存量。
