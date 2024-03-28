# 7.1.4 服务器选项、系统变量和状态变量参考

> 原文：[`dev.mysql.com/doc/refman/8.0/en/server-option-variable-reference.html`](https://dev.mysql.com/doc/refman/8.0/en/server-option-variable-reference.html)

以下表列出了所有适用于`mysqld`内的命令行选项、系统变量和状态变量。

该表列出了命令行选项（Cmd-line）、配置文件中有效的选项（Option file）、服务器系统变量（System Var）和状态变量（Status var）的统一列表，并指示每个选项或变量的有效位置。如果在命令行或配置文件中设置的服务器选项与相应系统变量的名称不同，则在相应选项的下方立即注明变量名称。对于系统和状态变量，变量的范围（Var Scope）是全局、会话或两者兼有。有关设置和使用选项和变量的详细信息，请参阅相应的项目描述。在适当的情况下，提供了有关项目的进一步信息的直接链接。

有关专用于 NDB Cluster 的此表版本，请参见第 25.4.2.5 节，“NDB Cluster mysqld 选项和变量参考”。

**表 7.1 命令行选项、系统变量和状态变量摘要**

| 名称 | 命令行 | 选项文件 | 系统变量 | 状态变量 | 变量范围 | 动态 |
| --- | --- | --- | --- | --- | --- | --- |
| 中止从属事件计数 | 是 | 是 |  |  |  |  |
| 中止的客户端 |  |  |  | 是 | 全局 | 否 |
| 中止连接 |  |  |  | 是 | 全局 | 否 |
| Acl_cache_items_count |  |  |  | 是 | 全局 | 否 |
| 登录时激活所有角色 | 是 | 是 | 是 |  | 全局 | 是 |
| admin_address | 是 | 是 | 是 |  | 全局 | 否 |
| admin_port | 是 | 是 | 是 |  | 全局 | 否 |
| admin-ssl | 是 | 是 |  |  |  |  |
| admin_ssl_ca | 是 | 是 | 是 |  | 全局 | 是 |
| admin_ssl_capath | 是 | 是 | 是 |  | 全局 | 是 |
| admin_ssl_cert | 是 | 是 | 是 |  | 全局 | 是 |
| admin_ssl_cipher | 是 | 是 | 是 |  | 全局 | 是 |
| admin_ssl_crl | 是 | 是 | 是 |  | 全局 | 是 |
| 管理员 SSL CRL 路径 | 是 | 是 | 是 |  | 全局 | 是 |
| 管理员 SSL 密钥 | 是 | 是 | 是 |  | 全局 | 是 |
| 管理员 TLS 密码套件 | 是 | 是 | 是 |  | 全局 | 是 |
| 管理员 TLS 版本 | 是 | 是 | 是 |  | 全局 | 是 |
| 允许可疑 UDF | 是 | 是 |  |  |  |  |
| ansi | 是 | 是 |  |  |  |  |
| 审计日志 | 是 | 是 |  |  |  |  |
| 审计日志缓冲区大小 | 是 | 是 | 是 |  | 全局 | 否 |
| 审计日志压缩 | 是 | 是 | 是 |  | 全局 | 否 |
| 审计日志连接策略 | 是 | 是 | 是 |  | 全局 | 是 |
| 当前会话的审计日志 |  |  | 是 |  | 两者 | 否 |
| 当前大小的审计日志 |  |  |  | 是 | 全局 | 否 |
| 审计日志数据库 | 是 | 是 | 是 |  | 全局 | 否 |
| 禁用审计日志 | 是 | 是 | 是 |  | 全局 | 是 |
| 审计日志加密 | 是 | 是 | 是 |  | 全局 | 否 |
| 审计日志事件最大丢弃大小 |  |  |  | 是 | 全局 | 否 |
| 审计日志事件 |  |  |  | 是 | 全局 | 否 |
| 过滤的审计日志事件 |  |  |  | 是 | 全局 | 否 |
| 丢失的审计日志事件 |  |  |  | 是 | 全局 | 否 |
| 已写入的审计日志事件 |  |  |  | 是 | 全局 | 否 |
| 排除账户的审计日志 | 是 | 是 | 是 |  | 全局 | 是 |
| 审计日志文件 | 是 | 是 | 是 |  | 全局 | 否 |
| 审计日志过滤器 ID |  |  | 是 |  | 两者 | 否 |
| 审计日志刷新 |  |  | 是 |  | 全局 | 是 |
| 审计日志刷新间隔秒数 | 是 |  | 是 |  | 全局 | 否 |
| 审计日志格式 | 是 | 是 | 是 |  | 全局 | 否 |
| audit_log_format_unix_timestamp | 是 | 是 | 是 |  | 全局 | 是 |
| audit_log_include_accounts | �� | 是 | 是 |  | 全局 | 是 |
| audit_log_max_size | 是 | 是 | 是 |  | 全局 | 是 |
| audit_log_password_history_keep_days | 是 | 是 | 是 |  | 全局 | 是 |
| audit_log_policy | 是 | 是 | 是 |  | 全局 | 否 |
| audit_log_prune_seconds | 是 | 是 | 是 |  | 全局 | 是 |
| audit_log_read_buffer_size | 是 | 是 | 是 |  | 变化 | 变化 |
| audit_log_rotate_on_size | 是 | 是 | 是 |  | 全局 | 是 |
| audit_log_statement_policy | 是 | 是 | 是 |  | 全局 | 是 |
| audit_log_strategy | 是 | 是 | 是 |  | 全局 | 否 |
| Audit_log_total_size |  |  |  | 是 | 全局 | 否 |
| Audit_log_write_waits |  |  |  | 是 | 全局 | 否 |
| authentication_fido_rp_id | 是 | 是 | 是 |  | 全局 | 是 |
| authentication_kerberos_service_key_tab | 是 | 是 | 是 |  | 全局 | 否 |
| authentication_kerberos_service_principal | 是 | 是 | 是 |  | 全局 | 是 |
| authentication_ldap_sasl_auth_method_name | 是 | 是 | 是 |  | 全局 | 是 |
| authentication_ldap_sasl_bind_base_dn | 是 | 是 | 是 |  | 全局 | 是 |
| authentication_ldap_sasl_bind_root_dn | 是 | 是 | 是 |  | 全局 | 是 |
| authentication_ldap_sasl_bind_root_pwd | 是 | 是 | 是 |  | 全局 | 是 |
| authentication_ldap_sasl_ca_path | 是 | 是 | 是 |  | 全局 | 是 |
| LDAP SASL 组搜索属性 | 是 | 是 | 是 |  | 全局 | 是 |
| LDAP SASL 组搜索过滤器 | 是 | 是 | 是 |  | 全局 | 是 |
| LDAP SASL 初始池大小 | 是 | 是 | 是 |  | 全局 | 是 |
| LDAP SASL 认证日志状态 | 是 | 是 | 是 |  | 全局 | 是 |
| LDAP SASL 最大池大小 | 是 | 是 | 是 |  | 全局 | 是 |
| LDAP SASL 转发 | 是 | 是 | 是 |  | ���局 | 是 |
| LDAP SASL 服务器主机 | 是 | 是 | 是 |  | 全局 | 是 |
| LDAP SASL 服务器端口 | 是 | 是 | 是 |  | 全局 | 是 |
| LDAP SASL 支持的方法 |  |  |  | 是 | 全局 | 否 |
| LDAP SASL TLS 认证 | 是 | 是 | 是 |  | 全局 | 是 |
| LDAP SASL 用户搜索属性 | 是 | 是 | 是 |  | 全局 | 是 |
| LDAP 简单认证方法名称 | 是 | 是 | 是 |  | 全局 | 是 |
| LDAP 简单绑定基础 DN | 是 | 是 | 是 |  | 全局 | 是 |
| LDAP 简单绑定根 DN | 是 | 是 | 是 |  | 全局 | 是 |
| LDAP 简单绑定根密码 | 是 | 是 | 是 |  | 全局 | 是 |
| LDAP 简单 CA 路径 | 是 | 是 | 是 |  | 全局 | 是 |
| LDAP 简单组搜索属性 | 是 | 是 | 是 |  | 全局 | 是 |
| authentication_ldap_simple_group_search_filter | 是 | 是 | 是 |  | 全局 | 是 |
| authentication_ldap_simple_init_pool_size | 是 | 是 | 是 |  | 全局 | 是 |
| authentication_ldap_simple_log_status | 是 | 是 | 是 |  | 全局 | 是 |
| authentication_ldap_simple_max_pool_size | 是 | 是 | 是 |  | 全局 | 是 |
| authentication_ldap_simple_referral | 是 | 是 | 是 |  | 全局 | 是 |
| authentication_ldap_simple_server_host | 是 | 是 | 是 |  | 全局 | 是 |
| authentication_ldap_simple_server_port | 是 | 是 | 是 |  | 全局 | 是 |
| authentication_ldap_simple_tls | 是 | 是 | 是 |  | 全局 | 是 |
| authentication_ldap_simple_user_search_attr | 是 | 是 | 是 |  | 全局 | 是 |
| authentication_policy | 是 | 是 | 是 |  | 全局 | 是 |
| authentication_windows_log_level | 是 | 是 | 是 |  | 全局 | 否 |
| authentication_windows_use_principal_name | 是 | 是 | 是 |  | 全局 | 否 |
| auto_generate_certs | 是 | 是 | 是 |  | 全局 | 否 |
| auto_increment_increment | 是 | 是 | 是 |  | 两者 | 是 |
| auto_increment_offset | 是 | 是 | 是 |  | 两者 | 是 |
| autocommit | 是 | 是 | 是 |  | 两者 | 是 |
| automatic_sp_privileges | 是 | 是 | 是 |  | 全局 | 是 |
| avoid_temporal_upgrade | 是 | 是 | 是 |  | 全局 | 是 |
| back_log | 是 | 是 | 是 |  | 全局 | 否 |
| basedir | 是 | 是 | 是 |  | 全局 | 否 |
| big_tables | 是 | 是 | 是 |  | 两者 | 是 |
| bind_address | 是 | 是 | 是 |  | 全局 | 否 |
| Binlog_cache_disk_use |  |  |  | 是 | 全局 | 否 |
| binlog_cache_size | 是 | 是 | 是 |  | 全局 | 是 |
| Binlog_cache_use |  |  |  | 是 | 全局 | 否 |
| binlog-checksum | 是 | 是 |  |  |  |  |
| binlog_checksum | 是 | 是 | 是 |  | 全局 | 是 |
| binlog_direct_non_transactional_updates | 是 | 是 | 是 |  | 两者 | 是 |
| binlog-do-db | 是 | 是 |  |  |  |  |
| binlog_encryption | 是 | 是 | 是 |  | 全局 | 是 |
| binlog_error_action | 是 | 是 | 是 |  | 全局 | 是 |
| binlog_expire_logs_auto_purge | 是 | 是 | 是 |  | 全局 | 是 |
| binlog_expire_logs_seconds | 是 | 是 | 是 |  | 全局 | 是 |
| binlog_format | 是 | 是 | 是 |  | 两者 | 是 |
| binlog_group_commit_sync_delay | 是 | 是 | 是 |  | 全局 | 是 |
| binlog_group_commit_sync_no_delay_count | 是 | 是 | 是 |  | 全局 | 是 |
| binlog_gtid_simple_recovery | 是 | 是 | 是 |  | 全局 | 否 |
| binlog-ignore-db | 是 | 是 |  |  |  |  |
| binlog_max_flush_queue_time | 是 | 是 | 是 |  | 全局 | 是 |
| binlog_order_commits | 是 | 是 | 是 |  | 全局 | 是 |
| binlog_rotate_encryption_master_key_at_startup | 是 | 是 | 是 |  | 全局 | 否 |
| binlog_row_event_max_size | 是 | 是 | 是 |  | 全局 | 否 |
| binlog_row_image | 是 | 是 | 是 |  | 两者 | 是 |
| binlog_row_metadata | 是 | 是 | 是 |  | 全局 | 是 |
| binlog_row_value_options | 是 | 是 | 是 |  | 两者 | 是 |
| binlog_rows_query_log_events | 是 | 是 | 是 |  | 两者 | 是 |
| Binlog_stmt_cache_disk_use |  |  |  | 是 | 全局 | 否 |
| binlog_stmt_cache_size | 是 | 是 | 是 |  | 全局 | 是 |
| Binlog_stmt_cache_use |  |  |  | 是 | 全局 | 否 |
| binlog_transaction_compression | 是 | 是 | 是 |  | 两者 | 是 |
| binlog_transaction_compression_level_zstd | 是 | 是 | 是 |  | 两者 | 是 |
| binlog_transaction_dependency_history_size | 是 | 是 | 是 |  | 全局 | 是 |
| binlog_transaction_dependency_tracking | 是 | 是 | 是 |  | 全局 | 是 |
| block_encryption_mode | 是 | 是 | 是 |  | 两者 | 是 |
| build_id |  |  | 是 |  | 全局 | 否 |
| bulk_insert_buffer_size | 是 | 是 | 是 |  | 两者 | 是 |
| Bytes_received |  |  |  | 是 | 两者 | 否 |
| Bytes_sent |  |  |  | 是 | 两者 | 否 |
| caching_sha2_password_auto_generate_rsa_keys | 是 | 是 | 是 |  | 全局 | 否 |
| caching_sha2_password_digest_rounds | 是 | 是 | 是 |  | 全局 | 否 |
| caching_sha2_password_private_key_path | 是 | 是 | 是 |  | 全局 | 否 |
| caching_sha2_password_public_key_path | 是 | 是 | 是 |  | 全局 | 否 |
| Caching_sha2_password_rsa_public_key |  |  |  | 是 | 全局 | 否 |
| character_set_client |  |  | 是 |  | 两者 | 是 |
| character-set-client-handshake | 是 | 是 |  |  |  |  |
| character_set_connection |  |  | 是 |  | 两者 | 是 |
| character_set_database (注 1) |  |  | 是 |  | 两者 | 是 |
| character_set_filesystem | 是 | 是 | 是 |  | 两者 | 是 |
| character_set_results |  |  | 是 |  | 两者 | 是 |
| character_set_server | 是 | 是 | 是 |  | 两者 | 是 |
| character_set_system |  |  | 是 |  | 全局 | 否 |
| character_sets_dir | 是 | 是 | 是 |  | 全局 | 否 |
| check_proxy_users | 是 | 是 | 是 |  | 全局 | 是 |
| chroot | 是 | 是 |  |  |  |  |
| clone_autotune_concurrency | 是 | 是 | 是 |  | 全局 | 是 |
| clone_block_ddl | 是 | 是 | 是 |  | 全局 | 是 |
| clone_buffer_size | 是 | 是 | 是 |  | 全局 | 是 |
| clone_ddl_timeout | 是 | 是 | 是 |  | 全局 | 是 |
| clone_delay_after_data_drop | 是 | 是 | 是 |  | 全局 | 是 |
| clone_donor_timeout_after_network_failure | ��� | 是 | 是 |  | 全局 | 是 |
| clone_enable_compression | 是 | 是 | 是 |  | 全局 | 是 |
| clone_max_concurrency | 是 | 是 | 是 |  | 全局 | 是 |
| clone_max_data_bandwidth | 是 | 是 | 是 |  | 全局 | 是 |
| clone_max_network_bandwidth | 是 | 是 | 是 |  | 全局 | 是 |
| clone_ssl_ca | 是 | 是 | 是 |  | 全局 | 是 |
| clone_ssl_cert | 是 | 是 | 是 |  | 全局 | 是 |
| clone_ssl_key | 是 | 是 | 是 |  | 全局 | 是 |
| clone_valid_donor_list | 是 | 是 | 是 |  | 全局 | 是 |
| collation_connection |  |  | 是 |  | 两者 | 是 |
| 数据库排序规则（注 1） |  |  | 是 |  | 两者 | 是 |
| 服务器排序规则 | 是 | 是 | 是 |  | 两者 | 是 |
| 管理命令 |  |  |  | 是 | 两者 | 否 |
| 修改数据库 |  |  |  | 是 | 两者 | 否 |
| 修改事件 |  |  |  | 是 | 两者 | 否 |
| 修改函数 |  |  |  | 是 | 两者 | 否 |
| 修改存储过程 |  |  |  | 是 | 两者 | 否 |
| 修改资源组 |  |  |  | 是 | 全局 | 否 |
| 修改服务器 |  |  |  | 是 | 两者 | 否 |
| 修改表 |  |  |  | 是 | 两者 | 否 |
| 修改表空间 |  |  |  | 是 | 两者 | 否 |
| 修改用户 |  |  |  | 是 | 两者 | 否 |
| 修改用户默认角色 |  |  |  | 是 | 全局 | 否 |
| 分析 |  |  |  | 是 | 两者 | 否 |
| 分配到键缓存 |  |  |  | 是 | 两者 | 否 |
| 开始 |  |  |  | 是 | 两者 | 否 |
| 二进制日志 |  |  |  | 是 | 两者 | 否 |
| 调用存储过程 |  |  |  | 是 | 两者 | 否 |
| 更改数据库 |  |  |  | 是 | 两者 | 否 |
| 更改主服务器 |  |  |  | 是 | 两者 | 否 |
| 更改复制过滤器 |  |  |  | 是 | 两者 | 否 |
| 更改复制源 |  |  |  | 是 | 两者 | 否 |
| 检查 |  |  |  | 是 | 两者 | 否 |
| 校验 |  |  |  | 是 | 两者 | 否 |
| 克隆 |  |  |  | 是 | 全局 | 否 |
| 提交 |  |  |  | 是 | 两者 | 否 |
| 创建数据库 |  |  |  | 是 | 两者 | 否 |
| 创建事件 |  |  |  | 是 | 两者 | 否 |
| 创建函数 |  |  |  | 是 | 两者 | 否 |
| 创建索引 |  |  |  | 是 | 两者 | 否 |
| 创建存储过程 |  |  |  | 是 | 两者 | 否 |
| 创建资源组 |  |  |  | 是 | 全局 | 否 |
| 创建角色 |  |  |  | 是 | 全局 | 否 |
| 创建服务器 |  |  |  | 是 | 两者 | 否 |
| 创建表 |  |  |  | 是 | 两者 | 否 |
| 创建触发器 |  |  |  | 是 | 两者 | 否 |
| 创建用户定义函数 |  |  |  | 是 | 两者 | 否 |
| 创建用户 |  |  |  | 是 | 两者 | 否 |
| 创建视图 |  |  |  | 是 | 两者 | 否 |
| 释放 SQL |  |  |  | 是 | 两者 | 否 |
| 删除 |  |  |  | 是 | 两者 | 否 |
| 多删除 |  |  |  | 是 | 两者 | 否 |
| 执行 |  |  |  | 是 | 两者 | 否 |
| 删除数据库 |  |  |  | 是 | 两者 | 否 |
| 删除事件 |  |  |  | 是 | 两者 | 否 |
| 删除函数 |  |  |  | 是 | 两者 | 否 |
| 删除索引 |  |  |  | 是 | 两者 | 否 |
| 删除存储过程 |  |  |  | 是 | 两者 | 否 |
| 删除资源组 |  |  |  | 是 | 全局 | 否 |
| 删除角色 |  |  |  | 是 | 全局 | 否 |
| 删除服务器 |  |  |  | 是 | 两者 | 否 |
| 删除表 |  |  |  | 是 | 两者 | 否 |
| 删除触发器 |  |  |  | 是 | 两者 | 否 |
| 删除用户 |  |  |  | 是 | 两者 | 否 |
| 删除视图 |  |  |  | 是 | 两者 | 否 |
| 空查询 |  |  |  | 是 | 两者 | 否 |
| 执行 SQL |  |  |  | 是 | 两者 | 否 |
| 其他解释 |  |  |  | 是 | 两者 | 否 |
| 刷新 |  |  |  | 是 | 两者 | 否 |
| 获取诊断信息 |  |  |  | 是 | 两者 | 否 |
| 授权 |  |  |  | 是 | 两者 | 否 |
| Com_grant_roles |  |  |  | 是 | 全局 | 否 |
| Com_group_replication_start |  |  |  | 是 | 全局 | 否 |
| Com_group_replication_stop |  |  |  | 是 | 全局 | 否 |
| Com_ha_close |  |  |  | 是 | 两者 | 否 |
| Com_ha_open |  |  |  | 是 | 两者 | 否 |
| Com_ha_read |  |  |  | 是 | 两者 | 否 |
| Com_help |  |  |  | 是 | 两者 | 否 |
| Com_insert |  |  |  | 是 | 两者 | 否 |
| Com_insert_select |  |  |  | 是 | 两者 | 否 |
| Com_install_component |  |  |  | 是 | 全局 | 否 |
| Com_install_plugin |  |  |  | 是 | 两者 | 否 |
| Com_kill |  |  |  | 是 | 两者 | 否 |
| Com_load |  |  |  | 是 | 两者 | 否 |
| Com_lock_tables |  |  |  | 是 | 两者 | 否 |
| Com_optimize |  |  |  | 是 | 两者 | 否 |
| Com_preload_keys |  |  |  | 是 | 两者 | 否 |
| Com_prepare_sql |  |  |  | 是 | 两者 | 否 |
| Com_purge |  |  |  | 是 | 两者 | 否 |
| Com_purge_before_date |  |  |  | 是 | 两者 | 否 |
| Com_release_savepoint |  |  |  | 是 | 两者 | 否 |
| Com_rename_table |  |  |  | 是 | 两者 | 否 |
| Com_rename_user |  |  |  | 是 | 两者 | 否 |
| Com_repair |  |  |  | 是 | 两者 | 否 |
| Com_replace |  |  |  | 是 | 两者 | 否 |
| Com_replace_select |  |  |  | 是 | 两者 | 否 |
| Com_replica_start |  |  |  | 是 | 两者 | 否 |
| Com_replica_stop |  |  |  | 是 | 两者 | 否 |
| Com_reset |  |  |  | 是 | 两者 | 否 |
| Com_resignal |  |  |  | 是 | 两者 | 否 |
| Com_restart |  |  |  | 是 | 两者 | 否 |
| Com_revoke |  |  |  | 是 | 两者 | 否 |
| 撤销所有 |  |  |  | 是 | 两者 | 否 |
| 撤销角色 |  |  |  | 是 | 全局 | 否 |
| 回滚 |  |  |  | 是 | 两者 | 否 |
| 回滚到保存点 |  |  |  | 是 | 两者 | 否 |
| 保存点 |  |  |  | 是 | 两者 | 否 |
| 选择 |  |  |  | 是 | 两者 | 否 |
| 设置选项 |  |  |  | 是 | 两者 | 否 |
| 设置资源组 |  |  |  | 是 | 全局 | 否 |
| 设置角色 |  |  |  | 是 | 全局 | 否 |
| 显示作者 |  |  |  | 是 | 两者 | 否 |
| 显示二进制日志事件 |  |  |  | 是 | 两者 | 否 |
| 显示二进制日志 |  |  |  | 是 | 两者 | 否 |
| 显示字符集 |  |  |  | 是 | 两者 | 否 |
| 显示校对 |  |  |  | 是 | 两者 | 否 |
| 显示贡献者 |  |  |  | 是 | 两者 | 否 |
| 显示创建数据库 |  |  |  | 是 | 两者 | 否 |
| 显示创建事件 |  |  |  | 是 | 两者 | 否 |
| 显示创建函数 |  |  |  | 是 | 两者 | 否 |
| 显示创建过程 |  |  |  | 是 | 两者 | 否 |
| 显示创建表 |  |  |  | 是 | 两者 | 否 |
| 显示创建触发器 |  |  |  | 是 | 两者 | 否 |
| 显示创建用户 |  |  |  | 是 | 两者 | 否 |
| 显示数据库 |  |  |  | 是 | 两者 | 否 |
| 显示引擎日志 |  |  |  | 是 | 两者 | 否 |
| 显示引擎互斥 |  |  |  | 是 | 两者 | 否 |
| 显示引擎状态 |  |  |  | 是 | 两者 | 否 |
| 显示错误 |  |  |  | 是 | 两者 | 否 |
| 显示事件 |  |  |  | 是 | 两者 | 否 |
| 显示字段 |  |  |  | 是 | 两者 | 否 |
| 显示函数代码 |  |  |  | 是 | 两者 | 否 |
| 显示函数状态 |  |  |  | 是 | 两者 | 否 |
| 显示授权 |  |  |  | 是 | 两者 | 否 |
| 显示键 |  |  |  | 是 | 两者 | 否 |
| 显示主服务器状态 |  |  |  | 是 | 两者 | 否 |
| 显示 NDB 状态 |  |  |  | 是 | 两者 | 否 |
| 显示打开表 |  |  |  | 是 | 两者 | 否 |
| 显示插件 |  |  |  | 是 | 两者 | 否 |
| 显示权限 |  |  |  | 是 | 两者 | 否 |
| 显示存储过程代码 |  |  |  | 是 | 两者 | 否 |
| 显示存储过程状态 |  |  |  | 是 | 两者 | 否 |
| 显示进程列表 |  |  |  | 是 | 两者 | 否 |
| 显示配置文件 |  |  |  | 是 | 两者 | 否 |
| 显示配置文件 |  |  |  | 是 | 两者 | 否 |
| 显示中继日志事件 |  |  |  | 是 | 两者 | 否 |
| 显示复制状态 |  |  |  | 是 | 两者 | 否 |
| 显示复制 |  |  |  | 是 | 两者 | 否 |
| 显示从服务器主机 |  |  |  | 是 | 两者 | 否 |
| 显示从服务器状态 |  |  |  | 是 | 两者 | 否 |
| 显示状态 |  |  |  | 是 | 两者 | 否 |
| 显示存储引擎 |  |  |  | 是 | 两者 | 否 |
| 显示表状态 |  |  |  | 是 | 两者 | 否 |
| 显示表信息 |  |  |  | 是 | 两者 | 否 |
| 显示触发器 |  |  |  | 是 | 两者 | 否 |
| 显示变量 |  |  |  | 是 | 两者 | 否 |
| 显示警告 |  |  |  | 是 | 两者 | 否 |
| 关闭连接 |  |  |  | 是 | 两者 | 否 |
| 发送信号 |  |  |  | 是 | 两者 | 否 |
| 启动从服务器 |  |  |  | 是 | 两者 | 否 |
| 停止从服务器 |  |  |  | 是 | 两者 | 否 |
| 关闭语句 |  |  |  | 是 | 两者 | 否 |
| Com_stmt_execute |  |  |  | 是 | 两者 | 否 |
| Com_stmt_fetch |  |  |  | 是 | 两者 | 否 |
| Com_stmt_prepare |  |  |  | 是 | 两者 | 否 |
| Com_stmt_reprepare |  |  |  | 是 | 两者 | 否 |
| Com_stmt_reset |  |  |  | 是 | 两者 | 否 |
| Com_stmt_send_long_data |  |  |  | 是 | 两者 | 否 |
| Com_truncate |  |  |  | 是 | 两者 | 否 |
| Com_uninstall_component |  |  |  | 是 | 全局 | 否 |
| Com_uninstall_plugin |  |  |  | 是 | 两者 | 否 |
| Com_unlock_tables |  |  |  | 是 | 两者 | 否 |
| Com_update |  |  |  | 是 | 两者 | 否 |
| Com_update_multi |  |  |  | 是 | 两者 | 否 |
| Com_xa_commit |  |  |  | 是 | 两者 | 否 |
| Com_xa_end |  |  |  | 是 | 两者 | 否 |
| Com_xa_prepare |  |  |  | 是 | 两者 | 否 |
| Com_xa_recover |  |  |  | 是 | 两者 | 否 |
| Com_xa_rollback |  |  |  | 是 | 两者 | 否 |
| Com_xa_start |  |  |  | 是 | 两者 | 否 |
| 完成类型 | 是 | 是 | 是 |  | 两者 | 是 |
| component_scheduler.enabled | 是 | 是 | 是 |  | 全局 | 是 |
| 压缩 |  |  |  | 是 | 会话 | 否 |
| 压缩算法 |  |  |  | 是 | 全局 | 否 |
| 压缩级别 |  |  |  | 是 | 全局 | 否 |
| 并发插入 | 是 | 是 | 是 |  | 全局 | 是 |
| 连接超时 | 是 | 是 | 是 |  | 全局 | 是 |
| 连接控制延迟生成 |  |  |  | 是 | 全局 | 否 |
| 连接控制失败连接阈值 | 是 | 是 | 是 |  | 全局 | 是 |
| 连接控制最大连接延迟 | 是 | 是 | 是 |  | 全局 | 是 |
| 连接控制最小连接延迟 | 是 | 是 | 是 |  | 全局 | 是 |
| 连接错误接受 |  |  |  | 是 | 全局 | 否 |
| 连接错误内部 |  |  |  | 是 | 全局 | 否 |
| 连接错误最大连接数 |  |  |  | 是 | 全局 | 否 |
| 连接错误对等地址 |  |  |  | 是 | 全局 | 否 |
| 连接错误选择 |  |  |  | 是 | 全局 | 否 |
| 连接错误 TCP 包装 |  |  |  | 是 | 全局 | 否 |
| 连接内存块大小 | 是 | 是 | 是 |  | 两者 | 是 |
| 连接内存限制 | 是 | 是 | 是 |  | 两者 | 是 |
| 连接数 |  |  |  | 是 | 全局 | 否 |
| 控制台 | 是 | 是 |  |  |  |  |
| 核心文件 | 是 | 是 |  |  |  |  |
| 核心文件 |  |  | 是 |  | 全局 | 否 |
| 创建管理员监听线程 | 是 | 是 | 是 |  | 全局 | 否 |
| 创建临时磁盘表 |  |  |  | 是 | 两者 | 否 |
| 创建临时文件 |  |  |  | 是 | 全局 | 否 |
| 创建临时表 |  |  |  | 是 | 两者 | 否 |
| CTE 最大递归深度 | 是 | 是 | 是 |  | 两者 | 是 |
| 当前 TLS CA |  |  |  | 是 | 全局 | 否 |
| 当前 TLS CA 路径 |  |  |  | 是 | 全局 | 否 |
| 当前 TLS 证书 |  |  |  | 是 | 全局 | 否 |
| 当前 TLS 密码 |  |  |  | 是 | 全局 | 否 |
| 当前 TLS 密码套件 |  |  |  | 是 | 全局 | 否 |
| 当前 TLS CRL |  |  |  | 是 | 全局 | 否 |
| Current_tls_crlpath |  |  |  | 是 | 全局 | 否 |
| Current_tls_key |  |  |  | 是 | 全局 | 否 |
| Current_tls_version |  |  |  | 是 | 全局 | 否 |
| daemon_memcached_enable_binlog | 是 | 是 | 是 |  | 全局 | 否 |
| daemon_memcached_engine_lib_name | 是 | 是 | 是 |  | 全局 | 否 |
| daemon_memcached_engine_lib_path | 是 | 是 | 是 |  | 全局 | 否 |
| daemon_memcached_option | 是 | 是 | 是 |  | 全局 | 否 |
| daemon_memcached_r_batch_size | 是 | 是 | 是 |  | 全局 | 否 |
| daemon_memcached_w_batch_size | 是 | 是 | 是 |  | 全局 | 否 |
| daemonize | 是 | 是 |  |  |  |  |
| datadir | 是 | 是 | 是 |  | 全局 | 否 |
| ddl-rewriter | 是 | 是 |  |  |  |  |
| debug | 是 | 是 | 是 |  | 两者 | 是 |
| debug_sync |  |  | 是 |  | 会话 | 是 |
| debug-sync-timeout | 是 | 是 |  |  |  |  |
| default_authentication_plugin | 是 | 是 | 是 |  | 全局 | 否 |
| default_collation_for_utf8mb4 |  |  | 是 |  | 两者 | 是 |
| default_password_lifetime | 是 | 是 | 是 |  | 全局 | 是 |
| default_storage_engine | 是 | 是 | 是 |  | 两者 | 是 |
| default_table_encryption | 是 | 是 | 是 |  | 两者 | 是 |
| default-time-zone | 是 | 是 |  |  |  |  |
| default_tmp_storage_engine | 是 | 是 | 是 |  | 两者 | 是 |
| default_week_format | 是 | 是 | 是 |  | 两者 | 是 |
| defaults-extra-file | 是 |  |  |  |  |  |
| defaults-file | 是 |  |  |  |  |  |
| defaults-group-suffix | 是 |  |  |  |  |  |
| delay_key_write | 是 | 是 | 是 |  | 全局 | 是 |
| Delayed_errors |  |  |  | 是 | 全局 | 否 |
| delayed_insert_limit | 是 | 是 | 是 |  | 全局 | 是 |
| Delayed_insert_threads |  |  |  | 是 | 全局 | 否 |
| delayed_insert_timeout | 是 | 是 | 是 |  | 全局 | 是 |
| delayed_queue_size | 是 | 是 | 是 |  | 全局 | 是 |
| Delayed_writes |  |  |  | 是 | 全局 | 否 |
| disabled_storage_engines | 是 | 是 | 是 |  | 全局 | 否 |
| disconnect_on_expired_password | 是 | 是 | 是 |  | 全局 | 否 |
| disconnect-slave-event-count | 是 | 是 |  |  |  |  |
| div_precision_increment | 是 | 是 | 是 |  | 两者 | 是 |
| dragnet.log_error_filter_rules | 是 | 是 | 是 |  | 全局 | 是 |
| dragnet.Status |  |  |  | 是 | 全局 | 否 |
| early-plugin-load | 是 | 是 |  |  |  |  |
| end_markers_in_json | 是 | 是 | 是 |  | 两者 | 是 |
| enforce_gtid_consistency | 是 | 是 | 是 |  | 全局 | 是 |
| enterprise_encryption.maximum_rsa_key_size | 是 | 是 | 是 |  | 全局 | 是 |
| enterprise_encryption.rsa_support_legacy_padding | 是 | 是 | 是 |  | 全局 | 是 |
| eq_range_index_dive_limit | 是 | 是 | 是 |  | 两者 | 是 |
| error_count |  |  | 是 |  | 会话 | 否 |
| Error_log_buffered_bytes |  |  |  | 是 | 全局 | 否 |
| Error_log_buffered_events |  |  |  | 是 | 全局 | 否 |
| Error_log_expired_events |  |  |  | 是 | 全局 | 否 |
| Error_log_latest_write |  |  |  | 是 | 全局 | 否 |
| 事件调度器 | 是 | 是 | 是 |  | 全局 | 是 |
| 退出信息 | 是 | 是 |  |  |  |  |
| 日志过期天数 | 是 | 是 | 是 |  | 全局 | 是 |
| 解释格式 | 是 | 是 | 是 |  | 两者 | 是 |
| 时间戳显式默认值 | 是 | 是 | 是 |  | 两者 | 是 |
| 外部锁定 | 是 | 是 |  |  |  |  |
| - *变量*: 跳过外部锁定 |  |  |  |  |  |  |
| 外部用户 |  |  | 是 |  | 会话 | 否 |
| 联合 | 是 | 是 |  |  |  |  |
| 防火墙访问拒绝 |  |  |  | 是 | 全局 | 否 |
| 防火墙访问授权 |  |  |  | 是 | 全局 | 否 |
| 防火墙缓存条目 |  |  |  | 是 | 全局 | 否 |
| 刷新 | 是 | 是 | 是 |  | 全局 | 是 |
| 刷新命令 |  |  |  | 是 | 全局 | 否 |
| 刷新时间 | 是 | 是 | 是 |  | 全局 | 是 |
| 外键检查 |  |  | 是 |  | 两者 | 是 |
| 全文布尔语法 | 是 | 是 | 是 |  | 全局 | 是 |
| 全文最大词长度 | 是 | 是 | 是 |  | 全局 | 否 |
| 全文最小词长度 | 是 | 是 | 是 |  | 全局 | 否 |
| 全文查询扩展限制 | 是 | 是 | 是 |  | 全局 | 否 |
| 全文停用词文件 | 是 | 是 | 是 |  | 全局 | 否 |
| gdb | 是 | 是 |  |  |  |  |
| 一般日志 | 是 | 是 | 是 |  | 全局 | 是 |
| 一般日志文件 | 是 | 是 | 是 |  | 全局 | 是 |
| 生成的随机密码长度 | 是 | 是 | 是 |  | 两者 | 是 |
| 全局连接内存 |  |  |  | 是 | 全局 | 否 |
| global_connection_memory_limit | 是 | 是 | 是 |  | 全局 | 是 |
| global_connection_memory_tracking | 是 | 是 | 是 |  | 两者 | 是 |
| group_concat_max_len | 是 | 是 | 是 |  | 两者 | 是 |
| group_replication_advertise_recovery_endpoints | 是 | 是 | 是 |  | 全局 | 是 |
| group_replication_allow_local_lower_version_join | 是 | 是 | 是 |  | 全局 | 是 |
| group_replication_auto_increment_increment | 是 | 是 | 是 |  | 全局 | 是 |
| group_replication_autorejoin_tries | 是 | 是 | 是 |  | 全局 | 是 |
| group_replication_bootstrap_group | 是 | 是 | 是 |  | 全局 | 是 |
| group_replication_clone_threshold | 是 | 是 | 是 |  | 全局 | 是 |
| group_replication_communication_debug_options | 是 | 是 | 是 |  | 全局 | 是 |
| group_replication_communication_max_message_size | 是 | 是 | 是 |  | 全局 | 是 |
| group_replication_communication_stack |  |  | 是 |  | 全局 | 否 |
| group_replication_components_stop_timeout | 是 | 是 | 是 |  | 全局 | 是 |
| group_replication_compression_threshold | 是 | 是 | 是 |  | 全局 | 是 |
| group_replication_consistency | 是 | 是 | 是 |  | 两者 | 是 |
| group_replication_enforce_update_everywhere_checks | 是 | 是 | 是 |  | 全局 | 是 |
| group_replication_exit_state_action | 是 | 是 | 是 |  | 全局 | 是 |
| group_replication_flow_control_applier_threshold | 是 | 是 | 是 |  | 全局 | 是 |
| group_replication_flow_control_certifier_threshold | 是 | 是 | 是 |  | 全局 | 是 |
| group_replication_flow_control_hold_percent | 是 | 是 | 是 |  | 全局 | 是 |
| group_replication_flow_control_max_quota | 是 | 是 | 是 |  | 全局 | 是 |
| group_replication_flow_control_member_quota_percent | 是 | 是 | 是 |  | 全局 | 是 |
| group_replication_flow_control_min_quota | 是 | 是 | 是 |  | 全局 | 是 |
| group_replication_flow_control_min_recovery_quota | 是 | 是 | 是 |  | 全局 | 是 |
| group_replication_flow_control_mode | 是 | 是 | 是 |  | 全局 | 是 |
| group_replication_flow_control_period | 是 | 是 | 是 |  | 全局 | 是 |
| group_replication_flow_control_release_percent | 是 | 是 | 是 |  | 全局 | 是 |
| group_replication_force_members | 是 | 是 | 是 |  | 全局 | 是 |
| group_replication_group_name | 是 | 是 | 是 |  | 全局 | 是 |
| group_replication_group_seeds | 是 | 是 | 是 |  | 全局 | 是 |
| group_replication_gtid_assignment_block_size | 是 | 是 | 是 |  | 全局 | 是 |
| group_replication_ip_allowlist | 是 | 是 | 是 |  | 全局 | 是 |
| group_replication_ip_whitelist | 是 | 是 | 是 |  | 全局 | 是 |
| group_replication_local_address | 是 | 是 | 是 |  | 全局 | 是 |
| group_replication_member_expel_timeout | 是 | 是 | 是 |  | 全局 | 是 |
| group_replication_member_weight | 是 | 是 | 是 |  | 全局 | 是 |
| group_replication_message_cache_size | 是 | 是 | 是 |  | 全局 | 是 |
| group_replication_paxos_single_leader | 是 | 是 | 是 |  | 全局 | 是 |
| group_replication_poll_spin_loops | 是 | 是 | 是 |  | 全局 | 是 |
| group_replication_primary_member |  |  |  | 是 | 全局 | 否 |
| group_replication_recovery_complete_at | 是 | 是 | 是 |  | 全局 | 是 |
| group_replication_recovery_compression_algorithms | 是 | 是 | 是 |  | 全局 | 是 |
| group_replication_recovery_get_public_key | 是 | 是 | 是 |  | 全局 | 是 |
| group_replication_recovery_public_key_path | 是 | 是 | 是 |  | 全局 | 是 |
| group_replication_recovery_reconnect_interval | 是 | 是 | 是 |  | 全局 | 是 |
| group_replication_recovery_retry_count | 是 | 是 | 是 |  | 全局 | 是 |
| group_replication_recovery_ssl_ca | 是 | 是 | 是 |  | 全局 | 是 |
| group_replication_recovery_ssl_capath | 是 | 是 | 是 |  | 全局 | 是 |
| group_replication_recovery_ssl_cert | 是 | 是 | 是 |  | 全局 | 是 |
| group_replication_recovery_ssl_cipher | 是 | 是 | 是 |  | 全局 | 是 |
| group_replication_recovery_ssl_crl | 是 | 是 | 是 |  | 全局 | 是 |
| group_replication_recovery_ssl_crlpath | 是 | 是 | 是 |  | 全局 | 是 |
| group_replication_recovery_ssl_key | 是 | 是 | 是 |  | 全局 | 是 |
| group_replication_recovery_ssl_verify_server_cert | 是 | 是 | 是 |  | 全局 | 是 |
| group_replication_recovery_tls_ciphersuites | 是 | 是 | 是 |  | 全局 | 是 |
| group_replication_recovery_tls_version | 是 | 是 | 是 |  | 全局 | 是 |
| group_replication_recovery_use_ssl | 是 | 是 | 是 |  | 全局 | 是 |
| group_replication_recovery_zstd_compression_level | 是 | 是 | 是 |  | 全局 | 是 |
| group_replication_single_primary_mode | 是 | 是 | 是 |  | 全局 | 是 |
| group_replication_ssl_mode | 是 | 是 | 是 |  | 全局 | 是 |
| group_replication_start_on_boot | 是 | 是 | 是 |  | 全局 | 是 |
| group_replication_tls_source | 是 | 是 | 是 |  | 全局 | 是 |
| group_replication_transaction_size_limit | 是 | 是 | 是 |  | 全局 | 是 |
| group_replication_unreachable_majority_timeout | 是 | 是 | 是 |  | 全局 | 是 |
| group_replication_view_change_uuid | 是 | 是 | 是 |  | 全局 | 是 |
| gtid_executed |  |  | 是 |  | 全局 | 否 |
| gtid_executed_compression_period | 是 | 是 | 是 |  | 全局 | 是 |
| gtid_mode | 是 | 是 | 是 |  | 全局 | 是 |
| gtid_next |  |  | 是 |  | 会话 | 是 |
| gtid_owned |  |  | 是 |  | 两者 | 否 |
| gtid_purged |  |  | 是 |  | 全局 | 是 |
| Handler_commit |  |  |  | 是 | 两者 | 否 |
| Handler_delete |  |  |  | 是 | 两者 | 否 |
| Handler_discover |  |  |  | 是 | 两者 | 否 |
| Handler_external_lock |  |  |  | 是 | 两者 | 否 |
| Handler_mrr_init |  |  |  | 是 | 两者 | 否 |
| Handler_prepare |  |  |  | 是 | 两者 | 否 |
| Handler_read_first |  |  |  | 是 | 两者 | 否 |
| Handler_read_key |  |  |  | 是 | 两者 | 否 |
| Handler_read_last |  |  |  | 是 | 两者 | 否 |
| Handler_read_next |  |  |  | 是 | 两者 | 否 |
| Handler_read_prev |  |  |  | 是 | 两者 | 否 |
| Handler_read_rnd |  |  |  | 是 | 两者 | 否 |
| Handler_read_rnd_next |  |  |  | 是 | 两者 | 否 |
| Handler_rollback |  |  |  | 是 | 两者 | 否 |
| Handler_savepoint |  |  |  | 是 | 两者 | 否 |
| Handler_savepoint_rollback |  |  |  | 是 | 两者 | 否 |
| Handler_update |  |  |  | 是 | 两者 | 否 |
| Handler_write |  |  |  | 是 | 两者 | 否 |
| have_compress |  |  | 是 |  | 全局 | 否 |
| have_dynamic_loading |  |  | 是 |  | 全局 | 否 |
| have_geometry |  |  | 是 |  | 全局 | 否 |
| have_openssl |  |  | 是 |  | 全局 | 否 |
| have_profiling |  |  | 是 |  | 全局 | 否 |
| have_query_cache |  |  | 是 |  | 全局 | 否 |
| have_rtree_keys |  |  | 是 |  | 全局 | 否 |
| have_ssl |  |  | 是 |  | 全局 | 否 |
| have_statement_timeout |  |  | 是 |  | 全局 | 否 |
| have_symlink |  |  | 是 |  | 全局 | 否 |
| help | 是 | 是 |  |  |  |  |
| histogram_generation_max_mem_size | 是 | 是 | 是 |  | 两者 | 是 |
| host_cache_size | 是 | 是 | 是 |  | 全局 | 是 |
| hostname |  |  | 是 |  | 全局 | 否 |
| identity |  |  | 是 |  | 会话 | 是 |
| immediate_server_version |  |  | 是 |  | 会话 | 是 |
| information_schema_stats_expiry | 是 | 是 | 是 |  | 两者 | 是 |
| init_connect | 是 | 是 | 是 |  | 全局 | 是 |
| init_file | 是 | 是 | 是 |  | 全局 | 否 |
| init_replica | 是 | 是 | 是 |  | 全局 | 是 |
| init_slave | 是 | 是 | 是 |  | 全局 | 是 |
| initialize | 是 | 是 |  |  |  |  |
| initialize-insecure | 是 | 是 |  |  |  |  |
| innodb | 是 | 是 |  |  |  |  |
| innodb_adaptive_flushing | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_adaptive_flushing_lwm | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_adaptive_hash_index | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_adaptive_hash_index_parts | 是 | 是 | 是 |  | 全局 | 否 |
| innodb_adaptive_max_sleep_delay | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_api_bk_commit_interval | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_api_disable_rowlock | 是 | 是 | 是 |  | 全局 | 否 |
| innodb_api_enable_binlog | 是 | 是 | 是 |  | 全局 | 否 |
| innodb_api_enable_mdl | 是 | 是 | 是 |  | 全局 | 否 |
| innodb_api_trx_level | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_autoextend_increment | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_autoinc_lock_mode | 是 | 是 | 是 |  | 全局 | 否 |
| innodb_background_drop_list_empty | 是 | 是 | 是 |  | 全局 | 是 |
| Innodb_buffer_pool_bytes_data |  |  |  | 是 | 全局 | 否 |
| Innodb_buffer_pool_bytes_dirty |  |  |  | 是 | 全局 | 否 |
| innodb_buffer_pool_chunk_size | 是 | 是 | 是 |  | 全局 | 否 |
| innodb_buffer_pool_debug | 是 | 是 | 是 |  | 全局 | 否 |
| innodb_buffer_pool_dump_at_shutdown | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_buffer_pool_dump_now | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_buffer_pool_dump_pct | 是 | 是 | 是 |  | 全局 | 是 |
| Innodb_buffer_pool_dump_status |  |  |  | 是 | 全局 | 否 |
| innodb_buffer_pool_filename | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_buffer_pool_in_core_file | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_buffer_pool_instances | 是 | 是 | 是 |  | 全局 | 否 |
| innodb_buffer_pool_load_abort | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_buffer_pool_load_at_startup | 是 | 是 | 是 |  | 全局 | 否 |
| innodb_buffer_pool_load_now | 是 | 是 | 是 |  | 全局 | 是 |
| Innodb_buffer_pool_load_status |  |  |  | 是 | 全局 | 否 |
| Innodb_buffer_pool_pages_data |  |  |  | 是 | 全局 | 否 |
| Innodb_buffer_pool_pages_dirty |  |  |  | 是 | 全局 | 否 |
| Innodb_buffer_pool_pages_flushed |  |  |  | 是 | 全局 | 否 |
| Innodb_buffer_pool_pages_free |  |  |  | 是 | 全局 | 否 |
| Innodb_buffer_pool_pages_latched |  |  |  | 是 | 全局 | 否 |
| Innodb_buffer_pool_pages_misc |  |  |  | 是 | 全局 | 否 |
| Innodb_buffer_pool_pages_total |  |  |  | 是 | 全局 | 否 |
| Innodb_buffer_pool_read_ahead |  |  |  | 是 | 全局 | 否 |
| Innodb_buffer_pool_read_ahead_evicted |  |  |  | 是 | 全局 | 否 |
| Innodb_buffer_pool_read_ahead_rnd |  |  |  | 是 | 全局 | 否 |
| Innodb_buffer_pool_read_requests |  |  |  | 是 | 全局 | 否 |
| Innodb_buffer_pool_reads |  |  |  | 是 | 全局 | 否 |
| Innodb_buffer_pool_resize_status |  |  |  | 是 | 全局 | 否 |
| Innodb_buffer_pool_resize_status_code |  |  |  | 是 | 全局 | 否 |
| Innodb_buffer_pool_resize_status_progress |  |  |  | 是 | 全局 | 否 |
| innodb_buffer_pool_size | 是 | 是 | 是 |  | 全局 | 是 |
| Innodb_buffer_pool_wait_free |  |  |  | 是 | 全局 | 否 |
| Innodb_buffer_pool_write_requests |  |  |  | 是 | 全局 | 否 |
| innodb_change_buffer_max_size | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_change_buffering | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_change_buffering_debug | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_checkpoint_disabled | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_checksum_algorithm | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_cmp_per_index_enabled | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_commit_concurrency | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_compress_debug | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_compression_failure_threshold_pct | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_compression_level | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_compression_pad_pct_max | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_concurrency_tickets | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_data_file_path | 是 | 是 | 是 |  | 全局 | 否 |
| Innodb_data_fsyncs |  |  |  | 是 | 全局 | 否 |
| innodb_data_home_dir | 是 | 是 | 是 |  | 全局 | 否 |
| Innodb_data_pending_fsyncs |  |  |  | 是 | 全局 | 否 |
| Innodb_data_pending_reads |  |  |  | 是 | 全局 | 否 |
| Innodb_data_pending_writes |  |  |  | 是 | 全局 | 否 |
| Innodb_data_read |  |  |  | 是 | 全局 | 否 |
| Innodb_data_reads |  |  |  | 是 | 全局 | 否 |
| Innodb_data_writes |  |  |  | 是 | 全局 | 否 |
| Innodb_data_written |  |  |  | 是 | 全局 | 否 |
| Innodb_dblwr_pages_written |  |  |  | 是 | 全局 | 否 |
| Innodb_dblwr_writes |  |  |  | 是 | 全局 | 否 |
| innodb_ddl_buffer_size | 是 | 是 | 是 |  | 两者 | 是 |
| innodb_ddl_log_crash_reset_debug | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_ddl_threads | 是 | 是 | 是 |  | 两者 | 是 |
| innodb_deadlock_detect | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_dedicated_server | 是 | 是 | 是 |  | 全局 | 否 |
| innodb_default_row_format | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_directories | 是 | 是 | 是 |  | 全局 | 否 |
| innodb_disable_sort_file_cache | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_doublewrite | 是 | 是 | 是 |  | 全局 | 不定 |
| innodb_doublewrite_batch_size | 是 | 是 | 是 |  | 全局 | 否 |
| innodb_doublewrite_dir | 是 | 是 | 是 |  | 全局 | 否 |
| innodb_doublewrite_files | 是 | 是 | 是 |  | 全局 | 否 |
| innodb_doublewrite_pages | 是 | 是 | 是 |  | 全局 | 否 |
| innodb_extend_and_initialize | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_fast_shutdown | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_fil_make_page_dirty_debug | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_file_per_table | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_fill_factor | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_flush_log_at_timeout | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_flush_log_at_trx_commit | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_flush_method | 是 | 是 | 是 |  | 全局 | 否 |
| innodb_flush_neighbors | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_flush_sync | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_flushing_avg_loops | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_force_load_corrupted | 是 | 是 | 是 |  | 全局 | 否 |
| innodb_force_recovery | 是 | 是 | 是 |  | 全局 | 否 |
| innodb_fsync_threshold | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_ft_aux_table |  |  | 是 |  | 全局 | 是 |
| innodb_ft_cache_size | 是 | 是 | 是 |  | 全局 | 否 |
| innodb_ft_enable_diag_print | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_ft_enable_stopword | 是 | 是 | 是 |  | 两者 | 是 |
| innodb_ft_max_token_size | 是 | 是 | 是 |  | 全局 | 否 |
| innodb_ft_min_token_size | 是 | 是 | 是 |  | 全局 | 否 |
| innodb_ft_num_word_optimize | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_ft_result_cache_limit | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_ft_server_stopword_table | 是 | 是 | 是 |  | ��局 | 是 |
| innodb_ft_sort_pll_degree | 是 | 是 | 是 |  | 全局 | 否 |
| innodb_ft_total_cache_size | 是 | 是 | 是 |  | 全局 | 否 |
| innodb_ft_user_stopword_table | 是 | 是 | 是 |  | 两者 | 是 |
| Innodb_have_atomic_builtins |  |  |  | 是 | 全局 | 否 |
| innodb_idle_flush_pct | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_io_capacity | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_io_capacity_max | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_limit_optimistic_insert_debug | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_lock_wait_timeout | 是 | 是 | 是 |  | 两者 | 是 |
| innodb_log_buffer_size | 是 | 是 | 是 |  | 全局 | 不定 |
| innodb_log_checkpoint_fuzzy_now | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_log_checkpoint_now | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_log_checksums | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_log_compressed_pages | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_log_file_size | 是 | 是 | 是 |  | 全局 | 否 |
| innodb_log_files_in_group | 是 | 是 | 是 |  | 全局 | 否 |
| innodb_log_group_home_dir | 是 | 是 | 是 |  | 全局 | 否 |
| innodb_log_spin_cpu_abs_lwm | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_log_spin_cpu_pct_hwm | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_log_wait_for_flush_spin_hwm | 是 | 是 | 是 |  | 全局 | 是 |
| Innodb_log_waits |  |  |  | 是 | 全局 | 否 |
| innodb_log_write_ahead_size | 是 | 是 | 是 |  | 全局 | 是 |
| Innodb_log_write_requests |  |  |  | 是 | 全局 | 否 |
| innodb_log_writer_threads | 是 | 是 | 是 |  | 全局 | 是 |
| Innodb_log_writes |  |  |  | 是 | 全局 | 否 |
| innodb_lru_scan_depth | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_max_dirty_pages_pct | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_max_dirty_pages_pct_lwm | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_max_purge_lag | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_max_purge_lag_delay | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_max_undo_log_size | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_merge_threshold_set_all_debug | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_monitor_disable | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_monitor_enable | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_monitor_reset | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_monitor_reset_all | 是 | 是 | 是 |  | 全局 | 是 |
| Innodb_num_open_files |  |  |  | 是 | 全局 | 否 |
| innodb_numa_interleave | 是 | 是 | 是 |  | 全局 | 否 |
| innodb_old_blocks_pct | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_old_blocks_time | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_online_alter_log_max_size | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_open_files | 是 | 是 | 是 |  | 全局 | 不定 |
| innodb_optimize_fulltext_only | 是 | 是 | 是 |  | 全局 | 是 |
| Innodb_os_log_fsyncs |  |  |  | 是 | 全局 | 否 |
| Innodb_os_log_pending_fsyncs |  |  |  | 是 | 全局 | 否 |
| Innodb_os_log_pending_writes |  |  |  | 是 | 全局 | 否 |
| Innodb_os_log_written |  |  |  | 是 | 全局 | 否 |
| innodb_page_cleaners | 是 | 是 | 是 |  | 全局 | 否 |
| Innodb_page_size |  |  |  | 是 | 全局 | 否 |
| innodb_page_size | 是 | 是 | 是 |  | 全局 | 否 |
| Innodb_pages_created |  |  |  | 是 | 全局 | 否 |
| Innodb_pages_read |  |  |  | 是 | 全局 | 否 |
| Innodb_pages_written |  |  |  | 是 | 全局 | 否 |
| innodb_parallel_read_threads | 是 | 是 | 是 |  | 会话 | 是 |
| innodb_print_all_deadlocks | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_print_ddl_logs | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_purge_batch_size | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_purge_rseg_truncate_frequency | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_purge_threads | 是 | 是 | 是 |  | 全局 | 否 |
| innodb_random_read_ahead | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_read_ahead_threshold | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_read_io_threads | 是 | 是 | 是 |  | 全局 | 否 |
| innodb_read_only | 是 | 是 | 是 |  | 全局 | 否 |
| innodb_redo_log_archive_dirs | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_redo_log_capacity | 是 | 是 | 是 |  | 全局 | 是 |
| Innodb_redo_log_capacity_resized |  |  |  | 是 | 全局 | 否 |
| Innodb_redo_log_checkpoint_lsn |  |  |  | 是 | 全局 | 否 |
| Innodb_redo_log_current_lsn |  |  |  | 是 | 全局 | 否 |
| Innodb_redo_log_enabled |  |  |  | 是 | 全局 | 否 |
| innodb_redo_log_encrypt | 是 | 是 | 是 |  | 全局 | 是 |
| Innodb_redo_log_flushed_to_disk_lsn |  |  |  | 是 | 全局 | 否 |
| Innodb_redo_log_logical_size |  |  |  | 是 | 全局 | 否 |
| Innodb_redo_log_physical_size |  |  |  | 是 | 全局 | 否 |
| Innodb_redo_log_read_only |  |  |  | 是 | 全局 | 否 |
| Innodb_redo_log_resize_status |  |  |  | 是 | 全局 | 否 |
| Innodb_redo_log_uuid |  |  |  | 是 | 全局 | 否 |
| innodb_replication_delay | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_rollback_on_timeout | 是 | 是 | 是 |  | 全局 | 否 |
| innodb_rollback_segments | 是 | 是 | 是 |  | 全局 | 是 |
| Innodb_row_lock_current_waits |  |  |  | 是 | 全局 | 否 |
| Innodb_row_lock_time |  |  |  | 是 | 全局 | 否 |
| Innodb_row_lock_time_avg |  |  |  | 是 | 全局 | 否 |
| Innodb_row_lock_time_max |  |  |  | 是 | 全局 | 否 |
| Innodb_row_lock_waits |  |  |  | 是 | 全局 | 否 |
| Innodb_rows_deleted |  |  |  | 是 | 全局 | 否 |
| Innodb_rows_inserted |  |  |  | 是 | 全局 | 否 |
| Innodb_rows_read |  |  |  | 是 | 全局 | 否 |
| Innodb_rows_updated |  |  |  | 是 | 全局 | 否 |
| innodb_saved_page_number_debug | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_segment_reserve_factor | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_sort_buffer_size | 是 | 是 | 是 |  | 全局 | 否 |
| innodb_spin_wait_delay | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_spin_wait_pause_multiplier | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_stats_auto_recalc | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_stats_include_delete_marked | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_stats_method | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_stats_on_metadata | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_stats_persistent | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_stats_persistent_sample_pages | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_stats_transient_sample_pages | 是 | 是 | 是 |  | 全局 | 是 |
| innodb-status-file | 是 | 是 |  |  |  |  |
| innodb_status_output | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_status_output_locks | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_strict_mode | 是 | 是 | 是 |  | 两者 | 是 |
| innodb_sync_array_size | 是 | 是 | 是 |  | 全局 | 否 |
| innodb_sync_debug | 是 | 是 | 是 |  | 全局 | 否 |
| innodb_sync_spin_loops | 是 | 是 | 是 |  | 全局 | 是 |
| Innodb_system_rows_deleted |  |  |  | 是 | 全局 | 否 |
| Innodb_system_rows_inserted |  |  |  | 是 | 全局 | 否 |
| Innodb_system_rows_read |  |  |  | 是 | 全局 | 否 |
| innodb_table_locks | 是 | 是 | 是 |  | 两者 | 是 |
| innodb_temp_data_file_path | 是 | 是 | 是 |  | 全局 | 否 |
| innodb_temp_tablespaces_dir | 是 | 是 | 是 |  | 全局 | 否 |
| innodb_thread_concurrency | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_thread_sleep_delay | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_tmpdir | 是 | 是 | 是 |  | 两者 | 是 |
| Innodb_truncated_status_writes |  |  |  | 是 | 全局 | 否 |
| innodb_trx_purge_view_update_only_debug | 是 | 是 | ��� |  | 全局 | 是 |
| innodb_trx_rseg_n_slots_debug | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_undo_directory | 是 | 是 | 是 |  | 全局 | 否 |
| innodb_undo_log_encrypt | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_undo_log_truncate | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_undo_tablespaces | 是 | 是 | 是 |  | 全局 | 不定 |
| Innodb_undo_tablespaces_active |  |  |  | 是 | 全局 | 否 |
| Innodb_undo_tablespaces_explicit |  |  |  | 是 | 全局 | 否 |
| Innodb_undo_tablespaces_implicit |  |  |  | 是 | 全局 | 否 |
| Innodb_undo_tablespaces_total |  |  |  | 是 | 全局 | 否 |
| innodb_use_fdatasync | 是 | 是 | 是 |  | 全局 | 是 |
| innodb_use_native_aio | 是 | 是 | 是 |  | 全局 | 否 |
| innodb_validate_tablespace_paths | 是 | 是 | 是 |  | 全局 | 否 |
| innodb_version |  |  | 是 |  | 全局 | 否 |
| innodb_write_io_threads | 是 | 是 | 是 |  | 全局 | 否 |
| insert_id |  |  | 是 |  | 会话 | 是 |
| install | 是 |  |  |  |  |  |
| install-manual | 是 |  |  |  |  |  |
| interactive_timeout | 是 | 是 | 是 |  | 两者 | 是 |
| internal_tmp_disk_storage_engine | 是 | 是 | 是 |  | 全局 | 是 |
| internal_tmp_mem_storage_engine | 是 | 是 | 是 |  | 两者 | 是 |
| join_buffer_size | 是 | 是 | 是 |  | 两者 | 是 |
| keep_files_on_create | 是 | 是 | 是 |  | 两者 | 是 |
| Key_blocks_not_flushed |  |  |  | 是 | 全局 | 否 |
| Key_blocks_unused |  |  |  | 是 | 全局 | 否 |
| Key_blocks_used |  |  |  | 是 | 全局 | 否 |
| key_buffer_size | 是 | 是 | 是 |  | 全局 | 是 |
| key_cache_age_threshold | 是 | 是 | 是 |  | 全局 | 是 |
| key_cache_block_size | 是 | 是 | 是 |  | 全局 | 是 |
| key_cache_division_limit | 是 | 是 | 是 |  | 全局 | 是 |
| Key_read_requests |  |  |  | 是 | 全局 | 否 |
| Key_reads |  |  |  | 是 | 全局 | 否 |
| Key_write_requests |  |  |  | 是 | 全局 | 否 |
| Key_writes |  |  |  | 是 | 全局 | 否 |
| keyring_aws_cmk_id | 是 | 是 | 是 |  | 全局 | 是 |
| keyring_aws_conf_file | 是 | 是 | 是 |  | 全局 | 否 |
| keyring_aws_data_file | 是 | 是 | 是 |  | 全局 | 否 |
| keyring_aws_region | 是 | 是 | 是 |  | 全局 | 是 |
| keyring_encrypted_file_data | 是 | 是 | 是 |  | 全局 | 是 |
| keyring_encrypted_file_password | 是 | 是 | 是 |  | 全局 | 是 |
| keyring_file_data | 是 | 是 | 是 |  | 全局 | 是 |
| keyring_hashicorp_auth_path | 是 | 是 | 是 |  | 全局 | 是 |
| keyring_hashicorp_ca_path | 是 | 是 | 是 |  | 全局 | 是 |
| keyring_hashicorp_caching | 是 | 是 | 是 |  | 全局 | 是 |
| keyring_hashicorp_commit_auth_path |  |  | 是 |  | 全局 | 否 |
| keyring_hashicorp_commit_ca_path |  |  | 是 |  | 全局 | 否 |
| keyring_hashicorp_commit_caching |  |  | 是 |  | 全局 | 否 |
| keyring_hashicorp_commit_role_id |  |  | 是 |  | 全局 | 否 |
| keyring_hashicorp_commit_server_url |  |  | 是 |  | 全局 | 否 |
| keyring_hashicorp_commit_store_path |  |  | 是 |  | 全局 | 否 |
| keyring_hashicorp_role_id | 是 | 是 | 是 |  | 全局 | 是 |
| keyring_hashicorp_secret_id | 是 | 是 | 是 |  | 全局 | 是 |
| keyring_hashicorp_server_url | 是 | 是 | 是 |  | 全局 | 是 |
| keyring_hashicorp_store_path | 是 | 是 | 是 |  | 全局 | 是 |
| keyring-migration-destination | 是 | 是 |  |  |  |  |
| keyring-migration-host | 是 | 是 |  |  |  |  |
| keyring-migration-password | 是 | 是 |  |  |  |  |
| keyring-migration-port | 是 | 是 |  |  |  |  |
| keyring-migration-socket | 是 | 是 |  |  |  |  |
| keyring-migration-source | 是 | 是 |  |  |  |  |
| keyring-migration-to-component | 是 | 是 |  |  |  |  |
| keyring-migration-user | 是 | 是 |  |  |  |  |
| keyring_oci_ca_certificate | 是 | 是 | 是 |  | 全局 | 否 |
| keyring_oci_compartment | 是 | 是 | 是 |  | 全局 | 否 |
| keyring_oci_encryption_endpoint | 是 | 是 | 是 |  | 全局 | 否 |
| keyring_oci_key_file | 是 | 是 | 是 |  | 全局 | 否 |
| keyring_oci_key_fingerprint | 是 | 是 | 是 |  | 全局 | 否 |
| keyring_oci_management_endpoint | 是 | 是 | 是 |  | 全局 | 否 |
| keyring_oci_master_key | 是 | 是 | 是 |  | 全局 | 否 |
| keyring_oci_secrets_endpoint | 是 | 是 | 是 |  | 全局 | 否 |
| keyring_oci_tenancy | 是 | 是 | 是 |  | 全局 | 否 |
| keyring_oci_user | 是 | 是 | 是 |  | 全局 | 否 |
| keyring_oci_vaults_endpoint | 是 | 是 | 是 |  | 全局 | 否 |
| keyring_oci_virtual_vault | 是 | 是 | 是 |  | 全局 | 否 |
| keyring_okv_conf_dir | 是 | 是 | 是 |  | 全局 | 是 |
| 密钥环操作 |  |  | 是 |  | 全局 | 是 |
| 大文件支持 |  |  | 是 |  | 全局 | 否 |
| 大页大小 |  |  | 是 |  | 全局 | 否 |
| 大页 | 是 | 是 | 是 |  | 全局 | 否 |
| last_insert_id |  |  | 是 |  | 会话 | 是 |
| 最后查询成本 |  |  |  | 是 | 会话 | 否 |
| 最后查询部分计划 |  |  |  | 是 | 会话 | 否 |
| lc_messages | 是 | 是 | 是 |  | 双方 | 是 |
| lc_messages_dir | 是 | 是 | 是 |  | 全局 | 否 |
| lc_time_names | 是 | 是 | 是 |  | 双方 | 是 |
| 许可证 |  |  | 是 |  | 全局 | 否 |
| 本地文件 | 是 | 是 | 是 |  | 全局 | 是 |
| 本地服务 | 是 |  |  |  |  |  |
| 锁定顺序 | 是 | 是 | 是 |  | 全局 | 否 |
| 锁定顺序调试循环 | 是 | 是 | 是 |  | 全局 | 否 |
| 锁定顺序调试丢失弧 | 是 | 是 | 是 |  | 全局 | 否 |
| 锁定顺序调试丢失密钥 | 是 | 是 | 是 |  | 全局 | 否 |
| 锁定顺序调试丢失解锁 | 是 | 是 | 是 |  | 全局 | 否 |
| 锁定顺序依赖关系 | 是 | 是 | 是 |  | 全局 | 否 |
| 锁定顺序额外依赖关系 | 是 | 是 | 是 |  | 全局 | 否 |
| 锁定顺序输出目录 | 是 | 是 | 是 |  | 全局 | 否 |
| 锁定顺序打印文本 | 是 | 是 | 是 |  | 全局 | 否 |
| 锁定顺序跟踪循环 | 是 | 是 | 是 |  | 全局 | 否 |
| 锁定顺序跟踪丢失弧 | 是 | 是 | 是 |  | 全局 | 否 |
| 锁定顺序跟踪丢失密钥 | 是 | 是 | 是 |  | 全局 | 否 |
| 锁定顺序跟踪丢失解锁 | 是 | 是 | 是 |  | 全局 | 否 |
| lock_wait_timeout | 是 | 是 | 是 |  | 两者 | 是 |
| Locked_connects |  |  |  | 是 | 全局 | 否 |
| locked_in_memory |  |  | 是 |  | 全局 | 否 |
| log-bin | 是 | 是 |  |  |  |  |
| log_bin |  |  | 是 |  | 全局 | 否 |
| log_bin_basename |  |  | 是 |  | 全局 | 否 |
| log_bin_index | 是 | 是 | 是 |  | 全局 | 否 |
| log_bin_trust_function_creators | 是 | 是 | 是 |  | 全局 | 是 |
| log_bin_use_v1_row_events | 是 | 是 | 是 |  | 全局 | 是 |
| log_error | 是 | 是 | 是 |  | 全局 | 否 |
| log_error_services | 是 | 是 | 是 |  | 全局 | 是 |
| log_error_suppression_list | 是 | 是 | 是 |  | 全局 | 是 |
| log_error_verbosity | 是 | 是 | 是 |  | 全局 | 是 |
| log-isam | 是 | 是 |  |  |  |  |
| log_output | 是 | 是 | 是 |  | 全局 | 是 |
| log_queries_not_using_indexes | 是 | 是 | 是 |  | 全局 | 是 |
| log_raw | 是 | 是 | 是 |  | 全局 | 是 |
| log_replica_updates | 是 | 是 | 是 |  | 全局 | 否 |
| log-short-format | 是 | 是 |  |  |  |  |
| log_slave_updates | 是 | 是 | 是 |  | 全局 | 否 |
| log_slow_admin_statements | 是 | 是 | 是 |  | 全局 | 是 |
| log_slow_extra | 是 | 是 | 是 |  | 全局 | 是 |
| log_slow_replica_statements | 是 | 是 | 是 |  | 全局 | 是 |
| log_slow_slave_statements | 是 | 是 | 是 |  | 全局 | 是 |
| log_statements_unsafe_for_binlog | 是 | 是 | 是 |  | 全局 | 是 |
| log_syslog | 是 | 是 | 是 |  | 全局 | 是 |
| log_syslog_facility | 是 | 是 | 是 |  | 全局 | 是 |
| log_syslog_include_pid | 是 | 是 | 是 |  | 全局 | 是 |
| log_syslog_tag | 是 | 是 | 是 |  | 全局 | 是 |
| log-tc | 是 | 是 |  |  |  |  |
| log-tc-size | 是 | 是 |  |  |  |  |
| log_throttle_queries_not_using_indexes | 是 | 是 | 是 |  | 全局 | 是 |
| log_timestamps | 是 | 是 | 是 |  | 全局 | 是 |
| long_query_time | 是 | 是 | 是 |  | 两者 | 是 |
| low_priority_updates | 是 | 是 | 是 |  | 两者 | 是 |
| lower_case_file_system |  |  | 是 |  | 全局 | 否 |
| lower_case_table_names | 是 | 是 | 是 |  | 全局 | 否 |
| mandatory_roles | 是 | 是 | 是 |  | 全局 | 是 |
| master-info-file | 是 | 是 |  |  |  |  |
| master_info_repository | 是 | 是 | 是 |  | 全局 | 是 |
| master-retry-count | 是 | 是 |  |  |  |  |
| master_verify_checksum | 是 | 是 | 是 |  | 全局 | 是 |
| max_allowed_packet | 是 | 是 | 是 |  | 两者 | 是 |
| max_binlog_cache_size | 是 | 是 | 是 |  | 全局 | 是 |
| max-binlog-dump-events | 是 | 是 |  |  |  |  |
| max_binlog_size | 是 | 是 | 是 |  | 全局 | 是 |
| max_binlog_stmt_cache_size | 是 | 是 | 是 |  | 全局 | 是 |
| max_connect_errors | 是 | 是 | 是 |  | 全局 | 是 |
| max_connections | 是 | 是 | 是 |  | 全局 | 是 |
| max_delayed_threads | 是 | 是 | 是 |  | 两者 | 是 |
| max_digest_length | 是 | 是 | 是 |  | 全局 | 否 |
| max_error_count | 是 | 是 | 是 |  | 两者 | 是 |
| max_execution_time | 是 | 是 | 是 |  | 两者 | 是 |
| 超过最大执行时间 |  |  |  | 是 | 两者 | 否 |
| 设置的最大执行时间 |  |  |  | 是 | 两者 | 否 |
| 设置失败的最大执行时间 |  |  |  | 是 | 两者 | 否 |
| max_heap_table_size | 是 | 是 | 是 |  | 两者 | 是 |
| max_insert_delayed_threads |  |  | 是 |  | 两者 | 是 |
| max_join_size | 是 | 是 | 是 |  | 两者 | 是 |
| max_length_for_sort_data | 是 | 是 | 是 |  | 两者 | 是 |
| max_points_in_geometry | 是 | 是 | 是 |  | 两者 | 是 |
| max_prepared_stmt_count | 是 | 是 | 是 |  | 全局 | 是 |
| max_relay_log_size | 是 | 是 | 是 |  | 全局 | 是 |
| max_seeks_for_key | 是 | 是 | 是 |  | 两者 | 是 |
| max_sort_length | 是 | 是 | 是 |  | 两者 | 是 |
| max_sp_recursion_depth | 是 | 是 | 是 |  | 两者 | 是 |
| 最大使用连接数 |  |  |  | 是 | 全局 | 否 |
| 最大使用连接时间 |  |  |  | 是 | 全局 | 否 |
| max_user_connections | 是 | 是 | 是 |  | 两者 | 是 |
| max_write_lock_count | 是 | 是 | 是 |  | 全局 | 是 |
| mecab_charset |  |  |  | 是 | 全局 | 否 |
| mecab_rc_file | 是 | 是 | 是 |  | 全局 | 否 |
| memlock | 是 | 是 |  |  |  |  |
| - *变量*: 内存中锁定 |  |  |  |  |  |  |
| metadata_locks_cache_size | 是 | 是 | 是 |  | 全局 | 否 |
| metadata_locks_hash_instances | 是 | 是 | 是 |  | 全局 | 否 |
| min_examined_row_limit | 是 | 是 | 是 |  | 两者 | 是 |
| myisam-block-size | 是 | 是 |  |  |  |  |
| myisam_data_pointer_size | 是 | 是 | 是 |  | 全局 | 是 |
| myisam_max_sort_file_size | 是 | 是 | 是 |  | 全局 | 是 |
| myisam_mmap_size | 是 | 是 | 是 |  | 全局 | 否 |
| myisam_recover_options | 是 | 是 | 是 |  | 全局 | 否 |
| myisam_repair_threads | 是 | 是 | 是 |  | 两者 | 是 |
| myisam_sort_buffer_size | 是 | 是 | 是 |  | 两者 | 是 |
| myisam_stats_method | 是 | 是 | 是 |  | 两者 | 是 |
| myisam_use_mmap | 是 | 是 | 是 |  | 全局 | 是 |
| mysql_firewall_mode | 是 | 是 | 是 |  | 全局 | 是 |
| mysql_firewall_trace | 是 | 是 | 是 |  | 全局 | 是 |
| mysql_native_password_proxy_users | 是 | 是 | 是 |  | 全局 | 是 |
| mysqlx | 是 | 是 |  |  |  |  |
| Mysqlx_aborted_clients |  |  |  | 是 | 全局 | 否 |
| Mysqlx_address |  |  |  | 是 | 全局 | 否 |
| mysqlx_bind_address | 是 | 是 | 是 |  | 全局 | 否 |
| Mysqlx_bytes_received |  |  |  | 是 | 两者 | 否 |
| Mysqlx_bytes_received_compressed_payload |  |  |  | 是 | 两者 | 否 |
| Mysqlx_bytes_received_uncompressed_frame |  |  |  | 是 | 两者 | 否 |
| Mysqlx_bytes_sent |  |  |  | 是 | 两者 | 否 |
| Mysqlx_bytes_sent_compressed_payload |  |  |  | 是 | 两者 | 否 |
| Mysqlx_bytes_sent_uncompressed_frame |  |  |  | 是 | 两者 | 否 |
| Mysqlx_compression_algorithm |  |  |  | 是 | 会话 | 否 |
| mysqlx_compression_algorithms | 是 | 是 | 是 |  | 全局 | 是 |
| Mysqlx_compression_level |  |  |  | 是 | 会话 | 否 |
| mysqlx_connect_timeout | 是 | 是 | 是 |  | 全局 | 是 |
| Mysqlx_connection_accept_errors |  |  |  | 是 | 两者 | 否 |
| Mysqlx_connection_errors |  |  |  | 是 | 两者 | 否 |
| Mysqlx_connections_accepted |  |  |  | 是 | 全局 | 否 |
| Mysqlx_connections_closed |  |  |  | 是 | 全局 | 否 |
| Mysqlx_connections_rejected |  |  |  | 是 | 全局 | 否 |
| Mysqlx_crud_create_view |  |  |  | 是 | 两者 | 否 |
| Mysqlx_crud_delete |  |  |  | 是 | 两者 | 否 |
| Mysqlx_crud_drop_view |  |  |  | 是 | 两者 | 否 |
| Mysqlx_crud_find |  |  |  | 是 | 两者 | 否 |
| Mysqlx_crud_insert |  |  |  | 是 | 两者 | 否 |
| Mysqlx_crud_modify_view |  |  |  | 是 | 两者 | 否 |
| Mysqlx_crud_update |  |  |  | 是 | 两者 | 否 |
| Mysqlx_cursor_close |  |  |  | 是 | 两者 | 否 |
| Mysqlx_cursor_fetch |  |  |  | 是 | 两者 | 否 |
| Mysqlx_cursor_open |  |  |  | 是 | 两者 | 否 |
| mysqlx_deflate_default_compression_level | 是 | 是 | 是 |  | 全局 | 是 |
| mysqlx_deflate_max_client_compression_level | 是 | 是 | 是 |  | 全局 | 是 |
| mysqlx_document_id_unique_prefix | 是 | 是 | 是 |  | 全局 | 是 |
| mysqlx_enable_hello_notice | 是 | 是 | 是 |  | 全局 | 是 |
| Mysqlx_errors_sent |  |  |  | 是 | 两者 | 否 |
| Mysqlx_errors_unknown_message_type |  |  |  | 是 | 两者 | 否 |
| Mysqlx_expect_close |  |  |  | 是 | 两者 | 否 |
| Mysqlx_expect_open |  |  |  | 是 | 两者 | 否 |
| mysqlx_idle_worker_thread_timeout | 是 | 是 | 是 |  | 全局 | 是 |
| Mysqlx_init_error |  |  |  | 是 | 两者 | 否 |
| mysqlx_interactive_timeout | 是 | 是 | 是 |  | 全局 | 是 |
| mysqlx_lz4_default_compression_level | 是 | 是 | 是 |  | 全局 | 是 |
| mysqlx_lz4_max_client_compression_level | 是 | 是 | 是 |  | 全局 | 是 |
| mysqlx_max_allowed_packet | 是 | 是 | 是 |  | 全局 | 是 |
| mysqlx_max_connections | 是 | 是 | 是 |  | 全局 | 是 |
| Mysqlx_messages_sent |  |  |  | 是 | 两者 | 否 |
| mysqlx_min_worker_threads | 是 | 是 | 是 |  | 全局 | 是 |
| Mysqlx_notice_global_sent |  |  |  | 是 | 两者 | 否 |
| Mysqlx_notice_other_sent |  |  |  | 是 | 两者 | 否 |
| Mysqlx_notice_warning_sent |  |  |  | 是 | 两者 | 否 |
| Mysqlx_notified_by_group_replication |  |  |  | 是 | 两者 | 否 |
| Mysqlx_port |  |  |  | 是 | 全局 | 否 |
| mysqlx_port | 是 | 是 | 是 |  | 全局 | 否 |
| mysqlx_port_open_timeout | 是 | 是 | 是 |  | 全局 | 否 |
| Mysqlx_prep_deallocate |  |  |  | 是 | 两者 | 否 |
| Mysqlx_prep_execute |  |  |  | 是 | 两者 | 否 |
| Mysqlx_prep_prepare |  |  |  | 是 | 两者 | 否 |
| mysqlx_read_timeout | 是 | 是 | 是 |  | 会话 | 是 |
| Mysqlx_rows_sent |  |  |  | 是 | 两者 | 否 |
| Mysqlx_sessions |  |  |  | 是 | 全局 | 否 |
| Mysqlx_sessions_accepted |  |  |  | 是 | 全局 | 否 |
| Mysqlx_sessions_closed |  |  |  | 是 | 全局 | 否 |
| Mysqlx_sessions_fatal_error |  |  |  | 是 | 全局 | 否 |
| Mysqlx_sessions_killed |  |  |  | 是 | 全局 | 否 |
| Mysqlx_sessions_rejected |  |  |  | 是 | 全局 | 否 |
| Mysqlx_socket |  |  |  | 是 | 全局 | 否 |
| mysqlx_socket | 是 | 是 | 是 |  | 全局 | 否 |
| Mysqlx_ssl_accept_renegotiates |  |  |  | 是 | 全局 | 否 |
| Mysqlx_ssl_accepts |  |  |  | 是 | 全局 | 否 |
| Mysqlx_ssl_active |  |  |  | 是 | 两者 | 否 |
| mysqlx_ssl_ca | 是 | 是 | 是 |  | 全局 | 否 |
| mysqlx_ssl_capath | 是 | 是 | 是 |  | 全局 | 否 |
| mysqlx_ssl_cert | 是 | 是 | 是 |  | 全局 | 否 |
| Mysqlx_ssl_cipher |  |  |  | 是 | 两者 | 否 |
| mysqlx_ssl_cipher | 是 | 是 | 是 |  | 全局 | 否 |
| Mysqlx_ssl_cipher_list |  |  |  | 是 | 两者 | 否 |
| mysqlx_ssl_crl | 是 | 是 | 是 |  | 全局 | 否 |
| mysqlx_ssl_crlpath | 是 | 是 | 是 |  | 全局 | 否 |
| Mysqlx_ssl_ctx_verify_depth |  |  |  | 是 | 两者 | 否 |
| Mysqlx_ssl_ctx_verify_mode |  |  |  | 是 | 两者 | 否 |
| Mysqlx_ssl_finished_accepts |  |  |  | 是 | 全局 | 否 |
| mysqlx_ssl_key | 是 | 是 | 是 |  | 全局 | 否 |
| Mysqlx_ssl_server_not_after |  |  |  | 是 | 全局 | 否 |
| Mysqlx_ssl_server_not_before |  |  |  | 是 | 全局 | 否 |
| Mysqlx_ssl_verify_depth |  |  |  | 是 | 全局 | 否 |
| Mysqlx_ssl_verify_mode |  |  |  | 是 | 全局 | 否 |
| Mysqlx_ssl_version |  |  |  | 是 | 两者 | 否 |
| Mysqlx_stmt_create_collection |  |  |  | 是 | 两者 | 否 |
| Mysqlx_stmt_create_collection_index |  |  |  | 是 | 两者 | 否 |
| Mysqlx_stmt_disable_notices |  |  |  | 是 | 两者 | 否 |
| Mysqlx_stmt_drop_collection |  |  |  | 是 | 两者 | 否 |
| Mysqlx_stmt_drop_collection_index |  |  |  | 是 | 两者 | 否 |
| Mysqlx_stmt_enable_notices |  |  |  | 是 | 两者 | 否 |
| Mysqlx_stmt_ensure_collection |  |  |  | 是 | 两者 | 否 |
| Mysqlx_stmt_execute_mysqlx |  |  |  | 是 | 两者 | 否 |
| Mysqlx_stmt_execute_sql |  |  |  | 是 | 两者 | 否 |
| Mysqlx_stmt_execute_xplugin |  |  |  | 是 | 两者 | 否 |
| Mysqlx_stmt_get_collection_options |  |  |  | 是 | 两者 | 否 |
| Mysqlx_stmt_kill_client |  |  |  | 是 | 两者 | 否 |
| Mysqlx_stmt_list_clients |  |  |  | 是 | 两者 | 否 |
| Mysqlx_stmt_list_notices |  |  |  | 是 | 两者 | 否 |
| Mysqlx_stmt_list_objects |  |  |  | 是 | 两者 | 否 |
| Mysqlx_stmt_modify_collection_options |  |  |  | 是 | 两者 | 否 |
| Mysqlx_stmt_ping |  |  |  | 是 | 两者 | 否 |
| mysqlx_wait_timeout | 是 | 是 | 是 |  | 会话 | 是 |
| Mysqlx_worker_threads |  |  |  | 是 | 全局 | 否 |
| Mysqlx_worker_threads_active |  |  |  | 是 | 全局 | 否 |
| mysqlx_write_timeout | 是 | 是 | 是 |  | 会话 | 是 |
| mysqlx_zstd_default_compression_level | 是 | 是 | 是 |  | 全局 | 是 |
| mysqlx_zstd_max_client_compression_level | 是 | 是 | 是 |  | 全局 | 是 |
| named_pipe | 是 | 是 | 是 |  | 全局 | 否 |
| named_pipe_full_access_group | 是 | 是 | 是 |  | 全局 | 否 |
| ndb_allow_copying_alter_table | 是 | 是 | 是 |  | 双方 | 是 |
| Ndb_api_adaptive_send_deferred_count |  |  |  | 是 | 全局 | 否 |
| Ndb_api_adaptive_send_deferred_count_replica |  |  |  | 是 | 全局 | 否 |
| Ndb_api_adaptive_send_deferred_count_session |  |  |  | 是 | 全局 | 否 |
| Ndb_api_adaptive_send_deferred_count_slave |  |  |  | 是 | 全局 | 否 |
| Ndb_api_adaptive_send_forced_count |  |  |  | 是 | 全局 | 否 |
| Ndb_api_adaptive_send_forced_count_replica |  |  |  | 是 | 全局 | 否 |
| Ndb_api_adaptive_send_forced_count_session |  |  |  | 是 | 全局 | 否 |
| Ndb_api_adaptive_send_forced_count_slave |  |  |  | 是 | 全局 | 否 |
| Ndb_api_adaptive_send_unforced_count |  |  |  | 是 | 全局 | 否 |
| Ndb_api_adaptive_send_unforced_count_replica |  |  |  | 是 | 全局 | 否 |
| Ndb_api_adaptive_send_unforced_count_session |  |  |  | 是 | 全局 | 否 |
| Ndb_api_adaptive_send_unforced_count_slave |  |  |  | 是 | 全局 | 否 |
| Ndb_api_bytes_received_count |  |  |  | 是 | 全局 | 否 |
| Ndb_api_bytes_received_count_replica |  |  |  | 是 | 全局 | 否 |
| Ndb_api_bytes_received_count_session |  |  |  | 是 | 会话 | 否 |
| Ndb_api_bytes_received_count_slave |  |  |  | 是 | 全局 | 否 |
| Ndb_api_bytes_sent_count |  |  |  | 是 | 全局 | 否 |
| Ndb_api_bytes_sent_count_replica |  |  |  | 是 | 全局 | 否 |
| Ndb_api_bytes_sent_count_session |  |  |  | 是 | 会话 | 否 |
| Ndb_api_bytes_sent_count_slave |  |  |  | 是 | 全局 | 否 |
| Ndb_api_event_bytes_count |  |  |  | 是 | 全局 | 否 |
| Ndb_api_event_bytes_count_injector |  |  |  | 是 | 全局 | 否 |
| Ndb_api_event_data_count |  |  |  | 是 | 全局 | 否 |
| Ndb_api_event_data_count_injector |  |  |  | 是 | 全局 | 否 |
| Ndb_api_event_nondata_count |  |  |  | 是 | 全局 | 否 |
| Ndb_api_event_nondata_count_injector |  |  |  | 是 | 全局 | 否 |
| Ndb_api_pk_op_count |  |  |  | 是 | 全局 | 否 |
| Ndb_api_pk_op_count_replica |  |  |  | 是 | 全局 | 否 |
| Ndb_api_pk_op_count_session |  |  |  | 是 | 会话 | 否 |
| Ndb_api_pk_op_count_slave |  |  |  | 是 | 全局 | 否 |
| Ndb_api_pruned_scan_count |  |  |  | 是 | 全局 | 否 |
| Ndb_api_pruned_scan_count_replica |  |  |  | 是 | 全局 | 否 |
| Ndb_api_pruned_scan_count_session |  |  |  | 是 | 会话 | 否 |
| Ndb_api_pruned_scan_count_slave |  |  |  | 是 | 全局 | 否 |
| Ndb_api_range_scan_count |  |  |  | 是 | 全局 | 否 |
| Ndb_api_range_scan_count_replica |  |  |  | 是 | 全局 | 否 |
| Ndb_api_range_scan_count_session |  |  |  | 是 | 会话 | 否 |
| Ndb_api_range_scan_count_slave |  |  |  | 是 | 全局 | 否 |
| Ndb_api_read_row_count |  |  |  | 是 | 全局 | 否 |
| Ndb_api_read_row_count_replica |  |  |  | 是 | 全局 | 否 |
| Ndb_api_read_row_count_session |  |  |  | 是 | 会话 | 否 |
| Ndb_api_read_row_count_slave |  |  |  | 是 | 全局 | 否 |
| Ndb_api_scan_batch_count |  |  |  | 是 | 全局 | 否 |
| Ndb_api_scan_batch_count_replica |  |  |  | 是 | 全局 | 否 |
| Ndb_api_scan_batch_count_session |  |  |  | 是 | 会话 | 否 |
| Ndb_api_scan_batch_count_slave |  |  |  | 是 | 全局 | 否 |
| Ndb_api_table_scan_count |  |  |  | 是 | 全局 | 否 |
| Ndb_api_table_scan_count_replica |  |  |  | 是 | 全局 | 否 |
| Ndb_api_table_scan_count_session |  |  |  | 是 | 会话 | 否 |
| Ndb_api_table_scan_count_slave |  |  |  | 是 | 全局 | 否 |
| Ndb_api_trans_abort_count |  |  |  | 是 | 全局 | 否 |
| Ndb_api_trans_abort_count_replica |  |  |  | 是 | 全局 | 否 |
| Ndb_api_trans_abort_count_session |  |  |  | 是 | 会话 | 否 |
| Ndb_api_trans_abort_count_slave |  |  |  | 是 | 全局 | 否 |
| Ndb_api_trans_close_count |  |  |  | 是 | 全局 | 否 |
| Ndb_api_trans_close_count_replica |  |  |  | 是 | 全局 | 否 |
| Ndb_api_trans_close_count_session |  |  |  | 是 | 会话 | 否 |
| Ndb_api_trans_close_count_slave |  |  |  | 是 | 全局 | 否 |
| Ndb_api_trans_commit_count |  |  |  | 是 | 全局 | 否 |
| Ndb_api_trans_commit_count_replica |  |  |  | 是 | 全局 | 否 |
| Ndb_api_trans_commit_count_session |  |  |  | 是 | 会话 | 否 |
| Ndb_api_trans_commit_count_slave |  |  |  | 是 | 全局 | 否 |
| Ndb_api_trans_local_read_row_count |  |  |  | 是 | 全局 | 否 |
| Ndb_api_trans_local_read_row_count_replica |  |  |  | 是 | 全局 | 否 |
| Ndb_api_trans_local_read_row_count_session |  |  |  | 是 | 会话 | 否 |
| Ndb_api_trans_local_read_row_count_slave |  |  |  | 是 | 全局 | 否 |
| Ndb_api_trans_start_count |  |  |  | 是 | 全局 | 否 |
| Ndb_api_trans_start_count_replica |  |  |  | 是 | 全局 | 否 |
| Ndb_api_trans_start_count_session |  |  |  | 是 | 会话 | 否 |
| Ndb_api_trans_start_count_slave |  |  |  | 是 | 全局 | 否 |
| Ndb_api_uk_op_count |  |  |  | 是 | 全局 | 否 |
| Ndb_api_uk_op_count_replica |  |  |  | 是 | 全局 | 否 |
| Ndb_api_uk_op_count_session |  |  |  | 是 | 会话 | 否 |
| Ndb_api_uk_op_count_slave |  |  |  | 是 | 全局 | 否 |
| Ndb_api_wait_exec_complete_count |  |  |  | 是 | 全局 | 否 |
| Ndb_api_wait_exec_complete_count_replica |  |  |  | 是 | 全局 | 否 |
| Ndb_api_wait_exec_complete_count_session |  |  |  | 是 | 会话 | 否 |
| Ndb_api_wait_exec_complete_count_slave |  |  |  | 是 | 全局 | 否 |
| Ndb_api_wait_meta_request_count |  |  |  | 是 | 全局 | 否 |
| Ndb_api_wait_meta_request_count_replica |  |  |  | 是 | 全局 | 否 |
| Ndb_api_wait_meta_request_count_session |  |  |  | 是 | 会话 | 否 |
| Ndb_api_wait_meta_request_count_slave |  |  |  | 是 | 全局 | 否 |
| Ndb_api_wait_nanos_count |  |  |  | 是 | 全局 | 否 |
| Ndb_api_wait_nanos_count_replica |  |  |  | 是 | 全局 | 否 |
| Ndb_api_wait_nanos_count_session |  |  |  | 是 | 会话 | 否 |
| Ndb_api_wait_nanos_count_slave |  |  |  | 是 | 全局 | 否 |
| Ndb_api_wait_scan_result_count |  |  |  | 是 | 全局 | 否 |
| Ndb_api_wait_scan_result_count_replica |  |  |  | 是 | 全局 | 否 |
| Ndb_api_wait_scan_result_count_session |  |  |  | 是 | 会话 | 否 |
| Ndb_api_wait_scan_result_count_slave |  |  |  | 是 | 全局 | 否 |
| ndb_applier_allow_skip_epoch | 是 | 是 | 是 |  | 全局 | 否 |
| ndb_autoincrement_prefetch_sz | 是 | 是 | 是 |  | 两者 | 是 |
| ndb_batch_size | 是 | 是 | 是 |  | 两者 | 是 |
| ndb_blob_read_batch_bytes | 是 | 是 | 是 |  | 两者 | 是 |
| ndb_blob_write_batch_bytes | 是 | 是 | 是 |  | 两者 | 是 |
| ndb_clear_apply_status | 是 |  | 是 |  | 全局 | 是 |
| ndb_cluster_connection_pool | 是 | 是 | 是 |  | 全局 | 否 |
| ndb_cluster_connection_pool_nodeids | 是 | 是 | 是 |  | 全局 | 否 |
| Ndb_cluster_node_id |  |  |  | 是 | 全局 | 否 |
| Ndb_config_from_host |  |  |  | 是 | 两者 | 否 |
| Ndb_config_from_port |  |  |  | 是 | 两者 | 否 |
| Ndb_config_generation |  |  |  | 是 | 全局 | 否 |
| Ndb_conflict_fn_epoch |  |  |  | 是 | 全局 | 否 |
| Ndb_conflict_fn_epoch_trans |  |  |  | 是 | 全局 | 否 |
| Ndb_conflict_fn_epoch2 |  |  |  | 是 | 全局 | 否 |
| Ndb_conflict_fn_epoch2_trans |  |  |  | 是 | 全局 | 否 |
| Ndb_conflict_fn_max |  |  |  | 是 | 全局 | 否 |
| Ndb_conflict_fn_max_del_win |  |  |  | 是 | 全局 | 否 |
| Ndb_conflict_fn_max_del_win_ins |  |  |  | 是 | 全局 | 否 |
| Ndb_conflict_fn_max_ins |  |  |  | 是 | 全局 | 否 |
| Ndb_conflict_fn_old |  |  |  | 是 | 全局 | 否 |
| Ndb_conflict_last_conflict_epoch |  |  |  | 是 | 全局 | 否 |
| Ndb_conflict_last_stable_epoch |  |  |  | 是 | 全局 | 否 |
| Ndb_conflict_reflected_op_discard_count |  |  |  | 是 | 全局 | 否 |
| Ndb_conflict_reflected_op_prepare_count |  |  |  | 是 | 全局 | 否 |
| Ndb_conflict_refresh_op_count |  |  |  | 是 | 全局 | 否 |
| ndb_conflict_role | 是 | 是 | 是 |  | 全局 | 是 |
| Ndb_conflict_trans_conflict_commit_count |  |  |  | 是 | 全局 | 否 |
| Ndb_conflict_trans_detect_iter_count |  |  |  | 是 | 全局 | 否 |
| Ndb_conflict_trans_reject_count |  |  |  | 是 | 全局 | 否 |
| Ndb_conflict_trans_row_conflict_count |  |  |  | 是 | 全局 | 否 |
| Ndb_conflict_trans_row_reject_count |  |  |  | 是 | 全局 | 否 |
| ndb-connectstring | 是 | 是 |  |  |  |  |
| ndb_data_node_neighbour | 是 | 是 | 是 |  | 全局 | 是 |
| ndb_dbg_check_shares | 是 | 是 | 是 |  | 两者 | 是 |
| ndb_default_column_format | 是 | 是 | 是 |  | 全局 | 是 |
| ndb_default_column_format | 是 | 是 | 是 |  | 全局 | 是 |
| ndb_deferred_constraints | 是 | 是 | 是 |  | 两者 | 是 |
| ndb_deferred_constraints | 是 | 是 | 是 |  | 两者 | 是 |
| ndb_distribution | 是 | 是 | 是 |  | 全局 | 是 |
| ndb_distribution | 是 | 是 | 是 |  | 全局 | 是 |
| Ndb_epoch_delete_delete_count |  |  |  | 是 | 全局 | 否 |
| ndb_eventbuffer_free_percent | 是 | 是 | 是 |  | 全局 | 是 |
| ndb_eventbuffer_max_alloc | 是 | 是 | 是 |  | 全局 | 是 |
| Ndb_execute_count |  |  |  | 是 | 全局 | 否 |
| ndb_extra_logging | 是 | 是 | 是 |  | 全局 | 是 |
| ndb_force_send | 是 | 是 | 是 |  | 两者 | 是 |
| ndb_fully_replicated | 是 | 是 | 是 |  | 两者 | 是 |
| ndb_index_stat_enable | 是 | 是 | 是 |  | 两者 | 是 |
| ndb_index_stat_option | 是 | 是 | 是 |  | 两者 | 是 |
| ndb_join_pushdown |  |  | 是 |  | 两者 | 是 |
| Ndb_last_commit_epoch_server |  |  |  | 是 | 全局 | 否 |
| Ndb_last_commit_epoch_session |  |  |  | 是 | 会话 | 否 |
| ndb_log_apply_status | 是 | 是 | 是 |  | 全局 | 否 |
| ndb_log_apply_status | 是 | 是 | 是 |  | 全局 | 否 |
| ndb_log_bin | 是 |  | 是 |  | 两者 | 否 |
| ndb_log_binlog_index | 是 |  | 是 |  | 全局 | 是 |
| ndb_log_empty_epochs | 是 | 是 | 是 |  | 全局 | 是 |
| ndb_log_empty_epochs | 是 | 是 | 是 |  | 全局 | 是 |
| ndb_log_empty_update | 是 | 是 | 是 |  | 全局 | 是 |
| ndb_log_empty_update | 是 | 是 | 是 |  | 全局 | 是 |
| ndb_log_exclusive_reads | 是 | 是 | 是 |  | 两者 | 是 |
| ndb_log_exclusive_reads | 是 | 是 | 是 |  | 两者 | 是 |
| ndb_log_fail_terminate | 是 | 是 | 是 |  | 全局 | 否 |
| ndb_log_orig | 是 | 是 | 是 |  | 全局 | 否 |
| ndb_log_orig | 是 | 是 | 是 |  | 全局 | 否 |
| ndb_log_transaction_compression | 是 | 是 | 是 |  | 全局 | 是 |
| ndb_log_transaction_compression_level_zstd | 是 | 是 | 是 |  | 全局 | 是 |
| ndb_log_transaction_dependency | 是 | 是 | 是 |  | 全局 | 否 |
| ndb_log_transaction_id | 是 | 是 | 是 |  | 全局 | 否 |
| ndb_log_transaction_id |  |  | 是 |  | 全局 | 否 |
| ndb_log_update_as_write | 是 | 是 | 是 |  | 全局 | 是 |
| ndb_log_update_minimal | 是 | 是 | 是 |  | 全局 | 是 |
| ndb_log_updated_only | 是 | 是 | 是 |  | 全局 | 是 |
| Ndb_metadata_blacklist_size |  |  |  | 是 | 全局 | 否 |
| ndb_metadata_check | 是 | 是 | 是 |  | 全局 | 是 |
| ndb_metadata_check_interval | 是 | 是 | 是 |  | 全局 | 是 |
| Ndb_metadata_detected_count |  |  |  | 是 | 全局 | 否 |
| Ndb_metadata_excluded_count |  |  |  | 是 | 全局 | 否 |
| ndb_metadata_sync |  |  | 是 |  | 全局 | 是 |
| Ndb_metadata_synced_count |  |  |  | 是 | 全局 | 否 |
| ndb-mgmd-host | 是 | 是 |  |  |  |  |
| ndb_nodeid | 是 | 是 |  | 是 | 全局 | 否 |
| Ndb_number_of_data_nodes |  |  |  | 是 | 全局 | 否 |
| ndb_optimization_delay | 是 | 是 | 是 |  | 全局 | 是 |
| ndb-optimized-node-selection | 是 |  |  |  |  |  |
| ndb_optimized_node_selection | 是 | 是 | 是 |  | 全局 | 否 |
| Ndb_pruned_scan_count |  |  |  | 是 | 全局 | 否 |
| Ndb_pushed_queries_defined |  |  |  | 是 | 全局 | 否 |
| Ndb_pushed_queries_dropped |  |  |  | 是 | 全局 | 否 |
| Ndb_pushed_queries_executed |  |  |  | 是 | 全局 | 否 |
| Ndb_pushed_reads |  |  |  | 是 | 全局 | 否 |
| ndb_read_backup | 是 | 是 | 是 |  | 全局 | 是 |
| ndb_recv_thread_activation_threshold | 是 | 是 | 是 |  | 全局 | 是 |
| ndb_recv_thread_cpu_mask | 是 | 是 | 是 |  | 全局 | 是 |
| ndb_replica_batch_size | 是 | 是 | 是 |  | 全局 | 是 |
| ndb_replica_blob_write_batch_bytes | 是 | 是 | 是 |  | 全局 | 是 |
| Ndb_replica_max_replicated_epoch |  |  | 是 |  | 全局 | 否 |
| ndb_report_thresh_binlog_epoch_slip | 是 | 是 | 是 |  | 全局 | 是 |
| ndb_report_thresh_binlog_mem_usage | 是 | 是 | 是 |  | 全局 | 是 |
| ndb_row_checksum |  |  | 是 |  | 两者 | 是 |
| Ndb_scan_count |  |  |  | 是 | 全局 | 否 |
| ndb_schema_dist_lock_wait_timeout | 是 | 是 | 是 |  | 全局 | 是 |
| ndb_schema_dist_timeout | 是 | 是 | 是 |  | 全局 | 否 |
| ndb_schema_dist_timeout | 是 | 是 | 是 |  | 全局 | 否 |
| ndb_schema_dist_upgrade_allowed | 是 | 是 | 是 |  | 全局 | 否 |
| ndb_show_foreign_key_mock_tables | 是 | 是 | 是 |  | 全局 | 是 |
| ndb_slave_conflict_role | 是 | 是 | 是 |  | 全局 | 是 |
| Ndb_slave_max_replicated_epoch |  |  |  | 是 | 全局 | 否 |
| Ndb_system_name |  |  | 是 |  | 全局 | 否 |
| ndb_table_no_logging |  |  | 是 |  | 会话 | 是 |
| ndb_table_temporary |  |  | 是 |  | 会话 | 是 |
| Ndb_trans_hint_count_session |  |  |  | 是 | 两者 | 否 |
| ndb-transid-mysql-connection-map | 是 |  |  |  |  |  |
| ndb_use_copying_alter_table |  |  | 是 |  | 两者 | 否 |
| ndb_use_exact_count |  |  | 是 |  | 两者 | 是 |
| ndb_use_transactions | 是 | 是 | 是 |  | 两者 | 是 |
| ndb_version |  |  | 是 |  | 全局 | 否 |
| ndb_version_string |  |  | 是 |  | 全局 | 否 |
| ndb_wait_connected | 是 | 是 | 是 |  | 全局 | 否 |
| ndb_wait_setup | 是 | 是 | 是 |  | 全局 | 否 |
| ndbcluster | 是 | 是 |  |  |  |  |
| ndbinfo | 是 |  |  |  |  |  |
| ndbinfo_database |  |  | 是 |  | 全局 | 否 |
| ndbinfo_max_bytes | 是 |  | 是 |  | 两者 | 是 |
| ndbinfo_max_rows | 是 |  | 是 |  | 两者 | 是 |
| ndbinfo_offline |  |  | 是 |  | 全局 | 是 |
| ndbinfo_show_hidden | 是 |  | 是 |  | 两者 | 是 |
| ndbinfo_table_prefix |  |  | 是 |  | 全局 | 否 |
| ndbinfo_version |  |  | 是 |  | 全局 | 否 |
| net_buffer_length | 是 | 是 | 是 |  | 两者 | 是 |
| net_read_timeout | 是 | 是 | 是 |  | 两者 | 是 |
| net_retry_count | 是 | 是 | 是 |  | 两者 | 是 |
| net_write_timeout | 是 | 是 | 是 |  | 两者 | 是 |
| new | 是 | 是 | 是 |  | 两者 | 是 |
| ngram_token_size | 是 | 是 | 是 |  | 全局 | 否 |
| no-dd-upgrade | 是 | 是 |  |  |  |  |
| no-defaults | 是 |  |  |  |  |  |
| no-monitor | 是 | 是 |  |  |  |  |
| 未刷新的延迟行 |  |  |  | 是 | 全局 | 否 |
| offline_mode | 是 | 是 | 是 |  | 全局 | 是 |
| old | 是 | 是 | 是 |  | 全局 | 否 |
| old_alter_table | 是 | 是 | 是 |  | 两者 | 是 |
| old-style-user-limits | 是 | 是 |  |  |  |  |
| 正在进行的匿名 gtid 违反事务计数 |  |  |  | 是 | 全局 | 否 |
| 正在进行的匿名事务计数 |  |  |  | 是 | 全局 | 否 |
| 正在进行的自动 gtid 违反事务计数 |  |  |  | 是 | 全局 | 否 |
| 已打开的文件 |  |  |  | 是 | 全局 | 否 |
| open_files_limit | 是 | 是 | 是 |  | 全局 | 否 |
| 已打开的流 |  |  |  | 是 | 全局 | 否 |
| 已打开的表定义 |  |  |  | 是 | 全局 | 否 |
| 已打开的表 |  |  |  | 是 | 两者 | 否 |
| 已打开的文件 |  |  |  | 是 | 全局 | 否 |
| 已打开的表定义 |  |  |  | 是 | 两者 | 否 |
| 已打开的表 |  |  |  | 是 | 两者 | 否 |
| optimizer_prune_level | 是 | 是 | 是 |  | 两者 | 是 |
| optimizer_search_depth | 是 | 是 | 是 |  | 两者 | 是 |
| optimizer_switch | 是 | 是 | 是 |  | 两者 | 是 |
| optimizer_trace | 是 | 是 | 是 |  | 两者 | 是 |
| optimizer_trace_features | 是 | 是 | 是 |  | 两者 | 是 |
| 优化器跟踪限制 | 是 | 是 | 是 |  | 两者 | 是 |
| 优化器跟踪最大内存大小 | 是 | 是 | 是 |  | 两者 | 是 |
| 优化器跟踪偏移量 | 是 | 是 | 是 |  | 两者 | 是 |
| 原始提交时间戳 |  |  | 是 |  | 会话 | 是 |
| 原始服务器版本 |  |  | 是 |  | 会话 | 是 |
| 解析器最大内存大小 | 是 | 是 | 是 |  | 两者 | 是 |
| 部分撤销 | 是 | 是 | 是 |  | 全局 | 是 |
| 密码历史 | 是 | 是 | 是 |  | 全局 | 是 |
| 密码要求当前 | 是 | 是 | 是 |  | 全局 | 是 |
| 密码重用间隔 | 是 | 是 | 是 |  | 全局 | 是 |
| 性能模式 | 是 | 是 | 是 |  | 全局 | 否 |
| 性能模式账户丢失 |  |  |  | 是 | 全局 | 否 |
| 性能模式账户大小 | 是 | 是 | 是 |  | 全局 | 否 |
| 性能模式条件类丢失 |  |  |  | 是 | 全局 | 否 |
| 性能模式条件实例丢失 |  |  |  | 是 | 全局 | 否 |
| 性能模式消费者事件阶段当前 | 是 | 是 |  |  |  |  |
| 性能模式消费者事件阶段历史 | 是 | 是 |  |  |  |  |
| 性能模式消费者事件阶段历史长 | 是 | 是 |  |  |  |  |
| 性能模式消费者事件语句 CPU | 是 | 是 |  |  |  |  |
| 性能模式消费者事件语句当前 | 是 | 是 |  |  |  |  |
| 性能模式消费者事件语句历史 | 是 | 是 |  |  |  |  |
| 性能模式消费者事件语句历史长 | 是 | 是 |  |  |  |  |
| 性能模式消费者事件事务当前 | 是 | 是 |  |  |  |  |
| 性能模式消费者事件事务历史 | 是 | 是 |  |  |  |  |
| 性能模式消费者事件事务历史长 | 是 | 是 |  |  |  |  |
| 性能模式消费者事件等待当前 | 是 | 是 |  |  |  |  |
| 性能模式消费者事件等待历史 | 是 | 是 |  |  |  |  |
| 性能模式消费者事件等待历史长 | 是 | 是 |  |  |  |  |
| 性能模式消费者全局仪器 | 是 | 是 |  |  |  |  |
| 性能模式消费者语句摘要 | 是 | 是 |  |  |  |  |
| 性能模式消费者线程仪器 | 是 | 是 |  |  |  |  |
| 性能模式消费者摘要丢失 |  |  |  | 是 | 全局 | 否 |
| performance_schema_digests_size | 是 | 是 | 是 |  | 全局 | 否 |
| performance_schema_error_size | 是 | 是 | 是 |  | 全局 | 否 |
| performance_schema_events_stages_history_long_size | 是 | 是 | 是 |  | 全局 | 否 |
| performance_schema_events_stages_history_size | 是 | 是 | 是 |  | 全局 | 否 |
| performance_schema_events_statements_history_long_size | 是 | 是 | 是 |  | 全局 | 否 |
| performance_schema_events_statements_history_size | 是 | 是 | 是 |  | 全局 | 否 |
| performance_schema_events_transactions_history_long_size | 是 | 是 | 是 |  | 全局 | 否 |
| performance_schema_events_transactions_history_size | 是 | 是 | 是 |  | 全局 | 否 |
| performance_schema_events_waits_history_long_size | 是 | 是 | 是 |  | 全局 | 否 |
| performance_schema_events_waits_history_size | 是 | 是 | 是 |  | 全局 | 否 |
| Performance_schema_file_classes_lost |  |  |  | 是 | 全局 | 否 |
| Performance_schema_file_handles_lost |  |  |  | 是 | 全局 | 否 |
| Performance_schema_file_instances_lost |  |  |  | 是 | 全局 | 否 |
| Performance_schema_hosts_lost |  |  |  | 是 | 全局 | 否 |
| performance_schema_hosts_size | 是 | 是 | 是 |  | 全局 | 否 |
| Performance_schema_index_stat_lost |  |  |  | 是 | 全局 | 否 |
| performance-schema-instrument | 是 | 是 |  |  |  |  |
| Performance_schema_locker_lost |  |  |  | 是 | 全局 | 否 |
| performance_schema_max_cond_classes | 是 | 是 | 是 |  | 全局 | 否 |
| performance_schema_max_cond_instances | 是 | 是 | 是 |  | 全局 | 否 |
| performance_schema_max_digest_length | 是 | 是 | 是 |  | 全局 | 否 |
| 性能模式最大摘要样本年龄 | 是 | 是 | 是 |  | 全局 | 是 |
| 性能模式最大文件类 | 是 | 是 | 是 |  | 全局 | 否 |
| 性能模式最大文件句柄 | 是 | 是 | 是 |  | 全局 | 否 |
| 性能模式最大文件实例 | 是 | 是 | 是 |  | 全局 | 否 |
| 性能模式最大索引统计 | 是 | 是 | 是 |  | 全局 | 否 |
| 性能模式最大内存类 | 是 | 是 | 是 |  | 全局 | 否 |
| 性能模式最大元数据锁 | 是 | 是 | 是 |  | 全局 | 否 |
| 性能模式最大互斥类 | 是 | 是 | 是 |  | 全局 | 否 |
| 性能模式最大互斥实例 | 是 | 是 | 是 |  | 全局 | 否 |
| 性能模式最大预处理语句实例 | 是 | 是 | 是 |  | 全局 | 否 |
| 性能模式最大��序实例 | 是 | 是 | 是 |  | 全局 | 否 |
| 性能模式最大读写锁类 | 是 | 是 | 是 |  | 全局 | 否 |
| 性能模式最大读写锁实例 | 是 | 是 | 是 |  | 全局 | 否 |
| 性能模式最大套接字类 | 是 | 是 | 是 |  | 全局 | 否 |
| 性能模式最大套接字实例 | 是 | 是 | 是 |  | 全局 | 否 |
| 性能模式最大 SQL 文本长度 | 是 | 是 | 是 |  | 全局 | 否 |
| 性能模式最大阶段类 | 是 | 是 | 是 |  | 全局 | 否 |
| 性能模式最大语句类 | 是 | 是 | 是 |  | 全局 | 否 |
| 性能模式最大语句堆栈 | 是 | 是 | 是 |  | 全局 | 否 |
| 性能模式最大表句柄 | 是 | 是 | 是 |  | 全局 | 否 |
| 性能模式最大表实例 | 是 | 是 | 是 |  | 全局 | 否 |
| 性能模式最大表锁统计 | 是 | 是 | 是 |  | 全局 | 否 |
| 性能模式最大线程类 | 是 | 是 | 是 |  | 全局 | 否 |
| 性能模式最大线程实例 | 是 | 是 | 是 |  | 全局 | 否 |
| 性能模式内存类丢失 |  |  |  | 是 | 全局 | 否 |
| 性能模式元数据锁丢失 |  |  |  | 是 | 全局 | 否 |
| 性能模式互斥类丢失 |  |  |  | 是 | 全局 | 否 |
| 性能模式互斥实例丢失 |  |  |  | 是 | 全局 | 否 |
| 性能模式嵌套语句丢失 |  |  |  | 是 | 全局 | 否 |
| 性能模式准备语句丢失 |  |  |  | 是 | 全局 | 否 |
| 性能模式程序丢失 |  |  |  | 是 | 全局 | 否 |
| 性能模式读写锁类丢失 |  |  |  | 是 | 全局 | 否 |
| 性能模式读写锁实例丢失 |  |  |  | 是 | 全局 | 否 |
| 性能模式会话连接属性最长见 |  |  |  | 是 | 全局 | 否 |
| 性能模式会话连接属性丢失 |  |  |  | 是 | 全局 | 否 |
| performance_schema_session_connect_attrs_size | 是 | 是 | 是 |  | 全局 | 否 |
| performance_schema_setup_actors_size | 是 | 是 | 是 |  | 全局 | 否 |
| performance_schema_setup_objects_size | 是 | 是 | 是 |  | 全局 | 否 |
| performance_schema_show_processlist | 是 | 是 | 是 |  | 全局 | 是 |
| 性能模式套接字类丢失 |  |  |  | 是 | 全局 | 否 |
| 性能模式套接字实例丢失 |  |  |  | 是 | 全局 | 否 |
| 性能模式阶段类丢失 |  |  |  | 是 | 全局 | 否 |
| 性能模式语句类丢失 |  |  |  | 是 | 全局 | 否 |
| 性能模式表句柄丢失 |  |  |  | 是 | 全局 | 否 |
| 性能模式表实例丢失 |  |  |  | 是 | 全局 | 否 |
| 性能模式表锁统计丢失 |  |  |  | 是 | 全局 | 否 |
| 性能模式线程类丢失 |  |  |  | 是 | 全局 | 否 |
| 性能模式线程实例丢失 |  |  |  | 是 | 全局 | 否 |
| 性能模式用户丢失 |  |  |  | 是 | 全局 | 否 |
| performance_schema_users_size | 是 | 是 | 是 |  | 全局 | 否 |
| persist_only_admin_x509_subject | 是 | 是 | 是 |  | 全局 | 否 |
| persist_sensitive_variables_in_plaintext | 是 | 是 | 是 |  | 全局 | 否 |
| persisted_globals_load | 是 | 是 | 是 |  | 全局 | 否 |
| pid_file | 是 | 是 | 是 |  | 全局 | 否 |
| plugin_dir | 是 | 是 | 是 |  | 全局 | 否 |
| plugin-load | 是 | 是 |  |  |  |  |
| plugin-load-add | 是 | 是 |  |  |  |  |
| plugin-xxx | 是 | 是 |  |  |  |  |
| port | 是 | 是 | 是 |  | 全局 | 否 |
| port-open-timeout | 是 | 是 |  |  |  |  |
| preload_buffer_size | 是 | 是 | 是 |  | 两者 | 是 |
| Prepared_stmt_count |  |  |  | 是 | 全局 | 否 |
| print-defaults | 是 |  |  |  |  |  |
| print_identified_with_as_hex | 是 | 是 | 是 |  | 两者 | 是 |
| profiling |  |  | 是 |  | 两者 | 是 |
| profiling_history_size | 是 | 是 | 是 |  | 两者 | 是 |
| protocol_compression_algorithms | 是 | 是 | 是 |  | 全局 | 是 |
| protocol_version |  |  | 是 |  | 全局 | 否 |
| proxy_user |  |  | 是 |  | 会话 | 否 |
| pseudo_replica_mode |  |  | 是 |  | 会话 | 是 |
| pseudo_slave_mode |  |  | 是 |  | 会话 | 是 |
| pseudo_thread_id |  |  | 是 |  | 会话 | 是 |
| Queries |  |  |  | 是 | 两者 | 否 |
| query_alloc_block_size | 是 | 是 | 是 |  | 两者 | 是 |
| query_prealloc_size | 是 | 是 | 是 |  | 两者 | 是 |
| Questions |  |  |  | 是 | 两者 | 否 |
| rand_seed1 |  |  | 是 |  | 会话 | 是 |
| rand_seed2 |  |  | 是 |  | 会话 | 是 |
| range_alloc_block_size | 是 | 是 | 是 |  | 两者 | 是 |
| range_optimizer_max_mem_size | 是 | 是 | 是 |  | 两者 | 是 |
| rbr_exec_mode |  |  | 是 |  | 会话 | 是 |
| read_buffer_size | 是 | 是 | 是 |  | 两者 | 是 |
| read_only | 是 | 是 | 是 |  | 全局 | 是 |
| read_rnd_buffer_size | 是 | 是 | 是 |  | 两者 | 是 |
| regexp_stack_limit | 是 | 是 | 是 |  | 全局 | 是 |
| regexp_time_limit | 是 | 是 | 是 |  | 全局 | 是 |
| relay_log | 是 | 是 | 是 |  | 全局 | 否 |
| relay_log_basename |  |  | 是 |  | 全局 | 否 |
| relay_log_index | 是 | 是 | 是 |  | 全局 | 否 |
| relay_log_info_file | 是 | 是 | 是 |  | 全局 | 否 |
| relay_log_info_repository | 是 | 是 | 是 |  | 全局 | 是 |
| relay_log_purge | 是 | 是 | 是 |  | 全局 | 是 |
| relay_log_recovery | 是 | 是 | 是 |  | 全局 | 否 |
| relay_log_space_limit | 是 | 是 | 是 |  | 全局 | 否 |
| remove | 是 |  |  |  |  |  |
| replica_allow_batching | 是 | 是 | 是 |  | 全局 | 是 |
| replica_checkpoint_group | 是 | 是 | 是 |  | 全局 | 是 |
| replica_checkpoint_period | 是 | 是 | 是 |  | 全局 | 是 |
| replica_compressed_protocol | 是 | 是 | 是 |  | 全局 | 是 |
| replica_exec_mode | 是 | 是 | 是 |  | 全局 | 是 |
| replica_load_tmpdir | 是 | 是 | 是 |  | 全局 | 否 |
| replica_max_allowed_packet | 是 | 是 | 是 |  | 全局 | 是 |
| replica_net_timeout | 是 | 是 | 是 |  | 全局 | 是 |
| Replica_open_temp_tables |  |  |  | 是 | 全局 | 否 |
| replica_parallel_type | 是 | 是 | 是 |  | 全局 | 是 |
| 复制并行工作者 | 是 | 是 | 是 |  | 全局 | 是 |
| 复制挂起作业最大大小 | 是 | 是 | 是 |  | 全局 | 是 |
| 复制保留提交顺序 | 是 | 是 | 是 |  | 全局 | 是 |
| 复制行上次使用的搜索算法 |  |  |  | 是 | 全局 | 否 |
| 复制跳过错误 | 是 | 是 | 是 |  | 全局 | 否 |
| 复制 SQL 验证校验和 | 是 | 是 | 是 |  | 全局 | 是 |
| 复制事务重试次数 | 是 | 是 | 是 |  | 全局 | 是 |
| 复制类型转换 | 是 | 是 | 是 |  | 全局 | 是 |
| 复制执行数据库 | 是 | 是 |  |  |  |  |
| 复制执行表 | 是 | 是 |  |  |  |  |
| 复制忽略数据库 | 是 | 是 |  |  |  |  |
| 复制忽略表 | 是 | 是 |  |  |  |  |
| 复制重写数据库 | 是 | 是 |  |  |  |  |
| 复制相同服务器 ID | 是 | 是 |  |  |  |  |
| 复制通配执行表 | 是 | 是 |  |  |  |  |
| 复制通配忽略表 | 是 | 是 |  |  |  |  |
| 复制优化静态插件配置 | 是 | 是 | 是 |  | 全局 | 是 |
| 复制发送者观察仅提交 | 是 | 是 | 是 |  | 全局 | 是 |
| 报告主机 | 是 | 是 | 是 |  | 全局 | 否 |
| 报告密码 | 是 | 是 | 是 |  | 全局 | 否 |
| 报告端口 | 是 | 是 | 是 |  | 全局 | 否 |
| 报告用户 | 是 | 是 | 是 |  | 全局 | 否 |
| 要求行格式 |  |  | 是 |  | 会话 | 是 |
| require_secure_transport | 是 | 是 | 是 |  | 全局 | 是 |
| Resource_group_supported |  |  |  | 是 | 全局 | 否 |
| resultset_metadata |  |  | 是 |  | 会话 | 是 |
| rewriter_enabled |  |  | 是 |  | 全局 | 是 |
| rewriter_enabled_for_threads_without_privilege_checks |  |  | 是 |  | 全局 | 是 |
| Rewriter_number_loaded_rules |  |  |  | 是 | 全局 | 否 |
| Rewriter_number_reloads |  |  |  | 是 | 全局 | 否 |
| Rewriter_number_rewritten_queries |  |  |  | 是 | 全局 | 否 |
| Rewriter_reload_error |  |  |  | 是 | 全局 | 否 |
| rewriter_verbose |  |  | 是 |  | 全局 | 是 |
| rpl_read_size | 是 | 是 | 是 |  | 全局 | 是 |
| Rpl_semi_sync_master_clients |  |  |  | 是 | 全局 | 否 |
| rpl_semi_sync_master_enabled | 是 | 是 | 是 |  | 全局 | 是 |
| Rpl_semi_sync_master_net_avg_wait_time |  |  |  | 是 | 全局 | 否 |
| Rpl_semi_sync_master_net_wait_time |  |  |  | 是 | 全局 | 否 |
| Rpl_semi_sync_master_net_waits |  |  |  | 是 | 全局 | 否 |
| Rpl_semi_sync_master_no_times |  |  |  | 是 | 全局 | 否 |
| Rpl_semi_sync_master_no_tx |  |  |  | 是 | 全局 | 否 |
| Rpl_semi_sync_master_status |  |  |  | 是 | 全局 | 否 |
| Rpl_semi_sync_master_timefunc_failures |  |  |  | 是 | 全局 | 否 |
| rpl_semi_sync_master_timeout | 是 | 是 | 是 |  | 全局 | 是 |
| rpl_semi_sync_master_trace_level | 是 | 是 | 是 |  | 全局 | 是 |
| Rpl_semi_sync_master_tx_avg_wait_time |  |  |  | 是 | 全局 | 否 |
| Rpl_semi_sync_master_tx_wait_time |  |  |  | 是 | 全局 | 否 |
| Rpl_semi_sync_master_tx_waits |  |  |  | 是 | 全局 | 否 |
| rpl_semi_sync_master_wait_for_slave_count | 是 | 是 | 是 |  | 全局 | 是 |
| rpl_semi_sync_master_wait_no_slave | 是 | 是 | 是 |  | 全局 | 是 |
| rpl_semi_sync_master_wait_point | 是 | 是 | 是 |  | 全局 | 是 |
| Rpl_semi_sync_master_wait_pos_backtraverse |  |  |  | 是 | 全局 | 否 |
| Rpl_semi_sync_master_wait_sessions |  |  |  | 是 | 全局 | 否 |
| Rpl_semi_sync_master_yes_tx |  |  |  | 是 | 全局 | 否 |
| rpl_semi_sync_replica_enabled | 是 | 是 | 是 |  | 全局 | 是 |
| Rpl_semi_sync_replica_status |  |  |  | 是 | 全局 | 否 |
| rpl_semi_sync_replica_trace_level | 是 | 是 | 是 |  | 全局 | 是 |
| rpl_semi_sync_slave_enabled | 是 | 是 | 是 |  | 全局 | 是 |
| Rpl_semi_sync_slave_status |  |  |  | 是 | 全局 | 否 |
| rpl_semi_sync_slave_trace_level | 是 | 是 | 是 |  | 全局 | 是 |
| Rpl_semi_sync_source_clients |  |  |  | 是 | 全局 | 否 |
| rpl_semi_sync_source_enabled | 是 | 是 | 是 |  | 全局 | 是 |
| Rpl_semi_sync_source_net_avg_wait_time |  |  |  | 是 | 全局 | 否 |
| Rpl_semi_sync_source_net_wait_time |  |  |  | 是 | 全局 | 否 |
| Rpl_semi_sync_source_net_waits |  |  |  | 是 | 全局 | 否 |
| Rpl_semi_sync_source_no_times |  |  |  | 是 | 全局 | 否 |
| Rpl_semi_sync_source_no_tx |  |  |  | 是 | 全局 | 否 |
| Rpl_semi_sync_source_status |  |  |  | 是 | 全局 | 否 |
| Rpl_semi_sync_source_timefunc_failures |  |  |  | 是 | 全局 | 否 |
| rpl_semi_sync_source_timeout | 是 | 是 | 是 |  | 全局 | 是 |
| rpl_semi_sync_source_trace_level | 是 | 是 | 是 |  | 全局 | 是 |
| Rpl_semi_sync_source_tx_avg_wait_time |  |  |  | 是 | 全局 | 否 |
| Rpl_semi_sync_source_tx_wait_time |  |  |  | 是 | 全局 | 否 |
| Rpl_semi_sync_source_tx_waits |  |  |  | 是 | 全局 | 否 |
| rpl_semi_sync_source_wait_for_replica_count | 是 | 是 | 是 |  | 全局 | 是 |
| rpl_semi_sync_source_wait_no_replica | 是 | 是 | 是 |  | 全局 | 是 |
| rpl_semi_sync_source_wait_point | 是 | 是 | 是 |  | 全局 | 是 |
| Rpl_semi_sync_source_wait_pos_backtraverse |  |  |  | 是 | 全局 | 否 |
| Rpl_semi_sync_source_wait_sessions |  |  |  | 是 | 全局 | 否 |
| Rpl_semi_sync_source_yes_tx |  |  |  | 是 | 全局 | 否 |
| rpl_stop_replica_timeout | 是 | 是 | 是 |  | 全局 | 是 |
| rpl_stop_slave_timeout | 是 | 是 | 是 |  | 全局 | 是 |
| Rsa_public_key |  |  |  | 是 | 全局 | 否 |
| safe-user-create | 是 | 是 |  |  |  |  |
| 模式定义缓存 | 是 | 是 | 是 |  | 全局 | 是 |
| secondary_engine_cost_threshold |  |  | 是 |  | 会话 | 是 |
| 次要引擎执行计数 |  |  |  | 是 | 两者 | 否 |
| secure_file_priv | 是 | 是 | 是 |  | 全局 | 否 |
| 选择完整连接 |  |  |  | 是 | 两者 | 否 |
| 选择完整范围连接 |  |  |  | 是 | 两者 | 否 |
| select_into_buffer_size | 是 | 是 | 是 |  | 两者 | 是 |
| select_into_disk_sync | 是 | 是 | 是 |  | 两者 | 是 |
| select_into_disk_sync_delay | 是 | 是 | 是 |  | 两者 | 是 |
| 选择范围 |  |  |  | 是 | 两者 | 否 |
| 选择范围检查 |  |  |  | 是 | 两者 | 否 |
| 选择扫描 |  |  |  | 是 | 两者 | 否 |
| server_id | 是 | 是 | 是 |  | 全局 | 是 |
| server_id_bits | 是 | 是 | 是 |  | 全局 | 否 |
| server_uuid |  |  | 是 |  | 全局 | 否 |
| session_track_gtids | 是 | 是 | 是 |  | 两者 | 是 |
| session_track_schema | 是 | 是 | 是 |  | 两者 | 是 |
| session_track_state_change | 是 | 是 | 是 |  | 两者 | 是 |
| session_track_system_variables | 是 | 是 | 是 |  | 两者 | 是 |
| session_track_transaction_info | 是 | 是 | 是 |  | 两者 | 是 |
| sha256_password_auto_generate_rsa_keys | 是 | 是 | 是 |  | 全局 | 否 |
| sha256_password_private_key_path | 是 | 是 | 是 |  | 全局 | 否 |
| sha256_password_proxy_users | 是 | 是 | 是 |  | 全局 | 是 |
| sha256_password_public_key_path | 是 | 是 | 是 |  | 全局 | 否 |
| shared_memory | 是 | 是 | 是 |  | 全局 | 否 |
| 共享内存基本名称 | 是 | 是 | 是 |  | 全局 | 否 |
| 在创建表时跳过辅助引擎 | 是 | 是 | 是 |  | 会话 | 是 |
| show_create_table_verbosity | 是 | 是 | 是 |  | 两者 | 是 |
| 在创建表和信息模式中显示 GIPK | 是 | 是 | 是 |  | 两者 | 是 |
| 显示旧的时间格式 | 是 | 是 | 是 |  | 两者 | 是 |
| show-replica-auth-info | 是 | 是 |  |  |  |  |
| show-slave-auth-info | 是 | 是 |  |  |  |  |
| 跳过字符集客户端握手 | 是 | 是 |  |  |  |  |
| 跳过外部锁定 | 是 | 是 | 是 |  | 全局 | 否 |
| 跳过授权表 | 是 | 是 |  |  |  |  |
| 跳过主机缓存 | 是 | 是 |  |  |  |  |
| 跳过名称解析 | 是 | 是 | 是 |  | 全局 | 否 |
| 跳过-ndbcluster | 是 | 是 |  |  |  |  |
| 跳过网络连接 | 是 | 是 | 是 |  | 全局 | 否 |
| 跳过新特性 | 是 | 是 |  |  |  |  |
| 跳过复制开始 | 是 | 是 | 是 |  | 全局 | 否 |
| 跳过显示数据库 | 是 | 是 | 是 |  | 全局 | 否 |
| 跳过从库开始 | 是 | 是 | 是 |  | 全局 | 否 |
| 跳过 SSL | 是 | 是 |  |  |  |  |
| 跳过堆栈跟踪 | 是 | 是 |  |  |  |  |
| 允许批处理的从库 | 是 | 是 | 是 |  | 全局 | 是 |
| 从库检查点组 | 是 | 是 | 是 |  | 全局 | 是 |
| 从库检查点周期 | 是 | 是 | 是 |  | 全局 | 是 |
| 从库压缩协议 | 是 | 是 | 是 |  | 全局 | 是 |
| slave_exec_mode | 是 | 是 | 是 |  | 全局 | 是 |
| slave_load_tmpdir | 是 | 是 | 是 |  | 全局 | 否 |
| slave_max_allowed_packet | 是 | 是 | 是 |  | 全局 | 是 |
| slave_net_timeout | 是 | 是 | 是 |  | 全局 | 是 |
| Slave_open_temp_tables |  |  |  | 是 | 全局 | 否 |
| slave_parallel_type | 是 | 是 | 是 |  | 全局 | 是 |
| slave_parallel_workers | 是 | 是 | 是 |  | 全局 | 是 |
| slave_pending_jobs_size_max | 是 | 是 | 是 |  | 全局 | 是 |
| slave_preserve_commit_order | 是 | 是 | 是 |  | 全局 | 是 |
| Slave_rows_last_search_algorithm_used |  |  |  | 是 | 全局 | 否 |
| slave_rows_search_algorithms | 是 | 是 | 是 |  | 全局 | 是 |
| slave_skip_errors | 是 | 是 | 是 |  | 全局 | 否 |
| slave-sql-verify-checksum | 是 | 是 |  |  |  |  |
| slave_sql_verify_checksum | 是 | 是 | 是 |  | 全局 | 是 |
| slave_transaction_retries | 是 | 是 | 是 |  | 全局 | 是 |
| slave_type_conversions | 是 | 是 | 是 |  | 全局 | 是 |
| Slow_launch_threads |  |  |  | 是 | 两者 | 否 |
| slow_launch_time | 是 | 是 | 是 |  | 全局 | 是 |
| Slow_queries |  |  |  | 是 | 两者 | 否 |
| slow_query_log | 是 | 是 | 是 |  | 全局 | 是 |
| slow_query_log_file | 是 | 是 | 是 |  | 全局 | 是 |
| slow-start-timeout | 是 | 是 |  |  |  |  |
| socket | 是 | 是 | 是 |  | 全局 | 否 |
| sort_buffer_size | 是 | 是 | 是 |  | 两者 | 是 |
| Sort_merge_passes |  |  |  | 是 | 两者 | 否 |
| Sort_range |  |  |  | 是 | 两者 | 否 |
| Sort_rows |  |  |  | 是 | 两者 | 否 |
| Sort_scan |  |  |  | 是 | 两者 | 否 |
| source_verify_checksum | 是 | 是 | 是 |  | 全局 | 是 |
| sporadic-binlog-dump-fail | 是 | 是 |  |  |  |  |
| sql_auto_is_null |  |  | 是 |  | 两者 | 是 |
| sql_big_selects |  |  | 是 |  | 两者 | 是 |
| sql_buffer_result |  |  | 是 |  | 两者 | 是 |
| sql_generate_invisible_primary_key | 是 | 是 | 是 |  | 两者 | 是 |
| sql_log_bin |  |  | 是 |  | 会话 | 是 |
| sql_log_off |  |  | 是 |  | 两者 | 是 |
| sql_mode | 是 | 是 | 是 |  | 两者 | 是 |
| sql_notes |  |  | 是 |  | 两者 | 是 |
| sql_quote_show_create |  |  | 是 |  | 两者 | 是 |
| sql_replica_skip_counter |  |  | 是 |  | 全局 | 是 |
| sql_require_primary_key | 是 | 是 | 是 |  | 两者 | 是 |
| sql_safe_updates |  |  | 是 |  | 两者 | 是 |
| sql_select_limit |  |  | 是 |  | 两者 | 是 |
| sql_slave_skip_counter |  |  | 是 |  | 全局 | 是 |
| sql_warnings |  |  | 是 |  | 两者 | 是 |
| ssl | 是 | 是 |  |  |  |  |
| Ssl_accept_renegotiates |  |  |  | 是 | 全局 | 否 |
| Ssl_accepts |  |  |  | 是 | 全局 | 否 |
| ssl_ca | 是 | 是 | 是 |  | 全局 | 不定 |
| Ssl_callback_cache_hits |  |  |  | 是 | 全局 | 否 |
| ssl_capath | 是 | 是 | 是 |  | 全局 | 不定 |
| ssl_cert | 是 | 是 | 是 |  | 全局 | 不定 |
| Ssl_cipher |  |  |  | 是 | 两者 | 否 |
| ssl_cipher | 是 | 是 | 是 |  | 全局 | 不同 |
| Ssl_cipher_list |  |  |  | 是 | 两者 | 否 |
| Ssl_client_connects |  |  |  | 是 | 全局 | 否 |
| Ssl_connect_renegotiates |  |  |  | 是 | 全局 | 否 |
| ssl_crl | 是 | 是 | 是 |  | 全局 | 不同 |
| ssl_crlpath | 是 | 是 | 是 |  | 全局 | 不同 |
| Ssl_ctx_verify_depth |  |  |  | 是 | 全局 | 否 |
| Ssl_ctx_verify_mode |  |  |  | 是 | 全局 | 否 |
| Ssl_default_timeout |  |  |  | 是 | 两者 | 否 |
| Ssl_finished_accepts |  |  |  | 是 | 全局 | 否 |
| Ssl_finished_connects |  |  |  | 是 | 全局 | 否 |
| ssl_fips_mode | 是 | 是 | 是 |  | 全局 | 否 |
| ssl_key | 是 | 是 | 是 |  | 全局 | 不同 |
| Ssl_server_not_after |  |  |  | 是 | 两者 | 否 |
| Ssl_server_not_before |  |  |  | 是 | 两者 | 否 |
| Ssl_session_cache_hits |  |  |  | 是 | 全局 | 否 |
| Ssl_session_cache_misses |  |  |  | 是 | 全局 | 否 |
| Ssl_session_cache_mode |  |  |  | 是 | 全局 | 否 |
| ssl_session_cache_mode | 是 | 是 | 是 |  | 全局 | 是 |
| Ssl_session_cache_overflows |  |  |  | 是 | 全局 | 否 |
| Ssl_session_cache_size |  |  |  | 是 | 全局 | 否 |
| Ssl_session_cache_timeout |  |  |  | 是 | 全局 | 否 |
| ssl_session_cache_timeout | 是 | 是 | 是 |  | 全局 | 是 |
| Ssl_session_cache_timeouts |  |  |  | 是 | 全局 | 否 |
| Ssl_sessions_reused |  |  |  | 是 | 会话 | 否 |
| Ssl_used_session_cache_entries |  |  |  | 是 | 全局 | 否 |
| Ssl_verify_depth |  |  |  | 是 | 两者 | 否 |
| Ssl_verify_mode |  |  |  | 是 | 两者 | 否 |
| Ssl_version |  |  |  | 是 | 两者 | 否 |
| standalone | 是 | 是 |  |  |  |  |
| stored_program_cache | 是 | 是 | 是 |  | 全局 | 是 |
| stored_program_definition_cache | 是 | 是 | 是 |  | 全局 | 是 |
| super-large-pages | 是 | 是 |  |  |  |  |
| super_read_only | 是 | 是 | 是 |  | 全局 | 是 |
| symbolic-links | 是 | 是 |  |  |  |  |
| sync_binlog | 是 | 是 | 是 |  | 全局 | 是 |
| sync_master_info | 是 | 是 | 是 |  | 全局 | 是 |
| sync_relay_log | 是 | 是 | 是 |  | 全局 | 是 |
| sync_relay_log_info | 是 | 是 | 是 |  | 全局 | 是 |
| sync_source_info | 是 | 是 | 是 |  | 全局 | 是 |
| sysdate-is-now | 是 | 是 |  |  |  |  |
| syseventlog.facility | 是 | 是 | 是 |  | 全局 | 是 |
| syseventlog.include_pid | 是 | 是 | 是 |  | 全局 | 是 |
| syseventlog.tag | 是 | 是 | 是 |  | 全局 | 是 |
| system_time_zone |  |  | 是 |  | 全局 | 否 |
| table_definition_cache | 是 | 是 | 是 |  | 全局 | 是 |
| table_encryption_privilege_check | 是 | 是 | 是 |  | 全局 | 是 |
| Table_locks_immediate |  |  |  | 是 | 全局 | 否 |
| Table_locks_waited |  |  |  | 是 | 全局 | 否 |
| table_open_cache | 是 | 是 | 是 |  | 全局 | 是 |
| Table_open_cache_hits |  |  |  | 是 | 两者 | 否 |
| 表打开缓存实例数 | 是 | 是 | 是 |  | 全局 | 否 |
| 表打开缓存未命中次数 |  |  |  | 是 | 双方 | 否 |
| 表打开缓存溢出次数 |  |  |  | 是 | 双方 | 否 |
| 表空间定义缓存 | 是 | 是 | 是 |  | 全局 | 是 |
| tc-heuristic-recover | 是 | 是 |  |  |  |  |
| Tc_log_max_pages_used |  |  |  | 是 | 全局 | 否 |
| Tc_log_page_size |  |  |  | 是 | 全局 | 否 |
| Tc_log_page_waits |  |  |  | 是 | 全局 | 否 |
| 遥测跟踪支持 |  |  |  | 是 | 全局 | 否 |
| 临时表最大内存映射 | 是 | 是 | 是 |  | 全局 | 是 |
| 临时表最大内存 | 是 | 是 | 是 |  | 全局 | 是 |
| 临时表使用内存映射 | 是 | 是 | 是 |  | 全局 | 是 |
| 术语使用先前 | 是 | 是 | 是 |  | 双方 | 是 |
| 线程缓存大小 | 是 | 是 | 是 |  | 全局 | 是 |
| 线程处理 | 是 | 是 | 是 |  | 全局 | 否 |
| 线程池算法 | 是 | 是 | 是 |  | 全局 | 否 |
| 线程池专用监听器 | 是 | 是 | 是 |  | 全局 | 否 |
| 线程池高优先级连接 | 是 | 是 | 是 |  | 双方 | 是 |
| 线程池最大活动查询线程数限制 | 是 | 是 | 是 |  | 全局 | 是 |
| 线程池最大事务限制 | 是 | 是 | 是 |  | 全局 | 是 |
| 线程池最大未使用线程数 | 是 | 是 | 是 |  | 全局 | 是 |
| 线程池优先级提升计时器 | 是 | 是 | 是 |  | 全局 | 是 |
| 线程池查询线程每组数 | 是 | 是 | 是 |  | 全局 | 是 |
| 线程池大小 | 是 | 是 | 是 |  | 全局 | 否 |
| 线程池停滞限制 | 是 | 是 | 是 |  | 全局 | 是 |
| 线程池事务延迟 | 是 | 是 | 是 |  | 全局 | 是 |
| 线程堆栈大小 | 是 | 是 | 是 |  | 全局 | 否 |
| 缓存的线程 |  |  |  | 是 | 全局 | 否 |
| 连接的线程数 |  |  |  | 是 | 全局 | 否 |
| 创建的线程数 |  |  |  | 是 | 全局 | 否 |
| 运行的线程数 |  |  |  | 是 | 全局 | 否 |
| 时区 |  |  | 是 |  | 两者 | 是 |
| 时间戳 |  |  | 是 |  | 会话 | 是 |
| TLS 密码套件 | 是 | 是 | 是 |  | 全局 | 是 |
| TLS 库版本 |  |  |  | 是 | 全局 | 否 |
| TLS 版本 | 是 | 是 | 是 |  | 全局 | 不定 |
| 临时表大小 | 是 | 是 | 是 |  | 两者 | 是 |
| 临时目录 | 是 | 是 | 是 |  | 全局 | 否 |
| 事务分配块大小 | 是 | 是 | 是 |  | 两者 | 是 |
| 允许批量处理事务 |  |  | 是 |  | 会话 | 是 |
| 事务隔离级别 | 是 | 是 | 是 |  | 两者 | 是 |
| 事务预分配大小 | 是 | 是 | 是 |  | 两者 | 是 |
| 只读事务 | 是 | 是 | 是 |  | 两者 | 是 |
| 事务写集提取 | 是 | 是 | 是 |  | 两者 | 是 |
| 唯一性检查 |  |  | 是 |  | 两者 | 是 |
| 可更新视图限制 | 是 | 是 | 是 |  | 两者 | 是 |
| 升级 | 是 | 是 |  |  |  |  |
| 运行时间 |  |  |  | 是 | 全局 | 否 |
| 刷新状态后的运行时间 |  |  |  | 是 | 全局 | 否 |
| use_secondary_engine |  |  | Yes |  | Session | Yes |
| user | Yes | Yes |  |  |  |  |
| validate-config | Yes | Yes |  |  |  |  |
| validate-password | Yes | Yes |  |  |  |  |
| validate_password_check_user_name | Yes | Yes | Yes |  | Global | Yes |
| validate_password_dictionary_file | Yes | Yes | Yes |  | Global | Yes |
| validate_password_dictionary_file_last_parsed |  |  |  | Yes | Global | No |
| validate_password_dictionary_file_words_count |  |  |  | Yes | Global | No |
| validate_password_length | Yes | Yes | Yes |  | Global | Yes |
| validate_password_mixed_case_count | Yes | Yes | Yes |  | Global | Yes |
| validate_password_number_count | Yes | Yes | Yes |  | Global | Yes |
| validate_password_policy | Yes | Yes | Yes |  | Global | Yes |
| validate_password_special_char_count | Yes | Yes | Yes |  | Global | Yes |
| validate_password.changed_characters_percentage | Yes | Yes | Yes |  | Global | Yes |
| validate_password.check_user_name | Yes | Yes | Yes |  | Global | Yes |
| validate_password.dictionary_file | Yes | Yes | Yes |  | Global | Yes |
| validate_password.dictionary_file_last_parsed |  |  |  | Yes | Global | No |
| validate_password.dictionary_file_words_count |  |  |  | Yes | Global | No |
| validate_password.length | Yes | Yes | Yes |  | Global | Yes |
| validate_password.mixed_case_count | 是 | 是 | 是 |  | 全局 | 是 |
| validate_password.number_count | 是 | 是 | 是 |  | 全局 | 是 |
| validate_password.policy | 是 | 是 | 是 |  | 全局 | 是 |
| validate_password.special_char_count | 是 | 是 | 是 |  | 全局 | 是 |
| validate-user-plugins | 是 | 是 |  |  |  |  |
| verbose | 是 | 是 |  |  |  |  |
| version |  |  | 是 |  | 全局 | 否 |
| version_comment |  |  | 是 |  | 全局 | 否 |
| version_compile_machine |  |  | 是 |  | 全局 | 否 |
| version_compile_os |  |  | 是 |  | 全局 | 否 |
| version_compile_zlib |  |  | 是 |  | 全局 | 否 |
| version_tokens_session | 是 | 是 | 是 |  | 两者 | 是 |
| version_tokens_session_number | 是 | 是 | 是 |  | 两者 | 否 |
| wait_timeout | 是 | 是 | 是 |  | 两者 | 是 |
| warning_count |  |  | 是 |  | 会话 | 否 |
| windowing_use_high_precision | 是 | 是 | 是 |  | 两者 | 是 |
| xa_detach_on_prepare | 是 | 是 | 是 |  | 两者 | 是 |
| 名称 | 命令行 | 选项文件 | 系统变量 | 状态变量 | 变量范围 | 动态 |

**注意：**

1\. 此选项是动态的，但应仅由服务器设置。不应手动设置此变量。
