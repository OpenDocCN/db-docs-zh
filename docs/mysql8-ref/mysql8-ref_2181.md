> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-user-summary-by-stages.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-user-summary-by-stages.html)

#### 30.4.3.44 用户 _summary_by_stages 和 x$user_summary_by_stages 视图

这些视图按用户分组总结阶段。默认情况下，按用户和降序总阶段延迟排序。

`user_summary_by_stages` 和 `x$user_summary_by_stages` 视图具有以下列：

+   `user`

    客户端用户名。在底层性能模式表中 `USER` 列为 `NULL` 的行被假定为后台线程，并以 `background` 主机名报告。

+   `event_name`

    阶段事件名称。

+   `total`

    用户的阶段事件发生总次数。

+   `total_latency`

    用户的阶段事件定时发生的总等待时间。

+   `avg_latency`

    用户的阶段事件定时发生的平均等待时间。
