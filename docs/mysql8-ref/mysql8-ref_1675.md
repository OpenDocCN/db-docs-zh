> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-tcp-definition-direct.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-tcp-definition-direct.html)

#### 25.4.3.11 NDB 集群使用直接连接的 TCP/IP 连接

使用数据节点之间的直接连接设置集群需要在集群 `config.ini` 文件的 `[tcp]` 部分明确指定连接的数据节点的交叉 IP 地址。

在下面的示例中，我们设想一个至少有四个主机的集群，分别用于管理服务器、SQL 节点和两个数据节点。整个集群位于 LAN 的 `172.23.72.*` 子网上。除了通常的网络连接外，两个数据节点使用标准交叉电缆直接连接，并使用 `1.1.0.*` 地址范围中的 IP 地址直接通信，如下所示：

```sql
# Management Server
[ndb_mgmd]
Id=1
HostName=172.23.72.20

# SQL Node
[mysqld]
Id=2
HostName=172.23.72.21

# Data Nodes
[ndbd]
Id=3
HostName=172.23.72.22

[ndbd]
Id=4
HostName=172.23.72.23

# TCP/IP Connections
[tcp]
NodeId1=3
NodeId2=4
HostName1=1.1.0.1
HostName2=1.1.0.2
```

只有在指定直接连接时才使用 `HostName1` 和 `HostName2` 参数。

使用数据节点之间的直接 TCP 连接可以通过使数据节点绕过以太网设备（如交换机、集线器或路由器）来提高集群的整体效率，从而减少集群的延迟。

注意

要充分利用这种方式的直接连接，当有两个以上的数据节点时，必须在同一节点组中的每个数据节点之间建立直接连接。
