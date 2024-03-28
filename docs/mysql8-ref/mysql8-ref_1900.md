# 28.3.5 INFORMATION_SCHEMA CHECK_CONSTRAINTS 表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/information-schema-check-constraints-table.html`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-check-constraints-table.html)

截至 MySQL 8.0.16，[`CREATE TABLE`](https://dev.mysql.com/doc/refman/8.0/en/create-table.html)允许表和列`CHECK`约束的核心特性，并且[`CHECK_CONSTRAINTS`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-check-constraints-table.html)表提供关于这些约束的信息。

[`CHECK_CONSTRAINTS`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-check-constraints-table.html)表具有以下列：

+   `CONSTRAINT_CATALOG`

    约束所属的目录的名称。此值始终为`def`。

+   `CONSTRAINT_SCHEMA`

    约束所属的模式（数据库）的名称。

+   `CONSTRAINT_NAME`

    约束的名称。

+   `CHECK_CLAUSE`

    指定约束条件的表达式。
