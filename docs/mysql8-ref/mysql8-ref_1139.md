# 16.5 INFORMATION_SCHEMA 和数据字典集成

> 原文：[`dev.mysql.com/doc/refman/8.0/en/data-dictionary-information-schema.html`](https://dev.mysql.com/doc/refman/8.0/en/data-dictionary-information-schema.html)

引入数据字典后，以下 `INFORMATION_SCHEMA` 表被实现为数据字典表上的视图：

+   `CHARACTER_SETS`

+   `CHECK_CONSTRAINTS`

+   `COLLATIONS`

+   `COLLATION_CHARACTER_SET_APPLICABILITY`

+   `COLUMNS`

+   `COLUMN_STATISTICS`

+   `EVENTS`

+   `FILES`

+   `INNODB_COLUMNS`

+   `INNODB_DATAFILES`

+   `INNODB_FIELDS`

+   `INNODB_FOREIGN`

+   `INNODB_FOREIGN_COLS`

+   `INNODB_INDEXES`

+   `INNODB_TABLES`

+   `INNODB_TABLESPACES`

+   `INNODB_TABLESPACES_BRIEF`

+   `INNODB_TABLESTATS`

+   `KEY_COLUMN_USAGE`

+   `KEYWORDS`

+   `PARAMETERS`

+   `PARTITIONS`

+   `REFERENTIAL_CONSTRAINTS`

+   `RESOURCE_GROUPS`

+   `ROUTINES`

+   `SCHEMATA`

+   `STATISTICS`

+   `ST_GEOMETRY_COLUMNS`

+   `ST_SPATIAL_REFERENCE_SYSTEMS`

+   `TABLES`

+   `TABLE_CONSTRAINTS`

+   `TRIGGERS`

+   `VIEWS`

+   `VIEW_ROUTINE_USAGE`

+   `VIEW_TABLE_USAGE`

现在对这些表的查询更加高效，因为它们从数据字典表中获取信息，而不是通过其他更慢的方式。特别是，对于每个是数据字典表上视图的 `INFORMATION_SCHEMA` 表：

+   服务器不再必须为 `INFORMATION_SCHEMA` 表的每个查询创建临时表。

+   当底层数据字典表存储先前通过目录扫描（例如，枚举数据库名称或数据库中表名称）或文件打开操作（例如，从`.frm`文件中读取信息）获取的值时，`INFORMATION_SCHEMA` 现在使用表查找而不是查询这些值。（此外，即使对于非视图的 `INFORMATION_SCHEMA` 表，例如数据库和表名称等值也是通过数据字典查找而不需要目录或文件扫描获取的。）

+   底层数据字典表上的索引允许优化器构建高效的查询执行计划，这在以前的实现中是不成立的，以前的实现是使用临时表处理 `INFORMATION_SCHEMA` 表的每个查询。

前述的改进也适用于显示与数据字典表上的视图对应的信息的`SHOW`语句。例如，`SHOW DATABASES`显示与`SCHEMATA`表相同的信息。

除了引入对数据字典表的视图之外，现在`STATISTICS`和`TABLES`表中包含的表统计信息现在被缓存以提高`INFORMATION_SCHEMA`查询性能。`information_schema_stats_expiry`系统变量定义了缓存表统计信息过期之前的时间段。默认值为 86400 秒（24 小时）。如果没有缓存的统计信息或统计信息已过期，则在查询表统计列时从存储引擎中检索统计信息。要随时更新给定表的缓存值，请使用`ANALYZE TABLE`。

`information_schema_stats_expiry`可以设置为`0`，以便`INFORMATION_SCHEMA`查询直接从存储引擎中检索最新的统计信息，这不如检索缓存的统计信息快。

更多信息，请参见 Section 10.2.3, “Optimizing INFORMATION_SCHEMA Queries”。

MySQL 8.0 中的`INFORMATION_SCHEMA`表与数据字典紧密相关，导致几个使用差异。请参见 Section 16.7, “Data Dictionary Usage Differences”。
