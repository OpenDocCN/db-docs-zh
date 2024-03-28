> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-firewall-group-allowlist-table.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-firewall-group-allowlist-table.html)

#### 29.12.17.2 The firewall_group_allowlist Table

`firewall_group_allowlist` 表提供了对 MySQL Enterprise Firewall 内存数据缓存的视图。它列出了已注册防火墙组配置文件的允许规则。它与提供防火墙数据持久存储的 `mysql.firewall_group_allowlist` 系统表一起使用；请参阅 MySQL Enterprise Firewall Tables。

`firewall_group_allowlist` 表包含以下列：

+   `NAME`

    组配置文件名称。

+   `RULE`

    表示配置文件可接受语句模式的规范化语句。配置文件允许列表是其规则的并集。

`firewall_group_allowlist` 表没有索引。

`TRUNCATE TABLE` 不允许用于 `firewall_group_allowlist` 表。

`firewall_group_allowlist` 表在 MySQL 8.0.23 中添加。
