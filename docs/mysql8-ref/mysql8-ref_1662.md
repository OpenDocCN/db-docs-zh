> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-params-other.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-params-other.html)

#### 25.4.2.4 其他 NDB Cluster 配置参��

本节中的列表提供了有关在`config.ini`文件中配置 NDB Cluster 的`[computer]`、`[tcp]`和`[shm]`部分中使用的参数的信息。有关各个参数的详细描述和其他信息，请参见第 25.4.3.10 节，“NDB Cluster TCP/IP 连接”或第 25.4.3.12 节，“NDB Cluster 共享内存连接”。

以下参数适用于`config.ini`文件中的`[computer]`部分：

+   `主机名`: 此计算机的主机名或 IP 地址。

+   `ID`: 此计算机的唯一标识符。

以下参数适用于`config.ini`文件中的`[tcp]`部分：

+   `允许未解析的主机名`: 当为 false（默认）时，管理节点无法解析主机名会导致致命错误；当为 true 时，未解析的主机名仅作为警告报告。

+   `校验和`: 如果启用了校验和，节点之间的所有信号都会被检查错误。

+   `组`: 用于组接近性；较小的值被解释为更接近。

+   `主机名 1`: 两台计算机通过 TCP 连接加入的第一台计算机的名称或 IP 地址。

+   `主机名 2`: 两台通过 TCP 连接加入的计算机中的第二台的名称或 IP 地址。

+   `节点 ID1`: 连接一侧的节点（数据节点、API 节点或管理节点）的 ID。

+   `节点 ID2`: 连接一侧的节点（数据节点、API 节点或管理节点）的 ID。

+   `节点服务器 ID`: 设置 TCP 连接的服务器端。

+   `过载限制`: 当发送缓冲区中有超过这么多未发送字节时，连接被视为过载。

+   `首选 IP 版本`: 指示 DNS 解析器对 IP 版本 4 或 6 的偏好。

+   `预发送校验和`: 如果启用了此参数和校验和，则执行预发送校验和检查，并检查节点之间的所有 TCP 信号是否有错误。

+   `代理`: ....

+   `ReceiveBufferMemory`: 此节点接收的信号的缓冲区字节数。

+   `SendBufferMemory`: 从此节点发送的信号的 TCP 缓冲区字节数。

+   `SendSignalId`: 在每个信号中发送 ID。在跟踪文件中使用。在调试版本中默认为 true。

+   `TcpSpinTime`: 在接收时进入睡眠之前旋转的时间。

+   `TCP_MAXSEG_SIZE`: 用于 TCP_MAXSEG 的值。

+   `TCP_RCV_BUF_SIZE`: 用于 SO_RCVBUF 的值。

+   `TCP_SND_BUF_SIZE`: 用于 SO_SNDBUF 的值。

+   `TcpBind_INADDR_ANY`: 为连接的服务器部分绑定 InAddrAny 而不是主机名。

以下参数适用于`config.ini`文件的`[shm]`部分：

+   `Checksum`: 如果启用了校验和，则检查节点之间的所有信号是否存在错误。

+   `Group`: 用于组接近性；较小的值被解释为更接近。

+   `HostName1`: 由 SHM 连接连接的两台计算机中的第一台的名称或 IP 地址。

+   `HostName2`: 由 SHM 连接连接的两台计算机中的第二台的名称或 IP 地址。

+   `NodeId1`: 连接一侧的节点（数据节点、API 节点或管理节点）的 ID。

+   `NodeId2`: 连接一侧的节点（数据节点、API 节点或管理节点）的 ID。

+   `NodeIdServer`: 设置 SHM 连接的服务器端。

+   `OverloadLimit`: 当发送缓冲区中有超过此数量的未发送字节时，连接被视为过载。

+   `PreSendChecksum`: 如果启用了此参数和 Checksum，则执行预发送校验和检查，并检查节点之间的所有 SHM 信号是否存在错误。

+   `SendBufferMemory`: 从此节点发送的信号在共享内存缓冲区中的字节数。

+   `SendSignalId`: 在每个信号中发送 ID。在跟踪文件中使用。

+   `ShmKey`: 共享内存键；当设置为 1 时，由 NDB 计算。

+   `ShmSpinTime`: 在接收时，睡眠前旋转的微秒数。

+   `ShmSize`: 共享内存段的大小。

+   `Signum`: 用于信号传递的信号编号。
