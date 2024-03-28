# 25.6.17 NDB 集群的 INFORMATION_SCHEMA 表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-information-schema-tables.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-information-schema-tables.html)

两个`INFORMATION_SCHEMA`表提供了在管理 NDB 集群时特别有用的信息。`FILES`表提供了关于 NDB 集群磁盘数据文件的信息（参见第 25.6.11.1 节，“NDB 集群磁盘数据对象”）。`ndb_transid_mysql_connection_map`表提供了事务、事务协调器和 API 节点之间的映射。

可以从`ndbinfo`数据库中的表中获取有关 NDB 集群事务、操作、线程、块以及性能的其他统计数据。有关这些表的信息，请参见第 25.6.16 节，“ndbinfo: NDB 集群信息数据库”。
