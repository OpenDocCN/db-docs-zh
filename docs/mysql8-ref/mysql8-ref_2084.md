> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-firewall-groups-table.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-firewall-groups-table.html)

#### 29.12.17.1 防火墙组表

`firewall_groups` 表提供了对 MySQL 企业防火墙内存数据缓存的视图。它列出了注册的防火墙组配置文件的名称和操作模式。它与提供防火墙数据持久存储的 `mysql.firewall_groups` 系统表一起使用；请参阅 MySQL 企业防火墙表。

`firewall_groups` 表具有以下列：

+   `名称`

    组配置文件名称。

+   `模式`

    配置文件的当前操作模式。允许的模式值为 `OFF`、`DETECTING`、`PROTECTING` 和 `RECORDING`。有关其含义的详细信息，请参阅 防火墙概念。

+   `用户主机`

    组配置文件的训练账户，在配置文件处于 `RECORDING` 模式时使用。数值为 `NULL`，或格式为 `*`user_name`*@*`host_name`*` 的非`NULL`账户：

    +   如果数值为 `NULL`，防火墙记录允许来自任何组成员账户的语句。

    +   如果数值为非`NULL`，防火墙记录仅允许来自指定账户（应为组成员）的语句。

`firewall_groups` 表没有索引。

不允许对 `firewall_groups` 表使用 `TRUNCATE TABLE`。

`firewall_groups` 表在 MySQL 8.0.23 中添加。
