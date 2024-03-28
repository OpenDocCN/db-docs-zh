> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-waits-by-user-by-latency.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-waits-by-user-by-latency.html)

#### 30.4.3.51 waits_by_user_by_latency 和 x$waits_by_user_by_latency 视图

这些视图按用户和事件分组总结等待事件。默认情况下，按用户和降序总延迟排序。空闲事件将被忽略。

[`waits_by_user_by_latency`](https://dev.mysql.com/doc/refman/8.0/en/sys-waits-by-user-by-latency.html "30.4.3.51 waits_by_user_by_latency 和 x$waits_by_user_by_latency 视图") 和 [`x$waits_by_user_by_latency`](https://dev.mysql.com/doc/refman/8.0/en/sys-waits-by-user-by-latency.html "30.4.3.51 waits_by_user_by_latency 和 x$waits_by_user_by_latency 视图") 视图具有以下列：

+   `用户`

    与连接相关联的用户。

+   `事件`

    事件名称。

+   `总数`

    用户的事件发生次数的总数。

+   `总延迟`

    用户事件发生的总等待时间。

+   `平均延迟`

    用户每次事件发生的平均等待时间。

+   `最大延迟`

    用户事件发生的单次最大等待时间。
