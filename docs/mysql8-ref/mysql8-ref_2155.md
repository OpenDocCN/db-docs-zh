> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-memory-by-user-by-current-bytes.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-memory-by-user-by-current-bytes.html)

#### 30.4.3.18 memory_by_user_by_current_bytes 和 x$memory_by_user_by_current_bytes 视图

这些视图按用户分组总结内存使用情况。默认情况下，按内存使用量降序排序行。

`memory_by_user_by_current_bytes` 和 `x$memory_by_user_by_current_bytes` 视图具有以下列：

+   `user`

    客户端用户名。假定底层性能模式表中的`USER`列为`NULL`的行是后台线程的行，并且以`background`主机名报告。

+   `current_count_used`

    当前分配但尚未释放的用户内存块数量。

+   `current_allocated`

    当前分配但尚未释放的用户字节总数。

+   `current_avg_alloc`

    每个用户内存块的当前分配字节数。

+   `current_max_alloc`

    用户的最大单个当前内存分配字节数。

+   `total_allocated`

    用户的总内存分配字节数。
