> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-nodes.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-nodes.html)

#### 25.6.16.47 ndbinfo nodes 表

该表包含数据节点状态的信息。对于在集群中运行的每个数据节点，该表中的相应行提供节点的节点 ID、状态和运行时间。对于正在启动的节点，还显示当前的启动阶段。

`nodes`表包含以下列：

+   `node_id`

    集群中数据节点的唯一节点 ID。

+   `uptime`

    节点自上次启动以来的时间，以秒为单位。

+   `status`

    数据节点的当前状态；请参阅文本以获取可能的值。

+   `start_phase`

    如果数据节点正在启动，当前的启动阶段。

+   `config_generation`

    在此数据节点上使用的集群配置文件的版本。

##### 注意

`uptime`列显示自上次启动或重新启动以来该节点运行的时间（以秒为单位）。这是一个`BIGINT` - INTEGER, INT, SMALLINT, TINYINT, MEDIUMINT, BIGINT")值。这个数字包括实际启动节点所需的时间；换句话说，这个计数器从第一次调用**ndbd**或**ndbmtd**")开始运行；因此，即使对于尚未完成启动的节点，`uptime`可能显示一个非零值。

`status`列显示节点的当前状态。这是其中之一：`NOTHING`、`CMVMI`、`STARTING`、`STARTED`、`SINGLEUSER`、`STOPPING_1`、`STOPPING_2`、`STOPPING_3`或`STOPPING_4`。当状态为`STARTING`时，您可以在`start_phase`列中看到当前的启动阶段（请参见本节后面的内容）。当集群处于单用户模式时（请参阅第 25.6.6 节，“NDB 集群单用户模式”），所有数据节点的`status`列中都显示`SINGLEUSER`。看到`STOPPING`状态之一并不一定意味着节点正在关闭，而可能意味着它正在进入新状态。例如，如果将集群置于单用户模式，有时可以看到数据节点在状态短暂报告为`STOPPING_2`，然后状态更改为`SINGLEUSER`。

`start_phase`列使用与**ndb_mgm**客户端`*`node_id`* STATUS`命令输出中使用的相同值范围（请参见第 25.6.1 节，“NDB 集群管理客户端中的命令”）。如果节点当前未启动，则此列显示`0`。有关带有描述的 NDB 集群启动阶段的列表，请参见第 25.6.4 节，“NDB 集群启动阶段摘要”。

`config_generation`列显示每个数据节点上生效的集群配置版本。在执行集群滚动重启以更改配置参数时，这可能很有用。例如，从以下`SELECT`语句的输出中，您可以看到节点 3 尚未使用最新版本的集群配置（`6`），尽管节点 1、2 和 4 正在使用：

```sql
mysql> USE ndbinfo;
Database changed
mysql> SELECT * FROM nodes;
+---------+--------+---------+-------------+-------------------+
| node_id | uptime | status  | start_phase | config_generation |
+---------+--------+---------+-------------+-------------------+
|       1 |  10462 | STARTED |           0 |                 6 |
|       2 |  10460 | STARTED |           0 |                 6 |
|       3 |  10457 | STARTED |           0 |                 5 |
|       4 |  10455 | STARTED |           0 |                 6 |
+---------+--------+---------+-------------+-------------------+
2 rows in set (0.04 sec)
```

因此，对于刚刚显示的情况，您应重新启动节点 3 以完成集群的滚动重启。

停止的节点不在此表中列出。假设您有一个具有 4 个数据节点（节点 ID 为 1、2、3 和 4）的 NDB 集群，并且所有节点正常运行，则此表包含 4 行，每个数据节点一行：

```sql
mysql> USE ndbinfo;
Database changed
mysql> SELECT * FROM nodes;
+---------+--------+---------+-------------+-------------------+
| node_id | uptime | status  | start_phase | config_generation |
+---------+--------+---------+-------------+-------------------+
|       1 |  11776 | STARTED |           0 |                 6 |
|       2 |  11774 | STARTED |           0 |                 6 |
|       3 |  11771 | STARTED |           0 |                 6 |
|       4 |  11769 | STARTED |           0 |                 6 |
+---------+--------+---------+-------------+-------------------+
4 rows in set (0.04 sec)
```

如果关闭其中一个节点，则此`SELECT`语句的输出中仅表示仍在运行的节点，如下所示：

```sql
ndb_mgm> 2 STOP
Node 2: Node shutdown initiated
Node 2: Node shutdown completed.
Node 2 has shutdown.
```

```sql
mysql> SELECT * FROM nodes;
+---------+--------+---------+-------------+-------------------+
| node_id | uptime | status  | start_phase | config_generation |
+---------+--------+---------+-------------+-------------------+
|       1 |  11807 | STARTED |           0 |                 6 |
|       3 |  11802 | STARTED |           0 |                 6 |
|       4 |  11800 | STARTED |           0 |                 6 |
+---------+--------+---------+-------------+-------------------+
3 rows in set (0.02 sec)
```
