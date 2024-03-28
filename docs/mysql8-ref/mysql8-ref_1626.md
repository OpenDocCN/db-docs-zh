# 25.2.7 NDB Cluster 的已知限制

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-limitations.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-limitations.html)

25.2.7.1 NDB Cluster 中与 SQL 语法不符的限制

25.2.7.2 NDB Cluster 与标准 MySQL 限制的限制和差异

25.2.7.3 NDB Cluster 中与事务处理相关的限制

25.2.7.4 NDB Cluster 错误处理

25.2.7.5 NDB Cluster 中与数据库对象相关的限制

25.2.7.6 NDB Cluster 中不支持或缺失的功能

25.2.7.7 NDB Cluster 中与性能相关的限制

25.2.7.8 仅限于 NDB Cluster 的问���

25.2.7.9 NDB Cluster 磁盘数据存储相关的限制

25.2.7.10 与多个 NDB Cluster 节点相关的限制

25.2.7.11 在 NDB Cluster 8.0 中已解决的以前的 NDB Cluster 问题

在接下来的章节中，我们将讨论当前 NDB Cluster 版本中已知的限制，与使用 `MyISAM` 和 `InnoDB` 存储引擎时可用功能的比较。如果您在 MySQL bugs 数据库的“Cluster”类别中检查 [`bugs.mysql.com`](http://bugs.mysql.com)，您可以找到以下类别下的已知 bug：“MySQL Server:”，我们打算在即将发布的 NDB Cluster 版本中进行修正：

+   NDB Cluster

+   Cluster 直接 API (NDBAPI)

+   Cluster 磁盘数据

+   Cluster 复制

+   ClusterJ

此信息旨在完整地描述刚刚设置的条件。您可以按照 第 1.5 节, “如何报告错误或问题” 中给出的说明，将您遇到的任何差异报告给 MySQL bugs 数据库。我们不打算在 NDB Cluster 8.0 中修复的任何问题都将添加到列表中。

查看 第 25.2.7.11 节, “在 NDB Cluster 8.0 中已解决的以前的 NDB Cluster 问题”，列出了在 NDB Cluster 8.0 中已解决的早期版本中的问题。

注意

NDB Cluster 复制特定的限制和其他问题在 第 25.7.3 节, “NDB Cluster 复制中已知问题” 中有描述。
