# 16.7 数据字典使用差异

> 原文：[`dev.mysql.com/doc/refman/8.0/en/data-dictionary-usage-differences.html`](https://dev.mysql.com/doc/refman/8.0/en/data-dictionary-usage-differences.html)

使用启用数据字典的 MySQL 服务器与没有数据字典的服务器相比，存在一些操作上的差异：

+   以前，启用`innodb_read_only`系统变量仅阻止了`InnoDB`存储引擎的表的创建和删除。从 MySQL 8.0 开始，启用`innodb_read_only`阻止了所有存储引擎的这些操作。任何存储引擎的表创建和删除操作都会修改`mysql`系统数据库中的数据字典表，但这些表使用`InnoDB`存储引擎，在启用`innodb_read_only`时无法修改。对需要修改数据字典表的其他表操作也适用相同原则。例如：

    +   `ANALYZE TABLE`失败，因为它更新了存储在数据字典中的表统计信息。

    +   `ALTER TABLE *`tbl_name`* ENGINE=*`engine_name`*`失败，因为它更新了存储在数据字典中的存储引擎指定。

    注意

    启用`innodb_read_only`对`mysql`系统数据库中的非数据字典表也有重要影响。有关详细信息，请参阅第 17.14 节，“InnoDB 启动选项和系统变量”中对`innodb_read_only`的描述。

+   以前，`mysql`系统数据库中的表对 DML 和 DDL 语句可见。从 MySQL 8.0 开始，数据字典表是不可见的，不能直接修改或查询。然而，在大多数情况下，有对应的`INFORMATION_SCHEMA`表可以查询。这使得在服务器开发进行时可以更改底层数据字典表，同时保持稳定的`INFORMATION_SCHEMA`接口供应用程序使用。

+   MySQL 8.0 中的`INFORMATION_SCHEMA`表与数据字典密切相关，导致了几个使用上的差异：

    +   以前，在`STATISTICS`和`TABLES`表中查询表统计信息的`INFORMATION_SCHEMA`查询直接从存储引擎中检索统计信息。从 MySQL 8.0 开始，默认情况下使用缓存的表统计信息。`information_schema_stats_expiry`系统变量定义缓存表统计信息过期之前的时间段。默认值为 86400 秒（24 小时）。（要随时更新给定表的缓存值，请使用`ANALYZE TABLE`。）如果没有缓存的统计信息或统计信息已过期，则在查询表统计列时从存储引擎中检索统计信息。要始终直接从存储引擎检索最新的统计信息，请将`information_schema_stats_expiry`设置为`0`。有关更多信息，请参见 Section 10.2.3, “Optimizing INFORMATION_SCHEMA Queries”。

    +   几个`INFORMATION_SCHEMA`表是数据字典表上的视图，这使得优化器可以使用这些基础表上的索引。因此，根据优化器的选择，`INFORMATION_SCHEMA`查询的结果行顺序可能与以前的结果不同。如果查询结果必须具有特定的行排序特性，请包含`ORDER BY`子句。

    +   对`INFORMATION_SCHEMA`表的查询可能以不同的大小写返回列名，而不同于早期的 MySQL 系列。应用程序应以不区分大小写的方式测试结果集列名。如果这不可行，一个解决方法是在选择列表中使用列别名，以返回所需大小写的列名。例如：

        ```sql
        SELECT TABLE_SCHEMA AS table_schema, TABLE_NAME AS table_name
        FROM INFORMATION_SCHEMA.TABLES WHERE TABLE_NAME = 'users';
        ```

    +   **mysqldump** 和 **mysqlpump** 不再转储`INFORMATION_SCHEMA`数据库，即使在命令行上明确命名。

    +   `CREATE TABLE *`dst_tbl`* LIKE *`src_tbl`*` 要求*`src_tbl`*是一个基本表，如果是一个在数据字典表上的`INFORMATION_SCHEMA`表的视图，则会失败。

    +   以前，从`INFORMATION_SCHEMA`表中选择的列的结果集标题使用查询中指定的大写。这个查询生成一个带有`table_name`标题的结果集：

        ```sql
        SELECT table_name FROM INFORMATION_SCHEMA.TABLES;
        ```

        从 MySQL 8.0 开始，这些标题是大写的；前面的查询生成一个带有`TABLE_NAME`标题的结果集。如果需要，可以使用列别名来实现不同的大小写。例如：

        ```sql
        SELECT table_name AS 'table_name' FROM INFORMATION_SCHEMA.TABLES;
        ```

+   数据目录影响着**mysqldump**和**mysqlpump**从`mysql`系统数据库中导出信息的方式：

    +   以前，可以导出`mysql`系统数据库中的所有表。从 MySQL 8.0 开始，**mysqldump**和**mysqlpump**只会导出该数据库中的非数据字典表。

    +   以前，在使用`--all-databases`选项时，不需要`--routines`和`--events`选项来包含存储过程和事件：导出包括`mysql`系统数据库，因此也包括包含存储过程和事件定义的`proc`和`event`表。从 MySQL 8.0 开始，`event`和`proc`表不再使用。相应对象的定义存储在数据字典表中，但这些表不会被导出。要在使用`--all-databases`进行导出时包含存储过程和事件，需要显式使用`--routines`和`--events`选项。

    +   以前，`--routines`选项需要`proc`表的`SELECT`权限。从 MySQL 8.0 开始，该表不再使用；`--routines`现在需要全局的`SELECT`权限。

    +   以前，可以一起导出存储过程和事件定义以及它们的创建和修改时间戳，通过导出`proc`和`event`表。从 MySQL 8.0 开始，这些表不再使用，因此无法导出时间戳。

+   以前，创建包含非法字符的存储过程会产生警告。从 MySQL 8.0 开始，这将会报错。
