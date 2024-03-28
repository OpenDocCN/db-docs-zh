> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-disk-data-storage-requirements.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-disk-data-storage-requirements.html)

#### 25.6.11.2 NDB 集群磁盘数据存储要求

以下项目适用于磁盘数据存储要求：

+   变长列的磁盘数据表占据固定的空间。对于每一行，这等于存储该列可能的最大值所需的空间。

    有关计算这些值的一般信息，请参阅第 13.7 节，“数据类型存储要求”。

    您可以通过查询信息模式`FILES`表来估算数据文件和撤销日志文件中可用空间的数量。有关更多信息和示例，请参阅第 28.3.15 节，“INFORMATION_SCHEMA FILES 表”。

    注意

    `OPTIMIZE TABLE`语句对磁盘数据表没有任何影响。

+   在磁盘数据表中，`TEXT`或`BLOB`列的前 256 个字节存储在内存中；只有剩余部分存储在磁盘上。

+   磁盘数据表中的每一行在内存中使用 8 个字节指向存储在磁盘上的数据。这意味着，在某些情况下，将内存列转换为基于磁盘的格式实际上可能导致更大的内存使用量。例如，将`CHAR(4)`列从基于内存的格式转换为基于磁盘的格式会将每行使用的`DataMemory`从 4 字节增加到 8 字节。

重要提示

使用`--initial`选项启动集群*不会*删除磁盘数据文件。在执行集群的初始重新启动之前，您必须手动删除这些文件。

通过确保`DiskPageBufferMemory`的大小足够，可以通过最小化磁盘寻址次数来提高磁盘数据表的性能。您可以查询`diskpagebuffer`表来帮助确定是否需要增加此参数的值。
