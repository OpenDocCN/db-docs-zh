# 28.3.33 The INFORMATION_SCHEMA SCHEMA_PRIVILEGES Table

> 原文：[`dev.mysql.com/doc/refman/8.0/en/information-schema-schema-privileges-table.html`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-schema-privileges-table.html)

`SCHEMA_PRIVILEGES`表提供有关模式（数据库）权限的信息。它从`mysql.db`系统表中获取其值。

`SCHEMA_PRIVILEGES`表具有以下列：

+   `GRANTEE`

    授予权限的帐户名称，格式为`'*`user_name`*'@'*`host_name`*'`。

+   `TABLE_CATALOG`

    模式所属的目录名称。该值始终为`def`。

+   `TABLE_SCHEMA`

    模式的名称。

+   `PRIVILEGE_TYPE`

    授予的权限。该值可以是在模式级别授予的任何权限；请参阅第 15.7.1.6 节，“GRANT 语句”。每行列出一个权限，因此每个受让人持有的模式权限都有一行。

+   `IS_GRANTABLE`

    如果用户拥有`GRANT OPTION`权限，则为`YES`，否则为`NO`。输出不会将`GRANT OPTION`列为`PRIVILEGE_TYPE='GRANT OPTION'`的单独行。

#### 注意

+   `SCHEMA_PRIVILEGES`是一个非标准的`INFORMATION_SCHEMA`表。

以下语句*不*等价：

```sql
SELECT ... FROM INFORMATION_SCHEMA.SCHEMA_PRIVILEGES

SHOW GRANTS ...
```
