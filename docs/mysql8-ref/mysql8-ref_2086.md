> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-firewall-membership-table.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-firewall-membership-table.html)

#### 29.12.17.3 The firewall_membership Table

`firewall_membership` 表提供了一个查看 MySQL Enterprise Firewall 内存数据缓存的视图。它列出了注册防火墙组个人资料的成员（账户）。它与提供防火墙数据持久存储的 `mysql.firewall_membership` 系统表一起使用；参见 MySQL Enterprise Firewall Tables。

`firewall_membership` 表有以下列：

+   `GROUP_ID`

    组个人资料名称。

+   `MEMBER_ID`

    一个属于个人资料的账户名称。

`firewall_membership` 表没有索引。

`TRUNCATE TABLE` 不允许用于 `firewall_membership` 表。

`firewall_membership` 表在 MySQL 8.0.23 中添加。
