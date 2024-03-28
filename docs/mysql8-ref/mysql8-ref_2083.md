# 29.12.17 性能模式防火墙表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-firewall-tables.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-firewall-tables.html)

29.12.17.1 firewall_groups 表

29.12.17.2 firewall_group_allowlist 表

29.12.17.3 firewall_membership 表

注意

此处描述的性能模式表在 MySQL 8.0.23 版本中可用。在 MySQL 8.0.23 之前，请改用相应的 `INFORMATION_SCHEMA` 表；请参阅 MySQL 企业防火墙表。

以下各节描述了与 MySQL 企业防火墙相关的性能模式表（请参阅 第 8.4.7 节，“MySQL 企业防火墙”）。它们提供有关防火墙操作的信息：

+   `firewall_groups`: 关于防火墙组配置文件的信息。

+   `firewall_group_allowlist`: 注册防火墙组配置文件的允许列表规则。

+   `firewall_membership`: 注册防火墙组配置文件的成员（帐户）。
