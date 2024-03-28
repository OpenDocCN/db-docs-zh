# 28.3.42 信息模式表约束表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/information-schema-table-constraints-table.html`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-table-constraints-table.html)

[`TABLE_CONSTRAINTS`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-table-constraints-table.html)表描述了哪些表具有约束。

[`TABLE_CONSTRAINTS`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-table-constraints-table.html)表具有以下列：

+   `CONSTRAINT_CATALOG`

    约束所属的目录的名称。此值始终为`def`。

+   `CONSTRAINT_SCHEMA`

    约束所属的模式（数据库）的名称。

+   `CONSTRAINT_NAME`

    约束的名称。

+   `TABLE_SCHEMA`

    表所属的模式（数据库）的名称。

+   `TABLE_NAME`

    表的名称。

+   `CONSTRAINT_TYPE`

    约束的类型。该值可以是`UNIQUE`、`PRIMARY KEY`、`FOREIGN KEY`，或者（从 MySQL 8.0.16 开始）`CHECK`。这是一个`CHAR`（而不是`ENUM`输出的`Key_name`列中获得的信息大致相同，当`Non_unique`列为`0`时。

+   `ENFORCED`

    对于`CHECK`约束，该值为`YES`或`NO`，表示约束是否被强制执行。对于其他约束，该值始终为`YES`。

    此列在 MySQL 8.0.16 中添加。
