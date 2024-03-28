# 7.3 mysql 系统模式

> 原文：[`dev.mysql.com/doc/refman/8.0/en/system-schema.html`](https://dev.mysql.com/doc/refman/8.0/en/system-schema.html)

`mysql`模式是系统模式。它包含存储 MySQL 服务器运行所需信息的表。一个广泛的分类是，`mysql`模式包含存储数据库对象元数据的数据字典表，以及用于其他操作目的的系统表。以下讨论将系统表集进一步细分为更小的类别。

+   数据字典表

+   授权系统表

+   对象信息系统表

+   日志系统表

+   服务器端帮助系统表

+   时区系统表

+   复制系统表

+   优化器系统表

+   杂项系统表

本节的其余部分列举了每个类别中的表，附带了额外信息的交叉引用。数据字典表和系统表使用`InnoDB`存储引擎，除非另有说明。

`mysql`系统表和数据字典表存储在 MySQL 数据目录中名为`mysql.ibd`的单个`InnoDB`表空间文件中。以前，这些表是在`mysql`数据库目录中的单独表空间文件中创建的。

可以为`mysql`系统模式表空间启用数据静态加密。有关更多信息，请参见第 17.13 节，“InnoDB 数据静态加密”。

### 数据字典表

这些表包括数据字典，其中包含有关数据库对象的元数据。有关更多信息，请参见第十六章，*MySQL 数据字典*。

重要

数据字典是 MySQL 8.0 中的新功能。启用数据字典的服务器与之前的 MySQL 版本相比存在一些一般操作上的差异。有关详细信息，请参阅第 16.7 节，“数据字典使用差异”。此外，从 MySQL 5.7 升级到 MySQL 8.0，升级过程与之前的 MySQL 版本有所不同，并且需要您通过检查特定先决条件来验证安装的升级准备情况。有关更多信息，请参阅第三章，“升级 MySQL”，特别是第 3.6 节，“准备安装以进行升级”。

+   `catalogs`: 目录信息。

+   `character_sets`: 可用字符集的信息。

+   `check_constraints`: 在表上定义的`CHECK`约束的信息。请参阅第 15.1.20.6 节，“CHECK 约束”。

+   `collations`: 每个字符集的排序规则信息。

+   `column_statistics`: 列值的直方图统计信息。请参阅第 10.9.6 节，“优化器统计信息”。

+   `column_type_elements`: 列使用的类型信息。

+   `columns`: 表中列的信息。

+   `dd_properties`: 用于标识数据字典属性的表，例如其版本。服务器使用此信息来确定数据字典是否必须升级到更新版本。

+   `events`: 事件调度器事件的信息。请参阅第 27.4 节，“使用事件调度器”。如果服务器使用`--skip-grant-tables`选项启动，则事件调度器将被禁用，并且在表中注册的事件不会运行。请参阅第 27.4.2 节，“事件调度器配置”。

+   `foreign_keys`, `foreign_key_column_usage`: 外键信息。

+   `index_column_usage`: 索引使用的列的信息。

+   `index_partitions`: 索引使用的分区信息。

+   `index_stats`: 用于存储执行`ANALYZE TABLE`时生成的动态索引统计信息。

+   `indexes`: 表索引的信息。

+   `innodb_ddl_log`: 存储崩溃安全的 DDL 操作的 DDL 日志。

+   `parameter_type_elements`: 存储过程和函数参数以及存储函数返回值的信息。

+   `parameters`: 存储过程和函数的信息。请参阅第 27.2 节，“使用存储过程”。

+   `resource_groups`: 资源组的信息。请参阅第 7.1.16 节，“资源组”。

+   `routines`: 关于存储过程和函数的信息。参见 Section 27.2, “Using Stored Routines”。

+   `schemata`: 关于模式的信息。在 MySQL 中，模式是数据库，因此该表提供有关数据库的信息。

+   `st_spatial_reference_systems`: 空间数据可用的空间参考系统信息。

+   `table_partition_values`: 关于表分区使用的数值信息。

+   `table_partitions`: 表使用的分区信息。

+   `table_stats`: 当执行`ANALYZE TABLE`时生成的动态表统计信息。

+   `tables`: 数据库中表的信息。

+   `tablespace_files`: 表空间使用的文件信息。

+   `tablespaces`: 活动表空间的信息。

+   `triggers`: 触发器的信息。

+   `view_routine_usage`: 视图和使用它们的存储函数之间的依赖关系信息。

+   `view_table_usage`: 用于跟踪视图和其基础表之间的依赖关系。

数据字典表是不可见的。它们不能通过`SELECT`读取，不会出现在`SHOW TABLES`的输出中，不会在`INFORMATION_SCHEMA.TABLES`表中列出等等。然而，在大多数情况下，可以查询相应的`INFORMATION_SCHEMA`表。从概念上讲，`INFORMATION_SCHEMA`提供了 MySQL 公开数据字典元数据的视图。例如，您不能直接从`mysql.schemata`表中选择：

```sql
mysql> SELECT * FROM mysql.schemata;
ERROR 3554 (HY000): Access to data dictionary table 'mysql.schemata' is rejected.
```

相反，从相应的`INFORMATION_SCHEMA`表中选择该信息：

```sql
mysql> SELECT * FROM INFORMATION_SCHEMA.SCHEMATA\G
*************************** 1\. row ***************************
              CATALOG_NAME: def
               SCHEMA_NAME: mysql
DEFAULT_CHARACTER_SET_NAME: utf8mb4
    DEFAULT_COLLATION_NAME: utf8mb4_0900_ai_ci
                  SQL_PATH: NULL
        DEFAULT_ENCRYPTION: NO
*************************** 2\. row ***************************
              CATALOG_NAME: def
               SCHEMA_NAME: information_schema
DEFAULT_CHARACTER_SET_NAME: utf8mb3
    DEFAULT_COLLATION_NAME: utf8mb3_general_ci
                  SQL_PATH: NULL
        DEFAULT_ENCRYPTION: NO
*************************** 3\. row ***************************
              CATALOG_NAME: def
               SCHEMA_NAME: performance_schema
DEFAULT_CHARACTER_SET_NAME: utf8mb4
    DEFAULT_COLLATION_NAME: utf8mb4_0900_ai_ci
                  SQL_PATH: NULL
        DEFAULT_ENCRYPTION: NO
...
```

没有与`mysql.indexes`完全对应的信息模式表，但`INFORMATION_SCHEMA.STATISTICS`包含了大部分相同的信息。

到目前为止，还没有与`mysql.foreign_keys`、`mysql.foreign_key_column_usage`完全对应的`INFORMATION_SCHEMA`表。获取外键信息的标准 SQL 方法是使用`INFORMATION_SCHEMA`的`REFERENTIAL_CONSTRAINTS`和`KEY_COLUMN_USAGE`表；这些表现在作为`foreign_keys`、`foreign_key_column_usage`和其他数据字典表的视图实现。

MySQL 8.0 之前的一些系统表已被数据字典表取代，不再存在于`mysql`系统模式中：

+   `events`数据字典表取代了 MySQL 8.0 之前的`event`表。

+   `parameters`和`routines`数据字典表共同取代了 MySQL 8.0 之前的`proc`表。

### 授权系统表

这些系统表包含有关用户帐户和其持有的权限的授权信息。有关这些表的结构、内容和目的的其他信息，请参见第 8.2.3 节，“授权表”。

从 MySQL 8.0 开始，授权表是`InnoDB`（事务性）表。以前，这些是`MyISAM`（非事务性）表。授权表存储引擎的更改伴随着 MySQL 8.0 中帐户管理语句行为的变化，例如`CREATE USER`和`GRANT`。以前，命名多个用户的帐户管理语句可能对某些用户成功，对其他用户失败。这些语句现在是事务性的，如果发生任何错误，则对所有命名用户成功或回滚并且不产生任何效果。

注意

如果 MySQL 从旧版本升级，但授权表未从`MyISAM`升级到`InnoDB`，服务器会将其视为只读，并且帐户管理语句会产生错误。有关升级说明，请参见第三章，*升级 MySQL*。

+   `user`: 用户账户、全局权限和其他非权限列。

+   `global_grants`: 将动态全局权限分配给用户；请参见静态权限与动态权限。

+   `db`: 数据库级别的权限。

+   `tables_priv`: 表级别的权限。

+   `columns_priv`: 列级别的权限。

+   `procs_priv`: 存储过程和函数权限。

+   `proxies_priv`: 代理用户权限。

+   `default_roles`: 此表列出了用户连接和认证后要激活的默认角色，或执行`SET ROLE DEFAULT`。

+   `role_edges`: 此表列出了角色子图的边缘。

    给定的`user`表行可能指向用户帐户或角色。服务器可以通过查询`role_edges`表来区分行是表示用户帐户、角色还是两者之间的关系的信息。

+   `password_history`: 关于密码更改的信息。

### 对象信息系统表

这些系统表包含有关组件、可加载函数和服务器端插件的信息：

+   `component`: 使用`INSTALL COMPONENT`安装的服务器组件的注册表。在此表中列出的任何组件都将由加载程序在服务器启动序列期间安装。请参见第 7.5.1 节，“安装和卸载组件”。

+   `func`: 通过`CREATE FUNCTION`安装的可加载函数的注册表。在正常启动序列期间，服务器会加载在此表中注册的函数。如果服务器使用`--skip-grant-tables`选项启动，则表中注册的函数不会被加载，也无法使用。参见 Section 7.7.1, “Installing and Uninstalling Loadable Functions”。

    注意

    类似于`mysql.func`系统表，性能模式`user_defined_functions`表列出使用`CREATE FUNCTION`安装的可加载函数。与`mysql.func`表不同，`user_defined_functions`表还列出了由服务器组件或插件自动安装的函数。这种差异使得`user_defined_functions`比`mysql.func`更适合用于检查已安装的函数。参见 Section 29.12.21.10, “The user_defined_functions Table”。

+   `plugin`: 通过`INSTALL PLUGIN`安装的服务器端插件的注册表。在正常启动序列期间，服务器会加载在此表中注册的插件。如果服务器使用`--skip-grant-tables`选项启动，则表中注册的插件不会被加载，也无法使用。参见 Section 7.6.1, “Installing and Uninstalling Plugins”。

### 日志系统表

服务器使用这些系统表进行日志记录：

+   `general_log`: 一般查询日志表。

+   `slow_log`: 慢查询日志表。

日志表使用`CSV`存储引擎。

更多信息，请参见 Section 7.4, “MySQL Server Logs”。

### 服务器端帮助系统表

这些系统表包含服务器端帮助信息：

+   `help_category`: 关于帮助类别的信息。

+   `help_keyword`: 与帮助主题相关联的关键词。

+   `help_relation`: 帮助关键词和主题之间的映射。

+   `help_topic`: 帮助主题内容。

更多信息，请参见 Section 7.1.17, “Server-Side Help Support”。

### 时区系统表

这些系统表包含时区信息：

+   `time_zone`: 时区 ID 以及它们是否使用闰秒。

+   `time_zone_leap_second`: 当闰秒发生时。

+   `time_zone_name`: 时区 ID 和名称之间的映射。

+   `time_zone_transition`, `time_zone_transition_type`: 时区描述。

更多信息，请参阅第 7.1.15 节，“MySQL 服务器时区支持”。

### 复制系统表

服务器使用这些系统表来支持复制：

+   `gtid_executed`: 用于存储 GTID 值的表。请参阅 mysql.gtid_executed 表。

+   `ndb_binlog_index`: NDB 集群复制的二进制日志信息。只有在服务器构建时使用`NDBCLUSTER`支持时才会创建此表。请参阅第 25.7.4 节，“NDB 集群复制模式和表”。

+   `slave_master_info`, `slave_relay_log_info`, `slave_worker_info`: 用于在副本服务器上存储复制信息。请参阅第 19.2.4 节，“中继日志和复制元数据存储库”。

所有列出的表格都使用`InnoDB`存储引擎。

### 优化器系统表

这些系统表供优化器使用：

+   `innodb_index_stats`, `innodb_table_stats`: 用于`InnoDB`持久性优化器统计信息。请参阅第 17.8.10.1 节，“配置持久性优化器统计参数”。

+   `server_cost`, `engine_cost`: 优化器成本模型使用包含有关查询执行过程中发生的操作的成本估算信息的表格。`server_cost`包含一般服务器操作的优化器成本估算。`engine_cost`包含特定存储引擎的操作的估算。请参阅第 10.9.5 节，“优化器成本模型”。

### 其他系统表

其他系统表不适用于前述类别：

+   `audit_log_filter`, `audit_log_user`: 如果安装了 MySQL 企业审计，这些表提供审计日志过滤器定义和用户帐户的持久性存储。请参阅审计日志表。

+   `firewall_group_allowlist`, `firewall_groups`, `firewall_memebership`, `firewall_users`, `firewall_whitelist`: 如果安装了 MySQL 企业防火墙，这些表提供防火墙使用的信息的持久性存储。请参阅第 8.4.7 节，“MySQL 企业防火墙”。

+   `servers`: 用于`FEDERATED`存储引擎。参见 Section 18.8.2.2, “使用 CREATE SERVER 创建 FEDERATED 表”。

+   `innodb_dynamic_metadata`: 由`InnoDB`存储引擎使用，用于存储快速变化的表元数据，如自增计数器值和索引树损坏标志。取代了存放在`InnoDB`系统表空间中的数据字典缓冲表。
