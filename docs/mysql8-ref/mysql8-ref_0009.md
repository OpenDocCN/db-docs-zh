# 1.4 MySQL 8.0 中新增、弃用或删除的服务器和状态变量和选项

> 原文：[`dev.mysql.com/doc/refman/8.0/en/added-deprecated-removed.html`](https://dev.mysql.com/doc/refman/8.0/en/added-deprecated-removed.html)

+   MySQL 8.0 中新增的选项和变量

+   MySQL 8.0 中已弃用的选项和变量

+   MySQL 8.0 中已删除的选项和变量

本节列出了在 MySQL 8.0 中首次添加、已弃用或已删除的服务器变量、状态变量和选项。

### MySQL 8.0 中新增的选项和变量

以下系统变量、状态变量和服务器选项已在 MySQL 8.0 中添加。

+   `Acl_cache_items_count`: 缓存的权限对象数量。MySQL 8.0.0 中新增。

+   `Audit_log_current_size`: 审计日志文件当前大小。MySQL 8.0.11 中新增。

+   `Audit_log_event_max_drop_size`: 最大丢弃的审计事件大小。MySQL 8.0.11 中新增。

+   `Audit_log_events`: 处理的审计事件数量。MySQL 8.0.11 中新增。

+   `Audit_log_events_filtered`: 过滤的审计事件数量。MySQL 8.0.11 中新增。

+   `Audit_log_events_lost`: 被丢弃的审计事件数量。MySQL 8.0.11 中新增。

+   `Audit_log_events_written`: 写入的审计事件数量。MySQL 8.0.11 中新增。

+   `Audit_log_total_size`: 写入的审计事件的总大小。MySQL 8.0.11 中新增。

+   `Audit_log_write_waits`: 写入延迟的审计事件数量。MySQL 8.0.11 中新增。

+   `Authentication_ldap_sasl_supported_methods`: SASL LDAP 认证支持的认证方法。MySQL 8.0.21 中新增。

+   `Caching_sha2_password_rsa_public_key`: caching_sha2_password 认证插件 RSA 公钥值。MySQL 8.0.4 中新增。

+   `Com_alter_resource_group`: ALTER RESOURCE GROUP 语句的计数。MySQL 8.0.3 中新增。

+   `Com_alter_user_default_role`: ALTER USER ... DEFAULT ROLE 语句的计数。MySQL 8.0.0 中新增。

+   `Com_change_replication_source`: CHANGE REPLICATION SOURCE TO 和 CHANGE MASTER TO 语句的计数。MySQL 8.0.23 版本中添加。

+   `Com_clone`: CLONE 语句的计数。MySQL 8.0.2 版本中添加。

+   `Com_create_resource_group`: CREATE RESOURCE GROUP 语句的计数。MySQL 8.0.3 版本中添加。

+   `Com_create_role`: CREATE ROLE 语句的计数。MySQL 8.0.0 版本中添加。

+   `Com_drop_resource_group`: DROP RESOURCE GROUP 语句的计数。MySQL 8.0.3 版本中添加。

+   `Com_drop_role`: DROP ROLE 语句的计数。MySQL 8.0.0 版本中添加。

+   `Com_grant_roles`: GRANT ROLE 语句的计数。MySQL 8.0.0 版本中添加。

+   `Com_install_component`: INSTALL COMPONENT 语句的计数。MySQL 8.0.0 版本中添加。

+   `Com_replica_start`: START REPLICA 和 START SLAVE 语句的计数。MySQL 8.0.22 版本中添加。

+   `Com_replica_stop`: STOP REPLICA 和 STOP SLAVE 语句的计数。MySQL 8.0.22 版本中添加。

+   `Com_restart`: RESTART 语句的计数。MySQL 8.0.4 版本中添加。

+   `Com_revoke_roles`: REVOKE ROLES 语句的计数。MySQL 8.0.0 版本中添加。

+   `Com_set_resource_group`: SET RESOURCE GROUP 语句的计数。MySQL 8.0.3 版本中添加。

+   `Com_set_role`: SET ROLE 语句的计数。MySQL 8.0.0 版本中添加。

+   `Com_show_replica_status`: SHOW REPLICA STATUS 和 SHOW SLAVE STATUS 语句的计数。MySQL 8.0.22 版本中添加。

+   `Com_show_replicas`: SHOW REPLICAS 和 SHOW SLAVE HOSTS 语句的计数。MySQL 8.0.22 版本中添加。

+   `Com_uninstall_component`: UNINSTALL COMPONENT 语句的计数。MySQL 8.0.0 版本中添加。

+   `Compression_algorithm`: 当前连接的压缩算法。MySQL 8.0.18 版本中添加。

+   `Compression_level`: 当前连接的压缩级别。MySQL 8.0.18 版本中添加。

+   `Connection_control_delay_generated`: 服务器延迟连接请求的次数。MySQL 8.0.1 版本中添加。

+   `Current_tls_ca`: ssl_ca 系统变量的当前值。MySQL 8.0.16 版本中添加。

+   `当前 _tls_ca 路径`: ssl_capath 系统变量的当前值。MySQL 8.0.16 中添加。

+   `当前 _tls 证书`: ssl_cert 系统变量的当前值。MySQL 8.0.16 中添加。

+   `当前 _tls 密码`: ssl_cipher 系统变量的当前值。MySQL 8.0.16 中添加。

+   `当前 _tls 密码套件`: tsl_ciphersuites 系统变量的当前值。MySQL 8.0.16 中添加。

+   `当前 _tls_crl`: ssl_crl 系统变量的当前值。MySQL 8.0.16 中添加。

+   `当前 _tls_crl 路径`: ssl_crlpath 系统变量的当前值。MySQL 8.0.16 中添加。

+   `当前 _tls 密钥`: ssl_key 系统变量的当前值。MySQL 8.0.16 中添加。

+   `当前 _tls 版本`: tls_version 系统变量的当前值。MySQL 8.0.16 中添加。

+   `错误日志缓冲字节`: error_log 表中使用的字节数。MySQL 8.0.22 中添加。

+   `错误日志缓冲事件`: error_log 表中的事件数量。MySQL 8.0.22 中添加。

+   `错误日志过期事件`: 从 error_log 表中丢弃的事件数量。MySQL 8.0.22 中添加。

+   `错误日志最新写入时间`: 写入 error_log 表的最后时间。MySQL 8.0.22 中添加。

+   `防火墙访问被拒绝`: MySQL 企业防火墙拒绝的语句数量。MySQL 8.0.11 中添加。

+   `防火墙访问已授权`: MySQL 企业防火墙接受的语句数量。MySQL 8.0.11 中添加。

+   `防火墙缓存条目`: MySQL 企业防火墙记录的语句数量。MySQL 8.0.11 中添加。

+   `全局连接内存`: 当前所有用户线程使用的内存量。MySQL 8.0.28 中添加。

+   `Innodb 缓冲池调整状态代码`: InnoDB 缓冲池调整状态代码。MySQL 8.0.31 中添加。

+   `Innodb 缓冲池调整状态进度`: InnoDB 缓冲池调整状态进度。MySQL 8.0.31 中添加。

+   `Innodb_redo_log_capacity_resized`: 最后一次完成容量调整操作后的重做日志容量。MySQL 8.0.30 中添加。

+   `Innodb_redo_log_checkpoint_lsn`: 重做日志检查点 LSN。MySQL 8.0.30 中添加。

+   `Innodb_redo_log_current_lsn`: 重做日志当前 LSN。MySQL 8.0.30 中添加。

+   `Innodb_redo_log_enabled`: InnoDB 重做日志状态。MySQL 8.0.21 中添加。

+   `Innodb_redo_log_flushed_to_disk_lsn`: 刷新到磁盘的重做日志 LSN。MySQL 8.0.30 中添加。

+   `Innodb_redo_log_logical_size`: 重做日志逻辑大小。MySQL 8.0.30 中添加。

+   `Innodb_redo_log_physical_size`: 重做日志物理大小。MySQL 8.0.30 中添加。

+   `Innodb_redo_log_read_only`: 重做日志是否为只读。MySQL 8.0.30 中添加。

+   `Innodb_redo_log_resize_status`: 重做日志调整大小状态。MySQL 8.0.30 中添加。

+   `Innodb_redo_log_uuid`: 重做日志 UUID。MySQL 8.0.30 中添加。

+   `Innodb_system_rows_deleted`: 从系统模式表中删除的行数。MySQL 8.0.19 中添加。

+   `Innodb_system_rows_inserted`: 插入到系统模式表中的行数。MySQL 8.0.19 中添加。

+   `Innodb_system_rows_read`: 从系统模式表中读取的行数。MySQL 8.0.19 中添加。

+   `Innodb_undo_tablespaces_active`: 活跃的撤销表空间数量。MySQL 8.0.14 中添加。

+   `Innodb_undo_tablespaces_explicit`: 用户创建的撤销表空间数量。MySQL 8.0.14 中添加。

+   `Innodb_undo_tablespaces_implicit`: InnoDB 创建的撤销表空间数量。MySQL 8.0.14 中添加。

+   `Innodb_undo_tablespaces_total`: 撤销表空间的总数。MySQL 8.0.14 中添加。

+   `Mysqlx_bytes_received_compressed_payload`: 接收的压缩消息负载字节数，解压缩前测量。MySQL 8.0.19 中添加。

+   `Mysqlx_bytes_received_uncompressed_frame`: 作为压缩消息负载接收的字节数，解压缩后测量。MySQL 8.0.19 中添加。

+   `Mysqlx_bytes_sent_compressed_payload`: 作为压缩消息负载发送的字节数，压缩后测量。MySQL 8.0.19 中添加。

+   `Mysqlx_bytes_sent_uncompressed_frame`: 作为压缩消息负载发送的字节数，压缩前测量。MySQL 8.0.19 中添加。

+   `Mysqlx_compression_algorithm`: 用于此会话的 X 协议连接的压缩算法。MySQL 8.0.20 中添加。

+   `Mysqlx_compression_level`: 用于此会话的 X 协议连接的压缩级别。MySQL 8.0.20 中添加。

+   `Replica_open_temp_tables`: 复制 SQL 线程当前打开的临时表数量。MySQL 8.0.26 中添加。

+   `Replica_rows_last_search_algorithm_used`: 此副本最近用于定位行的搜索算法（索引、表格或哈希扫描）。MySQL 8.0.26 中添加。

+   `Resource_group_supported`: 服务器是否支持资源组功能。MySQL 8.0.31 中添加。

+   `Rpl_semi_sync_replica_status`: 副本上是否正在运行半同步复制。MySQL 8.0.26 中添加。

+   `Rpl_semi_sync_source_clients`: 半同步复制副本的数量。MySQL 8.0.26 中添加。

+   `Rpl_semi_sync_source_net_avg_wait_time`: 源端等待来自副本回复的平均时间。MySQL 8.0.26 中添加。

+   `Rpl_semi_sync_source_net_wait_time`: 源端等待来自副本回复的总时间。MySQL 8.0.26 中添加。

+   `Rpl_semi_sync_source_net_waits`: 源端等待来自副本回复的总次数。MySQL 8.0.26 中添加。

+   `Rpl_semi_sync_source_no_times`: 源端关闭半同步复制的次数。MySQL 8.0.26 中添加。

+   `Rpl_semi_sync_source_no_tx`: 未成功确认的提交次数。MySQL 8.0.26 中添加。

+   `Rpl_semi_sync_source_status`: 源端是否正在进行半同步复制。在 MySQL 8.0.26 中添加。

+   `Rpl_semi_sync_source_timefunc_failures`: 调用时间函数时源端失败的次数。在 MySQL 8.0.26 中添加。

+   `Rpl_semi_sync_source_tx_avg_wait_time`: 每个事务源端等待的平均时间。在 MySQL 8.0.26 中添加。

+   `Rpl_semi_sync_source_tx_wait_time`: 源端等待事务的总时间。在 MySQL 8.0.26 中添加。

+   `Rpl_semi_sync_source_tx_waits`: 源端等待事务的总次数。在 MySQL 8.0.26 中添加。

+   `Rpl_semi_sync_source_wait_pos_backtraverse`: 源端等待具有比之前等待的事件更低的二进制坐标的总次数。在 MySQL 8.0.26 中添加。

+   `Rpl_semi_sync_source_wait_sessions`: 当前正在等待复制回复的会话数。在 MySQL 8.0.26 中添加。

+   `Rpl_semi_sync_source_yes_tx`: 成功确认的提交次数。在 MySQL 8.0.26 中添加。

+   `Secondary_engine_execution_count`: 转移到辅助引擎的查询数量。在 MySQL 8.0.13 中添加。

+   `Ssl_session_cache_timeout`: 缓存中当前 SSL 会话超时值。在 MySQL 8.0.29 中添加。

+   `Telemetry_traces_supported`: 是否支持服务器遥测跟踪。在 MySQL 8.0.33 中添加。

+   `Tls_library_version`: 使用的 OpenSSL 库的运行时版本。在 MySQL 8.0.30 中添加。

+   `activate_all_roles_on_login`: 是否在连接时激活所有用户角色。在 MySQL 8.0.2 中添加。

+   `admin-ssl`: 启用连接加密。在 MySQL 8.0.21 中添加。

+   `admin_address`: 用于管理接口连接的绑定 IP 地址。在 MySQL 8.0.14 中添加。

+   `admin_port`: 用于管理接口连接的 TCP/IP 号码。在 MySQL 8.0.14 中添加。

+   `admin_ssl_ca`: 包含受信任 SSL 证书颁发机构列表的文件。在 MySQL 8.0.21 中添加。

+   `admin_ssl_capath`: 包含受信任的 SSL 证书颁发机构证书文件的目录。在 MySQL 8.0.21 中添加。

+   `admin_ssl_cert`: 包含 X.509 证书的文件。在 MySQL 8.0.21 中添加。

+   `admin_ssl_cipher`: 连接加密的允许密码。在 MySQL 8.0.21 中添加。

+   `admin_ssl_crl`: 包含证书吊销列表的文件。在 MySQL 8.0.21 中添加。

+   `admin_ssl_crlpath`: 包含证书吊销列表文件的目录。在 MySQL 8.0.21 中添加。

+   `admin_ssl_key`: 包含 X.509 密钥的文件。在 MySQL 8.0.21 中添加。

+   `admin_tls_ciphersuites`: 加密连接的允许 TLSv1.3 密码套件。在 MySQL 8.0.21 中添加。

+   `admin_tls_version`: 加密连接的允许 TLS 协议。在 MySQL 8.0.21 中添加。

+   `audit-log`: 是否激活审计日志插件。在 MySQL 8.0.11 中添加。

+   `audit_log_buffer_size`: 审计日志缓冲区的大小。在 MySQL 8.0.11 中添加。

+   `audit_log_compression`: 审计日志文件压缩方法。在 MySQL 8.0.11 中添加。

+   `audit_log_connection_policy`: 连接相关事件的审计日志策略。在 MySQL 8.0.11 中添加。

+   `audit_log_current_session`: 是否审计当前会话。在 MySQL 8.0.11 中添加。

+   `audit_log_database`: 存储审计表的模式。在 MySQL 8.0.33 中添加。

+   `audit_log_disable`: 是否禁用审计日志。在 MySQL 8.0.28 中添加。

+   `audit_log_encryption`: 审计日志文件加密方法。在 MySQL 8.0.11 中添加。

+   `audit_log_exclude_accounts`: 不审计的账户。在 MySQL 8.0.11 中添加。

+   `audit_log_file`: 审计日志文件的名称。在 MySQL 8.0.11 中添加。

+   `audit_log_filter_id`: 当前审计日志过滤器的 ID。在 MySQL 8.0.11 中添加。

+   `audit_log_flush`: 关闭并重新打开审计日志文件。在 MySQL 8.0.11 中添加。

+   `audit_log_flush_interval_seconds`: 是否执行内存缓存的定期刷新。在 MySQL 8.0.34 中添加。

+   `audit_log_format`: 审计日志文件格式。在 MySQL 8.0.11 中添加。

+   `audit_log_format_unix_timestamp`: 是否在 JSON 格式审计日志中包含 Unix 时间戳。在 MySQL 8.0.26 中添加。

+   `audit_log_include_accounts`: 要审计的账户。在 MySQL 8.0.11 中添加。

+   `audit_log_max_size`: JSON 审计日志文件的组合大小限制。在 MySQL 8.0.26 中添加。

+   `audit_log_password_history_keep_days`: 保留归档审计日志加密密码的天数。在 MySQL 8.0.17 中添加。

+   `audit_log_policy`: 审计日志策略。在 MySQL 8.0.11 中添加。

+   `audit_log_prune_seconds`: 多少秒后审计日志文件将被修剪。在 MySQL 8.0.24 中添加。

+   `audit_log_read_buffer_size`: 审计日志文件读取缓冲区大小。在 MySQL 8.0.11 中添加。

+   `audit_log_rotate_on_size`: 在达到此大小时关闭并重新打开审计日志文件。在 MySQL 8.0.11 中添加。

+   `audit_log_statement_policy`: 与语句相关事件的审计日志策略。在 MySQL 8.0.11 中添加。

+   `audit_log_strategy`: 审计日志策略。在 MySQL 8.0.11 中添加。

+   `authentication_fido_rp_id`: 用于 FIDO 多因素认证的 Relying party ID。在 MySQL 8.0.27 中添加。

+   `authentication_kerberos_service_key_tab`: 包含用于认证 TGS 票证的 Kerberos 服务密钥的文件。在 MySQL 8.0.26 中添加。

+   `authentication_kerberos_service_principal`: Kerberos 服务主体名称。在 MySQL 8.0.26 中添加。

+   `authentication_ldap_sasl_auth_method_name`: 认证方法名称。在 MySQL 8.0.11 中添加。

+   `authentication_ldap_sasl_bind_base_dn`: LDAP 服务器基本分离名称。在 MySQL 8.0.11 中添加。

+   `authentication_ldap_sasl_bind_root_dn`: LDAP 服务器根分配名称。在 MySQL 8.0.11 中添加。

+   `authentication_ldap_sasl_bind_root_pwd`: LDAP 服务器根绑定密码。在 MySQL 8.0.11 中添加。

+   `authentication_ldap_sasl_ca_path`: LDAP 服务器证书颁发机构文件名。在 MySQL 8.0.11 中添加。

+   `authentication_ldap_sasl_group_search_attr`: LDAP 服务器组搜索属性。在 MySQL 8.0.11 中添加。

+   `authentication_ldap_sasl_group_search_filter`: LDAP 自定义组搜索过滤器。在 MySQL 8.0.11 中添加。

+   `authentication_ldap_sasl_init_pool_size`: LDAP 服务器初始连接池大小。在 MySQL 8.0.11 中添加。

+   `authentication_ldap_sasl_log_status`: LDAP 服务器日志级别。在 MySQL 8.0.11 中添加。

+   `authentication_ldap_sasl_max_pool_size`: LDAP 服务器最大连接池大小。在 MySQL 8.0.11 中添加。

+   `authentication_ldap_sasl_referral`: 是否启用 LDAP 搜索引荐。在 MySQL 8.0.20 中添加。

+   `authentication_ldap_sasl_server_host`: LDAP 服务器主机名或 IP 地址。在 MySQL 8.0.11 中添加。

+   `authentication_ldap_sasl_server_port`: LDAP 服务器端口号。在 MySQL 8.0.11 中添加。

+   `authentication_ldap_sasl_tls`: 是否使用加密连接到 LDAP 服务器。在 MySQL 8.0.11 中添加。

+   `authentication_ldap_sasl_user_search_attr`: LDAP 服务器用户搜索属性。在 MySQL 8.0.11 中添加。

+   `authentication_ldap_simple_auth_method_name`: 认证方法名称。在 MySQL 8.0.11 中添加。

+   `authentication_ldap_simple_bind_base_dn`: LDAP 服务器基本分配名称。在 MySQL 8.0.11 中添加。

+   `authentication_ldap_simple_bind_root_dn`: LDAP 服务器根分辨名称。在 MySQL 8.0.11 中添加。

+   `authentication_ldap_simple_bind_root_pwd`: LDAP 服务器根绑定密码。在 MySQL 8.0.11 中添加。

+   `authentication_ldap_simple_ca_path`: LDAP 服务器证书颁发机构文件名。在 MySQL 8.0.11 中添加。

+   `authentication_ldap_simple_group_search_attr`: LDAP 服务器组搜索属性。在 MySQL 8.0.11 中添加。

+   `authentication_ldap_simple_group_search_filter`: LDAP 自定义组搜索过滤器。在 MySQL 8.0.11 中添加。

+   `authentication_ldap_simple_init_pool_size`: LDAP 服务器初始连接池大小。在 MySQL 8.0.11 中添加。

+   `authentication_ldap_simple_log_status`: LDAP 服务器日志级别。在 MySQL 8.0.11 中添加。

+   `authentication_ldap_simple_max_pool_size`: LDAP 服务器最大连接池大小。在 MySQL 8.0.11 中添加。

+   `authentication_ldap_simple_referral`: 是否启用 LDAP 搜索引荐。在 MySQL 8.0.20 中添加。

+   `authentication_ldap_simple_server_host`: LDAP 服务器主机名或 IP 地址。在 MySQL 8.0.11 中添加。

+   `authentication_ldap_simple_server_port`: LDAP 服务器端口号。在 MySQL 8.0.11 中添加。

+   `authentication_ldap_simple_tls`: 是否使用加密连接到 LDAP 服务器。在 MySQL 8.0.11 中添加。

+   `authentication_ldap_simple_user_search_attr`: LDAP 服务器用户搜索属性。在 MySQL 8.0.11 中添加。

+   `authentication_policy`: 多因素认证插件。在 MySQL 8.0.27 中添加。

+   `authentication_windows_log_level`: Windows 认证插件日志级别。在 MySQL 8.0.11 中添加。

+   `authentication_windows_use_principal_name`: 是否使用 Windows 身份验证插件主体名称。MySQL 8.0.11 中添加。

+   `binlog_encryption`: 启用此服务器上二进制日志文件和中继日志文件的加密。MySQL 8.0.14 中添加。

+   `binlog_expire_logs_auto_purge`: 控制二进制日志文件的自动清理；当启用时，可以通过将 binlog_expire_logs_seconds 和 expire_logs_days 都设置为 0 来覆盖。MySQL 8.0.29 中添加。

+   `binlog_expire_logs_seconds`: 在此多少秒后清除二进制日志。MySQL 8.0.1 中添加。

+   `binlog_rotate_encryption_master_key_at_startup`: 在服务器启动时旋转二进制日志主密钥。MySQL 8.0.14 中添加。

+   `binlog_row_metadata`: 在使用基于行的日志记录时，是否记录所有或仅最小的与表相关的元数据到二进制日志中。MySQL 8.0.1 中添加。

+   `binlog_row_value_options`: 启用基于行的复制的部分 JSON 更新的二进制日志记录。MySQL 8.0.3 中添加。

+   `binlog_transaction_compression`: 启用二进制日志文件中事务负载的压缩。MySQL 8.0.20 中添加。

+   `binlog_transaction_compression_level_zstd`: 二进制日志文件中事务负载的压缩级别。MySQL 8.0.20 中添加。

+   `binlog_transaction_dependency_history_size`: 用于查找最后更新某行的事务的行哈希数。MySQL 8.0.1 中添加。

+   `binlog_transaction_dependency_tracking`: 用于评估哪些事务可以由副本的多线程应用程序并行执行的依赖信息来源（提交时间戳或事务写入集）。MySQL 8.0.1 中添加。

+   `build_id`: 在编译时生成的唯一构建 ID（仅限 Linux）。MySQL 8.0.31 中添加。

+   `caching_sha2_password_auto_generate_rsa_keys`: 是否自动生成 RSA 密钥对文件。MySQL 8.0.4 中添加。

+   `caching_sha2_password_digest_rounds`: caching_sha2_password 认证插件的哈希轮数。MySQL 8.0.24 中添加。

+   `caching_sha2_password_private_key_path`: SHA2 认证插件私钥路径名。MySQL 8.0.3 中添加。

+   `caching_sha2_password_public_key_path`: SHA2 认证插件公钥路径名。MySQL 8.0.3 中添加。

+   `clone_autotune_concurrency`: 启用动态生成线程以进行远程克隆操作。MySQL 8.0.17 中添加。

+   `clone_block_ddl`: 在克隆操作期间启用独占备份锁。MySQL 8.0.27 中添加。

+   `clone_buffer_size`: 定义在捐赠者 MySQL 服务器实例上的中间缓冲区大小。MySQL 8.0.17 中添加。

+   `clone_ddl_timeout`: 克隆操作等待备份锁的秒数。MySQL 8.0.17 中添加。

+   `clone_delay_after_data_drop`: 克隆过程开始前的延迟时间（秒）。MySQL 8.0.29 中添加。

+   `clone_donor_timeout_after_network_failure`: 网络故障后重新启动克隆操作的允许时间。MySQL 8.0.24 中添加。

+   `clone_enable_compression`: 在克隆过程中启用网络层数据压缩。MySQL 8.0.17 中添加。

+   `clone_max_concurrency`: 用于执行克隆操作的最大并发线程数。MySQL 8.0.17 中添加。

+   `clone_max_data_bandwidth`: 远程克隆操作每秒的最大数据传输速率（以 MiB 为单位）。MySQL 8.0.17 中添加。

+   `clone_max_network_bandwidth`: 远程克隆操作每秒的最大网络传输速率（以 MiB 为单位）。MySQL 8.0.17 中添加。

+   `clone_ssl_ca`: 指定证书颁发机构（CA）文件的路径。MySQL 8.0.14 中添加。

+   `clone_ssl_cert`: 指定公钥证书文件的路径。MySQL 8.0.14 中添加。

+   `clone_ssl_key`: 指定私钥文件的路径。MySQL 8.0.14 中添加。

+   `clone_valid_donor_list`: 定义远程克隆操作的捐赠主机地址。MySQL 8.0.17 中添加。

+   `component_scheduler.enabled`: 调度程序是否正在执行任务。MySQL 8.0.34 中添加。

+   `connection_control_failed_connections_threshold`: 连续失败的连接尝试次数，之后会发生延迟。MySQL 8.0.1 中添加。

+   `connection_control_max_connection_delay`: 服务器响应失败连接尝试的最大延迟时间（毫秒）。MySQL 8.0.1 中添加。

+   `connection_control_min_connection_delay`: 服务器响应失败连接尝试的最小延迟时间（毫秒）。MySQL 8.0.1 中添加。

+   `connection_memory_chunk_size`: 仅当用户内存使用量变化达到或超过此数量时才更新 Global_connection_memory；0 表示禁用更新。MySQL 8.0.28 中添加。

+   `connection_memory_limit`: 任何用户连接可以消耗的最大内存量，在此之前该用户的所有查询都将被拒绝。不适用于 MySQL root 等系统用户。MySQL 8.0.28 中添加。

+   `create_admin_listener_thread`: 是否使用专用监听线程处理管理接口上的连接。MySQL 8.0.14 中添加。

+   `cte_max_recursion_depth`: 公共表达式的最大递归深度。MySQL 8.0.3 中添加。

+   `ddl-rewriter`: 是否激活 ddl_rewriter 插件。MySQL 8.0.16 中添加。

+   `default_collation_for_utf8mb4`: utf8mb4 字符集的默认排序规则；仅供 MySQL 复制内部使用。MySQL 8.0.11 中添加。

+   `default_table_encryption`: 默认模式和表空间加密设置。MySQL 8.0.16 中添加。

+   `dragnet.Status`: 最近一次对 dragnet.log_error_filter_rules 的赋值结果。MySQL 8.0.12 中添加。

+   `dragnet.log_error_filter_rules`: 错误日志的过滤规则。MySQL 8.0.4 中添加。

+   `early-plugin-load`: 指定在加载强制内置插件和存储引擎初始化之前加载的插件。MySQL 8.0.0 中添加。

+   `enterprise_encryption.maximum_rsa_key_size`: MySQL 企业加密生成的 RSA 密钥的最大大小。在 MySQL 8.0.30 中添加。

+   `enterprise_encryption.rsa_support_legacy_padding`: 解密和验证传统的 MySQL 企业加密内容。在 MySQL 8.0.30 中添加。

+   `explain_format`: 决定 EXPLAIN 语句使用的默认输出格式。在 MySQL 8.0.32 中添加。

+   `generated_random_password_length`: 生成密码的最大长度。在 MySQL 8.0.18 中添加。

+   `global_connection_memory_limit`: 所有用户连接可以消耗的最大内存总量。当 Global_connection_memory 超出时，所有常规用户的查询都将被拒绝。不适用于 MySQL root 等系统用户。在 MySQL 8.0.28 中添加。

+   `global_connection_memory_tracking`: 是否计算全局连接内存使用情况（如 Global_connection_memory 所示）；默认为禁用。在 MySQL 8.0.28 中添加。

+   `group_replication_advertise_recovery_endpoints`: 分布式恢复提供的连接。在 MySQL 8.0.21 中添加。

+   `group_replication_autorejoin_tries`: 成员自动重新加入组的尝试次数。在 MySQL 8.0.16 中添加。

+   `group_replication_clone_threshold`: 捐赠者和接收者之间的事务编号差距，超过该差距时，将使用远程克隆操作进行状态传输。在 MySQL 8.0.17 中添加。

+   `group_replication_communication_debug_options`: Group Replication 组件的调试消息级别。在 MySQL 8.0.3 中添加。

+   `group_replication_communication_max_message_size`: Group Replication 通信的最大消息大小，较大的消息将被分段。在 MySQL 8.0.16 中添加。

+   `group_replication_communication_stack`: 指定是使用 XCom 通信堆栈还是 MySQL 通信堆栈来建立成员之间的组通信连接。在 MySQL 8.0.27 中添加。

+   `group_replication_consistency`: 组提供的事务一致性保证类型。在 MySQL 8.0.14 中添加。

+   `group_replication_exit_state_action`: 实例在非自愿离开组时的行为。在 MySQL 8.0.12 中添加。

+   `group_replication_flow_control_hold_percent`: 保留未使用的组配额的百分比。在 MySQL 8.0.2 中添加。

+   `group_replication_flow_control_max_quota`: 组的最大流控配额。在 MySQL 8.0.2 中添加。

+   `group_replication_flow_control_member_quota_percent`: 计算配额时成员应该假定可用于自身的配额百分比。在 MySQL 8.0.2 中添加。

+   `group_replication_flow_control_min_quota`: 每个成员可以分配的最低流控配额。在 MySQL 8.0.2 中添加。

+   `group_replication_flow_control_min_recovery_quota`: 每个成员可以分配的最低配额，因为另一个组成员正在恢复。在 MySQL 8.0.2 中添加。

+   `group_replication_flow_control_period`: 定义流控迭代之间等待的秒数。在 MySQL 8.0.2 中添加。

+   `group_replication_flow_control_release_percent`: 当流控不再需要限制写入成员时，应如何释放组配额。在 MySQL 8.0.2 中添加。

+   `group_replication_ip_allowlist`: 允许连接到组的主机列表（MySQL 8.0.22 及更高版本）。在 MySQL 8.0.22 中添加。

+   `group_replication_member_expel_timeout`: 成员被怀疑失败和被从组中驱逐之间的时间，导致组成员重新配置。在 MySQL 8.0.13 中添加。

+   `group_replication_member_weight`: 该成员被选为主要成员的机会。在 MySQL 8.0.2 中添加。

+   `group_replication_message_cache_size`: 组通信引擎消息缓存（XCom）的最大内存。在 MySQL 8.0.16 中添加。

+   `group_replication_paxos_single_leader`: 在单主模式中使用单一共识领导者。在 MySQL 8.0.27 中添加。

+   `group_replication_recovery_compression_algorithms`: 用于传出恢复连接的允许压缩算法。在 MySQL 8.0.18 中添加。

+   `group_replication_recovery_get_public_key`: 是否接受从捐赠者获取公钥的偏好。在 MySQL 8.0.4 中添加。

+   `group_replication_recovery_public_key_path`: 接受公钥信息。在 MySQL 8.0.4 中添加。

+   `group_replication_recovery_tls_ciphersuites`: 当 TLSv1.3 用于与此实例作为客户端（加入成员）连接加密时，允许的密码套件。在 MySQL 8.0.19 中添加。

+   `group_replication_recovery_tls_version`: 作为客户端（加入成员）连接加密的允许 TLS 协议。在 MySQL 8.0.19 中添加。

+   `group_replication_recovery_zstd_compression_level`: 用于使用 zstd 压缩的恢复连接的压缩级别。在 MySQL 8.0.18 中添加。

+   `group_replication_tls_source`: Group Replication 的 TLS 材料来源。在 MySQL 8.0.21 中添加。

+   `group_replication_unreachable_majority_timeout`: 等待导致少数派离开组的网络分区的时间。在 MySQL 8.0.2 中添加。

+   `group_replication_view_change_uuid`: 视图更改事件 GTIDs 的 UUID。在 MySQL 8.0.26 中添加。

+   `histogram_generation_max_mem_size`: 创建直方图统计信息的最大内存。在 MySQL 8.0.2 中添加。

+   `immediate_server_version`: 作为即时复制源的服务器的 MySQL 服务器发布号。在 MySQL 8.0.14 中添加。

+   `information_schema_stats_expiry`: 缓存表统计信息的过期设置。在 MySQL 8.0.3 中添加。

+   `init_replica`: 当副本连接到源时执行的语句。在 MySQL 8.0.26 中添加。

+   `innodb_buffer_pool_debug`: 当缓冲池小于 1GB 时，允许多个缓冲池实例。MySQL 8.0.0 中添加。

+   `innodb_buffer_pool_in_core_file`: 控制将缓冲池页面写入核心文件。MySQL 8.0.14 中添加。

+   `innodb_checkpoint_disabled`: 禁用检查点，以便故意服务器退出始终启动恢复。MySQL 8.0.2 中添加。

+   `innodb_ddl_buffer_size`: DDL 操作的最大缓冲区大小。MySQL 8.0.27 中添加。

+   `innodb_ddl_log_crash_reset_debug`: 重置 DDL 日志崩溃注入计数器的调试选项。MySQL 8.0.3 中添加。

+   `innodb_ddl_threads`: 用于索引创建的最大并行线程数。MySQL 8.0.27 中添加。

+   `innodb_deadlock_detect`: 启用或禁用死锁检测。MySQL 8.0.0 中添加。

+   `innodb_dedicated_server`: 启用缓冲池大小、日志文件大小���刷新方法的自动配置。MySQL 8.0.3 中添加。

+   `innodb_directories`: 定义在启动时扫描表空间数据文件的目录。MySQL 8.0.4 中添加。

+   `innodb_doublewrite_batch_size`: 每批写入的双写页数。MySQL 8.0.20 中添加。

+   `innodb_doublewrite_dir`: 双写缓冲文件目录。MySQL 8.0.20 中添加。

+   `innodb_doublewrite_files`: 双写文件数。MySQL 8.0.20 中添加。

+   `innodb_doublewrite_pages`: 每个线程的双写页数。MySQL 8.0.20 中添加。

+   `innodb_extend_and_initialize`: 控制在 Linux 上分配新表空间页面的方式。MySQL 8.0.22 中添加。

+   `innodb_fsync_threshold`: 控制 InnoDB 在创建新文件时调用 fsync 的频率。MySQL 8.0.13 中添加。

+   `innodb_idle_flush_pct`: 当 InnoDB 空闲时限制 I/O 操作。MySQL 8.0.18 中添加。

+   `innodb_log_checkpoint_fuzzy_now`: 强制 InnoDB 写入模糊检查点的调试选项。MySQL 8.0.13 中添加。

+   `innodb_log_spin_cpu_abs_lwm`: 在等待刷新的重做时，用户线程不再自旋的最低 CPU 使用量。MySQL 8.0.11 中添加。

+   `innodb_log_spin_cpu_pct_hwm`: 在等待刷新的重做时，用户线程不再自旋的最大 CPU 使用量。MySQL 8.0.11 中添加。

+   `innodb_log_wait_for_flush_spin_hwm`: 超过平均日志刷新时间的最大值时，用户线程不再自旋等待刷新的重做。MySQL 8.0.11 中添加。

+   `innodb_log_writer_threads`: 启用专用的日志写入线程来写入和刷新重做日志。MySQL 8.0.22 中添加。

+   `innodb_parallel_read_threads`: 并行索引读取线程数。MySQL 8.0.14 中添加。

+   `innodb_print_ddl_logs`: 是否将 DDL 日志打印到错误日志中。MySQL 8.0.3 中添加。

+   `innodb_redo_log_archive_dirs`: 标记的重做日志归档目录。MySQL 8.0.17 中添加。

+   `innodb_redo_log_capacity`: 重做日志文件的大小限制。MySQL 8.0.30 中添加。

+   `innodb_redo_log_encrypt`: 控制加密表空间的重做日志数据的加密。MySQL 8.0.1 中添加。

+   `innodb_scan_directories`: 定义在 InnoDB 恢复期间扫描表空间文件的目录。MySQL 8.0.2 中添加。

+   `innodb_segment_reserve_factor`: 作为空页保留的表空间文件段页面的百分比。MySQL 8.0.26 中添加。

+   `innodb_spin_wait_pause_multiplier`: 用于确定自旋等待循环中 PAUSE 指令数量的乘数值。MySQL 8.0.16 中添加。

+   `innodb_stats_include_delete_marked`: 在计算持久 InnoDB 统计信息时包括删除标记记录。MySQL 8.0.1 中添加。

+   `innodb_temp_tablespaces_dir`: 会话临时表空间路径。MySQL 8.0.13 中添加。

+   `innodb_tmpdir`: 在线 ALTER TABLE 操作期间创建的临时表文件的目录位置。MySQL 8.0.0 中添加。

+   `innodb_undo_log_encrypt`: 控制加密表空间的撤销日志数据的加密。MySQL 8.0.1 中添加。

+   `innodb_use_fdatasync`: 当刷新数据到操作系统时，InnoDB 是否使用 fdatasync() 而不是 fsync()。在 MySQL 8.0.26 中添加。

+   `innodb_validate_tablespace_paths`: 启用启动时的表空间路径验证。在 MySQL 8.0.21 中添加。

+   `internal_tmp_mem_storage_engine`: 用于内部内存临时表的存储引擎。在 MySQL 8.0.2 中添加。

+   `keyring-migration-destination`: 密钥迁移目标密钥环插件。在 MySQL 8.0.4 中添加。

+   `keyring-migration-host`: 用于连接到正在运行的服务器进行密钥迁移的主机名。在 MySQL 8.0.4 中添加。

+   `keyring-migration-password`: 用于连接到正在运行的服务器进行密钥迁移的密码。在 MySQL 8.0.4 中添加。

+   `keyring-migration-port`: 用于连接到正在运行的服务器进行密钥迁移的 TCP/IP 端口号。在 MySQL 8.0.4 中添加。

+   `keyring-migration-socket`: 用于连接到正在运行的服务器进行密钥迁移的 Unix 套接字文件或 Windows 命名管道。在 MySQL 8.0.4 中添加。

+   `keyring-migration-source`: 密钥迁移源密钥环插件。在 MySQL 8.0.4 中添加。

+   `keyring-migration-to-component`: 密钥迁移是从插件到组件。在 MySQL 8.0.24 中添加。

+   `keyring-migration-user`: 用于连接到正在运行的服务器进行密钥迁移的用户名。在 MySQL 8.0.4 中添加。

+   `keyring_aws_cmk_id`: AWS 密钥环插件客户主密钥 ID 值。在 MySQL 8.0.11 中添加。

+   `keyring_aws_conf_file`: AWS 密钥环插件配置文件位置。在 MySQL 8.0.11 中添加。

+   `keyring_aws_data_file`: AWS 密钥环插件存储文件位置。在 MySQL 8.0.11 中添加。

+   `keyring_aws_region`: AWS 密钥环插件区域。在 MySQL 8.0.11 中添加。

+   `keyring_encrypted_file_data`: keyring_encrypted_file 插件数据文件。在 MySQL 8.0.11 中添加。

+   `keyring_encrypted_file_password`: keyring_encrypted_file 插件密码。在 MySQL 8.0.11 中添加。

+   `keyring_hashicorp_auth_path`: HashiCorp Vault AppRole 认证路径。在 MySQL 8.0.18 中添加。

+   `keyring_hashicorp_ca_path`: keyring_hashicorp CA 文件路径。在 MySQL 8.0.18 中添加。

+   `keyring_hashicorp_caching`: 是否启用 keyring_hashicorp 缓存。在 MySQL 8.0.18 中添加。

+   `keyring_hashicorp_commit_auth_path`: 使用的 keyring_hashicorp_auth_path 值。在 MySQL 8.0.18 中添加。

+   `keyring_hashicorp_commit_ca_path`: 使用的 keyring_hashicorp_ca_path 值。在 MySQL 8.0.18 中添加。

+   `keyring_hashicorp_commit_caching`: 使用的 keyring_hashicorp_caching 值。在 MySQL 8.0.18 中添加。

+   `keyring_hashicorp_commit_role_id`: 使用的 keyring_hashicorp_role_id 值。在 MySQL 8.0.18 中添加。

+   `keyring_hashicorp_commit_server_url`: 使用的 keyring_hashicorp_server_url 值。在 MySQL 8.0.18 中添加。

+   `keyring_hashicorp_commit_store_path`: 使用的 keyring_hashicorp_store_path 值。在 MySQL 8.0.18 中添加。

+   `keyring_hashicorp_role_id`: HashiCorp Vault AppRole 认证角色 ID。在 MySQL 8.0.18 中添加。

+   `keyring_hashicorp_secret_id`: HashiCorp Vault AppRole 认证秘密 ID。在 MySQL 8.0.18 中添加。

+   `keyring_hashicorp_server_url`: HashiCorp Vault 服务器 URL。在 MySQL 8.0.18 中添加。

+   `keyring_hashicorp_store_path`: HashiCorp Vault 存储路径。在 MySQL 8.0.18 中添加。

+   `keyring_oci_ca_certificate`: 用于对等身份验证的 CA 证书文件。在 MySQL 8.0.22 中添加。

+   `keyring_oci_compartment`: OCI compartment OCID。在 MySQL 8.0.22 中添加。

+   `keyring_oci_encryption_endpoint`: OCI 加密服务器端点。在 MySQL 8.0.22 中添加。

+   `keyring_oci_key_file`: OCI RSA 私钥文件。在 MySQL 8.0.22 中添加。

+   `keyring_oci_key_fingerprint`: OCI RSA 私钥文件指纹。在 MySQL 8.0.22 中添加。

+   `keyring_oci_management_endpoint`: OCI 管理服务器端点。在 MySQL 8.0.22 中添加。

+   `keyring_oci_master_key`: OCI 主密钥 OCID。在 MySQL 8.0.22 中添加。

+   `keyring_oci_secrets_endpoint`: OCI 秘密服务器端点。在 MySQL 8.0.22 中添加。

+   `keyring_oci_tenancy`: OCI 租户 OCID。在 MySQL 8.0.22 中添加。

+   `keyring_oci_user`: OCI 用户 OCID。在 MySQL 8.0.22 中添加。

+   `keyring_oci_vaults_endpoint`: OCI 保险库服务器端点。在 MySQL 8.0.22 中添加。

+   `keyring_oci_virtual_vault`: OCI 保险库 OCID。在 MySQL 8.0.22 中添加。

+   `keyring_okv_conf_dir`: Oracle Key Vault 密钥环插件配置目录。在 MySQL 8.0.11 中添加。

+   `keyring_operations`: 是否启用密钥环操作。在 MySQL 8.0.4 中添加。

+   `lock_order`: 是否在运行时启用 LOCK_ORDER 工具。在 MySQL 8.0.17 中添加。

+   `lock_order_debug_loop`: 当 LOCK_ORDER 工具遇到标记为循环的依赖关系时，是否引发调试断言。在 MySQL 8.0.17 中添加。

+   `lock_order_debug_missing_arc`: 当 LOCK_ORDER 工具遇到未声明的依赖关系时，是否引发调试断言。在 MySQL 8.0.17 中添加。

+   `lock_order_debug_missing_key`: 当 LOCK_ORDER 工具遇到未正确使用性能模式进行仪器化的对象时，是否引发调试断言。在 MySQL 8.0.17 中添加。

+   `lock_order_debug_missing_unlock`: 当 LOCK_ORDER 工具遇到仍在持有的锁被销毁时，是否引发调试断言。在 MySQL 8.0.17 中添加。

+   `lock_order_dependencies`: lock_order_dependencies.txt 文件的路径。在 MySQL 8.0.17 中添加。

+   `lock_order_extra_dependencies`: 第二个依赖文件的路径。在 MySQL 8.0.17 中添加。

+   `lock_order_output_directory`: LOCK_ORDER 工具写入日志的目录。在 MySQL 8.0.17 中添加。

+   `lock_order_print_txt`: 是否执行锁定顺序图分析并打印文本报告。在 MySQL 8.0.17 中添加。

+   `lock_order_trace_loop`: 当 LOCK_ORDER 工具遇到标记为循环的依赖关系时，是否打印日志文件跟踪。在 MySQL 8.0.17 中添加。

+   `lock_order_trace_missing_arc`: 当 LOCK_ORDER 工具遇到未声明的依赖关系时，是否打印日志文件跟踪。在 MySQL 8.0.17 中添加。

+   `lock_order_trace_missing_key`: 当 LOCK_ORDER 工具遇到未正确使用性能模式进行仪表化的对象时，是否打印日志文件跟踪。在 MySQL 8.0.17 中添加。

+   `lock_order_trace_missing_unlock`: 当 LOCK_ORDER 工具遇到仍在持有的锁被销毁时，是否打印日志文件跟踪。在 MySQL 8.0.17 中添加。

+   `log_error_filter_rules`: 错误日志记录的过滤规则。在 MySQL 8.0.2 中添加。

+   `log_error_services`: 用于错误日志记录的组件。在 MySQL 8.0.2 中添加。

+   `log_error_suppression_list`: 要抑制的警告/信息错误日志消息。在 MySQL 8.0.13 中添加。

+   `log_replica_updates`: 是否应该将副本的复制 SQL 线程执行的更新记录到自己的二进制日志中。在 MySQL 8.0.26 中添加。

+   `log_slow_extra`: 是否向慢查询日志文件写入额外信息。在 MySQL 8.0.14 中添加。

+   `log_slow_replica_statements`: 导致副本执行的慢查询被写入慢查询日志。在 MySQL 8.0.26 中添加。

+   `mandatory_roles`: 自动授予所有用户的角色。在 MySQL 8.0.2 中添加。

+   `mysql_firewall_mode`: MySQL 企业防火墙是否正在运行。在 MySQL 8.0.11 中添加。

+   `mysql_firewall_trace`: 是否启用防火墙跟踪。在 MySQL 8.0.11 中添加。

+   `mysqlx`: X 插件是否已初始化。在 MySQL 8.0.11 中添加。

+   `mysqlx_compression_algorithms`: 允许 X 协议连接使用的压缩算法。在 MySQL 8.0.19 中添加。

+   `mysqlx_deflate_default_compression_level`: X 协议连接上 Deflate 算法的默认压缩级别。在 MySQL 8.0.20 中添加。

+   `mysqlx_deflate_max_client_compression_level`: X 协议连接上 Deflate 算法的最大允许压缩级别。MySQL 8.0.20 中添加。

+   `mysqlx_interactive_timeout`: 等待交互式客户端超时的秒数。MySQL 8.0.4 中添加。

+   `mysqlx_lz4_default_compression_level`: X 协议连接上 LZ4 算法的默认压缩级别。MySQL 8.0.20 中添加。

+   `mysqlx_lz4_max_client_compression_level`: X 协议连接上 LZ4 算法的最大允许压缩级别。MySQL 8.0.20 中添加。

+   `mysqlx_read_timeout`: 等待阻塞读操作完成的秒数。MySQL 8.0.4 中添加。

+   `mysqlx_wait_timeout`: 等待连接活动的秒数。MySQL 8.0.4 中添加。

+   `mysqlx_write_timeout`: 等待阻塞写操作完成的秒数。MySQL 8.0.4 中添加。

+   `mysqlx_zstd_default_compression_level`: X 协议连接上 zstd 算法的默认压缩级别。MySQL 8.0.20 中添加。

+   `mysqlx_zstd_max_client_compression_level`: X 协议连接上 zstd 算法的最大允许压缩级别。MySQL 8.0.20 中添加。

+   `named_pipe_full_access_group`: 被授予对命名管道完全访问权限的 Windows 组名称。MySQL 8.0.14 中添加。

+   `no-dd-upgrade`: 防止启动时自动升级数据字典表。MySQL 8.0.4 中添加。

+   `no-monitor`: 不要为 RESTART 需要的监视进程分叉。MySQL 8.0.12 中添加。

+   `original_commit_timestamp`: 事务在原始源上提交的时间。MySQL 8.0.1 中添加。

+   `original_server_version`: 事务最初提交的 MySQL 服务器发布版本号。MySQL 8.0.14 中添加。

+   `partial_revokes`: 是否启用部分撤销。MySQL 8.0.16 中添加。

+   `password_history`: 在重复使用密码之前需要更改的次数。MySQL 8.0.3 中添加。

+   `password_require_current`: 密码更改是否需要当前密码验证。MySQL 8.0.13 中添加。

+   `password_reuse_interval`: 在重复使用密码之前需要经过的天数。MySQL 8.0.3 中添加。

+   `performance-schema-consumer-events-statements-cpu`: 配置语句 CPU 使用情况消费者。MySQL 8.0.28 中添加。

+   `performance_schema_max_digest_sample_age`: 查询重新采样的时间范围（秒）。MySQL 8.0.3 中添加。

+   `performance_schema_show_processlist`: 选择 SHOW PROCESSLIST 实现。MySQL 8.0.22 中添加。

+   `persist_only_admin_x509_subject`: 启用持久化受限制系统变量的 SSL 证书 X.509 主题。MySQL 8.0.14 中添加。

+   `persist_sensitive_variables_in_plaintext`: 服务器是否允许以未加密格式存储敏感系统变量的值。MySQL 8.0.29 中添加。

+   `persisted_globals_load`: 是否加载持久化配置设置。MySQL 8.0.0 中添加。

+   `print_identified_with_as_hex`: 对于 SHOW CREATE USER，以十六进制打印包含不可打印字符的哈希值。MySQL 8.0.17 中添加。

+   `protocol_compression_algorithms`: 用于传入连接的允许压缩算法。MySQL 8.0.18 中添加。

+   `pseudo_replica_mode`: 供内部服务器使用。MySQL 8.0.26 中添加。

+   `regexp_stack_limit`: 正则表达式匹配堆栈大小限制。MySQL 8.0.4 中添加。

+   `regexp_time_limit`: 正则表达式匹配超时时间。MySQL 8.0.4 中添加。

+   `replica_checkpoint_group`: 多线程复制处理的最大事务数，在调用检查点操作以更新进度状态之前。不支持 NDB Cluster。MySQL 8.0.26 中添加。

+   `replica_checkpoint_period`: 更新多线程复制的进度状态，并在此毫秒数后将中继日志信息刷新到磁盘。不受 NDB Cluster 支持。在 MySQL 8.0.26 中添加。

+   `replica_compressed_protocol`: 使用源/复制协议的压缩。在 MySQL 8.0.26 中添加。

+   `replica_exec_mode`: 允许在 IDEMPOTENT 模式（键和一些其他错误被抑制）和 STRICT 模式之间切换复制线程；STRICT 模式是默认的，除了 NDB Cluster，在那里始终使用 IDEMPOTENT。在 MySQL 8.0.26 中添加。

+   `replica_load_tmpdir`: 复制时复制应该将其临时文件放置的位置 LOAD DATA 语句。在 MySQL 8.0.26 中添加。

+   `replica_max_allowed_packet`: 可以从复制源服务器发送到复制品的数据包的最大大小（以字节为单位）；覆盖 max_allowed_packet。在 MySQL 8.0.26 中添加。

+   `replica_net_timeout`: 等待来自源/复制连接的更多数据的秒数，然后中止读取。在 MySQL 8.0.26 中添加。

+   `replica_parallel_type`: 告诉复制使用时间戳信息（LOGICAL_CLOCK）或数据库分区（DATABASE）来并行化事务。在 MySQL 8.0.26 中添加。

+   `replica_parallel_workers`: 用于执行复制事务的应用程序线程数；当为 0 或 1 时，只有一个应用程序线程。NDB Cluster：请参阅文档。在 MySQL 8.0.26 中添加。

+   `replica_pending_jobs_size_max`: 持有尚未应用的事件的复制工作者队列的最大大小。在 MySQL 8.0.26 中添加。

+   `replica_preserve_commit_order`: 确保复制工作者的所有提交按照源上的顺序发生，以保持一致性在使用并行应用程序线程时。在 MySQL 8.0.26 中添加。

+   `replica_skip_errors`: 告诉复制线程在查询从提供的列表返回错误时继续复制。在 MySQL 8.0.26 中添加。

+   `replica_sql_verify_checksum`: 导致复制在从中继日志读取时检查校验和。在 MySQL 8.0.26 中添加。

+   `replica_transaction_retries`: 在事务由于死锁或超时而失败时，复制 SQL 线程重试事务的次数，然后放弃并停止。MySQL 8.0.26 中添加。

+   `replica_type_conversions`: 控制副本上的类型转换模式。值是以下列表中零个或多个元素的列表：ALL_LOSSY，ALL_NON_LOSSY。设置为空字符串以禁止源端和副本之间的类型转换。MySQL 8.0.26 中添加。

+   `replication_optimize_for_static_plugin_config`: 半同步复制的共享锁。MySQL 8.0.23 中添加。

+   `replication_sender_observe_commit_only`: 半同步复制的有限回调。MySQL 8.0.23 中添加。

+   `require_row_format`: 供内部服务器使用。MySQL 8.0.19 中添加。

+   `resultset_metadata`: 服务器是否返回结果集元数据。MySQL 8.0.3 中添加。

+   `rewriter_enabled_for_threads_without_privilege_checks`: 如果设置为 OFF，则对于执行时禁用权限检查的复制线程（PRIVILEGE_CHECKS_USER 为 NULL）跳过重写。MySQL 8.0.31 中添加。

+   `rpl_read_size`: 从二进制日志文件和中继日志文件中读取的最小数据量（以字节为单位）。MySQL 8.0.11 中添加。

+   `rpl_semi_sync_replica_enabled`: 副本是否启用半同步复制。MySQL 8.0.26 中添加。

+   `rpl_semi_sync_replica_trace_level`: 副本上的半同步复制调试跟踪级别。MySQL 8.0.26 中添加。

+   `rpl_semi_sync_source_enabled`: 源端是否启用半同步复制。MySQL 8.0.26 中添加。

+   `rpl_semi_sync_source_timeout`: 等待副本确认的毫秒数。MySQL 8.0.26 中添加。

+   `rpl_semi_sync_source_trace_level`: 源端上的半同步复制调试跟踪级别。MySQL 8.0.26 中添加。

+   `rpl_semi_sync_source_wait_for_replica_count`: 源端在继续之前必须收到每个事务的副本确认数。MySQL 8.0.26 中添加。

+   `rpl_semi_sync_source_wait_no_replica`: 是否源在没有副本的情况下等待超时。在 MySQL 8.0.26 中添加。

+   `rpl_semi_sync_source_wait_point`: 副本事务接收确认的等待点。在 MySQL 8.0.26 中添加。

+   `rpl_stop_replica_timeout`: STOP REPLICA 等待超时的秒数。在 MySQL 8.0.26 中添加。

+   `schema_definition_cache`: 可以在字典对象缓存中保留的模式定义对象的数量。在 MySQL 8.0.0 中添加。

+   `secondary_engine_cost_threshold`: 查询转移到辅助引擎的优化器成本阈值。在 MySQL 8.0.16 中添加。

+   `select_into_buffer_size`: 用于 OUTFILE 或 DUMPFILE 导出文件的缓冲区大小；覆盖 read_buffer_size。在 MySQL 8.0.22 中添加。

+   `select_into_disk_sync`: 在刷新 OUTFILE 或 DUMPFILE 导出文件的缓冲区后，与存储设备同步数据；OFF 禁用同步，是默认值。在 MySQL 8.0.22 中添加。

+   `select_into_disk_sync_delay`: 当 select_into_sync_disk = ON 时，设置每次同步 OUTFILE 或 DUMPFILE 导出文件缓冲区后的延迟时间（以毫秒为单位），否则不起作用。在 MySQL 8.0.22 中添加。

+   `show-replica-auth-info`: 在此源上的 SHOW REPLICAS 中显示用户名和密码。在 MySQL 8.0.26 中添加。

+   `show_create_table_skip_secondary_engine`: 是否从 SHOW CREATE TABLE 输出中排除 SECONDARY ENGINE 子句。在 MySQL 8.0.18 中添加。

+   `show_create_table_verbosity`: 是否在 SHOW CREATE TABLE 中显示 ROW_FORMAT，即使它具有默认值。在 MySQL 8.0.11 中添加。

+   `show_gipk_in_create_table_and_information_schema`: 是否在 SHOW 语句和 INFORMATION_SCHEMA 表中显示生成的不可见主键。在 MySQL 8.0.30 中添加。

+   `skip-replica-start`: 如果设置，则在副本服务器启动时不会自动启动复制。在 MySQL 8.0.26 中添加。

+   `source_verify_checksum`: 导致源在从二进制日志读取时检查校验和。在 MySQL 8.0.26 中添加。

+   `sql_generate_invisible_primary_key`: 是否为在此服务器上创建的任何没有显式主键的 InnoDB 表生成不可见主键。MySQL 8.0.30 中添加。

+   `sql_replica_skip_counter`: 复制应跳过的源事件数量。与 GTID 复制不兼容。MySQL 8.0.26 中添加。

+   `sql_require_primary_key`: 表是否必须有主键。MySQL 8.0.13 中添加。

+   `ssl_fips_mode`: 是否在服务器端启用 FIPS 模式。MySQL 8.0.11 中添加。

+   `ssl_session_cache_mode`: 是否启用服务器生成会话票据。MySQL 8.0.29 中添加。

+   `ssl_session_cache_timeout`: SSL 会话超时值（秒）。MySQL 8.0.29 中添加。

+   `sync_source_info`: 每 # 个事件后同步源信息。MySQL 8.0.26 中添加。

+   `syseventlog.facility`: syslog 消息的设施。MySQL 8.0.13 中添加。

+   `syseventlog.include_pid`: 是否在 syslog 消息中包含服务器 PID。MySQL 8.0.13 中添加。

+   `syseventlog.tag`: syslog 消息中服务器标识符的标记。MySQL 8.0.13 中添加。

+   `table_encryption_privilege_check`: 启用 TABLE_ENCRYPTION_ADMIN 权限检查。MySQL 8.0.16 中添加。

+   `tablespace_definition_cache`: 可以保存在字典对象缓存中的表空间定义对象的数量。MySQL 8.0.0 中添加。

+   `temptable_max_mmap`: TempTable 存储引擎可以从内存映射临时文件中分配的最大内存量。MySQL 8.0.23 中添加。

+   `temptable_max_ram`: 定义在数据存储在磁盘之前 TempTable 存储引擎可以占用的最大内存量。MySQL 8.0.2 中添加。

+   `temptable_use_mmap`: 定义当达到 temptable_max_ram 阈值时，TempTable 存储引擎是否分配内存映射文件。MySQL 8.0.16 中添加。

+   `terminology_use_previous`: 在不兼容更改的情况下使用指定版本之前的术语。MySQL 8.0.26 中添加。

+   `thread_pool_algorithm`: 线程池算法。MySQL 8.0.11 中添加。

+   `thread_pool_dedicated_listeners`: 在每个线程组中专门指定一个监听线程以监听网络事件。MySQL 8.0.23 中添加。

+   `thread_pool_high_priority_connection`: 当前会话是否为高优先级。MySQL 8.0.11 中添加。

+   `thread_pool_max_active_query_threads`: 每个组的活动查询线程的最大允许数量。MySQL 8.0.19 中添加。

+   `thread_pool_max_transactions_limit`: 线程池操作期间允许的最大事务数。MySQL 8.0.23 中添加。

+   `thread_pool_max_unused_threads`: 最大允许的未使用线程数。MySQL 8.0.11 中添加。

+   `thread_pool_prio_kickup_timer`: 语句移至高优先级执行前的时间。MySQL 8.0.11 中添加。

+   `thread_pool_query_threads_per_group`: 每个线程组的最大查询线程数。MySQL 8.0.31 中添加。

+   `thread_pool_size`: 线程池中线程组的数量。MySQL 8.0.11 中添加。

+   `thread_pool_stall_limit`: 语句被定义为停滞之前的时间。MySQL 8.0.11 中添加。

+   `thread_pool_transaction_delay`: 线程池执行新事务之前的延迟时间。MySQL 8.0.31 中添加。

+   `tls_ciphersuites`: 用于加密连接的 TLSv1.3 密码套件。MySQL 8.0.16 中添加。

+   `upgrade`: 控制启动时的自动升级。MySQL 8.0.16 中添加。

+   `use_secondary_engine`: 是否使用辅助引擎执行查询。MySQL 8.0.13 中添加。

+   `validate-config`: 验证服务器配置。MySQL 8.0.16 中添加。

+   `validate_password.changed_characters_percentage`: 新密码所需更改字符的最低百分比。MySQL 8.0.34 中添加。

+   `validate_password.check_user_name`: 是否检查密码与用户名是否匹配。MySQL 8.0.4 中添加。

+   `validate_password.dictionary_file`: validate_password 字典文件。在 MySQL 8.0.4 中添加。

+   `validate_password.dictionary_file_last_parsed`: 上次解析字典文件的时间。在 MySQL 8.0.4 中添加。

+   `validate_password.dictionary_file_words_count`: 字典文件中的单词数量。在 MySQL 8.0.4 中添加。

+   `validate_password.length`: validate_password 要求的密码长度。在 MySQL 8.0.4 中添加。

+   `validate_password.mixed_case_count`: validate_password 要求的大写/小写字符数量。在 MySQL 8.0.4 中添加。

+   `validate_password.number_count`: validate_password 要求的数字字符数量。在 MySQL 8.0.4 中添加。

+   `validate_password.policy`: validate_password 密码策略。在 MySQL 8.0.4 中添加。

+   `validate_password.special_char_count`: validate_password 要求的特殊字符数量。在 MySQL 8.0.4 中添加。

+   `version_compile_zlib`: 编译的 zlib 库版本。在 MySQL 8.0.11 中添加。

+   `windowing_use_high_precision`: 是否计算窗口函数以高精度。在 MySQL 8.0.2 中添加。

### MySQL 8.0 中已弃用的选项和变量

以下系统变量、状态变量和选项已在 MySQL 8.0 中弃用。

+   `Compression`: 客户端连接是否在客户端/服务器协议中使用压缩。在 MySQL 8.0.18 中已弃用。

+   `Slave_open_temp_tables`: 复制 SQL 线程当前打开的临时表数量。在 MySQL 8.0.26 中已弃用。

+   `Slave_rows_last_search_algorithm_used`: 此副本最近用于定位行以进行基于行的复制（索引、表或哈希扫描）的搜索算法。在 MySQL 8.0.26 中已弃用。

+   `abort-slave-event-count`: 用于调试和测试复制的 mysql-test 使用的选项。在 MySQL 8.0.29 中已弃用。

+   `admin-ssl`: 启用连接加密。在 MySQL 8.0.26 中已弃用。

+   `audit_log_connection_policy`: 连接相关事件的审计日志策略。在 MySQL 8.0.34 中已弃用。

+   `audit_log_exclude_accounts`: 不需要审计的账户。在 MySQL 8.0.34 中已弃用。

+   `audit_log_include_accounts`: 需要审计的账户。在 MySQL 8.0.34 中已弃用。

+   `audit_log_policy`: 审计日志策略。在 MySQL 8.0.34 中已弃用。

+   `audit_log_statement_policy`: 语句相关事件的审计日志策略。在 MySQL 8.0.34 中已弃用。

+   `authentication_fido_rp_id`: FIDO 多因素认证的 Relying party ID。在 MySQL 8.0.35 中已弃用。

+   `binlog_format`: 指定二进制日志的格式。在 MySQL 8.0.34 中已弃用。

+   `binlog_transaction_dependency_tracking`: 从中评估副本的多线程应用程序可以并行执行哪些事务的依赖信息来源（提交时间戳或事务写入集）。在 MySQL 8.0.35 中已弃用。

+   `character-set-client-handshake`: 在握手期间不忽略客户端字符集值。在 MySQL 8.0.35 中已弃用。

+   `daemon_memcached_enable_binlog`: . 在 MySQL 8.0.22 中已弃用。

+   `daemon_memcached_engine_lib_name`: 实现 InnoDB memcached 插件的共享库。在 MySQL 8.0.22 中已弃用。

+   `daemon_memcached_engine_lib_path`: 包含实现 InnoDB memcached 插件的共享库的目录。在 MySQL 8.0.22 中已弃用。

+   `daemon_memcached_option`: 在启动时传递给底层 memcached 守护程序的以空格分隔的选项。在 MySQL 8.0.22 中已弃用。

+   `daemon_memcached_r_batch_size`: 在开始新事务之前执行多少个 memcached 读操作。在 MySQL 8.0.22 中已弃用。

+   `daemon_memcached_w_batch_size`: 在开始新事务之前执行多少个 memcached 写操作。在 MySQL 8.0.22 中已弃用。

+   `default_authentication_plugin`: 默认认证插件。在 MySQL 8.0.27 中已弃用。

+   `disconnect-slave-event-count`: 用于调试和测试复制的 mysql-test 选项。在 MySQL 8.0.29 中已弃用。

+   `expire_logs_days`: 在这么多天后清除二进制日志。在 MySQL 8.0.3 中已弃用。

+   `group_replication_ip_whitelist`: 允许连接到组的主机列表。在 MySQL 8.0.22 中已弃用。

+   `group_replication_primary_member`: 当组以单主模式运行时的主要成员 UUID。如果组以多主模式运行，则为空字符串。在 MySQL 8.0.4 中已弃用。

+   `group_replication_recovery_complete_at`: 在状态传输后处理缓存事务时的恢复策略。在 MySQL 8.0.34 中已弃用。

+   `have_openssl`: mysqld 是否支持 SSL 连接。在 MySQL 8.0.26 中已弃用。

+   `have_ssl`: mysqld 是否支持 SSL 连接。在 MySQL 8.0.26 中已弃用。

+   `init_slave`: 当复制连接到源时执行的语句。在 MySQL 8.0.26 中已弃用。

+   `innodb_api_bk_commit_interval`: 多久自动提交使用 InnoDB memcached 接口的空��连接，单位为秒。在 MySQL 8.0.22 中已弃用。

+   `innodb_api_disable_rowlock`: . 在 MySQL 8.0.22 中已弃用。

+   `innodb_api_enable_binlog`: 允许使用 InnoDB memcached 插件与 MySQL 二进制日志。在 MySQL 8.0.22 中已弃用。

+   `innodb_api_enable_mdl`: 锁定 InnoDB memcached 插件使用的表，以防止通过 SQL 接口的 DDL 删除或更改。在 MySQL 8.0.22 中已弃用。

+   `innodb_api_trx_level`: 允许控制由 memcached 接口处理的查询的事务隔离级别。在 MySQL 8.0.22 中已弃用。

+   `innodb_log_file_size`: 日志组中每个日志文件的大小。在 MySQL 8.0.30 中已弃用。

+   `innodb_log_files_in_group`: 日志组中的 InnoDB 日志文件数。在 MySQL 8.0.30 中已弃用。

+   `innodb_undo_tablespaces`: 回滚段分布在的表空间文件数。在 MySQL 8.0.4 中已弃用。

+   `keyring_encrypted_file_data`: keyring_encrypted_file 插件数据文件。在 MySQL 8.0.34 中已弃用。

+   `keyring_encrypted_file_password`: keyring_encrypted_file 插件密码。在 MySQL 8.0.34 中已弃用。

+   `keyring_file_data`: keyring_file 插件数据文件。在 MySQL 8.0.34 中已弃用。

+   `keyring_oci_ca_certificate`: 用于对等身份验证的 CA 证书文件。在 MySQL 8.0.31 中已弃用。

+   `keyring_oci_compartment`: OCI 隔间 OCID。在 MySQL 8.0.31 中已弃用。

+   `keyring_oci_encryption_endpoint`: OCI 加密服务器端点。在 MySQL 8.0.31 中已弃用。

+   `keyring_oci_key_file`: OCI RSA 私钥文件。在 MySQL 8.0.31 中已弃用。

+   `keyring_oci_key_fingerprint`: OCI RSA 私钥文件指纹。在 MySQL 8.0.31 中已弃用。

+   `keyring_oci_management_endpoint`: OCI 管理服务器端点。在 MySQL 8.0.31 中已弃用。

+   `keyring_oci_master_key`: OCI 主密钥 OCID。在 MySQL 8.0.31 中已弃用。

+   `keyring_oci_secrets_endpoint`: OCI 秘密服务器端点。在 MySQL 8.0.31 中已弃用。

+   `keyring_oci_tenancy`: OCI 租户 OCID。在 MySQL 8.0.31 中已弃用。

+   `keyring_oci_user`: OCI 用户 OCID。在 MySQL 8.0.31 中已弃用。

+   `keyring_oci_vaults_endpoint`: OCI vaults 服务器端点。在 MySQL 8.0.31 中已弃用。

+   `keyring_oci_virtual_vault`: OCI 保险库 OCID。在 MySQL 8.0.31 中已弃用。

+   `log_bin_trust_function_creators`: 如果等于 0（默认值），那么当使用 --log-bin 时，只有具有 SUPER 特权的用户才允许存储函数创建，并且只有在创建的函数不会破坏二进制日志记录时才允许。在 MySQL 8.0.34 中已弃用。

+   `log_bin_use_v1_row_events`: 服务器是否使用版本 1 的二进制日志行事件。在 MySQL 8.0.18 中已弃用。

+   `log_slave_updates`: 复制是否应该将其复制 SQL 线程执行的更新记录到自己的二进制日志中。在 MySQL 8.0.26 中已弃用。

+   `log_slow_slave_statements`: 导致副本执行的慢查询被写入慢查询日志。在 MySQL 8.0.26 中已弃用。

+   `log_statements_unsafe_for_binlog`: 禁用写入错误日志的错误 1592 警告。在 MySQL 8.0.34 中已弃用。

+   `log_syslog`: 是否将错误日志写入 syslog。在 MySQL 8.0.2 中已弃用。

+   `master-info-file`: 记录源和 I/O 复制线程在源二进制日志中位置的文件位置和名称。在 MySQL 8.0.18 中已弃用。

+   `master_info_repository`: 是否将连接元数据存储库写入文件或表，其中包含源信息和源二进制日志中复制 I/O 线程位置。在 MySQL 8.0.23 中已弃用。

+   `master_verify_checksum`: 导致源在读取二进制日志时检查校验和。在 MySQL 8.0.26 中已弃用。

+   `max_length_for_sort_data`: 排序记录中的最大字节数。在 MySQL 8.0.20 中已弃用。

+   `myisam_repair_threads`: 修复 MyISAM 表时要使用的线程数。1 表示禁用并行修复。在 MySQL 8.0.29 中已弃用。

+   `new`: 使用非常新的，可能是“不安全”的功能。在 MySQL 8.0.35 中已弃用。

+   `no-dd-upgrade`: 防止在启动时自动升级数据字典表。在 MySQL 8.0.16 中已弃用。

+   `old`: 导致服务器恢复到旧版本中存在的某些行为。在 MySQL 8.0.35 中已弃用。

+   `old-style-user-limits`: 启用旧式用户限制（在 5.0.3 之前，用户资源是按每个用户+主机计算而不是按账户计算）。在 MySQL 8.0.30 中已弃用。

+   `performance_schema_show_processlist`: 选择 SHOW PROCESSLIST 的实现。在 MySQL 8.0.35 中已弃用。

+   `pseudo_slave_mode`: 供内部服务器使用。在 MySQL 8.0.26 中已弃用。

+   `query_prealloc_size`: 用于查询解析和执行的持久缓冲区。在 MySQL 8.0.29 中已弃用。

+   `relay_log_info_file`: 应用程序元数据存储库的文件名，副本在其中记录有关中继日志的信息。在 MySQL 8.0.18 中已弃用。

+   `relay_log_info_repository`: 是否将复制 SQL 线程的位置写入中继日志到文件或表中。在 MySQL 8.0.23 中已弃用。

+   `replica_parallel_type`: 告诉复制使用时间戳信息（LOGICAL_CLOCK）或数据库分区（DATABASE）来并行化事务。在 MySQL 8.0.29 中已弃用。

+   `rpl_stop_slave_timeout`: STOP REPLICA 或 STOP SLAVE 在超时之前等待的秒数。在 MySQL 8.0.26 中已弃用。

+   `safe-user-create`: 不允许没有对 mysql.user 表具有写权限的用户创建新用户；此选项已弃用并被忽略。在 MySQL 8.0.11 中已弃用。

+   `show-slave-auth-info`: 在此源上在 SHOW REPLICAS 和 SHOW SLAVE HOSTS 中显示用户名和密码。在 MySQL 8.0.26 中已弃用。

+   `skip-character-set-client-handshake`: 在握手期间忽略客户���端发送的字符集值。在 MySQL 8.0.35 中已弃用。

+   `skip-host-cache`: 不缓存主机名。在 MySQL 8.0.30 中已弃用。

+   `skip-new`: 不使用新的、可能错误的例程。在 MySQL 8.0.35 中已弃用。

+   `skip-slave-start`: 如果设置，当复制服务器启动时不会自动启动复制。在 MySQL 8.0.26 中已弃用。

+   `slave-skip-errors`: 告诉复制线程在查询从提供的列表返回错误时继续复制。在 MySQL 8.0.26 中已弃用。

+   `slave_checkpoint_group`: 在调用检查点操作更新进度状态之前，多线程复制处理的最大事务数。不支持 NDB Cluster。在 MySQL 8.0.26 中已弃用。

+   `slave_checkpoint_period`: 在此毫秒数后更新多线程复制的进度状态，并将中继日志信息刷新到磁盘。不支持 NDB Cluster。在 MySQL 8.0.26 中已弃用。

+   `slave_compressed_protocol`: 使用源/复制协议的压缩。在 MySQL 8.0.18 中已弃用。

+   `slave_load_tmpdir`: 复制 LOAD DATA 语句时，复制应将临时文件放置的位置。在 MySQL 8.0.26 中已弃用。

+   `slave_max_allowed_packet`: 可以从复制源服务器发送到复制品的数据包的最大大小（以字节为单位）；覆盖 max_allowed_packet。在 MySQL 8.0.26 中已弃用。

+   `slave_net_timeout`: 等待源/复制连接中更多数据的秒数，然后中止读取。在 MySQL 8.0.26 中已弃用。

+   `slave_parallel_type`: 告诉复制使用时间戳信息（LOGICAL_CLOCK）或数据库分区（DATABASE）来并行化事���。在 MySQL 8.0.26 中已弃用。

+   `slave_parallel_workers`: 用于并行执行复制事务的应用程序线程数；0 或 1 禁用复制多线程。NDB Cluster：请参阅文档。在 MySQL 8.0.26 中已弃用。

+   `slave_pending_jobs_size_max`: 持有尚未应用的事件的复制工作者队列的最大大小。在 MySQL 8.0.26 中已弃用。

+   `slave_preserve_commit_order`: 确保复制工作者的所有提交按照源上的顺序发生，以保持一致性，当使用并行应用程序线程时。在 MySQL 8.0.26 中已弃用。

+   `slave_rows_search_algorithms`: 确定用于复制更新批处理的搜索算法。从此列表中选择任意 2 或 3 个：INDEX_SEARCH，TABLE_SCAN，HASH_SCAN。在 MySQL 8.0.18 中已弃用。

+   `slave_sql_verify_checksum`: 导致复制时从中继日志读取时检查校验和。在 MySQL 8.0.26 中已弃用。

+   `slave_transaction_retries`: 复制 SQL 线程在事务失败（死锁或已过锁等待超时）时重试事务的次数，然后放弃并停止。在 MySQL 8.0.26 中已弃用。

+   `slave_type_conversions`: 控制复制上的类型转换模式。值是此列表中零个或多个元素：ALL_LOSSY，ALL_NON_LOSSY。设置为空字符串以禁止源和复制之间的类型转换。在 MySQL 8.0.26 中已弃用。

+   `sql_slave_skip_counter`: 复制应跳过的源事件数。与 GTID 复制不兼容。在 MySQL 8.0.26 中已弃用。

+   `ssl`: 启用连接加密。在 MySQL 8.0.26 中已弃用。

+   `ssl_fips_mode`: 是否在服务器端启用 FIPS 模式。在 MySQL 8.0.34 中已弃用。

+   `symbolic-links`: 允许 MyISAM 表使用符号链接。在 MySQL 8.0.2 中已弃用。

+   `sync_master_info`: 每第#个事件后同步源信息。在 MySQL 8.0.26 中已弃用。

+   `sync_relay_log_info`: 每第#个事件后将 relay.info 文件同步到磁盘。在 MySQL 8.0.34 中已弃用。

+   `temptable_use_mmap`: 当 temptable_max_ram 阈值达到时，TempTable 存储引擎是否分配内存映射文件。在 MySQL 8.0.26 中已弃用。

+   `transaction_prealloc_size`: 用于存储二进制日志中事务的持久缓冲区。在 MySQL 8.0.29 中已弃用。

+   `transaction_write_set_extraction`: 定义在事务期间提取的写入哈希算法。在 MySQL 8.0.26 中已弃用。

### 在 MySQL 8.0 中已移除的选项和变量

以下系统变量、状态变量和选项已在 MySQL 8.0 中移除。

+   `Com_alter_db_upgrade`: ALTER DATABASE ... UPGRADE DATA DIRECTORY NAME 语句的计数。在 MySQL 8.0.0 中已移除。

+   `Innodb_available_undo_logs`: InnoDB 回滚段的总数；与 innodb_rollback_segments 不同，后者显示活动回滚段的数量。在 MySQL 8.0.2 中已移除。

+   `Qcache_free_blocks`: 查询缓存中的空闲内存块数。在 MySQL 8.0.3 中已移除。

+   `Qcache_free_memory`: 查询缓存的空闲内存量。在 MySQL 8.0.3 中已移除。

+   `Qcache_hits`: 查询缓存命中次数。在 MySQL 8.0.3 中已移除。

+   `Qcache_inserts`: 查询缓存插入次数。在 MySQL 8.0.3 中已移除。

+   `Qcache_lowmem_prunes`: 由于缓存中内存不足而从查询缓存中删除的查询数量。在 MySQL 8.0.3 中已移除。

+   `Qcache_not_cached`: 非缓存查询的数量（不可缓存或由于 query_cache_type 设置而未缓存）。在 MySQL 8.0.3 中已移除。

+   `Qcache_queries_in_cache`: 查询缓存中注册的查询数量。在 MySQL 8.0.3 中已移除。

+   `Qcache_total_blocks`: 查询缓存中的总块数。在 MySQL 8.0.3 中已移除。

+   `Slave_heartbeat_period`: 复制的心跳间隔，单位为秒。在 MySQL 8.0.1 中已移除。

+   `Slave_last_heartbeat`: 显示最新心跳信号接收时间，格式为时间戳。在 MySQL 8.0.1 中已移除。

+   `Slave_received_heartbeats`: 自上次重置以来副本接收的心跳数。在 MySQL 8.0.1 中已移除。

+   `Slave_retried_transactions`: 自启动以来，复制 SQL 线程重试事务的总次数。在 MySQL 8.0.1 中已移除。

+   `Slave_running`: 该服务器作为副本的状态（复制 I/O 线程状态）。在 MySQL 8.0.1 中已移除。

+   `bootstrap`: mysql 安装脚本使用。在 MySQL 8.0.0 中已移除。

+   `date_format`: 日期格式（未使用）。在 MySQL 8.0.3 中移除。

+   `datetime_format`: DATETIME/TIMESTAMP 格式（未使用）。在 MySQL 8.0.3 中移除。

+   `des-key-file`: 从给定文件加载 des_encrypt()和 des_encrypt 的密钥。在 MySQL 8.0.3 中移除。

+   `group_replication_allow_local_disjoint_gtids_join`: 允许当前服务器加入组，即使它具有组中不存在的事务。在 MySQL 8.0.4 中移除。

+   `have_crypt`: crypt()系统调用的可用性。在 MySQL 8.0.3 中移除。

+   `ignore-db-dir`: 将目录视为非数据库目录。在 MySQL 8.0.0 中移除。

+   `ignore_builtin_innodb`: 忽略内置 InnoDB。在 MySQL 8.0.3 中移除。

+   `ignore_db_dirs`: 视为非数据库目录的目录。在 MySQL 8.0.0 中移除。

+   `innodb_checksums`: 启用 InnoDB 校验和验证。在 MySQL 8.0.0 中移除。

+   `innodb_disable_resize_buffer_pool_debug`: 禁用 InnoDB 缓冲池的调整大小。在 MySQL 8.0.0 中移除。

+   `innodb_file_format`: 新 InnoDB 表的格式。在 MySQL 8.0.0 中移除。

+   `innodb_file_format_check`: InnoDB 是否执行文件格式兼容性检查。在 MySQL 8.0.0 中移除。

+   `innodb_file_format_max`: 共享表空间中的文件格式标记。在 MySQL 8.0.0 中移除。

+   `innodb_large_prefix`: 启用更长的列前缀索引键。在 MySQL 8.0.0 中移除。

+   `innodb_locks_unsafe_for_binlog`: 强制 InnoDB 不使用 next-key 锁定，而只使用行级锁定。在 MySQL 8.0.0 中移除。

+   `innodb_scan_directories`: 定义在 InnoDB 恢复期间扫描表空间文件的目录。在 MySQL 8.0.4 中移除。

+   `innodb_stats_sample_pages`: 用于索引分布统计的样本页数。在 MySQL 8.0.0 中移除。

+   `innodb_support_xa`: 启用 InnoDB 对 XA 两阶段提交的支持。在 MySQL 8.0.0 中移除。

+   `innodb_undo_logs`: InnoDB 使用的撤销日志（回滚段）数量；innodb_rollback_segments 的别名。在 MySQL 8.0.2 中移除。

+   `internal_tmp_disk_storage_engine`: 内部临时表的存储引擎。在 MySQL 8.0.16 中移除。

+   `log-warnings`: 将一些非关键警告写入日志文件。在 MySQL 8.0.3 中移除。

+   `log_builtin_as_identified_by_password`: 是否以向后兼容的方式记录 CREATE/ALTER USER、GRANT。在 MySQL 8.0.11 中移除。

+   `log_error_filter_rules`: 错误日志过滤规则。在 MySQL 8.0.4 中移除。

+   `log_syslog`: 是否将错误日志写入 syslog。在 MySQL 8.0.13 中移除。

+   `log_syslog_facility`: syslog 消息的设施。在 MySQL 8.0.13 中移除。

+   `log_syslog_include_pid`: 是否在 syslog 消息中包含服务器 PID。在 MySQL 8.0.13 中移除。

+   `log_syslog_tag`: syslog 消息中服务器标识符的标记。在 MySQL 8.0.13 中移除。

+   `max_tmp_tables`: 未使用。在 MySQL 8.0.3 中移除。

+   `metadata_locks_cache_size`: 元数据锁缓存的大小。在 MySQL 8.0.13 中已移除。

+   `metadata_locks_hash_instances`: 元数据锁哈希数。在 MySQL 8.0.13 中已移除。

+   `multi_range_count`: 在范围选择期间一次发送给表处理程序的最大范围数。在 MySQL 8.0.3 中已移除。

+   `myisam_repair_threads`: 修复 MyISAM 表时要使用的线程数。1 禁用并行修复。在 MySQL 8.0.30 中已移除。

+   `old_passwords`: 为 PASSWORD() 选择密码哈希方法。在 MySQL 8.0.11 中已移除。

+   `partition`: 启用（或禁用）分区支持。在 MySQL 8.0.0 中已移除。

+   `query_cache_limit`: 不缓存大于此大小的结果。在 MySQL 8.0.3 中已移除。

+   `query_cache_min_res_unit`: 为结果分配空间的最小单位大小（在写入所有结果数据后修剪最后一个单位）。在 MySQL 8.0.3 中已移除。

+   `query_cache_size`: 用于存储旧查询结果的内存分配。在 MySQL 8.0.3 中已移除。

+   `query_cache_type`: 查询缓存类型。在 MySQL 8.0.3 中已移除。

+   `query_cache_wlock_invalidate`: 在写入锁定时使查询缓存中的查询无效。在 MySQL 8.0.3 中已移除。

+   `secure_auth`: 禁止具有旧（4.1 之前）密码的帐户进行身份验证。在 MySQL 8.0.3 中已移除。

+   `show_compatibility_56`: 用于 SHOW STATUS/VARIABLES 的兼容性。在 MySQL 8.0.1 中已移除。

+   `skip-partition`: 不启用用户定义的分区。在 MySQL 8.0.0 中已移除。

+   `sync_frm`: 在创建时将 .frm 同步到磁盘。默认情况下启用。在 MySQL 8.0.0 中已移除。

+   `temp-pool`: 使用此选项会导致大多数创建的临时文件使用一组小的名称，而不是为每个新文件使用唯一名称。在 MySQL 8.0.1 中已移除。

+   `time_format`: TIME 格式（未使用）。在 MySQL 8.0.3 中已移除。

+   `tx_isolation`: 默认事务隔离级别。在 MySQL 8.0.3 中已移除。

+   `tx_read_only`: 默认事务访问模式。在 MySQL 8.0.3 中已移除。
