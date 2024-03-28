> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-memory-global-by-current-bytes.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-memory-global-by-current-bytes.html)

#### 30.4.3.19 memory_global_by_current_bytes 和 x$memory_global_by_current_bytes 视图

这些视图按分配类型（即按事件）汇总内存使用情况。默认情况下，按内存使用量降序排序行。

`memory_global_by_current_bytes` 和 `x$memory_global_by_current_bytes` 视图具有以下列：

+   `event_name`

    内存事件名称。

+   `current_count`

    事件发生的总次数。

+   `current_alloc`

    为事件分配但尚未释放的当前分配字节数。

+   `current_avg_alloc`

    当前为事件分配的每个内存块的字节数。

+   `high_count`

    为事件分配的内存块数量的高水位标记。

+   `high_alloc`

    为事件分配的字节数的高水位标记。

+   `high_avg_alloc`

    平均每个内存块为事件分配的字节数的高水位标记。
