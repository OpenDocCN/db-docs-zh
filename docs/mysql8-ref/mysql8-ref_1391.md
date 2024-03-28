> 译文：[`dev.mysql.com/doc/refman/8.0/en/replica-logs-status.html`](https://dev.mysql.com/doc/refman/8.0/en/replica-logs-status.html)

#### 19.2.4.2 复制元数据存储库

复制服务器创建两个复制元数据存储库，连接元数据存储库和应用程序元数据存储库。复制元数据存储库在复制服务器关闭时保留。如果使用基于二进制日志文件位置的复制，则在复制重新启动时，它会读取这两个存储库，以确定它以前在读取源的二进制日志和处理自己的中继日志方面进行了多远。如果使用基于 GTID 的复制，则复制不会使用复制元数据存储库进行此目的，但确实需要其中包含的其他元数据。

+   复制的*连接元数据存储库*包含复制 I/O（接收器）线程需要连接到复制源服务器并从源的二进制日志中检索事务的信息。该存储库中的元数据包括连接配置、复制用户帐户详细信息、连接的 SSL 设置，以及复制接收器线程当前正在从源的二进制日志中读取的文件名和位置。

+   复制的*应用程序元数据存储库*包含复制 SQL（应用程序）线程需要读取并应用来自复制的中继日志的事务的信息。该存储库中的元数据包括复制应用程序线程已执行中继日志中的事务的文件名和位置，以及源二进制日志中的等效位置。它还包括应用事务过程的元数据，例如工作线程数和通道的`PRIVILEGE_CHECKS_USER`帐户。

连接元数据存储库写入`mysql`系统模式中的`slave_master_info`表中，而应用程序元数据存储库写入`mysql`系统模式中的`slave_relay_log_info`表中。如果**mysqld**无法初始化用于复制元数据存储库的表，则会发出警告消息，但允许复制继续启动。当从不支持使用表作为存储库的版本升级到支持表的版本时，最有可能发生这种情况。

重要

1.  不要尝试手动更新或插入`mysql.slave_master_info`或`mysql.slave_relay_log_info`表中的行。这样做可能导致未定义的行为，并且不受支持。在复制正在进行时，不允许执行需要对`slave_master_info`和`slave_relay_log_info`表中的任一表或两个表进行写锁定的任何语句（尽管允许随时执行仅执行读取的语句）。

1.  对于连接元数据存储库表`mysql.slave_master_info`的访问权限应该限制在数据库管理员，因为它包含用于连接到源的复制用户帐户名和密码。使用受限访问模式来保护包含此表的数据库备份。从 MySQL 8.0.21 开始，您可以从连接元数据存储库中清除复制用户帐户凭据，并始终使用`START REPLICA`语句或`START GROUP_REPLICATION`语句提供它们。这种方法意味着复制通道始终需要操作员干预才能重新启动，但帐户名和密码不会记录在复制元数据存储库中。

`RESET REPLICA`会清除复制元数据存储库中的数据，但不包括复制连接参数（取决于 MySQL Server 版本）。有关详细信息，请参阅`RESET REPLICA`的描述。

从 MySQL 8.0.27 开始，您可以在`CHANGE REPLICATION SOURCE TO`语句上设置`GTID_ONLY`选项，以阻止复制通道将文件名和文件位置持久化到复制元数据存储库中。这样可以避免在 GTID 基础复制实际上不需要它们的情况下对表进行写入和读取。使用`GTID_ONLY`设置时，当副本队列和应用程序在事务中排队和应用事件，或者当复制线程停止和启动时，连接元数据存储库和应用程序元数据存储库不会被更新。文件位置在内存中进行跟踪，并且如果需要，可以使用`SHOW REPLICA STATUS`语句查看。复制元数据存储库仅在以下情况下同步：

+   当发出`CHANGE REPLICATION SOURCE TO`语句时。

+   当发出`RESET REPLICA`语句时。`RESET REPLICA ALL`会删除而不是更新存储库，因此它们会隐式同步。

+   当初始化复制通道时。

+   如果将复制元数据存储库从文件移动到表中。

在 MySQL 8.0 之前，要将复制元数据存储库创建为表，需要在服务器启动时指定`master_info_repository=TABLE`和`relay_log_info_repository=TABLE`。否则，存储库将被创建为数据目录中的文件，命名为`master.info`和`relay-log.info`，或者使用`--master-info-file`选项和`relay_log_info_file`系统变量指定的替代名称和位置。从 MySQL 8.0 开始，默认情况下将复制元数据存储库创建为表，并且所有这些系统变量的使用已被弃用。

`mysql.slave_master_info`和`mysql.slave_relay_log_info`表是使用`InnoDB`事务存储引擎创建的。对应用程序元数据存储库表的更新与事务一起提交，这意味着记录在该存储库中的副本进度信息始终与已应用于数据库的内容一致，即使在发生意外服务器停机的情况下也是如此。有关在副本上设置的组合对于意外停机最具弹性的信息，请参阅 Section 19.4.2, “处理副本意外停机”。

当您备份副本的数据或传输其数据的快照以创建新副本时，请确保包括包含复制元数据存储库的`mysql.slave_master_info`和`mysql.slave_relay_log_info`表。对于克隆操作，请注意，当复制元数据存储库被创建为表时，在克隆操作期间会将其复制到接收方，但当它们被创建为文件时，则不会被复制。当使用基于二进制日志文件位置的复制时，需要复制元数据存储库以在重新启动已恢复、复制或克隆的副本后恢复复制。如果您没有中继日志文件，但仍然有应用程序元数据存储库，则可以检查它以确定复制 SQL 线程在源二进制日志中执行到哪个程度。然后，您可以使用`CHANGE REPLICATION SOURCE TO`语句（从 MySQL 8.0.23 开始）或`CHANGE MASTER TO`语句（在 MySQL 8.0.23 之前）与`SOURCE_LOG_FILE` | `MASTER_LOG_FILE`和`SOURCE_LOG_POS` | `MASTER_LOG_POS`选项告诉副本从该点重新读取源的二进制日志（前提是所需的二进制日志仍然存在于源）。

另外创建了一个附加的存储库，即应用程序工作程序元数据存储库，主要用于内部使用，并保存有关多线程复制品上工作线程的状态信息。应用程序工作程序元数据存储库包括每个工作线程的中继日志文件和源二进制日志文件的名称和位置。如果将应用程序元数据存储库创建为表（这是默认设置），则将应用程序工作程序元数据存储库写入`mysql.slave_worker_info`表中。如果将应用程序元数据存储库写入文件，则将应用程序工作程序元数据存储库写入`worker-relay-log.info`文件中。对于外部使用，工作线程的状态信息显示在性能模式`replication_applier_status_by_worker`中。

复制元数据存储库最初包含类似于`SHOW REPLICA STATUS`语句输出中显示的信息，该语句在第 15.4.2 节“用于控制复制服务器的 SQL 语句”中讨论。此后，复制元数据存储库中添加了更多信息，这些信息不会被`SHOW REPLICA STATUS`语句显示。

对于连接元数据存储库，以下表显示了`mysql.slave_master_info`表中的列与`SHOW REPLICA STATUS`显示的列以及已弃用的`master.info`文件中的行之间的对应关系。

| `slave_master_info` 表列 | `SHOW REPLICA STATUS` 列 | `master.info` 文件行 | 描述 |
| --- | --- | --- | --- |
| `Number_of_lines` | [无] | 1 | 表中的列数（或文件中的行数） |
| `Master_log_name` | `Source_Log_File` | 2 | 当前正在从源读取的二进制日志的名称 |
| `Master_log_pos` | `Read_Source_Log_Pos` | 3 | 已从源读取的二进制日志中的当前位置 |
| `Host` | `Source_Host` | 4 | 复制源服务器的主机名 |
| `User_name` | `Source_User` | 5 | 用于连接到源的复制用户帐户名称 |
| `User_password` | 密码（`SHOW REPLICA STATUS`不显示） | 6 | 用于连接到源的复制用户帐户密码 |
| `Port` | `Source_Port` | 7 | 用于连接到复制源服务器的网络端口 |
| `Connect_retry` | `Connect_Retry` | 8 | 复制品在尝试重新连接到源之前等待的时间段（以秒为单位） |
| `Enabled_ssl` | `Source_SSL_Allowed` | 9 | 复制端是否支持 SSL 连接 |
| `Ssl_ca` | `Source_SSL_CA_File` | 10 | 用于证书颁发机构（CA）证书的文件 |
| `Ssl_capath` | `Source_SSL_CA_Path` | 11 | 证书颁发机构（CA）证书的路径 |
| `Ssl_cert` | `Source_SSL_Cert` | 12 | SSL 证书文件的名称 |
| `Ssl_cipher` | `Source_SSL_Cipher` | 13 | SSL 连接握手中可能使用的密码列表 |
| `Ssl_key` | `Source_SSL_Key` | 14 | SSL 密钥文件的名称 |
| `Ssl_verify_server_cert` | `Source_SSL_Verify_Server_Cert` | 15 | 是否验证服务器证书 |
| `Heartbeat` | [无] | 16 | 复制心跳之间的间隔时间，单位为秒 |
| `Bind` | `Source_Bind` | 17 | 连接源端时应使用的复制端网络接口 |
| `Ignored_server_ids` | `Replicate_Ignore_Server_Ids` | 18 | 要忽略的服务器 ID 列表。请注意，对于`Ignored_server_ids`，服务器 ID 列表前面会有要忽略的服务器 ID 总数。 |
| `Uuid` | `Source_UUID` | 19 | 源端的唯一 ID |
| `Retry_count` | `Source_Retry_Count` | 20 | 允许的最大重连尝试次数 |
| `Ssl_crl` | [无] | 21 | SSL 证书吊销列表文件的路径 |
| `Ssl_crlpath` | [无] | 22 | 包含 SSL 证书吊销列表文件的目录路径 |
| `Enabled_auto_position` | `Auto_position` | 23 | 是否使用 GTID 自动定位 |
| `Channel_name` | `Channel_name` | 24 | 复制通道的名称 |
| `Tls_version` | `Source_TLS_Version` | 25 | 源端的 TLS 版本 |
| `Public_key_path` | `Source_public_key_path` | 26 | RSA 公钥文件的名称 |
| `Get_public_key` | `Get_source_public_key` | 27 | 是否从源端请求 RSA 公钥 |
| `Network_namespace` | `Network_namespace` | 28 | 网络命名空间 |
| `Master_compression_algorithm` | [无] | 29 | 与源端连接的允许压缩算法 |
| `Master_zstd_compression_level` | [无] | 30 | `zstd`压缩级别 |
| `Tls_ciphersuites` | [无] | 31 | TLSv1.3 允许的密码套件 |
| `Source_connection_auto_failover` | [无] | 32 | 是否激活异步连接故障转移机制 |
| `Gtid_only` | [无] | 33 | 通道是否仅使用 GTID 并且不保留位置 |
| `slave_master_info` 表列 | `SHOW REPLICA STATUS` 列 | `master.info` 文件行 | 描述 |

对于应用程序元数据存储库，以下表格显示了`mysql.slave_relay_log_info`表中的列与`SHOW REPLICA STATUS`显示的列以及已弃用的`relay-log.info`文件中的行之间的对应关系。

| `slave_relay_log_info` 表列 | `SHOW REPLICA STATUS` 列 | `relay-log.info` 文件行 | 描述 |
| --- | --- | --- | --- |
| `Number_of_lines` | [无] | 1 | 表中的列数或文件中的行数 |
| `Relay_log_name` | `中继日志文件` | 2 | 当前中继日志文件的名称 |
| `Relay_log_pos` | `Relay_Log_Pos` | 3 | 中继日志文件中的当前位置；到达此位置的事件已在副本数据库上执行 |
| `Master_log_name` | `源二进制日志文件` | 4 | 从中继日志文件中读取事件的源二进制日志文件的名称 |
| `Master_log_pos` | `Exec_Source_Log_Pos` | 5 | 在副本上已执行的事件在源二进制日志文件中的相应位置 |
| `Sql_delay` | `SQL_Delay` | 6 | 副本必须滞后源的秒数 |
| `Number_of_workers` | [无] | 7 | 并行应用复制事务的工作线程数 |
| `Id` | [无] | 8 | 用于内部目的的 ID；目前始终为 1 |
| `Channel_name` | `通道名称` | 9 | 复制通道的名称 |
| `Privilege_checks_username` | [无] | 10 | 通道的`PRIVILEGE_CHECKS_USER`账户的用户名 |
| `Privilege_checks_hostname` | [无] | 11 | 通道的`PRIVILEGE_CHECKS_USER`账户的主机名 |
| `Require_row_format` | [无] | 12 | 通道是否仅接受基于行的事件 |
| `Require_table_primary_key_check` | [无] | 13 | 通道对于`CREATE TABLE`和`ALTER TABLE`操作是否要求表必须有主键的策略 |
| `Assign_gtids_to_anonymous_transactions_type` | [无] | 14 | 如果通道为尚未具有 GTID 的复制事务分配 GTID，使用副本的本地 UUID，则该值为`LOCAL`；如果通道使用手动设置的 UUID 进行分配，则该值为`UUID`。如果通道在这种情况下不分配 GTID，则该值为`OFF`。 |
| `Assign_gtids_to_anonymous_transactions_value` | [无] | 15 | 用于分配给匿名事务的 GTIDs 的 UUID |
| `slave_relay_log_info` 表列 | `SHOW REPLICA STATUS` 列 | `relay-log.info` 文件中的行 | 描述 |
