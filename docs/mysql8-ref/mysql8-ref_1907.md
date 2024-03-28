# 28.3.12 The INFORMATION_SCHEMA ENABLED_ROLES Table

> 原文：[`dev.mysql.com/doc/refman/8.0/en/information-schema-enabled-roles-table.html`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-enabled-roles-table.html)

`ENABLED_ROLES` 表（自 MySQL 8.0.19 起可用）提供了关于当前会话中启用的角色的信息。

`ENABLED_ROLES` 表包含以下列：

+   `ROLE_NAME`

    被授予角色的用户名称部分。

+   `ROLE_HOST`

    被授予角色的主机名部分。

+   `IS_DEFAULT`

    `YES` 或 `NO`，取决于角色是否是默认角色。

+   `IS_MANDATORY`

    `YES` 或 `NO`，取决于角色是否是强制性的。
