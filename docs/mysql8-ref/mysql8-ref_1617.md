# 25.2.1 NDB 集群核心概念

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-basics.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-basics.html)

`NDBCLUSTER`（也称为`NDB`）是一种提供高可用性和数据持久性功能的内存存储引擎。

`NDBCLUSTER`存储引擎可以配置一系列故障转移和负载平衡选项，但最简单的方法是从集群级别开始配置存储引擎。NDB 集群的`NDB`存储引擎包含完整的数据集，仅依赖于集群内部的其他数据。

NDB 集群的“Cluster”部分独立于 MySQL 服务器进行配置。在 NDB 集群中，集群的每个部分被视为一个节点。

注意

在许多情境中，“节点”一词用于指代计算机，但在讨论 NDB 集群时，它指的是一个*进程*。可以在单台计算机上运行多个节点；对于运行一个或多个集群节点的计算机，我们使用术语集群主机。

有三种类型的集群节点，在最小的 NDB 集群配置中，至少有三个节点，每种类型一个：

+   管理节点：这种类型的节点的角色是管理 NDB 集群中的其他节点，执行提供配置数据、启动和停止节点以及运行备份等功能。由于这种节点类型管理其他节点的配置，因此应首先启动此类型的节点，然后再启动任何其他节点。管理节点使用命令**ndb_mgmd**启动。

+   数据节点：此类型节点存储集群数据。数据节点的数量与片段副本的数量相同，乘以片段的数量（参见第 25.2.2 节，“NDB Cluster 节点、节点组、片段副本和分区”）。例如，具有两个片段副本，每个副本有两个片段，您需要四个数据节点。一个片段副本足以用于数据存储，但不提供冗余；因此，建议具有两个（或更多）片段副本以提供冗余性，从而提供高可用性。数据节点使用命令**ndbd**（参见第 25.5.1 节，“ndbd — The NDB Cluster Data Node Daemon”）或**ndbmtd**")（参见第 25.5.3 节，“ndbmtd — The NDB Cluster Data Node Daemon (Multi-Threaded)”")）启动。

    NDB Cluster 表通常完全存储在内存中，而不是在磁盘上（这就是为什么我们将 NDB Cluster 称为内存数据库）。然而，一些 NDB Cluster 数据可以存储在磁盘上；有关更多信息，请参见第 25.6.11 节，“NDB Cluster 磁盘数据表”。

+   SQL 节点：这是访问集群数据的节点。在 NDB Cluster 的情况下，SQL 节点是使用 `NDBCLUSTER` 存储引擎的传统 MySQL 服务器。SQL 节点是使用 `--ndbcluster` 和 `--ndb-connectstring` 选项启动的 **mysqld** 进程，这些选项在本章的其他地方有解释，可能还有其他 MySQL 服务器选项。

    SQL 节点实际上只是一种专门类型的 API 节点，用于指定任何访问 NDB Cluster 数据的应用程序。另一个 API 节点的示例是**ndb_restore** 实用程序，用于恢复集群备份。可以使用 NDB API 编写此类应用程序。有关 NDB API 的基本信息，请参见使用 NDB API 入门。

重要

在生产环境中不现实期望使用三节点设置。这种配置不提供冗余性；要从 NDB Cluster 的高可用性功能中受益，必须使用多个数据和 SQL 节点。强烈建议使用多个管理节点。

关于 NDB 集群中节点、节点组、片段副本和分区之间关系的简要介绍，请参见 Section 25.2.2, “NDB Cluster Nodes, Node Groups, Fragment Replicas, and Partitions”。

集群的配置涉及配置集群中的每个单独节点，并设置节点之间的单独通信链接。NDB 集群当前设计的目的是数据节点在处理器性能、内存空间和带宽方面是同质的。此外，为了提供单一的配置点，集群作为整体的所有配置数据都位于一个配置文件中。

管理服务器管理集群配置文件和集群日志。集群中的每个节点从管理服务器检索配置数据，因此需要一种确定管理服务器位置的方式。当数据节点发生有趣的事件时，节点会将有关这些事件的信息传输到管理服务器，然后管理服务器将信息写入集群日志。

此外，可以有任意数量的集群客户端进程或应用程序。这些包括标准 MySQL 客户端、`NDB`特定的 API 程序和管理客户端。下面几段将对这些进行描述。

**标准 MySQL 客户端。** NDB 集群可以与使用 PHP、Perl、C、C++、Java、Python 等编写的现有 MySQL 应用程序一起使用。这些客户端应用程序向 MySQL 服务器发送 SQL 语句，并从充当 NDB 集群 SQL 节点的 MySQL 服务器接收响应，方式与它们与独立的 MySQL 服务器交互的方式基本相同。

使用 NDB 集群作为数据源的 MySQL 客户端可以修改以利用与多个 MySQL 服务器连接以实现负载平衡和故障转移的能力。例如，使用 Connector/J 5.0.6 及更高版本的 Java 客户端可以使用`jdbc:mysql:loadbalance://` URL（在 Connector/J 5.1.7 中改进）透明地实现负载平衡；有关使用 Connector/J 与 NDB 集群的更多信息，请参见 Using Connector/J with NDB Cluster。

**NDB 客户端程序。** 可以编写客户端程序，直接从`NDBCLUSTER`存储引擎访问 NDB 集群数据，绕过可能连接到集群的任何 MySQL 服务器，使用 NDB API，一个高级 C++ API。这样的应用程序可能对不需要数据的 SQL 接口的专用目的很有用。有关更多信息，请参见 The NDB API。

还可以使用 NDB Cluster Connector for Java 为 NDB Cluster 编写`NDB`特定的 Java 应用程序。这个 NDB Cluster Connector 包括 ClusterJ，一个类似于 Hibernate 和 JPA 等对象关系映射持久性框架的高级数据库 API，直接连接到`NDBCLUSTER`，因此不需要访问 MySQL 服务器。有关更多信息，请参阅 Java 和 NDB Cluster，以及 ClusterJ API 和数据对象模型。

NDB Cluster 还支持使用 Node.js 编写的 JavaScript 应用程序。MySQL Connector for JavaScript 包括适配器，用于直接访问`NDB`存储引擎以及 MySQL 服务器。使用此连接器的应用程序通常是事件驱动的，并且使用类似于 ClusterJ 所采用的领域对象模型。有关更多信息，请参阅 MySQL NoSQL Connector for JavaScript。

**管理客户端。** 这些客户端连接到管理服务器，并提供命令以优雅地启动和停止节点，启动和停止消息跟踪（仅限调试版本），显示节点版本和状态，启动和停止备份等。这种类型程序的示例是 NDB Cluster 提供的**ndb_mgm**管理客户端（请参阅 Section 25.5.5，“ndb_mgm — NDB Cluster 管理客户端”）。此类应用程序可以使用 MGM API 编写，这是一个与一个或多个 NDB Cluster 管理服务器直接通信的 C 语言 API。有关更多信息，请参阅 MGM API。

Oracle 还提供了 MySQL Cluster Manager，它提供了一个高级命令行界面，简化了许多复杂的 NDB Cluster 管理任务，比如重新启动具有大量节点的 NDB Cluster。MySQL Cluster Manager 客户端还支持获取和设置大多数节点配置参数以及与 NDB Cluster 相关的**mysqld**服务器选项和变量的命令。MySQL Cluster Manager 8.0 支持 NDB 8.0。有关更多信息，请参阅 MySQL Cluster Manager 8.0.36 用户手册。

**事件日志。** NDB Cluster 按类别（启动、关闭、错误、检查点等）、优先级和严重性记录事件。所有可报告事件的完整列表可在 Section 25.6.3，“NDB Cluster 生成的事件报告”中找到。事件日志有以下两种类型：

+   集群日志：记录整个集群的所有所需报告事件。

+   节点日志：每个单独节点都保留的单独日志。

注意

在正常情况下，只需保留和检查集群日志即可。节点日志只需在应用程序开发和调试目的时进行查看。

**检查点。** 一般来说，当数据保存到磁盘时，就说达到了一个检查点。更具体地说，对于 NDB 集群，检查点是一个时间点，所有已提交的事务都存储在磁盘上。关于`NDB`存储引擎，有两种类型的检查点共同工作，以确保维护集群数据的一致视图。这些显示在以下列表中：

+   本地检查点（LCP）：这是特定于单个节点的检查点；然而，LCP 几乎同时发生在集群中的所有节点上。LCP 通常每隔几分钟发生一次；确切的间隔会有所变化，取决于节点存储的数据量、集群活动水平和其他因素。

    NDB 8.0 支持部分 LCP，可以在某些条件下显著提高性能。请参阅`EnablePartialLcp`和`RecoveryWork`配置参数的描述，这些参数启用部分 LCP 并控制它们使用的存储量。

+   全局检查点（GCP）：每隔几秒钟会发生一次 GCP，当所有节点的事务同步并且重做日志刷新到磁盘时。

有关本地检查点和全局检查点创建的文件和目录的更多信息，请参阅 NDB 集群数据节点文件系统目录。

**传输器。** 我们使用传输器一词来表示数据节点之间使用的数据传输机制。MySQL NDB 集群 8.0 支持以下三种传输器：

+   *以太网上的 TCP/IP*。参见第 25.4.3.10 节，“NDB 集群 TCP/IP 连接”。

+   *直接 TCP/IP*。使用机器到机器的连接。参见第 25.4.3.11 节，“NDB 集群使用直接连接的 TCP/IP 连接”。

    尽管这个传输器使用与前一项提到的相同的 TCP/IP 协议，但需要不同的硬件设置和不同的配置。因此，它被认为是 NDB 集群的一个独立传输机制。

+   *共享内存（SHM）*。参见第 25.4.3.12 节，“NDB 集群共享内存连接”。

由于它是无处不在的，大多数用户在 NDB 集群中使用 TCP/IP over Ethernet。

无论使用何种传输器，`NDB` 都会尽力确保数据节点进程之间的通信使用尽可能大的块进行，因为这有利于所有类型的数据传输。
