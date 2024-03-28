> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-wait-classes-global-by-latency.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-wait-classes-global-by-latency.html)

#### 30.4.3.49 wait_classes_global_by_latency 和 x$wait_classes_global_by_latency 视图

这些视图按事件类别分组总结等待类别总延迟。默认情况下，按降序总延迟排序行。空闲事件被忽略。

事件类别由事件名称中第一个三个组件之后的所有内容剥离而确定。例如，`wait/io/file/sql/slow_log`的类别是`wait/io/file`。

`wait_classes_global_by_latency`和`x$wait_classes_global_by_latency`视图具有以下列：

+   `event_class`

    事件类别。

+   `total`

    事件类别中事件发生的总次数。

+   `total_latency`

    事件类别中定时事件的总等待时间。

+   `min_latency`

    事件类别中定时事件的最小单个等待时间。

+   `avg_latency`

    事件类别中每个定时事件的平均等待时间。

+   `max_latency`

    事件类别中定时事件的最大单个等待时间。
