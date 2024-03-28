# 28.3.44 The INFORMATION_SCHEMA TABLE_PRIVILEGES Table

> 原文：[`dev.mysql.com/doc/refman/8.0/en/information-schema-table-privileges-table.html`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-table-privileges-table.html)

`TABLE_PRIVILEGES`表提供有关表特权的信息。它从`mysql.tables_priv`系统表中获取其值。

`TABLE_PRIVILEGES`表具有以下列：

+   `GRANTEE`

    授予特权的帐户名称，格式为`'*`user_name`*'@'*`host_name`*'`。

+   `TABLE_CATALOG`

    表所属目录的名称。该值始终为`def`。

+   `TABLE_SCHEMA`

    表所属模式（数据库）的名称。

+   `TABLE_NAME`

    表的名称。

+   `PRIVILEGE_TYPE`

    授予的特权。该值可以是可以在表级别授予的任何特权；参见第 15.7.1.6 节，“GRANT 语句”。每行列出一个特权，因此每个受让人持有的表特权都有一行。

+   `IS_GRANTABLE`

    如果用户具有`GRANT OPTION`特权，则为`YES`，否则为`NO`。输出不会将`GRANT OPTION`列为具有`PRIVILEGE_TYPE='GRANT OPTION'`的单独行。

#### 注意事项

+   `TABLE_PRIVILEGES`是一个非标准的`INFORMATION_SCHEMA`表。

以下语句*不*等价：

```sql
SELECT ... FROM INFORMATION_SCHEMA.TABLE_PRIVILEGES

SHOW GRANTS ...
```
