# 28.3.47 INFORMATION_SCHEMA USER_PRIVILEGES 表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/information-schema-user-privileges-table.html`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-user-privileges-table.html)

`USER_PRIVILEGES` 表提供有关全局权限的信息。它的值来自于 `mysql.user` 系统表。

`USER_PRIVILEGES` 表包含以下列：

+   `GRANTEE`

    被授予权限的帐户名称，格式为 `'*`user_name`*'@'*`host_name`*'`。

+   `TABLE_CATALOG`

    目录的名称。该值始终为 `def`。

+   `PRIVILEGE_TYPE`

    授予的权限。该值可以是在全局级别授予的任何权限；请参阅 Section 15.7.1.6, “GRANT Statement”。每行列出一个权限，因此每个被授予权限的受让人都有一行。

+   `IS_GRANTABLE`

    如果用户具有 `GRANT OPTION` 权限，则为 `YES`，否则为 `NO`。输出不会将 `GRANT OPTION` 列为具有 `PRIVILEGE_TYPE='GRANT OPTION'` 的单独行。

#### 注意

+   `USER_PRIVILEGES` 是一个非标准的 `INFORMATION_SCHEMA` 表。

以下语句*不*等价：

```sql
SELECT ... FROM INFORMATION_SCHEMA.USER_PRIVILEGES

SHOW GRANTS ...
```
