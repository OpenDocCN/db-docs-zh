# 28.3.27 INFORMATION_SCHEMA ROLE_COLUMN_GRANTS 表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/information-schema-role-column-grants-table.html`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-role-column-grants-table.html)

`ROLE_COLUMN_GRANTS` 表（自 MySQL 8.0.19 起可用）提供了关于当前启用角色可用或授予的列权限的信息。

`ROLE_COLUMN_GRANTS` 表具有以下列：

+   `GRANTOR`

    授予角色的帐户的用户名部分。

+   `GRANTOR_HOST`

    授予角色的帐户的主机名部分。

+   `GRANTEE`

    授予角色的帐户的用户名部分。

+   `GRANTEE_HOST`

    授予角色的帐户的主机名部分。

+   `TABLE_CATALOG`

    适用于角色的目录名称。该值始终为 `def`。

+   `TABLE_SCHEMA`

    适用于角色的模式（数据库）名称。

+   `TABLE_NAME`

    适用于角色的表名。

+   `COLUMN_NAME`

    适用于角色的列名。

+   `PRIVILEGE_TYPE`

    授予的权限。该值可以是可以在列级别授予的任何权限；请参阅 Section 15.7.1.6, “GRANT Statement”。每行列出一个权限，因此每个受让人持有的列权限都有一行。

+   `IS_GRANTABLE`

    `YES` 或 `NO`，取决于角色是否可授予给其他帐户。
