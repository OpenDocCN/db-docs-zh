# 20.9 组复制变量

> 原文：[`dev.mysql.com/doc/refman/8.0/en/group-replication-options.html`](https://dev.mysql.com/doc/refman/8.0/en/group-replication-options.html)

20.9.1 组复制系统变量

20.9.2 组复制状态变量

接下来的两个部分包含了关于 MySQL 服务器系统和服务器状态变量的信息，这些变量是特定于组复制插件的。

**表 20.4 组复制变量和选项摘要**

| 名称 | 命令行 | 选项文件 | 系统变量 | 状态变量 | 变量范围 | 动态 |
| --- | --- | --- | --- | --- | --- | --- |
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
| group_replication_recovery_get_public_key | 是 | 是 | 是 |  | 全局 | 是 |
| group_replication_recovery_public_key_path | 是 | 是 | 是 |  | 全局 | 是 |
| group_replication_recovery_reconnect_interval | 是 | 是 | 是 |  | ��局 | 是 |
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
| group_replication_single_primary_mode | 是 | 是 | 是 |  | 全局 | 是 |
| group_replication_ssl_mode | 是 | 是 | 是 |  | 全局 | 是 |
| group_replication_start_on_boot | 是 | 是 | 是 |  | 全局 | 是 |
| group_replication_transaction_size_limit | 是 | 是 | 是 |  | 全局 | 是 |
| group_replication_unreachable_majority_timeout | 是 | 是 | 是 |  | 全局 | 是 |
| group_replication_view_change_uuid | 是 | 是 | 是 |  | 全局 | 是 |
| 名称 | 命令行 | 选项文件 | 系统变量 | 状态变量 | 变量范围 | 动态 |
