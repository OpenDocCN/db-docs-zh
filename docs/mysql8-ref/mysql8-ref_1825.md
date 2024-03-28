# 25.7.1 NDB 集群复制：缩写和符号

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-replication-abbreviations.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-replication-abbreviations.html)

在本节中，我们使用以下缩写或符号来指代源集群和副本集群，以及在集群或集群节点上运行的进程和命令：

**表 25.69 本节中使用的缩写，指的是源集群和副本集群，以及在集群节点上运行的进程和命令**

| 符号或缩写 | 描述（指的是...） |
| --- | --- |
| *`S`* | 充当（主要）复制源的集群 |
| *`R`* | 充当（主要）副本的集群 |
| `shell*`S`*>` | 在源集群上发出的 shell 命令 |
| `mysql*`S`*>` | 在作为 SQL 节点运行的单个 MySQL 服务器上发出的 MySQL 客户端命令 |
| `mysql*`S*`*>` | 在参与复制源集群的所有 SQL 节点上发出的 MySQL 客户端命令 |
| `shell*`R`*>` | 在副本集群上发出的 shell 命令 |
| `mysql*`R`*>` | 在副本集群上作为 SQL 节点运行的单个 MySQL 服务器上发出的 MySQL 客户端命令 |
| `mysql*`R*`*>` | 在参与副本集群的所有 SQL 节点上发出的 MySQL 客户端命令 |
| *`C`* | 主要复制通道 |
| *`C'`* | 次要复制通道 |
| *`S'`* | 次要复制源 |
| *`R'`* | 次要副本 |
| 符号或缩写 | 描述（指的是...） |
