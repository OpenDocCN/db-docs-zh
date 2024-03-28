# 3.6 准备升级安装

> 原文：[`dev.mysql.com/doc/refman/8.0/en/upgrade-prerequisites.html`](https://dev.mysql.com/doc/refman/8.0/en/upgrade-prerequisites.html)

在升级到最新的 MySQL 8.0 版本之前，请通过执行以下描述的初步检查来确保您当前的 MySQL 5.7 或 MySQL 8.0 服务器实例的升级准备就绪。否则，升级过程可能会失败。

提示

考虑使用 MySQL Shell 升级检查工具，它使您能够验证 MySQL 服务器实例是否准备好升级。您可以选择一个目标 MySQL Server 版本，从 MySQL Server 8.0.11 到与当前 MySQL Shell 版本号匹配的 MySQL Server 版本号。升级检查工具执行与指定目标版本相关的自动检查，并建议您进行进一步的手动检查。升级检查工具适用于 MySQL 5.7、8.0 和 8.3 的所有 GA 版本。MySQL Shell 的安装说明可以在这里找到。

初步检查：

1.  不能存在以下问题：

    +   不能有使用过时数据类型或函数的表。

        如果表中包含旧的时间列（`TIME`、`DATETIME`和`TIMESTAMP`列不支持分数秒精度）的旧格式（在 5.6.4 之前），则不支持直接升级到 MySQL 8.0。如果您的表仍然使用旧的时间列格式，请在尝试直接升级到 MySQL 8.0 之前使用`REPAIR TABLE`进行升级。有关更多信息，请参阅服务器更改，在 MySQL 5.7 参考手册中。

    +   不能有孤立的`.frm`文件。

    +   触发器不能有缺失或空的定义者，也不能有无效的创建上下文（由`SHOW TRIGGERS`显示的`character_set_client`，`collation_connection`，`Database Collation`属性或`INFORMATION_SCHEMA` `TRIGGERS`表）。任何此类触发器必须被导出和恢复以修复问题。

    要检查这些问题，请执行以下命令：

    ```sql
    mysqlcheck -u root -p --all-databases --check-upgrade
    ```

    如果**mysqlcheck**报告任何错误，请纠正这些问题。

1.  不能有使用不具有本机分区支持的存储引擎的分区表。要识别这样的表，请执行以下查询：

    ```sql
    SELECT TABLE_SCHEMA, TABLE_NAME
    FROM INFORMATION_SCHEMA.TABLES
    WHERE ENGINE NOT IN ('innodb', 'ndbcluster')
    AND CREATE_OPTIONS LIKE '%partitioned%';
    ```

    查询报告的任何表必须被更改为使用`InnoDB`或变为非分区表。要将表存储引擎更改为`InnoDB`，请执行此语句：

    ```sql
    ALTER TABLE *table_name* ENGINE = INNODB;
    ```

    有关将`MyISAM`表转换为`InnoDB`的信息，请参见 Section 17.6.1.5，“从 MyISAM 转换表到 InnoDB”。

    要使分区表变为非分区表，请执行此语句：

    ```sql
    ALTER TABLE *table_name* REMOVE PARTITIONING;
    ```

1.  在 MySQL 8.0 中可能保留了以前未保留的一些关键字。请参见 Section 11.3，“关键字和保留字”。这可能导致以前用作标识符的单词变得非法。要修复受影响的语句，请使用标识符引用。请参见 Section 11.2，“模式对象名称”。

1.  MySQL 5.7 `mysql`系统数据库中不能有与 MySQL 8.0 数据字典使用的表同名的表。要识别具有这些名称的表，请执行此查询：

    ```sql
    SELECT TABLE_SCHEMA, TABLE_NAME
    FROM INFORMATION_SCHEMA.TABLES
    WHERE LOWER(TABLE_SCHEMA) = 'mysql'
    and LOWER(TABLE_NAME) IN
    (
    'catalogs',
    'character_sets',
    'check_constraints',
    'collations',
    'column_statistics',
    'column_type_elements',
    'columns',
    'dd_properties',
    'events',
    'foreign_key_column_usage',
    'foreign_keys',
    'index_column_usage',
    'index_partitions',
    'index_stats',
    'indexes',
    'parameter_type_elements',
    'parameters',
    'resource_groups',
    'routines',
    'schemata',
    'st_spatial_reference_systems',
    'table_partition_values',
    'table_partitions',
    'table_stats',
    'tables',
    'tablespace_files',
    'tablespaces',
    'triggers',
    'view_routine_usage',
    'view_table_usage'
    );
    ```

    查询报告的任何表必须被删除或重命名（使用`RENAME TABLE`）。这可能还需要对使用受影响表的应用程序进行更改。

1.  不能有外键约束名超过 64 个字符的表。使用此查询标识约束名过长的表：

    ```sql
    SELECT TABLE_SCHEMA, TABLE_NAME
    FROM INFORMATION_SCHEMA.TABLES
    WHERE TABLE_NAME IN
      (SELECT LEFT(SUBSTR(ID,INSTR(ID,'/')+1),
                   INSTR(SUBSTR(ID,INSTR(ID,'/')+1),'_ibfk_')-1)
       FROM INFORMATION_SCHEMA.INNODB_SYS_FOREIGN
       WHERE CHAR_LENGTH(SUBSTR(ID,INSTR(ID,'/')+1))>64);
    ```

    对于约束名超过 64 个字符的表，请删除约束并使用不超过 64 个字符的约束名重新添加它（使用`ALTER TABLE`）。

1.  不能定义由`sql_mode`系统变量定义的过时 SQL 模式。尝试使用过时的 SQL 模式会阻止 MySQL 8.0 启动。应该修改使用过时 SQL 模式的应用程序以避免它们。有关 MySQL 8.0 中删除的 SQL 模式的信息，请参见服务器更改。

1.  不能有显式定义列名超过 64 个字符的视图（MySQL 5.7 允许具有长达 255 个字符的列名的视图）。为避免升级错误，应在升级之前修改此类视图。目前，识别列名超过 64 个字符的视图的唯一方法是使用`SHOW CREATE VIEW`检查视图定义。您还可以通过查询信息模式`VIEWS`表来检查视图定义。

1.  在个别`ENUM`或`SET`列元素中，字符长度超过 255 个字符或 1020 个字节的表格或存储过程是不允许的。在 MySQL 8.0 之前，`ENUM`或`SET`列元素的最大组合长度为 64K。在 MySQL 8.0 中，单个`ENUM`或`SET`列元素的最大字符长度为 255 个字符，最大字节长度为 1020 个字节（1020 字节限制支持多字节字符集）。在升级到 MySQL 8.0 之前，修改任何超出新限制的`ENUM`或`SET`列元素。如果不这样做，升级将因错误而失败。

1.  在升级到 MySQL 8.0.13 或更高版本之前，不能有驻留在共享`InnoDB`表空间中的表分区，其中包括系统表空间和通用表空间。通过查询`INFORMATION_SCHEMA`来识别共享表空间中的表分区：

    如果从 MySQL 5.7 升级，请运行此查询：

    ```sql
    SELECT DISTINCT NAME, SPACE, SPACE_TYPE FROM INFORMATION_SCHEMA.INNODB_SYS_TABLES
      WHERE NAME LIKE '%#P#%' AND SPACE_TYPE NOT LIKE 'Single';
    ```

    如果从较早的 MySQL 8.0 版本升级，请运行此查询：

    ```sql
    SELECT DISTINCT NAME, SPACE, SPACE_TYPE FROM INFORMATION_SCHEMA.INNODB_TABLES
      WHERE NAME LIKE '%#P#%' AND SPACE_TYPE NOT LIKE 'Single';
    ```

    使用`ALTER TABLE ... REORGANIZE PARTITION`将表分区从共享表空间移动到每个表的表空间：

    ```sql
    ALTER TABLE *table_name* REORGANIZE PARTITION *partition_name*
      INTO (*partition_definition* TABLESPACE=innodb_file_per_table);
    ```

1.  不能有来自 MySQL 8.0.12 或更低版本的查询和存储程序定义使用`ASC`或`DESC`修饰符用于`GROUP BY`子句。否则，升级到 MySQL 8.0.13 或更高版本可能会失败，复制到 MySQL 8.0.13 或更高版本的复制服务器也可能失败。有关更多详细信息，请参见 SQL Changes。

1.  您的 MySQL 5.7 安装不得使用 MySQL 8.0 不支持的功能。这里的任何更改都是特定于安装的，但以下示例说明了要查找的内容：

    在 MySQL 8.0 中已删除了一些服务器启动选项和系统变量。请参阅 MySQL 8.0 中删除的功能，以及第 1.4 节，“MySQL 8.0 中添加、弃用或删除的服务器和状态变量和选项”。如果您使用其中任何内容，升级需要进行配置更改。

    例如：由于数据字典提供有关数据库对象的信息，服务器不再检查数据目录中的目录名称以查找数据库。因此，`--ignore-db-dir`选项是多余的并已被移除。为了处理这个问题，在升级到 MySQL 8.0 之前，从启动配置中删除任何`--ignore-db-dir`的实例。此外，在升级到 MySQL 8.0 之前，删除或移动命名数据目录的子目录。（或者，让 8.0 服务器将这些目录添加到数据字典作为数据库，然后使用`DROP DATABASE`删除每个数据库。）

1.  如果您打算在升级时将`lower_case_table_names`设置为 1，请确保在升级之前模式和表名称均为小写。否则，可能会由于模式或表名称大小写不匹配而导致失败。您可以使用以下查询检查包含大写字符的模式和表名称：

    ```sql
    mysql> select TABLE_NAME, if(sha(TABLE_NAME) !=sha(lower(TABLE_NAME)),'Yes','No') as UpperCase from information_schema.tables;
    ```

    截至 MySQL 8.0.19 版本，如果`lower_case_table_names=1`，升级过程将检查表和模式名称，确保所有字符均为小写。如果发现表或模式名称包含大写字符，则升级过程将因错误而失败。

    注意

    不建议在升级时更改`lower_case_table_names`设置。

如果由于上述任何问题导致 MySQL 8.0 升级失败，则服务器将撤销对数据目录的所有更改。在这种情况下，删除所有重做日志文件并在现有数据目录上重新启动 MySQL 5.7 服务器以解决错误。重做日志文件（`ib_logfile*`）默认位于 MySQL 数据目录中。在错误修复后，在尝试再次升级之前执行慢关闭（通过设置`innodb_fast_shutdown=0`）。
