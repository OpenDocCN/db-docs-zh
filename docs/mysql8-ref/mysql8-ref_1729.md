# 25.6.7 添加 NDB 集群数据节点在线

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-online-add-node.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-online-add-node.html)

25.6.7.1 添加 NDB 集群数据节点在线：一般问题

25.6.7.2 添加 NDB 集群数据节点在线：基本过程

25.6.7.3 添加 NDB 集群数据节点在线：详细示例

本节描述了如何“在线”添加 NDB 集群数据节点，即在不需要完全关闭集群并重新启动的情况下进行该过程。

重要提示

目前，您必须将新数据节点添加到 NDB 集群作为新节点组的一部分。此外，无法在线更改片段副本数（或节点组中的节点数）。
