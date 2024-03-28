# 20.2 开始

> 原文：[`dev.mysql.com/doc/refman/8.0/en/group-replication-getting-started.html`](https://dev.mysql.com/doc/refman/8.0/en/group-replication-getting-started.html)

20.2.1 在单主模式下部署群组复制

20.2.2 本地部署群组复制

MySQL Group Replication 是作为 MySQL 服务器的插件提供的；组中的每个服务器都需要配置和安装插件。本节提供了一个详细的教程，介绍了创建至少三个成员的复制组所需的步骤。

提示

要部署多个 MySQL 实例，您可以使用 InnoDB 集群，它使您能够轻松管理一组 MySQL 服务器实例在 MySQL Shell 中。InnoDB 集群将 MySQL Group Replication 封装在一个编程环境中，使您能够轻松部署一组 MySQL 实例以实现高可用性。此外，InnoDB 集群与 MySQL Router 无缝接口，使您的应用程序能够连接到集群而无需编写自己的故障转移过程。然而，对于不需要高可用性的类似用例，您可以使用 InnoDB ReplicaSet。有关 MySQL Shell 的安装说明，请参阅这里。
