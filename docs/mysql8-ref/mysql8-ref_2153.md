> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-memory-by-host-by-current-bytes.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-memory-by-host-by-current-bytes.html)

#### 30.4.3.16 memory_by_host_by_current_bytes 和 x$memory_by_host_by_current_bytes 视图

这些视图按主机分组总结内存使用情况。默认情况下，按内存使用量降序排序行。

`memory_by_host_by_current_bytes` 和 `x$memory_by_host_by_current_bytes` 视图具有以下列：

+   `host`

    客户端连接的主机。假定底层性能模式表中的 `HOST` 列为 `NULL` 的行是用于后台线程的，并且报告的主机名为 `background`。

+   `current_count_used`

    尚未释放的主机的当前分配内存块数。

+   `current_allocated`

    尚未释放的主机的当前分配字节数。

+   `current_avg_alloc`

    每个主机的当前分配字节数。

+   `current_max_alloc`

    主机的最大单个当前内存分配（以字节为单位）。

+   `total_allocated`

    主机的总内存分配（以字节为单位）。
