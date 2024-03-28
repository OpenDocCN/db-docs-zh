> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-params-ndbd.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-params-ndbd.html)

#### 25.4.2.1 NDB Cluster Data Node Configuration Parameters

本节中的列表提供有关在`config.ini`文件的`[ndbd]`或`[ndbd default]`部分中用于配置 NDB Cluster 数据��点的参数的信息。有关每个参数的详细描述和其他附加信息，请参见 Section 25.4.3.6, “Defining NDB Cluster Data Nodes”。

这些参数也适用于**ndbmtd**")，即**ndbd**的多线程版本。接下来是针对**ndbmtd**")的特定参数列表。

+   `Arbitration`: 在节点故障事件中避免脑裂问题时应如何执行仲裁。

+   `ArbitrationTimeout`: 数据库分区等待仲裁信号的最长时间（毫秒）。

+   `BackupDataBufferSize`: 备份数据缓冲区的默认大小（以字节为单位）。

+   `BackupDataDir`: 存储备份的路径。请注意，该设置始终附加字符串'/BACKUP'，因此*有效*默认值为 FileSystemPath/BACKUP。

+   `BackupDiskWriteSpeedPct`: 设置数据节点分配的最大写入速度（MaxDiskWriteSpeed）的百分比，用于在启动备份时为 LCPs 保留。

+   `BackupLogBufferSize`: 备份日志缓冲区的默认大小（以字节为单位）。

+   `BackupMaxWriteSize`: 备份所做的文件系统写入的最大大小（以字节为单位）。

+   `BackupMemory`: 每个节点为备份分配的总内存（以字节为单位）。

+   `BackupReportFrequency`: 备份期间备份状态报告的频率（以秒为单位）。

+   `BackupWriteSize`: 备份所做的文件系统写入的默认大小（以字节为单位）。

+   `BatchSizePerLocalScan`: 用于计算保持锁扫描的锁记录数。

+   `BuildIndexThreads`: 在系统或节点重新启动期间用于构建有序索引的线程数。在运行 ndb_restore --rebuild-indexes 时也适用。将此参数设置为 0 会禁用多线程构建有序索引。

+   `CompressedBackup`: 在写入时使用 zlib 压缩备份。

+   `CompressedLCP`: 使用 zlib 编写压缩的 LCP。

+   `ConnectCheckIntervalDelay`: 数据节点连接性检查阶段之间的时间间隔。如果在 1 个间隔后没有响应，则数据节点被视为可疑，在 2 个间隔后被视为死亡。

+   `CrashOnCorruptedTuple`: 启用后，每当检测到损坏的元组时，强制节点关闭。

+   `DataDir`: 此节点的数据目录。

+   `DataMemory`: 每个数据节点为存储数据分配的字节数；取决于可用系统 RAM 和 IndexMemory 的大小。

+   `DefaultHashMapSize`: 设置用于表哈希映射的大小（桶数）。支持三个值：0、240 和 3840。

+   `DictTrace`: 启用 DBDICT 调试；用于 NDB 开发。

+   `DiskDataUsingSameDisk`: 如果磁盘数据表空间位于不同物理磁盘上，则设置为 false。

+   `DiskIOThreadPool`: 用于文件访问的未绑定线程数��仅适用于磁盘数据。

+   `Diskless`: 在不使用磁盘的情况下运行。

+   `DiskPageBufferEntries`: 在 DiskPageBufferMemory 中分配的内存；非常大的磁盘事务可能需要增加此值。

+   `DiskPageBufferMemory`: 每个数据节点为磁盘页缓冲区缓存分配的字节数。

+   `DiskSyncSize`: 在强制同步之前写入文件的数据量。

+   `EnablePartialLcp`: 启用部分 LCP（true）；如果禁用（false），则所有 LCP 都写入完整检查点。

+   `EnableRedoControl`: 启用用于控制重做日志使用情况的自适应检查点速度。

+   `EncryptedFileSystem`: 加密本地检查点和表空间文件。实验性功能；不支持生产环境。

+   `EventLogBufferSize`: 数据节点内用于 NDB 日志事件的循环缓冲区大小。

+   `ExecuteOnComputer`: 引用先前定义的计算机的字符串。

+   `ExtraSendBufferMemory`: 用于发送缓冲区的内存，额外分配给 TotalSendBufferMemory 或 SendBufferMemory。默认值（0）允许最多 16MB。

+   `FileSystemPath`: 数据节点存储数据的目录路径（目录必须存在）。

+   `FileSystemPathDataFiles`: 数据节点存储其磁盘数据文件的目录路径。默认���为 FilesystemPathDD（如果已设置）；否则，如果已设置 FilesystemPath，则使用该值；否则，使用 DataDir 的值。

+   `FileSystemPathDD`: 数据节点存储其磁盘数据和撤销文件的目录路径。默认值为 FileSystemPath（如果已设置）；否则，使用 DataDir 的值。

+   `FileSystemPathUndoFiles`: 数据节点为磁盘数据存储撤销文件的目录路径。默认值为 FilesystemPathDD（如果已设置）；否则，如果已设置 FilesystemPath，则使用该值；否则，使用 DataDir 的值。

+   `FragmentLogFileSize`: 每个重做日志文件的大小。

+   `HeartbeatIntervalDbApi`: API 节点与数据节点之间心跳的时间间隔。（连续错过 3 次心跳后，API 连接关闭）。

+   `HeartbeatIntervalDbDb`: 数据节点之间心跳的时间间隔；如果连续错过 3 次心跳，则数据节点被视为已死亡。

+   `HeartbeatOrder`: 设置数据节点检查彼此心跳的顺序，以确定给定节点是否仍处于活动状态并连接到集群。必须对所有数据节点为零，或对所有数据节点为不同的非零值；有关更多指导，请参阅文档。

+   `HostName`: 此数据节点的主机名或 IP 地址。

+   `IndexMemory`: 每个数据节点用于存储索引的字节数；取决于可用系统 RAM 和 DataMemory 的大小。

+   `IndexStatAutoCreate`: 在创建索引时启用/禁用自动统计信息收集。

+   `IndexStatAutoUpdate`: 监视索引的更改并触发自动统计信息更新。

+   `IndexStatSaveScale`: 用于确定存储的索引统计信息大小的缩放因子。

+   `IndexStatSaveSize`: 每个索引保存的统计信息的最大字节数。

+   `IndexStatTriggerPct`: DML 操作的索引统计信息更新的阈值百分比变化。该值由 IndexStatTriggerScale 缩小。

+   `IndexStatTriggerScale`: 将 IndexStatTriggerPct 按照此数量缩小，乘以索引大小的以 2 为底的对数，用于大型索引。设置为 0 以禁用缩放。

+   `IndexStatUpdateDelay`: 给定索引的自动索引统计信息更新之间的最小延迟。0 表示没��延迟。

+   `InitFragmentLogFiles`: 使用稀疏或完整格式初始化片段日志文件。

+   `InitialLogFileGroup`: 描述在初始启动期间创建的日志文件组。查看文档以获取格式。

+   `InitialNoOfOpenFiles`: 每个数据节点初始打开文件数。（每个文件创建一个线程）。

+   `InitialTablespace`: 描述在初始启动期间创建的表空间。查看文档以获取格式。

+   `InsertRecoveryWork`: 用于插入行的 RecoveryWork 百分比；除非使用部分本地检查点，否则不起作用。

+   `KeepAliveSendInterval`: 数据节点之间链接发送保持活动信号的时间间隔，以毫秒为单位。设置为 0 以禁用。

+   `LateAlloc`: 在与管理服务器建立连接后分配内存。

+   `LcpScanProgressTimeout`: 本地检查点片段扫描在节点关闭之前可以停滞的最长时间，以确保系统范围内的 LCP 进度。使用 0 来禁用。

+   `LocationDomainId`: 将此数据节点分配给特定的可用域或区域。0（默认）表示未设置。

+   `LockExecuteThreadToCPU`: 逗号分隔的 CPU ID 列表。

+   `LockMaintThreadsToCPU`: 指示哪个 CPU 运行维护线程的 CPU ID。

+   `LockPagesInMainMemory`: 0=禁用锁定，1=内存分配后锁定，2=内存分配前锁定。

+   `LogLevelCheckpoint`: 本地和全局检查点信息的日志级别打印到标准输出。

+   `LogLevelCongestion`: 拥塞信息打印到标准输出的级别。

+   `LogLevelConnection`: 节点连接/断开信息打印到标准输出的级别。

+   `LogLevelError`: 传输器、心跳错误打印到标准输出。

+   `LogLevelInfo`: 心跳和日志信息打印到标准输出。

+   `LogLevelNodeRestart`: 节点重新启动和节点故障信息打印到标准输出的级别。

+   `LogLevelShutdown`: 节点关闭信息打印到标准输出的级别。

+   `LogLevelStartup`: 节点启动信息打印到标准输出的级别。

+   `LogLevelStatistic`: 事务、操作和传输器信息打印到标准输出的级别。

+   `LongMessageBuffer`: 每个数据节点为内部长消息分配的字节数。

+   `MaxAllocate`: 不再使用；没有效果。

+   `MaxBufferedEpochs`: 订阅节点可以滞后的允许的时代编号（未处理时代）数量。超过会导致滞后的订阅者被断开连接。

+   `MaxBufferedEpochBytes`: 为缓冲时代分配的总字节数。

+   `MaxDiskDataLatency`: 在开始中止事务之前，允许的磁盘访问平均延迟（毫秒）的最大值。

+   `MaxDiskWriteSpeed`: 当没有重新启动时，LCP 和备份每秒可写入的最大字节数。

+   `MaxDiskWriteSpeedOtherNodeRestart`: 当另一个节点重新启动时，LCP 和备份每秒可以写入的最大字节数。

+   `MaxDiskWriteSpeedOwnRestart`: 当此节点重新启动时，LCP 和备份每秒可以写入的最大字节数。

+   `MaxFKBuildBatchSize`: 用于构建外键的最大扫描批处理大小。增加此值可能加快外键构建速度，但也会影响正在进行的流量。

+   `MaxDMLOperationsPerTransaction`: 事务的限制大小；如果需要的 DML 操作超过此数量，则中止事务。

+   `MaxLCPStartDelay`: LCP 在检查检查点互斥锁（以允许其他数据节点完成元数据同步）之前轮询的时间（以便将自身放入锁队列，以便并行恢复表数据）的秒数。

+   `MaxNoOfAttributes`: 建议存储在数据库中���属性总数（所有表的总和）。

+   `MaxNoOfConcurrentIndexOperations`: 在一个数据节点上可以同时执行的索引操作的总数。

+   `MaxNoOfConcurrentOperations`: 事务协调器中操作记录的最大数量。

+   `MaxNoOfConcurrentScans`: 数据节点上同时执行的最大扫描数。

+   `MaxNoOfConcurrentSubOperations`: 并发订阅者操作的最大数量。

+   `MaxNoOfConcurrentTransactions`: 在此数据节点上同时执行的最大事务数，可以同时执行的事务总数为此值乘以集群中数据节点的数量。

+   `MaxNoOfFiredTriggers`: 在一个数据节点上可以同时触发的触发器总数。

+   `MaxNoOfLocalOperations`: 在此数据节点上定义的操作记录的最大数量。

+   `MaxNoOfLocalScans`: 在此数据节点上并行进行的片段扫描的最大数量。

+   `MaxNoOfOpenFiles`: 每个数据节点打开的文件的最大数量。（每个文件创建一个线程）。

+   `MaxNoOfOrderedIndexes`: 系统中可定义的有序索引总数。

+   `MaxNoOfSavedMessages`: 写入错误日志的最大错误消息数和保留的最大跟踪文件数。

+   `MaxNoOfSubscribers`: 订阅者的最大数量。

+   `MaxNoOfSubscriptions`: 订阅的最大数量（默认为 0 = MaxNoOfTables）。

+   `MaxNoOfTables`: 建议存储在数据库中的 NDB 表的总数。

+   `MaxNoOfTriggers`: 系统中可定义的触发器总数。

+   `MaxNoOfUniqueHashIndexes`: 系统中可定义的唯一哈希索引总数。

+   `MaxParallelCopyInstances`: 节点重启期间的并行复制数。默认值为 0，使用两个节点上的 LDM 数，最多为 16。

+   `MaxParallelScansPerFragment`: 每个片段的最大并行扫描数。一旦达到此限制，扫描将被串行化。

+   `MaxReorgBuildBatchSize`: 重新组织表分区时使用的最大扫描批量大小。增加此值可能加快表分区重新组织的速度，但也会影响正在进行的流量。

+   `MaxStartFailRetries`: 数据节点启动失败时的最大重试次数，要求 StopOnError = 0。将其设置为 0 会导致启动尝试无���继续。

+   `MaxUIBuildBatchSize`: 用于构建唯一键的最大扫描批量大小。增加此值可能加快唯一键的构建速度，但也会影响正在进行的流量。

+   `MemReportFrequency`: 内存报告的频率（以秒为单位）；0 = 仅在超过百分比限制时报告。

+   `MinDiskWriteSpeed`: LCP 和备份每秒可写入的最小字节数。

+   `MinFreePct`: 保留用于重启的内存资源百分比。

+   `NodeGroup`: 数据节点所属的节点组；仅在集群的初始启动期间使用。

+   `NodeGroupTransporters`: 在同一节点组中节点之间使用的传输器数量。

+   `NodeId`: 在集群中唯一标识数据节点的编号。

+   `NoOfFragmentLogFiles`: 每个数据节点所属的 4 个文件集中每个 16 MB 重做日志文件的数量。

+   `NoOfReplicas`: 数据库中所有数据的副本数量。

+   `Numa`: （仅限 Linux；需要 libnuma）控制 NUMA 支持。将其设置为 0 允许系统确定数据节点进程的交错使用；设置为 1 表示由数据节点确定。

+   `ODirect`: 尽可能使用 O_DIRECT 文件读取和写入。

+   `ODirectSyncFlag`: O_DIRECT 写入被视为同步写入；当 ODirect 未启用、InitFragmentLogFiles 设置为 SPARSE 或两者都设置时将被忽略。

+   `RealtimeScheduler`: 当为 true 时，数据节点线程被调度为实时线程。默认值为 false。

+   `RecoveryWork`: LCP 文件的存储开销百分比：较大的值意味着在正常操作中工作较少，在恢复期间工作较多。

+   `RedoBuffer`: 每个数据节点用于写入重做日志的字节数。

+   `RedoOverCommitCounter`: 当超过 RedoOverCommitLimit 这么多次时，事务将被中止，并且操作将按照 DefaultOperationRedoProblemAction 指定的方式处理。

+   `RedoOverCommitLimit`: 每次刷新当前重做缓冲区所需时间超过这么多秒时，将比较已发生的次数与 RedoOverCommitCounter。

+   `RequireEncryptedBackup`: 备份是否必须加密（1 = 需要加密，否则为 0）。

+   `ReservedConcurrentIndexOperations`: 在一个数据节点上具有专用资源的同时索引操作的数量。

+   `ReservedConcurrentOperations`: 在一个数据节点上具有专用资源的事务协调器中同时操作的数量。

+   `保留并发扫描数`: 在一个数据节点上具有专用资源的同时扫描数。

+   `保留并发事务数`: 在一个数据节点上具有专用资源的同时事务数。

+   `保留触发器数`: 在一个数据节点上具有专用资源的触发器数。

+   `保留本地扫描数`: 在一个数据节点上具有专用资源的同时片段扫描数。

+   `保留事务缓冲区内存`: 分配给每个数据节点的键和属性数据的动态缓冲区空间（以字节为单位）。

+   `插入错误重启控制`: 插入错误导致的重启控制类型（当启用 StopOnError 时）。

+   `重启订阅者连接超时`: 数据节点等待订阅 API 节点连接的时间。设置为 0 以禁用超时，总是解析为最接近的整秒。

+   `调度执行计时器`: 在发送之前执行调度器的微秒数。

+   `调度器响应性`: 设置 NDB 调度器响应优化 0-10；较高的值提供更好的响应时间但降低吞吐量。

+   `调度器自旋计时器`: 在休眠之前执行调度器的微秒数。

+   `服务器端口`: 用于为来自 API 节点的传入连接设置传输器的端口。

+   `共享全局内存`: 为任何用途在每个数据节点上分配的总字节数。

+   `自旋方法`: 数据节点使用的自旋方法；详细信息请参阅文档。

+   `启动失败重试延迟`: 启动失败后重试之前的延迟时间（需要 StopOnError = 0）。

+   `启动失败超时`: 在终止之前等待的毫秒数（0=永远等待）。

+   `无节点组超时启动`: 在尝试启动之前等待无节点组的时间（0=永远）。

+   `StartPartialTimeout`: 尝试在所���节点都不完整的情况下启动之前等待的毫秒数。（0=永远等待）。

+   `StartPartitionedTimeout`: 尝试启动分区之前等待的毫秒数。（0=永远等待）。

+   `StartupStatusReportFrequency`: 启动过程中状态报告的频率。

+   `StopOnError`: 当设置为 0 时，数据节点会在节点故障后自动重新启动和恢复。

+   `StringMemory`: 字符串内存的默认大小（0 到 100 = 最大值的百分比，101+ = 实际字节数）。

+   `TcpBind_INADDR_ANY`: 绑定 IP_ADDR_ANY，以便可以从任何地方进行连接（用于自动生成的连接）。

+   `TimeBetweenEpochs`: 各个时期之间的时间间隔（用于复制同步）。

+   `TimeBetweenEpochsTimeout`: 时期之间的超时时间。超时会导致节点关闭。

+   `TimeBetweenGlobalCheckpoints`: 将事务组提交到磁盘之间的时间间隔。

+   `TimeBetweenGlobalCheckpointsTimeout`: 事务组提交到磁盘的最小超时时间。

+   `TimeBetweenInactiveTransactionAbortCheck`: 检查非活动事务之间的时间间隔。

+   `TimeBetweenLocalCheckpoints`: 对数据库进行快照的时间间隔（以字节的基数-2 对数表示）。

+   `TimeBetweenWatchDogCheck`: 数据节点内部执行检查之间的时间间隔。

+   `TimeBetweenWatchDogCheckInitial`: 数据节点内部执行检查之间的时间间隔（在分配内存时的早期阶段）。

+   `TotalSendBufferMemory`: 用于所有传输器发送缓冲区的总内存。

+   `TransactionBufferMemory`: 为每个数据节点分配的用于键和属性数据的动态缓冲空间（以字节为单位）。

+   `TransactionDeadlockDetectionTimeout`: 事务在数据节点内执行的时间。这是事务协调器等待每个参与事务的数据节点执行请求的时间。如果数据节点花费的时间超过这个时间，事务将被中止。

+   `TransactionInactiveTimeout`: 应用程序在执行事务的另一部分之前等待的毫秒数。这是事务协调器等待应用程序执行或发送事务的另一部分（查询、语句）的时间。如果应用程序花费太长时间，则事务将被���止。超时 = 0 表示应用程序永远不会超时。

+   `TransactionMemory`: 每个数据节点上为事务分配的内存。

+   `TwoPassInitialNodeRestartCopy`: 在初始节点重新启动期间进行 2 次数据复制，从而实现有序索引的多线程构建。

+   `UndoDataBuffer`: 未使用；没有效果。

+   `UndoIndexBuffer`: 未使用；没有效果。

+   `UseShm`: 在此数据节点和在同一主机上运行的 API 节点之间使用共享内存连接。

+   `WatchDogImmediateKill`: 当为真时，线程在发生看门狗问题时立即被终止；用于测试和调试。

以下参数特定于**ndbmtd**"):

+   `AutomaticThreadConfig`: 使用自动线程配置；覆盖任何 ThreadConfig 和 MaxNoOfExecutionThreads 的设置，并禁用 ClassicFragmentation。

+   `ClassicFragmentation`: 当为真时，使用传统表碎片化；设置为假以在 LDM 之间灵活分配碎片。由 AutomaticThreadConfig 禁用。

+   `EnableMultithreadedBackup`: 启用多线程备份。

+   `MaxNoOfExecutionThreads`: 仅对 ndbmtd，指定最大执行线程数。

+   `MaxSendDelay`: ndbmtd 发送的最大延迟微秒数。

+   `NoOfFragmentLogParts`: 属于此数据节点的重做日志文件组数。

+   `NumCPUs`: 指定与 AutomaticThreadConfig 一起使用的 CPU 数量。

+   `PartitionsPerNode`: 确定在每个数据节点上创建的表分区数；如果启用了 ClassicFragmentation，则不使用。

+   `ThreadConfig`: 用于配置多线程数据节点（ndbmtd）。默认为空字符串；请参阅文档以获取语法和其他信息。
