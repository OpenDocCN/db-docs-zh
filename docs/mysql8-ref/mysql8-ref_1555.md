# 20.8.2 Group Replication 离线升级

> 原文：[`dev.mysql.com/doc/refman/8.0/en/group-replication-offline-upgrade.html`](https://dev.mysql.com/doc/refman/8.0/en/group-replication-offline-upgrade.html)

要执行 Group Replication 组的离线升级，您需要将每个成员从组中移除，对成员进行升级，然后像往常一样重新启动组。在多主组中，您可以以任何顺序关闭成员。在单主组中，首先关闭每个次要成员，然后最后关闭主要成员。参见第 20.8.3.2 节，“升级 Group Replication 成员”了解如何从组中移除成员并关闭 MySQL。

一旦组离线，升级所有成员。参见第三章，*升级 MySQL*了解如何执行升级。当所有成员都升级完毕后，重新启动成员。

如果在组离线时升级所有复制组成员，然后重新启动组，成员将使用新版本的 Group Replication 通信协议版本加入，因此该版本将成为组的通信协议版本。如果您有要求允许较早版本的成员加入，您可以使用`group_replication_set_communication_protocol()`函数降级通信协议版本，指定具有最旧安装服务器版本的潜在组成员的 MySQL 服务器版本。
