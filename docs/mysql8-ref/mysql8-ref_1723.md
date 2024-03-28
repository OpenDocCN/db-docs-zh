> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-logging-management-commands.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-logging-management-commands.html)

#### 25.6.3.1 NDB Cluster Logging Management Commands

**ndb_mgm**支持与集群日志和节点日志相关的许多管理命令。在接下来的列表中，*`node_id`*表示存储节点 ID 或关键字`ALL`，表示该命令应用于所有集群的数据节点。

+   `CLUSTERLOG ON`

    打开集群日志。

+   `CLUSTERLOG OFF`

    关闭集群日志。

+   `CLUSTERLOG INFO`

    提供有关集群日志设置的信息。

+   `*`node_id`* CLUSTERLOG *`category`*=*`threshold`*`

    在集群日志中记录*`category`*事件，其优先级小于或等于*`threshold`*。

+   `CLUSTERLOG FILTER *`severity_level`*`

    切换指定*`severity_level`*事件的集群日志记录。

以下表描述了集群日志类别阈值的默认设置（所有数据节点）。如果事件的优先级值低于或等于优先级阈值，则会在集群日志中报告。

注意

事件按数据节点报告，并且阈值可以在不同节点上设置为不同值。

**表 25.54 集群日志类别，默认阈值设置**

| Category | 默认阈值（所有数据节点） |
| --- | --- |
| `STARTUP` | `7` |
| `SHUTDOWN` | `7` |
| `STATISTICS` | `7` |
| `CHECKPOINT` | `7` |
| `NODERESTART` | `7` |
| `CONNECTION` | `8` |
| `ERROR` | `15` |
| `INFO` | `7` |
| `BACKUP` | `15` |
| `CONGESTION` | `7` |
| `SCHEMA` | `7` |
| Category | 默认阈值（所有数据节点） |

`STATISTICS`类别可以提供大量有用的数据。有关更多信息，请参见第 25.6.3.3 节，“在 NDB Cluster 管理客户端中使用 CLUSTERLOG STATISTICS”。

阈值用于过滤每个类别中的事件。例如，具有优先级 3 的`STARTUP`事件除非`STARTUP`的阈值设置为 3 或更高，否则不会记录。如果阈值为 3，则只发送优先级为 3 或更低的事件。

以下表显示了事件严重级别。

注意

这些对应于 Unix 的`syslog`级别，除了未使用或映射的`LOG_EMERG`和`LOG_NOTICE`。

**表 25.55 事件严重级别**

| 严重级别值 | 严重级别 | 描述 |
| --- | --- | --- |
| 1 | `ALERT` | 应立即纠正的条件，比如损坏的系统数据库 |
| 2 | `CRITICAL` | 临界条件，比如设备错误或资源不足 |
| 3 | `ERROR` | 需要纠正的条件，比如配置错误 |
| 4 | `WARNING` | 不是错误的条件，但可能需要特殊处理 |
| 5 | `INFO` | 信息消息 |
| 6 | `DEBUG` | 用于`NDBCLUSTER`开发的调试消息 |

事件严重级别可以通过`CLUSTERLOG FILTER`（见上文）打开或关闭。如果打开了一个严重级别，那么所有优先级小于或等于类别阈值的事件都会被记录下来。如果关闭了严重级别，则不会记录属于该严重级别的任何事件。

重要提示

集群日志级别是根据每个**ndb_mgmd**、每个订阅者的基础进行设置。这意味着，在具有多个管理服务器的 NDB 集群中，使用连接到一个管理服务器的**ndb_mgm**实例中的`CLUSTERLOG`命令仅影响由该管理服务器生成的日志，而不影响其他任何管理服务器生成的日志。这也意味着，如果其中一个管理服务器重新启动，那么只有由该管理服务器生成的日志会受到由重新启动引起的日志级别重置的影响。
