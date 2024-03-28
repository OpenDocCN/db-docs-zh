# 25.2.5 NDB 8.0 中新增、弃用或移除的选项、变量和参数

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-added-deprecated-removed.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-added-deprecated-removed.html)

+   NDB 8.0 中引入的参数

+   NDB 8.0 中弃用的参数

+   NDB 8.0 中移除的参数

+   NDB 8.0 中引入的选项和变量

+   NDB 8.0 中弃用的选项和变量

+   NDB 8.0 中移除的选项和变量

接下来的几节包含有关 `NDB` 节点配置参数和 NDB 特定的 **mysqld** 选项和变量的信息，这些信息已经在 NDB 8.0 中新增、弃用或移除。

#### NDB 8.0 中引入的参数

以下是在 NDB 8.0 中新增的节点配置参数。

+   `AllowUnresolvedHostNames`: 当为 false（默认）时，管理节点无法解析主机名会导致致命错误；当为 true 时，未解析的主机名仅报告警告。NDB 8.0.22 中添加。

+   `AutomaticThreadConfig`: 使用自动线程配置；覆盖任何 ThreadConfig 和 MaxNoOfExecutionThreads 的设置，并禁用 ClassicFragmentation。NDB 8.0.23 中添加。

+   `ClassicFragmentation`: 当为 true 时，使用传统表碎片化；设置为 false 以启用碎片在 LDM 之间的灵活分布。被 AutomaticThreadConfig 禁用。NDB 8.0.23 中添加。

+   `DiskDataUsingSameDisk`: 如果磁盘数据表空间位于不同物理磁盘上，则设置为 false。NDB 8.0.19 中添加。

+   `EnableMultithreadedBackup`: 启用多线程备份。NDB 8.0.16 中添加。

+   `EncryptedFileSystem`: 加密本地检查点和表空间文件。实验性功能；不支持生产环境使用。NDB 8.0.29 中添加。

+   `KeepAliveSendInterval`: 数据节点之间链接上保持活动信号之间的时间间隔，以毫秒为单位。设置为 0 以禁用。在 NDB 8.0.27 中添加。

+   `MaxDiskDataLatency`: 允许的磁盘访问平均延迟的最大值（毫秒），在开始中止事务之前。在 NDB 8.0.19 中添加。

+   `NodeGroupTransporters`: 同一节点组中节点之间使用的传输器数量。在 NDB 8.0.20 中添加。

+   `NumCPUs`: 指定与 AutomaticThreadConfig 一起使用的 CPU 数量。在 NDB 8.0.23 中添加。

+   `PartitionsPerNode`: 确定在每个数据节点上创建的表分区数；如果启用了 ClassicFragmentation，则不使用。在 NDB 8.0.23 中添加。

+   `PreferIPVersion`: 指示 IP 版本 4 或 6 的 DNS 解析器首选项。在 NDB 8.0.26 中添加。

+   `RequireEncryptedBackup`: 备份是否必须加密（1 = 需要加密，否则为 0）。在 NDB 8.0.22 中添加。

+   `ReservedConcurrentIndexOperations`: 在一个数据节点上具有专用资源的同时索引操作数量。在 NDB 8.0.16 中添加。

+   `ReservedConcurrentOperations`: 在一个数据节点上事务协调器中具有专用资源的同时操作数量。在 NDB 8.0.16 中添加。

+   `ReservedConcurrentScans`: 在一个数据节点上具有专用资源的同时扫描数量。在 NDB 8.0.16 中添加。

+   `ReservedConcurrentTransactions`: 在一个数据节点上具有专用资源的同时事务数量。在 NDB 8.0.16 中添加。

+   `ReservedFiredTriggers`: 在一个数据节点上具有专用资源的触发器数量。在 NDB 8.0.16 中添加。

+   `ReservedLocalScans`: 在一个数据节点上具有专用资源的同时片段扫描数量。在 NDB 8.0.16 中添加。

+   `ReservedTransactionBufferMemory`: 分配给每个数据节点的键和属性数据的动态缓冲空间（以字节为单位）。在 NDB 8.0.16 中添加。

+   `SpinMethod`: 确定数据节点使用的旋转方法；详细信息请参阅文档。在 NDB 8.0.20 中添加。

+   `TcpSpinTime`: 在接收时进入休眠之前旋转的时间。在 NDB 8.0.20 中添加。

+   `TransactionMemory`: 每个数据节点上为事务分配的内存。在 NDB 8.0.19 中添加。

#### 在 NDB 8.0 中已弃用的参数

在 NDB 8.0 中已弃用以下节点配置参数。

+   `BatchSizePerLocalScan`: 用于计算具有保持锁的扫描的锁记录数量。在 NDB 8.0.19 中已弃用。

+   `MaxAllocate`: 不再使用；没有效果。在 NDB 8.0.27 中已弃用。

+   `MaxNoOfConcurrentIndexOperations`: 在一个数据节点上可以同时执行的索引操作的总数。在 NDB 8.0.19 中已弃用。

+   `MaxNoOfConcurrentTransactions`: 此数据节点上同时执行的事务数的最大值，可以同时执行的事务总数是此值乘以集群中数据节点的数量。在 NDB 8.0.19 中已弃用。

+   `MaxNoOfFiredTriggers`: 在一个数据节点上可以同时触发的触发器总数。在 NDB 8.0.19 中已弃用。

+   `MaxNoOfLocalOperations`: 此数据节点上定义的操作记录的最大数量。在 NDB 8.0.19 中已弃用。

+   `MaxNoOfLocalScans`: 此数据节点上并行片段扫描的最大数量。在 NDB 8.0.19 中已弃用。

+   `ReservedTransactionBufferMemory`: 为每个数据节点分配的用于键和属性数据的动态缓冲空间（以字节为单位）。在 NDB 8.0.19 中已弃用。

+   `UndoDataBuffer`: 未使用；没有效果。在 NDB 8.0.27 中已弃用。

+   `UndoIndexBuffer`: 未使用；没有效果。在 NDB 8.0.27 中已弃用。

#### 在 NDB 8.0 中删除的参数

在 NDB 8.0 中没有删除任何节点配置参数。

#### 在 NDB 8.0 中引入的选项和变量

在 NDB 8.0 中已添加以下系统变量、状态变量和服务器选项。

+   `Ndb_api_adaptive_send_deferred_count_replica`: 该副本未实际发送的自适应发送调用次数。NDB 8.0.23 中添加。

+   `Ndb_api_adaptive_send_forced_count_replica`: 该副本发送的强制发送的自适应发送次数。NDB 8.0.23 中添加。

+   `Ndb_api_adaptive_send_unforced_count_replica`: 该副本发送的非强制发送的自适应发送次数。NDB 8.0.23 中添加。

+   `Ndb_api_bytes_received_count_replica`: 该副本从数据节点接收的数据量（以字节为单位）。NDB 8.0.23 中添加。

+   `Ndb_api_bytes_sent_count_replica`: 该副本发送到数据节点的数据量（以字节为单位）。NDB 8.0.23 中添加。

+   `Ndb_api_pk_op_count_replica`: 该副本基于或使用主键��操作次数。NDB 8.0.23 中添加。

+   `Ndb_api_pruned_scan_count_replica`: 该副本将扫描修剪为一个分区的次数。NDB 8.0.23 中添加。

+   `Ndb_api_range_scan_count_replica`: 该副本启动的范围扫描次数。NDB 8.0.23 中添加。

+   `Ndb_api_read_row_count_replica`: 该副本读取的总行数。NDB 8.0.23 中添加。

+   `Ndb_api_scan_batch_count_replica`: 该副本接收的行批次数。NDB 8.0.23 中添加。

+   `Ndb_api_table_scan_count_replica`: 该副本启动的表扫描次数，包括内部表的扫描。NDB 8.0.23 中添加。

+   `Ndb_api_trans_abort_count_replica`: 该副本中事务被中止的次数。NDB 8.0.23 中添加。

+   `Ndb_api_trans_close_count_replica`: 该副本中事务被中止的次数（可能大于 TransCommitCount 和 TransAbortCount 之和）。NDB 8.0.23 中添加。

+   `Ndb_api_trans_commit_count_replica`: 该副本提交的事务数。NDB 8.0.23 中添加。

+   `Ndb_api_trans_local_read_row_count_replica`: 此副本已读取的总行数。在 NDB 8.0.23 中添加。

+   `Ndb_api_trans_start_count_replica`: 此副本启动的事务次数。在 NDB 8.0.23 中添加。

+   `Ndb_api_uk_op_count_replica`: 此副本基于或使用唯一键的操作次数。在 NDB 8.0.23 中添加。

+   `Ndb_api_wait_exec_complete_count_replica`: 等待此副本操作执行完成而被阻塞的线程次数。在 NDB 8.0.23 中添加。

+   `Ndb_api_wait_meta_request_count_replica`: 等待此副本通过元数据信号而被阻塞的线程次数。在 NDB 8.0.23 中添加。

+   `Ndb_api_wait_nanos_count_replica`: 此副本等待数据节点发出某种类型信号的总时间（以纳秒为单位）。在 NDB 8.0.23 中添加。

+   `Ndb_api_wait_scan_result_count_replica`: 等待此副本通过扫描信号而被阻塞的线程次数。在 NDB 8.0.23 中添加。

+   `Ndb_config_generation`: 集群当前配置的生成编号。在 NDB 8.0.24 中添加。

+   `Ndb_conflict_fn_max_del_win_ins`: 基于 NDB$MAX_DEL_WIN_INS()结果的冲突解决机制在插入操作中被应用的次数。在 NDB 8.0.30 中添加。

+   `Ndb_conflict_fn_max_ins`: "greater timestamp wins"冲突解决机制在插入操作中被应用的次数。在 NDB 8.0.30 中添加。

+   `Ndb_metadata_blacklist_size`: NDB 二进制日志线程未能同步的 NDB 元数据对象数量；在 NDB 8.0.22 中更名为 Ndb_metadata_excluded_count。在 NDB 8.0.18 中添加。

+   `Ndb_metadata_detected_count`: NDB 元数据更改监视线程检测到更改的次数。在 NDB 8.0.16 中添加。

+   `Ndb_metadata_excluded_count`: NDB 二进制日志线程未能同步的 NDB 元数据对象数量。在 NDB 8.0.22 中添加。

+   `Ndb_metadata_synced_count`: 已同步的 NDB 元数据对象数量。在 NDB 8.0.18 中添加。

+   `Ndb_trans_hint_count_session`: 在此会话中启动的使用提示的事务数量。在 NDB 8.0.17 中添加。

+   `ndb-applier-allow-skip-epoch`: 允许复制应用程序跳过时代。在 NDB 8.0.28 中添加。

+   `ndb-log-fail-terminate`: 如果无法完全记录所有找到的行事件，则终止 mysqld 进程。在 NDB 8.0.21 中添加。

+   `ndb-log-transaction-dependency`: 使二进制日志线程为写入二进制日志的每个事务计算事务依赖关系。在 NDB 8.0.33 中添加。

+   `ndb-schema-dist-timeout`: 在检测到模式分发超时之前等待多长时间。在 NDB 8.0.17 中添加。

+   `ndb_conflict_role`: 副本在冲突检测和解决中扮��的角色。值为 PRIMARY、SECONDARY、PASS 或 NONE（默认）。只能在停止复制 SQL 线程时更改。有关更多信息，请参阅文档。在 NDB 8.0.23 中添加。

+   `ndb_dbg_check_shares`: 检查任何残留的共享（仅限调试构建）。在 NDB 8.0.13 中添加。

+   `ndb_log_transaction_compression`: 是否压缩 NDB 二进制日志；也可以通过启用--binlog-transaction-compression 选项在启动时启用。在 NDB 8.0.31 中添加。

+   `ndb_log_transaction_compression_level_zstd`: 写入压缩事务到 NDB 二进制日志时要使用的 ZSTD 压缩级别。在 NDB 8.0.31 中添加。

+   `ndb_metadata_check`: 启用自动检测 NDB 元数据与 MySQL 数据字典的更改；默认启用。在 NDB 8.0.16 中添加。

+   `ndb_metadata_check_interval`: 每隔多少秒执行一次检查 NDB 元数据与 MySQL 数据字典的更改。在 NDB 8.0.16 中添加。

+   `ndb_metadata_sync`: 触发立即同步 NDB 字典和 MySQL 数据字典之间的所有更改；导致忽略 ndb_metadata_check 和 ndb_metadata_check_interval 值。同步完成后重置为 false。在 NDB 8.0.19 中添加。

+   `ndb_replica_batch_size`: 副本应用程序的批处理大小（以字节为单位）。在 NDB 8.0.30 中添加。

+   `ndb_schema_dist_lock_wait_timeout`: 在模式分发期间等待锁定的时间，超时后返回错误。在 NDB 8.0.18 中添加。

+   `ndb_schema_dist_timeout`: 在模式分发期间检测超时之前等待的时间。在 NDB 8.0.16 中添加。

+   `ndb_schema_dist_upgrade_allowed`: 当连接到 NDB 时允许模式分发表升级。在 NDB 8.0.17 中添加。

+   `ndbinfo`: 启用 ndbinfo 插件（如果支持）。在 NDB 8.0.13 中添加。

+   `replica_allow_batching`: 为副本打开或关闭更新批处���。在 NDB 8.0.26 中添加。

#### 在 NDB 8.0 中已弃用的选项和变量

以下系统变量、状态变量和选项已在 NDB 8.0 中弃用。

+   `Ndb_api_adaptive_send_deferred_count_slave`: 此副本未实际发送的自适应发送调用次数。在 NDB 8.0.23 中已弃用。

+   `Ndb_api_adaptive_send_forced_count_slave`: 此副本发送的强制发送自适应发送次数。在 NDB 8.0.23 中已弃用。

+   `Ndb_api_adaptive_send_unforced_count_slave`: 此副本发送的非强制发送自适应发送次数。在 NDB 8.0.23 中已弃用。

+   `Ndb_api_bytes_received_count_slave`: 此副本从数据节点接收的数据量（以字节为单位）。在 NDB 8.0.23 中已弃用。

+   `Ndb_api_bytes_sent_count_slave`: 此副本发送到数据节点的数据量（以字节为单位）。在 NDB 8.0.23 中已弃用。

+   `Ndb_api_pk_op_count_slave`: 此副本基于或使用主键的操作次数。在 NDB 8.0.23 中已弃用。

+   `Ndb_api_pruned_scan_count_slave`: 此副本已被修剪到一个分区的扫描次数。在 NDB 8.0.23 中已弃用。

+   `Ndb_api_range_scan_count_slave`: 此副本启动的范围扫描次数。在 NDB 8.0.23 中已弃用。

+   `Ndb_api_read_row_count_slave`: 该副本已读取的总行数。在 NDB 8.0.23 中已弃用。

+   `Ndb_api_scan_batch_count_slave`: 该副本接收的行批次数量。在 NDB 8.0.23 中已弃用。

+   `Ndb_api_table_scan_count_slave`: 该副本开始的表扫描次数，包括内部表的扫描。在 NDB 8.0.23 中已弃用。

+   `Ndb_api_trans_abort_count_slave`: 该副本中被中止的事务数量。在 NDB 8.0.23 中已弃用。

+   `Ndb_api_trans_close_count_slave`: 该副本中被中止的事务数量（可能大于 TransCommitCount 和 TransAbortCount 之和）。在 NDB 8.0.23 中已弃用。

+   `Ndb_api_trans_commit_count_slave`: 该副本提交的事务数量。在 NDB 8.0.23 中已弃用。

+   `Ndb_api_trans_local_read_row_count_slave`: 该副本已读取的总行数。在 NDB 8.0.23 中已弃用。

+   `Ndb_api_trans_start_count_slave`: 该副本开始的事务数量。在 NDB 8.0.23 中已弃用。

+   `Ndb_api_uk_op_count_slave`: 该副本基于或使用唯一键的操作次数。在 NDB 8.0.23 中已弃用。

+   `Ndb_api_wait_exec_complete_count_slave`: 该副本中线程被阻塞等待操作执行完成的次数。在 NDB 8.0.23 中已弃用。

+   `Ndb_api_wait_meta_request_count_slave`: 该副本中线程被阻塞等待基于元数据的信号的次数。在 NDB 8.0.23 中已弃用。

+   `Ndb_api_wait_nanos_count_slave`: 该副本等待来自数据节点某种类型信号的总时间（以纳秒为单位）。在 NDB 8.0.23 中已弃用。

+   `Ndb_api_wait_scan_result_count_slave`: 该副本中线程被阻塞等待基于扫描的信号的次数。在 NDB 8.0.23 中已弃用。

+   `Ndb_metadata_blacklist_size`: NDB 二进制日志线程未能同步的 NDB 元数据对象数量；在 NDB 8.0.22 中更名为 Ndb_metadata_excluded_count。在 NDB 8.0.21 中已弃用。

+   `Ndb_replica_max_replicated_epoch`: 此副本上最近提交的 NDB 时代。当此值大于或等于 Ndb_conflict_last_conflict_epoch 时，尚未检测到冲突。在 NDB 8.0.23 中已弃用。

+   `ndb_slave_conflict_role`: 副本在冲突检测和解决中扮演的角色。值为 PRIMARY、SECONDARY、PASS 或 NONE（默认）。只能在停止复制 SQL 线程时更改。有关更多信息，请参阅文档。在 NDB 8.0.23 中已弃用。

+   `slave_allow_batching`: 打开或关闭副本的更新批处理。在 NDB 8.0.26 中已弃用。

#### 在 NDB 8.0 中移除的选项和变量

以下系统变量、状态变量和选项已在 NDB 8.0 中移除。

+   `Ndb_metadata_blacklist_size`: NDB 二进制日志线程未能同步的 NDB 元数据对象数量；在 NDB 8.0.22 中更名为 Ndb_metadata_excluded_count。在 NDB 8.0.22 中已移除。
