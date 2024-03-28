# 28.3.31 The INFORMATION_SCHEMA SCHEMATA Table

> 原文：[`dev.mysql.com/doc/refman/8.0/en/information-schema-schemata-table.html`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-schemata-table.html)

一个模式是一个数据库，因此`SCHEMATA`表提供关于数据库的信息。

`SCHEMATA`表具有以下列：

+   `CATALOG_NAME`

    模式所属目录的名称。此值始终为`def`。

+   `SCHEMA_NAME`

    模式的名称。

+   `DEFAULT_CHARACTER_SET_NAME`

    模式默认字符集。

+   `DEFAULT_COLLATION_NAME`

    模式默认排序规则。

+   `SQL_PATH`

    此值始终为`NULL`。

+   `DEFAULT_ENCRYPTION`

    模式默认加密。此列在 MySQL 8.0.16 中添加。

模式名称也可以从`SHOW DATABASES`语句中获取。参见 Section 15.7.7.14, “SHOW DATABASES Statement”。以下语句是等效的：

```sql
SELECT SCHEMA_NAME AS `Database`
  FROM INFORMATION_SCHEMA.SCHEMATA
  [WHERE SCHEMA_NAME LIKE '*wild*']

SHOW DATABASES
  [LIKE '*wild*']
```

除非具有全局`SHOW DATABASES`权限，否则只能看到具有某种特权的数据库。

注意

因为任何静态全局特权都被视为对所有数据库的特权，任何静态全局特权都使用户能够使用`SHOW DATABASES`或通过检查`INFORMATION_SCHEMA`的`SCHEMATA`表来查看所有数据库名称，除了在数据库级别通过部分撤销限制的数据库。

#### 注意

+   `SCHEMATA_EXTENSIONS`表通过提供有关模式选项的信息来扩展`SCHEMATA`表。
