# 19.1.7 常见的复制管理任务

> 原文：[`dev.mysql.com/doc/refman/8.0/en/replication-administration.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-administration.html)

19.1.7.1 检查复制状态

19.1.7.2 在副本上暂停复制

19.1.7.3 跳过事务

一旦复制已经启动，它将在不需要太多常规管理的情况下执行。本节描述了如何检查复制状态，如何暂停副本，以及如何跳过副本上的失败事务。

提示

要部署多个 MySQL 实例，您可以使用 InnoDB 集群，它使您能够轻松管理一组 MySQL 服务器实例在 MySQL Shell 中。InnoDB 集群将 MySQL Group Replication 包装在一个编程环境中，使您能够轻松部署一组 MySQL 实例以实现高可用性。此外，InnoDB 集群与 MySQL Router 无缝接口，使您的应用程序可以连接到集群而无需编写自己的故障转移过程。然而，对于不需要高可用性的类似用例，您可以使用 InnoDB ReplicaSet。有关 MySQL Shell 的安装说明可以在这里找到。
