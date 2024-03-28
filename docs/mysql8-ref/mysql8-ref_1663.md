> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-option-tables.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-option-tables.html)

#### 25.4.2.5 NDB Cluster mysqld 选项和变量参考

以下列表包括在 NDB Cluster 中作为 SQL 节点运行时`mysqld`中适用的命令行选项、系统变量和状态变量。有关*所有*与**mysqld**相关的命令行选项、系统变量和状态变量的参考，请参阅第 7.1.4 节，“服务器选项、系统变量和状态变量参考”。

+   `Com_show_ndb_status`: SHOW NDB STATUS 语句的计数。

+   `Handler_discover`: 表被发现的次数。

+   `ndb-applier-allow-skip-epoch`: 允许复制应用程序跳过时代。

+   `ndb-batch-size`: 用于 NDB 事务批处理的大小（以字节为单位）。

+   `ndb-blob-read-batch-bytes`: 指定大型 BLOB 读取应该分批处理的字节数。0 = 无限制。

+   `ndb-blob-write-batch-bytes`: 指定大型 BLOB 写入应该分批处理的字节数。0 = 无限制。

+   `ndb-cluster-connection-pool`: MySQL 使用的集群连接数。

+   `ndb-cluster-connection-pool-nodeids`: 用于 MySQL 连接到集群的节点 ID 的逗号分隔列表；列表中的节点数必须与--ndb-cluster-connection-pool 设置的值匹配。

+   `ndb-connectstring`: 分发此集群的配置信息的 NDB 管理服务器的地址。

+   `ndb-default-column-format`: 在创建或添加表列时，默认使用此值（FIXED 或 DYNAMIC）作为 COLUMN_FORMAT 和 ROW_FORMAT 选项。

+   `ndb-deferred-constraints`: 指定应该推迟对唯一索引的约束检查（支持的情况下）直到提交时间。通常不需要或使用；仅用于测试目的��

+   `ndb-distribution`: 新表在 NDBCLUSTER 中的默认分布（KEYHASH 或 LINHASH，默认为 KEYHASH）。

+   `ndb-log-apply-status`: 导致作为副本的 MySQL 服务器在其自己的二进制日志中记录从其直接来源接收到的 mysql.ndb_apply_status 更新，使用自己的服务器 ID。仅在使用--ndbcluster 选项启动服务器时有效。

+   `ndb-log-empty-epochs`: 启用时，即使启用了--log-slave-updates，也会导致未发生更改的时代被写入 ndb_apply_status 和 ndb_binlog_index 表。

+   `ndb-log-empty-update`: 启用时，即使启用了--log-slave-updates，也会导致产生没有更改的更新被写入 ndb_apply_status 和 ndb_binlog_index 表。

+   `ndb-log-exclusive-reads`: 使用独占锁记录主键读取；允许基于读取冲突进行冲突解决。

+   `ndb-log-fail-terminate`: 如果无法完全记录所有找到的行事件，则终止 mysqld 进程。

+   `ndb-log-orig`: 在 mysql.ndb_binlog_index 表中记录原始服务器 ID 和时代。

+   `ndb-log-transaction-dependency`: 使二进制日志线程为每个写入二进制日志的事务计算事务依赖关系。

+   `ndb-log-transaction-id`: 将 NDB 事务 ID 写入二进制日志。需要--log-bin-v1-events=OFF。

+   `ndb-log-update-minimal`: 以最小格式记录更新。

+   `ndb-log-updated-only`: 仅记录更新（ON）或完整行（OFF）。

+   `ndb-log-update-as-write`: 在源上切换更新的日志记录方式，可以是更新（OFF）或写入（ON）。

+   `ndb-mgmd-host`: 设置连接到管理服务器的主机（和端口，如果需要）。

+   `ndb-nodeid`: 此 MySQL 服务器的 NDB 集群节点 ID。

+   `ndb-optimized-node-selection`: 启用用于事务节点选择的优化。默认启用；使用--skip-ndb-optimized-node-selection 来禁用。

+   `ndb-transid-mysql-connection-map`: 启用或禁用 ndb_transid_mysql_connection_map 插件；即启用或禁用具有该名称的 INFORMATION_SCHEMA 表。

+   `ndb-wait-connected`: MySQL 服务器在接受 MySQL 客户端连接之前等待连接到集群管理和数据节点的时间（以秒为单位）。

+   `ndb-wait-setup`: MySQL 服务器等待 NDB 引擎设置完成的时间（以秒为单位）。

+   `ndb-allow-copying-alter-table`: 将其设置为 OFF，以防止 ALTER TABLE 在 NDB 表上使用复制操作。

+   `Ndb_api_adaptive_send_deferred_count`: 由此 MySQL 服务器（SQL 节点）未实际发送的自适应发送调用次数。

+   `Ndb_api_adaptive_send_deferred_count_session`: 此客户端会话中未实际发送的自适应发送调用次数。

+   `Ndb_api_adaptive_send_deferred_count_replica`: 此副本未实际发送的自适应发送调用次数。

+   `Ndb_api_adaptive_send_deferred_count_slave`: 此副本未实际发送的自适应发送调用次数。

+   `Ndb_api_adaptive_send_forced_count`: 由此 MySQL 服务器（SQL 节点）发送的带有强制发送设置的自适应发送次数。

+   `Ndb_api_adaptive_send_forced_count_session`: 此客户端会话中带有强制发送设置的自适应发送次数。

+   `Ndb_api_adaptive_send_forced_count_replica`: 此副本发送的带有强制发送设置的自适应发送次数。

+   `Ndb_api_adaptive_send_forced_count_slave`: 此副本发送的带有强制发送设置的自适应发送次数。

+   `Ndb_api_adaptive_send_unforced_count`: 由此 MySQL 服务器（SQL 节点）发送的无强制发送的自适应发送次数。

+   `Ndb_api_adaptive_send_unforced_count_session`: 此客户端会话中未带有强制发送的自适应发送次数。

+   `Ndb_api_adaptive_send_unforced_count_replica`: 该副本发送的无强制发送的自适应发送数量。

+   `Ndb_api_adaptive_send_unforced_count_slave`: 该副本发送的无强制发送的自适应发送数量。

+   `Ndb_api_bytes_received_count`: 从数据节点接收的数据量（以字节为单位）由此 MySQL 服务器（SQL 节点）。

+   `Ndb_api_bytes_received_count_session`: 从数据节点接收的数据量（以字节为单位）在此客户端会话中。

+   `Ndb_api_bytes_received_count_replica`: 从数据节点接收的数据量（以字节为单位）由此副本。

+   `Ndb_api_bytes_received_count_slave`: 从数据节点接收��数据量（以字节为单位）由此副本。

+   `Ndb_api_bytes_sent_count`: 发送到数据节点的数据量（以字节为单位）由此 MySQL 服务器（SQL 节点）。

+   `Ndb_api_bytes_sent_count_session`: 发送到数据节点的数据量（以字节为单位）在此客户端会话中。

+   `Ndb_api_bytes_sent_count_replica`: 该副本发送到数据节点的数据量（以字节为单位）。

+   `Ndb_api_bytes_sent_count_slave`: 该副本发送到数据节点的数据量（以字节为单位）。

+   `Ndb_api_event_bytes_count`: MySQL 服务器（SQL 节点）接收的事件字节数。

+   `Ndb_api_event_bytes_count_injector`: NDB 二进制日志注入器线程接收的事件数据字节数。

+   `Ndb_api_event_data_count`: MySQL 服务器（SQL 节点）接收的行更改事件数量。

+   `Ndb_api_event_data_count_injector`: NDB 二进制日志注入器线程接收的行更改事件数量。

+   `Ndb_api_event_nondata_count`: MySQL 服务器（SQL 节点）接收的除行更改事件之外的事件数量。

+   `Ndb_api_event_nondata_count_injector`: NDB 二进制日志注入线程接收的事件数量，除了行更改事件之外。

+   `Ndb_api_pk_op_count`: 由此 MySQL 服务器（SQL 节点）基于或使用主键的操作数量。

+   `Ndb_api_pk_op_count_session`: 在此客户端会话中基于或使用主键的操作数量。

+   `Ndb_api_pk_op_count_replica`: 由此副本基于或使用主键的操作数量。

+   `Ndb_api_pk_op_count_slave`: 由此副本基于或使用主键的操作数量。

+   `Ndb_api_pruned_scan_count`: 由此 MySQL 服务器（SQL 节点）修剪为一个分区的扫描次数。

+   `Ndb_api_pruned_scan_count_session`: 在此客户端会话中修剪为一个分区的扫描次数。

+   `Ndb_api_pruned_scan_count_replica`: 由此副本修剪为一个分区的扫描次数。

+   `Ndb_api_pruned_scan_count_slave`: 由此副本修剪为一个分区的扫描次数。

+   `Ndb_api_range_scan_count`: 由此 MySQL 服务器（SQL 节点）启动的范围扫描次数。

+   `Ndb_api_range_scan_count_session`: 在此客户端会话中启动的范围扫描次数。

+   `Ndb_api_range_scan_count_replica`: 由此副本启动的范围扫描次数。

+   `Ndb_api_range_scan_count_slave`: 由此副本启动的范围扫描次数。

+   `Ndb_api_read_row_count`: 由此 MySQL 服务器（SQL 节点）读取的总行数。

+   `Ndb_api_read_row_count_session`: 在此客户端会话中读取的总行数。

+   `Ndb_api_read_row_count_replica`: 由此副本读取的总行数。

+   `Ndb_api_read_row_count_slave`: 该副本已读取的总行数。

+   `Ndb_api_scan_batch_count`: 由此 MySQL 服务器（SQL 节点）接收的行批次数量。

+   `Ndb_api_scan_batch_count_session`: 在此客户端会话中接收的行批次数量。

+   `Ndb_api_scan_batch_count_replica`: 该副本接收的行批次数量。

+   `Ndb_api_scan_batch_count_slave`: 该副本接收的行批次数量。

+   `Ndb_api_table_scan_count`: 由此 MySQL 服务器（SQL 节点）启动的表扫描次数，包括内部表的扫描。

+   `Ndb_api_table_scan_count_session`: 在此客户端会话中启动的表扫描次数，包括内部表的扫描。

+   `Ndb_api_table_scan_count_replica`: 该副本启动的表扫描次数，包括内部表的扫描。

+   `Ndb_api_table_scan_count_slave`: 该副本启动的表扫描次数，包括内部表的扫描。

+   `Ndb_api_trans_abort_count`: 由此 MySQL 服务器（SQL 节点）中止的事务数量。

+   `Ndb_api_trans_abort_count_session`: 在此客户端会话中中止的事务数量。

+   `Ndb_api_trans_abort_count_replica`: 该副本中被中止的事务数量。

+   `Ndb_api_trans_abort_count_slave`: 该副本中被中止的事务数量。

+   `Ndb_api_trans_close_count`: 由此 MySQL 服务器（SQL 节点）中止的事务数量（可能大于 TransCommitCount 和 TransAbortCount 的总和）。

+   `Ndb_api_trans_close_count_session`: 在此客户端会话中中止的事务数量（可能大于 TransCommitCount 和 TransAbortCount 的总和）。

+   `Ndb_api_trans_close_count_replica`: 此副本中中止的事务数（可能大于 TransCommitCount 和 TransAbortCount 之和）。

+   `Ndb_api_trans_close_count_slave`: 此副本中中止的事务数（可能大于 TransCommitCount 和 TransAbortCount 之和）。

+   `Ndb_api_trans_commit_count`: 此 MySQL 服务器（SQL 节点）提交的事务数。

+   `Ndb_api_trans_commit_count_session`: 在此客户端会话中提交的事务数。

+   `Ndb_api_trans_commit_count_replica`: 此副本提交的事务数。

+   `Ndb_api_trans_commit_count_slave`: 此副本提交的事务数。

+   `Ndb_api_trans_local_read_row_count`: 此 MySQL 服务器（SQL 节点）已读取的总行数。

+   `Ndb_api_trans_local_read_row_count_session`: 在此客户端会话中已读取的总行数。

+   `Ndb_api_trans_local_read_row_count_replica`: 此副本已读取的总行数。

+   `Ndb_api_trans_local_read_row_count_slave`: 此副本已读取的总行数。

+   `Ndb_api_trans_start_count`: 此 MySQL 服务器（SQL 节点）启动的事务数。

+   `Ndb_api_trans_start_count_session`: 在此客户端会话中启动的事务数。

+   `Ndb_api_trans_start_count_replica`: 此副本启动的事务数。

+   `Ndb_api_trans_start_count_slave`: 此副本启动的事务数。

+   `Ndb_api_uk_op_count`: 此 MySQL 服务器（SQL 节点）基于或使用唯一键的操作数。

+   `Ndb_api_uk_op_count_session`: 在此客户端会话中基于或使用唯一键的操作数。

+   `Ndb_api_uk_op_count_replica`: 此副本基于或使用唯一键的操作次数。

+   `Ndb_api_uk_op_count_slave`: 此副本基于或使用唯一键的操作次数。

+   `Ndb_api_wait_exec_complete_count`: 线程在等待此 MySQL 服务器（SQL 节点）完成操作执行时被阻塞的次数。

+   `Ndb_api_wait_exec_complete_count_session`: 客户端会话中线程在等待操作执行完成时被阻塞的次数。

+   `Ndb_api_wait_exec_complete_count_replica`: 线程在等待此副本完成操作执行时被阻塞的次数。

+   `Ndb_api_wait_exec_complete_count_slave`: 线程在等待此副本完成操作执行时被阻塞的次数。

+   `Ndb_api_wait_meta_request_count`: 线程等待此 MySQL 服务器（SQL 节点）基于元数据的信号时被阻塞的次数。

+   `Ndb_api_wait_meta_request_count_session`: 客户端会话中线程等待基于元数据的信号时被阻塞的次数。

+   `Ndb_api_wait_meta_request_count_replica`: 线程等待此副本基于元数据的信号时被阻塞的次数。

+   `Ndb_api_wait_meta_request_count_slave`: 线程等待此副本基于元数据的信号时被阻塞的次数。

+   `Ndb_api_wait_nanos_count`: 此 MySQL 服务器（SQL 节点）等待来自数据节点某种类型信号的总时间（以纳秒为单位）。

+   `Ndb_api_wait_nanos_count_session`: 客户端会话中等待来自数据节点某种类型信号的总时间（以纳秒为单位）。

+   `Ndb_api_wait_nanos_count_replica`: 此副本等待来自数据节点某种类型信号的总时间（以纳秒为单位）。

+   `Ndb_api_wait_nanos_count_slave`: 此副本等待来自数据节点的某种类型信号的总时间（以纳秒为单位）。

+   `Ndb_api_wait_scan_result_count`: 等待此 MySQL 服务器（SQL 节点）通过扫描信号而被阻塞的线程次数。

+   `Ndb_api_wait_scan_result_count_session`: 在此客户端会话中等待扫描信号而被阻塞的线程次数。

+   `Ndb_api_wait_scan_result_count_replica`: 等待此副本通过扫描信号而被阻塞的线程次数。

+   `Ndb_api_wait_scan_result_count_slave`: 等待此副本通过扫描信号而被阻塞的线程次数。

+   `ndb_autoincrement_prefetch_sz`: NDB 自增预取大小。

+   `ndb_clear_apply_status`: 导致 RESET SLAVE/RESET REPLICA 清除 ndb_apply_status 表中的所有行；默认为 ON。

+   `Ndb_cluster_node_id`: 当作为 NDB 集群 SQL 节点时，此服务器的节点 ID。

+   `Ndb_config_from_host`: NDB 集群管理服务器主机名或 IP 地址。

+   `Ndb_config_from_port`: 用于连接 NDB 集群管理服务器的端口。

+   `Ndb_config_generation`: 集群当前配置的生成编号。

+   `Ndb_conflict_fn_epoch`: 通过 NDB$EPOCH() NDB 复制冲突检测函数发现的行数。

+   `Ndb_conflict_fn_epoch2`: 通过 NDB 复制 NDB$EPOCH2()冲突检测函数发现的行数。

+   `Ndb_conflict_fn_epoch2_trans`: 通过 NDB 复制 NDB$EPOCH2_TRANS()冲突检测函数发现的行数。

+   `Ndb_conflict_fn_epoch_trans`: 通过 NDB$EPOCH_TRANS()冲突检测函数发现的行数。

+   `Ndb_conflict_fn_max`: 基于“较大时间戳获胜”的 NDB 复制冲突解决已应用于更新和删除操作的次数。

+   `Ndb_conflict_fn_max_del_win`: 基于 NDB$MAX_DELETE_WIN()结果的 NDB 复制冲突解决已应用于更新和删除操作的次数。

+   `Ndb_conflict_fn_max_ins`: 基于“较大时间戳获胜”的 NDB 复制冲突解决已应用于插入操作的次数。

+   `Ndb_conflict_fn_max_del_win_ins`: 基于 NDB$MAX_DEL_WIN_INS()结果的 NDB 复制冲突解决已应用于插入操作的次数。

+   `Ndb_conflict_fn_old`: 在 NDB 复制中应用“相同时间戳获胜”冲突解决的次数。

+   `Ndb_conflict_last_conflict_epoch`: 在此副本上检测到冲突的最近 NDB 时代。

+   `Ndb_conflict_last_stable_epoch`: 通过事务冲突函数发现存在冲突的行数。

+   `Ndb_conflict_reflected_op_discard_count`: 由于执行期间出现错误而未应用的反射操作数量。

+   `Ndb_conflict_reflected_op_prepare_count`: 已准备好执行的接收到的反射操作数量。

+   `Ndb_conflict_refresh_op_count`: 已准备好的刷新操作数量。

+   `ndb_conflict_role`: 副本在冲突检测和解决中扮演的角色。值为 PRIMARY、SECONDARY、PASS 或 NONE（默认）。只能在停止复制 SQL 线程时更改。详细信息请参阅文档。

+   `Ndb_conflict_trans_conflict_commit_count`: 需要事务冲突处理后提交的时代事务数量。

+   `Ndb_conflict_trans_detect_iter_count`: 提交时代事务所需的内部迭代次数。应略大于或等于 Ndb_conflict_trans_conflict_commit_count。

+   `Ndb_conflict_trans_reject_count`: 事务冲突函数发现后被拒绝的事务数。

+   `Ndb_conflict_trans_row_conflict_count`: 事务冲突函数发现的冲突行数。包括任何包含在或依赖于冲突事务中的行。

+   `Ndb_conflict_trans_row_reject_count`: 事务冲突函数发现后重新调整的总行数。包括 Ndb_conflict_trans_row_conflict_count 和任何包含在或依赖于冲突事务中的行。

+   `ndb_data_node_neighbour`: 指定集群数据节点“最接近”此 MySQL 服务器，用于事务提示和完全复制表。

+   `ndb_default_column_format`: 设置用于新 NDB 表的默认行格式和列格式（FIXED 或 DYNAMIC）。

+   `ndb_deferred_constraints`: 指定应推迟约束检查（如果支持）。通常不需要或不使用；仅用于测试目的。

+   `ndb_dbg_check_shares`: 检查任何残留的共享（仅限调试版本）。

+   `ndb-schema-dist-timeout`: 在检测模式分发超时之前等待的时间。

+   `ndb_distribution`: NDBCLUSTER 中新表的默认分布（KEYHASH 或 LINHASH��默认为 KEYHASH）。

+   `Ndb_epoch_delete_delete_count`: 检测到的删除-删除冲突数（应用删除操作，但行不存在）。

+   `ndb_eventbuffer_free_percent`: 达到 ndb_eventbuffer_max_alloc 设置的限制后，在事件缓冲区中恢复缓冲之前应该可用的空闲内存百分比。

+   `ndb_eventbuffer_max_alloc`: NDB API 可用于缓冲事件的最大内存分配。默认为 0（无限制）。

+   `Ndb_execute_count`: 操作向 NDB 内核进行的往返次数。

+   `ndb_extra_logging`: 控制在 MySQL 错误日志中记录 NDB 集群模式、连接和数据分发事件。

+   `ndb_force_send`: 强制立即将缓冲区发送到 NDB，而不等待其他线程。

+   `ndb_fully_replicated`: 新 NDB 表是否完全复制。

+   `ndb_index_stat_enable`: 在查询优化中使用 NDB 索引统计。

+   `ndb_index_stat_option`: 用于 NDB 索引统计的逗号分隔的可调整选项列表；列表不应包含空格。

+   `ndb_join_pushdown`: 启用将连接下推到数据节点。

+   `Ndb_last_commit_epoch_server`: NDB 最近提交的时期。

+   `Ndb_last_commit_epoch_session`: 此 NDB 客户端最近提交的时期。

+   `ndb_log_apply_status`: MySQL 服务器是否作为副本记录 mysql.ndb_apply_status 更新，从其直接来源接收到的更新，使用自己的二进制日志，使用自己的服务器 ID。

+   `ndb_log_bin`: 将更新写入 NDB 表的二进制日志。仅在启用二进制日志记录时有效，使用--log-bin 选项。

+   `ndb_log_binlog_index`: 将时期和二进制日志位置之间的映射插入 ndb_binlog_index 表。默认为 ON。仅在启用二进制日志记录时有效。

+   `ndb_log_empty_epochs`: 启用时，将没有更改的时期写入 ndb_apply_status 和 ndb_binlog_index 表，即使启用了 log_replica_updates 或 log_slave_updates。

+   `ndb_log_empty_update`: 启用时，产生没有更改的更新将写入 ndb_apply_status 和 ndb_binlog_index 表，即使启用了 log_replica_updates 或 log_slave_updates。

+   `ndb_log_exclusive_reads`: 使用独占锁记录主键读取；允许基于读取冲突进行冲突解决。

+   `ndb_log_orig`: 是否在 mysql.ndb_binlog_index 表中记录原始服务器的 ID 和时期。在启动 mysqld 时使用--ndb-log-orig 选项设置。

+   `ndb_log_transaction_id`: 是否将 NDB 事务 ID 写入二进制日志（只读）。

+   `ndb_log_transaction_compression`: 是否压缩 NDB 二进制日志；也可以通过启用--binlog-transaction-compression 选项在启动时启用。

+   `ndb_log_transaction_compression_level_zstd`: 写入 NDB 二进制日志时使用的 ZSTD 压缩级别。

+   `ndb_metadata_check`: 启用自动检测 NDB 元数据相对于 MySQL 数据字典的更改；默认启用。

+   `Ndb_metadata_blacklist_size`: NDB 二进制日志线程未能同步的 NDB 元数据对象数量；在 NDB 8.0.22 中更名为 Ndb_metadata_excluded_count。

+   `ndb_metadata_check_interval`: 每隔几秒执行一次相对于 MySQL 数据字典的 NDB 元数据更改检查。

+   `Ndb_metadata_detected_count`: NDB 元数据更改监视线程检测到更改的次数。

+   `Ndb_metadata_excluded_count`: NDB 二进制日志线程未能同步的 NDB 元数据对象数量。

+   `ndb_metadata_sync`: 触发 NDB 字典和 MySQL 数据字典之间所有更改的立即同步；同步完成后重置为 false，ndb_metadata_check 和 ndb_metadata_check_interval 值将被忽略。

+   `Ndb_metadata_synced_count`: 已同步的 NDB 元数据对象数量。

+   `Ndb_number_of_data_nodes`: 此 NDB 集群中的数据节点数；仅在服务器参与集群时设置。

+   `ndb-optimization-delay`: 在 NDB 表上通过 OPTIMIZE TABLE 处理一组行之间等待的毫秒数。

+   `ndb_optimized_node_selection`: SQL 节点选择用作事务协调器的集群数据节点的方式。

+   `Ndb_pruned_scan_count`: 自上次启动集群以来由 NDB 执行的扫描次数，其中可以使用分区修剪。

+   `Ndb_pushed_queries_defined`: API 节点尝试下推到数据节点的连接数。

+   `Ndb_pushed_queries_dropped`: API 节点尝试推送但失败的连接数。

+   `Ndb_pushed_queries_executed`: 成功推送并在数据节点上执行的连接数。

+   `Ndb_pushed_reads`: 通过推送连接在数据节点上执行的读取次数。

+   `ndb_read_backup`: 为所有 NDB 表启用从任何副本读取；使用 CREATE TABLE 或 ALTER TABLE 中的 NDB_TABLE=READ_BACKUP={0|1}为单个 NDB 表启用或禁用。

+   `ndb_recv_thread_activation_threshold`: 当接收线程接管集群连接的轮询时的激活阈值（以同时活动线程计量）。

+   `ndb_recv_thread_cpu_mask`: 将接收线程锁定到特定 CPU 的 CPU 掩码；以十六进制形式指定。详细信息请参阅文档。

+   `Ndb_replica_max_replicated_epoch`: 此副本上最近提交的 NDB 时期。当此值大于或等于 Ndb_conflict_last_conflict_epoch 时，尚未检测到冲突。

+   `ndb_replica_batch_size`: 副本应用程序的批处理大小（以字节为单位）。

+   `ndb_report_thresh_binlog_epoch_slip`: NDB 7.5 及更高版本：完全缓冲但尚未被二进制日志注入器线程消耗的时期数量的阈值，超过该阈值会生成 BUFFERED_EPOCHS_OVER_THRESHOLD 事件缓冲区状态消息；在 NDB 7.5 之前：在报告二进制日志状态之前滞后的时期数量的阈值。

+   `ndb_report_thresh_binlog_mem_usage`: 在报告二进制日志状态之前剩余内存百分比的阈值。

+   `ndb_row_checksum`: 启用时设置行校验和；默认启用。

+   `Ndb_scan_count`: 自上次启动集群以来 NDB 执行的扫描总数。

+   `ndb_schema_dist_lock_wait_timeout`: 在模式分发期间等待锁定的时间，超时则返回错误。

+   `ndb_schema_dist_timeout`: 在模式分发期间检测超时之前等待的时间。

+   `ndb_schema_dist_upgrade_allowed`: 允许连接到 NDB 时进行模式分发表升级。

+   `ndb_show_foreign_key_mock_tables`: 显示用于支持 foreign_key_checks=0 的模拟表。

+   `ndb_slave_conflict_role`: 副本在冲突检测和解决中扮演的角色。值为 PRIMARY、SECONDARY、PASS 或 NONE（默认）。只能在停止复制 SQL 线程时更改。有关更多信息，请参阅文档。

+   `Ndb_slave_max_replicated_epoch`: 最近在此副本上提交的 NDB 时代。当此值大于或等于 Ndb_conflict_last_conflict_epoch 时，尚未检测到冲突。

+   `Ndb_system_name`: 配置的集群系统名称；如果服务器未连接到 NDB，则为空。

+   `ndb_table_no_logging`: 启用此设置时创建的 NDB 表不会被检查点到磁盘（尽管会创建表模式文件）。在使用 NDBCLUSTER 创建表或更改为使用 NDBCLUSTER 时设置会持续整个表的生命周期。

+   `ndb_table_temporary`: NDB 表在磁盘上不是持久的：不会创建模式文件，表也不会被记录。

+   `Ndb_trans_hint_count_session`: 在此会话中启动的使用提示的事务数量。

+   `ndb_use_copying_alter_table`: 在 NDB Cluster 中使用复制 ALTER TABLE 操作。

+   `ndb_use_exact_count`: 强制 NDB 在 SELECT COUNT(*)查询规划期间使用记录计数以加快此类查询的速度。

+   `ndb_use_transactions`: 设置为 OFF，以禁用 NDB 的事务支持。除了某些特殊情况外不建议使用；有关详细信息，请参阅文档。

+   `ndb_version`: 显示构建和 NDB 引擎版本为整数。

+   `ndb_version_string`: 显示构建信息，包括 NDB 引擎版本，格式为 ndb-x.y.z。

+   `ndbcluster`: 启用 NDB Cluster（如果此版本的 MySQL 支持）。通过`--skip-ndbcluster`禁用。

+   `ndbinfo`: 启用 ndbinfo 插件（如果支持）。

+   `ndbinfo_database`: 用于 NDB 信息数据库的名称；只读。

+   `ndbinfo_max_bytes`: 仅用于调试。

+   `ndbinfo_max_rows`: 仅用于调试。

+   `ndbinfo_offline`: 将 ndbinfo 数据库置于离线模式，在该模式下，表或视图不返回任何行。

+   `ndbinfo_show_hidden`: 是否在 mysql 客户端中显示 ndbinfo 内部基本表；默认为关闭。

+   `ndbinfo_table_prefix`: 用于命名 ndbinfo 内部基本表的前缀；只读。

+   `ndbinfo_version`: ndbinfo 引擎版本；只读。

+   `replica_allow_batching`: 打开或关闭副本的更新批处理。

+   `server_id_bits`: 用于标识服务器的 server_id 中实际使用的最低有效位数，允许 NDB API 应用程序将应用程序数据存储在最高有效位中。server_id 必须小于 2 的这个值的幂。

+   `skip-ndbcluster`: 禁用 NDB Cluster 存储引擎。

+   `slave_allow_batching`: 打开或关闭副本的更新批处理。

+   `transaction_allow_batching`: 允许在一个事务中批处理语句。禁用 AUTOCOMMIT 以使用。
