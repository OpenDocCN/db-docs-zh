# 28.3.1 INFORMATION_SCHEMA 通用表参考

> 原文：[`dev.mysql.com/doc/refman/8.0/en/information-schema-general-table-reference.html`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-general-table-reference.html)

以下表格总结了`INFORMATION_SCHEMA`通用表。更详细的信息，请参阅各个表的描述。

**表 28.2 INFORMATION_SCHEMA 通用表**

| 表名 | 描述 | 引入版本 | 废弃版本 |
| --- | --- | --- | --- |
| `ADMINISTRABLE_ROLE_AUTHORIZATIONS` | 当前用户或角色可授权的用户或角色 | 8.0.19 |  |
| `APPLICABLE_ROLES` | 当前用户适用的角色 | 8.0.19 |  |
| `CHARACTER_SETS` | 可用字符集 |  |  |
| `CHECK_CONSTRAINTS` | 表和列的检查约束 | 8.0.16 |  |
| `COLLATION_CHARACTER_SET_APPLICABILITY` | 每个排序规则适用的字符集 |  |  |
| `COLLATIONS` | 每个字符集的排序规则 |  |  |
| `COLUMN_PRIVILEGES` | 列上定义的权限 |  |  |
| `COLUMN_STATISTICS` | 列值的直方图统计 |  |  |
| `COLUMNS` | 每个表中的列 |  |  |
| `COLUMNS_EXTENSIONS` | 主要和次要存储引擎的列属性 | 8.0.21 |  |
| `ENABLED_ROLES` | 当前会话中启用的角色 | 8.0.19 |  |
| `ENGINES` | 存储引擎属性 |  |  |
| `EVENTS` | 事件管理器事件 |  |  |
| `FILES` | 存储表空间数据的文件 |  |  |
| `KEY_COLUMN_USAGE` | 具有约束的关键列 |  |  |
| `KEYWORDS` | MySQL 关键词 |  |  |
| `ndb_transid_mysql_connection_map` | NDB 事务信息 |  |  |
| `OPTIMIZER_TRACE` | 优化器跟踪活动产生的信息 |  |  |
| `PARAMETERS` | 存储过程参数和存储函数返回值 |  |  |
| `PARTITIONS` | 表分区信息 |  |  |
| `PLUGINS` | 插件信息 |  |  |
| `PROCESSLIST` | 当前执行线程的信息 |  |  |
| `PROFILING` | 语句分析信息 |  |  |
| `REFERENTIAL_CONSTRAINTS` | 外键信息 |  |  |
| `RESOURCE_GROUPS` | 资源组信息 |  |  |
| `ROLE_COLUMN_GRANTS` | 当前启用角色可用或授予的列权限 | 8.0.19 |  |
| `ROLE_ROUTINE_GRANTS` | 当前启用角色可用或授予的例程权限 | 8.0.19 |  |
| `ROLE_TABLE_GRANTS` | 当前启用角色可用或授予的表权限 | 8.0.19 |  |
| `ROUTINES` | 存储过程信息 |  |  |
| `SCHEMA_PRIVILEGES` | 在模式上定义的权限 |  |  |
| `SCHEMATA` | 模式信息 |  |  |
| `SCHEMATA_EXTENSIONS` | 模式选项 | 8.0.22 |  |
| `ST_GEOMETRY_COLUMNS` | 每个存储空间数据的表中的列 |  |  |
| `ST_SPATIAL_REFERENCE_SYSTEMS` | 可用的空间参考系统 |  |  |
| `ST_UNITS_OF_MEASURE` | ST_Distance()可接受的单位 | 8.0.14 |  |
| `STATISTICS` | 表索引统计 |  |  |
| `TABLE_CONSTRAINTS` | 哪些表具有约束 |  |  |
| `TABLE_CONSTRAINTS_EXTENSIONS` | 主要和次要存储引擎的表约束属性 | 8.0.21 |  |
| `TABLE_PRIVILEGES` | 表上定义的权限 |  |  |
| `TABLES` | 表信息 |  |  |
| `TABLES_EXTENSIONS` | 主要和次要存储引擎的表属性 | 8.0.21 |  |
| `TABLESPACES` | 表空间信息 |  | 8.0.22 |
| `TABLESPACES_EXTENSIONS` | 主要存储引擎的表空间属性 | 8.0.21 |  |
| `TRIGGERS` | 触发器信息 |  |  |
| `USER_ATTRIBUTES` | 用户评论和属性 | 8.0.21 |  |
| `USER_PRIVILEGES` | 每个用户全局定义的权限 |  |  |
| `VIEW_ROUTINE_USAGE` | 在视图中使用的存储函数 | 8.0.13 |  |
| `VIEW_TABLE_USAGE` | 在视图中使用的表和视图 | 8.0.13 |  |
| `视图` | 视图信息 |  |  |
| 表名 | 描述 | 引入 | 废弃 |
