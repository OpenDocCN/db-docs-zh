> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-limitations-resolved.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-limitations-resolved.html)

#### 25.2.7.11 NDB Cluster 8.0 中已解决的以前的 NDB Cluster 问题

以前版本的 NDB Cluster 中存在的许多限制和相关问题已在 NDB 8.0 中得到解决。以下简要描述了这些问题：

+   **数据库和表名。** 在 NDB 7.6 及更早版本中，使用 `NDB` 存储引擎时，数据库名和表名的最大允许长度均为 63 字节，使用超过此限制的数据库名或表名的语句将失败并显示适当的错误。在 NDB 8.0 中，此限制被取消；`NDB` 数据库和表的标识符现在可以使用最多 64 个字符，与其他 MySQL 数据库和表名一样。

+   **IPv6 支持。** 在 NDB 8.0.22 之前，NDB Cluster 中节点之间连接所使用的所有网络地址必须使用或解析为 IPv4 地址。从 NDB 8.0.22 开始，`NDB` 支持所有类型的集群节点和使用 NDB API 或 MGM API 的应用程序的 IPv6 地址。

    欲了解更多信息，请参阅升级或降级 NDB Cluster 时已知的问题。

+   **多线程副本。** 在 NDB 8.0.32 及更早版本中，不支持 NDB Cluster 复制的多线程副本。此限制在 NDB Cluster 8.0.33 中解除。

    欲了解更多信息，请参阅第 25.7.3 节，“NDB Cluster 复制中已知的问题”。
