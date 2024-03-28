# 28.3.50 信息模式 VIEW_TABLE_USAGE 表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/information-schema-view-table-usage-table.html`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-view-table-usage-table.html)

`VIEW_TABLE_USAGE` 表（自 MySQL 8.0.13 起可用）提供了关于视图定义中使用的表和视图的信息。

您只能查看您拥有某些权限的视图信息，以及您拥有某些权限的表信息。

`VIEW_TABLE_USAGE` 表包含以下列：

+   `VIEW_CATALOG`

    视图所属的目录的名称。该值始终为`def`。

+   `VIEW_SCHEMA`

    视图所属的模式（数据库）的名称。

+   `VIEW_NAME`

    视图的名称。

+   `TABLE_CATALOG`

    视图定义中使用的表或视图所属的目录名称。该值始终为`def`。

+   `TABLE_SCHEMA`

    视图定义中使用的表或视图所属的模式（数据库）的名称。

+   `TABLE_NAME`

    视图定义中使用的表或视图的名称。
