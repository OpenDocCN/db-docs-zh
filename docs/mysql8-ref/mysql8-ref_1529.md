> 原文：[`dev.mysql.com/doc/refman/8.0/en/group-replication-configuring-ssl-for-recovery.html`](https://dev.mysql.com/doc/refman/8.0/en/group-replication-configuring-ssl-for-recovery.html)

#### 20.6.3.2 分布式恢复的安全套接字层（SSL）连接

重要

当使用 MySQL 通信堆栈（`group_replication_communication_stack=MYSQL`）和成员之间的安全连接（`group_replication_ssl_mode` 未设置为 `DISABLED`）时，本节讨论的安全设置不仅适用于分布式恢复连接，还适用于成员之间的组通信。请参阅 Section 20.6.1, “Communication Stack for Connection Security Management”。

无论是使用标准的 SQL 客户端连接还是分布式恢复端点进行分布式恢复连接，为了安全配置连接，您可以使用 Group Replication 的专用分布式恢复 SSL 选项。这些选项对应用于组通信连接的服务器 SSL 选项，但仅适用于分布式恢复连接。默认情况下，分布式恢复连接不使用 SSL，即使您为组通信连接激活了 SSL，服务器 SSL 选项也不适用于分布式恢复连接。您必须单独配置这些连接。

如果远程克隆操作作为分布式恢复的一部分使用，则 Group Replication 会自动配置克隆插件的 SSL 选项，以匹配您对分布式恢复 SSL 选项的设置。（有关克隆插件如何使用 SSL 的详细信息，请参阅 为克隆配置加密连接。）

分布式恢复 SSL 选项如下：

+   `group_replication_recovery_use_ssl`：设置为 `ON` 以使 Group Replication 在分布式恢复连接中使用 SSL，包括远程克隆操作和从捐赠者的二进制日志进行状态传输。您可以只设置此选项，而不设置其他分布式恢复 SSL 选项，在这种情况下，服务器会自动生成用于连接的证书，并使用默认的密码套件。如果要为连接配置证书和密码套件，请使用其他分布式恢复 SSL 选项进行配置。

+   `group_replication_recovery_ssl_ca`: 用于分布式恢复连接的证书颁发机构（CA）文件的路径名。Group Replication 会自动配置克隆 SSL 选项`clone_ssl_ca`以匹配此路径。

    `group_replication_recovery_ssl_capath`: 包含受信任 SSL 证书颁发机构（CA）证书文件的目录的路径名。

+   `group_replication_recovery_ssl_cert`: SSL 公钥证书文件的路径名，用于分布式恢复连接。Group Replication 会自动配置克隆 SSL 选项`clone_ssl_cert`以匹配此路径。

+   `group_replication_recovery_ssl_key`: SSL 私钥文件的路径名，用于分布式恢复连接。Group Replication 会自动配置克隆 SSL 选项`clone_ssl_cert`以匹配此路径。

+   `group_replication_recovery_ssl_verify_server_cert`: 使分布式恢复连接检查捐赠者发送证书中服务器的通用名称值。将此选项设置为`ON`相当于为群组通信连接的`group_replication_ssl_mode`选项设置`VERIFY_IDENTITY`。

+   `group_replication_recovery_ssl_crl`: 包含证书吊销列表的文件的路径名。

+   `group_replication_recovery_ssl_crlpath`: 包含证书吊销列表的目录的路径名。

+   `group_replication_recovery_ssl_cipher`: 用于分布式恢复连接的连接加密的可接受密码列表。指定一个或多个以冒号分隔的密码名称列表。有关 MySQL 支持的加密密码的信息，请参见第 8.3.2 节，“加密连接 TLS 协议和密码”。

+   `group_replication_recovery_tls_version`: 一个逗号分隔的列表，包含了在这个服务器实例作为分布式恢复连接的客户端（即加入成员）时，连接加密所允许的一个或多个 TLS 协议。此系统变量的默认值取决于 MySQL Server 版本中支持的 TLS 协议版本。每个作为客户端（加入成员）和服务器（捐赠者）参与每个分布式恢复连接的组成员协商他们都设置支持的最高协议版本。此系统变量从 MySQL 8.0.19 版本开始提供。

+   `group_replication_recovery_tls_ciphersuites`: 一个以冒号分隔的列表，包含了在使用 TLSv1.3 进行连接加密时，分布式恢复连接的客户端（即加入成员）时所允许的一个或多个密码套件。如果在使用 TLSv1.3 时将此系统变量设置为`NULL`（如果您没有设置系统变量，则默认为此），则允许默认启用的密码套件，如第 8.3.2 节“加密连接 TLS 协议和密码”中所列出的。如果将此系统变量设置为空字符串，则不允许使用任何密码套件，因此也不会使用 TLSv1.3。此系统变量从 MySQL 8.0.19 版本开始提供。
