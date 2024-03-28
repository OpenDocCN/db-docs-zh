> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-transporter-details.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-transporter-details.html)

#### 25.6.16.64 ndbinfo transporter_details 表

此表包含有关单个 NDB 传输器的信息，而不是由 `transporters` 表显示的聚合信息。`transporter_details` 表在 NDB 8.0.37 中添加。

`transporter_details` 表包含以下列：

+   `node_id`

    此数据节点在集群中的唯一节点 ID

+   `block_instance`

+   `trp_id`

    传输器 ID

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

    在此传输器上建立连接的次数

+   `overloaded`

    如果此传输器当前过载，则为 1，否则为 0

+   `overload_count`

    此传输器自连接以来进入过载状态的次数

+   `slowdown`

    如果此传输器处于减速状态，则为 1，否则为 0

+   `slowdown_count`

    此传输器自连接以来进入减速状态的次数

+   `encrypted`

    如果此传输器使用 TLS 连接，则此列为 `1`，否则为 `0`。

`transporter_details` 表显示集群中每个传输器的状态。有关此表中每列的更多信息，请参阅 `transporters` 表的注释。
