> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-config-nodes.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-config-nodes.html)

#### 25.6.16.9 ndbinfo config_nodes 表

`config_nodes` 表显示在 NDB 集群 `config.ini` 文件中配置的节点。对于每个节点，表显示包含节点 ID、节点类型（管理节点、数据节点或 API 节点）以及配置为运行的主机的名称或 IP 地址的行。

此表不显示给定节点是否实际运行，或者当前是否连接到集群。有关连接到 NDB 集群的节点的信息可以从 `nodes` 和 `processes` 表中获取。

`config_nodes` 表包含以下列：

+   `node_id`

    节点的 ID

+   `node_type`

    节点的类型

+   `node_hostname`

    节点所在主机的名称或 IP 地址

##### 注意

`node_id` 列显示在 `config.ini` 文件中用于此节点的节点 ID；如果未指定任何节点 ID，则显示将自动分配给此节点的节点 ID。

`node_type` 列显示以下三个值之一：

+   `MGM`: 管理节点。

+   `NDB`: 数据节点。

+   `API`: API 节点；包括 SQL 节点。

`node_hostname` 列显示在 `config.ini` 文件中指定的节点主机。对于 API 节点，如果在集群配置文件中未设置 `HostName`，则此处可能为空。如果在配置文件中未为数据节点设置 `HostName`，则此处使用 `localhost`。如果未为管理节点指定 `HostName`，则也使用 `localhost`。
