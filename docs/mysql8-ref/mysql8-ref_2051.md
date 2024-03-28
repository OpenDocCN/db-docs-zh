> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-replication-connection-configuration-table.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-replication-connection-configuration-table.html)

#### 29.12.11.1 replication_connection_configuration 表

此表显示了复制品用于连接到源的配置参数。表中存储的参数可以通过`CHANGE REPLICATION SOURCE TO`语句（从 MySQL 8.0.23 开始）或`CHANGE MASTER TO`语句（在 MySQL 8.0.23 之前）在运行时更改。

与`replication_connection_status`表相比，`replication_connection_configuration`变化较少。它包含定义复制品如何连接到源以及在连接期间保持不变的值，而`replication_connection_status`包含在连接期间更改的值。

`replication_connection_configuration`表具有以下列。列描述指示从中获取列值的对应`CHANGE REPLICATION SOURCE TO` | `CHANGE MASTER TO`选项，并且本节后面给出的表显示了`replication_connection_configuration`列和`SHOW REPLICA STATUS`列之间的对应关系。

+   `通道名称`

    显示此行的复制通道。始终存在一个默认的复制通道，并且可以添加更多的复制通道。有关更多信息，请参见第 19.2.2 节，“复制通道”。

+   `主机`

    复制品连接的源主机名。(`CHANGE REPLICATION SOURCE TO` 选项：`SOURCE_HOST`，`CHANGE MASTER TO` 选项：`MASTER_HOST`)

+   `端口`

    用于连接到源的端口。(`CHANGE REPLICATION SOURCE TO` 选项：`SOURCE_PORT`，`CHANGE MASTER TO` 选项：`MASTER_PORT`)

+   `用户`

    用于连接到源的复制用户帐户的用户名。 （`将复制源更改为`选项：`SOURCE_USER`，`更改主服务器为`选项：`MASTER_USER`）

+   `NETWORK_INTERFACE`

    副本绑定到的网络接口（如果有）。 （`将复制源更改为`选项：`SOURCE_BIND`，`更改主服务器为`选项：`MASTER_BIND`）

+   `AUTO_POSITION`

    如果使用 GTID 自动定位，则为 1；否则为 0。 （`将复制源更改为`选项：`SOURCE_AUTO_POSITION`，`更改主服务器为`选项：`MASTER_AUTO_POSITION`）

+   `SSL_ALLOWED`，`SSL_CA_FILE`，`SSL_CA_PATH`，`SSL_CERTIFICATE`，`SSL_CIPHER`，`SSL_KEY`，`SSL_VERIFY_SERVER_CERTIFICATE`，`SSL_CRL_FILE`，`SSL_CRL_PATH`

    这些列显示副本用于连接到源的 SSL 参数（如果有）。

    `SSL_ALLOWED`具有以下值：

    +   如果允许与源的 SSL 连接，则为`是`

    +   如果不允许与源的 SSL 连接，则为`否`

    +   如果允许 SSL 连接但副本未启用 SSL 支持，则`忽略`

    (`将复制源更改为`其他 SSL 列的选项：`SOURCE_SSL_CA`，`SOURCE_SSL_CAPATH`，`SOURCE_SSL_CERT`，`SOURCE_SSL_CIPHER`，`SOURCE_SSL_CRL`，`SOURCE_SSL_CRLPATH`，`SOURCE_SSL_KEY`，`SOURCE_SSL_VERIFY_SERVER_CERT`。

    `更改主服务器为`其他 SSL 列的选项：`MASTER_SSL_CA`，`MASTER_SSL_CAPATH`，`MASTER_SSL_CERT`，`MASTER_SSL_CIPHER`，`MASTER_SSL_CRL`，`MASTER_SSL_CRLPATH`，`MASTER_SSL_KEY`，`MASTER_SSL_VERIFY_SERVER_CERT`。

+   `CONNECTION_RETRY_INTERVAL`

    连接重试之间的秒数。 （`将复制源更改为`选项：`SOURCE_CONNECT_RETRY`，`更改主服务器为`选项：`MASTER_CONNECT_RETRY`）

+   `CONNECTION_RETRY_COUNT`

    在连接丢失的情况下，副本可以尝试重新连接到源的次数。 （`将复制源更改为`选项：`SOURCE_RETRY_COUNT`，`更改主服务器为`选项：`MASTER_RETRY_COUNT`）

+   `HEARTBEAT_INTERVAL`

    副本上的复制心跳���隔，以秒为单位。 （`将复制源更改为`选项：`SOURCE_HEARTBEAT_PERIOD`，`更改主服务器为`选项：`MASTER_HEARTBEAT_PERIOD`）

+   `TLS_VERSION`

    复制连接中允许副本使用的 TLS 协议版本列表。有关 TLS 版本信息，请参见第 8.3.2 节，“加密连接 TLS 协议和密码”。（`将复制源更改为`选项：`SOURCE_TLS_VERSION`，`更改主服务器为`选项：`MASTER_TLS_VERSION`）

+   `TLS_CIPHERSUITES`

    允许副本使用的密码套件列表。有关 TLS 密码套件信息，请参见第 8.3.2 节，“加密连接 TLS 协议和密码”。（`将复制源更改为`选项：`SOURCE_TLS_CIPHERSUITES`，`更改主服务器为`选项：`MASTER_TLS_CIPHERSUITES`）

+   `PUBLIC_KEY_PATH`

    指向包含源端所需用于 RSA 密钥对密码交换的公钥副本的文件的路径名。文件必须采用 PEM 格式。此列适用于使用`sha256_password`或`caching_sha2_password`认证插件进行身份验证的复制品。 (`CHANGE REPLICATION SOURCE TO`选项：`SOURCE_PUBLIC_KEY_PATH`，`CHANGE MASTER TO`选项：`MASTER_PUBLIC_KEY_PATH`)

    如果给定`PUBLIC_KEY_PATH`并指定有效的公钥文件，则优先于`GET_PUBLIC_KEY`。

+   `GET_PUBLIC_KEY`

    是否从源端请求所需用于 RSA 密钥对密码交换的公钥。此列适用于使用`caching_sha2_password`认证插件进行身份验证的复制品。对于该插件，除非请求，否则源端不会发送公钥。 (`CHANGE REPLICATION SOURCE TO`选项：`GET_SOURCE_PUBLIC_KEY`，`CHANGE MASTER TO`选项：`GET_MASTER_PUBLIC_KEY`)

    如果给定`PUBLIC_KEY_PATH`并指定有效的公钥文件，则优先于`GET_PUBLIC_KEY`。

+   `NETWORK_NAMESPACE`

    网络命名空间名称；如果连接使用默认（全局）命名空间，则为空。有关网络命名空间的信息，请参见第 7.1.14 节，“网络命名空间支持”。此列在 MySQL 8.0.22 中添加。

+   `COMPRESSION_ALGORITHM`

    用于源端连接的允许压缩算法。 (`CHANGE REPLICATION SOURCE TO`选项：`SOURCE_COMPRESSION_ALGORITHMS`，`CHANGE MASTER TO`选项：`MASTER_COMPRESSION_ALGORITHMS`)

    有关更多信息，请参见第 6.2.8 节，“连接压缩控制”。

    此列在 MySQL 8.0.18 中添加。

+   `ZSTD_COMPRESSION_LEVEL`

    用于使用`zstd`压缩算法的源端连接的压缩级别。 (`CHANGE REPLICATION SOURCE TO`选项：`SOURCE_ZSTD_COMPRESSION_LEVEL`，`CHANGE MASTER TO`选项：`MASTER_ZSTD_COMPRESSION_LEVEL`)

    有关更多信息，请参见第 6.2.8 节，“连接压缩控制”。

    此列在 MySQL 8.0.18 中添加。

+   `SOURCE_CONNECTION_AUTO_FAILOVER`

    是否为此复制通道激活异步连接故障转移机制。 (`CHANGE REPLICATION SOURCE TO`选项：`SOURCE_CONNECTION_AUTO_FAILOVER`，`CHANGE MASTER TO`选项：`SOURCE_CONNECTION_AUTO_FAILOVER`)

    有关更多信息，请参见第 19.4.9 节，“使用异步连接故障转移切换源和复制品”。

    此列在 MySQL 8.0.22 中添加。

+   `GTID_ONLY`

    表示此通道仅在事务队列和应用程序过程以及恢复中使用 GTIDs，并且不在复制元数据存储库中保留二进制日志和中继日志文件名和文件位置。 (`CHANGE REPLICATION SOURCE TO` 选项：`GTID_ONLY`，`CHANGE MASTER TO` 选项：`GTID_ONLY`)

    有关更多信息，请参见 第 20.4.1 节，“GTIDs 和组复制”。

    此列在 MySQL 8.0.27 中添加。

`replication_connection_configuration` 表具有以下索引：

+   主键为 (`CHANNEL_NAME`)

`TRUNCATE TABLE` 不允许用于 `replication_connection_configuration` 表。

以下表显示了 `replication_connection_configuration` 列与 `SHOW REPLICA STATUS` 列之间的对应关系。

| `replication_connection_configuration` 列 | `SHOW REPLICA STATUS` 列 |
| --- | --- |
| `CHANNEL_NAME` | `Channel_name` |
| `HOST` | `Source_Host` |
| `PORT` | `Source_Port` |
| `USER` | `Source_User` |
| `NETWORK_INTERFACE` | `Source_Bind` |
| `AUTO_POSITION` | `Auto_Position` |
| `SSL_ALLOWED` | `Source_SSL_Allowed` |
| `SSL_CA_FILE` | `Source_SSL_CA_File` |
| `SSL_CA_PATH` | `Source_SSL_CA_Path` |
| `SSL_CERTIFICATE` | `Source_SSL_Cert` |
| `SSL_CIPHER` | `Source_SSL_Cipher` |
| `SSL_KEY` | `Source_SSL_Key` |
| `SSL_VERIFY_SERVER_CERTIFICATE` | `Source_SSL_Verify_Server_Cert` |
| `SSL_CRL_FILE` | `Source_SSL_Crl` |
| `SSL_CRL_PATH` | `Source_SSL_Crlpath` |
| `CONNECTION_RETRY_INTERVAL` | `Source_Connect_Retry` |
| `CONNECTION_RETRY_COUNT` | `Source_Retry_Count` |
| `HEARTBEAT_INTERVAL` | 无 |
| `TLS_VERSION` | `Source_TLS_Version` |
| `PUBLIC_KEY_PATH` | `Source_public_key_path` |
| `GET_PUBLIC_KEY` | `Get_source_public_key` |
| `NETWORK_NAMESPACE` | `Network_Namespace` |
| `COMPRESSION_ALGORITHM` | [无] |
| `ZSTD_COMPRESSION_LEVEL` | [无] |
| `GTID_ONLY` | [无] |
| `replication_connection_configuration` 列 | `SHOW REPLICA STATUS` 列 |
