> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-accounts-table.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-accounts-table.html)

#### 29.12.8.1 账户表

`accounts`表包含每个连接到 MySQL 服务器的账户的行。对于每个账户，该表计算当前和总连接数。表大小在服务器启动时自动调整。要显式设置表大小，请在服务器启动时设置`performance_schema_accounts_size`系统变量。要禁用账户统计信息，请将此变量设置为 0。

`accounts`表具有以下列。有关性能模式如何在此表中维护行的描述，包括`TRUNCATE TABLE`的影响，请参见第 29.12.8 节，“性能模式连接表”。

+   `用户`

    连接的客户端用户名。对于内部线程或未能进行身份验证的用户会话，此处为`NULL`。

+   `主机`

    客户端连接的主机。对于内部线程或未能进行身份验证的用户会话，此处为`NULL`。

+   `当前连接数`

    该账户的当前连接数。

+   `总连接数`

    该账户的总连接数。

+   `最大会话受控内存`

    报告属于该账户的会话使用的最大受控内存量。

    这一列是在 MySQL 8.0.31 中添加的。

+   `最大会话总内存`

    报告属于该账户的会话使用的最大内存量。

    这一列是在 MySQL 8.0.31 中添加的。

`accounts`表具有以下索引：

+   主键为(`用户`, `主机`)
