> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-memory-by-thread-by-current-bytes.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-memory-by-thread-by-current-bytes.html)

#### 30.4.3.17 The memory_by_thread_by_current_bytes and x$memory_by_thread_by_current_bytes Views

这些视图按线程分组总结内存使用情况。默认情况下，按内存使用量降序排序。

`memory_by_thread_by_current_bytes` 和 `x$memory_by_thread_by_current_bytes` 视图具有以下列：

+   `thread_id`

    线程 ID。

+   `user`

    线程用户或线程名称。

+   `current_count_used`

    尚未释放的线程的当前分配内存块数。

+   `current_allocated`

    尚未释放的线程的当前分配字节数。

+   `current_avg_alloc`

    每个线程的内存块的当前分配字节数。

+   `current_max_alloc`

    线程的最大单个当前内存分配字节数。

+   `total_allocated`

    线程的总内存分配字节数。
