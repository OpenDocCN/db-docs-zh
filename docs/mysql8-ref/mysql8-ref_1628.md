> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-limitations-limits.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-limitations-limits.html)

#### 25.2.7.2 NDB Cluster 与标准 MySQL 限制的限制和差异

在本节中，我们列出了在 NDB Cluster 中发现的与标准 MySQL 中发现的限制不同的限制，或者在标准 MySQL 中找不到的限制。

**内存使用和恢复。** 当数据插入到`NDB`表中时，内存被消耗，当删除时不会自动恢复，而是遵循以下规则：

+   对`NDB`表的`DELETE`语句使得先前被删除行使用的内存仅供同一表上的插入重用。然而，通过执行`OPTIMIZE TABLE`可以使这个内存可供一般重用。

    对集群进行滚动重启也会释放任何被删除行使用的内存。参见第 25.6.5 节，“执行 NDB Cluster 的滚动重启”。

+   对`NDB`表执行`DROP TABLE`或`TRUNCATE TABLE`操作会释放该表使用的内存，以便任何`NDB`表重用，无论是同一表还是另一个`NDB`表。

    注意

    请记住，`TRUNCATE TABLE`会删除并重新创建表。请参阅第 15.1.37 节，“TRUNCATE TABLE 语句”。

+   **集群配置所施加的限制。** 存在一些硬限制，可配置，但集群中可用的主内存设置了限制。请参阅第 25.4.3 节，“NDB Cluster 配置文件”中的完整配置参数列表。大多数配置参数可以在线升级。这些硬限制包括：

    +   数据库内存大小和索引内存大小（`DataMemory`和`IndexMemory`）。

        `DataMemory`被分配为 32KB 页面。每个`DataMemory`页面被使用后，都会分配给特定的表；一旦分配，这个内存只能通过删除表来释放。

        更多信息请参见第 25.4.3.6 节，“定义 NDB Cluster 数据节点”。

    +   每个事务中可以执行的操作的最大数量是通过配置参数`MaxNoOfConcurrentOperations`和`MaxNoOfLocalOperations`来设置的。

        注意

        批量加载、`TRUNCATE TABLE`和`ALTER TABLE`通过运行多个事务来处理为特殊情况，因此不受此限制的影响。

    +   与表和索引相关的不同限制。例如，集群中有序索引的最大数量由`MaxNoOfOrderedIndexes`确定，每个表中有序索引的最大数量为 16。

+   **节点和数据对象的最大限制。**以下限制适用于集群节点和元数据对象的数量：

    +   数据节点的最大数量为 144。（在 NDB 7.6 及更早版本中，此数量为 48。）

        数据节点的节点 ID 必须在 1 到 144 的范围内。

        管理和 API 节点可以使用 1 到 255 范围内的节点 ID。

    +   NDB Cluster 中节点的总最大数量为 255。此数字包括所有 SQL 节点（MySQL 服务器）、API 节点（访问集群的应用程序除 MySQL 服务器之外）、数据节点和管理服务器。

    +   在当前版本的 NDB Cluster 中，元数据对象的最大数量为 20320。此限制是硬编码的。

    更多信息请参见第 25.2.7.11 节，“NDB Cluster 8.0 中解决的先前 NDB Cluster 问题”。
