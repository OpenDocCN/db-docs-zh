# 28.3.49 The INFORMATION_SCHEMA VIEW_ROUTINE_USAGE Table

> 原文：[`dev.mysql.com/doc/refman/8.0/en/information-schema-view-routine-usage-table.html`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-view-routine-usage-table.html)

`VIEW_ROUTINE_USAGE` 表（自 MySQL 8.0.13 起可用）提供有关视图定义中使用的存储函数的信息。该表不列出有关内置（本机）函数或在定义中使用的可加载函数的信息。

您只能查看您拥有某些权限的视图信息，以及您拥有某些权限的函数信息。

`VIEW_ROUTINE_USAGE` 表具有以下列：

+   `TABLE_CATALOG`

    视图所属的目录的名称。此值始终为`def`。

+   `TABLE_SCHEMA`

    视图所属的模式（数据库）的名称。

+   `TABLE_NAME`

    视图的名称。

+   `SPECIFIC_CATALOG`

    视图定义中使用的函数所属的目录名称。此值始终为`def`。

+   `SPECIFIC_SCHEMA`

    视图定义中使用的函数所属的模式（数据库）的名称。

+   `SPECIFIC_NAME`

    视图定义中使用的函数的名称。
