# 20.1.3 多主和单主模式

> 原文：[`dev.mysql.com/doc/refman/8.0/en/group-replication-deploying-in-multi-primary-or-single-primary-mode.html`](https://dev.mysql.com/doc/refman/8.0/en/group-replication-deploying-in-multi-primary-or-single-primary-mode.html)

20.1.3.1 单主模式

20.1.3.2 多主模式

Group Replication 可以在单主模式或多主模式下运行。该组的模式是一个全组配置设置，由`group_replication_single_primary_mode`系统变量指定，所有成员必须相同。`ON`表示单主模式，这是默认模式，`OFF`表示多主模式。不可能让组的成员以不同模式部署，例如一个成员配置为多主模式，而另一个成员处于单主模式。

在 Group Replication 运行时无法手动更改`group_replication_single_primary_mode`的值。从 MySQL 8.0.13 开始，您可以使用`group_replication_switch_to_single_primary_mode()`和`group_replication_switch_to_multi_primary_mode()`函数在 Group Replication 仍在运行时将组从一种模式转换到另一种模式。这些函数管理更改组模式的过程，并确保数据的安全性和一致性。在早期版本中，要更改组的模式，您必须停止 Group Replication 并在所有成员上更改`group_replication_single_primary_mode`的值。然后执行组的完全重启（由具有`group_replication_bootstrap_group=ON`的服务器引导）以实施对新操作配置的更改。您无需重新启动服务器。

无论部署模式如何，Group Replication 不处理客户端故障转移。这必须由中间件框架（如 MySQL Router 8.0）、代理、连接器或应用程序本身处理。
