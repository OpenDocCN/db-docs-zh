> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-hosts-table.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-hosts-table.html)

#### 29.12.8.2 主机表

`hosts` 表包含每个连接到 MySQL 服务器的客户端主机的行。对于每个主机名，表计算当前和总连接数。表大小在服务器启动时自动调整。要显式设置表大小，请在服务器启动时设置`performance_schema_hosts_size` 系统变量。要禁用主机统计信息，请将此变量设置为 0。

`hosts` 表具有以下列。有关性能模式如何维护此表中的行的描述，包括`TRUNCATE TABLE`的影响，请参见第 29.12.8 节，“性能模式连接表”。

+   `HOST`

    客户端连接的主机。对于内部线程或未能通过身份验证的用户会话，此值为`NULL`。

+   `CURRENT_CONNECTIONS`

    主机的当前连接数。

+   `TOTAL_CONNECTIONS`

    主机的总连接数。

+   `MAX_SESSION_CONTROLLED_MEMORY`

    报告属于主机的会话使用的最大受控内存量。

    该列在 MySQL 8.0.31 中添加。

+   `MAX_SESSION_TOTAL_MEMORY`

    报告属于主机的会话使用的最大内存量。

    该列在 MySQL 8.0.31 中添加。

`hosts` 表具有以下索引：

+   主键为 (`HOST`)
