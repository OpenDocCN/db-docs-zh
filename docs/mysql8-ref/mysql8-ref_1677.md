> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-data-node-memory-management.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-data-node-memory-management.html)

#### 25.4.3.13 数据节点内存管理

所有数据节点的内存分配都在节点启动时执行。这确保数据节点可以稳定运行，而不使用交换内存，以便`NDB`可用于延迟敏感（实时）应用程序。数据节点启动时分配以下类型的内存：

+   数据内存

+   共享全局内存

+   重做日志缓冲区

+   作业缓冲区

+   发送缓冲区

+   用于磁盘数据记录的页缓存

+   模式事务内存

+   事务内存

+   撤销日志缓冲区

+   查询内存

+   块对象

+   模式内存

+   块数据结构

+   长信号内存

+   共享内存通信缓冲区

`NDB`内存管理器，管理大部分数据节点内存，处理以下内存资源：

+   数据内存（`DataMemory`）

+   重做日志缓冲区（`RedoBuffer`）

+   作业缓冲区

+   发送缓冲区（`SendBufferMemory`，`TotalSendBufferMemory`，`ExtraSendBufferMemory`）

+   磁盘数据记录页缓存（`DiskPageBufferMemory`，`DiskPageBufferEntries`）

+   事务内存（`TransactionMemory`）

+   查询内存

+   磁盘访问记录

+   文件缓冲区

每个资源都设置了一个保留内存区域和一个最大内存区域。保留内存区域只能被其保留的资源使用，不能与其他资源共享；给定资源永远不能分配超过为其允许的最大内存的内存。没有最大内存的资源可以扩展使用内存管理器中的所有共享内存。

这些资源的全局共享内存大小由`SharedGlobalMemory`配���参数控制（默认值：128 MB）。

数据内存始终保留，从共享内存中不获取任何内存。它使用`DataMemory`配置参数进行控制，其最大值为 16384 GB。`DataMemory`是记录存储的地方，包括哈希索引（每行约 15 字节）、有序索引（每个索引每行 10-12 字节）和行头（每行 16-32 字节）。

重做日志缓冲区也仅使用保留内存；这由`RedoBuffer`配置参数控制，该参数设置每个 LDM 线程的重做日志缓冲区大小。这意味着实际使用的内存量是此参数值乘以数据节点中的 LDM 线程数。

作业缓冲区仅使用保留内存；此内存的大小由`NDB`根据各种类型线程的数量计算而得。

发送缓冲区有一个保留部分，但也可以分配额外的共享全局内存的 25%。发送缓冲区的保留大小分两步计算：

1.  使用`TotalSendBufferMemory`配置参数的值（无默认值）或所有单个连接到数据节点的单个连接使用的所有单个发送缓冲区的总和。数据节点连接到所有其他数据节点，所有 API 节点和所有管理节点。这意味着，在具有 2 个数据节点，2 个管理节点和每个数据节点连接到 10 个 API 节点的集群中，每个数据节点有 13 个节点连接。由于数据节点连接的`SendBufferMemory`的默认值为 2 兆字节，因此总计为 26 MB。

1.  要获取发送缓冲区的总保留大小，需要将`ExtraSendBufferMemory`配置参数的值（如果有，默认值为 0）与前一步得到的值相加。

换句话说，如果已设置`TotalSendBufferMemory`，则发送缓冲区大小为`TotalSendBufferMemory + ExtraSendBufferMemory`；否则，发送缓冲区的大小等于`([*节点连接数*] * SendBufferMemory) + ExtraSendBufferMemory`。

用于磁盘数据记录的页面缓存仅使用保留资源；此资源的大小由`DiskPageBufferMemory`配置参数控制（默认为 64 MB）。还分配了 32 KB 磁盘页面条目的内存；其数量由`DiskPageBufferEntries`配置参数确定（默认为 10）。

事务内存有一个保留部分，可以由`NDB`计算，也可以使用在 NDB 8.0 中引入的`TransactionMemory`配置参数显式设置（以前，此值始终由`NDB`计算）；事务内存还可以使用无限量的共享全局内存。事务内存用于处理所有操作资源，包括事务、扫描、锁定、扫描缓冲区和触发器操作。它还在更新表行时保存表行，然后在下一次提交时将它们写入数据内存。

以前，操作记录使用专用资源，其大小由多个配置参数控制。在 NDB 8.0 中，所有这些资源都从一个公共事务内存资源分配，并且还可以使用全局共享内存资源。可以使用单个`TransactionMemory`配置参数来控制此资源的大小。

保留用于撤销日志缓冲区的内存可以使用`InitialLogFileGroup`配置参数进行设置。如果撤销日志缓冲区是作为`CREATE LOGFILE GROUP` SQL 语句的一部分创建的，则内存来自事务内存。

与磁盘数据资源相关的一些元数据资源也没有保留部分，仅使用共享全局内存。因此，共享全局共享内存在发送缓冲区、事务内存和磁盘数据元数据之间共享。

如果未设置`TransactionMemory`，则根据以下参数计算：

+   `MaxNoOfConcurrentOperations`

+   `MaxNoOfConcurrentTransactions`

+   `MaxNoOfFiredTriggers`

+   `MaxNoOfLocalOperations`

+   `MaxNoOfConcurrentIndexOperations`

+   `MaxNoOfConcurrentScans`

+   `MaxNoOfLocalScans`

+   `BatchSizePerLocalScan`

+   `TransactionBufferMemory`

当显式设置`TransactionMemory`时，上述配置参数都不用于计算内存大小。此外，参数`MaxNoOfConcurrentIndexOperations`、`MaxNoOfFiredTriggers`、`MaxNoOfLocalOperations`和`MaxNoOfLocalScans`与`TransactionMemory`不兼容，不能与之同时设置；如果在`config.ini`配置文件中设置了`TransactionMemory`并且这四个参数中的任何一个也被设置，管理服务器将无法启动。*注意*：在 NDB 8.0.29 之前，对于`MaxNoOfFiredTriggers`、`MaxNoOfLocalScans`或`MaxNoOfLocalOperations`，这个限制并未强制执行（Bug #102509，Bug #32474988）。

`MaxNoOfConcurrentIndexOperations`、`MaxNoOfFiredTriggers`、`MaxNoOfLocalOperations`和`MaxNoOfLocalScans`参数在 NDB 8.0 中已弃用；您应该期望它们在将来的 MySQL NDB Cluster 版本中被移除。

在 NDB 8.0.29 之前，不可能同时设置`MaxNoOfConcurrentTransactions`、`MaxNoOfConcurrentOperations`或`MaxNoOfConcurrentScans`与`TransactionMemory`。

事务内存资源包含大量内存池。每个内存池代表一个对象类型，并包含一组对象；每个池在启动时包括一个分配给池的保留部分；这些保留内存永远不会返回到共享全局内存。使用仅具有单个级别的数据结构来查找保留记录，以便快速检索，这意味着每个池中应该保留一定数量的记录。每个池中保留的记录数量对性能和保留内存分配有一定影响，但通常只在某些非常高级的用例中需要显式设置保留大小。

可通过设置以下配置参数来控制池的保留部分的大小：

+   `ReservedConcurrentIndexOperations`

+   `ReservedFiredTriggers`

+   `ReservedConcurrentOperations`

+   `ReservedLocalScans`

+   `ReservedConcurrentTransactions`

+   `ReservedConcurrentScans`

+   `ReservedTransactionBufferMemory`

对于上述列出的任何未在`config.ini`中明确设置的参数，保留设置将计算为相应最大设置的 25%。例如，如果未设置，`ReservedConcurrentIndexOperations`将计算为`MaxNoOfConcurrentIndexOperations`的 25%，`ReservedLocalScans`将计算为`MaxNoOfLocalScans`的 25%。

注意

如果未设置`ReservedTransactionBufferMemory`，则将计算为`TransactionBufferMemory`的 25%。

保留记录数是每个数据节点的；这些记录在每个节点上处理它们的线程（LDM 和 TC 线程）之间分割。在大多数情况下，仅设置`TransactionMemory`就足够了，并且允许池中的记录数由其值控制。

`MaxNoOfConcurrentScans` 限制每个 TC 线程中可以活动的并发扫描数。这对防止集群过载很重要。

`MaxNoOfConcurrentOperations` 限制可以同时活动的更新事务中的操作数。（简单读取不受此参数影响。）需要限制此数字，因为需要为节点故障处理预分配内存，并且在处理最大数量的活动操作时必须为处理节点故障时一个 TC 线程中的最大数量的活动操作提供资源。`MaxNoOfConcurrentOperations` 必须在所有节点上设置为相同的数字（最简单的方法是在`config.ini`全局配置文件的`[ndbd default]`部分中一次设置它的值）。虽然可以使用滚动重启（参见第 25.6.5 节“执行 NDB Cluster 的滚动重启”）来增加其值，但由于在滚动重启期间可能发生节点故障的可能性，不建议以这种方式减少其值。

可以通过`MaxDMLOperationsPerTransaction`参数限制 NDB Cluster 中单个事务的大小。如果未设置此参数，则一个事务的大小受`MaxNoOfConcurrentOperations`限制，因为此参数限制每个 TC 线程的并发操作总数。

模式内存大小由以下一组配置参数控制：

+   `MaxNoOfSubscriptions`

+   `MaxNoOfSubscribers`

+   `MaxNoOfConcurrentSubOperations`

+   `MaxNoOfAttributes`

+   `MaxNoOfTables`

+   `MaxNoOfOrderedIndexes`

+   `MaxNoOfUniqueHashIndexes`

+   `MaxNoOfTriggers`

节点数和 LDM 线程数也对模式内存大小产生重大影响，因为每个表和每个分区（及其片段副本）的分区数必须在模式内存中表示。

此外，在启动期间还分配了一些其他记录。这些记录相对较小。每个线程中的每个块包含使用内存的块对象。与其他数据节点内存结构相比，这种内存大小通常也相当小。
