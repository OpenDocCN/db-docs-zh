# 28.3.28 INFORMATION_SCHEMA ROLE_ROUTINE_GRANTS 表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/information-schema-role-routine-grants-table.html`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-role-routine-grants-table.html)

`ROLE_ROUTINE_GRANTS` 表（自 MySQL 8.0.19 起可用）提供有关当前启用角色可用或授予的角色例程特权的信息。

`ROLE_ROUTINE_GRANTS` 表具有以下列：

+   `GRANTOR`

    授予该角色的帐户的用户名部分。

+   `GRANTOR_HOST`

    授予该角色的帐户的主机名部分。

+   `GRANTEE`

    授予该角色的帐户的用户名部分。

+   `GRANTEE_HOST`

    授予该角色的帐户的主机名部分。

+   `SPECIFIC_CATALOG`

    例程所属的目录名称。该值始终为 `def`。

+   `SPECIFIC_SCHEMA`

    例程所属的模式（数据库）的名称。

+   `SPECIFIC_NAME`

    例程的名称。

+   `ROUTINE_CATALOG`

    例程所属的目录名称。该值始终为 `def`。

+   `ROUTINE_SCHEMA`

    例程所属的模式（数据库）的名称。

+   `ROUTINE_NAME`

    例程的名称。

+   `PRIVILEGE_TYPE`

    授予的特权。该值可以是在例程级别授予的任何特权；请参阅第 15.7.1.6 节，“GRANT 语句”。每行列出一个特权，因此每个受让人持有的列特权都有一行。

+   `IS_GRANTABLE`

    `YES` 或 `NO`，取决于该角色是否可授予其他帐户。
