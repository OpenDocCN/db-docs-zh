# 第二十四章 InnoDB ReplicaSet

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-innodb-replicaset-introduction.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-innodb-replicaset-introduction.html)

本章介绍了 MySQL InnoDB ReplicaSet，它结合了 MySQL 技术，使您能够部署和管理第十九章，*复制*。本内容是 InnoDB ReplicaSet 的高级概述，有关完整文档，请参阅 MySQL InnoDB ReplicaSet。

一个 InnoDB ReplicaSet 至少由两个 MySQL 服务器实例组成，并提供您熟悉的所有 MySQL 复制功能，如读取扩展和数据安全。InnoDB ReplicaSet 使用以下 MySQL 技术：

+   MySQL Shell，这是一个用于 MySQL 的高级客户端和代码编辑器。

+   MySQL 服务器和第十九章，*复制*，使一组 MySQL 实例能够提供可用性和异步读取扩展。InnoDB ReplicaSet 提供了一种替代的、易于使用的编程方式来处理复制。

+   MySQL Router，一个轻量级中间件，提供应用程序与 InnoDB ReplicaSet 之间的透明路由。

InnoDB ReplicaSet 的接口类似于 MySQL InnoDB Cluster，您可以使用 MySQL Shell 来操作 MySQL 服务器实例作为一个 ReplicaSet，并且 MySQL Router 也与 InnoDB Cluster 紧密集成。

基于 MySQL 复制，InnoDB ReplicaSet 有一个主实例，它复制到一个或多个次要实例。InnoDB ReplicaSet 不提供 InnoDB Cluster 提供的所有功能，如自动故障转移或多主模式。但是，它支持配置、添加和删除实例等功能的方式类似。例如，在发生故障时，您可以手动切换到或故障转移到次要实例。甚至可以采用现有的复制部署，然后将其管理为 InnoDB ReplicaSet。

您可以使用 AdminAPI 来操作 InnoDB ReplicaSet，该 API 作为 MySQL Shell 的一部分提供。AdminAPI 可用于 JavaScript 和 Python，并非常适合脚本编写和自动化部署 MySQL 以实现高可用性和可伸缩性。通过使用 MySQL Shell 的 AdminAPI，您可以避免手动配置许多实例的需要。相反，AdminAPI 提供了一种有效的现代接口来管理 MySQL 实例集，并使您能够从一个中心工具中进行部署、管理和监控。

要开始使用 InnoDB ReplicaSet，您需要[下载](https://dev.mysql.com/downloads/shell/)和安装 MySQL Shell。您需要一些安装了 MySQL Server 实例的主机，并且您还可以安装 MySQL Router。

InnoDB ReplicaSet 支持 MySQL Clone，这使您可以简单地提供实例。过去，在新实例加入 MySQL 复制部署之前，您需要以某种方式手动将事务传输到加入实例。这可能涉及制作文件副本，手动复制它们等等。您只需添加一个实例到复制集中，它就会自动提供。

同样，InnoDB ReplicaSet 与 MySQL Router 紧密集成，您可以使用 AdminAPI 一起使用它们。MySQL Router 可以根据 InnoDB ReplicaSet 自动配置自身，在一个称为引导过程的过程中，这样您就无需手动配置路由。MySQL Router 然后透明地连接客户端应用程序到 InnoDB ReplicaSet，为客户端连接提供路由和负载平衡。这种集成还使您能够使用 AdminAPI 管理针对 InnoDB ReplicaSet 引导的 MySQL Router 的某些方面。InnoDB ReplicaSet 状态信息包括有关针对 ReplicaSet 引导的 MySQL Router 的详细信息。操作使您能够在 ReplicaSet 级别创建 MySQL Router 用户，与针对 ReplicaSet 引导的 MySQL Router 一起工作，等等。

关于这些技术的更多信息，请参阅描述中链接的用户文档。除了这些用户文档外，还有关于 MySQL Shell JavaScript API 参考或 MySQL Shell Python API 参考中所有 AdminAPI 方法的开发人员文档，可从 Connectors and APIs 获取。
