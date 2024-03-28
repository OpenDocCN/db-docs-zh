# 28.3.35 INFORMATION_SCHEMA ST_GEOMETRY_COLUMNS 表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/information-schema-st-geometry-columns-table.html`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-st-geometry-columns-table.html)

`ST_GEOMETRY_COLUMNS` 表提供关于存储空间数据的表列的信息。此表基于 SQL/MM（ISO/IEC 13249-3）标准，并带有如下扩展。MySQL 将`ST_GEOMETRY_COLUMNS` 实现为`INFORMATION_SCHEMA` `COLUMNS` 表上的视图。

`ST_GEOMETRY_COLUMNS` 表包含以下列：

+   `TABLE_CATALOG`

    包含列的表所属的目录的名称。此值始终为`def`。

+   `TABLE_SCHEMA`

    包含列的表所属的模式（数据库）的名称。

+   `TABLE_NAME`

    包含列的表的名称。

+   `COLUMN_NAME`

    列的名称。

+   `SRS_NAME`

    空间参考系统（SRS）名称。

+   `SRS_ID`

    空间参考系统 ID（SRID）。

+   `GEOMETRY_TYPE_NAME`

    列数据类型。允许的值为：`geometry`、`point`、`linestring`、`polygon`、`multipoint`、`multilinestring`、`multipolygon`、`geometrycollection`。此列是 MySQL 对标准的扩展。
