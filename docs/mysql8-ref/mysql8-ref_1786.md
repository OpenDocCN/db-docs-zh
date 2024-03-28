> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-files.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-files.html)

#### 25.6.16.35 ndbinfo 文件表

`files`表提供了关于`NDB`磁盘数据表使用的文件和其他对象的信息，并包含以下列：

+   `id`

    对象 ID

+   `type`

    对象的类型；可以是`Log file group`、`Tablespace`、`Undo file`或`Data file`之一

+   `name`

    对象的名称

+   `parent`

    父对象的 ID

+   `parent_name`

    父对象的名称

+   `free_extents`

    空闲范围的数量

+   `total_extents`

    总范围数

+   `extent_size`

    范围大小（MB）

+   `initial_size`

    初始大小（字节）

+   `maximum_size`

    最大大小（字节）

+   `autoextend_size`

    自动扩展大小（字节）

对于日志文件组和表空间，`parent`始终为`0`，而`parent_name`、`free_extents`、`total_extents`、`extent_size`、`initial_size`、`maximum_size`和`autoentend_size`列都为`NULL`。

如果在`NDB`中没有创建磁盘数据对象，则`files`表为空。有关更多信息，请参见第 25.6.11.1 节，“NDB Cluster Disk Data Objects”。

`files`表是在 NDB 8.0.29 中添加的。

另请参阅第 28.3.15 节，“INFORMATION_SCHEMA FILES Table”。
