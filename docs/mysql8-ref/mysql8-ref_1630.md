> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-limitations-error-handling.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-limitations-error-handling.html)

#### 25.2.7.4 NDB 集群错误处理

启动、停止或重新启动节点可能会导致临时错误，导致一些事务失败。这些情况包括以下情况：

+   **临时错误。** 当首次启动节点时，可能会出现错误 1204 临时故障，分布发生变化和类似临时错误。

+   **由于节点故障而导致的错误。** 任何数据节点的停止或故障都可能导致多种不同的节点故障错误。（但在执行计划关闭集群时不应有中止事务。）

在这两种情况下，生成的任何错误都必须在应用程序内处理。这应该通过重试事务来完成。

另请参阅第 25.2.7.2 节，“NDB 集群与标准 MySQL 限制的限制和差异”。
