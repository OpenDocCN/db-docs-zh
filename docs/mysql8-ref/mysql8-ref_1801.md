> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-processes.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-processes.html)

#### 25.6.16.50 ndbinfo 进程表

此表包含有关 NDB Cluster 节点进程的信息；每个节点在表中由一行表示。仅显示连接到集群的节点在此表中。您可以从`nodes`和`config_nodes`表中获取有关已配置但未连接到集群的节点的信息。

`processes`表包含以下列：

+   `node_id`

    集群中节点的唯一节点 ID

+   `node_type`

    节点类型（管理、数据或 API 节点；参见文本）

+   `node_version`

    `NDB`软件程序在此节点上运行的版本。

+   `process_id`

    此节点的进程 ID

+   `angel_process_id`

    此节点的 angel 进程的进程 ID

+   `process_name`

    可执行文件的名称

+   `service_URI`

    此节点的服务 URI（参见文本）

##### 备注

`node_id`是分配给此节点的 ID。

`node_type`列显示以下三个值之一：

+   `MGM`：管理节点。

+   `NDB`：数据节点。

+   `API`：API 或 SQL 节点。

对于 NDB Cluster 发行版附带的可执行文件，`node_version`显示软件 Cluster 版本字符串，例如`8.0.35-ndb-8.0.35`。

`process_id`是节点可执行文件的进程 ID，可以通过主机操作系统使用像 Linux 上的**top**或 Windows 平台上的任务管理器这样的进程显示应用程序来显示。

`angel_process_id`是节点的 angel 进程的系统进程 ID，它确保在发生故障时数据节点或 SQL 自动重新启动。对于管理节点和 SQL 节点以外的其他类型的 API 节点，此列的值为`NULL`。

`process_name`列显示正在运行的可执行文件的名称。对于管理节点，这是`ndb_mgmd`。对于数据节点，这是`ndbd`（单线程）或`ndbmtd`（多线程）。对于 SQL 节点，这是`mysqld`。对于其他类型的 API 节点，它是连接到集群的可执行程序的名称；NDB API 应用程序可以使用`Ndb_cluster_connection::set_name()`设置此列的自定义值。

`service_URI`显示了服务网络地址。对于管理节点和数据节点，使用的方案是`ndb://`。对于 SQL 节点，使用的是`mysql://`。默认情况下，除 SQL 节点外的 API 节点使用`ndb://`作为方案；NDB API 应用程序可以使用`Ndb_cluster_connection::set_service_uri()`将其设置为自定义值。无论节点类型如何，方案后面都是 NDB 传输器用于该节点的 IP 地址。对于管理节点和 SQL 节点，此地址包括端口号（通常为管理节点的 1186 和 SQL 节点的 3306）。如果 SQL 节点是使用`bind_address`系统变量启动的，则使用此地址而不是传输器地址，除非绑定地址设置为`*`、`0.0.0.0`或`::`。

附加路径信息可以包含在`service_URI`值中，反映了 SQL 节点的各种配置选项。例如，`mysql://198.51.100.3/tmp/mysql.sock`表示 SQL 节点是使用`skip_networking`系统变量启动的，而`mysql://198.51.100.3:3306/?server-id=1`表明该 SQL 节点启用了复制功能。
