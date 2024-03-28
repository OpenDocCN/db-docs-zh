# 28.3.9 INFORMATION_SCHEMA COLUMNS_EXTENSIONS 表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/information-schema-columns-extensions-table.html`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-columns-extensions-table.html)

`COLUMNS_EXTENSIONS` 表（自 MySQL 8.0.21 起可用）提供有关为主存储引擎和次要存储引擎定义的列属性的信息。

注意

`COLUMNS_EXTENSIONS` 表保留供将来使用。

`COLUMNS_EXTENSIONS` 表包含以下列：

+   `TABLE_CATALOG`

    表所属目录的名称。该值始终为 `def`。

+   `TABLE_SCHEMA`

    表所属模式（数据库）的名称。

+   `TABLE_NAME`

    表的名称。

+   `COLUMN_NAME`

    列的名称。

+   `ENGINE_ATTRIBUTE`

    为主存储引擎定义的列属性。保留供将来使用。

+   `SECONDARY_ENGINE_ATTRIBUTE`

    为次要存储引擎定义的列属性。保留供将来使用。
