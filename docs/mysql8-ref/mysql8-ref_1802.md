> [`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-resources.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-resources.html)

#### 25.6.16.51 ndbinfo 资源表

此表提供有关数据节点资源可用性和使用情况的信息。

这些资源有时被称为超级池。

`resources`表包含以下列：

+   `node_id`

    此数据节点的唯一节点 ID。

+   `resource_name`

    资源的名称；参见文本。

+   `reserved`

    为此资源保留的 32KB 页面数。

+   `used`

    实际由此资源使用的 32KB 页面数。

+   `max`

    自节点上次启动以来使用的此资源的最大数量（32KB 页面数）。

##### 注意事项

`resource_name`可以是以下表中显示的任何一个名称：

+   `RESERVED`：系统保留；不能被覆盖。

+   `TRANSACTION_MEMORY`：为此数据节点上的事务分配的内存。在 NDB 8.0.19 及更高版本中，可以使用`TransactionMemory`配置参数来控制。

+   `DISK_OPERATIONS`：如果分配了日志文件组，则使用撤销日志缓冲区的大小来设置此资源的大小。此资源仅用于为撤销日志文件组分配撤销日志缓冲区；只能有一个这样的组。根据需要通过`CREATE LOGFILE GROUP`进行过度分配。

+   `DISK_RECORDS`：为磁盘数据操作分配的记录。

+   `DATA_MEMORY`：用于主内存元组、索引和哈希索引。DataMemory 和 IndexMemory 的总和，如果 IndexMemory 已设置，则再加上 8 个 32KB 的页面。不能过度分配。

+   `JOBBUFFER`：由 NDB 调度程序用于分配作业缓冲区；不能过度分配。每个线程大约为 2MB，以及所有可以通信的线程两个方向各有 1MB 的缓冲区。对于大型配置，这将消耗数 GB。

+   `FILE_BUFFERS`：由`DBLQH`内核块中的重做日志处理程序使用；不能过度分配。大小为`NoOfFragmentLogParts` * `RedoBuffer`，再加上每个日志文件部分 1MB。

+   `TRANSPORTER_BUFFERS`: 由**ndbmtd**")使用的发送缓冲区；`TotalSendBufferMemory`和`ExtraSendBufferMemory`之和。这个资源可以过度分配多达 25%。`TotalSendBufferMemory`通过对每个节点的发送缓冲区内存求和来计算，其默认值为 2 MB。因此，在一个有四个数据节点和八个 API 节点的系统中，数据节点有 12 * 2 MB 的发送缓冲区内存。`ExtraSendBufferMemory`由**ndbmtd**")使用，每个线程额外使用 2 MB 内存。因此，对于 4 个 LDM 线程、2 个 TC 线程、1 个主线程、1 个复制线程和 2 个接收线程，`ExtraSendBufferMemory`为 10 * 2 MB。可以通过设置`SharedGlobalMemory`数据节点配置参数来过度分配这个资源。

+   `DISK_PAGE_BUFFER`: 用于磁盘页缓冲区；由`DiskPageBufferMemory`配置参数确定。不能过度分配。

+   `QUERY_MEMORY`: 被`DBSPJ`内核块使用。

+   `SCHEMA_TRANS_MEMORY`: 最小为 2 MB；可以过度分配以使用任何剩余的可用内存。
