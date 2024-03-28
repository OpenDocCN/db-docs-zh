> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndb-innodb-engines.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndb-innodb-engines.html)

#### 25.2.6.1 NDB 和 InnoDB 存储引擎之间的差异

`NDB` 存储引擎采用分布式、无共享架构实现，这导致它在许多方面的行为与 `InnoDB` 有所不同。对于不习惯使用 `NDB` 的人来说，由于其分布式特性涉及事务、外键、表限制和其他特性，可能会出现意外行为。这些情况如下表所示：

**表 25.2 InnoDB 和 NDB 存储引擎之间的差异**

| 特性 | `InnoDB`（MySQL 8.0） | `NDB` 8.0 |
| --- | --- | --- |
| MySQL 服务器版本 | 8.0 | 8.0 |
| `InnoDB` 版本 | `InnoDB` 8.0.36 | `InnoDB` 8.0.36 |
| NDB Cluster 版本 | 不适用 | `NDB` 8.0.35/8.0.35 |
| 存储限制 | 64TB | 128TB |
| 外键 | 是 | 是 |
| 事务 | 所有标准类型 | `READ COMMITTED` |
| MVCC | 是 | 否 |
| 数据压缩 | 是 | 否（NDB 检查点和备份文件可以进行压缩） |
| 大型行支持（> 14K） | 对 `VARBINARY`、`VARCHAR`、`BLOB` 和 `TEXT` 列支持 | 仅对 `BLOB` 和 `TEXT` 列支持（使用这些类型存储大量数据可能降低 NDB 性能） |
| 复制支持 | 使用 MySQL 复制的异步和半同步复制；MySQL Group Replication | NDB Cluster 内的自动同步复制；NDB 集群之间的异步复制，使用 MySQL 复制（不支持半同步复制） |
| 读操作的扩展 | 是（MySQL 复制） | 是（NDB Cluster 中的自动分区；NDB Cluster 复制） |
| 写操作的扩展 | 需要应用级分区（分片） | 是（NDB Cluster 中的自动分区对应用程序透明） |
| 高可用性（HA） | 内置，来自 InnoDB 集群 | 是（设计用于 99.999% 的正常运行时间） |
| 节点故障恢复和故障转移 | 来自 MySQL Group Replication | 自动（NDB 架构的关键元素） |
| 节点故障恢复时间 | 30 秒或更长 | 通常< 1 秒 |
| 实时性能 | 否 | 是 |
| 内存表 | 否 | 是（一些数据可以选择性地存储在磁盘上；内存和磁盘数据存储都是持久的） |
| NoSQL 访问存储引擎 | 是 | 是（包括 Memcached、Node.js/JavaScript、Java、JPA、C++和 HTTP/REST 等多个 API） |
| 并发和并行写入 | 是 | 最多 48 个写入者，优化并发写入 |
| 冲突检测和解决（多源） | 是（MySQL 集群复制） | 是 |
| 哈希索引 | 否 | 是 |
| 节点在线添加 | 使用 MySQL 集群复制的读/写副本 | 是（所有节点类型） |
| 在线升级 | 是（使用复制） | 是 |
| 在线模式修改 | 是，作为 MySQL 8.0 的一部分 | 是 |
| 特性 | `InnoDB`（MySQL 8.0） | `NDB` 8.0 |
