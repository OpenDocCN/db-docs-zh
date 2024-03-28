# 28.3.29 INFORMATION_SCHEMA ROLE_TABLE_GRANTS 表

> [`dev.mysql.com/doc/refman/8.0/en/information-schema-role-table-grants-table.html`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-role-table-grants-table.html)

`ROLE_TABLE_GRANTS` 表（自 MySQL 8.0.19 起可用）提供有关当前启用角色可用或授予的角色的表权限的信息。

`ROLE_TABLE_GRANTS` 表包含以下列：

+   `GRANTOR`

    授予角色的帐户的用户名部分。

+   `GRANTOR_HOST`

    授予角色的帐户的主机名部分。

+   `GRANTEE`

    授予角色的帐户的用户名部分。

+   `GRANTEE_HOST`

    授予角色的帐户的主机名部分。

+   `TABLE_CATALOG`

    角色适用的目录名称。该值始终为`def`。

+   `TABLE_SCHEMA`

    角色适用的模式（数据库）的名称。

+   `TABLE_NAME`

    角色适用的表的名称。

+   `PRIVILEGE_TYPE`

    授予的权限。该值可以是在表级别授予的任何权限；请参阅 Section 15.7.1.6, “GRANT Statement”。每行列出一个权限，因此每个被授予者持有的列权限都有一行。

+   `IS_GRANTABLE`

    `YES` 或 `NO`，取决于角色是否可授予其他帐户。
