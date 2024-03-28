> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-host-summary-by-stages.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-host-summary-by-stages.html)

#### 30.4.3.4 主机 _summary_by_stages 和 x$host_summary_by_stages 视图

这些视图按主机分组总结语句阶段。默认情况下，按主机和降序总延迟排序行。

`host_summary_by_stages` 和 `x$host_summary_by_stages` 视图具有以下列：

+   `host`

    客户端连接的主机。假定底层性能模式表中的`HOST`列为`NULL`的行是用于后台线程的，并报告为`background`主机名。

+   `event_name`

    舞台事件名称。

+   `total`

    主机的舞台事件发生次数总数。

+   `total_latency`

    主机的舞台事件定时发生的总等待时间。

+   `avg_latency`

    主机的舞台事件每次定时发生的平均等待时间。
