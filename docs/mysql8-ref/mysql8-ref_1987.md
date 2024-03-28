# 28.7.3 [INFORMATION_SCHEMA MYSQL_FIREWALL_WHITELIST](https://dev.mysql.com/doc/refman/8.0/en/information-schema-mysql-firewall-whitelist-table.html) 表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/information-schema-mysql-firewall-whitelist-table.html`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-mysql-firewall-whitelist-table.html)

`MYSQL_FIREWALL_WHITELIST` 表提供了对 MySQL 企业防火墙内存数据缓存的视图。它列出了已注册防火墙帐户配置文件的允许列表规则。它与提供防火墙数据持久存储的`mysql.firewall_whitelist`系统表一起使用；请参阅 MySQL 企业防火墙表。

`MYSQL_FIREWALL_WHITELIST` 表具有以下列：

+   `USERHOST`

    帐户配置文件名称。每个帐户名称的格式为`*`user_name`*@*`host_name`*`。

+   `RULE`

    表示配置文件中可接受的语句模式的规范化语句。配置文件允许列表是其规则的并集。

截至 MySQL 8.0.26，此表已被弃用，并可能在将来的 MySQL 版本中删除。请参阅将帐户配置文件迁移到组配置文件。
