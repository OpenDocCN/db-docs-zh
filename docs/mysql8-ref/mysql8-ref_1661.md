> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-params-api.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-params-api.html)

#### 25.4.2.3 NDB 集群 SQL 节点和 API 节点配置参数

本节中的列表提供了有关在 `config.ini` 文件中配置 NDB 集群 SQL 节点和 API 节点的 `[mysqld]` 和 `[api]` 部分中使用的参数的信息。有关每个参数的详细描述和其他附加信息，请参见 第 25.4.3.7 节，“在 NDB 集群中定义 SQL 和其他 API 节点”。

+   `ApiVerbose`: 启用 NDB API 调试；用于 NDB 开发。

+   `仲裁延迟`: 在被要求仲裁时，仲裁者在投票前等待的毫秒数。

+   `仲裁级别`: 如果为 0，则 API 节点不是仲裁者。内核按顺序选择仲裁者 1、2。

+   `自动重新连接`: 指定当 API 节点与集群断开连接时是否应完全重新连接。

+   `批处理字节大小`: 字节的默认批处理大小。

+   `批处理大小`: 记录数量的默认批处理大小。

+   `连接退避最大时间`: 指定此 API 节点允许在尝试连接到任何给定数据节点之间的最长时间（~100ms 分辨率）。不包括连接尝试正在进行时经过的时间，最坏情况下可能需要几秒钟。通过将其设置为 0 来禁用。如果当前没有数据节点连接到此 API 节点，则将使用 StartConnectBackoffMaxTime。

+   `连接映射`: 指定要连接的数据节点。

+   `默认哈希映射大小`: 设置用于表哈希映射的大小（桶数）。支持三个值：0、240 和 3840。

+   `默认操作重做问题操作`: 当 RedoOverCommitCounter 超过时如何处理操作。

+   `在计算机上执行`: 引用先前定义的计算机的字符串。

+   `额外发送缓冲区内存`: 用于发送缓冲区的内存，除了 TotalSendBufferMemory 或 SendBufferMemory 分配的内存之外。默认值（0）允许最多 16MB。

+   `HeartbeatThreadPriority`: 设置 API 节点的心跳线程策略和优先级；请参阅手册以获取允许的值。

+   `HostName`: 此 SQL 或 API 节点的主机名或 IP 地址。

+   `Id`: 标识 MySQL 服务器或 API 节点（Id）的编号。现已弃用；请改用 NodeId。

+   `LocationDomainId`: 将此 API 节点分配给特定的可用域或区域。0（默认）表示未设置。

+   `MaxScanBatchSize`: 一次扫描的最大集合批量大小。

+   `NodeId`: 在集群中唯一标识 SQL 节点或 API 节点的编号。

+   `StartConnectBackoffMaxTime`: 与 ConnectBackoffMaxTime 相同，只是如果没有数据节点连接到此 API 节点，则使用此参数。

+   `TotalSendBufferMemory`: 用于所有传输器发送缓冲区的总内存。

+   `wan`: 使用 WAN TCP 设置作为默认设置。

关于 NDB Cluster 的 MySQL 服务器选项的讨论，请参见 Section 25.4.3.9.1, “MySQL Server Options for NDB Cluster”。关于与 NDB Cluster 相关的 MySQL 服务器系统变量的信息，请参见 Section 25.4.3.9.2, “NDB Cluster System Variables”。

注意

要将新的 SQL 或 API 节点添加到运行中的 NDB Cluster 配置中，必须在向 `config.ini` 文件（或文件，如果您使用多个管理服务器）添加新的 `[mysqld]` 或 `[api]` 部分后，对所有集群节点执行滚动重启。在新的 SQL 或 API 节点可以连接到集群之前，必须执行此操作。

如果新的 SQL 或 API 节点可以利用集群配置中以前未使用的 API 插槽连接到集群，则无需执行任何集群重启。
