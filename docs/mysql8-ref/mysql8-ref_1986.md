# 28.7.2 INFORMATION_SCHEMA MYSQL_FIREWALL_USERS 表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/information-schema-mysql-firewall-users-table.html`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-mysql-firewall-users-table.html)

`MYSQL_FIREWALL_USERS` 表提供了对 MySQL 企业防火墙内存数据缓存的视图。它列出了注册的防火墙账户配置文件的名称和操作模式。它与提供防火墙数据持久存储的 `mysql.firewall_users` 系统表一起使用；请参阅 MySQL 企业防火墙表。

`MYSQL_FIREWALL_USERS` 表包含以下列：

+   `USERHOST`

    账户配置文件名称。每个账户名称的格式为 `*`user_name`*@*`host_name`*`。

+   `MODE`

    该配置文件的当前操作模式。允许的模式值为 `OFF`, `DETECTING`, `PROTECTING`, `RECORDING`, 和 `RESET`。有关它们含义的详细信息，请参阅防火墙概念。

截至 MySQL 8.0.26，此表已被弃用，并可能在未来的 MySQL 版本中移除。请参阅将账户配置文件迁移到组配置文件。
