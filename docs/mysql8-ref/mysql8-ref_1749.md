> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-tde-limitations.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-tde-limitations.html)

#### 25.6.14.3 NDB 文件系统加密限制

NDB 集群中的透明数据加密受以下限制和限制：

+   文件系统密码必须提供给每个单独的数据节点。

+   文件系统密码轮换需要对数据节点进行初始滚动重启；这必须手动执行，或者由`NDB`外部应用程序执行。

+   对于仅具有单个副本的集群（`NoOfReplicas = 1`），需要进行完整备份和恢复以进行文件系统密码轮换。

+   所有数据加密密钥的轮换需要初始节点重新启动。
