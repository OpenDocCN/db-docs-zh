# 25.6.6 NDB 集群单用户模式

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-single-user-mode.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-single-user-mode.html)

单用户模式使数据库管理员能够将对数据库系统的访问限制为单个 API 节点，例如 MySQL 服务器（SQL 节点）或一个 **ndb_restore** 实例。进入单用户模式时，所有其他 API 节点的连接将被优雅地关闭，所有正在运行的事务都将被中止。不允许启动新事务。

一旦集群进入单用户模式，只有指定的 API 节点被授予对数据库的访问权限。

您可以使用 **ndb_mgm** 客户端中的 `ALL STATUS` 命令来查看集群何时进入单用户模式。您还可以检查 `ndbinfo.nodes` 表的 `status` 列（有关更多信息，请参见 第 25.6.16.47 节，“ndbinfo nodes 表”）。

示例:

```sql
ndb_mgm> ENTER SINGLE USER MODE 5
```

执行此命令后，集群进入单用户模式，其节点 ID 为 `5` 的 API 节点成为集群唯一允许的用户。

在上述命令中指定的节点必须是 API 节点；尝试指定任何其他类型的节点都将被拒绝。

注意

当调用上述命令时，运行在指定节点上的所有事务都将被中止，连接将被关闭，服务器必须重新启动。

命令 `EXIT SINGLE USER MODE` 将集群的数据节点从单用户模式更改为正常模式。等待连接的 API 节点（例如 MySQL 服务器）再次被允许连接（即等待集群准备就绪并可用）。在状态更改期间和之后，被指定为单用户节点的 API 节点继续运行（如果仍然连接）。

示例:

```sql
ndb_mgm> EXIT SINGLE USER MODE
```

在单用户模式下运行时，有两种推荐的处理节点故障的方法：

+   方法 1:

    1.  完成所有单用户模式事务

    1.  发出 `EXIT SINGLE USER MODE` 命令

    1.  重新启动集群的数据节点

+   方法 2:

    在进入单用户模式之前重新启动存储节点。
