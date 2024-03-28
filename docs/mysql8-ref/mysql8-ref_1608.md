> 原文：[`dev.mysql.com/doc/refman/8.0/en/x-plugin-option-variable-reference.html`](https://dev.mysql.com/doc/refman/8.0/en/x-plugin-option-variable-reference.html)

#### 22.5.6.1 X 插件选项和变量参考

该表提供了 X 插件提供的命令选项、系统变量和状态变量的概述。

**表 22.2 X 插件选项和变量参考**

| 名称 | 命令行 | 选项文件 | 系统变量 | 状态变量 | 变量范围 | 动态 |
| --- | --- | --- | --- | --- | --- | --- |
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
| mysqlx_compression_algorithms | 是 | 是 | ��� |  | 全局 | 是 |
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
| 名称 | 命令行 | 选项文件 | 系统变量 | 状态变量 | 变量范围 | 动态 |
