# 25.6.11 NDB Cluster 磁盘数据表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-disk-data.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-disk-data.html)

25.6.11.1 NDB Cluster 磁盘数据对象

25.6.11.2 NDB Cluster 磁盘数据存储要求

NDB Cluster 支持将`NDB`表的非索引列存储在磁盘上，而不是在 RAM 中。列数据和日志元数据保存在数据文件和撤销日志文件中，概念化为表空间和日志文件组，如下一节所述—参见第 25.6.11.1 节，“NDB Cluster 磁盘数据对象”。

NDB Cluster 磁盘数据性能可能受多个配置参数的影响。有关这些参数及其影响的信息，请参见磁盘数据配置参数，以及磁盘数据和 GCP 停止错误。

当使用单独的磁盘存储磁盘数据文件时，您还应将`DiskDataUsingSameDisk`数据节点配置参数设置为`false`。

参见磁盘数据文件系统参数。

NDB 8.0 在使用固态硬盘的磁盘数据表时提供了改进的支持，特别是使用 NVMe 的硬盘。有关更多信息，请参阅以下文档：

+   磁盘数据延迟参数

+   第 25.6.16.31 节，“ndbinfo diskstat 表”

+   第 25.6.16.32 节，“ndbinfo diskstats_1sec 表”

+   第 25.6.16.49 节，“ndbinfo pgman_time_track_stats 表”
