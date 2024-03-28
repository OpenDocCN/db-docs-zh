> 原文：[`dev.mysql.com/doc/refman/8.0/en/audit-log-restrictions.html`](https://dev.mysql.com/doc/refman/8.0/en/audit-log-restrictions.html)

#### 8.4.5.12 审计日志限制

MySQL Enterprise Audit 受到以下一般限制：

+   仅记录 SQL 语句。通过非 SQL API（如 memcached、Node.JS 和 NDB API）进行的更改不会被记录。

+   仅记录顶层语句，不记录存储程序内部的语句，如触发器或存储过程中的语句。

+   诸如 `LOAD DATA` 这类语句引用的文件内容不会被记录。

**NDB 集群。** 可以在 MySQL NDB 集群中使用 MySQL Enterprise Audit，但需符合以下条件：

+   所有要记录的更改必须使用 SQL 接口完成。使用非 SQL 接口进行的更改（如 NDB API、memcached 或 ClusterJ 提供的接口）不会被记录。

+   插件必须安装在用于在集群上执行 SQL 的每个 MySQL 服务器上。

+   审计插件数据必须在用于集群的所有 MySQL 服务器之间进行聚合。这种聚合是应用程序或用户的责任。
