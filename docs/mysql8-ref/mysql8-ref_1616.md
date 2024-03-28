# 25.2 NDB Cluster 概述

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-overview.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-overview.html)

25.2.1 NDB Cluster 核心概念

25.2.2 NDB Cluster 节点、节点组、片副本和分区

25.2.3 NDB Cluster 硬件、软件和网络要求

25.2.4 MySQL NDB Cluster 8.0 中的新功能

25.2.5 NDB 8.0 中添加、弃用或删除的选项、变量和参数

25.2.6 使用 InnoDB 的 MySQL 服务器与 NDB Cluster 比较

25.2.7 NDB Cluster 已知限制

NDB Cluster 是一种技术，可以在共享无关系统中对内存数据库进行集群化。共享无关架构使系统能够使用非常廉价的硬件，并且对硬件或软件的特定要求最少。

NDB Cluster 设计为没有任何单点故障。在共享无关系统中，预期每个组件都有自己的内存和磁盘，并且不建议或支持使用共享存储机制，如网络共享、网络文件系统和 SAN。

NDB Cluster 将标准 MySQL 服务器与名为`NDB`的内存集群存储引擎集成在一起（代表“*N*etwork *D*ata*B*ase”）。在我们的文档中，术语`NDB`指的是特定于存储引擎的设置部分，而“MySQL NDB Cluster”指的是一个或多个 MySQL 服务器与`NDB`存储引擎的组合。

一个 NDB Cluster 由一组计算机（称为主机）组成，每台主机运行一个或多个进程。这些进程称为节点，可能包括 MySQL 服务器（用于访问 NDB 数据）、数据节点（用于存储数据）、一个或多个管理服务器，以及可能的其他专门的数据访问程序。NDB Cluster 中这些组件的关系如下所示：

**图 25.1 NDB Cluster 组件**

![在这个集群中，三个 MySQL 服务器（mysqld 程序）是提供对存储数据的四个数据节点（ndbd 程序）的 SQL 节点。SQL 节点和数据节点受 NDB 管理服务器（ndb_mgmd 程序）控制。各种客户端和 API 可以与 SQL 节点交互 - mysql 客户端、MySQL C API、PHP、Connector/J 和 Connector/NET。还可以使用 NDB API 创建自定义客户端与数据节点或 NDB 管理服务器交互。NDB 管理客户端（ndb_mgm 程序）与 NDB 管理服务器交互。](img/b70cee53955396617b4a0c5cb25be64c.png)

所有这些程序共同组成一个 NDB 集群（参见第 25.5 节，“NDB 集群程序”）。当数据由`NDB`存储引擎存储时，表格（和表格数据）存储在数据节点中。这样的表格可以直接从集群中的所有其他 MySQL 服务器（SQL 节点）访问。因此，在一个将数据存储在集群中的工资应用程序中，如果一个应用程序更新了员工的工资，所有查询这些数据的其他 MySQL 服务器都可以立即看到这一变化。

从 NDB 8.0.31 开始，NDB 集群 8.0 SQL 节点使用与 MySQL Server 8.0 发行版提供的相同的**mysqld**服务器守护程序。在 NDB 8.0.30 和之前的版本中，它与 MySQL Server 提供的**mysqld**二进制文件在许多关键方面有所不同，而且这两个版本的**mysqld**是不可互换的。您应该记住，*一个未连接到 NDB 集群的任何版本的**mysqld**实例都无法使用`NDB`存储引擎，也无法访问任何 NDB 集群数据*。

存储在 NDB 集群数据节点中的数据可以进行镜像；集群可以处理单个数据节点的故障，除了一小部分事务由于丢失事务状态而中止之外，没有其他影响。因为事务应用程序预计会处理事务失败，所以这不应该成为问题的源头。

单个节点可以停止和重新启动，然后重新加入系统（集群）。滚动重启（逐个重新启动所有节点）用于进行配置更改和软件升级（参见第 25.6.5 节，“执行 NDB 集群的滚动重启”）。滚动重启也用作在线添加新数据节点的过程的一部分（参见第 25.6.7 节，“在线添加 NDB 集群数据节点”）。有关数据节点、它们在 NDB 集群中的组织方式以及它们如何处理和存储 NDB 集群数据的更多信息，请参见第 25.2.2 节，“NDB 集群节点、节点组、片段副本和分区”。

备份和恢复 NDB Cluster 数据库可以使用 NDB Cluster 管理客户端中找到的 `NDB`-本地功能以及 NDB Cluster 发行版中包含的 **ndb_restore** 程序。更多信息，请参见 Section 25.6.8, “Online Backup of NDB Cluster”，以及 Section 25.5.23, “ndb_restore — Restore an NDB Cluster Backup”。您还可以使用标准 MySQL 功能提供的 **mysqldump** 和 MySQL 服务器来实现此目的。有关更多信息，请参见 Section 6.5.4, “mysqldump — A Database Backup Program”。

NDB Cluster 节点可以使用不同的传输机制进行节点间通信；在大多数实际部署中，使用标准 100 Mbps 或更快的以太网硬件上的 TCP/IP。
