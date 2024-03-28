# 28.2 INFORMATION_SCHEMA 表参考

> 原文：[`dev.mysql.com/doc/refman/8.0/en/information-schema-table-reference.html`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-table-reference.html)

以下表总结了所有可用的`INFORMATION_SCHEMA`表。更详细的信息，请参阅各个表的描述。

**表 28.1 INFORMATION_SCHEMA 表**

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
| `CONNECTION_CONTROL_FAILED_LOGIN_ATTEMPTS` | 每个账户连续失败连接尝试的当前次数 |  |  |
| `ENABLED_ROLES` | 当前会话中启用的角色 | 8.0.19 |  |
| `ENGINES` | 存储引擎属性 |  |  |
| `EVENTS` | 事件管理器事件 |  |  |
| `FILES` | 存储表空间数据的文件 |  |  |
| `INNODB_BUFFER_PAGE` | InnoDB 缓冲池中的页面 |  |  |
| `INNODB_BUFFER_PAGE_LRU` | InnoDB 缓冲池中页面的 LRU 排序 |  |  |
| `INNODB_BUFFER_POOL_STATS` | InnoDB 缓冲池统计信息 |  |  |
| `INNODB_CACHED_INDEXES` | InnoDB 缓冲池中每个索引缓存的索引页数 |  |  |
| `INNODB_CMP` | 与压缩的 InnoDB 表相关的操作状态 |  |  |
| `INNODB_CMP_PER_INDEX` | 与压缩的 InnoDB 表和索引相关的操作状态 |  |  |
| `INNODB_CMP_PER_INDEX_RESET` | 与压缩的 InnoDB 表和索引相关的操作状态 |  |  |
| `INNODB_CMP_RESET` | 与压缩的 InnoDB 表相关的操作状态 |  |  |
| `INNODB_CMPMEM` | InnoDB 缓冲池中压缩页面的状态 |  |  |
| `INNODB_CMPMEM_RESET` | InnoDB 缓冲池中压缩页面的状态 |  |  |
| `INNODB_COLUMNS` | 每个 InnoDB 表中的列 |  |  |
| `INNODB_DATAFILES` | InnoDB 每表一个文件和通用表空间的数据文件路径信息 |  |  |
| `INNODB_FIELDS` | InnoDB 索引的关键列 |  |  |
| `INNODB_FOREIGN` | InnoDB 外键元数据 |  |  |
| `INNODB_FOREIGN_COLS` | InnoDB 外键列状态信息 |  |  |
| `INNODB_FT_BEING_DELETED` | INNODB_FT_DELETED 表的快照 |  |  |
| `INNODB_FT_CONFIG` | InnoDB 表全文索引和相关处理的元数据 |  |  |
| `INNODB_FT_DEFAULT_STOPWORD` | InnoDB 全文索引的默认停用词列表 |  |  |
| `INNODB_FT_DELETED` | 从 InnoDB 表全文索引中删除的行 |  |  |
| `INNODB_FT_INDEX_CACHE` | InnoDB 全文索引中新插入行的标记信息 |  |  |
| `INNODB_FT_INDEX_TABLE` | 用于处理针对 InnoDB 表全文索引的文本搜索的倒排索引信息 |  |  |
| `INNODB_INDEXES` | InnoDB 索引元数据 |  |  |
| `INNODB_METRICS` | InnoDB 性能信息 |  |  |
| `INNODB_SESSION_TEMP_TABLESPACES` | 会话临时表空间元数据 | 8.0.13 |  |
| `INNODB_TABLES` | InnoDB 表元数据 |  |  |
| `INNODB_TABLESPACES` | InnoDB 每表文件、通用和撤销表空间元数据 |  |  |
| `INNODB_TABLESPACES_BRIEF` | 简要的文件、通用、撤销和系统表空间元��据 |  |  |
| `INNODB_TABLESTATS` | InnoDB 表低级状态信息 |  |  |
| `INNODB_TEMP_TABLE_INFO` | 关于活动用户创建的 InnoDB 临时表的信息 |  |  |
| `INNODB_TRX` | 活动的 InnoDB 事务信息 |  |  |
| `INNODB_VIRTUAL` | InnoDB 虚拟生成列元数据 |  |  |
| `KEY_COLUMN_USAGE` | 具有约束的关键列 |  |  |
| `KEYWORDS` | MySQL 关键字 |  |  |
| `MYSQL_FIREWALL_USERS` | 账户配置的防火墙内存数据 |  | 8.0.26 |
| `MYSQL_FIREWALL_WHITELIST` | 账户配置的防火墙内存数据允许列表 |  | 8.0.26 |
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
| `ST_GEOMETRY_COLUMNS` | 每个表中存储空间数据的列 |  |  |
| `ST_SPATIAL_REFERENCE_SYSTEMS` | 可用的空间参考系统 |  |  |
| `ST_UNITS_OF_MEASURE` | ST_Distance()可接受的单位 | 8.0.14 |  |
| `STATISTICS` | 表索引统计信息 |  |  |
| `TABLE_CONSTRAINTS` | 哪些表具有约束 |  |  |
| `TABLE_CONSTRAINTS_EXTENSIONS` | 主要和次要存储引擎的表约束属性 | 8.0.21 |  |
| `TABLE_PRIVILEGES` | 在表上定义的权限 |  |  |
| `TABLES` | 表信息 |  |  |
| `TABLES_EXTENSIONS` | 主要和次要存储引擎的表属性 | 8.0.21 |  |
| `TABLESPACES` | 表空间信息 |  | 8.0.22 |
| `TABLESPACES_EXTENSIONS` | 主存储引擎表空间属性 | 8.0.21 |  |
| `TP_THREAD_GROUP_STATE` | 线程池线程组状态 |  |  |
| `TP_THREAD_GROUP_STATS` | 线程池线程组统计信息 |  |  |
| `TP_THREAD_STATE` | 线程池线程信息 |  |  |
| `TRIGGERS` | 触发器信息 |  |  |
| `USER_ATTRIBUTES` | 用户注释和属性 | 8.0.21 |  |
| `USER_PRIVILEGES` | 每个用户全局定义的权限 |  |  |
| `VIEW_ROUTINE_USAGE` | 在视图中使用的存储函数 | 8.0.13 |  |
| `VIEW_TABLE_USAGE` | 在视图中使用的表和视图 | 8.0.13 |  |
| `VIEWS` | 视图信息 |  |  |
| 表名 | 描述 | 引入版本 | 废弃版本 |
