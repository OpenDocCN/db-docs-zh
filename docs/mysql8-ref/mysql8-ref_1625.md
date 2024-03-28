> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndb-innodb-usage.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndb-innodb-usage.html)

#### 25.2.6.3 NDB 和 InnoDB 特性使用摘要

当将应用程序特性要求与`InnoDB`和`NDB`的能力进行比较时，有些特性显然与一个存储引擎更兼容。

下表列出了根据每个特性通常更适合的存储引擎支持的应用程序特性。

**表 25.4 根据每个特性通常更适合的存储引擎支持的应用程序特性**

| `InnoDB` 的首选应用程序要求 | `NDB` 的首选应用程序要求 |
| --- | --- |

|

+   外键

    注意

    NDB Cluster 8.0 支持外键

+   完整表扫描

+   非常大的数据库、行或事务

+   除了`READ COMMITTED`之外的事务

|

+   写扩展

+   99.999% 的正常运行时间

+   节点的在线添加和在线模式操作

+   多个 SQL 和 NoSQL API（参见 NDB 集群 API：概述和概念）

+   实时性能

+   有限使用`BLOB`列

+   支持外键，尽管它们的使用可能会对高吞吐量的性能产生影响

|
