# 28.3.3 `INFORMATION_SCHEMA APPLICABLE_ROLES` 表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/information-schema-applicable-roles-table.html`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-applicable-roles-table.html)

`APPLICABLE_ROLES` 表（自 MySQL 8.0.19 起可用）提供了关于适用于当前用户的角色的信息。

`APPLICABLE_ROLES` 表具有以下列：

+   `USER`

    当前用户账户的用户名部分。

+   `HOST`

    当前用户账户的主机名部分。

+   `GRANTEE`

    被授予角色的账户的用户名部分。

+   `GRANTEE_HOST`

    被授予角色的账户的主机名部分。

+   `ROLE_NAME`

    被授予角色的用户名部分。

+   `ROLE_HOST`

    被授予角色的主机名部分。

+   `IS_GRANTABLE`

    `YES` 或 `NO`，取决于角色是否可授予给其他账户。

+   `IS_DEFAULT`

    `YES` 或 `NO`，取决于角色是否为默认角色。

+   `IS_MANDATORY`

    `YES` 或 `NO`，取决于角色是否为强制角色。
