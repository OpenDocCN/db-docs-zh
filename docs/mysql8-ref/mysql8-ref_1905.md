# 28.3.10 INFORMATION_SCHEMA COLUMN_PRIVILEGES 表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/information-schema-column-privileges-table.html`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-column-privileges-table.html)

[`COLUMN_PRIVILEGES`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-column-privileges-table.html) 表提供有关列权限的信息。它从 `mysql.columns_priv` 系统表中获取其值。

[`COLUMN_PRIVILEGES`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-column-privileges-table.html) 表具有以下列：

+   `GRANTEE`

    授予权限的帐户的名称，格式为 `'*`user_name`*'@'*`host_name`*'`。

+   `TABLE_CATALOG`

    包含列的表所属的目录的名称。此值始终为 `def`。

+   `TABLE_SCHEMA`

    表中包含列的模式（数据库）的名称。

+   `TABLE_NAME`

    包含列的表的名称。

+   `COLUMN_NAME`

    列的名称。

+   `PRIVILEGE_TYPE`

    授予的权限。该值可以是可以在列级别授予的任何权限；请参见第 15.7.1.6 节，“GRANT Statement”。每行列出一个权限，因此每个受让人持有的列权限都有一行。

    在 `SHOW FULL COLUMNS` 的输出中，权限都在一个列中且为小写，例如，`select,insert,update,references`。在 [`COLUMN_PRIVILEGES`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-column-privileges-table.html) 中，每行一个权限，且为大写。

+   `IS_GRANTABLE`

    如果用户具有 `GRANT OPTION` 权限，则为 `YES`，否则为 `NO`。输出不会将 `GRANT OPTION` 列为具有 `PRIVILEGE_TYPE='GRANT OPTION'` 的单独行。

#### 注意

+   [`COLUMN_PRIVILEGES`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-column-privileges-table.html) 是一个非标准的 `INFORMATION_SCHEMA` 表。

以下语句*不*等价：

```sql
SELECT ... FROM INFORMATION_SCHEMA.COLUMN_PRIVILEGES

SHOW GRANTS ...
```
