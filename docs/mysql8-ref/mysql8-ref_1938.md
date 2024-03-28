# 28.3.43 INFORMATION_SCHEMA TABLE_CONSTRAINTS_EXTENSIONS 表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/information-schema-table-constraints-extensions-table.html`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-table-constraints-extensions-table.html)

`TABLE_CONSTRAINTS_EXTENSIONS` 表（自 MySQL 8.0.21 起可用）提供有关为主要和辅助存储引擎定义的表约束属性的信息。

注意

`TABLE_CONSTRAINTS_EXTENSIONS` 表保留供将来使用。

`TABLE_CONSTRAINTS_EXTENSIONS` 表包含以下列：

+   `CONSTRAINT_CATALOG`

    表所属目录的名称。

+   `CONSTRAINT_SCHEMA`

    表所属模式（数据库）的名称。

+   `CONSTRAINT_NAME`

    约束的名称。

+   `TABLE_NAME`

    表的名称。

+   `ENGINE_ATTRIBUTE`

    为主要存储引擎定义的约束属性。保留供将来使用。

+   `SECONDARY_ENGINE_ATTRIBUTE`

    为辅助存储引擎定义的约束属性。保留供将来使用。
