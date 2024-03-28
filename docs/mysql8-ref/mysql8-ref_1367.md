> 原文：[`dev.mysql.com/doc/refman/8.0/en/replication-options-reference.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-options-reference.html)

#### 19.1.6.1 复制和二进制日志选项和变量参考

下面的两个部分提供了关于适用于复制和二进制日志的 MySQL 命令行选项和系统变量的基本信息。

##### 复制选项和变量

下面列表中的命令行选项和系统变量与复制源服务器和副本相关。第 19.1.6.2 节，“复制源选项和变量” 提供了有关与复制源服务器相关的选项和变量的更详细信息。有关与副本相关的选项和变量的更多信息，请参见第 19.1.6.3 节，“副本服务器选项和变量”。

+   `abort-slave-event-count`: 用于 MySQL 测试的调试和复制测试的选项。

+   `auto_increment_increment`: AUTO_INCREMENT 列按此值递增。

+   `auto_increment_offset`: 添加到 AUTO_INCREMENT 列的偏移量。

+   `Com_change_master`: CHANGE REPLICATION SOURCE TO 和 CHANGE MASTER TO 语句的计数。

+   `Com_change_replication_source`: CHANGE REPLICATION SOURCE TO 和 CHANGE MASTER TO 语句的计数。

+   `Com_replica_start`: START REPLICA 和 START SLAVE 语句的计数。

+   `Com_replica_stop`: STOP REPLICA 和 STOP SLAVE 语句的计数。

+   `Com_show_master_status`: SHOW MASTER STATUS 语句的计数。

+   `Com_show_replica_status`: SHOW REPLICA STATUS 和 SHOW SLAVE STATUS 语句的计数。

+   `Com_show_replicas`: SHOW REPLICAS 和 SHOW SLAVE HOSTS 语句的计数。

+   `Com_show_slave_hosts`: SHOW REPLICAS 和 SHOW SLAVE HOSTS 语句的计数。

+   `Com_show_slave_status`: SHOW REPLICA STATUS 和 SHOW SLAVE STATUS 语句的计数。

+   `Com_slave_start`: START REPLICA 和 START SLAVE 语句的计数。

+   `Com_slave_stop`: STOP REPLICA 和 STOP SLAVE 语句的计数。

+   `disconnect-slave-event-count`: mysql-test 用于调试和测试复制的选项。

+   `enforce_gtid_consistency`: 防止无法以事务安全方式记录的语句执行。

+   `expire_logs_days`: 在多少天后清除二进制日志。

+   `gtid_executed`: 全局：二进制日志中的所有 GTID（全局）或当前事务（会话）中的所有 GTID。只读。

+   `gtid_executed_compression_period`: 每发生这么多次事务时压缩 gtid_executed 表。0 表示永远不压缩此表。仅在禁用二进制日志记录时适用。

+   `gtid_mode`: 控制是否启用基于 GTID 的日志记录以及事务日志可以包含的类型。

+   `gtid_next`: 指定后续事务或事务的 GTID；详细信息请参阅文档。

+   `gtid_owned`: 由此客户端（会话）或所有客户端拥有的 GTID 集合，以及所有者的线程 ID（全局）。只读。

+   `gtid_purged`: 已从二进制日志中清除的所有 GTID 集合。

+   `immediate_server_version`: 作为即时复制源的服务器的 MySQL 服务器发布号码。

+   `init_replica`: 当复制品连接到源时执行的语句。

+   `init_slave`: 当复制品连接到源时执行的语句。

+   `log_bin_trust_function_creators`: 如果等于 0（默认值），则在使用--log-bin 时，只允许具有 SUPER 特权的用户创建存储函数，并且只有创建的函数不会破坏二进制日志记录时才允许。

+   `log_statements_unsafe_for_binlog`: 禁用写入错误日志中的错误 1592 警告。

+   `master-info-file`: 记住源和 I/O 复制线程在源二进制日志中的位置的文件的位置和名称。

+   `master-retry-count`: 在放弃之前，复制品尝试连接到源的次数。

+   `master_info_repository`: 是否将包含源信息和复制 I/O 线程位置在源二进制日志中的连接元数据存储库写入文件或表。

+   `max_relay_log_size`: 如果非零，则中继日志在其大小超过此值时会自动旋转。如果为零，则旋转发生的大小由 max_binlog_size 的值确定。

+   `original_commit_timestamp`: 事务在原始源上提交时的时间。

+   `original_server_version`: 事务最初提交的服务器的 MySQL 服务器发布号。

+   `relay_log`: 用于中继日志的位置和基本名称。

+   `relay_log_basename`: 中继日志的完整路径，包括文件名。

+   `relay_log_index`: 用于保存最后中继日志列表的文件的位置和名称。

+   `relay_log_info_file`: 用于复制记录有关中继日志信息的应用程序元数据存储库的文件名。

+   `relay_log_info_repository`: 是否将复制 SQL 线程在中继日志中的位置写入文件或表。

+   `relay_log_purge`: 确定是否清除中继日志。

+   `relay_log_recovery`: 是否启用在启动时从源自动恢复中继日志文件；必须为崩溃安全的复制启用。

+   `relay_log_space_limit`: 用于所有中继日志的最大空间。

+   `replica_checkpoint_group`: 多线程复制在调用检查点操作更新进度状态之前处理的最大事务数。不支持 NDB 集群。

+   `replica_checkpoint_period`: 在此毫秒数后更新多线程复制的进度状态并将中继日志信息刷新到磁盘。不支持 NDB 集群。

+   `replica_compressed_protocol`: 使用源/复制协议��压缩。

+   `replica_exec_mode`: 允许在幂等模式（键和一些其他错误被抑制）和严格模式之间切换复制线程；严格模式是默认的，除了 NDB 集群，那里始终使用幂等模式。

+   `replica_load_tmpdir`: 复制 LOAD DATA 语句时，复制品应将其临时文件放置的位置。

+   `replica_max_allowed_packet`: 从���制源服务器发送到复制品的数据包的最大大小（以字节为单位）；覆盖 max_allowed_packet。

+   `replica_net_timeout`: 从源/复制连接中等待更多数据的秒数，然后中止读取。

+   `Replica_open_temp_tables`: 复制 SQL 线程当前打开的临时表数。

+   `replica_parallel_type`: 告诉复制品使用时间戳信息（LOGICAL_CLOCK）或数据库分区（DATABASE）来并行化事务。

+   `replica_parallel_workers`: 用于执行复制事务的应用程序线程数；当此值为 0 或 1 时，只有一个应用程序线程。NDB 集群：请参阅文档。

+   `replica_pending_jobs_size_max`: 持有尚未应用的事件的复制工作者队列的最大大小。

+   `replica_preserve_commit_order`: 确保复制工作者的所有提交按照源上的顺序发生，以在使用并行应用程序线程时保持一致性。

+   `Replica_rows_last_search_algorithm_used`: 此复制品最近用于定位行以进行基于行的复制（索引、表或哈希扫描）的搜索算法。

+   `replica_skip_errors`: 告诉复制线程在查询从提供的列表返回错误时继续复制。

+   `replica_transaction_retries`: 在事务由于死锁或经过的锁等待超时而失败时，复制 SQL 线程重试事务的次数，然后放弃并停止。

+   `replica_type_conversions`: 控制复制品上的类型转换模式。值是此列表中零个或多个元素的列表：ALL_LOSSY、ALL_NON_LOSSY。设置为空字符串以禁止源和复制品之间的类型转换。

+   `replicate-do-db`: 告诉复制 SQL 线程将复制限制在指定数据库中。

+   `replicate-do-table`: 告诉复制 SQL 线程将复制限制在指定表中。

+   `replicate-ignore-db`: 告诉复制 SQL 线程不要复制到指定数据库。

+   `replicate-ignore-table`: 告诉复制 SQL 线程不要复制到指定表。

+   `replicate-rewrite-db`: 更新具有与原始名称不同的数据库。

+   `replicate-same-server-id`: 在复制中，如果启用，则不跳过具有我们服务器 ID 的事件。

+   `replicate-wild-do-table`: 告诉复制 SQL 线程将复制限制在与指定通配符模式匹配的表中。

+   `replicate-wild-ignore-table`: 告诉复制 SQL 线程不要复制与给定通配符模式匹配的表。

+   `replication_optimize_for_static_plugin_config`: 针对静态插件配置的共享锁。

+   `replication_sender_observe_commit_only`: 半同步复制的有限回调。

+   `report_host`: 副本的主机名或 IP 地址，在副本注册期间报告给源。

+   `report_password`: 副本服务器应向源报告的任意密码；与用于复制用户帐户的密码不同。

+   `report_port`: 连接到副本并在副本注册期间向源报告的端口。

+   `report_user`: 副本服务器应向源报告的任意用户名；与用于复制用户帐户的名称不同。

+   `rpl_read_size`: 设置从二进制日志文件和中继日志文件中读取的最小数据量（以字节为单位）。

+   `Rpl_semi_sync_master_clients`: 半同步副本的数量。

+   `rpl_semi_sync_master_enabled`: 源上是否启用了半同步复制。

+   `Rpl_semi_sync_master_net_avg_wait_time`: 源等待来自副本回复的平均时间。

+   `Rpl_semi_sync_master_net_wait_time`: 源等待来自副本回复的总时间。

+   `Rpl_semi_sync_master_net_waits`: 源端等待来自副本的回复的总次数。

+   `Rpl_semi_sync_master_no_times`: 源端关闭半同步复制的次数。

+   `Rpl_semi_sync_master_no_tx`: 未成功确认的提交次数。

+   `Rpl_semi_sync_master_status`: 源端是否正在运行半同步复制。

+   `Rpl_semi_sync_master_timefunc_failures`: 调用时间函数时源端失败的次数。

+   `rpl_semi_sync_master_timeout`: 等待副本确认的毫秒数。

+   `rpl_semi_sync_master_trace_level`: 源端上的半同步复制调试跟踪级别。

+   `Rpl_semi_sync_master_tx_avg_wait_time`: 源端等待每个事务的平均时间。

+   `Rpl_semi_sync_master_tx_wait_time`: 源端等待事务的总时间。

+   `Rpl_semi_sync_master_tx_waits`: 源端等待事务的总次数。

+   `rpl_semi_sync_master_wait_for_slave_count`: 源端在继续之前必须收到每个事务的副本确认数。

+   `rpl_semi_sync_master_wait_no_slave`: 即使没有副本，源端是否等待超时。

+   `rpl_semi_sync_master_wait_point`: 副本事务接收确认的等待点。

+   `Rpl_semi_sync_master_wait_pos_backtraverse`: 源端等待具有比先前等待的事件更低的二进制坐标的事件的总次数。

+   `Rpl_semi_sync_master_wait_sessions`: 当前正在等待副本回复的会话数。

+   `Rpl_semi_sync_master_yes_tx`: 成功确认的提交次数。

+   `rpl_semi_sync_replica_enabled`: 副本上是否启用了半同步复制。

+   `Rpl_semi_sync_replica_status`: 复制端是否正在运行半同步复制。

+   `rpl_semi_sync_replica_trace_level`: 复制端半同步复制的调试跟踪级别。

+   `rpl_semi_sync_slave_enabled`: 复制端是否启用半同步复制。

+   `Rpl_semi_sync_slave_status`: 复制端是否正在运行半同步复制。

+   `rpl_semi_sync_slave_trace_level`: 复制端半同步复制的调试跟踪级别。

+   `Rpl_semi_sync_source_clients`: 半同步复制的复制端数量。

+   `rpl_semi_sync_source_enabled`: 源端是否启用半同步复制。

+   `Rpl_semi_sync_source_net_avg_wait_time`: 源端等待来自复制端回复的平均时间。

+   `Rpl_semi_sync_source_net_wait_time`: 源端等待复制端回复的总时间。

+   `Rpl_semi_sync_source_net_waits`: 源端等待复制端回复的总次数。

+   `Rpl_semi_sync_source_no_times`: 源端关闭半同步复制的次数。

+   `Rpl_semi_sync_source_no_tx`: 未成功确认的提交次数。

+   `Rpl_semi_sync_source_status`: 源端是否正在运行半同步复制。

+   `Rpl_semi_sync_source_timefunc_failures`: 调用时间函数时源端失败的次数。

+   `rpl_semi_sync_source_timeout`: 等待复制端确认的毫秒数。

+   `rpl_semi_sync_source_trace_level`: 源端半同步复制的调试跟踪级别。

+   `Rpl_semi_sync_source_tx_avg_wait_time`: 源端等待每个事务的平均时间。

+   `Rpl_semi_sync_source_tx_wait_time`: 源端等待事务的总时间。

+   `Rpl_semi_sync_source_tx_waits`: 源等待事务的总次数。

+   `rpl_semi_sync_source_wait_for_replica_count`: 在继续之前，源必须在每个事务中收到的副本确认数。

+   `rpl_semi_sync_source_wait_no_replica`: 源是否在没有副本的情况下等待超时。

+   `rpl_semi_sync_source_wait_point`: 等待副本事务接收确认的等待点���

+   `Rpl_semi_sync_source_wait_pos_backtraverse`: 源等待具有比先前等待的事件更低的二进制坐标的事件的总次数。

+   `Rpl_semi_sync_source_wait_sessions`: 当前正在等待副本回复的会话数。

+   `Rpl_semi_sync_source_yes_tx`: 成功确认的提交次数。

+   `rpl_stop_replica_timeout`: STOP REPLICA 等待超时的秒数。

+   `rpl_stop_slave_timeout`: STOP REPLICA 或 STOP SLAVE 等待超时的秒数。

+   `server_uuid`: 服务器的全局唯一 ID，在服务器启动时自动生成。

+   `show-replica-auth-info`: 在此源上的 SHOW REPLICAS 中显示用户名和密码。

+   `show-slave-auth-info`: 在此源上的 SHOW REPLICAS 和 SHOW SLAVE HOSTS 中显示用户名和密码。

+   `skip-replica-start`: 如果设置，当副本服务器启动时不会自动启动复制。

+   `skip-slave-start`: 如果设置，当副本服务器启动时不会自动启动复制。

+   `slave-skip-errors`: 当查询从提供的列表返回错误时，告诉复制线程继续复制。

+   `slave_checkpoint_group`: 多线程副本处理的最大事务数，在调用检查点操作以更新进度状态之前。不受 NDB Cluster 支持。

+   `slave_checkpoint_period`: 在此毫秒数后更新多线程副本的进度状态并将中继日志信息刷新到磁盘。NDB Cluster 不支持。

+   `slave_compressed_protocol`: 使用源/副本协议的压缩。

+   `slave_exec_mode`: 允许在`IDEMPOTENT`模式（键和一些其他错误被抑制）和`STRICT`模式之间切换复制线程；`STRICT`模式是默认的，除了 NDB Cluster，始终使用`IDEMPOTENT`。

+   `slave_load_tmpdir`: 当复制`LOAD DATA`语句时，副本应将临时文件放置的位置。

+   `slave_max_allowed_packet`: 从复制源服务器发送到副本的数据包的最大大小（以字节为单位）；覆盖`max_allowed_packet`。

+   `slave_net_timeout`: 等待源/副本连接中更多数据的秒数，超时前中止读取。

+   `Slave_open_temp_tables`: 复制 SQL 线程当前打开的临时表数。

+   `slave_parallel_type`: 告诉副本使用时间戳信息（LOGICAL_CLOCK）或数据库分区（DATABASE）来并行化事务。

+   `slave_parallel_workers`: 用于并行执行复制事务的应用程序线程数；0 或 1 禁用副本多线程。NDB Cluster：请参阅文档。

+   `slave_pending_jobs_size_max`: 持有尚未应用的事件的副本工作线程队列的最大大小。

+   `slave_preserve_commit_order`: 确保副本工作线程的所有提交按照源上的顺序发生，以保持一致性，当使用并行应用程序线程时。

+   `Slave_rows_last_search_algorithm_used`: 此副本最近用于定位行以进行基于行的复制（索引、表或哈希扫描）的搜索算法。

+   `slave_rows_search_algorithms`: 确定用于副本更新批处理的搜索算法。从此列表中选择任意 2 或 3 个：INDEX_SEARCH、TABLE_SCAN、HASH_SCAN。

+   `slave_transaction_retries`: 在复制 SQL 线程在死锁或经过的锁等待超时失败的情况下，重试事务的次数，然后放弃并停止。

+   `slave_type_conversions`: 控制副本上的类型转换模式。 值是以下列表中的零个或多个元素：ALL_LOSSY，ALL_NON_LOSSY。 将其设置为空字符串以禁止源和副本之间的类型转换。

+   `sql_log_bin`: 控制当前会话的二进制日志记录。

+   `sql_replica_skip_counter`: 副本应跳过的源事件数。 与 GTID 复制不兼容。

+   `sql_slave_skip_counter`: 副本应跳过的源事件数。 与 GTID 复制不兼容。

+   `sync_master_info`: 每第#个事件后同步源信息。

+   `sync_relay_log`: 每第#个事件后将 relay 日志同步到磁盘。

+   `sync_relay_log_info`: 每第#个事件后将 relay.info 文件同步到磁盘。

+   `sync_source_info`: 每第#个事件后同步源信息。

+   `terminology_use_previous`: 在不兼容更改的版本之前使用术语。

+   `transaction_write_set_extraction`: 定义在事务期间提取的写入哈希的算法。

对于所有与**mysqld**一起使用的命令行选项、系统变量和状态变量的列表，请参阅 Section 7.1.4, “Server Option, System Variable, and Status Variable Reference”。

##### 二进制日志选项和变量

以下列表中的命令行选项和系统变量与二进制日志相关。 Section 19.1.6.4, “Binary Logging Options and Variables”提供了有关与二进制日志相关的选项和变量的更详细信息。 有关二进制日志的其他一般信息，请参阅 Section 7.4.4, “The Binary Log”。

+   `binlog-checksum`: 启用或禁用二进制日志校验和。

+   `binlog-do-db`: 限制二进制日志记录到特定数据库。

+   `binlog-ignore-db`: 告诉源不要将对给定数据库的更新写入二进制日志。

+   `binlog-row-event-max-size`: 二进制日志最大事件大小。

+   `Binlog_cache_disk_use`: 使用临时文件而不是二进制日志缓存的事务数。

+   `binlog_cache_size`: 在事务期间保存 SQL 语句以用于二进制日志的缓存大小。

+   `Binlog_cache_use`: 使用临时二进制日志缓存的事务数。

+   `binlog_checksum`: 启用或禁用二进制日志校验和。

+   `binlog_direct_non_transactional_updates`: 导致使用语句格式对非事务引擎进行的更新直接写入二进制日志。在使用之前请查看文档。

+   `binlog_encryption`: 在此服务器上为二进制日志文件和中继日志文件启用加密。

+   `binlog_error_action`: 控制服务器无法写入二进制日志时会发生什么。

+   `binlog_expire_logs_auto_purge`: 控制二进制日志文件的自动清理；当启用时，可以通过将 binlog_expire_logs_seconds 和 expire_logs_days 都设置为 0 来覆盖。

+   `binlog_expire_logs_seconds`: 在这么多秒后清理二进制日志。

+   `binlog_format`: 指定二进制日志的格式。

+   `binlog_group_commit_sync_delay`: 设置在将事务同步到磁盘之前等待的微秒数。

+   `binlog_group_commit_sync_no_delay_count`: 设置在中止由 binlog_group_commit_sync_delay 指定的当前延迟之前等待的最大事务数。

+   `binlog_gtid_simple_recovery`: 控制在 GTID 恢复期间如何迭代二进制日志。

+   `binlog_max_flush_queue_time`: 在刷新到二进制日志之前读取事务的时间。

+   `binlog_order_commits`: 是否按照写入二进制日志的顺序提交。

+   `binlog_rotate_encryption_master_key_at_startup`: 在服务器启动时旋转二进制日志主密钥。

+   `binlog_row_image`: 记录行更改时使用完整或最小图像。

+   `binlog_row_metadata`: 在使用基于行的日志记录时，记录所有或仅最小的与表相关的元数据到二进制日志中。

+   `binlog_row_value_options`: 启用基于行复制的部分 JSON 更新的二进制日志记录。

+   `binlog_rows_query_log_events`: 启用时，在使用基于行的日志记录时启用行查询日志事件的记录。默认情况下禁用。在为早于 5.6 版本的副本/读取器生成日志时不要启用。

+   `Binlog_stmt_cache_disk_use`: 使用临时文件而不是二进制日志语句缓存的非事务语句数量。

+   `binlog_stmt_cache_size`: 在事务期间为二进制日志保留非事务语句的缓存大小。

+   `Binlog_stmt_cache_use`: 使用临时二进制日志语句缓存的语句数量。

+   `binlog_transaction_compression`: 启用二进制日志文件中事务负载的压缩。

+   `binlog_transaction_compression_level_zstd`: 二进制日志文件中事务负载的压缩级别。

+   `binlog_transaction_dependency_history_size`: 用于查找最后更新某行的事务的行哈希数量。

+   `binlog_transaction_dependency_tracking`: 用于评估哪些事务可以由副本的多线程应用程序并行执行的依赖信息来源（提交时间戳或事务写入集）。

+   `Com_show_binlog_events`: SHOW BINLOG EVENTS 语句的计数。

+   `Com_show_binlogs`: SHOW BINLOGS 语句的计数。

+   `log-bin`: 二进制日志文件的基本名称。

+   `log-bin-index`: 二进制日志索引文件的名称。

+   `log_bin`: 是否启用二进制日志。

+   `log_bin_basename`: 二进制日志文件的路径和基本名称。

+   `log_bin_use_v1_row_events`: 服务器是否使用版本 1 的二进制日志行事件。

+   `log_replica_updates`: 副本是否应将其复制 SQL 线程执行的更新记录到自己的二进制日志中。

+   `log_slave_updates`: 副本是否应将其复制 SQL 线程执行的更新记录到自己的二进制日志中。

+   `master_verify_checksum`: 使源在从二进制日志读取时检查校验和。

+   `max-binlog-dump-events`: mysql-test 用于调试和测试复制的选项。

+   `max_binlog_cache_size`: 可用于限制用于缓存多语句事务的总字节大小。

+   `max_binlog_size`: 当大小超过此值时，二进制日志会自动轮换。

+   `max_binlog_stmt_cache_size`: 可用于限制在事务期间缓存所有非事务语句的总大小。

+   `replica_sql_verify_checksum`: 使副本在从中继日志读取时检查校验和。

+   `slave-sql-verify-checksum`: 使副本在从中继日志读取时检查校验和。

+   `slave_sql_verify_checksum`: 使副本在从中继日志读取时检查校验和。

+   `source_verify_checksum`: 使源在从二进制日志读取时检查校验和。

+   `sporadic-binlog-dump-fail`: mysql-test 用于调试和测试复制的选项。

+   `sync_binlog`: 每第#个事件后将二进制日志同步刷新到磁盘。

对于所有与**mysqld**一起使用的命令行选项、系统和状态变量的列表，请参见第 7.1.4 节，“服务器选项、系统变量和状态变量参考”。
