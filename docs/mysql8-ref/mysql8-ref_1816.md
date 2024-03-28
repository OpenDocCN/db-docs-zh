> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-transporters.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-transporters.html)

#### 25.6.16.65 ndbinfo transporters 表

此表包含有关 NDB 传输器的聚合信息。在 NDB 8.0.37 及更高版本中，您可以从`transporter_details`表中获取有关单个传输器的类似信息。

`transporters` 表包含以下列：

+   `node_id`

    此数据节点在集群中的唯一节点 ID

+   `remote_node_id`

    远程数据节点的节点 ID

+   `status`

    连接的状态

+   `remote_address`

    远程主机的名称或 IP 地址

+   `bytes_sent`

    使用此连接发送的字节数

+   `bytes_received`

    使用此连接接收的字节数

+   `connect_count`

    自此传输器建立连接以来的连接次数

+   `overloaded`

    如果此传输器当前超载，则为 1，否则为 0

+   `overload_count`

    此传输器自连接以来进入超载状态的次数

+   `slowdown`

    如果此传输器处于减速状态，则为 1，否则为 0

+   `slowdown_count`

    自连接以来此传输器进入减速状态的次数

##### 注释

对于集群中每个运行的数据节点，`transporters` 表显示一行，显示该节点与集群中所有节点（包括自身）的每个连接的状态。此信息显示在表的*status*列中，该列可以具有以下任一值：`CONNECTING`、`CONNECTED`、`DISCONNECTING`或`DISCONNECTED`。

配置但当前未连接到集群的 API 和管理节点的连接显示为`DISCONNECTED`状态。在此表中不显示`node_id`为当前未连接的数据节点的行（这类似于`ndbinfo.nodes`表中省略了断开连接的节点。

`remote_address` 是显示在`remote_node_id`列中的节点的主机名或地址。从此节点发送的`bytes_sent`和此节点接收的`bytes_received`分别是使用此连接发送和接收的字节数。对于状态为`CONNECTING`或`DISCONNECTED`的节点，这些列始终显示`0`。

假设您有一个由 2 个数据节点、2 个 SQL 节点和 1 个管理节点组成的 5 节点集群，如`SHOW`命令在**ndb_mgm**客户端的输出中所示：

```sql
ndb_mgm> SHOW
Connected to Management Server at: localhost:1186
Cluster Configuration
---------------------
[ndbd(NDB)]     2 node(s)
id=1    @10.100.10.1  (8.0.35-ndb-8.0.35, Nodegroup: 0, *)
id=2    @10.100.10.2  (8.0.35-ndb-8.0.35, Nodegroup: 0)

[ndb_mgmd(MGM)] 1 node(s)
id=10   @10.100.10.10  (8.0.35-ndb-8.0.35)

[mysqld(API)]   2 node(s)
id=20   @10.100.10.20  (8.0.35-ndb-8.0.35)
id=21   @10.100.10.21  (8.0.35-ndb-8.0.35)
```

`transporters` 表中有 10 行——第一个数据节点有 5 行，第二个数据节点也有 5 行——假设所有数据节点都在运行，如下所示：

```sql
mysql> SELECT node_id, remote_node_id, status
 ->   FROM ndbinfo.transporters;
+---------+----------------+---------------+
| node_id | remote_node_id | status        |
+---------+----------------+---------------+
|       1 |              1 | DISCONNECTED  |
|       1 |              2 | CONNECTED     |
|       1 |             10 | CONNECTED     |
|       1 |             20 | CONNECTED     |
|       1 |             21 | CONNECTED     |
|       2 |              1 | CONNECTED     |
|       2 |              2 | DISCONNECTED  |
|       2 |             10 | CONNECTED     |
|       2 |             20 | CONNECTED     |
|       2 |             21 | CONNECTED     |
+---------+----------------+---------------+
10 rows in set (0.04 sec)
```

如果您使用 **ndb_mgm** 客户端中的 `2 STOP` 命令关闭此集群中的一个数据节点，然后重复上一个查询（再次使用 **mysql** 客户端），则此表现在仅显示 5 行—每个连接从剩余管理节点到另一个节点，包括自身和当前离线的数据节点，显示每个剩余连接到当前离线的数据节点的状态为 `CONNECTING`，如下所示：

```sql
mysql> SELECT node_id, remote_node_id, status
 ->   FROM ndbinfo.transporters;
+---------+----------------+---------------+
| node_id | remote_node_id | status        |
+---------+----------------+---------------+
|       1 |              1 | DISCONNECTED  |
|       1 |              2 | CONNECTING    |
|       1 |             10 | CONNECTED     |
|       1 |             20 | CONNECTED     |
|       1 |             21 | CONNECTED     |
+---------+----------------+---------------+
5 rows in set (0.02 sec)
```

`connect_count`、`overloaded`、`overload_count`、`slowdown` 和 `slowdown_count` 计数器在连接时重置，并在远程节点断开连接后保留其值。`bytes_sent` 和 `bytes_received` 计数器也在连接时重置，因此在断开连接后保留其值（直到下次连接重置它们）。

`overloaded` 和 `overload_count` 列所指的 *过载* 状态发生在此传输器的发送缓冲区包含超过 `OVerloadLimit` 字节（默认为 `SendBufferMemory` 的 80%，即 0.8 * 2097152 = 1677721 字节）时。当给定传输器处于过载状态时，任何尝试使用此传输器的新事务都会因错误 1218（NDB 内核中的发送缓冲区过载）而失败。这影响扫描和主键操作。

此表中 `slowdown` 和 `slowdown_count` 列引用的 *减速* 状态发生在传输器的发送缓冲区包含超过过载限制的 60% 时（默认为 0.6 * 2097152 = 1258291 字节）。在此状态下，使用此传输器的任何新扫描都会将其批量大小减小以减少传输器的负载。

发送缓冲区减速或过载的常见原因包括以下内容：

+   数据大小，特别是存储在 `TEXT` 列或 `BLOB` 列（或两种列类型）中的数据量

+   在进行二进制日志记录的 SQL 节点上与数据节点（ndbd 或 ndbmtd）位于同一主机上

+   每个事务或事务批次中的大量行

+   配置问题，如不足的 `SendBufferMemory`

+   硬件问题，如内存不足或网络连接质量差

参见 Section 25.4.3.14, “配置 NDB 集群发送缓冲区参数”。
