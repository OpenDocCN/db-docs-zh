> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-users-table.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-users-table.html)

#### 29.12.8.3 用户表

`users` 表包含每个连接到 MySQL 服务器的用户的一行。对于每个用户名，表计算当前和总连接数。表的大小在服务器启动时自动调整。要显式设置表大小，请在服务器启动时设置 `performance_schema_users_size` 系统变量。要禁用用户统计信息，请将此变量设置为 0。

`users` 表具有以下列。有关性能模式如何维护此表中的行的描述，包括 `TRUNCATE TABLE` 的影响，请参见 第 29.12.8 节，“性能模式连接表”。

+   `USER`

    连接的客户端用户名。对于内部线程或未能通过身份验证的用户会话，此值为 `NULL`。

+   `CURRENT_CONNECTIONS`

    用户的当前连接数。

+   `TOTAL_CONNECTIONS`

    用户的总连接数。

+   `MAX_SESSION_CONTROLLED_MEMORY`

    报告属于用户的会话使用的最大受控内存量。

    此列在 MySQL 8.0.31 中添加。

+   `MAX_SESSION_TOTAL_MEMORY`

    报告属于用户的会话使用的最大内存量。

    此列在 MySQL 8.0.31 中添加。

`users` 表具有以下索引：

+   主键为 (`USER`)
