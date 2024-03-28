> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndb-innodb-workloads.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndb-innodb-workloads.html)

#### 25.2.6.2 NDB 和 InnoDB 工作负载

NDB Cluster 具有一系列独特属性，使其非常适合提供需要高可用性、快速故障转移、高吞吐量和低延迟的应用程序。由于其分布式架构和多节点实现，NDB Cluster 还具有特定的约束条件，可能会导致某些工作负载性能不佳。关于一些常见类型的基于数据库驱动的应用程序工作负载，`NDB`和`InnoDB`存储引擎之间行为上的一些主要差异显示在以下表格中：

**表 25.3 InnoDB 和 NDB 存储引擎之间的差异，常见类型的数据驱动应用程序工作负载。**

| 工作负载 | `InnoDB` | NDB Cluster（`NDB`） |
| --- | --- | --- |
| 高交易量 OLTP 应用程序 | 是 | 是 |
| DSS 应用程序（数据集市、分析） | 是 | 有限（跨 OLTP 数据集的连接操作不超过 3TB） |
| 自定义应用程序 | 是 | 是 |
| 打包应用程序 | 是 | 有限（应主要使用主键访问）；NDB Cluster 8.0 支持外键 |
| 网络电信应用（HLR、HSS、SDP） | 否 | 是 |
| 会话管理和缓存 | 是 | 是 |
| 电子商务应用程序 | 是 | 是 |
| 用户配置文件管理，AAA 协议 | 是 | 是 |
