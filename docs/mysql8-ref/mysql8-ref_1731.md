> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-online-add-node-basics.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-online-add-node-basics.html)

#### 25.6.7.2 在线添加 NDB Cluster 数据节点：基本过程

在本节中，我们列出了向 NDB Cluster 添加新数据节点所需的基本步骤。此过程适用于数据节点进程使用 **ndbd** 或 **ndbmtd**") 二进制文件。有关更详细的示例，请参见 第 25.6.7.3 节，“在线添加 NDB Cluster 数据节点：详细示例”。

假设您已经有一个运行中的 NDB Cluster，添加在线数据节点需要以下步骤：

1.  编辑集群配置 `config.ini` 文件，添加新的与要添加的节点对应的 `[ndbd]` 部分。如果集群使用多个管理服务器，则需要对所有管理服务器使用的 `config.ini` 文件进行更改。

    您必须小心，确保在 `config.ini` 文件中添加的任何新数据节点的节点 ID 不重叠使用现有节点的节点 ID。如果您有使用动态分配的节点 ID 的 API 节点，并且这些 ID 与您要用于新数据节点的节点 ID 匹配，那么可以强制任何此类 API 节点“迁移”，如本过程后面所述。

1.  对所有 NDB Cluster 管理服务器执行滚动重启。

    重要提示

    所有管理服务器必须使用 `--reload` 或 `--initial` 选项重新启动，以强制读取新配置。

1.  对所有现有的 NDB Cluster 数据节点执行滚动重启。在重新启动现有数据节点时，通常不需要（甚至不建议）使用`--initial`。

    如果您正在使用具有动态分配的 ID 的 API 节点，这些 ID 与您希望分配给新数据节点的任何节点 ID 匹配，那么在重新启动此步骤中的任何数据节点进程之前，必须重新启动所有 API 节点（包括 SQL 节点）。这将导致任何具有先前未明确分配的节点 ID 的 API 节点放弃这些节点 ID 并获取新的节点 ID。

1.  对连接到 NDB Cluster 的任何 SQL 或 API 节点执行滚动重启。

1.  启动新数据节点。

    新数据节点可以以任何顺序启动。只要在所有现有数据节点的滚动重启完成后启动它们，并在进行下一步之前启动它们即可。

1.  在 NDB Cluster 管理客户端中执行一个或多个`CREATE NODEGROUP`命令，以创建新数据节点所属的新节点组或节点组。

1.  重新分配集群的数据到所有数据节点，包括新节点。通常通过在**mysql**客户端中为每个`NDBCLUSTER`表发出一个`ALTER TABLE ... ALGORITHM=INPLACE, REORGANIZE PARTITION`语句来完成此操作。

    *异常*：对于使用`MAX_ROWS`选项创建的表，此语句不起作用；而是使用`ALTER TABLE ... ALGORITHM=INPLACE MAX_ROWS=...`来重新组织这些表。您还应该记住，以这种方式使用`MAX_ROWS`设置分区数量已被弃用，应该使用`PARTITION_BALANCE`代替；有关更多信息，请参见第 15.1.20.12 节，“设置 NDB 注释选项”。

    注意

    这仅适用于在添加新节点组时已经存在的表。在添加新节点组后创建的表中的数据会自动分布；但是，在添加新节点之前存在的表`tbl`中添加的数据直到重新组织该表后才会使用新节点进行分布。

1.  `ALTER TABLE ... REORGANIZE PARTITION ALGORITHM=INPLACE`重新组织分区但不会回收“旧”节点上释放的空间。您可以通过为每个`NDBCLUSTER`表在**mysql**客户端中发出一个`OPTIMIZE TABLE`语句来执行此操作。

    这适用于内存中`NDB`表的可变宽度列使用的空间。`OPTIMIZE TABLE`不支持内存表的固定宽度列；也不支持磁盘数据表。

您可以添加所有所需的节点，然后连续发出几个`CREATE NODEGROUP`命令来将新节点组添加到集群中。
