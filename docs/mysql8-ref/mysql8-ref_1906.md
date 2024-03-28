# 28.3.11 INFORMATION_SCHEMA COLUMN_STATISTICS 表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/information-schema-column-statistics-table.html`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-column-statistics-table.html)

`COLUMN_STATISTICS` 表提供对列值直方图统计信息的访问。

有关直方图统计信息，请参阅 Section 10.9.6, “优化器统计信息”和 Section 15.7.3.1, “ANALYZE TABLE 语句”。

你只能查看你拥有某些权限的列的信息。

`COLUMN_STATISTICS` 表包含以下列：

+   `SCHEMA_NAME`

    统计信息适用的模式的名称。

+   `TABLE_NAME`

    统计信息适用的列的名称。

+   `COLUMN_NAME`

    统计信息适用的列的名称。

+   `HISTOGRAM`

    一个描述列统计信息的`JSON`对象，存储为直方图。
