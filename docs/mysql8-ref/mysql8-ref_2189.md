> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-waits-global-by-latency.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-waits-global-by-latency.html)

#### 30.4.3.52 waits_global_by_latency 和 x$waits_global_by_latency 视图

这些视图按事件分组总结等待事件。默认情况下，按总延迟降序排序行。空闲事件将被忽略。

`waits_global_by_latency` 和 `x$waits_global_by_latency` 视图包含以下列：

+   `events`

    事件名称。

+   `total`

    事件发生的总次数。

+   `total_latency`

    事件定时发生的总等待时间。

+   `avg_latency`

    事件定时发生的平均等待时间。

+   `max_latency`

    事件定时发生的最长等待时间。
