# 28.3.39 **INFORMATION_SCHEMA TABLES_EXTENSIONS** 表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/information-schema-tables-extensions-table.html`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-tables-extensions-table.html)

`TABLES_EXTENSIONS` 表（自 MySQL 8.0.21 起可用）提供有关为主要和次要存储引擎定义的表属性的信息。

注意

`TABLES_EXTENSIONS` 表保留供将来使用。

`TABLES_EXTENSIONS` 表具有以下列：

+   `TABLE_CATALOG`

    表所属目录的名称。此值始终为`def`。

+   `TABLE_SCHEMA`

    表所属模式（数据库）的名称。

+   `TABLE_NAME`

    表的名称。

+   `ENGINE_ATTRIBUTE`

    为主要存储引擎定义的表属性。保留供将来使用。

+   `SECONDARY_ENGINE_ATTRIBUTE`

    为次要存储引擎定义的表属性。保留供将来使用。
