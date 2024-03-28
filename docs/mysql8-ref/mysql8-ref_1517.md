> 译文：[`dev.mysql.com/doc/refman/8.0/en/group-replication-distributed-recovery-connections.html`](https://dev.mysql.com/doc/refman/8.0/en/group-replication-distributed-recovery-connections.html)

#### 20.5.4.1 分布式恢复的连接

当一个加入成员在分布式恢复期间连接到在线现有成员进行状态传输时，加入成员在连接上充当客户端，而现有成员充当服务器。当通过这个连接从捐赠者的二进制日志进行状态传输（使用异步复制通道`group_replication_recovery`）时，加入成员充当副本，而现有成员充当源。当远程克隆操作在这个连接上进行时，加入成员充当接收者，而现有成员充当捐赠者。适用于这些角色的配置设置在 Group Replication 上下文之外也适用于 Group Replication，除非它们被 Group Replication 特定的配置设置或行为所覆盖。

现有成员为分布式恢复向加入成员提供的连接并不是 Group Replication 用于组内在线成员之间通信的连接。

+   Group Replication 的组通信引擎（XCom，一种 Paxos 变体）用于远程 XCom 实例之间的 TCP 通信的连接由`group_replication_local_address`系统变量指定。这个连接用于在线成员之间的 TCP/IP 消息。与本地实例的通信通过使用共享内存的输入通道进行。

+   对于分布式恢复，直到 MySQL 8.0.20，组成员向加入成员提供他们的标准 SQL 客户端连接，由 MySQL Server 的`hostname`和`port`系统变量指定。如果`report_port`系统变量指定了替代端口号，则使用该端口号。

+   从 MySQL 8.0.21 开始，组成员可以向加入成员提供一组专用客户端连接作为分布式恢复端点的替代列表，允许您单独控制分布式恢复流量，而不是由成员的常规客户端用户连接。您可以使用`group_replication_advertise_recovery_endpoints`系统变量指定此列表，并在加入组时成员将其分布式恢复端点列表传输给组。默认情况下，成员继续像早期版本一样提供标准 SQL 客户端连接。

重要提示

如果加入成员无法使用由 MySQL Server 的`hostname`系统变量定义的主机名正确识别其他成员，则分布式恢复可能会失败。建议运行 MySQL 的操作系统具有经过正确配置的唯一主机名，可以使用 DNS 或本地设置。服务器用于 SQL 客户端连接的主机名可以在性能模式表`replication_group_members`的`Member_host`列中验证。如果多个组成员外部化由操作系统设置的默认主机名，那么加入成员可能无法将其解析为正确的成员地址并无法连接进行分布式恢复。在这种情况下，您可以使用 MySQL Server 的`report_host`系统变量为每个服务器配置一个唯一的主机名来外部化。

加入成员建立分布式恢复连接的步骤如下：

1.  当成员加入组时，它使用其`group_replication_group_seeds`系统变量中包含的种子成员列表中的一个连接，最初使用该列表中指定的`group_replication_local_address`连接。种子成员可能是组的子集。

1.  通过此连接，种子成员使用 Group Replication 的成员服务向加入成员提供了一个在线组中所有成员的列表，以视图的形式呈现。成员信息包括每个成员提供的用于分布式恢复的分布式恢复端点或标准 SQL 客户端连接的详细信息。

1.  加入成员从此列表中选择一个适合的组成员作为其分布式恢复的捐赠者，遵循第 20.5.4.4 节“分布式恢复的容错”中描述的行为。

1.  然后，加入成员尝试使用捐赠者广告的分布式恢复端点连接到捐赠者，依次尝试列表中指定的顺序。如果捐赠者没有提供端点，则加入成员尝试使用捐赠者的标准 SQL 客户端连接。连接的 SSL 要求如第 20.5.4.1.4 节“分布式恢复的 SSL 和身份验证”中描述的`group_replication_recovery_ssl_*`选项所指定。

1.  如果加入成员无法连接到所选的提供者，它将尝试与其他适当的提供者重试，遵循 第 20.5.4.4 节，“分布式恢复的容错性” 中描述的行为。请注意，如果加入成员在不建立连接的情况下耗尽了广告端点列表，则不会回退到提供者的标准 SQL 客户端连接，而是切换到另一个提供者。

1.  当加入成员与提供者建立分布式恢复连接时，它将使用该连接进行状态传输，如 第 20.5.4 节，“分布式恢复” 中所述。用于连接的主机和端口在加入成员的日志中显示。请注意，如果使用远程克隆操作，当加入成员在操作结束时重新启动时，它将与新的提供者建立连接，以从二进制日志进行状态传输。这可能是与用于远程克隆操作的原始提供者不同的成员的连接，或者可能是与原始提供者的不同连接。无论如何，分布式恢复过程都将像在原始提供者的情况下一样继续。

##### 20.5.4.1.1 选择分布式恢复端点的地址

由 `group_replication_advertise_recovery_endpoints` 系统变量提供的 IP 地址作为分布式恢复端点不需要为 MySQL Server 配置（即，它们不需要由 `admin_address` 系统变量指定或在 `bind_address` 系统变量的列表中）。它们必须分配给服务器。使用的任何主机名必须解析为本地 IP 地址。可以使用 IPv4 和 IPv6 地址。

为了 MySQL Server 配置分布式恢复端点所提供的端口必须通过 `port`、`report_port` 或 `admin_port` 系统变量指定。服务器必须在这些端口上监听 TCP/IP 连接。如果指定了 `admin_port`，则分布式恢复的复制用户需要 `SERVICE_CONNECTION_ADMIN` 权限才能连接。选择 `admin_port` 可以将分布式恢复连接与常规 MySQL 客户端连接分开。

加入的成员依次尝试列表中指定的每个端点。如果`group_replication_advertise_recovery_endpoints`设置为`DEFAULT`而不是端点列表，则提供标准的 SQL 客户端连接。请注意，标准的 SQL 客户端连接不会自动包含在分布式恢复端点列表中，并且不会作为备用方案提供，如果捐赠者的端点列表在没有连接的情况下耗尽。如果您希望将标准的 SQL 客户端连接作为多个分布式恢复端点之一提供，您必须在由`group_replication_advertise_recovery_endpoints`指定的列表中显式包含它。您可以将其放在最后一位，以便作为连接的最后手段。

一个组成员的分布式恢复端点（或标准的 SQL 客户端连接，如果没有提供端点）不需要添加到由`group_replication_ip_allowlist`（从 MySQL 8.0.22 开始）或`group_replication_ip_whitelist`地址。加入的成员必须通过允许列表允许的初始连接到组，以便检索分布式恢复的地址或地址。

当设置系统变量并发出`START GROUP_REPLICATION`语句时，您列出的分布式恢复端点将得到验证。如果列表无法正确解析，或者如果主机上的任何端点无法访问，因为服务器没有在其上监听，Group Replication 将记录错误并且不会启动。

##### 20.5.4.1.2 分布式恢复的压缩

从 MySQL 8.0.18 开始，您可以选择通过从捐赠者的二进制日志进行状态传输的方法为分布式恢复配置压缩。压缩可以使分布式恢复受益，特别是在网络带宽有限且捐赠者必须向加入成员传输许多事务时。`group_replication_recovery_compression_algorithms`和`group_replication_recovery_zstd_compression_level`系统变量配置允许的压缩算法，以及在从捐赠者的二进制日志进行状态传输时使用的`zstd`压缩级别。有关更多信息，请参见第 6.2.8 节，“连接压缩控制”。

请注意，这些压缩设置不适用于远程克隆操作。当远程克隆操作用于分布式恢复时，克隆插件的`clone_enable_compression`设置适用。

##### 20.5.4.1.3 分布式恢复的复制用户

分布式恢复需要一个具有正确权限的复制用户，以便 Group Replication 可以建立直接成员间的复制通道。复制用户还必须具有正确权限，以在捐赠者上充当远程克隆操作的克隆用户。在每个组成员上进行分布式恢复时必须使用相同的复制用户。有关设置此复制用户的说明，请参见第 20.2.1.3 节，“分布式恢复的用户凭据”。有关保护复制用户凭据的说明，请参见第 20.6.3.1 节，“分布式恢复的安全用户凭据”。

##### 20.5.4.1.4 分布式恢复的 SSL 和身份验证

分布式恢复的 SSL 配置与正常组通信的 SSL 配置是分开的，由服务器的 SSL 设置和`group_replication_ssl_mode`系统变量确定。对于分布式恢复连接，专用的 Group Replication 分布式恢复 SSL 系统变量可用于配置专门用于分布式恢复的证书和密码。

默认情况下，分布式恢复连接不使用 SSL。要激活它，请设置`group_replication_recovery_use_ssl=ON`，并按照 Section 20.6.3, “Securing Distributed Recovery Connections”中描述的方式配置 Group Replication 分布式恢复 SSL 系统变量。您需要设置一个用于 SSL 的复制用户。

当配置分布式恢复使用 SSL 时，Group Replication 会将此设置应用于远程克隆操作，以及从捐赠者的二进制日志进行状态传输。Group Replication 会自动配置克隆 SSL 选项（`clone_ssl_ca`，`clone_ssl_cert`，和`clone_ssl_key`）的设置，以匹配相应的 Group Replication 分布式恢复选项（`group_replication_recovery_ssl_ca`，`group_replication_recovery_ssl_cert`，和`group_replication_recovery_ssl_key`)的设置。

如果您没有为分布式恢复使用 SSL（因此`group_replication_recovery_use_ssl`设置为`OFF`），并且 Group Replication 的复制用户帐户使用`caching_sha2_password`插件（这是 MySQL 8.0 中的默认设置）或`sha256_password`插件进行身份验证，则会使用 RSA 密钥对进行密码交换。在这种情况下，要么使用`group_replication_recovery_public_key_path`系统变量指定 RSA 公钥文件，要么使用`group_replication_recovery_get_public_key`系统变量从源请求公钥，如 Section 20.6.3.1.1, “Replication User With The Caching SHA-2 Authentication Plugin”中所述。
