> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-pgman-time-track-stats.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-pgman-time-track-stats.html)

#### 25.6.16.49 ndbinfo pgman_time_track_stats 表

该表提供有关 NDB Cluster Disk Data 表空间磁盘操作延迟的信息。

`pgman_time_track_stats` 表包含以下列：

+   `node_id`

    集群中此节点的唯一节点 ID

+   `block_number`

    块编号（来自 `blocks` 表）

+   `block_instance`

    块实例编号

+   `upper_bound`

    上限

+   `page_reads`

    页面读取延迟（毫秒）

+   `page_writes`

    页面写入延迟（毫秒）

+   `log_waits`

    日志等待延迟（毫秒）

+   `get_page`

    `get_page()` 调用的延迟（毫秒）

##### 注意

读取延迟（`page_reads` 列）衡量了从发送读取请求到文件系统线程直到读取完成并报告给执行线程的时间。写入延迟（`page_writes`）以类似的方式计算。从磁盘数据表空间读取或写入的页面大小始终为 32 KB。

日志等待延迟（`log_waits` 列）是页面写入必须等待撤销日志刷新的时间长度，这必须在每次页面写入之前完成。
