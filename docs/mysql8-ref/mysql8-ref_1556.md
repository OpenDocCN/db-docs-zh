# 20.8.3 集群复制在线升级

> 原文：[`dev.mysql.com/doc/refman/8.0/en/group-replication-online-upgrade.html`](https://dev.mysql.com/doc/refman/8.0/en/group-replication-online-upgrade.html)

20.8.3.1 在线升级考虑事项

20.8.3.2 升级集群复制成员

20.8.3.3 集群复制在线升级方法

20.8.3.4 使用**mysqlbackup**进行集群复制升级

当您有一个正在运行的集群需要升级，但您需要保持集群在线以提供应用程序服务时，您需要考虑升级的方法。本节描述了在线升级涉及的不同元素，以及如何升级您的集群的各种方法。
