> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-log-startup-messages.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-log-startup-messages.html)

#### 25.6.2.2 NDB 集群日志启动消息

可能的启动消息及其描述如下列表所示：

+   `初始启动，等待 %s 连接，节点 [ 所有: %s 已连接: %s 无等待: %s ]`

+   `等待直到节点 %s 连接，节点 [ 所有: %s 已连接: %s 无等待: %s ]`

+   `等待 %u 秒以连接节点 %s，节点 [ 所有: %s 已连接: %s 无等待: %s ]`

+   `等待非分区启动，节点 [ 所有: %s 已连接: %s 缺失: %s 无等待: %s ]`

+   `等待 %u 秒进行非分区启动，节点 [ 所有: %s 已连接: %s 缺失: %s 无等待: %s ]`

+   `初始启动节点为 %s [ 缺失: %s 无等待: %s ]`

+   `启动所有节点 %s`

+   `启动节点为 %s [ 缺失: %s 无等待: %s ]`

+   `启动可能已分区，节点为 %s [ 缺失: %s 无等待: %s ]`

+   `未知的启动报告: 0x%x [ %s %s %s %s ]`
