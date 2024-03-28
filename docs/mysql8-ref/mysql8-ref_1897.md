# 28.3.2 INFORMATION_SCHEMA ADMINISTRABLE_ROLE_AUTHORIZATIONS 表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/information-schema-administrable-role-authorizations-table.html`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-administrable-role-authorizations-table.html)

[`ADMINISTRABLE_ROLE_AUTHORIZATIONS`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-administrable-role-authorizations-table.html) 表（自 MySQL 8.0.19 起可用）提供有关当前用户或角色适用的哪些角色可以授予给其他用户或角色的信息。

[`ADMINISTRABLE_ROLE_AUTHORIZATIONS`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-administrable-role-authorizations-table.html) 表具有以下列：

+   `USER`

    当前用户帐户的用户名称部分。

+   `HOST`

    当前用户帐户的主机名部分。

+   `GRANTEE`

    授予角色的帐户的用户名称部分。

+   `GRANTEE_HOST`

    授予角色的帐户的主机名部分。

+   `ROLE_NAME`

    授予角色的用户名称部分。

+   `ROLE_HOST`

    授予角色的主机名部分。

+   `IS_GRANTABLE`

    `YES` 或 `NO`，取决于角色是否可以授予给其他帐户。

+   `IS_DEFAULT`

    `YES` 或 `NO`，取决于角色是否是默认角色。

+   `IS_MANDATORY`

    `YES` 或 `NO`，取决于角色是否是强制角色。
