> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-limitations-disk-data.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-limitations-disk-data.html)

#### 25.2.7.9 与 NDB 集群磁盘数据存储相关的限制

**磁盘数据对象的最大值和最小值。** 磁盘数据对象受以下最大值和最小值的限制：

+   表空间的最大数量：2³² (4294967296)

+   每个表空间的数据文件的最大数量：2¹⁶ (65536)

+   表空间数据文件的区段的最小和最大可能大小分别为 32K 和 2G。有关更多信息，请参见第 15.1.21 节，“CREATE TABLESPACE Statement”。

此外，在使用 NDB 磁盘数据表时，您应该注意以下关于数据文件和区段的问题：

+   数据文件使用`DataMemory`。使用方式与内存数据相同。

+   数据文件使用文件描述符。重要的是要记住，数据文件始终处于打开状态，这意味着文件描述符始终在使用中，不能用于其他系统任务。

+   区段需要足够的`DiskPageBufferMemory`；您必须为此参数保留足够的空间，以考虑所有区段使用的所有内存（区段数量乘以区段大小）。

**磁盘数据表和无磁盘模式。** 在无磁盘模式下运行集群时，不支持使用磁盘数据表。
