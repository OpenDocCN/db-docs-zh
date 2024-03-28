# 28.3.25 INFORMATION_SCHEMA REFERENTIAL_CONSTRAINTS 表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/information-schema-referential-constraints-table.html`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-referential-constraints-table.html)

`REFERENTIAL_CONSTRAINTS`表提供有关外键的信息。

`REFERENTIAL_CONSTRAINTS`表具有以下列：

+   `CONSTRAINT_CATALOG`

    约束所属的目录的名称。此值始终为`def`。

+   `CONSTRAINT_SCHEMA`

    约束所属的模式（数据库）的名称。

+   `CONSTRAINT_NAME`

    约束的名称。

+   `UNIQUE_CONSTRAINT_CATALOG`

    包含约束引用的唯一约束的目录的名称。此值始终为`def`。

+   `UNIQUE_CONSTRAINT_SCHEMA`

    包含约束引用的唯一约束的模式的名称。

+   `UNIQUE_CONSTRAINT_NAME`

    约束引用的唯一约束的名称。

+   `MATCH_OPTION`

    约束`MATCH`属性的值。目前唯一有效的值是`NONE`。

+   `UPDATE_RULE`

    约束`ON UPDATE`属性的值。可能的值为`CASCADE`、`SET NULL`、`SET DEFAULT`、`RESTRICT`、`NO ACTION`。

+   `DELETE_RULE`

    约束`ON DELETE`属性的值。可能的值为`CASCADE`、`SET NULL`、`SET DEFAULT`、`RESTRICT`、`NO ACTION`。

+   `TABLE_NAME`

    表的名称。此值与`TABLE_CONSTRAINTS`表中的相同。

+   `REFERENCED_TABLE_NAME`

    约束引用的表的名称。
