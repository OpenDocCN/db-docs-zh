> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-waits-by-host-by-latency.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-waits-by-host-by-latency.html)

#### 30.4.3.50 waits_by_host_by_latency 和 x$waits_by_host_by_latency 视图

这些视图按主机和事件分组总结等待事件。默认情况下，按主机和降序总延迟排序行。空闲事件被忽略。

[`waits_by_host_by_latency`](https://dev.mysql.com/doc/refman/8.0/en/sys-waits-by-host-by-latency.html "30.4.3.50 waits_by_host_by_latency 和 x$waits_by_host_by_latency 视图") 和 [`x$waits_by_host_by_latency`](https://dev.mysql.com/doc/refman/8.0/en/sys-waits-by-host-by-latency.html "30.4.3.50 waits_by_host_by_latency 和 x$waits_by_host_by_latency 视图") 视图具有以下列：

+   `host`

    连接来源主机。

+   `event`

    事件名称。

+   `total`

    主机的事件发生总次数。

+   `total_latency`

    主机的事件定时发生的总等待时间。

+   `avg_latency`

    主机的事件每次定时发生的平均等待时间。

+   `max_latency`

    主机的事件定时发生的最大单个等待时间。
