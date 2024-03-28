# 25.7.2 NDB Cluster Replication 的一般要求

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-replication-general.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-replication-general.html)

一个复制通道需要两个充当复制服务器的 MySQL 服务器（分别用于源和副本）。例如，这意味着在具有两个复制通道的复制设置中（为提供冗余的额外通道），应该总共有四个复制节点，每个集群两个。

正如本节和后续节中所述的 NDB Cluster 的复制取决于基于行的复制。这意味着复制源 MySQL 服务器必须使用`--binlog-format=ROW`或`--binlog-format=MIXED`运行，如 Section 25.7.6, “Starting NDB Cluster Replication (Single Replication Channel)”")中所述。有关基于行的复制的一般信息，请参见 Section 19.2.1, “Replication Formats”。

重要提示

如果您尝试使用`--binlog-format=STATEMENT`与 NDB Cluster Replication，由于源集群上的`ndb_binlog_index`表和副本集群上的`ndb_apply_status`表的`epoch`列未更新，复制将无法正常工作（参见 Section 25.7.4, “NDB Cluster Replication Schema and Tables”）。相反，只有作为复制源的 MySQL 服务器上的更新会传播到副本，源集群中任何其他 SQL 节点的更新都不会被复制。

`--binlog-format`选项的默认值为`MIXED`。

在任一集群中用于复制的每个 MySQL 服务器必须在所有参与任一集群的 MySQL 复制服务器中具有唯一标识（源和副本集群上不能共享相同 ID 的复制服务器）。这可以通过使用`--server-id=*`id`*`选项启动每个 SQL 节点来完成，其中*`id`*是唯一的整数。尽管这并非绝对必要，但我们在本讨论中假设所有 NDB Cluster 二进制文件都是相同版本。

在 MySQL 复制中通常是真实的，涉及的两个 MySQL 服务器（**mysqld** 进程）必须在复制协议版本和它们支持的 SQL 功能集方面彼此兼容（请参见 Section 19.5.2，“MySQL 版本之间的复制兼容性”）。由于 NDB 集群和 MySQL Server 8.0 发行版之间的二进制文件之间的差异，NDB 集群复制有额外的要求，即两个 **mysqld** 二进制文件必须来自 NDB 集群发行版。确保 **mysqld** 服务器兼容的最简单和最容易的方法是对所有源和副本 **mysqld** 二进制文件使用相同的 NDB 集群发行版。

我们假设副本服务器或集群专用于复制源集群，并且没有其他数据存储在其中。

所有被复制的 `NDB` 表必须使用 MySQL 服务器和客户端创建。使用 NDB API 创建的表和其他数据库对象（例如，使用 `Dictionary::createTable()`）对 MySQL 服务器不可见，因此不会被复制。通过 NDB API 应用程序对使用 MySQL 服务器创建的现有表的更新可以被复制。

注意

可以使用基于语句的复制来复制 NDB 集群。但是，在这种情况下，以下限制适用：

+   所有对作为源的集群上数据行的更新必须指向单个 MySQL 服务器。

+   不可能使用多个同时的 MySQL 复制进程来复制集群。

+   只有在 SQL 级别进行的更改才会被复制。

这些是基于语句的复制相对于基于行的复制的其他限制；有关两种复制格式之间差异的更具体信息，请参见 Section 19.2.1.1，“基于语句和基于行的复制的优缺点”。
