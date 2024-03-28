# 25.4.4 使用高速互连与 NDB 集群

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-interconnects.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-interconnects.html)

甚至在 1996 年设计`NDBCLUSTER`之前，就已经明显地意识到在构建并行数据库中将会遇到的一个主要问题是网络中节点之间的通信。因此，`NDBCLUSTER`从一开始就被设计成允许使用多种不同的数据传输机制或传输器。

NDB Cluster 8.0 支持其中三种（参见第 25.2.1 节，“NDB Cluster 核心概念”）。第四种传输器，可扩展一致性接口（SCI），也曾在非常旧的`NDB`版本中得到支持。这需要专门的硬件、软件和现在不再可用的 MySQL 二进制文件。
