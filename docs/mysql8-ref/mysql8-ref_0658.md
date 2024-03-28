# 11.2.1 标识符长度限制

> 原文：[`dev.mysql.com/doc/refman/8.0/en/identifier-length.html`](https://dev.mysql.com/doc/refman/8.0/en/identifier-length.html)

以下表描述了每种标识符类型的最大长度。

| 标识符类型 | 最大长度（字符） |
| --- | --- |
| 数据库 | 64（包括 NDB Cluster 8.0.18 及更高版本） |
| 表 | 64（包括 NDB Cluster 8.0.18 及更高版本） |
| 列 | 64 |
| 索引 | 64 |
| 约束 | 64 |
| 存储过程 | 64 |
| 视图 | 64 |
| 表空间 | 64 |
| 服务器 | 64 |
| 日志文件组 | 64 |
| 别名 | 256（见表后的例外） |
| 复合语句标签 | 16 |
| 用户定义变量 | 64 |
| 资源组 | 64 |
| 标识符类型 | 最大长度（字符） |

在`CREATE VIEW`语句中，列名的别名将与最大列长度为 64 个字符进行检查（而不是最大别名长度为 256 个字符）。

对于不包含约束名称的约束定义，服务器内部生成一个从关联表名派生的名称。例如，内部生成的外键和`CHECK`约束名称由表名加上 `_ibfk_` 或 `_chk_` 和一个数字组成。如果表名接近约束名称的长度限制，那么约束名称所需的额外字符可能导致该名称超过限制，从而导致错误。

标识符使用 Unicode（UTF-8）进行存储。这适用于表定义中的标识符以及存储在 `mysql` 数据库的授权表中的标识符。授权表中的标识符字符串列的大小以字符为单位。您可以使用多字节字符，而不会减少这些列中存储的值所允许的字符数。

在 NDB 8.0.18 之前，NDB Cluster 对数据库和表的名称 imposed 了最大长度为 63 个字符的限制。从 NDB 8.0.18 开始，此限制被移除。请参阅 Section 25.2.7.11, “Previous NDB Cluster Issues Resolved in NDB Cluster 8.0”。

MySQL 账户名称中的用户名称和主机名等值是字符串，而不是标识符。有关存储在授权表中的这些值的最大长度的信息，请参阅 Grant Table Scope Column Properties。