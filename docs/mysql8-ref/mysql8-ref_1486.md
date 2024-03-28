> 原文：[`dev.mysql.com/doc/refman/8.0/en/group-replication-observability.html`](https://dev.mysql.com/doc/refman/8.0/en/group-replication-observability.html)

#### 20.1.4.4 可观察性

尽管 Group Replication 插件中内置了许多自动化功能，但有时您可能需要了解幕后发生的情况。这就是 Group Replication 和 Performance Schema 的仪表化变得重要的地方。整个系统的状态（包括视图、冲突统计和服务状态）可以通过 Performance Schema 表进行查询。复制协议的分布式性质以及服务器实例之间达成一致并因此在事务和元数据上进行同步使得检查组的状态变得更加简单。例如，您可以连接到组中的单个服务器，并通过在与 Group Replication 相关的 Performance Schema 表上发出 select 语句来获取本地和全局信息。有关更多信息，请参见 第 20.4 节，“监控 Group Replication”。
