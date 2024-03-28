# 25.6.3 在 NDB Cluster 中生成的事件报告

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-event-reports.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-event-reports.html)

25.6.3.1 NDB Cluster Logging Management Commands

25.6.3.2 NDB Cluster Log Events

25.6.3.3 在 NDB Cluster 管理客户端中使用 CLUSTERLOG STATISTICS

在本节中，我们讨论 NDB Cluster 提供的事件日志类型，以及记录的事件类型。

NDB Cluster 提供两种类型的事件日志：

+   集群日志，其中包括所有集群节点生成的事件。集群日志是推荐用于大多数情况的日志，因为它提供了整个集群的日志信息。

    默认情况下，集群日志保存在名为`ndb_*`node_id`*_cluster.log`的文件中（其中*`node_id`*是管理服务器的节点 ID），位于管理服务器的`DataDir`中。

    集群日志信息也可以发送到`stdout`或`syslog`设施，而不是或除了保存到文件中，这取决于为`DataDir`和`LogDestination`配置参数设置的值。有关这些参数的更多信息，请参见 Section 25.4.3.5, “Defining an NDB Cluster Management Server”。

+   节点日志对每个节点是本地的。

    节点事件日志生成的输出被写入文件`ndb_*`node_id`*_out.log`（其中*`node_id`*是节点的节点 ID），位于节点的`DataDir`中。节点事件日志为管理节点和数据节点生成。

    节点日志仅用于应用程序开发或调试应用程序代码。

每个可报告事件可以根据三个不同的标准进行区分：

+   *类别*：可以是以下任一值之一：`STARTUP`、`SHUTDOWN`、`STATISTICS`、`CHECKPOINT`、`NODERESTART`、`CONNECTION`、`ERROR`或`INFO`。

+   *优先级*：表示为从 0 到 15 的数字之一，其中 0 表示“最重要”，15 表示“最不重要”。

+   *严重级别*：可以是以下任一值之一：`ON`、`DEBUG`、`INFO`、`WARNING`、`ERROR`、`CRITICAL`、`ALERT`或`ALL`。（有时也称为日志级别。）

可使用 NDB 管理客户端`CLUSTERLOG`命令在这些属性上过滤集群日志。此命令仅影响集群日志，并不影响节点日志；可以使用**ndb_mgm** `NODELOG DEBUG` 命令打开和关闭一个或多个节点日志中的调试日志记录。

NDB Cluster 生成的日志消息的格式（截至 NDB 8.0.26）如下所示：

```sql
*timestamp* [*node_type*] *level* -- Node *node_id*: *message*
```

日志中的每行，或日志消息，包含以下信息：

+   以`*`YYYY`*-*`MM`*-*`DD`* *`HH`*:*`MM`*:*`SS`*`格式的*`timestamp`*。时间戳值目前仅解析到整秒；不支持小数秒。

+   *`node_type`*，即执行日志记录的节点或应用程序的类型。在集群日志中，这始终是`[MgmSrvr]`；在数据节点日志中，始终是`[ndbd]`。在由 NDB API 应用程序和工具生成的日志中可能出现`[NdbApi]`和其他值。

+   事件的*`level`*，有时也称为其严重级别或日志级别。有关严重级别的更多信息，请参见本节前面以及第 25.6.3.1 节，“NDB Cluster 日志管理命令”。

+   报告事件的节点的 ID（*`node_id`*）。

+   包含事件描述的*`message`*。日志中出现的最常见事件类型是集群中不同节点之间的连接和断开连接，以及发生检查点时。在某些情况下，描述可能包含状态或其他信息。

这里显示了实际集群日志中的示例：

```sql
2021-06-10 10:01:07 [MgmtSrvr] INFO     -- Node 5: Start phase 5 completed (system restart)
2021-06-10 10:01:07 [MgmtSrvr] INFO     -- Node 6: Start phase 5 completed (system restart)
2021-06-10 10:01:07 [MgmtSrvr] INFO     -- Node 5: Start phase 6 completed (system restart)
2021-06-10 10:01:07 [MgmtSrvr] INFO     -- Node 6: Start phase 6 completed (system restart)
2021-06-10 10:01:07 [MgmtSrvr] INFO     -- Node 5: President restarts arbitration thread [state=1]
2021-06-10 10:01:07 [MgmtSrvr] INFO     -- Node 5: Start phase 7 completed (system restart)
2021-06-10 10:01:07 [MgmtSrvr] INFO     -- Node 6: Start phase 7 completed (system restart)
2021-06-10 10:01:07 [MgmtSrvr] INFO     -- Node 5: Start phase 8 completed (system restart)
2021-06-10 10:01:07 [MgmtSrvr] INFO     -- Node 6: Start phase 8 completed (system restart)
2021-06-10 10:01:07 [MgmtSrvr] INFO     -- Node 5: Start phase 9 completed (system restart)
2021-06-10 10:01:07 [MgmtSrvr] INFO     -- Node 6: Start phase 9 completed (system restart)
2021-06-10 10:01:07 [MgmtSrvr] INFO     -- Node 5: Start phase 50 completed (system restart)
2021-06-10 10:01:07 [MgmtSrvr] INFO     -- Node 6: Start phase 50 completed (system restart)
2021-06-10 10:01:07 [MgmtSrvr] INFO     -- Node 5: Start phase 101 completed (system restart)
2021-06-10 10:01:07 [MgmtSrvr] INFO     -- Node 6: Start phase 101 completed (system restart)
2021-06-10 10:01:07 [MgmtSrvr] INFO     -- Node 5: Started (mysql-8.0.35 ndb-8.0.35)
2021-06-10 10:01:07 [MgmtSrvr] INFO     -- Node 6: Started (mysql-8.0.35 ndb-8.0.35)
2021-06-10 10:01:07 [MgmtSrvr] INFO     -- Node 5: Node 50: API mysql-8.0.35 ndb-8.0.35
2021-06-10 10:01:07 [MgmtSrvr] INFO     -- Node 6: Node 50: API mysql-8.0.35 ndb-8.0.35
2021-06-10 10:01:08 [MgmtSrvr] INFO     -- Node 6: Prepare arbitrator node 50 [ticket=75fd00010fa8b608]
2021-06-10 10:01:08 [MgmtSrvr] INFO     -- Node 5: Started arbitrator node 50 [ticket=75fd00010fa8b608]
2021-06-10 10:01:08 [MgmtSrvr] INFO     -- Node 6: Communication to Node 100 opened
2021-06-10 10:01:08 [MgmtSrvr] INFO     -- Node 6: Communication to Node 101 opened
2021-06-10 10:01:08 [MgmtSrvr] INFO     -- Node 5: Communication to Node 100 opened
2021-06-10 10:01:08 [MgmtSrvr] INFO     -- Node 5: Communication to Node 101 opened
2021-06-10 10:01:36 [MgmtSrvr] INFO     -- Alloc node id 100 succeeded
2021-06-10 10:01:36 [MgmtSrvr] INFO     -- Nodeid 100 allocated for API at 127.0.0.1
2021-06-10 10:01:36 [MgmtSrvr] INFO     -- Node 100: mysqld --server-id=1
2021-06-10 10:01:36 [MgmtSrvr] INFO     -- Node 5: Node 100 Connected
2021-06-10 10:01:36 [MgmtSrvr] INFO     -- Node 6: Node 100 Connected
2021-06-10 10:01:36 [MgmtSrvr] INFO     -- Node 5: Node 100: API mysql-8.0.35 ndb-8.0.35
2021-06-10 10:01:36 [MgmtSrvr] INFO     -- Node 6: Node 100: API mysql-8.0.35 ndb-8.0.35
```

有关更多信息，请参见第 25.6.3.2 节，“NDB Cluster 日志事件”。
