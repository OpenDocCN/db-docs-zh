# 25.2.6 MySQL Server 使用 InnoDB 与 NDB Cluster 比较

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-compared.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-compared.html)

25.2.6.1 NDB 和 InnoDB 存储引擎之间的差异

25.2.6.2 NDB 和 InnoDB 工作负载

25.2.6.3 NDB 和 InnoDB 特性使用总结

MySQL Server 提供了多种存储引擎选择。由于 `NDB` 和 `InnoDB` 都可以作为 MySQL 的事务性存储引擎，因此 MySQL Server 的用户有时会对 NDB Cluster 感兴趣。他们将 `NDB` 视为 MySQL 8.0 默认的 `InnoDB` 存储引擎的可能替代品或升级。虽然 `NDB` 和 `InnoDB` 共享一些共同特性，但在架构和实现上存在差异，因此一些现有的 MySQL Server 应用程序和使用场景可能非常适合 NDB Cluster，但并非所有情况都适用。

在本节中，我们讨论并比较了 NDB 8.0 使用的 `NDB` 存储引擎与 MySQL 8.0 中使用的 `InnoDB` 的一些特性。接下来的几节提供了技术比较。在许多情况下，关于何时何地使用 NDB Cluster 必须根据具体情况做出决策，考虑所有因素。虽然本文档无法为每种可能的使用场景提供具体细节，但我们也尝试就一些常见应用类型相对适用于 `NDB` 而非 `InnoDB` 后端提供一些非常一般性的指导。

NDB Cluster 8.0 使用基于 MySQL 8.0 的 **mysqld**，包括对 `InnoDB` 1.1 的支持。虽然可以在 NDB Cluster 中使用 `InnoDB` 表，但这些表不是集群化的。不可能使用 NDB Cluster 8.0 发行版中的程序或库与 MySQL Server 8.0，或者反过来。

尽管一些常见的商业应用程序可以在 NDB Cluster 或 MySQL 服务器上运行（最有可能使用`InnoDB`存储引擎），但存在一些重要的架构和实现差异。 第 25.2.6.1 节，“NDB 和 InnoDB 存储引擎之间的差异”提供了这些差异的摘要。由于这些差异，一些使用场景显然更适合其中一个引擎；请参见第 25.2.6.2 节，“NDB 和 InnoDB 工作负载”。这反过来影响了更适合与`NDB`或`InnoDB`一起使用的应用程序类型。请参见第 25.2.6.3 节，“NDB 和 InnoDB 特性使用摘要”，以比较它们在常见类型的数据库应用程序中的相对适用性。

有关`NDB`和`MEMORY`存储引擎的相对特性的信息，请参见何时使用 MEMORY 或 NDB Cluster。

有关 MySQL 存储引擎的其他信息，请参见第十八章，*替代存储引擎*。
