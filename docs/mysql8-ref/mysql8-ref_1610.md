> 原文：[`dev.mysql.com/doc/refman/8.0/en/x-plugin-status-variables.html`](https://dev.mysql.com/doc/refman/8.0/en/x-plugin-status-variables.html)

#### 22.5.6.3 X Plugin 状态变量

X Plugin 状态变量具有以下含义。

+   `Mysqlx_aborted_clients`

    因输入或输出错误而断开连接的客户端数量。

+   `Mysqlx_address`

    X Plugin 接受 TCP/IP 连接的网络地址或地址。如果使用`mysqlx_bind_address`系统变量指定了多个地址，则`Mysqlx_address`仅显示绑定成功的地址。如果对于`mysqlx_bind_address`指定的每个网络地址绑定失败，或者使用了`skip_networking`选项，则`Mysqlx_address`的值为`UNDEFINED`。如果 X Plugin 启动尚未完成，则`Mysqlx_address`的值为空。

+   `Mysqlx_bytes_received`

    通过网络接收的总字节数。如果连接使用压缩，此数字包括压缩前测量的压缩消息负载（`Mysqlx_bytes_received_compressed_payload`），未压缩的消息中未压缩的项目（例如 X 协议标头）以及任何未压缩的消息。

+   `Mysqlx_bytes_received_compressed_payload`

    作为压缩消息负载接收的字节数，解压缩前测量。

+   `Mysqlx_bytes_received_uncompressed_frame`

    作为压缩消息负载接收的字节数，解压缩后测量。

+   `Mysqlx_bytes_sent`

    通过网络发送的总字节数。如果连接使用压缩，此数字包括压缩后测量的压缩消息负载（`Mysqlx_bytes_sent_compressed_payload`），未压缩的消息中未压缩的项目（例如 X 协议标头）以及任何未压缩的消息。

+   `Mysqlx_bytes_sent_compressed_payload`

    作为压缩消息负载发送的字节数，压缩后测量。

+   `Mysqlx_bytes_sent_uncompressed_frame`

    作为压缩消息负载发送的字节数，压缩前测量。

+   `Mysqlx_compression_algorithm`

    （会话范围）此会话的 X 协议连接使用的压缩算法。允许的压缩算法由`mysqlx_compression_algorithms`系统变量列出。

+   `Mysqlx_compression_level`

    （会话范围）此会话的 X 协议连接使用的压缩级别。

+   `Mysqlx_connection_accept_errors`

    导致接受错误的连接数。

+   `Mysqlx_connection_errors`

    导致错误的连接数。

+   `Mysqlx_connections_accepted`

    已接受的连接数。

+   `Mysqlx_connections_closed`

    已关闭的连接数。

+   `Mysqlx_connections_rejected`

    已拒绝的连接数。

+   `Mysqlx_crud_create_view`

    收到的创建视图请求数。

+   `Mysqlx_crud_delete`

    收到的删除请求数。

+   `Mysqlx_crud_drop_view`

    收到的删除视图请求数。

+   `Mysqlx_crud_find`

    收到的查找请求数。

+   `Mysqlx_crud_insert`

    收到的插入请求数。

+   `Mysqlx_crud_modify_view`

    收到的修改视图请求数。

+   `Mysqlx_crud_update`

    收到的更新请求数。

+   `Mysqlx_cursor_close`

    收到的关闭游标消息数

+   `Mysqlx_cursor_fetch`

    收到的获取游标消息数

+   `Mysqlx_cursor_open`

    收到的打开游标消息数

+   `Mysqlx_errors_sent`

    发送给客户端的错误数。

+   `Mysqlx_errors_unknown_message_type`

    收到的未知消息类型数量。

+   `Mysqlx_expect_close`

    已关闭的期望块数量。

+   `Mysqlx_expect_open`

    已打开的期望块数量。

+   `Mysqlx_init_error`

    初始化期间的错误数量。

+   `Mysqlx_messages_sent`

    发送给客户端的所有类型消息总数。

+   `Mysqlx_notice_global_sent`

    发送给客户端的全局通知数量。

+   `Mysqlx_notice_other_sent`

    发送回客户端的其他类型通知数量。

+   `Mysqlx_notice_warning_sent`

    发送回客户端的警告通知数量。

+   `Mysqlx_notified_by_group_replication`

    发送给客户端的 Group Replication 通知数量。

+   `Mysqlx_port`

    X 插件正在监听的 TCP 端口。如果网络绑定失败，或者启用了 `skip_networking` 系统变量，则该值显示为 `UNDEFINED`。

+   `Mysqlx_prep_deallocate`

    收到的准备语句释放消息数量

+   `Mysqlx_prep_execute`

    收到的准备语句执行消息数量

+   `Mysqlx_prep_prepare`

    收到的准备语句消息数量

+   `Mysqlx_rows_sent`

    发送回客户端的行数。

+   `Mysqlx_sessions`

    已打开的会话数量。

+   `Mysqlx_sessions_accepted`

    已接受的会话尝试次数。

+   `Mysqlx_sessions_closed`

    已关闭的会话数量。

+   `Mysqlx_sessions_fatal_error`

    已关闭且出现致命错误的会话数量。

+   `Mysqlx_sessions_killed`

    已终止的会话数量。

+   `Mysqlx_sessions_rejected`

    被拒绝的会话尝试次数。

+   `Mysqlx_socket`

    X 插件正在监听的 Unix 套接字。

+   `Mysqlx_ssl_accept_renegotiates`

    建立连接所需的协商次数。

+   `Mysqlx_ssl_accepts`

    接受的 SSL 连接数量。

+   `Mysqlx_ssl_active`

    SSL 是否激活。

+   `Mysqlx_ssl_cipher`

    当前的 SSL 密码（非 SSL 连接为空）。

+   `Mysqlx_ssl_cipher_list`

    可能的 SSL 密码列表（非 SSL 连接为空）。

+   `Mysqlx_ssl_ctx_verify_depth`

    当前在 ctx 中设置的证书验证深度限制。

+   `Mysqlx_ssl_ctx_verify_mode`

    当前在 ctx 中设置的证书验证模式。

+   `Mysqlx_ssl_finished_accepts`

    成功的 SSL 连接到服务器的数量。

+   `Mysqlx_ssl_server_not_after`

    SSL 证书有效的最后一个日期。

+   `Mysqlx_ssl_server_not_before`

    SSL 证书有效的第一个日期。

+   `Mysqlx_ssl_verify_depth`

    SSL 连接的证书验证深度。

+   `Mysqlx_ssl_verify_mode`

    SSL 连接的证书验证模式。

+   `Mysqlx_ssl_version`

    用于 SSL 连接的协议名称。

+   `Mysqlx_stmt_create_collection`

    收到的创建集合语句的数量。

+   `Mysqlx_stmt_create_collection_index`

    收到的创建集合索引语句的数量。

+   `Mysqlx_stmt_disable_notices`

    收到的禁用通知语句的数量。

+   `Mysqlx_stmt_drop_collection`

    收到的删除集合语句的数量。

+   `Mysqlx_stmt_drop_collection_index`

    收到的删除集合索引语句的数量。

+   `Mysqlx_stmt_enable_notices`

    收到的启用通知语句数量。

+   `Mysqlx_stmt_ensure_collection`

    收到的确保集合语句数量。

+   `Mysqlx_stmt_execute_mysqlx`

    收到的带有命名空间设置为 `mysqlx` 的 StmtExecute 消息数量。

+   `Mysqlx_stmt_execute_sql`

    收到的针对 SQL 命名空间的 StmtExecute 请求数量。

+   `Mysqlx_stmt_execute_xplugin`

    收到的针对 `xplugin` 命名空间的 StmtExecute 请求数量。从 MySQL 8.0.19 开始，`xplugin` 命名空间已被移除，因此不再使用此状态变量。 

+   `Mysqlx_stmt_get_collection_options`

    收到的获取集合对象语句数量。

+   `Mysqlx_stmt_kill_client`

    收到的终止客户端语句数量。

+   `Mysqlx_stmt_list_clients`

    收到的列出客户端语句数量。

+   `Mysqlx_stmt_list_notices`

    收到的列出通知语句数量。

+   `Mysqlx_stmt_list_objects`

    收到的列出对象语句数量。

+   `Mysqlx_stmt_modify_collection_options`

    收到的修改集合选项语句数量。

+   `Mysqlx_stmt_ping`

    收到的 ping 语句数量。

+   `Mysqlx_worker_threads`

    可用的工作线程数量。

+   `Mysqlx_worker_threads_active`

    当前正在使用的工作线程数量。
