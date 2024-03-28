# 7.1.5 服务器系统变量参考

> 原文：[`dev.mysql.com/doc/refman/8.0/en/server-system-variable-reference.html`](https://dev.mysql.com/doc/refman/8.0/en/server-system-variable-reference.html)

以下表格列出了所有适用于`mysqld`的系统变量。

表格列出了命令行选项（Cmd-line）、配置文件中有效的选项（Option file）、服务器系统变量（System Var）和状态变量（Status var）在一个统一的列表中，指示每个选项或变量的有效位置。如果在命令行或选项文件中设置的服务器选项与相应系统变量的名称不同，那么变量名称将立即在相应选项下方注明。变量的范围（Var Scope）是全局的、会话的或两者都是。请参阅相应项目描述以了解设置和使用变量的详细信息。在适当的情况下，提供了有关项目的更多信息的直接链接。

**表格 7.2 系统变量摘要**

| 名称 | 命令行 | 选项文件 | 系统变量 | 变量范围 | 动态 |
| --- | --- | --- | --- | --- | --- |
| 登录时激活所有角色 | 是 | 是 | 是 | 全局 | 是 |
| admin_address | 是 | 是 | 是 | 全局 | 否 |
| admin_port | 是 | 是 | 是 | 全局 | 否 |
| admin_ssl_ca | 是 | 是 | 是 | 全局 | 是 |
| admin_ssl_capath | 是 | 是 | 是 | 全局 | 是 |
| admin_ssl_cert | 是 | 是 | 是 | 全局 | 是 |
| admin_ssl_cipher | 是 | 是 | 是 | 全局 | 是 |
| admin_ssl_crl | 是 | 是 | 是 | 全局 | 是 |
| admin_ssl_crlpath | 是 | 是 | 是 | 全局 | 是 |
| admin_ssl_key | 是 | 是 | 是 | 全局 | 是 |
| admin_tls_ciphersuites | 是 | 是 | 是 | 全局 | 是 |
| admin_tls_version | 是 | 是 | 是 | 全局 | 是 |
| 审计日志缓冲区大小 | 是 | 是 | 是 | 全局 | 否 |
| 审计日��压缩 | 是 | 是 | 是 | 全局 | 否 |
| 审计日志连接策略 | 是 | 是 | 是 | 全局 | 是 |
| 审计日志当前会话 |  |  | 是 | 两者 | 否 |
| 审计日志数据库 | 是 | 是 | 是 | 全局 | 否 |
| 禁用审计日志 | 是 | 是 | 是 | 全局 | 是 |
| 审计日志加密 | 是 | 是 | 是 | 全局 | 否 |
| 审计日志排除账户 | 是 | 是 | 是 | 全局 | 是 |
| 审计日志文件 | 是 | 是 | 是 | 全局 | 否 |
| 审计日志过滤器 ID |  |  | 是 | 两者 | 否 |
| 审计日志刷新 |  |  | 是 | 全局 | 是 |
| 审计日志刷新间隔秒数 | 是 |  | 是 | 全局 | 否 |
| 审计日志格式 | 是 | 是 | 是 | 全局 | 否 |
| 审计日志格式 Unix 时间戳 | 是 | 是 | 是 | 全局 | 是 |
| 审计日志包含账户 | 是 | 是 | 是 | 全局 | 是 |
| 审计日志最大大小 | 是 | 是 | 是 | 全局 | 是 |
| 审计日志密码历史保留天数 | 是 | 是 | 是 | 全局 | 是 |
| 审计日志策略 | 是 | 是 | 是 | 全局 | 否 |
| 审计日志修剪秒数 | 是 | 是 | 是 | 全局 | 是 |
| 审计日志读取缓冲区大小 | 是 | 是 | 是 | 不定 | 不定 |
| 按大小轮转审计日志 | 是 | 是 | 是 | 全局 | 是 |
| 审计日志语句策略 | 是 | 是 | 是 | 全局 | 是 |
| 审计日志策略 | 是 | 是 | 是 | 全局 | 否 |
| 身份验证 FIDO RP ID | 是 | 是 | 是 | 全局 | 是 |
| 身份验证 Kerberos 服务密钥表 | 是 | 是 | 是 | 全局 | 否 |
| 身份验证 Kerberos 服务主体 | 是 | 是 | 是 | 全局 | 是 |
| 身份验证 LDAP SASL 认证方法名称 | 是 | 是 | 是 | 全局 | 是 |
| 身份验证 LDAP SASL 绑定基本 DN | 是 | 是 | 是 | 全局 | 是 |
| authentication_ldap_sasl_bind_root_dn | 是 | 是 | 是 | 全局 | 是 |
| authentication_ldap_sasl_bind_root_pwd | 是 | 是 | 是 | 全局 | 是 |
| authentication_ldap_sasl_ca_path | 是 | 是 | 是 | 全局 | 是 |
| authentication_ldap_sasl_group_search_attr | 是 | 是 | 是 | 全局 | 是 |
| authentication_ldap_sasl_group_search_filter | 是 | 是 | 是 | 全局 | 是 |
| authentication_ldap_sasl_init_pool_size | 是 | 是 | 是 | 全局 | 是 |
| authentication_ldap_sasl_log_status | 是 | 是 | 是 | 全局 | 是 |
| authentication_ldap_sasl_max_pool_size | 是 | 是 | 是 | 全局 | 是 |
| authentication_ldap_sasl_referral | 是 | 是 | 是 | 全局 | 是 |
| authentication_ldap_sasl_server_host | 是 | 是 | 是 | 全局 | 是 |
| authentication_ldap_sasl_server_port | 是 | 是 | 是 | 全局 | 是 |
| authentication_ldap_sasl_tls | 是 | 是 | 是 | 全局 | 是 |
| authentication_ldap_sasl_user_search_attr | 是 | 是 | 是 | 全局 | 是 |
| authentication_ldap_simple_auth_method_name | 是 | 是 | 是 | 全局 | 是 |
| authentication_ldap_simple_bind_base_dn | 是 | 是 | 是 | 全局 | 是 |
| authentication_ldap_simple_bind_root_dn | 是 | 是 | 是 | 全局 | 是 |
| authentication_ldap_simple_bind_root_pwd | 是 | 是 | 是 | 全局 | 是 |
| authentication_ldap_simple_ca_path | 是 | 是 | 是 | 全局 | 是 |
| authentication_ldap_simple_group_search_attr | 是 | 是 | 是 | 全局 | 是 |
| authentication_ldap_simple_group_search_filter | 是 | 是 | 是 | 全局 | 是 |
| authentication_ldap_simple_init_pool_size | 是 | 是 | 是 | 全局 | 是 |
| authentication_ldap_simple_log_status | 是 | 是 | 是 | 全局 | 是 |
| authentication_ldap_simple_max_pool_size | 是 | 是 | 是 | 全局 | 是 |
| authentication_ldap_simple_referral | 是 | 是 | 是 | 全局 | 是 |
| authentication_ldap_simple_server_host | 是 | 是 | 是 | 全局 | 是 |
| authentication_ldap_simple_server_port | 是 | 是 | 是 | 全局 | 是 |
| authentication_ldap_simple_tls | 是 | 是 | 是 | 全局 | 是 |
| authentication_ldap_simple_user_search_attr | 是 | 是 | 是 | 全局 | 是 |
| authentication_policy | 是 | 是 | 是 | 全局 | 是 |
| authentication_windows_log_level | 是 | 是 | 是 | 全局 | 否 |
| authentication_windows_use_principal_name | 是 | 是 | 是 | 全局 | 否 |
| auto_generate_certs | 是 | 是 | 是 | 全局 | 否 |
| auto_increment_increment | 是 | 是 | 是 | 两者 | 是 |
| auto_increment_offset | 是 | 是 | 是 | 两者 | 是 |
| autocommit | 是 | 是 | 是 | 两者 | 是 |
| automatic_sp_privileges | 是 | 是 | 是 | 全局 | 是 |
| avoid_temporal_upgrade | 是 | 是 | 是 | 全局 | 是 |
| back_log | 是 | 是 | 是 | 全局 | 否 |
| basedir | 是 | 是 | 是 | 全局 | 否 |
| big_tables | 是 | 是 | 是 | 两者 | 是 |
| bind_address | 是 | 是 | 是 | 全局 | 否 |
| binlog_cache_size | 是 | 是 | 是 | 全局 | 是 |
| binlog_checksum | 是 | 是 | 是 | 全局 | 是 |
| binlog_direct_non_transactional_updates | 是 | 是 | 是 | 两者 | 是 |
| binlog_encryption | 是 | 是 | 是 | 全局 | 是 |
| binlog_error_action | 是 | 是 | 是 | 全局 | 是 |
| binlog_expire_logs_auto_purge | 是 | 是 | 是 | 全局 | 是 |
| binlog_expire_logs_seconds | 是 | 是 | 是 | 全局 | 是 |
| binlog_format | 是 | 是 | 是 | 两者 | 是 |
| binlog_group_commit_sync_delay | 是 | 是 | 是 | 全局 | 是 |
| binlog_group_commit_sync_no_delay_count | 是 | 是 | 是 | 全局 | 是 |
| binlog_gtid_simple_recovery | 是 | 是 | 是 | 全局 | 否 |
| binlog_max_flush_queue_time | 是 | 是 | 是 | 全局 | 是 |
| binlog_order_commits | 是 | 是 | 是 | 全局 | 是 |
| binlog_rotate_encryption_master_key_at_startup | 是 | 是 | 是 | 全局 | 否 |
| binlog_row_event_max_size | 是 | 是 | 是 | 全局 | 否 |
| binlog_row_image | 是 | 是 | 是 | 两者 | 是 |
| binlog_row_metadata | 是 | 是 | 是 | 全局 | 是 |
| binlog_row_value_options | 是 | 是 | 是 | 两者 | 是 |
| binlog_rows_query_log_events | 是 | 是 | 是 | 两者 | 是 |
| binlog_stmt_cache_size | 是 | 是 | 是 | 全局 | 是 |
| binlog_transaction_compression | 是 | 是 | 是 | 两者 | 是 |
| binlog_transaction_compression_level_zstd | 是 | 是 | 是 | 两者 | 是 |
| binlog_transaction_dependency_history_size | 是 | 是 | 是 | 全局 | 是 |
| binlog_transaction_dependency_tracking | 是 | 是 | 是 | 全局 | 是 |
| block_encryption_mode | 是 | 是 | 是 | 两者 | 是 |
| build_id |  |  | 是 | 全局 | 否 |
| bulk_insert_buffer_size | 是 | 是 | 是 | 两者 | 是 |
| caching_sha2_password_auto_generate_rsa_keys | 是 | 是 | 是 | 全局 | 否 |
| caching_sha2_password_digest_rounds | 是 | 是 | 是 | 全局 | 否 |
| caching_sha2_password_private_key_path | 是 | 是 | 是 | 全局 | 否 |
| caching_sha2_password_public_key_path | 是 | 是 | 是 | 全局 | 否 |
| character_set_client |  |  | 是 | 两者 | 是 |
| character_set_connection |  |  | 是 | 两者 | 是 |
| character_set_database (注 1) |  |  | 是 | 两者 | 是 |
| character_set_filesystem | 是 | 是 | 是 | 两者 | 是 |
| character_set_results |  |  | 是 | 两者 | 是 |
| character_set_server | 是 | 是 | 是 | 两者 | 是 |
| character_set_system |  |  | 是 | 全局 | 否 |
| character_sets_dir | 是 | 是 | 是 | 全局 | 否 |
| check_proxy_users | 是 | 是 | 是 | 全局 | 是 |
| 克隆自动调谐并发性 | 是 | 是 | 是 | 全局 | 是 |
| 克隆阻止 DDL | 是 | 是 | 是 | 全局 | 是 |
| 克隆缓冲区大小 | 是 | 是 | 是 | 全局 | 是 |
| 克隆 DDL 超时 | 是 | 是 | 是 | 全局 | 是 |
| 删除数据后延迟 | 是 | 是 | 是 | 全局 | 是 |
| 网络故障后捐赠者超时 | 是 | 是 | 是 | 全局 | 是 |
| 克隆启用压缩 | 是 | 是 | 是 | 全局 | 是 |
| 克隆最大并发数 | 是 | 是 | 是 | 全局 | 是 |
| 克隆最大数据带宽 | 是 | 是 | 是 | 全局 | 是 |
| 克隆最大网络带宽 | 是 | 是 | 是 | 全局 | 是 |
| 克隆 SSL CA | 是 | 是 | 是 | 全局 | 是 |
| 克隆 SSL 证书 | 是 | 是 | 是 | 全局 | 是 |
| 克隆 SSL 密钥 | 是 | 是 | 是 | 全局 | 是 |
| 克隆有效捐赠者列表 | 是 | 是 | 是 | 全局 | 是 |
| 连接排序规则 |  |  | 是 | 两者 | 是 |
| 数据库排序规则（注 1） |  |  | 是 | 两者 | 是 |
| 服务器排序规则 | 是 | 是 | 是 | 两者 | 是 |
| 完成类型 | 是 | 是 | 是 | 两者 | 是 |
| 组件调度程序启用 | 是 | 是 | 是 | 全局 | 是 |
| 并发插入 | 是 | 是 | 是 | 全局 | 是 |
| 连接超时 | 是 | 是 | 是 | 全局 | 是 |
| 连接控制失败连接阈值 | 是 | 是 | 是 | 全局 | 是 |
| 连接控制最大连接延迟 | 是 | 是 | 是 | 全局 | 是 |
| connection_control_min_connection_delay | 是 | 是 | 是 | 全局 | 是 |
| connection_memory_chunk_size | 是 | 是 | 是 | 两者 | 是 |
| connection_memory_limit | 是 | 是 | 是 | 两者 | 是 |
| core_file |  |  | 是 | 全局 | 否 |
| create_admin_listener_thread | 是 | 是 | 是 | 全局 | 否 |
| cte_max_recursion_depth | 是 | 是 | 是 | 两者 | 是 |
| daemon_memcached_enable_binlog | 是 | 是 | 是 | 全局 | 否 |
| daemon_memcached_engine_lib_name | 是 | 是 | 是 | 全局 | 否 |
| daemon_memcached_engine_lib_path | 是 | 是 | 是 | 全局 | 否 |
| daemon_memcached_option | 是 | 是 | 是 | 全局 | 否 |
| daemon_memcached_r_batch_size | 是 | 是 | 是 | 全局 | 否 |
| daemon_memcached_w_batch_size | 是 | 是 | 是 | 全局 | 否 |
| datadir | 是 | 是 | 是 | 全局 | 否 |
| debug | 是 | 是 | 是 | 两者 | 是 |
| debug_sync |  |  | 是 | 会话 | 是 |
| default_authentication_plugin | 是 | 是 | 是 | 全局 | 否 |
| default_collation_for_utf8mb4 |  |  | 是 | 两者 | 是 |
| default_password_lifetime | 是 | 是 | 是 | 全局 | 是 |
| default_storage_engine | 是 | 是 | 是 | 两者 | 是 |
| default_table_encryption | 是 | 是 | 是 | 两者 | 是 |
| default_tmp_storage_engine | 是 | 是 | 是 | 两者 | 是 |
| default_week_format | 是 | 是 | 是 | 两者 | 是 |
| delay_key_write | 是 | 是 | 是 | 全局 | 是 |
| delayed_insert_limit | 是 | 是 | 是 | 全局 | 是 |
| delayed_insert_timeout | 是 | 是 | 是 | 全局 | 是 |
| delayed_queue_size | 是 | 是 | 是 | 全局 | 是 |
| disabled_storage_engines | 是 | 是 | 是 | 全局 | 否 |
| disconnect_on_expired_password | 是 | 是 | 是 | 全局 | 否 |
| div_precision_increment | 是 | 是 | 是 | 两者 | 是 |
| dragnet.log_error_filter_rules | 是 | 是 | 是 | 全局 | 是 |
| end_markers_in_json | 是 | 是 | 是 | 两者 | 是 |
| enforce_gtid_consistency | 是 | 是 | 是 | 全局 | 是 |
| enterprise_encryption.maximum_rsa_key_size | 是 | 是 | 是 | 全局 | 是 |
| enterprise_encryption.rsa_support_legacy_padding | 是 | 是 | 是 | 全局 | 是 |
| eq_range_index_dive_limit | 是 | 是 | 是 | 两者 | 是 |
| error_count |  |  | 是 | 会话 | 否 |
| event_scheduler | 是 | 是 | 是 | 全局 | 是 |
| expire_logs_days | 是 | 是 | 是 | 全局 | 是 |
| explain_format | 是 | 是 | 是 | 两者 | 是 |
| explicit_defaults_for_timestamp | 是 | 是 | 是 | 两者 | 是 |
| external_user |  |  | 是 | 会话 | 否 |
| flush | 是 | 是 | 是 | 全局 | 是 |
| flush_time | 是 | 是 | 是 | 全局 | 是 |
| foreign_key_checks |  |  | 是 | 两者 | 是 |
| ft_boolean_syntax | 是 | 是 | 是 | 全局 | 是 |
| ft_max_word_len | 是 | 是 | 是 | 全局 | 否 |
| ft_min_word_len | 是 | 是 | 是 | 全局 | 否 |
| ft_query_expansion_limit | 是 | 是 | 是 | 全局 | 否 |
| ft_stopword_file | 是 | 是 | 是 | 全局 | 否 |
| general_log | 是 | 是 | 是 | 全局 | 是 |
| general_log_file | 是 | 是 | 是 | 全局 | 是 |
| generated_random_password_length | 是 | 是 | 是 | 两者 | 是 |
| global_connection_memory_limit | 是 | 是 | 是 | 全局 | 是 |
| global_connection_memory_tracking | 是 | 是 | 是 | 两者 | 是 |
| group_concat_max_len | 是 | 是 | 是 | 两者 | 是 |
| group_replication_advertise_recovery_endpoints | 是 | 是 | 是 | 全局 | 是 |
| group_replication_allow_local_lower_version_join | 是 | 是 | 是 | 全局 | 是 |
| group_replication_auto_increment_increment | 是 | 是 | 是 | 全局 | 是 |
| group_replication_autorejoin_tries | 是 | 是 | 是 | 全局 | 是 |
| group_replication_bootstrap_group | 是 | 是 | 是 | 全局 | 是 |
| group_replication_clone_threshold | 是 | 是 | 是 | 全局 | 是 |
| group_replication_communication_debug_options | 是 | 是 | 是 | 全局 | 是 |
| group_replication_communication_max_message_size | 是 | 是 | 是 | 全局 | 是 |
| group_replication_communication_stack |  |  | 是 | 全局 | 否 |
| group_replication_components_stop_timeout | 是 | 是 | 是 | 全局 | 是 |
| group_replication_compression_threshold | 是 | 是 | 是 | 全局 | 是 |
| group_replication_consistency | 是 | 是 | 是 | 两者 | 是 |
| group_replication_enforce_update_everywhere_checks | 是 | 是 | 是 | 全局 | 是 |
| group_replication_exit_state_action | 是 | 是 | 是 | 全局 | 是 |
| group_replication_flow_control_applier_threshold | 是 | 是 | 是 | 全局 | 是 |
| group_replication_flow_control_certifier_threshold | 是 | 是 | 是 | 全局 | 是 |
| group_replication_flow_control_hold_percent | 是 | 是 | 是 | 全局 | 是 |
| group_replication_flow_control_max_quota | 是 | 是 | 是 | 全局 | 是 |
| group_replication_flow_control_member_quota_percent | 是 | 是 | 是 | 全局 | 是 |
| group_replication_flow_control_min_quota | 是 | 是 | 是 | 全局 | 是 |
| group_replication_flow_control_min_recovery_quota | 是 | 是 | 是 | 全局 | 是 |
| group_replication_flow_control_mode | 是 | 是 | 是 | 全局 | 是 |
| group_replication_flow_control_period | 是 | 是 | 是 | 全局 | 是 |
| group_replication_flow_control_release_percent | 是 | 是 | 是 | 全局 | 是 |
| group_replication_force_members | 是 | 是 | 是 | 全局 | 是 |
| group_replication_group_name | 是 | 是 | 是 | 全局 | 是 |
| group_replication_group_seeds | 是 | 是 | 是 | 全局 | 是 |
| group_replication_gtid_assignment_block_size | 是 | 是 | 是 | 全局 | 是 |
| group_replication_ip_allowlist | 是 | 是 | 是 | 全局 | 是 |
| group_replication_ip_whitelist | 是 | 是 | 是 | 全局 | 是 |
| group_replication_local_address | 是 | 是 | 是 | 全局 | 是 |
| group_replication_member_expel_timeout | 是 | 是 | 是 | 全局 | 是 |
| group_replication_member_weight | 是 | 是 | 是 | 全局 | 是 |
| group_replication_message_cache_size | 是 | 是 | 是 | 全局 | 是 |
| group_replication_paxos_single_leader | 是 | 是 | 是 | 全局 | 是 |
| group_replication_poll_spin_loops | 是 | 是 | 是 | 全局 | 是 |
| group_replication_recovery_complete_at | 是 | 是 | 是 | 全局 | 是 |
| group_replication_recovery_compression_algorithms | 是 | 是 | 是 | 全局 | 是 |
| group_replication_recovery_get_public_key | 是 | 是 | 是 | 全局 | 是 |
| group_replication_recovery_public_key_path | 是 | 是 | 是 | 全局 | 是 |
| group_replication_recovery_reconnect_interval | 是 | 是 | 是 | 全局 | 是 |
| group_replication_recovery_retry_count | 是 | 是 | 是 | 全局 | 是 |
| group_replication_recovery_ssl_ca | 是 | 是 | 是 | 全局 | 是 |
| group_replication_recovery_ssl_capath | 是 | 是 | 是 | 全局 | 是 |
| group_replication_recovery_ssl_cert | 是 | 是 | 是 | 全局 | 是 |
| group_replication_recovery_ssl_cipher | 是 | 是 | 是 | 全局 | 是 |
| group_replication_recovery_ssl_crl | 是 | 是 | 是 | 全局 | 是 |
| group_replication_recovery_ssl_crlpath | 是 | 是 | 是 | 全局 | 是 |
| group_replication_recovery_ssl_key | 是 | 是 | 是 | 全局 | 是 |
| group_replication_recovery_ssl_verify_server_cert | 是 | 是 | 是 | 全局 | 是 |
| group_replication_recovery_tls_ciphersuites | 是 | 是 | 是 | 全局 | 是 |
| group_replication_recovery_tls_version | 是 | 是 | 是 | 全局 | 是 |
| group_replication_recovery_use_ssl | 是 | 是 | 是 | 全局 | 是 |
| group_replication_recovery_zstd_compression_level | 是 | 是 | 是 | 全局 | 是 |
| group_replication_single_primary_mode | 是 | 是 | 是 | 全局 | 是 |
| group_replication_ssl_mode | 是 | 是 | 是 | 全局 | 是 |
| group_replication_start_on_boot | 是 | 是 | 是 | 全局 | 是 |
| group_replication_tls_source | 是 | 是 | 是 | 全局 | 是 |
| group_replication_transaction_size_limit | 是 | 是 | 是 | 全局 | 是 |
| group_replication_unreachable_majority_timeout | 是 | 是 | 是 | 全局 | 是 |
| group_replication_view_change_uuid | 是 | 是 | 是 | 全局 | 是 |
| gtid_executed |  |  | 是 | 全局 | 否 |
| gtid_executed_compression_period | 是 | 是 | 是 | 全局 | 是 |
| gtid_mode | 是 | 是 | 是 | 全局 | 是 |
| gtid_next |  |  | 是 | 会话 | 是 |
| gtid_owned |  |  | 是 | 两者 | 否 |
| gtid_purged |  |  | 是 | 全局 | 是 |
| have_compress |  |  | 是 | 全局 | 否 |
| have_dynamic_loading |  |  | 是 | 全局 | 否 |
| have_geometry |  |  | 是 | 全局 | 否 |
| have_openssl |  |  | 是 | 全局 | 否 |
| have_profiling |  |  | 是 | 全局 | 否 |
| have_query_cache |  |  | 是 | 全局 | 否 |
| have_rtree_keys |  |  | 是 | 全局 | 否 |
| have_ssl |  |  | 是 | 全局 | 否 |
| have_statement_timeout |  |  | 是 | 全局 | 否 |
| have_symlink |  |  | 是 | 全局 | 否 |
| 直方图生成最大内存大小 | 是 | 是 | 是 | 两者 | 是 |
| 主机缓存大小 | 是 | 是 | 是 | 全局 | 是 |
| 主机名 |  |  | 是 | 全局 | 否 |
| identity |  |  | 是 | 会话 | 是 |
| immediate_server_version |  |  | 是 | 会话 | 是 |
| information_schema_stats_expiry | 是 | 是 | 是 | 两者 | 是 |
| init_connect | 是 | 是 | 是 | 全局 | 是 |
| init_file | 是 | 是 | 是 | 全局 | 否 |
| init_replica | 是 | 是 | 是 | 全局 | 是 |
| init_slave | 是 | 是 | 是 | 全局 | 是 |
| innodb 自适应刷新 | 是 | 是 | 是 | 全局 | 是 |
| innodb 自适应刷新 lwm | 是 | 是 | 是 | 全局 | 是 |
| innodb 自适应哈希索引 | 是 | 是 | 是 | 全局 | 是 |
| innodb 自适应哈希索引部分 | 是 | 是 | 是 | 全局 | 否 |
| innodb 自适应最大睡眠延迟 | 是 | 是 | 是 | 全局 | 是 |
| innodb_api_bk_commit_interval | 是 | 是 | 是 | 全局 | 是 |
| innodb_api_disable_rowlock | 是 | 是 | 是 | 全局 | 否 |
| innodb_api_enable_binlog | 是 | 是 | 是 | 全局 | 否 |
| innodb_api_enable_mdl | 是 | 是 | 是 | 全局 | 否 |
| innodb_api_trx_level | 是 | 是 | 是 | 全局 | 是 |
| innodb_autoextend_increment | 是 | 是 | 是 | 全局 | 是 |
| innodb_autoinc_lock_mode | 是 | 是 | 是 | 全局 | 否 |
| innodb_background_drop_list_empty | 是 | 是 | 是 | 全局 | 是 |
| innodb_buffer_pool_chunk_size | 是 | 是 | 是 | 全局 | 否 |
| innodb_buffer_pool_debug | 是 | 是 | 是 | 全局 | 否 |
| innodb_buffer_pool_dump_at_shutdown | 是 | 是 | 是 | 全局 | 是 |
| innodb_buffer_pool_dump_now | 是 | 是 | 是 | 全局 | 是 |
| innodb_buffer_pool_dump_pct | 是 | 是 | 是 | 全局 | 是 |
| innodb_buffer_pool_filename | 是 | 是 | 是 | 全局 | 是 |
| innodb_buffer_pool_in_core_file | 是 | 是 | 是 | 全局 | 是 |
| innodb_buffer_pool_instances | 是 | 是 | 是 | 全局 | 否 |
| innodb_buffer_pool_load_abort | 是 | 是 | 是 | 全局 | 是 |
| innodb_buffer_pool_load_at_startup | 是 | 是 | 是 | 全局 | 否 |
| innodb_buffer_pool_load_now | 是 | 是 | 是 | 全局 | 是 |
| innodb_buffer_pool_size | 是 | 是 | 是 | 全局 | 是 |
| innodb_change_buffer_max_size | 是 | 是 | 是 | 全局 | 是 |
| innodb_change_buffering | 是 | 是 | 是 | 全局 | 是 |
| innodb_change_buffering_debug | 是 | 是 | 是 | 全局 | 是 |
| innodb_checkpoint_disabled | 是 | 是 | 是 | 全局 | 是 |
| innodb_checksum_algorithm | 是 | 是 | 是 | 全局 | 是 |
| innodb_cmp_per_index_enabled | 是 | 是 | 是 | 全局 | 是 |
| innodb_commit_concurrency | 是 | 是 | 是 | 全局 | 是 |
| innodb_compress_debug | 是 | 是 | 是 | 全局 | 是 |
| innodb_compression_failure_threshold_pct | 是 | 是 | 是 | 全局 | 是 |
| innodb_compression_level | 是 | 是 | 是 | 全局 | 是 |
| innodb_compression_pad_pct_max | 是 | 是 | 是 | 全局 | 是 |
| innodb_concurrency_tickets | 是 | 是 | 是 | 全局 | 是 |
| innodb_data_file_path | 是 | 是 | 是 | 全局 | 否 |
| innodb_data_home_dir | 是 | 是 | 是 | 全局 | 否 |
| innodb_ddl_buffer_size | 是 | 是 | 是 | 两者 | 是 |
| innodb_ddl_log_crash_reset_debug | 是 | 是 | 是 | 全局 | 是 |
| innodb_ddl_threads | 是 | 是 | 是 | 两者 | 是 |
| innodb_deadlock_detect | 是 | 是 | 是 | 全局 | 是 |
| innodb_dedicated_server | 是 | 是 | 是 | 全局 | 否 |
| innodb_default_row_format | 是 | 是 | 是 | 全局 | 是 |
| innodb_directories | 是 | 是 | 是 | 全局 | 否 |
| innodb_disable_sort_file_cache | 是 | 是 | 是 | 全局 | 是 |
| innodb_doublewrite | 是 | 是 | 是 | 全局 | 不定 |
| innodb_doublewrite_batch_size | 是 | 是 | 是 | 全局 | 否 |
| innodb_doublewrite_dir | 是 | 是 | 是 | 全局 | 否 |
| innodb_doublewrite_files | 是 | 是 | 是 | 全局 | 否 |
| innodb_doublewrite_pages | 是 | 是 | 是 | 全局 | 否 |
| innodb_extend_and_initialize | 是 | 是 | 是 | 全局 | 是 |
| innodb_fast_shutdown | 是 | 是 | 是 | 全局 | 是 |
| innodb_fil_make_page_dirty_debug | 是 | 是 | 是 | 全局 | 是 |
| innodb_file_per_table | 是 | 是 | 是 | 全局 | 是 |
| innodb_fill_factor | 是 | 是 | 是 | 全局 | 是 |
| innodb_flush_log_at_timeout | 是 | 是 | 是 | 全局 | 是 |
| innodb_flush_log_at_trx_commit | 是 | 是 | 是 | 全局 | 是 |
| innodb_flush_method | 是 | 是 | 是 | 全局 | 否 |
| innodb_flush_neighbors | 是 | 是 | 是 | 全局 | 是 |
| innodb_flush_sync | 是 | 是 | 是 | 全局 | 是 |
| innodb_flushing_avg_loops | 是 | 是 | 是 | 全局 | 是 |
| innodb_force_load_corrupted | 是 | 是 | 是 | 全局 | 否 |
| innodb_force_recovery | 是 | 是 | 是 | 全局 | 否 |
| innodb_fsync_threshold | 是 | 是 | 是 | 全局 | 是 |
| innodb_ft_aux_table |  |  | 是 | 全局 | 是 |
| innodb_ft_cache_size | 是 | 是 | 是 | 全局 | 否 |
| innodb_ft_enable_diag_print | 是 | 是 | 是 | 全局 | 是 |
| innodb_ft_enable_stopword | 是 | 是 | 是 | 全局 | 是 |
| innodb_ft_max_token_size | 是 | 是 | 是 | 全局 | 否 |
| innodb_ft_min_token_size | 是 | 是 | 是 | 全局 | 否 |
| innodb_ft_num_word_optimize | 是 | 是 | 是 | 全局 | 是 |
| innodb_ft_result_cache_limit | 是 | 是 | 是 | 全局 | 是 |
| innodb_ft_server_stopword_table | 是 | 是 | 是 | 全局 | 是 |
| innodb_ft_sort_pll_degree | 是 | 是 | 是 | 全局 | 否 |
| innodb_ft_total_cache_size | 是 | 是 | 是 | 全局 | 否 |
| innodb_ft_user_stopword_table | 是 | 是 | 是 | 全局 | 是 |
| innodb_idle_flush_pct | 是 | 是 | 是 | 全局 | 是 |
| innodb_io_capacity | 是 | 是 | 是 | 全局 | 是 |
| innodb_io_capacity_max | 是 | 是 | 是 | 全局 | 是 |
| innodb_limit_optimistic_insert_debug | 是 | 是 | 是 | 全局 | 是 |
| innodb_lock_wait_timeout | 是 | 是 | 是 | 两者 | 是 |
| innodb_log_buffer_size | 是 | 是 | 是 | 全局 | 不同 |
| innodb_log_checkpoint_fuzzy_now | 是 | 是 | 是 | 全局 | 是 |
| innodb_log_checkpoint_now | 是 | 是 | 是 | 全局 | 是 |
| innodb_log_checksums | 是 | 是 | 是 | 全局 | 是 |
| innodb_log_compressed_pages | 是 | 是 | 是 | 全局 | 是 |
| innodb_log_file_size | 是 | 是 | 是 | 全局 | 否 |
| innodb_log_files_in_group | 是 | 是 | 是 | 全局 | 否 |
| innodb_log_group_home_dir | 是 | 是 | ��� | 全局 | 否 |
| innodb_log_spin_cpu_abs_lwm | 是 | 是 | 是 | 全局 | 是 |
| innodb_log_spin_cpu_pct_hwm | 是 | 是 | 是 | 全局 | 是 |
| innodb_log_wait_for_flush_spin_hwm | 是 | 是 | 是 | 全局 | 是 |
| innodb_log_write_ahead_size | 是 | 是 | 是 | 全局 | 是 |
| innodb_log_writer_threads | 是 | 是 | 是 | 全局 | 是 |
| innodb_lru_scan_depth | 是 | 是 | 是 | 全局 | 是 |
| innodb_max_dirty_pages_pct | 是 | 是 | 是 | 全局 | 是 |
| innodb_max_dirty_pages_pct_lwm | 是 | 是 | 是 | 全局 | 是 |
| innodb_max_purge_lag | 是 | 是 | 是 | 全局 | 是 |
| innodb_max_purge_lag_delay | 是 | 是 | 是 | 全局 | 是 |
| innodb_max_undo_log_size | 是 | 是 | 是 | 全局 | 是 |
| innodb_merge_threshold_set_all_debug | 是 | 是 | 是 | 全局 | 是 |
| innodb_monitor_disable | 是 | 是 | 是 | 全局 | 是 |
| innodb_monitor_enable | 是 | 是 | 是 | 全局 | 是 |
| innodb_monitor_reset | 是 | 是 | 是 | 全局 | 是 |
| innodb_monitor_reset_all | 是 | 是 | 是 | 全局 | 是 |
| innodb_numa_interleave | 是 | 是 | 是 | 全局 | 否 |
| innodb_old_blocks_pct | 是 | 是 | 是 | 全局 | 是 |
| innodb_old_blocks_time | 是 | 是 | 是 | 全局 | 是 |
| innodb_online_alter_log_max_size | 是 | 是 | 是 | 全局 | 是 |
| innodb_open_files | 是 | 是 | 是 | 全局 | 不定 |
| innodb_optimize_fulltext_only | 是 | 是 | 是 | 全局 | 是 |
| innodb_page_cleaners | 是 | 是 | 是 | 全局 | 否 |
| innodb_page_size | 是 | 是 | 是 | 全局 | 否 |
| innodb_parallel_read_threads | 是 | 是 | 是 | 会话 | 是 |
| innodb_print_all_deadlocks | 是 | 是 | 是 | 全局 | 是 |
| innodb_print_ddl_logs | 是 | 是 | 是 | 全局 | 是 |
| innodb_purge_batch_size | 是 | 是 | 是 | 全局 | 是 |
| innodb_purge_rseg_truncate_frequency | 是 | 是 | 是 | 全局 | 是 |
| innodb_purge_threads | 是 | 是 | 是 | 全局 | 否 |
| innodb_random_read_ahead | 是 | 是 | 是 | 全局 | 是 |
| innodb_read_ahead_threshold | 是 | 是 | 是 | 全局 | 是 |
| innodb_read_io_threads | 是 | 是 | 是 | 全局 | 否 |
| innodb_read_only | 是 | �� | 是 | 全局 | 否 |
| innodb_redo_log_archive_dirs | 是 | 是 | 是 | 全局 | 是 |
| innodb_redo_log_capacity | 是 | 是 | 是 | 全局 | 是 |
| innodb_redo_log_encrypt | 是 | 是 | 是 | 全局 | 是 |
| innodb_replication_delay | 是 | 是 | 是 | 全局 | 是 |
| innodb_rollback_on_timeout | 是 | 是 | 是 | 全局 | 否 |
| innodb_rollback_segments | 是 | 是 | 是 | 全局 | 是 |
| innodb_saved_page_number_debug | 是 | 是 | 是 | 全局 | 是 |
| innodb_segment_reserve_factor | 是 | 是 | 是 | 全局 | 是 |
| innodb_sort_buffer_size | 是 | 是 | 是 | 全局 | 否 |
| innodb_spin_wait_delay | 是 | 是 | 是 | 全局 | 是 |
| innodb_spin_wait_pause_multiplier | 是 | 是 | 是 | 全局 | 是 |
| innodb_stats_auto_recalc | 是 | 是 | 是 | 全局 | 是 |
| innodb_stats_include_delete_marked | 是 | 是 | 是 | 全局 | 是 |
| innodb_stats_method | 是 | 是 | 是 | 全局 | 是 |
| innodb_stats_on_metadata | 是 | 是 | 是 | 全局 | 是 |
| innodb_stats_persistent | 是 | 是 | 是 | 全局 | 是 |
| innodb_stats_persistent_sample_pages | 是 | 是 | 是 | 全局 | 是 |
| innodb_stats_transient_sample_pages | 是 | 是 | 是 | 全局 | 是 |
| innodb_status_output | 是 | 是 | 是 | 全局 | 是 |
| innodb_status_output_locks | 是 | 是 | 是 | 全局 | 是 |
| innodb_strict_mode | 是 | 是 | 是 | 两者 | 是 |
| innodb_sync_array_size | 是 | 是 | 是 | 全局 | 否 |
| innodb_sync_debug | 是 | 是 | 是 | 全局 | 否 |
| innodb_sync_spin_loops | 是 | 是 | 是 | 全局 | 是 |
| innodb_table_locks | 是 | 是 | 是 | 两者 | 是 |
| innodb_temp_data_file_path | 是 | 是 | 是 | 全局 | 否 |
| innodb_temp_tablespaces_dir | 是 | 是 | 是 | 全局 | 否 |
| innodb_thread_concurrency | 是 | 是 | 是 | 全局 | 是 |
| innodb_thread_sleep_delay | 是 | 是 | 是 | 全局 | 是 |
| innodb_tmpdir | 是 | 是 | 是 | 两者 | 是 |
| innodb_trx_purge_view_update_only_debug | 是 | 是 | 是 | 全局 | 是 |
| innodb_trx_rseg_n_slots_debug | 是 | 是 | 是 | 全局 | 是 |
| innodb_undo_directory | 是 | 是 | 是 | 全局 | 否 |
| innodb_undo_log_encrypt | 是 | 是 | 是 | 全局 | 是 |
| innodb_undo_log_truncate | 是 | 是 | 是 | 全局 | 是 |
| innodb_undo_tablespaces | 是 | 是 | 是 | 全局 | 不同 |
| innodb_use_fdatasync | 是 | 是 | 是 | 全局 | 是 |
| innodb_use_native_aio | 是 | 是 | 是 | 全局 | 否 |
| innodb_validate_tablespace_paths | 是 | 是 | 是 | 全局 | 否 |
| innodb_version |  |  | 是 | 全局 | 否 |
| innodb_write_io_threads | 是 | 是 | 是 | 全局 | 否 |
| insert_id |  |  | 是 | 会话 | 是 |
| interactive_timeout | 是 | 是 | 是 | 两者 | 是 |
| internal_tmp_disk_storage_engine | 是 | 是 | 是 | 全局 | 是 |
| internal_tmp_mem_storage_engine | 是 | 是 | 是 | 两者 | 是 |
| join_buffer_size | 是 | 是 | 是 | 两者 | 是 |
| keep_files_on_create | 是 | 是 | 是 | 两者 | 是 |
| key_buffer_size | 是 | 是 | 是 | 全局 | 是 |
| key_cache_age_threshold | 是 | 是 | 是 | 全局 | 是 |
| key_cache_block_size | 是 | 是 | 是 | 全局 | 是 |
| key_cache_division_limit | 是 | 是 | 是 | 全局 | 是 |
| keyring_aws_cmk_id | 是 | 是 | 是 | 全局 | 是 |
| keyring_aws_conf_file | 是 | 是 | 是 | 全局 | 否 |
| keyring_aws_data_file | 是 | 是 | 是 | 全局 | 否 |
| AWS 地区密钥环 | 是 | 是 | 是 | 全局 | 是 |
| 加密文件数据 | 是 | 是 | 是 | 全局 | 是 |
| 加密文件密码 | 是 | 是 | 是 | 全局 | 是 |
| 文件数据密钥环 | 是 | 是 | 是 | 全局 | 是 |
| 密钥环哈希图认证路径 | 是 | 是 | 是 | 全局 | 是 |
| 密钥环哈希图 CA 路径 | 是 | 是 | 是 | 全局 | 是 |
| 密钥环哈希图缓存 | 是 | 是 | 是 | 全局 | 是 |
| 密钥环哈希图提交认证路径 |  |  | 是 | 全局 | 否 |
| 密钥环哈希图提交 CA 路径 |  |  | 是 | 全局 | 否 |
| 密钥环哈希图提交缓存 |  |  | 是 | 全局 | 否 |
| 密钥环哈希图提交角色 ID |  |  | 是 | 全局 | 否 |
| 密钥环哈希图提交服务器 URL |  |  | 是 | 全局 | 否 |
| 密钥环哈希图存储路径 |  |  | 是 | 全局 | 否 |
| 密钥环哈希图角色 ID | 是 | 是 | 是 | 全局 | 是 |
| 密钥环哈希图秘密 ID | 是 | 是 | 是 | 全局 | 是 |
| 密钥环哈希图服务器 URL | 是 | 是 | 是 | 全局 | 是 |
| 密钥环哈希图存储路径 | 是 | 是 | 是 | 全局 | 是 |
| OCI CA 证书 | 是 | 是 | 是 | 全局 | 否 |
| OCI 区域 | 是 | 是 | 是 | 全局 | 否 |
| OCI 加密端点 | 是 | 是 | 是 | 全局 | 否 |
| OCI 密钥文件 | 是 | 是 | 是 | 全局 | 否 |
| OCI 密钥指纹 | 是 | 是 | 是 | 全局 | 否 |
| keyring_oci_management_endpoint | 是 | 是 | 是 | 全局 | 否 |
| keyring_oci_master_key | 是 | 是 | 是 | 全局 | 否 |
| keyring_oci_secrets_endpoint | 是 | 是 | 是 | 全局 | 否 |
| keyring_oci_tenancy | 是 | 是 | 是 | 全局 | 否 |
| keyring_oci_user | 是 | 是 | 是 | 全局 | 否 |
| keyring_oci_vaults_endpoint | 是 | �� | 是 | 全局 | 否 |
| keyring_oci_virtual_vault | 是 | 是 | 是 | 全局 | 否 |
| keyring_okv_conf_dir | 是 | 是 | 是 | 全局 | 是 |
| keyring_operations |  |  | 是 | 全局 | 是 |
| large_files_support |  |  | 是 | 全局 | 否 |
| large_page_size |  |  | 是 | 全局 | 否 |
| large_pages | 是 | 是 | 是 | 全局 | 否 |
| last_insert_id |  |  | 是 | 会话 | 是 |
| lc_messages | 是 | 是 | 是 | 两者 | 是 |
| lc_messages_dir | 是 | 是 | 是 | 全局 | 否 |
| lc_time_names | 是 | 是 | 是 | 两者 | 是 |
| license |  |  | 是 | 全局 | 否 |
| local_infile | 是 | 是 | 是 | 全局 | 是 |
| lock_order | 是 | 是 | 是 | 全局 | 否 |
| lock_order_debug_loop | 是 | 是 | 是 | 全局 | 否 |
| lock_order_debug_missing_arc | 是 | 是 | 是 | 全局 | 否 |
| lock_order_debug_missing_key | 是 | 是 | 是 | 全局 | 否 |
| lock_order_debug_missing_unlock | 是 | 是 | 是 | 全局 | 否 |
| lock_order_dependencies | 是 | 是 | 是 | 全局 | 否 |
| lock_order_extra_dependencies | 是 | 是 | 是 | 全局 | 否 |
| lock_order_output_directory | 是 | 是 | 是 | 全局 | 否 |
| 锁定顺序打印 txt | 是 | 是 | 是 | 全局 | 否 |
| 锁定顺序跟踪循环 | 是 | 是 | 是 | 全局 | 否 |
| 锁定顺序跟踪丢失弧 | 是 | 是 | 是 | 全局 | 否 |
| 锁定顺序跟踪丢失键 | 是 | 是 | 是 | 全局 | 否 |
| 锁定顺序跟踪丢失解锁 | 是 | 是 | 是 | 全局 | 否 |
| 锁定等待超时 | 是 | 是 | 是 | 双方 | 是 |
| 内存中锁定 |  |  | 是 | 全局 | 否 |
| 记录二进制日志 |  |  | 是 | 全局 | 否 |
| 记录二进制日志基本名称 |  |  | 是 | 全局 | 否 |
| 记录二进制日志索引 | 是 | 是 | 是 | 全局 | 否 |
| 记录二进制日志信任函数创建者 | 是 | 是 | 是 | 全局 | 是 |
| 记录二进制日志使用 v1 行事件 | 是 | 是 | 是 | 全局 | 是 |
| 错误日志 | 是 | 是 | 是 | 全局 | 否 |
| 记录错误服务 | 是 | 是 | 是 | 全局 | 是 |
| 记录错误抑制列表 | 是 | 是 | 是 | 全局 | 是 |
| 记录错误详细程度 | 是 | 是 | 是 | 全局 | 是 |
| 记录输出 | 是 | 是 | 是 | 全局 | 是 |
| 记录未使用索引的查询 | 是 | 是 | 是 | 全局 | 是 |
| 记录原始 | 是 | 是 | 是 | 全局 | 是 |
| 记录副本更新 | 是 | 是 | 是 | 全局 | 否 |
| 记录从库更新 | 是 | 是 | 是 | 全局 | 否 |
| 记录慢管理员语句 | 是 | 是 | 是 | 全局 | 是 |
| 记录慢额外 | 是 | 是 | 是 | 全局 | 是 |
| 记录慢从库语句 | 是 | 是 | 是 | 全局 | 是 |
| 记录慢从库语句 | 是 | 是 | 是 | 全局 | 是 |
| 日志不安全的语句用于二进制日志 | 是 | 是 | 是 | 全局 | 是 |
| 日志系统日志 | 是 | 是 | 是 | 全局 | 是 |
| 日志系统日志设施 | 是 | 是 | 是 | 全局 | 是 |
| 日志系统日志包括 PID | 是 | 是 | 是 | 全局 | 是 |
| 日志系统日志标签 | 是 | 是 | 是 | 全局 | 是 |
| 记录未使用索引的查询的节流 | 是 | 是 | 是 | 全局 | 是 |
| 记录时间戳 | 是 | 是 | 是 | 全局 | 是 |
| 长查询时间 | 是 | 是 | 是 | 两者 | 是 |
| 低优先级更新 | 是 | 是 | 是 | 两者 | 是 |
| 小写文件系统 |  |  | 是 | 全局 | 否 |
| 小写表名 | 是 | 是 | 是 | 全局 | 否 |
| 强制角色 | 是 | 是 | 是 | 全局 | 是 |
| 主服务器信息存储库 | 是 | 是 | 是 | 全局 | 是 |
| 主服务器验证校验和 | 是 | 是 | 是 | 全局 | 是 |
| 最大允许数据包大小 | 是 | 是 | 是 | 两者 | 是 |
| 最大二进制日志缓存大小 | 是 | 是 | 是 | 全局 | 是 |
| 最大二进制日志大小 | 是 | 是 | 是 | 全局 | 是 |
| 最大二进制日志语句缓存大小 | 是 | 是 | 是 | 全局 | 是 |
| 最大连接错误数 | 是 | 是 | 是 | 全局 | 是 |
| 最大连接数 | 是 | 是 | 是 | 全局 | 是 |
| 最大延迟线程数 | 是 | 是 | 是 | 两者 | 是 |
| 最大摘要长度 | 是 | 是 | 是 | 全局 | 否 |
| 最大错误计数 | 是 | 是 | 是 | 两者 | 是 |
| 最大执行时间 | 是 | 是 | 是 | 两者 | 是 |
| 最大堆表大小 | 是 | 是 | 是 | 两者 | 是 |
| max_insert_delayed_threads |  |  | 是 | 全局 | 是 |
| max_join_size | 是 | 是 | 是 | 全局 | 是 |
| max_length_for_sort_data | 是 | 是 | 是 | 全局 | 是 |
| max_points_in_geometry | 是 | 是 | 是 | 全局 | 是 |
| max_prepared_stmt_count | 是 | 是 | 是 | 全局 | 是 |
| max_relay_log_size | 是 | 是 | 是 | 全局 | 是 |
| max_seeks_for_key | 是 | 是 | 是 | 全局 | 是 |
| max_sort_length | 是 | 是 | 是 | 全局 | 是 |
| max_sp_recursion_depth | 是 | 是 | 是 | 全局 | 是 |
| max_user_connections | 是 | 是 | 是 | 全局 | 是 |
| max_write_lock_count | 是 | 是 | 是 | 全局 | 是 |
| mecab_rc_file | 是 | 是 | 是 | 全局 | 否 |
| metadata_locks_cache_size | 是 | 是 | 是 | 全局 | 否 |
| metadata_locks_hash_instances | 是 | 是 | 是 | 全局 | 否 |
| min_examined_row_limit | 是 | 是 | 是 | 全局 | 是 |
| myisam_data_pointer_size | 是 | 是 | 是 | 全局 | 是 |
| myisam_max_sort_file_size | 是 | 是 | 是 | 全局 | 是 |
| myisam_mmap_size | 是 | 是 | 是 | 全局 | 否 |
| myisam_recover_options | 是 | 是 | 是 | 全局 | 否 |
| myisam_repair_threads | 是 | 是 | 是 | 全局 | 是 |
| myisam_sort_buffer_size | 是 | 是 | 是 | 全局 | 是 |
| myisam_stats_method | 是 | 是 | 是 | 全局 | 是 |
| myisam_use_mmap | 是 | 是 | 是 | 全局 | 是 |
| mysql_firewall_mode | 是 | 是 | 是 | 全局 | 是 |
| mysql_firewall_trace | 是 | 是 | 是 | 全局 | 是 |
| mysql_native_password_proxy_users | 是 | 是 | 是 | 全局 | 是 |
| mysqlx_bind_address | 是 | 是 | 是 | 全局 | 否 |
| mysqlx_compression_algorithms | 是 | 是 | 是 | 全局 | 是 |
| mysqlx_connect_timeout | 是 | 是 | 是 | 全局 | 是 |
| mysqlx_deflate_default_compression_level | 是 | 是 | 是 | 全局 | 是 |
| mysqlx_deflate_max_client_compression_level | 是 | 是 | 是 | 全局 | 是 |
| mysqlx_document_id_unique_prefix | 是 | 是 | 是 | 全局 | 是 |
| mysqlx_enable_hello_notice | 是 | 是 | 是 | 全局 | 是 |
| mysqlx_idle_worker_thread_timeout | 是 | 是 | 是 | 全局 | 是 |
| mysqlx_interactive_timeout | 是 | 是 | 是 | 全局 | 是 |
| mysqlx_lz4_default_compression_level | 是 | 是 | 是 | 全局 | 是 |
| mysqlx_lz4_max_client_compression_level | 是 | 是 | 是 | 全局 | 是 |
| mysqlx_max_allowed_packet | 是 | 是 | 是 | 全局 | 是 |
| mysqlx_max_connections | 是 | 是 | 是 | 全局 | 是 |
| mysqlx_min_worker_threads | 是 | 是 | 是 | 全局 | 是 |
| mysqlx_port | 是 | 是 | 是 | 全局 | 否 |
| mysqlx_port_open_timeout | 是 | 是 | 是 | 全局 | 否 |
| mysqlx_read_timeout | 是 | 是 | 是 | 会话 | 是 |
| mysqlx_socket | 是 | 是 | 是 | 全局 | 否 |
| mysqlx_ssl_ca | 是 | 是 | 是 | 全局 | 否 |
| mysqlx_ssl_capath | 是 | 是 | 是 | 全局 | 否 |
| mysqlx_ssl_cert | 是 | 是 | 是 | 全局 | 否 |
| mysqlx_ssl_cipher | 是 | 是 | 是 | 全局 | 否 |
| mysqlx_ssl_crl | 是 | 是 | 是 | 全局 | 否 |
| mysqlx_ssl_crlpath | 是 | 是 | 是 | 全局 | 否 |
| mysqlx_ssl_key | 是 | 是 | 是 | 全局 | 否 |
| mysqlx_wait_timeout | 是 | 是 | 是 | 会话 | 是 |
| mysqlx_write_timeout | 是 | 是 | 是 | 会话 | 是 |
| mysqlx_zstd_default_compression_level | 是 | 是 | 是 | 全局 | 是 |
| mysqlx_zstd_max_client_compression_level | 是 | 是 | 是 | 全局 | 是 |
| named_pipe | 是 | 是 | 是 | 全局 | 否 |
| named_pipe_full_access_group | 是 | 是 | 是 | 全局 | 否 |
| ndb_allow_copying_alter_table | 是 | 是 | 是 | 两者 | 是 |
| ndb_applier_allow_skip_epoch | 是 | 是 | 是 | 全局 | 否 |
| ndb_autoincrement_prefetch_sz | 是 | 是 | 是 | 两者 | �� |
| ndb_batch_size | 是 | 是 | 是 | 两者 | 是 |
| ndb_blob_read_batch_bytes | 是 | 是 | 是 | 两者 | 是 |
| ndb_blob_write_batch_bytes | 是 | 是 | 是 | 两者 | 是 |
| ndb_clear_apply_status | 是 |  | 是 | 全局 | 是 |
| ndb_cluster_connection_pool | 是 | 是 | 是 | 全局 | 否 |
| ndb_cluster_connection_pool_nodeids | 是 | 是 | 是 | 全局 | 否 |
| ndb_conflict_role | 是 | 是 | 是 | 全局 | 是 |
| ndb_data_node_neighbour | 是 | 是 | 是 | 全局 | 是 |
| ndb_dbg_check_shares | 是 | 是 | 是 | 两者 | 是 |
| ndb_default_column_format | 是 | 是 | 是 | 全局 | 是 |
| ndb_default_column_format | 是 | 是 | 是 | 全局 | 是 |
| ndb_deferred_constraints | 是 | 是 | 是 | 两者 | 是 |
| ndb_deferred_constraints | 是 | 是 | 是 | 两者 | 是 |
| ndb_distribution | 是 | 是 | 是 | 全局 | 是 |
| ndb_distribution | 是 | 是 | 是 | 全局 | 是 |
| ndb_eventbuffer_free_percent | 是 | 是 | 是 | 全局 | 是 |
| ndb_eventbuffer_max_alloc | 是 | 是 | 是 | 全局 | 是 |
| ndb_extra_logging | 是 | 是 | 是 | 全局 | 是 |
| ndb_force_send | 是 | 是 | 是 | 两者 | 是 |
| ndb_fully_replicated | 是 | 是 | 是 | 两者 | 是 |
| ndb_index_stat_enable | 是 | 是 | 是 | 两者 | 是 |
| ndb_index_stat_option | 是 | 是 | 是 | 两者 | 是 |
| ndb_join_pushdown |  |  | 是 | 两者 | 是 |
| ndb_log_apply_status | 是 | 是 | 是 | 全局 | 否 |
| ndb_log_apply_status | 是 | 是 | 是 | 全局 | 否 |
| ndb_log_bin | 是 |  | 是 | 两者 | 否 |
| ndb_log_binlog_index | 是 |  | 是 | 全局 | 是 |
| ndb_log_empty_epochs | 是 | 是 | 是 | 全局 | 是 |
| ndb_log_empty_epochs | 是 | 是 | 是 | 全局 | 是 |
| ndb_log_empty_update | 是 | 是 | 是 | 全局 | 是 |
| ndb_log_empty_update | 是 | 是 | 是 | 全局 | 是 |
| ndb_log_exclusive_reads | 是 | 是 | 是 | 两者 | 是 |
| ndb_log_exclusive_reads | 是 | 是 | 是 | 两者 | 是 |
| ndb_log_fail_terminate | 是 | 是 | 是 | 全局 | 否 |
| ndb_log_orig | 是 | 是 | 是 | 全局 | 否 |
| ndb_log_orig | 是 | 是 | 是 | 全局 | 否 |
| ndb_log_transaction_compression | 是 | 是 | 是 | 全局 | 是 |
| ndb_log_transaction_compression_level_zstd | 是 | 是 | 是 | 全局 | 是 |
| ndb_log_transaction_dependency | 是 | 是 | 是 | 全局 | 否 |
| ndb_log_transaction_id | 是 | 是 | 是 | 全局 | 否 |
| ndb_log_transaction_id |  |  | 是 | 全局 | 否 |
| ndb_log_update_as_write | 是 | 是 | 是 | 全局 | 是 |
| ndb_log_update_minimal | 是 | 是 | 是 | 全局 | 是 |
| ndb_log_updated_only | 是 | 是 | 是 | 全局 | 是 |
| ndb_metadata_check | 是 | 是 | 是 | 全局 | 是 |
| ndb_metadata_check_interval | 是 | 是 | 是 | 全局 | 是 |
| ndb_metadata_sync |  |  | 是 | 全局 | 是 |
| ndb_optimization_delay | 是 | 是 | 是 | 全局 | 是 |
| ndb_optimized_node_selection | 是 | 是 | 是 | 全局 | 否 |
| ndb_read_backup | 是 | 是 | 是 | 全局 | 是 |
| ndb_recv_thread_activation_threshold | 是 | 是 | 是 | 全局 | 是 |
| ndb_recv_thread_cpu_mask | 是 | 是 | 是 | 全局 | 是 |
| ndb_replica_batch_size | 是 | 是 | 是 | 全局 | 是 |
| ndb_replica_blob_write_batch_bytes | 是 | 是 | 是 | 全局 | 是 |
| Ndb_replica_max_replicated_epoch |  |  | 是 | 全局 | 否 |
| ndb_report_thresh_binlog_epoch_slip | 是 | 是 | 是 | 全局 | 是 |
| ndb_report_thresh_binlog_mem_usage | 是 | 是 | 是 | 全局 | 是 |
| ndb_row_checksum |  |  | 是 | 两者 | 是 |
| ndb_schema_dist_lock_wait_timeout | 是 | 是 | 是 | 全局 | 是 |
| ndb_schema_dist_timeout | 是 | 是 | 是 | 全局 | 否 |
| ndb_schema_dist_timeout | 是 | 是 | 是 | 全局 | 否 |
| ndb_schema_dist_upgrade_allowed | 是 | 是 | 是 | 全局 | 否 |
| ndb_show_foreign_key_mock_tables | 是 | 是 | 是 | 全局 | 是 |
| ndb_slave_conflict_role | 是 | 是 | 是 | 全局 | 是 |
| Ndb_system_name |  |  | 是 | 全局 | 否 |
| ndb_table_no_logging |  |  | 是 | 会话 | 是 |
| ndb_table_temporary |  |  | 是 | 会话 | 是 |
| ndb_use_copying_alter_table |  |  | 是 | 两者 | 否 |
| ndb_use_exact_count |  |  | 是 | 两者 | 是 |
| ndb_use_transactions | 是 | 是 | 是 | 两者 | 是 |
| ndb_version |  |  | 是 | 全局 | 否 |
| ndb_version_string |  |  | 是 | 全局 | 否 |
| ndb_wait_connected | 是 | 是 | 是 | 全局 | 否 |
| ndb_wait_setup | 是 | 是 | 是 | 全局 | 否 |
| ndbinfo_database |  |  | 是 | 全局 | 否 |
| ndbinfo_max_bytes | 是 |  | 是 | 两者 | 是 |
| ndbinfo_max_rows | 是 |  | 是 | 两者 | 是 |
| ndbinfo_offline |  |  | 是 | 全局 | 是 |
| ndbinfo_show_hidden | 是 |  | 是 | 两者 | 是 |
| ndbinfo_table_prefix |  |  | 是 | 全局 | 否 |
| ndbinfo_version |  |  | 是 | 全局 | 否 |
| net_buffer_length | 是 | 是 | 是 | 两者 | 是 |
| net_read_timeout | 是 | 是 | 是 | 两者 | 是 |
| net_retry_count | 是 | 是 | 是 | 两者 | 是 |
| net_write_timeout | 是 | 是 | 是 | 两者 | 是 |
| new | 是 | 是 | 是 | 两者 | 是 |
| ngram_token_size | 是 | 是 | 是 | 全局 | 否 |
| offline_mode | 是 | 是 | 是 | 全局 | 是 |
| old | 是 | 是 | 是 | 全局 | 否 |
| old_alter_table | 是 | 是 | 是 | 两者 | 是 |
| open_files_limit | 是 | 是 | 是 | 全局 | 否 |
| optimizer_prune_level | 是 | 是 | 是 | 两者 | 是 |
| optimizer_search_depth | 是 | 是 | 是 | 两者 | 是 |
| optimizer_switch | 是 | 是 | 是 | 两者 | 是 |
| optimizer_trace | 是 | 是 | 是 | 两者 | 是 |
| optimizer_trace_features | 是 | 是 | 是 | 两者 | 是 |
| optimizer_trace_limit | 是 | 是 | 是 | 两者 | 是 |
| optimizer_trace_max_mem_size | 是 | 是 | 是 | 两者 | 是 |
| optimizer_trace_offset | 是 | 是 | 是 | 两者 | 是 |
| original_commit_timestamp |  |  | 是 | 会话 | 是 |
| original_server_version |  |  | 是 | 会话 | 是 |
| parser_max_mem_size | 是 | 是 | 是 | 两者 | 是 |
| 部分撤销 | 是 | 是 | 是 | 全局 | 是 |
| 密码历史 | 是 | 是 | 是 | 全局 | 是 |
| 密码要求当前 | 是 | 是 | 是 | 全局 | 是 |
| 密码重用间隔 | 是 | 是 | 是 | 全局 | 是 |
| 性能模式 | 是 | 是 | 是 | 全局 | 否 |
| 性能模式账户大小 | 是 | 是 | 是 | 全局 | 否 |
| 性能模式摘要大小 | 是 | 是 | 是 | 全局 | 否 |
| 性能模式错误大小 | 是 | 是 | 是 | 全局 | 否 |
| 性能模式事件阶段历史长大小 | 是 | 是 | 是 | 全局 | 否 |
| 性能模式事件阶段历史大小 | 是 | 是 | 是 | 全局 | 否 |
| 性能模式事件语句历史长大小 | 是 | 是 | 是 | 全局 | 否 |
| 性能模式事件语句历史大小 | 是 | 是 | 是 | 全局 | 否 |
| 性能模式事件事务历史长大小 | 是 | 是 | 是 | 全局 | 否 |
| 性能模式事件事务历史大小 | 是 | 是 | 是 | 全局 | 否 |
| 性能模式事件等待历史长大小 | 是 | 是 | 是 | 全局 | 否 |
| 性能模式事件等待历史大小 | 是 | 是 | 是 | 全局 | 否 |
| 性能模式主机大小 | 是 | 是 | 是 | 全局 | 否 |
| 性能模式最大条件类 | 是 | 是 | 是 | 全局 | 否 |
| performance_schema_max_cond_instances | 是 | 是 | 是 | 全局 | 否 |
| performance_schema_max_digest_length | 是 | 是 | 是 | 全局 | 否 |
| performance_schema_max_digest_sample_age | 是 | 是 | 是 | 全局 | 是 |
| performance_schema_max_file_classes | 是 | 是 | 是 | 全局 | 否 |
| performance_schema_max_file_handles | 是 | 是 | 是 | 全局 | 否 |
| performance_schema_max_file_instances | 是 | 是 | 是 | 全局 | 否 |
| performance_schema_max_index_stat | 是 | 是 | 是 | 全局 | 否 |
| performance_schema_max_memory_classes | 是 | 是 | 是 | 全局 | 否 |
| performance_schema_max_metadata_locks | 是 | 是 | 是 | 全局 | 否 |
| performance_schema_max_mutex_classes | 是 | 是 | 是 | 全局 | 否 |
| performance_schema_max_mutex_instances | 是 | 是 | 是 | 全局 | 否 |
| performance_schema_max_prepared_statements_instances | 是 | 是 | 是 | 全局 | 否 |
| performance_schema_max_program_instances | 是 | 是 | 是 | 全局 | 否 |
| performance_schema_max_rwlock_classes | 是 | 是 | 是 | 全局 | 否 |
| performance_schema_max_rwlock_instances | 是 | 是 | 是 | 全局 | 否 |
| performance_schema_max_socket_classes | 是 | 是 | 是 | 全局 | 否 |
| performance_schema_max_socket_instances | 是 | 是 | 是 | 全局 | 否 |
| performance_schema_max_sql_text_length | 是 | 是 | 是 | 全局 | 否 |
| performance_schema_max_stage_classes | 是 | 是 | 是 | 全局 | 否 |
| performance_schema_max_statement_classes | 是 | 是 | 是 | 全局 | 否 |
| performance_schema_max_statement_stack | 是 | 是 | 是 | 全局 | 否 |
| performance_schema_max_table_handles | 是 | 是 | 是 | 全局 | 否 |
| performance_schema_max_table_instances | 是 | 是 | 是 | 全局 | 否 |
| performance_schema_max_table_lock_stat | 是 | 是 | 是 | 全局 | 否 |
| performance_schema_max_thread_classes | 是 | 是 | 是 | 全局 | 否 |
| performance_schema_max_thread_instances | 是 | 是 | 是 | 全局 | 否 |
| performance_schema_session_connect_attrs_size | 是 | 是 | 是 | 全局 | 否 |
| performance_schema_setup_actors_size | 是 | 是 | 是 | 全局 | 否 |
| performance_schema_setup_objects_size | 是 | 是 | 是 | 全局 | 否 |
| performance_schema_show_processlist | 是 | 是 | 是 | 全局 | 是 |
| performance_schema_users_size | 是 | 是 | 是 | 全局 | 否 |
| persist_only_admin_x509_subject | 是 | 是 | 是 | 全局 | 否 |
| persist_sensitive_variables_in_plaintext | 是 | 是 | 是 | 全局 | 否 |
| persisted_globals_load | 是 | 是 | 是 | 全局 | 否 |
| pid_file | 是 | 是 | 是 | 全局 | 否 |
| plugin_dir | 是 | 是 | 是 | 全局 | 否 |
| port | 是 | 是 | 是 | 全局 | 否 |
| preload_buffer_size | 是 | 是 | 是 | 两者 | 是 |
| print_identified_with_as_hex | 是 | 是 | 是 | 两者 | 是 |
| profiling |  |  | 是 | 两者 | 是 |
| profiling_history_size | 是 | 是 | 是 | 两者 | 是 |
| protocol_compression_algorithms | 是 | 是 | 是 | 全局 | 是 |
| protocol_version |  |  | 是 | 全局 | 否 |
| proxy_user |  |  | 是 | 会话 | 否 |
| pseudo_replica_mode |  |  | 是 | 会话 | 是 |
| pseudo_slave_mode |  |  | 是 | 会话 | 是 |
| pseudo_thread_id |  |  | 是 | 会话 | 是 |
| query_alloc_block_size | 是 | 是 | 是 | 两者 | 是 |
| query_prealloc_size | 是 | 是 | 是 | 两者 | 是 |
| rand_seed1 |  |  | 是 | 会话 | 是 |
| rand_seed2 |  |  | 是 | 会话 | 是 |
| range_alloc_block_size | 是 | 是 | 是 | 两者 | 是 |
| range_optimizer_max_mem_size | 是 | 是 | 是 | 两者 | 是 |
| rbr_exec_mode |  |  | 是 | 会话 | 是 |
| read_buffer_size | 是 | 是 | 是 | 两者 | 是 |
| read_only | 是 | 是 | 是 | 全局 | 是 |
| read_rnd_buffer_size | 是 | 是 | 是 | 两者 | 是 |
| regexp_stack_limit | 是 | 是 | 是 | 全局 | 是 |
| regexp_time_limit | 是 | 是 | 是 | 全局 | 是 |
| relay_log | 是 | 是 | 是 | 全局 | 否 |
| relay_log_basename |  |  | 是 | 全局 | 否 |
| relay_log_index | 是 | 是 | 是 | 全局 | 否 |
| relay_log_info_file | 是 | 是 | 是 | 全局 | 否 |
| relay_log_info_repository | 是 | 是 | 是 | 全局 | 是 |
| relay_log_purge | 是 | 是 | 是 | 全局 | 是 |
| relay_log_recovery | 是 | 是 | 是 | 全局 | 否 |
| relay_log_space_limit | 是 | 是 | 是 | 全局 | 否 |
| replica_allow_batching | 是 | 是 | 是 | 全局 | 是 |
| replica_checkpoint_group | 是 | 是 | 是 | 全局 | 是 |
| replica_checkpoint_period | 是 | 是 | 是 | 全局 | 是 |
| replica_compressed_protocol | 是 | 是 | 是 | 全局 | 是 |
| replica_exec_mode | 是 | 是 | 是 | 全局 | 是 |
| replica_load_tmpdir | 是 | 是 | 是 | 全局 | 否 |
| replica_max_allowed_packet | 是 | 是 | 是 | 全局 | 是 |
| replica_net_timeout | 是 | 是 | 是 | 全局 | 是 |
| replica_parallel_type | 是 | 是 | 是 | 全局 | 是 |
| replica_parallel_workers | 是 | 是 | 是 | 全局 | 是 |
| replica_pending_jobs_size_max | 是 | 是 | 是 | 全局 | 是 |
| replica_preserve_commit_order | 是 | 是 | 是 | 全局 | 是 |
| replica_skip_errors | 是 | 是 | 是 | 全局 | 否 |
| replica_sql_verify_checksum | 是 | 是 | 是 | 全局 | 是 |
| replica_transaction_retries | 是 | 是 | 是 | 全局 | 是 |
| replica_type_conversions | 是 | 是 | 是 | 全局 | 是 |
| replication_optimize_for_static_plugin_config | 是 | 是 | 是 | 全局 | 是 |
| replication_sender_observe_commit_only | 是 | 是 | 是 | 全局 | 是 |
| report_host | 是 | 是 | 是 | 全局 | 否 |
| report_password | 是 | 是 | 是 | 全局 | 否 |
| report_port | 是 | 是 | 是 | 全局 | 否 |
| report_user | 是 | 是 | 是 | 全局 | 否 |
| require_row_format |  |  | 是 | 会话 | 是 |
| require_secure_transport | 是 | 是 | 是 | 全局 | 是 |
| resultset_metadata |  |  | 是 | 会话 | 是 |
| 重写器启用 |  |  | 是 | 全局 | 是 |
| 无需特权检查线程启用重写器 |  |  | 是 | 全局 | 是 |
| 重写器详细模式 |  |  | 是 | 全局 | 是 |
| rpl_read_size | 是 | 是 | 是 | 全局 | 是 |
| rpl_semi_sync_master_enabled | 是 | 是 | 是 | 全局 | 是 |
| rpl_semi_sync_master_timeout | 是 | 是 | 是 | 全局 | 是 |
| rpl_semi_sync_master_trace_level | 是 | 是 | 是 | 全局 | 是 |
| rpl_semi_sync_master_wait_for_slave_count | 是 | 是 | 是 | 全局 | 是 |
| rpl_semi_sync_master_wait_no_slave | 是 | 是 | 是 | 全局 | 是 |
| rpl_semi_sync_master_wait_point | 是 | 是 | 是 | 全局 | 是 |
| rpl_semi_sync_replica_enabled | 是 | 是 | 是 | 全局 | 是 |
| rpl_semi_sync_replica_trace_level | 是 | 是 | 是 | 全局 | 是 |
| rpl_semi_sync_slave_enabled | 是 | 是 | 是 | 全局 | 是 |
| rpl_semi_sync_slave_trace_level | 是 | 是 | 是 | 全局 | 是 |
| rpl_semi_sync_source_enabled | 是 | 是 | 是 | 全局 | 是 |
| rpl_semi_sync_source_timeout | 是 | 是 | 是 | 全局 | 是 |
| rpl_semi_sync_source_trace_level | 是 | 是 | 是 | 全局 | 是 |
| rpl_semi_sync_source_wait_for_replica_count | 是 | 是 | 是 | 全局 | 是 |
| rpl_semi_sync_source_wait_no_replica | 是 | 是 | 是 | 全局 | 是 |
| rpl_semi_sync_source_wait_point | 是 | 是 | 是 | 全局 | 是 |
| rpl_stop_replica_timeout | 是 | 是 | 是 | 全局 | 是 |
| rpl_stop_slave_timeout | 是 | 是 | 是 | 全局 | 是 |
| schema_definition_cache | 是 | 是 | 是 | 全局 | 是 |
| secondary_engine_cost_threshold |  |  | 是 | 会话 | 是 |
| secure_file_priv | 是 | 是 | 是 | 全局 | 否 |
| select_into_buffer_size | 是 | 是 | 是 | 两者 | 是 |
| select_into_disk_sync | 是 | 是 | 是 | 两者 | 是 |
| select_into_disk_sync_delay | 是 | 是 | 是 | 两者 | 是 |
| server_id | 是 | 是 | 是 | 全局 | 是 |
| server_id_bits | 是 | 是 | 是 | 全局 | 否 |
| server_uuid |  |  | 是 | 全局 | 否 |
| session_track_gtids | 是 | 是 | 是 | 两者 | 是 |
| session_track_schema | 是 | 是 | 是 | 两者 | 是 |
| session_track_state_change | 是 | 是 | 是 | 两者 | 是 |
| session_track_system_variables | 是 | 是 | 是 | 两者 | 是 |
| session_track_transaction_info | 是 | 是 | 是 | 两者 | 是 |
| sha256_password_auto_generate_rsa_keys | 是 | 是 | 是 | 全局 | 否 |
| sha256_password_private_key_path | 是 | 是 | 是 | 全局 | 否 |
| sha256_password_proxy_users | 是 | 是 | 是 | 全局 | 是 |
| sha256_password_public_key_path | 是 | 是 | 是 | 全局 | 否 |
| shared_memory | 是 | 是 | 是 | 全局 | 否 |
| shared_memory_base_name | 是 | 是 | 是 | 全局 | 否 |
| show_create_table_skip_secondary_engine | 是 | 是 | 是 | 会话 | 是 |
| show_create_table_verbosity | 是 | 是 | 是 | 两者 | 是 |
| show_gipk_in_create_table_and_information_schema | 是 | 是 | 是 | 两者 | 是 |
| show_old_temporals | 是 | 是 | 是 | 两者 | 是 |
| skip_external_locking | 是 | 是 | 是 | 全局 | 否 |
| skip_name_resolve | 是 | 是 | 是 | 全局 | 否 |
| skip_networking | 是 | 是 | 是 | 全局 | 否 |
| skip_replica_start | 是 | 是 | 是 | 全局 | 否 |
| skip_show_database | 是 | 是 | 是 | 全局 | 否 |
| skip_slave_start | 是 | 是 | 是 | 全局 | 否 |
| slave_allow_batching | 是 | 是 | 是 | 全局 | 是 |
| slave_checkpoint_group | 是 | 是 | 是 | 全局 | 是 |
| slave_checkpoint_period | 是 | 是 | 是 | 全局 | 是 |
| slave_compressed_protocol | 是 | 是 | 是 | 全局 | 是 |
| slave_exec_mode | 是 | 是 | 是 | 全局 | 是 |
| slave_load_tmpdir | 是 | 是 | 是 | 全局 | 否 |
| slave_max_allowed_packet | 是 | 是 | 是 | 全局 | 是 |
| slave_net_timeout | 是 | 是 | 是 | 全局 | 是 |
| slave_parallel_type | 是 | 是 | 是 | 全局 | 是 |
| slave_parallel_workers | 是 | 是 | 是 | 全局 | 是 |
| slave_pending_jobs_size_max | 是 | 是 | 是 | 全局 | 是 |
| slave_preserve_commit_order | 是 | 是 | 是 | 全局 | 是 |
| slave_rows_search_algorithms | 是 | 是 | 是 | 全局 | 是 |
| slave_skip_errors | 是 | 是 | 是 | 全局 | 否 |
| slave_sql_verify_checksum | 是 | 是 | 是 | 全局 | 是 |
| slave_transaction_retries | 是 | 是 | 是 | 全局 | 是 |
| slave_type_conversions | 是 | 是 | 是 | 全局 | 是 |
| slow_launch_time | 是 | 是 | 是 | 全局 | 是 |
| slow_query_log | 是 | 是 | 是 | 全局 | 是 |
| slow_query_log_file | 是 | 是 | 是 | 全局 | 是 |
| socket | 是 | 是 | 是 | 全局 | 否 |
| sort_buffer_size | 是 | 是 | 是 | 两者 | 是 |
| source_verify_checksum | 是 | 是 | 是 | 全��� | 是 |
| sql_auto_is_null |  |  | 是 | 两者 | 是 |
| sql_big_selects |  |  | 是 | 两者 | 是 |
| sql_buffer_result |  |  | 是 | 两者 | 是 |
| sql_generate_invisible_primary_key | 是 | 是 | 是 | 两者 | 是 |
| sql_log_bin |  |  | 是 | 会话 | 是 |
| sql_log_off |  |  | 是 | 两者 | 是 |
| sql_mode | 是 | 是 | 是 | 两者 | 是 |
| sql_notes |  |  | 是 | 两者 | 是 |
| sql_quote_show_create |  |  | 是 | 两者 | 是 |
| sql_replica_skip_counter |  |  | 是 | 全局 | 是 |
| sql_require_primary_key | 是 | 是 | 是 | 两者 | 是 |
| sql_safe_updates |  |  | 是 | 两者 | 是 |
| sql_select_limit |  |  | 是 | 两者 | 是 |
| sql_slave_skip_counter |  |  | 是 | 全局 | 是 |
| sql_warnings |  |  | 是 | 两者 | 是 |
| ssl_ca | 是 | 是 | 是 | 全局 | 不定 |
| ssl_capath | 是 | 是 | 是 | 全局 | 不同 |
| ssl_cert | 是 | 是 | 是 | 全局 | 不同 |
| ssl_cipher | 是 | 是 | 是 | 全局 | 不同 |
| ssl_crl | 是 | 是 | 是 | 全局 | 不同 |
| ssl_crlpath | 是 | 是 | 是 | 全局 | 不同 |
| ssl_fips_mode | 是 | 是 | 是 | 全局 | 否 |
| ssl_key | 是 | 是 | 是 | 全局 | 不同 |
| ssl_session_cache_mode | 是 | 是 | 是 | 全局 | 是 |
| ssl_session_cache_timeout | 是 | 是 | 是 | 全局 | 是 |
| stored_program_cache | 是 | 是 | 是 | 全局 | 是 |
| stored_program_definition_cache | 是 | 是 | 是 | 全局 | 是 |
| super_read_only | 是 | 是 | 是 | 全局 | 是 |
| sync_binlog | 是 | 是 | 是 | 全局 | 是 |
| sync_master_info | 是 | 是 | 是 | 全局 | 是 |
| sync_relay_log | 是 | 是 | 是 | 全局 | 是 |
| sync_relay_log_info | 是 | 是 | 是 | 全局 | 是 |
| sync_source_info | 是 | 是 | 是 | 全局 | 是 |
| syseventlog.facility | 是 | 是 | 是 | 全局 | 是 |
| syseventlog.include_pid | 是 | 是 | 是 | 全局 | 是 |
| syseventlog.tag | 是 | 是 | 是 | 全局 | 是 |
| system_time_zone |  |  | 是 | 全局 | 否 |
| table_definition_cache | 是 | 是 | 是 | 全局 | 是 |
| table_encryption_privilege_check | 是 | 是 | 是 | 全局 | 是 |
| table_open_cache | 是 | 是 | 是 | 全局 | 是 |
| table_open_cache_instances | 是 | 是 | 是 | 全局 | 否 |
| tablespace_definition_cache | 是 | 是 | 是 | 全局 | 是 |
| temptable_max_mmap | 是 | 是 | 是 | 全局 | 是 |
| temptable_max_ram | 是 | 是 | 是 | 全局 | 是 |
| temptable_use_mmap | 是 | 是 | 是 | 全局 | 是 |
| terminology_use_previous | 是 | 是 | 是 | 两者 | 是 |
| thread_cache_size | 是 | 是 | 是 | 全局 | 是 |
| thread_handling | 是 | 是 | 是 | 全局 | 否 |
| thread_pool_algorithm | 是 | 是 | 是 | 全局 | 否 |
| thread_pool_dedicated_listeners | 是 | 是 | 是 | 全局 | 否 |
| thread_pool_high_priority_connection | 是 | 是 | 是 | 两者 | 是 |
| thread_pool_max_active_query_threads | 是 | 是 | 是 | 全局 | 是 |
| thread_pool_max_transactions_limit | 是 | 是 | 是 | 全局 | 是 |
| thread_pool_max_unused_threads | 是 | 是 | 是 | 全局 | 是 |
| thread_pool_prio_kickup_timer | 是 | 是 | 是 | 全局 | 是 |
| thread_pool_query_threads_per_group | 是 | 是 | 是 | 全局 | 是 |
| thread_pool_size | 是 | 是 | 是 | 全局 | 否 |
| thread_pool_stall_limit | 是 | 是 | 是 | 全局 | 是 |
| thread_pool_transaction_delay | 是 | 是 | 是 | 全局 | 是 |
| thread_stack | 是 | 是 | 是 | 全局 | 否 |
| time_zone |  |  | 是 | 两者 | 是 |
| timestamp |  |  | 是 | 会话 | 是 |
| tls_ciphersuites | 是 | 是 | 是 | 全局 | 是 |
| tls_version | 是 | 是 | 是 | 全局 | 不定 |
| tmp_table_size | 是 | 是 | 是 | 两者 | 是 |
| tmpdir | 是 | 是 | 是 | 全局 | 否 |
| transaction_alloc_block_size | 是 | 是 | 是 | 两者 | 是 |
| transaction_allow_batching |  |  | 是 | 会话 | 是 |
| transaction_isolation | 是 | 是 | 是 | 两者 | 是 |
| transaction_prealloc_size | 是 | 是 | 是 | 两者 | 是 |
| transaction_read_only | 是 | 是 | 是 | 两者 | 是 |
| transaction_write_set_extraction | 是 | 是 | 是 | 两者 | 是 |
| unique_checks |  |  | 是 | 两者 | 是 |
| updatable_views_with_limit | 是 | 是 | 是 | 两者 | 是 |
| use_secondary_engine |  |  | 是 | 会话 | 是 |
| validate_password_check_user_name | 是 | 是 | 是 | 全局 | 是 |
| validate_password_dictionary_file | 是 | 是 | 是 | 全局 | 是 |
| validate_password_length | 是 | 是 | 是 | 全局 | 是 |
| validate_password_mixed_case_count | 是 | 是 | 是 | 全局 | 是 |
| validate_password_number_count | 是 | 是 | 是 | 全局 | 是 |
| validate_password_policy | 是 | 是 | 是 | 全局 | 是 |
| validate_password_special_char_count | 是 | 是 | 是 | 全局 | 是 |
| validate_password_changed_characters_percentage | 是 | 是 | 是 | 全局 | 是 |
| validate_password_check_user_name | 是 | 是 | 是 | 全局 | 是 |
| validate_password.dictionary_file | 是 | 是 | 是 | 全局 | 是 |
| validate_password.length | 是 | 是 | 是 | 全局 | 是 |
| validate_password.mixed_case_count | 是 | 是 | 是 | 全局 | 是 |
| validate_password.number_count | 是 | 是 | 是 | 全局 | 是 |
| validate_password.policy | 是 | 是 | 是 | 全局 | 是 |
| validate_password.special_char_count | 是 | 是 | 是 | 全局 | 是 |
| version |  |  | 是 | 全局 | 否 |
| version_comment |  |  | 是 | 全局 | 否 |
| version_compile_machine |  |  | 是 | 全局 | 否 |
| version_compile_os |  |  | 是 | 全局 | 否 |
| version_compile_zlib |  |  | 是 | 全局 | 否 |
| version_tokens_session | 是 | 是 | 是 | 两者 | 是 |
| version_tokens_session_number | 是 | 是 | 是 | 两者 | 否 |
| wait_timeout | 是 | 是 | 是 | 两者 | 是 |
| warning_count |  |  | 是 | 会话 | 否 |
| windowing_use_high_precision | 是 | 是 | 是 | 两者 | 是 |
| xa_detach_on_prepare | 是 | 是 | 是 | 两者 | 是 |
| 名称 | 命令行 | 选项文件 | 系统变量 | 变量范围 | 动态 |

**注意：**

1\. 这个选项是动态的，但应该只由服务器设置。不应该手动设置这个变量。
