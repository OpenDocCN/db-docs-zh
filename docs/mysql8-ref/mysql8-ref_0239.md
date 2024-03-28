# 7.1.10 服务器状态变量

> 原文：[`dev.mysql.com/doc/refman/8.0/en/server-status-variables.html`](https://dev.mysql.com/doc/refman/8.0/en/server-status-variables.html)

MySQL 服务器维护许多状态变量，提供有关其操作的信息。您可以通过使用`SHOW [GLOBAL | SESSION] STATUS`语句（参见第 15.7.7.37 节，“SHOW STATUS 语句”）查看这些变量及其值。可选的`GLOBAL`关键字聚合所有连接的值，`SESSION`显示当前连接的值。

```sql
mysql> SHOW GLOBAL STATUS;
+-----------------------------------+------------+
| Variable_name                     | Value      |
+-----------------------------------+------------+
| Aborted_clients                   | 0          |
| Aborted_connects                  | 0          |
| Bytes_received                    | 155372598  |
| Bytes_sent                        | 1176560426 |
...
| Connections                       | 30023      |
| Created_tmp_disk_tables           | 0          |
| Created_tmp_files                 | 3          |
| Created_tmp_tables                | 2          |
...
| Threads_created                   | 217        |
| Threads_running                   | 88         |
| Uptime                            | 1389872    |
+-----------------------------------+------------+
```

许多状态变量通过`FLUSH STATUS`语句重置为 0。

本节提供了每个状态变量的描述。有关状态变量摘要，请参见第 7.1.6 节，“服务器状态变量参考”。有关特定于 NDB 集群的状态变量信息，请参见第 25.4.3.9.3 节，“NDB 集群状态变量”。

状态变量具有以下含义。

+   `Aborted_clients`

    连接由于客户端异常关闭连接而中止的次数。请参见第 B.3.2.9 节，“通信错误和中止连接”。

+   `Aborted_connects`

    尝试连接到 MySQL 服务器失败的次数。请参见第 B.3.2.9 节，“通信错误和中止连接”。

    要获取更多与连接相关的信息，请查看`Connection_errors_*`xxx`*`状态变量和`host_cache`表。

+   `Authentication_ldap_sasl_supported_methods`

    实现 SASL LDAP 身份验证的`authentication_ldap_sasl`插件支持多种身份验证方法，但根据主机系统配置的不同，它们可能并非全部可用。`Authentication_ldap_sasl_supported_methods`变量提供了支持的方法的可发现性。其值是由支持的方法名称以空格分隔的字符串。例如："SCRAM-SHA 1 SCRAM-SHA-256 GSSAPI"

    此变量已添加到 MySQL 8.0.21 中。

+   `Binlog_cache_disk_use`

    使用临时二进制日志缓存的事务数量，但超过了`binlog_cache_size`的值，并使用临时文件存储事务中的语句。

    导致二进制日志事务缓存写入磁盘的非事务语句的数量在`Binlog_stmt_cache_disk_use`状态变量中单独跟踪。

+   `Acl_cache_items_count`

    缓存的权限对象数量。每个对象是用户及其活动角色的权限组合。

+   `Binlog_cache_use`

    使用二进制日志缓存的事务数量。

+   `Binlog_stmt_cache_disk_use`

    使用二进制日志语句缓存的非事务语句的数量，但超过了`binlog_stmt_cache_size`的值，并使用临时文件存储这些语句。

+   `Binlog_stmt_cache_use`

    使用二进制日志语句缓存的非事务语句的数量。

+   `Bytes_received`

    从所有客户端接收的字节数。

+   `Bytes_sent`

    发送给所有客户端的字节数。

+   `Caching_sha2_password_rsa_public_key`

    `caching_sha2_password`身份验证插件用于 RSA 密钥对密码交换的公钥。仅当服务器成功初始化由`caching_sha2_password_private_key_path`和`caching_sha2_password_public_key_path`系统变量命名的文件中的私钥和公钥时，该值才不为空。`Caching_sha2_password_rsa_public_key`的值来自后者的文件。

+   `Com_*`xxx`*`

    `Com_*`xxx`*`语句计数变量指示每个*`xxx`*语句已执行的次数。每种类型的语句都有一个状态变量。例如，`Com_delete`和`Com_update`分别计算`DELETE`和`UPDATE`语句的次数。`Com_delete_multi`和`Com_update_multi`类似，但适用于使用多表语法的`DELETE`和`UPDATE`语句。

    所有`Com_stmt_*`xxx`*`变量都会增加，即使准备语句参数未知或在执行过程中发生错误。换句话说，它们的值对应于发出的请求次数，而不是成功完成的请求次数。例如，因为状态变量在每次服务器启动时都会初始化，并且不会在重新启动时保留，跟踪`RESTART`和`SHUTDOWN`语句的`Com_restart`和`Com_shutdown`变量通常值为零，但如果执行了但失败了`RESTART`或`SHUTDOWN`语句，则可以为非零。

    `Com_stmt_*`xxx`*`状态变量如下：

    +   `Com_stmt_prepare`

    +   `Com_stmt_execute`

    +   `Com_stmt_fetch`

    +   `Com_stmt_send_long_data`

    +   `Com_stmt_reset`

    +   `Com_stmt_close`

    这些变量代表准备语句命令。它们的名称指的是在网络层中使用的`COM_*`xxx`*`命令集。换句话说，每当执行准备语句 API 调用，如**mysql_stmt_prepare()**、**mysql_stmt_execute()**等时，它们的值会增加。然而，`Com_stmt_prepare`、`Com_stmt_execute`和`Com_stmt_close`也会分别增加`PREPARE`、`EXECUTE`或`DEALLOCATE PREPARE`。此外，旧的语句计数变量`Com_prepare_sql`、`Com_execute_sql`和`Com_dealloc_sql`的值也会增加`PREPARE`、`EXECUTE`和`DEALLOCATE PREPARE`语句。`Com_stmt_fetch`代表从游标中获取时发出的网络往返总数。

    `Com_stmt_reprepare` 表示服务器自动重新准备语句的次数，例如，在语句引用的表或视图的元数据更改后。重新准备操作会增加`Com_stmt_reprepare`，也会增加`Com_stmt_prepare`。

    `Com_explain_other` 表示执行的`EXPLAIN FOR CONNECTION`语句的数量。请参阅第 10.8.4 节，“获取命名连接的执行计划信息”。

    `Com_change_repl_filter` 表示执行的`CHANGE REPLICATION FILTER`语句的数量。

+   `压缩`

    客户端连接是否在客户端/服务器协议中使用压缩。

    截至 MySQL 8.0.18，此状态变量已弃用；预计将在未来的 MySQL 版本中移除。请参阅 Configuring Legacy Connection Compression。

+   `Compression_algorithm`

    当前连接到服务器的压缩算法的名称。该值可以是`protocol_compression_algorithms`系统变量值中允许的任何算法。例如，如果连接不使用压缩，则值为`uncompressed`，如果连接使用`zlib`算法，则值为`zlib`。

    欲了解更多信息，请参阅 Section 6.2.8, “Connection Compression Control”。

    该变量在 MySQL 8.0.18 中添加。

+   `Compression_level`

    当前连接到服务器的压缩级别。对于`zlib`连接，该值为 6（默认`zlib`算法压缩级别），对于`zstd`连接，该值为 1 到 22，对于`uncompressed`连接，该值为 0。

    欲了解更多信息，请参阅 Section 6.2.8, “Connection Compression Control”。

    该变量在 MySQL 8.0.18 中添加。

+   `Connection_errors_*`xxx`*`

    这些变量提供有关客户端连接过程中发生的错误的信息。它们仅为全局变量，代表来自所有主机的连接中聚合的错误计数。这些变量跟踪主机缓存未解决的错误（请参阅 Section 7.1.12.3, “DNS Lookups and the Host Cache”），例如与 TCP 连接无关的错误，在连接过程的早期阶段发生的错误（甚至在知道 IP 地址之前），或者不特定于任何特定 IP 地址的错误（例如内存不足条件）。

    +   `Connection_errors_accept`

        在监听端口调用`accept()`时发生的错误次数。

    +   `Connection_errors_internal`

        由于服务器内部错误（例如无法启动新线程或内存不足）而拒绝的连接数。

    +   `Connection_errors_max_connections`

        由于达到服务器`max_connections`限制而拒绝的连接数。

    +   `Connection_errors_peer_address`

        在搜索连接客户端 IP 地址时发生的错误数量。

    +   `Connection_errors_select`

        在监听端口上调用`select()`或`poll()`时发生的错误数量。（此操作的失败并不一定意味着客户端连接被拒绝。）

    +   `Connection_errors_tcpwrap`

        `libwrap`库拒绝的连接数量。

+   `Connections`

    尝试连接到 MySQL 服务器的连接尝试次数（成功或不成功）。

+   `Created_tmp_disk_tables`

    服务器在执行语句时创建的内部磁盘临时表的数量。

    您可以通过比较`Created_tmp_disk_tables`和`Created_tmp_tables`的值来比较创建的内部磁盘临时表的数量和创建的内部临时表的总数。

    注意

    由于已知限制，`Created_tmp_disk_tables`不计算在内存映射文件中创建的磁盘临时表。默认情况下，TempTable 存储引擎溢出机制在内存映射文件中创建内部临时表。此行为由`temptable_use_mmap`变量控制，默认情况下启用。

    另请参阅第 10.4.4 节，“MySQL 中的内部临时表使用”。

+   `Created_tmp_files`

    **mysqld**创建的临时文件数量。

+   `Created_tmp_tables`

    服务器在执行语句时创建的内部临时表的数量。

    您可以通过比较`Created_tmp_disk_tables`和`Created_tmp_tables`的值来比较创建的内部磁盘临时表的数量和创建的内部临时表的总数。

    另请参阅第 10.4.4 节，“MySQL 中的内部临时表使用”。

    每次调用`SHOW STATUS`语句都会使用一个内部临时表，并增加全局`Created_tmp_tables`值。

+   `Current_tls_ca`

    服务器在新连接中使用的 SSL 上下文中的活动`ssl_ca`值。如果系统变量已更改但尚未执行`ALTER INSTANCE RELOAD TLS`以从与上下文相关的系统变量值重新配置 SSL 上下文并更新相应的状态变量，则此上下文值可能与当前`ssl_ca`系统变量值不同。（这些值之间的潜在差异适用于每个对应的上下文相关系统和状态变量对。请参阅用于加密连接的服务器端运行时配置和监控。）

    这个变量是在 MySQL 8.0.16 中添加的。

    截至 MySQL 8.0.21，`Current_tls_*`xxx`*`状态变量值也可以通过性能模式`tls_channel_status`表获得。请参阅第 29.12.21.9 节，“tls_channel_status 表”。

+   `Current_tls_capath`

    服务器在新连接中使用的 TLS 上下文中的活动`ssl_capath`值。关于此状态变量与其对应系统变量之间关系的说明，请参阅`Current_tls_ca`的描述。

    这个变量是在 MySQL 8.0.16 中添加的。

+   `Current_tls_cert`

    服务器在新连接中使用的 TLS 上下文中的活动`ssl_cert`值。关于此状态变量与其对应系统变量之间关系的说明，请参阅`Current_tls_ca`的描述。

    这个变量是在 MySQL 8.0.16 中添加的。

+   `Current_tls_cipher`

    服务器在新连接中使用的 TLS 上下文中的活动`ssl_cipher`值。关于此状态变量与其对应系统变量之间关系的说明，请参阅`Current_tls_ca`的描述。

    这个变量是在 MySQL 8.0.16 中添加的。

+   `Current_tls_ciphersuites`

    服务器在新连接中使用的 TLS 上下文中的活动`tls_ciphersuites`值。有关此状态变量与其对应系统变量之间关系的说明，请参阅`Current_tls_ca`的描述。

    该变量是在 MySQL 8.0.16 中添加的。

+   `Current_tls_crl`

    服务器在新连接中使用的 TLS 上下文中的活动`ssl_crl`值。有关此状态变量与其对应系统变量之间关系的说明，请参阅`Current_tls_ca`的描述。

    该变量是在 MySQL 8.0.16 中添加的。

    注意

    当重新加载 TLS 上下文时，OpenSSL 会在此过程中重新加载包含 CRL（证书吊销列表）的文件。如果 CRL 文件很大，服务器会分配一个大块内存（文件大小的十倍），在新实例加载时会加倍，而旧实例尚未释放。大量分配被释放后，进程驻留内存不会立即减少，因此如果你重复发出带有大型 CRL 文件的`ALTER INSTANCE RELOAD TLS`语句，进程驻留内存使用量可能会因此增长。

+   `Current_tls_crlpath`

    服务器在新连接中使用的 TLS 上下文中的活动`ssl_crlpath`值。有关此状态变量与其对应系统变量之间关系的说明，请参阅`Current_tls_ca`的描述。

    该变量是在 MySQL 8.0.16 中添加的。

+   `Current_tls_key`

    服务器在新连接中使用的 TLS 上下文中的活动`ssl_key`值。有关此状态变量与其对应系统变量之间关系的说明，请参阅`Current_tls_ca`的描述。

    该变量是在 MySQL 8.0.16 中添加的。

+   `Current_tls_version`

    服务器在新连接中使用的 TLS 上下文中的活动`tls_version`值。有关此状态变量与其对应系统变量之间关系的说明，请参阅`Current_tls_ca`的描述。

    该变量是在 MySQL 8.0.16 中添加的。

+   `Delayed_errors`

    此状态变量已被弃用（因为不支持`DELAYED`插入）；预计在将来的版本中将其移除。

+   `Delayed_insert_threads`

    此状态变量已弃用（因为不支持 `DELAYED` 插入）；预计将在将来的版本中删除。

+   `Delayed_writes`

    此状态变量已弃用（因为不支持 `DELAYED` 插入）；预计将在将来的版本中删除。

+   `dragnet.Status`

    最近对 `dragnet.log_error_filter_rules` 系统变量的赋值结果，如果没有进行此类赋值，则为空。

    此变量是在 MySQL 8.0.12 中添加的。

+   `Error_log_buffered_bytes`

    当前在性能模式 `error_log` 表中使用的字节数。可能会出现值减少的情况，例如，如果新事件无法容纳，直到丢弃旧事件，但新事件比旧事件小。

    此变量是在 MySQL 8.0.22 中添加的。

+   `Error_log_buffered_events`

    当前在性能模式 `error_log` 表中存在的事件数量。与 `Error_log_buffered_bytes` ��似，可能会出现值减少的情况。

    此变量是在 MySQL 8.0.22 中添加的。

+   `Error_log_expired_events`

    从性能模式 `error_log` 表中丢弃的事件数量，以腾出空间供新事件使用。

    此变量是在 MySQL 8.0.22 中添加的。

+   `Error_log_latest_write`

    最后一次写入性能模式 `error_log` 表的时间。

    此变量是在 MySQL 8.0.22 中添加的。

+   `Flush_commands`

    服务器刷新表的次数，无论是因为用户执行了 `FLUSH TABLES` 语句还是由于内部服务器操作。也会通过接收 `COM_REFRESH` 数据包来增加。这与 `Com_flush` 相反，后者指示执行了多少次 `FLUSH` 语句，无论是 `FLUSH TABLES`、`FLUSH LOGS` 等等。

+   `Global_connection_memory`

    所有用户连接到服务器的内存使用情况。系统线程或 MySQL 根帐户使用的内存包括在内，但这些线程或用户不会因内存使用而断开连接。除非启用了`global_connection_memory_tracking`（默认情况下为禁用），否则不会计算此内存。性能模式也必须启用。

    您可以通过设置`connection_memory_chunk_size`间接控制此变量更新的频率。

    `Global_connection_memory`状态变量是在 MySQL 8.0.28 中引入的。

+   `Handler_commit`

    内部`COMMIT`语句的次数。

+   `Handler_delete`

    从表中删除行的次数。

+   `Handler_external_lock`

    服务器为每次调用其`external_lock()`函数递增此变量，这通常发生在访问表实例的开始和结束时。存储引擎之间可能存在差异。例如，可以使用此变量来发现访问分区表的语句在锁定发生之前剪枝了多少分区：检查语句的计数器增加了多少，减去 2（表本身的 2 次调用），然后除以 2 以获取锁定的分区数。

+   `Handler_mrr_init`

    服务器使用存储引擎自己的多范围读取实现进行表访问的次数。

+   `Handler_prepare`

    两阶段提交操作准备阶段的计数器。

+   `Handler_read_first`

    读取索引中第一个条目的次数。如果此值很高，则表明服务器正在执行大量完整索引扫描（例如，`SELECT col1 FROM foo`，假设`col1`已建立索引）。

+   `Handler_read_key`

    基于键读取行的请求数。如果此值很高，则表明您的表针对查询已正确建立索引。

+   `Handler_read_last`

    读取索引中最后一个键的请求数。使用`ORDER BY`时，服务器会发出第一个键的请求，然后是几个下一个键的请求，而使用`ORDER BY DESC`时，服务器会发出最后一个键的请求，然后是几个前一个键的请求。

+   `Handler_read_next`

    按键顺序读取下一行的请求数。如果您正在查询带有范围约束的索引列或正在执行索引扫描，则此值会增加。

+   `Handler_read_prev`

    按键顺序读取上一行的请求数。此读取方法主要用于优化`ORDER BY ... DESC`。

+   `Handler_read_rnd`

    基于固定位置读取一行的请求数。如果您正在执行需要对结果进行排序的大量查询，则此值会很高。您可能有很多需要 MySQL 扫描整个表或连接未正确使用键的查询。

+   `Handler_read_rnd_next`

    读取数据文件中下一行的请求数。如果您正在执行大量表扫描，则此值会很高。通常这表明您的表没有正确索引，或者您的查询没有利用您拥有的索引。

+   `Handler_rollback`

    存储引擎执行回滚操作的请求数。

+   `Handler_savepoint`

    存储引擎放置保存点的请求数。

+   `Handler_savepoint_rollback`

    存储引擎回滚到保存点的请求数。

+   `Handler_update`

    更新表中一行的请求数。

+   `Handler_write`

    在表中插入一行的请求数。

+   `Innodb_buffer_pool_dump_status`

    记录在`InnoDB` 缓冲池中保存的页的操作进度，由`innodb_buffer_pool_dump_at_shutdown`或`innodb_buffer_pool_dump_now`的设置触发。

    有关相关信息和示例，请参见第 17.8.3.6 节，“保存和恢复缓冲池状态”。

+   `Innodb_buffer_pool_load_status`

    通过读取与较早时间点对应的一组页来预热 `InnoDB` 缓冲池的操作进度，由`innodb_buffer_pool_load_at_startup`或`innodb_buffer_pool_load_now`的设置触发。如果该操作引入了太多开销，您可以通过设置`innodb_buffer_pool_load_abort`来取消它。

    有关相关信息和示例，请参见第 17.8.3.6 节，“保存和恢复缓冲池状态”。

+   `Innodb_buffer_pool_bytes_data`

    包含数据的`InnoDB` 缓冲池中的总字节数。该数字包括脏和干净页。当压缩表导致缓冲池持有不同大小的页时，比`Innodb_buffer_pool_pages_data`更准确的内存使用量计算。

+   `Innodb_buffer_pool_pages_data`

    包含数据的`InnoDB` 缓冲池中的页数量。该数字包括脏和干净页。使用压缩表时，报告的`Innodb_buffer_pool_pages_data`值可能大于`Innodb_buffer_pool_pages_total`（Bug #59550）。

+   `Innodb_buffer_pool_bytes_dirty`

    当压缩表导致缓冲池持有不同大小的页时，`InnoDB` 缓冲池中脏页所持有的总字节数。比`Innodb_buffer_pool_pages_dirty`更准确的内存使用量计算。

+   `Innodb_buffer_pool_pages_dirty`

    当前在`InnoDB` 缓冲池中的脏页数量。

+   `Innodb_buffer_pool_pages_flushed`

    请求从`InnoDB` 缓冲池中刷新页的次数。

+   `Innodb_buffer_pool_pages_free`

    `InnoDB` 缓冲池中空闲页的数量。

+   `Innodb_buffer_pool_pages_latched`

    `InnoDB` 缓冲池中被锁定的页的数量。这些页当前正在被读取或写入，或者由于某种原因无法刷新或移除。由于计算这个变量的成本很高，因此只有在服务器构建时定义了`UNIV_DEBUG`系统时才可用。

+   `Innodb_buffer_pool_pages_misc`

    `InnoDB` 缓冲池中因已分配用于行锁或自适应哈希索引等管理开销而繁忙的页的数量。此值也可以计算为`Innodb_buffer_pool_pages_total` − `Innodb_buffer_pool_pages_free` − `Innodb_buffer_pool_pages_data`。在使用压缩表时，`Innodb_buffer_pool_pages_misc`可能报告一个超出范围的值（Bug #59550）。

+   `Innodb_buffer_pool_pages_total`

    `InnoDB` 缓冲池的总大小，以页为单位。在使用压缩表时，报告的`Innodb_buffer_pool_pages_data`值可能大于`Innodb_buffer_pool_pages_total`（Bug #59550）

+   `Innodb_buffer_pool_read_ahead`

    由后台预读线程读入`InnoDB` 缓冲池的页面数量。

+   `Innodb_buffer_pool_read_ahead_evicted`

    由后台预读线程读入`InnoDB` 缓冲池的页面数量，随后被查询未访问而被驱逐的页面数量。

+   `Innodb_buffer_pool_read_ahead_rnd`

    由`InnoDB`发起的“随机”预读次数。当查询以随机顺序扫描表的大部分内容时会发生这种情况。

+   `Innodb_buffer_pool_read_requests`

    逻辑读请求的数量。

+   `Innodb_buffer_pool_reads`

    `InnoDB`无法从缓冲池中满足的逻辑读数量，必须直接从磁盘读取。

+   `Innodb_buffer_pool_resize_status`

    调整`InnoDB` 缓冲池大小的操作状态是动态的，通过动态设置`innodb_buffer_pool_size`参数触发。`innodb_buffer_pool_size`参数是动态的，允许您在不重启服务器的情况下调整缓冲池大小。有关更多信息，请参见在线配置 InnoDB 缓冲池大小。

+   `Innodb_buffer_pool_resize_status_code`

    报告用于跟踪在线缓冲池调整操作的状态代码。每个状态代码代表调整操作中的一个阶段。状态代码包括：

    +   0: 没有正在进行的调整大小操作

    +   1: 开始调整大小

    +   2: 禁用 AHI（自适应哈希索引）

    +   3: 撤回块

    +   4: 获取全局锁

    +   5: 调整池

    +   6: 调整哈希

    +   7: 调整失败

    您可以将此状态变量与`Innodb_buffer_pool_resize_status_progress`结合使用，以跟踪调整操作每个阶段的进度。`Innodb_buffer_pool_resize_status_progress`变量报告一个百分比值，指示当前阶段的进度。

    更多信息，请参阅监控在线缓冲池调整进度。

+   `Innodb_buffer_pool_resize_status_progress`

    报告一个百分比值，指示在线缓冲池调整操作当前阶段的进度。此变量与`Innodb_buffer_pool_resize_status_code`一起使用，后者报告当前在线缓冲池调整操作阶段的状态代码。

    百分比值在每个缓冲池实例处理后更新。随着状态代码（由`Innodb_buffer_pool_resize_status_code`报告）从一个状态变为另一个状态，百分比值将重置为 0。

    有关相关信息，请参阅监控在线缓冲池调整进度。

+   `Innodb_buffer_pool_wait_free`

    通常，对`InnoDB` 缓冲池的写入是在后台进行的。当`InnoDB`需要读取或创建一个页且没有可用的干净页时，`InnoDB`首先刷新一些脏页并等待该操作完成。此计数器计算这些等待的实例。如果`innodb_buffer_pool_size`已经适当设置，这个值应该很小。

+   `Innodb_buffer_pool_write_requests`

    写入到`InnoDB` 缓冲池的次数。

+   `Innodb_data_fsyncs`

    到目前为止的`fsync()`操作次数。`fsync()`调用的频率受`innodb_flush_method`配置选项的设置影响。

    如果启用了`innodb_use_fdatasync`，则计算`fdatasync()`操作的次数。

+   `Innodb_data_pending_fsyncs`

    当前待处理的`fsync()`操作数量。`fsync()`调用的频率受`innodb_flush_method`配置选项的设置影响。

+   `Innodb_data_pending_reads`

    当前待处理读取的数量。

+   `Innodb_data_pending_writes`

    当前待处理写入次数。

+   `Innodb_data_read`

    服务器启动以来读取的数据量（以字节为单位）。

+   `Innodb_data_reads`

    总数据读取次数（操作系统文件读取）。

+   `Innodb_data_writes`

    总数据写入次数。

+   `Innodb_data_written`

    到目前为止写入的数据量，以字节为单位。

+   `Innodb_dblwr_pages_written`

    写入到双写缓冲区的页面数量。参见第 17.11.1 节，“InnoDB 磁盘 I/O”。

+   `Innodb_dblwr_writes`

    执行的双写操作次数。参见第 17.11.1 节，“InnoDB 磁盘 I/O”。

+   `Innodb_have_atomic_builtins`

    指示服务器是否使用原子指令构建。

+   `Innodb_log_waits`

    日志缓冲区太小，需要等待刷新才能继续的次数。

+   `Innodb_log_write_requests`

    对`InnoDB` 重做日志 的写入请求次数。

+   `Innodb_log_writes`

    对`InnoDB` 重做日志 文件的物理写入次数。

+   `Innodb_num_open_files`

    `InnoDB`当前打开的文件数量。

+   `Innodb_os_log_fsyncs`

    对`InnoDB` 重做日志 文件执行的`fsync()`写入次数。

+   `Innodb_os_log_pending_fsyncs`

    `InnoDB` 重做日志 文件的待处理`fsync()`操作数量。

+   `Innodb_os_log_pending_writes`

    对`InnoDB` 重做日志 文件的待写入次数。

+   `Innodb_os_log_written`

    写入到 `InnoDB` 重做日志 文件的字节数。

+   `Innodb_page_size`

    `InnoDB` 页面大小（默认为 16KB）。许多值以页面为单位计数；页面大小使它们可以轻松转换为字节。

+   `Innodb_pages_created`

    由对 `InnoDB` 表的操作创建的页面数。

+   `Innodb_pages_read`

    由于对 `InnoDB` 表的操作从 `InnoDB` 缓冲池中读取的页面数。

+   `Innodb_pages_written`

    由对 `InnoDB` 表的操作写入的页面数。

+   `Innodb_redo_log_enabled`

    重做日志记录是否已启用或��禁用。请参见 禁用重做日志。

    此变量在 MySQL 8.0.21 中添加。

+   `Innodb_redo_log_capacity_resized`

    所有重做日志文件的总重做日志容量（以字节为单位），在最后一次完成的容量调整操作之后。该值包括普通和备用重做日志文件。

    如果没有待处理的调整容量向下操作，`Innodb_redo_log_capacity_resized` 应等于 `innodb_redo_log_capacity` 设置。调整容量向上的操作是瞬时的。

    有关相关信息，请参见 Section 17.6.5, “重做日志”。

    此变量在 MySQL 8.0.30 中添加。

+   `Innodb_redo_log_checkpoint_lsn`

    重做日志检查点 LSN。有关相关信息，请参见 Section 17.6.5, “重做日志”。

    此变量在 MySQL 8.0.30 中添加。

+   `Innodb_redo_log_current_lsn`

    当前 LSN 表示重做日志缓冲区中的最后写入位置。`InnoDB` 在请求操作系统将数据写入当前重做日志文件之前，将数据写入 MySQL 进程内的重做日志缓冲区。有关相关信息，请参见 Section 17.6.5, “重做日志”。

    此变量在 MySQL 8.0.30 中添加。

+   `Innodb_redo_log_flushed_to_disk_lsn`

    刷新到磁盘的 LSN。`InnoDB` 首先将数据写入重做日志，然后请求操作系统将数据刷新到磁盘。刷新到磁盘的 LSN 表示 `InnoDB` 知道已刷新到磁盘的重做日志中的最后位置。有关相关信息，请参见 Section 17.6.5, “重做日志”。

    此变量是在 MySQL 8.0.30 中添加的。

+   `Innodb_redo_log_logical_size`

    数据大小值，以字节为单位，表示包含正在使用的重做日志数据的 LSN 范围，从重做日志消费者所需的最旧块到最新写入块。有关相关信息，请参见第 17.6.5 节，“重做日志”。

    此变量是在 MySQL 8.0.30 中添加的。

+   `Innodb_redo_log_physical_size`

    当前在磁盘上所有重做日志文件当前消耗的字节大小，不包括备用重做日志文件。有关相关信息，请参见第 17.6.5 节，“重做日志”。

    此变量是在 MySQL 8.0.30 中添加的。

+   `Innodb_redo_log_read_only`

    重做日志是否为只读。

    此变量是在 MySQL 8.0.30 中添加的。

+   `Innodb_redo_log_resize_status`

    重做日志调整大小状态指示重做日志容量调整机制的当前状态。可能的值包括：

    +   `OK`：没有问题，也没有待处理的重做日志容量调整操作。

    +   `正在调整大小向下`：正在进行向下调整操作。

    向上调整操作是瞬时的，因此没有待处理状态。

    此变量是在 MySQL 8.0.30 中添加的。

+   `Innodb_redo_log_uuid`

    重做日志 UUID。

    此变量是在 MySQL 8.0.30 中添加的。

+   `Innodb_row_lock_current_waits`

    当前由`InnoDB`表上的操作等待的行锁的数量。

+   `Innodb_row_lock_time`

    获取`InnoDB`表的行锁所花费的总时间，单位为毫秒。

+   `Innodb_row_lock_time_avg`

    获取`InnoDB`表的行锁的平均时间，单位为毫秒。

+   `Innodb_row_lock_time_max`

    获取`InnoDB`表的行锁的最大时间，单位为毫秒。

+   `Innodb_row_lock_waits`

    `InnoDB`表上的操作必须等待行锁的次数。

+   `Innodb_rows_deleted`

    从`InnoDB`表中删除的行数。

+   `Innodb_rows_inserted`

    插入到`InnoDB`表中的行数。

+   `Innodb_rows_read`

    从`InnoDB`表中读取的行数。

+   `Innodb_rows_updated`

    在`InnoDB`表中更新的估计行数。

    注意

    此值并非百分之百准确。要获得准确（但更昂贵）的结果，请使用`ROW_COUNT()`。

+   `Innodb_system_rows_deleted`

    从系统创建的模式所属的`InnoDB`表中删除的行数。

+   `Innodb_system_rows_inserted`

    插入到系统创建的模式所属的`InnoDB`表中的行数。

+   `Innodb_system_rows_read`

    从系统创建的模式所属的`InnoDB`表中读取的行数。

+   `Innodb_truncated_status_writes`

    `SHOW ENGINE INNODB STATUS`语句的输出被截断的次数。

+   `Innodb_undo_tablespaces_active`

    活动撤销表空间的数量。包括隐式（由`InnoDB`创建）和显式（用户创建）的撤销表空间。有关撤销表空间的信息，请参见第 17.6.3.4 节，“撤销表空间”。

+   `Innodb_undo_tablespaces_explicit`

    用户创建的撤销表空间的数量。有关撤销表空间的信息，请参见第 17.6.3.4 节，“撤销表空间”。

+   `Innodb_undo_tablespaces_implicit`

    `InnoDB`创建的撤销表空间的数量。当初始化 MySQL 实例时，`InnoDB`会创建两个默认的撤销表空间。有关撤销表空间的信息，请参见第 17.6.3.4 节，“撤销表空间”。

+   `Innodb_undo_tablespaces_total`

    撤销表空间的总数。包括隐式（由`InnoDB`创建）和显式（用户创建）的撤销表空间，活动和非活动的。有关撤销表空间的信息，请参见第 17.6.3.4 节，“撤销表空间”。

+   `Key_blocks_not_flushed`

    `MyISAM`键缓存中已更改但尚未刷新到磁盘的关键块数。

+   `Key_blocks_unused`

    `MyISAM`键缓存中未使用的块数。您可以使用此值来确定键缓存的使用情况；请参阅第 7.1.8 节，“服务器系统变量”中对`key_buffer_size`的讨论。

+   `Key_blocks_used`

    `MyISAM`键缓存中使用的块数。此值是一个高水位标记，表示曾经同时使用的最大块数。

+   `Key_read_requests`

    请求从`MyISAM`键缓存中读取键块的次数。

+   `Key_reads`

    从磁盘读取键块到`MyISAM`键缓存的物理读取次数。如果`Key_reads`很大，则您的`key_buffer_size`值可能太小。缓存未命中率可以计算为`Key_reads`/`Key_read_requests`。

+   `Key_write_requests`

    请求将键块写入`MyISAM`键缓存的次数。

+   `Key_writes`

    从`MyISAM`键缓存到磁盘的键块的物理写入次数。

+   `Last_query_cost`

    查询优化器计算的上一个编译查询的总成本。这对于比较相同查询的不同查询计划的成本很有用。默认值为 0 表示尚未编译任何查询。默认值为 0。`Last_query_cost`具有会话范围。

    在 MySQL 8.0.16 及更高版本中，此变量显示具有多个查询块的查询的成本，总结每个查询块的成本估计，估计非可缓存子查询执行的次数，并将这些查询块的成本乘以子查询执行的次数。 (Bug #92766, Bug #28786951) 在 MySQL 8.0.16 之前，`Last_query_cost`仅对简单的“平面”查询进行准确计算，而不适用于包含子查询或`UNION`等复杂查询。 (对于后者，该值设置为 0。)

+   `Last_query_partial_plans`

    查询优化器在构建上一个查询的执行计划时所做的迭代次数。

    `Last_query_partial_plans`具有会话范围。

+   `Locked_connects`

    尝试连接被锁定用户账户的次数。有关账户锁定和解锁的信息，请参见第 8.2.20 节，“账户锁定”。

+   `Max_execution_time_exceeded`

    执行超时时间已超过的`SELECT`语句数量。

+   `Max_execution_time_set`

    设置了非零执行超时时间的`SELECT`语句数量。这包括包含非零`MAX_EXECUTION_TIME`优化提示的语句，以及不包含此类提示但在`max_execution_time`系统变量指示的超时时间非零时执行的语句。

+   `Max_execution_time_set_failed`

    尝试设置执行超时失败的`SELECT`语句数量。

+   `Max_used_connections`

    服务器启动以来同时使用的最大连接数。

+   `Max_used_connections_time`

    达到当前值时的`Max_used_connections`时间。

+   `Not_flushed_delayed_rows`

    此状态变量已弃用（因为不支持`DELAYED`插入）；预计在将来的版本中将其移除。

+   `mecab_charset`

    当前由 MeCab 全文本解析器插件使用的字符集。有关相关信息，请参见第 14.9.9 节，“MeCab 全文本解析器插件”。

+   `Ongoing_anonymous_transaction_count`

    显示已标记为匿名的进行中事务数量。这可用于确保没有进一步的事务等待处理。

+   `Ongoing_anonymous_gtid_violating_transaction_count`

    此状态变量仅在调试构建中可用。显示使用`gtid_next=ANONYMOUS`并违反 GTID 一致性的进行中事务数量。

+   `Ongoing_automatic_gtid_violating_transaction_count`

    此状态变量仅在调试构建中可用。显示正在使用`gtid_next=AUTOMATIC`并违反 GTID 一致性的进行中事务的数量。

+   `Open_files`

    已打开的文件数量。此计数包括服务器打开的常规文件。它不包括其他类型的文件，如套接字或管道。此计数还不包括存储引擎使用其自己的内部函数而不是请求服务器级别执行的文件。

+   `Open_streams`

    已打开的流数量（主要用于日志记录）。

+   `Open_table_definitions`

    已缓存的表定义数量。

+   `Open_tables`

    已打开的表数量。

+   `Opened_files`

    已使用`my_open()`（一个`mysys`库函数）打开的文件数量。服务器的部分打开文件而不使用此函数的部分不会增加计数。

+   `Opened_table_definitions`

    已缓存的表定义数量。

+   `Opened_tables`

    已打开的表数量。如果`Opened_tables`很大，则您的`table_open_cache`值可能太小。

+   `Performance_schema_*`xxx`*`

    第 29.16 节，“性能模式状态变量”中列出了性能模式状态变量。这些变量提供了关于由于内存限制而无法加载或创建的仪器信息。

+   `Prepared_stmt_count`

    当前准备语句的数量。（最大语句数由`max_prepared_stmt_count`系统变量给出。）

+   `Queries`

    服务器执行的语句数量。此变量包括存储程序内执行的语句，不同于`Questions`变量。它不计算`COM_PING`或`COM_STATISTICS`命令。

    本节开头的讨论说明了如何将此语句计数状态变量与其他类似变量相关联。

+   `Questions`

    服务器执行的语句数量。这仅包括由客户端发送到服务器的语句，不包括存储程序内执行的语句，与`Queries`变量不同。此变量不计算`COM_PING`、`COM_STATISTICS`、`COM_STMT_PREPARE`、`COM_STMT_CLOSE`或`COM_STMT_RESET`命令。

    本节开头的讨论说明了如何将此语句计数状态变量与其他类似变量相关联。

+   `Replica_open_temp_tables`

    从 MySQL 8.0.26 开始，使用`Replica_open_temp_tables`代替`Slave_open_temp_tables`，后者从该版本开始已被弃用。在 MySQL 8.0.26 之前的版本中，请使用`Slave_open_temp_tables`。

    `Replica_open_temp_tables`显示了复制 SQL 线程当前打开的临时表的数量。如果值大于零，则关闭复制品是不安全的；请参阅第 19.5.1.31 节，“复制和临时表”。此变量报告了*所有*复制通道的打开临时表的总数。

+   `Replica_rows_last_search_algorithm_used`

    从 MySQL 8.0.26 开始，使用`Replica_rows_last_search_algorithm_used`代替`Slave_rows_last_search_algorithm_used`，后者从该版本开始已被弃用。在 MySQL 8.0.26 之前的版本中，请使用`Slave_rows_last_search_algorithm_used`。

    `Replica_rows_last_search_algorithm_used`显示了此复制品最近用于定位行的搜索算法。结果显示了复制品在任何通道上执行的最后一个事务中使用的搜索算法是使用索引、表扫描还是哈希。

    使用的方法取决于`slave_rows_search_algorithms`系统变量的设置（现已弃用），以及相关表上可用的键。

    此变量仅适用于 MySQL 的调试版本。

+   `Resource_group_supported`

    指示资源组功能是否受支持。

    在某些平台或 MySQL 服务器配置中，资源组可能不可用或存在限制。特别是，Linux 系统可能需要一些安装方法的手动步骤。有关详细信息，请参阅资源组限制。

+   `Rpl_semi_sync_master_clients`

    半同步副本的数量。

    当在副本上安装了`rpl_semi_sync_master`（`semisync_master.so`库）插件以设置半同步复制时，`Rpl_semi_sync_master_clients`可用。如果安装了`rpl_semi_sync_source`插件（`semisync_source.so`库），则可用`Rpl_semi_sync_source_clients`。

+   `Rpl_semi_sync_master_net_avg_wait_time`

    源等待复制回复的平均时间（以微秒为单位）。这个变量始终为`0`，已被弃用；预计将在将来的版本中移除。

    当在副本上安装了`rpl_semi_sync_master`（`semisync_master.so`库）插件以设置半同步复制时，`Rpl_semi_sync_master_net_avg_wait_time`可用。如果安装了`rpl_semi_sync_source`插件（`semisync_source.so`库），则可用`Rpl_semi_sync_source_net_avg_wait_time`。

+   `Rpl_semi_sync_master_net_wait_time`

    源等待复制回复的总时间（以微秒为单位）。这个变量始终为`0`，已被弃用；预计将在将来的版本中移除。

    当在副本上安装了`rpl_semi_sync_master`（`semisync_master.so`库）插件以设置半同步复制时，`Rpl_semi_sync_master_net_wait_time`可用。如果安装了`rpl_semi_sync_source`插件（`semisync_source.so`库），则可用`Rpl_semi_sync_source_net_wait_time`。

+   `Rpl_semi_sync_master_net_waits`

    源等待复制回复的总次数。

    当在副本上安装了`rpl_semi_sync_master`（`semisync_master.so`库）插件以建立半同步复制时，`Rpl_semi_sync_master_net_waits`可用。如果安装了`rpl_semi_sync_source`插件（`semisync_source.so`库），则可用`Rpl_semi_sync_source_net_waits`。

+   `Rpl_semi_sync_master_no_times`

    源头关闭半同步复制的次数。

    当在副本上安装了`rpl_semi_sync_master`（`semisync_master.so`库）插件以建立半同步复制时，`Rpl_semi_sync_master_no_times`可用。如果安装了`rpl_semi_sync_source`插件（`semisync_source.so`库），则可用`Rpl_semi_sync_source_no_times`。

+   `Rpl_semi_sync_master_no_tx`

    未被副本成功确认的提交次数。

    当在副本上安装了`rpl_semi_sync_master`（`semisync_master.so`库）插件以建立半同步复制时，`Rpl_semi_sync_master_no_tx`可用。如果安装了`rpl_semi_sync_source`插件（`semisync_source.so`库），则可用`Rpl_semi_sync_source_no_tx`。

+   `Rpl_semi_sync_master_status`

    源头当前是否在运行半同步复制。如果插件已启用并发生了提交确认，则值为`ON`。如果插件未启用或源头由于提交确认超时而回退到异步复制，则值为`OFF`。

    当在副本上安装了`rpl_semi_sync_master`（`semisync_master.so`库）插件以建立半同步复制时，`Rpl_semi_sync_master_status`可用。如果安装了`rpl_semi_sync_source`插件（`semisync_source.so`库），则可用`Rpl_semi_sync_source_status`。

+   `Rpl_semi_sync_master_timefunc_failures`

    源头在调用`gettimeofday()`等时间函数时失败的次数。

    `Rpl_semi_sync_master_timefunc_failures` 在副本上安装了`rpl_semi_sync_master`（`semisync_master.so`库）插件以设置半同步复制时可用。如果安装了`rpl_semi_sync_source`插件（`semisync_source.so`库），则可用`Rpl_semi_sync_source_timefunc_failures`。

+   `Rpl_semi_sync_master_tx_avg_wait_time`

    源等待每个事务的平均时间（以微秒为单位）。

    `Rpl_semi_sync_master_tx_avg_wait_time` 在副本上安装了`rpl_semi_sync_master`（`semisync_master.so`库）插件以设置半同步复制时可用。如果安装了`rpl_semi_sync_source`插件（`semisync_source.so`库），则可用`Rpl_semi_sync_source_tx_avg_wait_time`。

+   `Rpl_semi_sync_master_tx_wait_time`

    源等待事务的总时间（以微秒为单位）。

    `Rpl_semi_sync_master_tx_wait_time` 在副本上安装了`rpl_semi_sync_master`（`semisync_master.so`库）插件以设置半同步复制时可用。如果安装了`rpl_semi_sync_source`插件（`semisync_source.so`库），则可用`Rpl_semi_sync_source_tx_wait_time`。

+   `Rpl_semi_sync_master_tx_waits`

    源等待事务的总次数。

    `Rpl_semi_sync_master_tx_waits` 在副本上安装了`rpl_semi_sync_master`（`semisync_master.so`库）插件以设置半同步复制时可用。如果安装了`rpl_semi_sync_source`插件（`semisync_source.so`库），则可用`Rpl_semi_sync_source_tx_waits`。

+   `Rpl_semi_sync_master_wait_pos_backtraverse`

    源等待事件的总次数，其二进制坐标低于先前等待的事件。当事务开始等待回复的顺序与它们的二进制日志事件写入顺序不同时，可能会发生这种情况。

    当在副本端安装了`rpl_semi_sync_master`（`semisync_master.so`库）插件以设置半同步复制时，`Rpl_semi_sync_master_wait_pos_backtraverse`可用。如果安装了`rpl_semi_sync_source`插件（`semisync_source.so`库），则可用`Rpl_semi_sync_source_wait_pos_backtraverse`。

+   `Rpl_semi_sync_master_wait_sessions`

    当前等待副本回复的会话数量。

    当在副本端安装了`rpl_semi_sync_master`（`semisync_master.so`库）插件以设置半同步复制时，`Rpl_semi_sync_master_wait_sessions`可用。如果安装了`rpl_semi_sync_source`插件（`semisync_source.so`库），则可用`Rpl_semi_sync_source_wait_sessions`。

+   `Rpl_semi_sync_master_yes_tx`

    已被副本成功确认的提交数量。

    当在副本端安装了`rpl_semi_sync_master`（`semisync_master.so`库）插件以设置半同步复制时，`Rpl_semi_sync_master_yes_tx`可用。如果安装了`rpl_semi_sync_source`插件（`semisync_source.so`库），则可用`Rpl_semi_sync_source_yes_tx`。

+   `Rpl_semi_sync_source_clients`

    半同步副本的数量。

    当在源端安装了`rpl_semi_sync_source`（`semisync_source.so`库）插件以设置半同步复制时，`Rpl_semi_sync_source_clients`可用。如果安装了`rpl_semi_sync_master`插件（`semisync_master.so`库），则可用`Rpl_semi_sync_master_clients`。

+   `Rpl_semi_sync_source_net_avg_wait_time`

    源端等待副本回复的平均时间（以微秒为单位）。此变量始终为`0`，已被弃用；预计将在将来的版本中移除。

    当在源端安装了`rpl_semi_sync_source`（`semisync_source.so`库）插件以设置半同步复制时，可以使用`Rpl_semi_sync_source_net_avg_wait_time`。如果安装了`rpl_semi_sync_master`插件（`semisync_master.so`库），则可以使用`Rpl_semi_sync_master_net_avg_wait_time`。

+   `Rpl_semi_sync_source_net_wait_time`

    源端等待副本回复的总时间（以微秒为单位）。此变量始终为`0`，已被弃用；预计将在将来的版本中移除。

    当在源端安装了`rpl_semi_sync_source`（`semisync_source.so`库）插件以设置半同步复制时，可以使用`Rpl_semi_sync_source_net_wait_time`。如果安装了`rpl_semi_sync_master`插件（`semisync_master.so`库），则可以使用`Rpl_semi_sync_master_net_wait_time`。

+   `Rpl_semi_sync_source_net_waits`

    源端等待副本回复的总次数。

    当在源端安装了`rpl_semi_sync_source`（`semisync_source.so`库）插件以设置半同步复制时，可以使用`Rpl_semi_sync_source_net_waits`。如果安装了`rpl_semi_sync_master`插件（`semisync_master.so`库），则可以使用`Rpl_semi_sync_master_net_waits`。

+   `Rpl_semi_sync_source_no_times`

    源端关闭半同步复制的次数。

    当在源端安装了`rpl_semi_sync_source`（`semisync_source.so`库）插件以设置半同步复制时，可以使用`Rpl_semi_sync_source_no_times`。如果安装了`rpl_semi_sync_master`插件（`semisync_master.so`库），则可以使用`Rpl_semi_sync_master_no_times`。

+   `Rpl_semi_sync_source_no_tx`

    未被副本成功确认的提交次数。

    `Rpl_semi_sync_source_no_tx` 是在源端安装了`rpl_semi_sync_source`（`semisync_source.so`库）插件以设置半同步复制时可用的。如果安装了`rpl_semi_sync_master`插件（`semisync_master.so`库），则会使用`Rpl_semi_sync_master_no_tx`。

+   `Rpl_semi_sync_source_status`

    源端当前是否正在运行半同步复制。如果插件已启用并发生了提交确认，则值为`ON`。如果插件未启用或源端由于提交确认超时而回退到异步复制，则值为`OFF`。

    `Rpl_semi_sync_source_status` 是在源端安装了`rpl_semi_sync_source`（`semisync_source.so`库）插件以设置半同步复制时可用的。如果安装了`rpl_semi_sync_master`插件（`semisync_master.so`库），则会使用`Rpl_semi_sync_master_status`。

+   `Rpl_semi_sync_source_timefunc_failures`

    源端调用诸如`gettimeofday()`等时间函数失败的次数。

    `Rpl_semi_sync_source_timefunc_failures` 是在源端安装了`rpl_semi_sync_source`（`semisync_source.so`库）插件以设置半同步复制时可用的。如果安装了`rpl_semi_sync_master`插件（`semisync_master.so`库），则会使用`Rpl_semi_sync_master_timefunc_failures`。

+   `Rpl_semi_sync_source_tx_avg_wait_time`

    源端等待每个事务的平均时间（以微秒为单位）。

    `Rpl_semi_sync_source_tx_avg_wait_time` 是在源端安装了`rpl_semi_sync_source`（`semisync_source.so`库）插件以设置半同步复制时可用的。如果安装了`rpl_semi_sync_master`插件（`semisync_master.so`库），则会使用`Rpl_semi_sync_master_tx_avg_wait_time`。

+   `Rpl_semi_sync_source_tx_wait_time`

    源端等待事务的总时间（以微秒为单位）。

    当在源端安装了`rpl_semi_sync_source`（`semisync_source.so`库）插件以设置半同步复制时，就会出现`Rpl_semi_sync_source_tx_wait_time`。如果安装了`rpl_semi_sync_master`插件（`semisync_master.so`库），则会出现`Rpl_semi_sync_master_tx_wait_time`。

+   `Rpl_semi_sync_source_tx_waits`

    源端等待事务的总次数。

    当在源端安装了`rpl_semi_sync_source`（`semisync_source.so`库）插件以设置半同步复制时，就会出现`Rpl_semi_sync_source_tx_waits`。如果安装了`rpl_semi_sync_master`插件（`semisync_master.so`库），则会出现`Rpl_semi_sync_master_tx_waits`。

+   `Rpl_semi_sync_source_wait_pos_backtraverse`

    源端等待具有比先前等待的事件更低的二进制坐标的事件的总次数。当事务开始等待回复的顺序与它们的二进制日志事件写入的顺序不同时，就会发生这种情况。

    `Rpl_semi_sync_source_wait_pos_backtraverse`是在源端安装了`rpl_semi_sync_source`（`semisync_source.so`库）插件以设置半同步复制时才会出现。如果安装了`rpl_semi_sync_master`插件（`semisync_master.so`库），则会出现`Rpl_semi_sync_master_wait_pos_backtraverse`。

+   `Rpl_semi_sync_source_wait_sessions`

    当前等待副本回复的会话数。

    当在源端安装了`rpl_semi_sync_source`（`semisync_source.so`库）插件以设置半同步复制时，就会出现`Rpl_semi_sync_source_wait_sessions`。如果安装了`rpl_semi_sync_master`插件（`semisync_master.so`库），则会出现`Rpl_semi_sync_master_wait_sessions`。

+   `Rpl_semi_sync_source_yes_tx`

    副本成功确认的提交次数。

    当在源上安装了`rpl_semi_sync_source`（`semisync_source.so`库）插件以设置半同步复制时，`Rpl_semi_sync_source_yes_tx`可用。如果安装了`rpl_semi_sync_master`插件（`semisync_master.so`库），则可用`Rpl_semi_sync_master_yes_tx`。

+   `Rpl_semi_sync_replica_status`

    显示复制品上当前是否正在运行半同步复制。如果插件已启用且复制 I/O（接收器）线程正在运行，则为`ON`，否则为`OFF`。

    当在复制品上安装了`rpl_semi_sync_replica`（`semisync_replica.so`库）插件以设置半同步复制时，`Rpl_semi_sync_replica_status`可用。如果安装了`rpl_semi_sync_slave`插件（`semisync_slave.so`库），则可用`Rpl_semi_sync_slave_status`。

+   `Rpl_semi_sync_slave_status`

    显示复制品上当前是否正在运行半同步复制。如果插件已启用且复制 I/O（接收器）线程正在运行，则为`ON`，否则为`OFF`。

    当在复制品上安装了`rpl_semi_sync_slave`（`semisync_slave.so`库）插件以设置半同步复制时，`Rpl_semi_sync_slave_status`可用。如果安装了`rpl_semi_sync_replica`插件（`semisync_replica.so`库），则可用`Rpl_semi_sync_replica_status`。

+   `Rsa_public_key`

    此变量的值是`sha256_password`认证插件用于 RSA 密钥对密码交换的公钥。仅当服务器成功初始化由`sha256_password_private_key_path`和`sha256_password_public_key_path`系统变量命名的文件中的私钥和公钥时，值才不为空。`Rsa_public_key`的值来自后一个文件。

    有关`sha256_password`的信息，请参见第 8.4.1.3 节，“SHA-256 可插拔认证”。

+   `Secondary_engine_execution_count`

    转移到辅助引擎的查询数量。此变量在 MySQL 8.0.13 中添加。

    用于 HeatWave。请参阅 MySQL HeatWave 用户指南。

+   `Select_full_join`

    执行表扫描的连接数，因为它们不使用索引。如果此值不为 0，则应仔细检查表的索引。

+   `Select_full_range_join`

    使用范围搜索的连接表的数量。

+   `Select_range`

    使用第一个表范围的连接数。即使该值很大，通常也不是关键问题。

+   `Select_range_check`

    在每行后检查键使用情况的无键连接数。如果此值不为 0，则应仔细检查表的索引。

+   `Select_scan`

    对第一个表进行全面扫描的连接数。

+   `Slave_open_temp_tables`

    从 MySQL 8.0.26 开始，`Slave_open_temp_tables` 已弃用，应改用别名 `Replica_open_temp_tables`。在 MySQL 8.0.26 之前的版本中，请使用 `Slave_open_temp_tables`。

    `Slave_open_temp_tables` 显示复制 SQL 线程当前打开的临时表的数量。如果该值大于零，则不安全关闭副本；请参见 第 19.5.1.31 节，“复制和临时表”。此变量报告所有复制通道的打开临时表的总数。

+   `Slave_rows_last_search_algorithm_used`

    从 MySQL 8.0.26 开始，`Slave_rows_last_search_algorithm_used` 已弃用，应改用别名 `Replica_rows_last_search_algorithm_used`。在 MySQL 8.0.26 之前的版本中，请使用 `Slave_rows_last_search_algorithm_used`。

    `Slave_rows_last_search_algorithm_used` 显示此复制品最近用于定位行以进行基于行的复制的搜索算法。结果显示复制品是否在任何通道上执行的最后一个事务中使用索引、表扫描或哈希作为搜索算法。

    使用取决于`slave_rows_search_algorithms`系统变量的设置以及相关表上可用的密钥。

    此变量仅适用于 MySQL 的调试版本。

+   `Slow_launch_threads`

    超过`slow_launch_time`秒创建的线程数。

+   `Slow_queries`

    超过`long_query_time`秒的查询次数。无论是否启用慢查询日志，此计数器都会递增。有关该日志的信息，请参阅第 7.4.5 节，“慢查询日志”。

+   `Sort_merge_passes`

    排序算法已经执行的合并次数。如果这个值很大，您应该考虑增加`sort_buffer_size`系统变量的值。

+   `Sort_range`

    使用范围进行的排序次数。

+   `Sort_rows`

    已排序行的数量。

+   `Sort_scan`

    通过扫描表进行的排序次数。

+   `Ssl_accept_renegotiates`

    建立连接所需的协商次数。

+   `Ssl_accepts`

    已接受的 SSL 连接数量。

+   `Ssl_callback_cache_hits`

    回调缓存命中次数。

+   `Ssl_cipher`

    当前的加密密码（未加密连接为空）。

+   `Ssl_cipher_list`

    可能的 SSL 密码列表（非 SSL 连接为空）。如果 MySQL 支持 TLSv1.3，则该值包括可能的 TLSv1.3 密码套件。请参阅第 8.3.2 节，“加密连接 TLS 协议和密码”。

+   `Ssl_client_connects`

    尝试连接到启用 SSL 的复制源服务器的 SSL 连接次数。

+   `Ssl_connect_renegotiates`

    建立到启用 SSL 的复制源服务器的连接所需的协商次数。

+   `Ssl_ctx_verify_depth`

    SSL 上下文验证深度（测试链中有多少证书）。

+   `Ssl_ctx_verify_mode`

    SSL 上下文验证模式。

+   `Ssl_default_timeout`

    默认 SSL 超时时间。

+   `Ssl_finished_accepts`

    成功连接到服务器的 SSL 连接数。

+   `Ssl_finished_connects`

    成功复制到启用 SSL 的复制源服务器的连接数。

+   `Ssl_server_not_after`

    SSL 证书有效的最后日期。要检查 SSL 证书到期信息，请使用此语句：

    ```sql
    mysql> SHOW STATUS LIKE 'Ssl_server_not%';
    +-----------------------+--------------------------+
    | Variable_name         | Value                    |
    +-----------------------+--------------------------+
    | Ssl_server_not_after  | Apr 28 14:16:39 2025 GMT |
    | Ssl_server_not_before | May  1 14:16:39 2015 GMT |
    +-----------------------+--------------------------+
    ```

+   `Ssl_server_not_before`

    SSL 证书有效的第一个日期。

+   `Ssl_session_cache_hits`

    SSL 会话缓存命中次数。

+   `Ssl_session_cache_misses`

    SSL 会话缓存未命中次数。

+   `Ssl_session_cache_mode`

    SSL 会话缓存模式。当`ssl_session_cache_mode`服务器变量的值为`ON`时，`Ssl_session_cache_mode`状态变量的值为`SERVER`。

+   `Ssl_session_cache_overflows`

    SSL 会话缓存溢出次数。

+   `Ssl_session_cache_size`

    SSL 会话缓存大小。

+   `Ssl_session_cache_timeout`

    缓存中 SSL 会话的超时值（以秒为单位）。

+   `Ssl_session_cache_timeouts`

    SSL 会话缓存超时次数。

+   `Ssl_sessions_reused`

    如果当前 MySQL 会话中未使用 TLS，或者未重用 TLS 会话，则此值为 0；否则为 1。

    `Ssl_sessions_reused`具有会话范围。

+   `Ssl_used_session_cache_entries`

    使用了多少 SSL 会话缓存条目。

+   `Ssl_verify_depth`

    复制 SSL 连接的验证深度。

+   `Ssl_verify_mode`

    服务器用于使用 SSL 的连接的验证模式。该值是一个位掩码；位在`openssl/ssl.h`头文件中定义：

    ```sql
    # define SSL_VERIFY_NONE                 0x00
    # define SSL_VERIFY_PEER                 0x01
    # define SSL_VERIFY_FAIL_IF_NO_PEER_CERT 0x02
    # define SSL_VERIFY_CLIENT_ONCE          0x04
    ```

    `SSL_VERIFY_PEER`表示服务器要求客户端证书。如果客户端提供了证书，服务器将进行验证，并仅在验证成功时继续。`SSL_VERIFY_CLIENT_ONCE`表示仅在初始握手中执行对客户端证书的请求。

+   `Ssl_version`

    连接的 SSL 协议版本（例如，TLSv1）。如果连接未加密，则该值为空。

+   `Table_locks_immediate`

    申请表锁可以立即获得的次数。

+   `Table_locks_waited`

    申请表锁无法立即获得并需要等待的次数。如果这个数字很高并且出现性能问题，你应该首先优化你的查询，然后要么拆分你的表或表，要么使用复制。

+   `Table_open_cache_hits`

    开放表缓存查找的命中次数。

+   `Table_open_cache_misses`

    开放表缓存查找的未命中次数。

+   `Table_open_cache_overflows`

    开放表缓存的溢出次数。这是在打开或关闭表后，缓存实例有一个未使用的条目且实例的大小大于`table_open_cache` / `table_open_cache_instances`时的次数。

+   `Tc_log_max_pages_used`

    对于当**mysqld**充当内部 XA 事务恢复的事务协调器时使用的日志的内存映射实现，此变量指示自服务器启动以来日志使用的最大页数。如果`Tc_log_max_pages_used`和`Tc_log_page_size`的乘积始终明显小于日志大小，则大小大于必要大小，可以减小。（大小由`--log-tc-size`选项设置。此变量未使用：对于基于二进制日志的恢复是不需要的，并且除非能够进行两阶段提交并支持 XA 事务的存储引擎数量大于一个，否则不会使用内存映射恢复日志方法。（`InnoDB`是唯一适用的引擎。）

+   `Tc_log_page_size`

    用于 XA 恢复日志的内存映射实现的页面大小。默认值使用`getpagesize()`确定。出于与`Tc_log_max_pages_used`描述的相同原因，此变量未使用。

+   `Tc_log_page_waits`

    对于恢复日志的内存映射实现，每次服务器无法提交事务并必须等待日志中的空闲页时，此变量会递增。如果此值较大，您可能需要增加日志大小（使用`--log-tc-size`选项）。对于基于二进制日志的恢复，每次二进制日志无法关闭因为有两阶段提交正在进行时，此变量会递增。（关闭操作会等待所有此类事务完成。）

+   `Telemetry_traces_supported`

    服务器遥测跟踪是否受支持。

    有关更多信息，请参阅 MySQL 源代码文档中的*服务器遥测跟踪服务*部分。

+   `Threads_cached`

    线程缓存中的线程数。

+   `Threads_connected`

    当前打开连接的数量。

+   `Threads_created`

    用于处理连接创建的线程数。如果`Threads_created`较大，您可能需要增加`thread_cache_size`的值。缓存未命中率可计算为`Threads_created`/`Connections`。

+   `Threads_running`

    非休眠状态的线程数。

+   `Tls_library_version`

    此 MySQL 实例中使用的 OpenSSL 库的运行时版本。

    此变量是在 MySQL 8.0.30 中添加的。

+   `Uptime`

    服务器已经运行的秒数。

+   `Uptime_since_flush_status`

    距离最近一次`FLUSH STATUS`语句的秒数。
