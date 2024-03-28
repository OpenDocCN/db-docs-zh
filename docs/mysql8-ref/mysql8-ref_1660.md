> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-params-mgmd.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-params-mgmd.html)

#### 25.4.2.2 NDB 集群管理节点配置参数

本节中的列表提供了有关在`config.ini`文件中配置 NDB 集群管理节点的`[ndb_mgmd]`或`[mgm]`部分中使用的参数的信息。有关每个参数的详细描述和其他附加信息，请参阅第 25.4.3.5 节，“定义 NDB 集群管理服务器”。

+   `ArbitrationDelay`: 在被要求进行仲裁时，仲裁者在投票之前等待的时间（毫秒）。

+   `ArbitrationRank`: 如果为 0，则管理节点不是仲裁者。内核按顺序选择仲裁者为 1、2。

+   `DataDir`: 此节点的数据目录。

+   `ExecuteOnComputer`: 引用先前定义的计算机的字符串。

+   `ExtraSendBufferMemory`: 用于发送缓冲区的内存，除了由 TotalSendBufferMemory 或 SendBufferMemory 分配的内存。默认值（0）允许最多使用 16MB。

+   `HeartbeatIntervalMgmdMgmd`: 管理节点之间心跳之间的时间；在 3 次错过心跳后，认为管理节点之间的连接已丢失。

+   `HeartbeatThreadPriority`: 设置管理节点的心跳线程策略和优先级；请参阅手册以获取允许的数值。

+   `HostName`: 此管理节点的主机名或 IP 地址。

+   `Id`: 标识管理节点的编号。现已弃用；请使用 NodeId 代替。

+   `LocationDomainId`: 将此管理节点分配给特定的可用域或区域。0（默认）表示未设置。

+   `LogDestination`: 日志消息发送到的位置：控制台、系统日志或指定的日志文件。

+   `NodeId`: 在集群中唯一标识管理节点的编号。

+   `PortNumber`: 发送命令和从管理服务器获取配置的端口号。

+   `PortNumberStats`: 用于从管理服务器获取统计信息的端口号。

+   `TotalSendBufferMemory`: 用于所有传输器发送缓冲区的总内存。

+   `wan`: 将 WAN TCP 设置用作默认设置。

注意

在管理节点的配置中进行更改后，需要对集群执行滚动重启，以使新配置生效。查看 Section 25.4.3.5, “定义 NDB 集群管理服务器”，获取更多信息。

要向正在运行的 NDB 集群添加新的管理服务器，还需要在修改任何现有的`config.ini`文件后对所有集群节点执行滚动重启。有关在使用多个管理节点时出现的问题的更多信息，请参阅 Section 25.2.7.10, “与多个 NDB 集群节点相关的限制”。
