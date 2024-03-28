# 15.1.9 ALTER TABLE 语句

> 原文：[`dev.mysql.com/doc/refman/8.0/en/alter-table.html`](https://dev.mysql.com/doc/refman/8.0/en/alter-table.html)

15.1.9.1 ALTER TABLE 分区操作

15.1.9.2 ALTER TABLE 和生成列

15.1.9.3 ALTER TABLE 示例

```sql
ALTER TABLE *tbl_name*
    [*alter_option* [, *alter_option*] ...]
    [*partition_options*]

*alter_option*: {
    *table_options*
  | ADD [COLUMN] *col_name* *column_definition*
        [FIRST | AFTER *col_name*]
  | ADD [COLUMN] (*col_name* *column_definition*,...)
  | ADD {INDEX | KEY} [*index_name*]
        [*index_type*] (*key_part*,...) [*index_option*] ...
  | ADD {FULLTEXT | SPATIAL} [INDEX | KEY] [*index_name*]
        (*key_part*,...) [*index_option*] ...
  | ADD [CONSTRAINT [*symbol*]] PRIMARY KEY
        [*index_type*] (*key_part*,...)
        [*index_option*] ...
  | ADD [CONSTRAINT [*symbol*]] UNIQUE [INDEX | KEY]
        [*index_name*] [*index_type*] (*key_part*,...)
        [*index_option*] ...
  | ADD [CONSTRAINT [*symbol*]] FOREIGN KEY
        [*index_name*] (*col_name*,...)
        *reference_definition*
  | ADD [CONSTRAINT [*symbol*]] CHECK (*expr*) [[NOT] ENFORCED]
  | DROP {CHECK | CONSTRAINT} *symbol*
  | ALTER {CHECK | CONSTRAINT} *symbol* [NOT] ENFORCED
  | ALGORITHM [=] {DEFAULT | INSTANT | INPLACE | COPY}
  | ALTER [COLUMN] *col_name* {
        SET DEFAULT {*literal* | (*expr*)}
      | SET {VISIBLE | INVISIBLE}
      | DROP DEFAULT
    }
  | ALTER INDEX *index_name* {VISIBLE | INVISIBLE}
  | CHANGE [COLUMN] *old_col_name* *new_col_name* *column_definition*
        [FIRST | AFTER *col_name*]
  | [DEFAULT] CHARACTER SET [=] *charset_name* [COLLATE [=] *collation_name*]
  | CONVERT TO CHARACTER SET *charset_name* [COLLATE *collation_name*]
  | {DISABLE | ENABLE} KEYS
  | {DISCARD | IMPORT} TABLESPACE
  | DROP [COLUMN] *col_name*
  | DROP {INDEX | KEY} *index_name*
  | DROP PRIMARY KEY
  | DROP FOREIGN KEY *fk_symbol*
  | FORCE
  | LOCK [=] {DEFAULT | NONE | SHARED | EXCLUSIVE}
  | MODIFY [COLUMN] *col_name* *column_definition*
        [FIRST | AFTER *col_name*]
  | ORDER BY *col_name* [, *col_name*] ...
  | RENAME COLUMN *old_col_name* TO *new_col_name*
  | RENAME {INDEX | KEY} *old_index_name* TO *new_index_name*
  | RENAME [TO | AS] *new_tbl_name*
  | {WITHOUT | WITH} VALIDATION
}

*partition_options*:
    *partition_option* [*partition_option*] ...

*partition_option*: {
    ADD PARTITION (*partition_definition*)
  | DROP PARTITION *partition_names*
  | DISCARD PARTITION {*partition_names* | ALL} TABLESPACE
  | IMPORT PARTITION {*partition_names* | ALL} TABLESPACE
  | TRUNCATE PARTITION {*partition_names* | ALL}
  | COALESCE PARTITION *number*
  | REORGANIZE PARTITION *partition_names* INTO (*partition_definitions*)
  | EXCHANGE PARTITION *partition_name* WITH TABLE *tbl_name* [{WITH | WITHOUT} VALIDATION]
  | ANALYZE PARTITION {*partition_names* | ALL}
  | CHECK PARTITION {*partition_names* | ALL}
  | OPTIMIZE PARTITION {*partition_names* | ALL}
  | REBUILD PARTITION {*partition_names* | ALL}
  | REPAIR PARTITION {*partition_names* | ALL}
  | REMOVE PARTITIONING
}

*key_part*: {*col_name* [(*length*)] | (*expr*)} [ASC | DESC]

*index_type*:
    USING {BTREE | HASH}

*index_option*: {
    KEY_BLOCK_SIZE [=] *value*
  | *index_type*
  | WITH PARSER *parser_name*
  | COMMENT '*string*'
  | {VISIBLE | INVISIBLE}
}

*table_options*:
    *table_option* [[,] *table_option*] ...

*table_option*: {
    AUTOEXTEND_SIZE [=] *value*
  | AUTO_INCREMENT [=] *value*
  | AVG_ROW_LENGTH [=] *value*
  | [DEFAULT] CHARACTER SET [=] *charset_name*
  | CHECKSUM [=] {0 | 1}
  | [DEFAULT] COLLATE [=] *collation_name*
  | COMMENT [=] '*string*'
  | COMPRESSION [=] {'ZLIB' | 'LZ4' | 'NONE'}
  | CONNECTION [=] '*connect_string*'
  | {DATA | INDEX} DIRECTORY [=] '*absolute path to directory*'
  | DELAY_KEY_WRITE [=] {0 | 1}
  | ENCRYPTION [=] {'Y' | 'N'}
  | ENGINE [=] *engine_name*
  | ENGINE_ATTRIBUTE [=] '*string*'
  | INSERT_METHOD [=] { NO | FIRST | LAST }
  | KEY_BLOCK_SIZE [=] *value*
  | MAX_ROWS [=] *value*
  | MIN_ROWS [=] *value*
  | PACK_KEYS [=] {0 | 1 | DEFAULT}
  | PASSWORD [=] '*string*'
  | ROW_FORMAT [=] {DEFAULT | DYNAMIC | FIXED | COMPRESSED | REDUNDANT | COMPACT}
  | SECONDARY_ENGINE_ATTRIBUTE [=] '*string*'
  | STATS_AUTO_RECALC [=] {DEFAULT | 0 | 1}
  | STATS_PERSISTENT [=] {DEFAULT | 0 | 1}
  | STATS_SAMPLE_PAGES [=] *value*
  | TABLESPACE *tablespace_name* [STORAGE {DISK | MEMORY}]
  | UNION [=] (*tbl_name*[,*tbl_name*]...)
}

*partition_options*:
    (see CREATE TABLE options)
```

`ALTER TABLE` 改变表的结构。例如，您可以添加或删除列，创建或销毁索引，更改现有列的类型，或重命名列或表本身。您还可以更改诸如用于表的存储引擎或表注释等特性。

+   要使用`ALTER TABLE`，您需要对表具有`ALTER`、`CREATE`和`INSERT`权限。重命名表需要对旧表具有`ALTER`和`DROP`权限，对新表具有`ALTER`、`CREATE`和`INSERT`权限。

+   在表名之后，指定要进行的更改。如果没有给出任何更改，则`ALTER TABLE`不执行任何操作。

+   许多允许的更改的语法与`CREATE TABLE`语句的子句类似。*`column_definition`* 子句对于 `ADD` 和 `CHANGE` 使用与`CREATE TABLE`相同的语法。有关更多信息，请参见 Section 15.1.20, “CREATE TABLE Statement”。

+   `COLUMN` 这个词是可选的，可以省略，除了 `RENAME COLUMN`（用于区分列重命名操作和表重命名操作）。

+   在单个`ALTER TABLE`语句中允许多个 `ADD`、`ALTER`、`DROP` 和 `CHANGE` 子句，用逗号分隔。这是 MySQL 对标准 SQL 的扩展，标准 SQL 每个`ALTER TABLE`语句只允许每个子句中的一个。例如，要在单个语句中删除多个列，请执行以下操作：

    ```sql
    ALTER TABLE t2 DROP COLUMN c, DROP COLUMN d;
    ```

+   如果存储引擎不支持尝试的`ALTER TABLE`操作，则可能会产生警告。此类警告可以使用`SHOW WARNINGS`显示。请参阅第 15.7.7.42 节，“SHOW WARNINGS 语句”。有关故障排除`ALTER TABLE`的信息，请参阅附录 B.3.6.1，“ALTER TABLE 问题”。

+   有关生成列的信息，请参阅第 15.1.9.2 节，“ALTER TABLE 和生成列”。

+   有关用法示例，请参阅第 15.1.9.3 节，“ALTER TABLE 示例”。

+   在 MySQL 8.0.17 及更高版本中，`InnoDB`支持对 JSON 列添加多值索引，使用*`key_part`*规范可以采用`(CAST *`json_path`* AS *`type`* ARRAY)`的形式。请参阅多值索引，了解有关多值索引创建和使用的详细信息，以及多值索引的限制和限制。

+   使用`mysql_info()` C API 函数，您可以查找由`ALTER TABLE`复制了多少行。请参阅 mysql_info()。

`ALTER TABLE`语句还有几个额外方面，在本节中的以下主题下进行了描述：

+   表选项

+   性能和空间要求

+   并发控制

+   添加和删除列

+   重命名、重新定义和重新排序列

+   主键和索引

+   外键和其他约束

+   更改字符集

+   导入 InnoDB 表

+   MyISAM 表的行顺序

+   分区选项

#### 表选项

*`table_options`*表示可以在`CREATE TABLE`语句中使用的表选项，例如`ENGINE`、`AUTO_INCREMENT`、`AVG_ROW_LENGTH`、`MAX_ROWS`、`ROW_FORMAT`或`TABLESPACE`。

有关所有表选项的描述，请参见 Section 15.1.20, “CREATE TABLE Statement”。但是，当作为表选项给出时，`ALTER TABLE`会忽略`DATA DIRECTORY`和`INDEX DIRECTORY`。`ALTER TABLE`仅允许它们作为分区选项，并要求您具有`FILE`权限。

使用`ALTER TABLE`的表选项提供了一种方便的方式来更改单个表的特性。例如：

+   如果`t1`当前不是`InnoDB`表，则此语句将其存储引擎更改为`InnoDB`。

    ```sql
    ALTER TABLE t1 ENGINE = InnoDB;
    ```

    +   有关将表切换到`InnoDB`存储引擎时的注意事项，请参见 Section 17.6.1.5, “Converting Tables from MyISAM to InnoDB”。

    +   当指定`ENGINE`子句时，`ALTER TABLE`会重建表。即使表已经具有指定的存储引擎，这也是正确的。

    +   在现有的`InnoDB`表上运行`ALTER TABLE *`tbl_name`* ENGINE=INNODB`执行一个“null”`ALTER TABLE`操作，可用于碎片整理`InnoDB`表，如 Section 17.11.4, “Defragmenting a Table”中所述。在`InnoDB`表上运行`ALTER TABLE *`tbl_name`* FORCE`执行相同的功能。

    +   `ALTER TABLE *`tbl_name`* ENGINE=INNODB`和`ALTER TABLE *`tbl_name`* FORCE`使用在线 DDL。有关更多信息，请参见 Section 17.12, “InnoDB and Online DDL”。

    +   尝试更改表的存储引擎的结果受所需存储引擎是否可用以及`NO_ENGINE_SUBSTITUTION` SQL 模式设置的影响，如 Section 7.1.11, “Server SQL Modes”中所述。

    +   为防止数据意外丢失，`ALTER TABLE`不能用于将表的存储引擎更改为`MERGE`或`BLACKHOLE`。

+   要将`InnoDB`表更改为使用压缩行存储格式：

    ```sql
    ALTER TABLE t1 ROW_FORMAT = COMPRESSED;
    ```

+   `ENCRYPTION`子句为`InnoDB`表启用或禁用页面级数据加密。必须安装和配置一个密钥环插件才能启用加密。

    如果启用了`table_encryption_privilege_check`变量，则需要`TABLE_ENCRYPTION_ADMIN`权限才能使用与默认模式加密设置不同的设置的`ENCRYPTION`子句。

    在 MySQL 8.0.16 之前，`ENCRYPTION`子句仅在更改位于每表一个文件表空间中的表时受支持。从 MySQL 8.0.16 开始，`ENCRYPTION`子句也支持位于通用表空间中的表。

    对于位于通用表空间中的表，表和表空间加密必须匹配。

    不允许通过将表移动到不同的表空间或更改存储引擎来更改表加密，而不明确指定`ENCRYPTION`子句。

    从 MySQL 8.0.16 开始，如果表使用不支持加密的存储引擎，则不允许使用值为`'N'`或`''`的`ENCRYPTION`子句。以前，该子句是被接受的。在使用不支持加密的存储引擎在启用加密的模式中创建不带`ENCRYPTION`子句的表也是不允许的。

    欲了解更多信息，请参阅第 17.13 节，“InnoDB 数据静态加密”。

+   要重置当前自增值：

    ```sql
    ALTER TABLE t1 AUTO_INCREMENT = 13;
    ```

    不能将计数器重置为小于或等于当前正在使用的值。对于`InnoDB`和`MyISAM`，如果该值小于或等于当前在`AUTO_INCREMENT`列中的最大值，则该值将重置为当前最大`AUTO_INCREMENT`列值加一。

+   要更改默认表字符集：

    ```sql
    ALTER TABLE t1 CHARACTER SET = utf8mb4;
    ```

    另请参阅更改字符集。

+   要添加（或更改）表注释：

    ```sql
    ALTER TABLE t1 COMMENT = 'New table comment';
    ```

+   使用带有`TABLESPACE`选项的`ALTER TABLE`来在现有通用表空间、每表一个文件表空间和系统表空间之间移动`InnoDB`表。参见使用 ALTER TABLE 在表空间之间移动表。

    +   `ALTER TABLE ... TABLESPACE` 操作总是导致完整的表重建，即使`TABLESPACE`属性未从其先前值更改。

    +   `ALTER TABLE ... TABLESPACE` 语法不支持将表从临时表空间移动到持久表空间。

    +   支持与`CREATE TABLE ... TABLESPACE`一起使用的`DATA DIRECTORY`子句，在`ALTER TABLE ... TABLESPACE`中不受支持，并且如果指定了将被忽略。

    +   有关`TABLESPACE`选项的功能和限制的更多信息，请参见`CREATE TABLE`。

+   MySQL NDB Cluster 8.0 支持设置`NDB_TABLE`选项，用于控制表的分区平衡（片段计数类型）、从任何副本读取的能力、完全复制，或这些选项的任意组合，作为`ALTER TABLE`语句的表注释的一部分，与`CREATE TABLE`一样，如下例所示：

    ```sql
    ALTER TABLE t1 COMMENT = "NDB_TABLE=READ_BACKUP=0,PARTITION_BALANCE=FOR_RA_BY_NODE";
    ```

    还可以在`ALTER TABLE`语句中为`NDB`表的列设置`NDB_COMMENT`选项，如下所示：

    ```sql
    ALTER TABLE t1 
      CHANGE COLUMN c1 c1 BLOB 
        COMMENT = 'NDB_COLUMN=BLOB_INLINE_SIZE=4096,MAX_BLOB_PART_SIZE';
    ```

    通过 NDB 8.0.30 及更高版本支持以这种方式设置 blob 内联大小。请注意，`ALTER TABLE ... COMMENT ...`会丢弃表的任何现有注释。有关更多信息和示例，请参见设置 NDB_TABLE 选项。

+   `ENGINE_ATTRIBUTE`和`SECONDARY_ENGINE_ATTRIBUTE`选项（自 MySQL 8.0.21 起可用）用于指定主要和次要存储引擎的表、列和索引属性。这些选项保留供将来使用。索引属性无法更改。必须删除索引并以所需更改重新添加，这可以在单个`ALTER TABLE`语句中执行。

要验证表选项是否按预期更改，请使用`SHOW CREATE TABLE`，或查询信息模式`TABLES`表。

#### 性能和空间要求

`ALTER TABLE`操作使用以下算法之一进行处理：

+   `COPY`: 操作在原始表的副本上执行，并且表数据逐行从原始表复制到新表。不允许并发的 DML。

+   `INPLACE`: 操作避免复制表数据，但可能会就地重建表。在操作的准备和执行阶段可能会短暂地对表进行独占的元数据锁定。通常支持并发的 DML。

+   `INSTANT`: 操作仅修改数据字典中的元数据。在操作的执行阶段可能会短暂地对表进行独占的元数据锁定。表数据不受影响，使操作瞬间完成。允许并发的 DML。（MySQL 8.0.12 中引入）

对于使用`NDB`存储引擎的表，这些算法的工作方式如下：

+   `COPY`：`NDB`创建表的副本并对其进行更改；然后 NDB Cluster 处理程序在旧表和新表之间复制数据。随后，`NDB`删除旧表并重命名新表。

    有时也被称为“复制”或“离线”`ALTER TABLE`。

+   `INPLACE`：数据节点进行所需更改；NDB Cluster 处理程序不复制数据或以其他方式参与。

    有时也被称为“非复制”或“在线”`ALTER TABLE`。

+   `INSTANT`：不被`NDB`支持。

更多信息请参见 Section 25.6.12, “NDB Cluster 中的 ALTER TABLE 在线操作”。

`ALGORITHM`子句是可选的。如果省略了`ALGORITHM`子句，MySQL 将对支持它的存储引擎和`ALTER TABLE`子句使用`ALGORITHM=INSTANT`。否则，将使用`ALGORITHM=INPLACE`。如果不支持`ALGORITHM=INPLACE`，则使用`ALGORITHM=COPY`。

注意

使用`ALGORITHM=INSTANT`向分区表添加列后，不再可以对表执行`ALTER TABLE ... EXCHANGE PARTITION`操作。

指定`ALGORITHM`子句要求操作使用指定的算法对支持它的子句和存储引擎，否则将失败并显示错误。指定`ALGORITHM=DEFAULT`与省略`ALGORITHM`子句相同。

使用`COPY`算法的`ALTER TABLE`操作会等待正在修改表的其他操作完成。在对表副本应用更改后，数据被复制过去，原始表被删除，表副本被重命名为原始表的名称。在`ALTER TABLE`操作执行时，原始表可以被其他会话读取（除了不久前提到的例外）。在`ALTER TABLE`操作开始后对表的更新和写入被暂停，直到新表准备就绪，然后自动重定向到新表。表的临时副本被创建在原始表的数据库目录中，除非是将表移动到不同目录中的数据库的`RENAME TO`操作。

先前提到的异常是，`ALTER TABLE`在准备清除过时表结构时会阻止读取（不仅仅是写入）。在这一点上，它必须获取独占锁。为此，它等待当前读取器完成，并阻止新的读取和写入。

使用`COPY`算法的`ALTER TABLE`操作会阻止并发的 DML 操作。仍然允许并发查询。也就是说，表复制操作始终至少包括`LOCK=SHARED`的并发限制（允许查询但不允许 DML）。您可以通过指定`LOCK=EXCLUSIVE`进一步限制支持`LOCK`子句的操作的并发性，从而阻止 DML 和查询。有关更多信息，请参阅并发控制。

要强制使用`COPY`算法进行本来不使用的`ALTER TABLE`操作，指定`ALGORITHM=COPY`或启用`old_alter_table`系统变量。如果`old_alter_table`设置与具有非`DEFAULT`值的`ALGORITHM`子句之间存在冲突，则`ALGORITHM`子句优先。

对于`InnoDB`表，使用`COPY`算法进行`ALTER TABLE`操作的表位于共享表空间中，可能会增加表空间使用量。这种操作需要额外的空间，与表中的数据加索引一样多。对于位于共享表空间中的表，在操作期间使用的额外空间不会像位于 file-per-table 表空间中的表那样释放回操作系统。

有关在线 DDL 操作的空间要求，请参阅 Section 17.12.3, “Online DDL Space Requirements”。

支持`INPLACE`算法的`ALTER TABLE`操作包括：

+   `InnoDB`支持的`ALTER TABLE`操作在线 DDL 功能。请参阅 Section 17.12.1, “Online DDL Operations”。

+   重命名表。MySQL 会重命名与表*`tbl_name`*对应的文件而不进行复制。（您也可以使用`RENAME TABLE`语句来重命名表。请参见第 15.1.36 节，“RENAME TABLE Statement”。）专门授予重命名表的权限不会迁移到新名称。必须手动更改它们。

+   仅修改表元数据的操作。这些操作是立即执行的，因为服务器不会触及表内容。仅元数据操作包括：

    +   重命名列。在 NDB Cluster 8.0.18 及更高版本中，此操作也可以在线执行。

    +   更改列的默认值（除了`NDB`表）。

    +   通过在有效成员值列表的*末尾*添加新的枚举或集合成员来修改`ENUM`或`SET`列的定义，只要数据类型的存储大小不变。例如，在具有 8 个成员的`SET`列中添加一个成员会将每个值所需的存储从 1 字节更改为 2 字节；这需要复制表。在列表中间添加成员会导致现有成员的重新编号，这需要复制表。

    +   更改空间列的定义以删除`SRID`属性。（添加或更改`SRID`属性需要重建，不能就地完成，因为服务器必须验证所有值是否具有指定的`SRID`值。）

    +   截至 MySQL 8.0.14，更改列字符集的条件如下：

        +   列数据类型为`CHAR`、`VARCHAR`、`TEXT`类型或`ENUM`。

        +   字符集从`utf8mb3`更改为`utf8mb4`，或任何字符集更改为`binary`。

        +   列上没有索引。

    +   截至 MySQL 8.0.14，更改生成列的条件如下：

        +   对于`InnoDB`表，修改生成的存储列但不更改其类型、表达式或可空性的语句。

        +   对于非`InnoDB`表，修改生成的存储或虚拟列但不更改其类型、表达式或可空性的语句。

        更改列注释的示例。

+   重命名索引。

+   为`InnoDB`和`NDB`表添加或删除辅助索引。请参见第 17.12.1 节，“在线 DDL 操作”。

+   对于`NDB`表，对可变宽度列添加和删除索引的操作。这些操作在线进行，无需复制表格，并且在大部分时间内不会阻塞并发的 DML 操作。请参见第 25.6.12 节，“NDB Cluster 中 ALTER TABLE 的在线操作”。

+   使用`ALTER INDEX`操作修改索引可见性。

+   修改包含依赖于具有`DEFAULT`值的列的生成列的表的列。例如，可以在不重建表格的情况下进行单独列的`NULL`属性更改。

支持`INSTANT`算法的`ALTER TABLE`操作包括：

+   添加列。此功能称为“即时`ADD COLUMN`”。有限制条件。请参见第 17.12.1 节，“在线 DDL 操作”。

+   删除列。此功能称为“即时`DROP COLUMN`”。有限制条件。请参见第 17.12.1 节，“在线 DDL 操作”。

+   添加或删除虚拟列。

+   添加或删除列默认值。

+   修改`ENUM`或`SET`列的定义。与上述描述的`ALGORITHM=INSTANT`相同的限制条件适用。

+   更改索引类型。

+   重命名表。与上述描述的`ALGORITHM=INSTANT`相同的限制条件适用。

有关支持`ALGORITHM=INSTANT`的操作的更多信息，请参见第 17.12.1 节，“在线 DDL 操作”。

`ALTER TABLE`将 MySQL 5.5 的时间列升级为 5.6 格式，用于`ADD COLUMN`、`CHANGE COLUMN`、`MODIFY COLUMN`、`ADD INDEX`和`FORCE`操作。此转换不能使用`INPLACE`算法进行，因为必须重建表格，因此在这些情况下指定`ALGORITHM=INPLACE`会导致错误。如有必要，请指定`ALGORITHM=COPY`。

如果对通过`KEY`对表进行分区的多列索引进行的`ALTER TABLE`操作改变了列的顺序，则只能使用`ALGORITHM=COPY`执行。

`WITHOUT VALIDATION`和`WITH VALIDATION`子句影响`ALTER TABLE`对虚拟生成列修改的是否进行原地操作。请参见第 15.1.9.2 节，“ALTER TABLE 和生成列”。

NDB Cluster 8.0 支持使用与标准 MySQL Server 相同的`ALGORITHM=INPLACE`语法进行在线操作。`NDB`不支持在线更改表空间；从 NDB 8.0.21 开始，不允许这样做。有关更多信息，请参见第 25.6.12 节，“NDB Cluster 中的 ALTER TABLE 在线操作”。

在 NDB 8.0.27 及更高版本中，执行复制`ALTER TABLE`时，会检查确保没有对受影响表进行并发写入。如果发现有任何并发写入，`NDB`会拒绝`ALTER TABLE`语句并引发`ER_TABLE_DEF_CHANGED`。

使用`DISCARD ... PARTITION ... TABLESPACE`或`IMPORT ... PARTITION ... TABLESPACE`的`ALTER TABLE`不会创建任何临时表或临时分区文件。

使用`ADD PARTITION`、`DROP PARTITION`、`COALESCE PARTITION`、`REBUILD PARTITION`或`REORGANIZE PARTITION`的`ALTER TABLE`不会创建临时表（除非与`NDB`表一起使用）；但是，这些操作可以并且会创建临时分区文件。

`RANGE`或`LIST`分区的`ADD`或`DROP`操作是立即操作或几乎是立即操作。`HASH`或`KEY`分区的`ADD`或`COALESCE`操作会在所有分区之间复制数据，除非使用了`LINEAR HASH`或`LINEAR KEY`；这实际上等同于创建一个新表，尽管`ADD`或`COALESCE`操作是逐个分区执行的。`REORGANIZE`操作只复制已更改的分区，不会触及未更改的分区。

对于`MyISAM`表，可以通过将`myisam_sort_buffer_size`系统变量设置为较高的值来加快索引重建（修改过程中最慢的部分）。

#### 并发控制

对于支持的`ALTER TABLE`操作，可以使用`LOCK`子句来控制在修改表时对表进行并发读写的级别。为此子句指定非默认值可以要求在修改操作期间具有一定程度的并发访问或独占性，并且如果所请求的锁定程度不可用，则会停止操作。

仅允许对使用`ALGORITHM=INSTANT`的操作使用`LOCK = DEFAULT`。其他`LOCK`子句参数不适用。

`LOCK`子句的参数为：

+   `LOCK = DEFAULT`

    给定`ALGORITHM`子句（如果有）和`ALTER TABLE`操作的最大并发级别：如果支持，允许并发读写。如果不支持，则允许并发读取。如果不支持，则强制独占访问。

+   `LOCK = NONE`

    如果支持，允许并发读写。否则，将发生错误。

+   `LOCK = SHARED`

    如果支持，允许并发读取但阻止写入。即使存储引擎支持给定`ALGORITHM`子句（如果有）和`ALTER TABLE`操作的并发写入，写入也会被阻止。如果不支持并发读取，则会发生错误。

+   `LOCK = EXCLUSIVE`

    强制独占访问。即使存储引擎支持给定`ALGORITHM`子句（如果有）和`ALTER TABLE`操作的并发读/写，也会执行此操作。

#### 添加和删除列

使用`ADD`来向表中添加新列，使用`DROP`来移除现有列。`DROP *`col_name`*`是 MySQL 对标准 SQL 的扩展。

要在表行中的特定位置添加列，请使用`FIRST`或`AFTER *`col_name`*`。默认情况下，将列添加到最后。

如果表只包含一列，则无法删除该列。如果您打算删除表，请改用`DROP TABLE`语句。

如果从表中删除列，则这些列也将从它们所属的任何索引中删除。如果组成索引的所有列都被删除，则索引也将被删除。如果使用`CHANGE`或`MODIFY`来缩短具有索引的列，并且结果列长度小于索引长度，则 MySQL 会自动缩短索引。

对于`ALTER TABLE ... ADD`，如果列具有使用非确定性函数的表达式默认值，该语句可能会产生警告或错误。有关更多信息，请参见第 13.6 节，“数据类型默认值”和第 19.1.3.7 节，“带有 GTID 的复制限制”。

#### 重命名、重新定义和重新排序列

`CHANGE`，`MODIFY`，`RENAME COLUMN`和`ALTER`子句使得可以更改现有列的名称和定义。它们具有以下比较特性：

+   `CHANGE`：

    +   可以重命名列并更改其定义，或两者兼而有之。

    +   拥有比`MODIFY`或`RENAME COLUMN`更多的功能，但某些操作的便利性受到牺牲。如果不重命名，`CHANGE`需要两次命名列，并且如果只是重命名，则需要重新指定列定义。

    +   使用`FIRST`或`AFTER`可以重新排序列。

+   `MODIFY`：

    +   可以更改列定义但不能更改其名称。

    +   比`CHANGE`更方便，可以更改列定义而不重命名。

    +   使用`FIRST`或`AFTER`可以重新排序列。

+   `RENAME COLUMN`：

    +   可以更改列名但不能更改其定义。

    +   比`CHANGE`更方便，可以重命名列而不改变其定义。

+   `ALTER`：仅用于更改列的默认值。

`CHANGE`是 MySQL 对标准 SQL 的扩展。`MODIFY`和`RENAME COLUMN`是为了 Oracle 兼容性而添加的 MySQL 扩展。

要更改列的名称和定义，请使用`CHANGE`，指定旧名称和新名称以及新定义。例如，要将`INT NOT NULL`列从`a`重命名为`b`并将其定义更改为使用`BIGINT`数据类型同时保留`NOT NULL`属性，请执行以下操作：

```sql
ALTER TABLE t1 CHANGE a b BIGINT NOT NULL;
```

要更改列定义但不更改其名称，请使用`CHANGE`或`MODIFY`。使用`CHANGE`，语法要求两个列名，因此您必须指定相同的名称两次以保持名称不变。例如，要更改列`b`的定义，请执行以下操作：

```sql
ALTER TABLE t1 CHANGE b b INT NOT NULL;
```

`MODIFY`更方便地更改定义而不更改名称，因为它只需要列名一次：

```sql
ALTER TABLE t1 MODIFY b INT NOT NULL;
```

要更改列名称但不更改其定义，请使用`CHANGE`或`RENAME COLUMN`。使用`CHANGE`，语法要求列定义，因此要保持定义不变，您必须重新指定列当前具有的定义。例如，要将`INT NOT NULL`列从`b`重命名为`a`，请执行以下操作：

```sql
ALTER TABLE t1 CHANGE b a INT NOT NULL;
```

使用`RENAME COLUMN`更方便地更改名称而不更改定义，因为它只需要旧名称和新名称：

```sql
ALTER TABLE t1 RENAME COLUMN b TO a;
```

通常，您不能将列重命名为表中已存在的名称。但是，有时情况并非如此，例如当您交换名称或通过循环移动它们时。如果表中有列名为`a`、`b`和`c`，则以下操作是有效的：

```sql
-- swap a and b
ALTER TABLE t1 RENAME COLUMN a TO b,
               RENAME COLUMN b TO a;
-- "rotate" a, b, c through a cycle
ALTER TABLE t1 RENAME COLUMN a TO b,
               RENAME COLUMN b TO c,
               RENAME COLUMN c TO a;
```

对于使用`CHANGE`或`MODIFY`进行列定义更改，定义必须包括数据类型和应适用于新列的所有属性，除了索引属性如`PRIMARY KEY`或`UNIQUE`之外。原始定义中存在但未为新定义指定的属性不会被继承。假设列`col1`被定义为`INT UNSIGNED DEFAULT 1 COMMENT 'my column'`，并且您修改列如下，意图仅将`INT`更改为`BIGINT`：

```sql
ALTER TABLE t1 MODIFY col1 BIGINT;
```

该语句将数据类型从`INT`更改为`BIGINT`，但也会删除`UNSIGNED`、`DEFAULT`和`COMMENT`属性。要保留它们，语句必须显式包含它们：

```sql
ALTER TABLE t1 MODIFY col1 BIGINT UNSIGNED DEFAULT 1 COMMENT 'my column';
```

对于使用`CHANGE`或`MODIFY`进行数据类型更改，MySQL 会尽可能将现有列值转换为新类型。

警告

此转换可能导致数据的更改。例如，如果缩短字符串列，值可能会被截断。为防止如果转换为新数据类型会导致数据丢失，则在使用`ALTER TABLE`之前启用严格的 SQL 模式（请参阅 Section 7.1.11, “Server SQL Modes”）。

如果您使用`CHANGE`或`MODIFY`缩短具有索引的列，并且结果列长度小于索引长度，则 MySQL 会自动缩短索引。

对于由`CHANGE`或`RENAME COLUMN`重命名的列，MySQL 会自动重命名对重命名列的引用：

+   引用旧列的索引，包括不可见索引和禁用的`MyISAM`索引。

+   外键引用旧列。

对于被`CHANGE`或`RENAME COLUMN`重命名的列，MySQL 不会自动重命名这些引用到重命名列的引用：

+   引用重命名列的生成列和分区表达式。您必须在同一`ALTER TABLE`语句中使用`CHANGE`重新定义这样的表达式，以重命名列。

+   引用重命名列的视图和存储程序。您必须手动修改这些对象的定义，以引用新的列名。

要在表内重新排序列，请在`CHANGE`或`MODIFY`操作中使用`FIRST`和`AFTER`。

`ALTER ... SET DEFAULT` 或 `ALTER ... DROP DEFAULT` 分别指定列的新默认值或移除旧的默认值。如果移除了旧的默认值并且列可以为`NULL`，新的默认值将为`NULL`。如果列不能为`NULL`，MySQL 将分配一个默认值，如第 13.6 节，“数据类型默认值”中所述。

截至 MySQL 8.0.23，`ALTER ... SET VISIBLE` 和 `ALTER ... SET INVISIBLE` 允许更改列的可见性。请参见第 15.1.20.10 节，“不可见列”。

#### 主键和索引

`DROP PRIMARY KEY` 删除主键。如果没有主键，将会出现错误。关于主键的性能特性的信息，尤其是对于`InnoDB`表，参见第 10.3.2 节，“主键优化”。

如果启用了`sql_require_primary_key`系统变量，尝试删除主键将产生错误。

如果向表中添加`UNIQUE INDEX`或`PRIMARY KEY`，MySQL 会将其存储在任何非唯一索引之前，以便尽早检测到重复键。

`DROP INDEX` 删除一个索引。这是 MySQL 对标准 SQL 的扩展。请参见第 15.1.27 节，“DROP INDEX 语句”。要确定索引名称，请使用`SHOW INDEX FROM *`tbl_name`*`。

一些存储引擎允许在创建索引时指定索引类型。*`index_type`*指定符的语法为`USING *`type_name`*`。有关`USING`的详细信息，请参见第 15.1.15 节，“CREATE INDEX 语句”。首选位置是在列列表之后。预计在未来的 MySQL 版本中将删除在列列表之前使用该选项的支持。

*`index_option`*值指定索引的附加选项。`USING`是其中之一。有关允许的*`index_option`*值的详细信息，请参见第 15.1.15 节，“CREATE INDEX Statement”。

`RENAME INDEX *`old_index_name`* TO *`new_index_name`*` 重命名索引。这是 MySQL 对标准 SQL 的扩展。表的内容保持不变。*`old_index_name`*必须是表中现有索引的名称，该索引不会被相同的`ALTER TABLE`语句删除。*`new_index_name`*是新的索引名称，在应用更改后的表中不能重复索引的名称。两个索引名称都不能是`PRIMARY`。

如果在`MyISAM`表上使用`ALTER TABLE`，所有非唯一索引将在单独的批处理中创建（就像`REPAIR TABLE`一样）。当有许多索引时，这应该使`ALTER TABLE`速度更快。

对于`MyISAM`表，可以显式控制键的更新。使用`ALTER TABLE ... DISABLE KEYS`告诉 MySQL 停止更新非唯一索引。然后使用`ALTER TABLE ... ENABLE KEYS`重新创建丢失的索引。`MyISAM`使用一种比逐个插入键更快的特殊算法来执行此操作，因此在执行大量插入操作之前禁用键应该会显著加快速度。使用`ALTER TABLE ... DISABLE KEYS`需要除了前面提到的权限之外的`INDEX`权限。

在非唯一索引被禁用时，它们对于诸如`SELECT`和`EXPLAIN`等通常会使用它们的语句将被忽略。

在`ALTER TABLE`语句之后，可能需要运行`ANALYZE TABLE`来更新索引基数信息。参见第 15.7.7.22 节，“SHOW INDEX Statement”。

`ALTER INDEX`操作允许将索引设置为可见或不可见。不可见索引不会被优化器使用。索引可见性的修改适用于主键之外的索引（显式或隐式），并且不能使用`ALGORITHM=INSTANT`执行。此功能与存储引擎无关（支持任何引擎）。有关更多信息，请参见第 10.3.12 节，“不可见索引”。

#### 外键和其他约束

`FOREIGN KEY`和`REFERENCES`子句由`InnoDB`和`NDB`存储引擎支持，它们实现`ADD [CONSTRAINT [*`symbol`*]] FOREIGN KEY [*`index_name`*] (...) REFERENCES ... (...)`。请参见第 15.1.20.5 节，“外键约束”。对于其他存储引擎，这些子句会被解析但被忽略。

对于`ALTER TABLE`，与`CREATE TABLE`不同，如果给定，`ADD FOREIGN KEY`会忽略*`index_name`*并使用自动生成的外键名称。作为解决方法，包含`CONSTRAINT`子句来指定外键名称：

```sql
ADD CONSTRAINT *name* FOREIGN KEY (....) ...
```

重要提示

MySQL 会默默忽略内联的`REFERENCES`规范，其中引用是作为列规范的一部分定义的。MySQL 只接受作为单独`FOREIGN KEY`规范的一部分定义的`REFERENCES`子句。

注意

分区化的`InnoDB`表不支持外键。这个限制不适用于`NDB`表，包括那些明确由`[LINEAR] KEY`分区的表。更多信息，请参见第 26.6.2 节，“与存储引擎相关的分区限制”。

MySQL 服务器和 NDB 集群都支持使用`ALTER TABLE`来删除外键：

```sql
ALTER TABLE *tbl_name* DROP FOREIGN KEY *fk_symbol*;
```

在同一`ALTER TABLE`语句中添加和删除外键对于`ALTER TABLE ... ALGORITHM=INPLACE`是支持的，但对于`ALTER TABLE ... ALGORITHM=COPY`不支持。

服务器禁止更改可能导致引用完整性丢失的外键列。一个解决方法是在更改列定义之前使用`ALTER TABLE ... DROP FOREIGN KEY`，然后在之后使用`ALTER TABLE ... ADD FOREIGN KEY`。禁止的更改示例包括：

+   更改可能不安全的外键列的数据类型。例如，将`VARCHAR(20)`更改为`VARCHAR(30)`是允许的，但将其更改为`VARCHAR(1024)`是不允许的，因为这会改变存储单个值所需的长度字节数。

+   在非严格模式下将`NULL`列更改为`NOT NULL`是被禁止的，以防止将`NULL`值转换为默认的非`NULL`值，而在引用表中没有相应的值。在严格模式下允许此操作，但如果需要任何此类转换，则会返回错误。

`ALTER TABLE *`tbl_name`* RENAME *`new_tbl_name`*` 在内部更改生成的外键约束名称和以字符串“*`tbl_name`*_ibfk_”开头的用户定义的外键约束名称，以反映新表名称。`InnoDB`将以字符串“*`tbl_name`*_ibfk_”开头的外键约束名称解释为内部生成的名称。

在 MySQL 8.0.16 之前，`ALTER TABLE`仅允许以下有限版本的`CHECK`约束添加语法，该语法被解析并忽略：

```sql
ADD CHECK (*expr*)
```

从 MySQL 8.0.16 开始，`ALTER TABLE`允许为现有表添加、删除或更改`CHECK`约束：

+   添加新的`CHECK`约束：

    ```sql
    ALTER TABLE *tbl_name*
        ADD [CONSTRAINT [*symbol*]] CHECK (*expr*) [[NOT] ENFORCED];
    ```

    约束语法元素的含义与`CREATE TABLE`相同。请参阅第 15.1.20.6 节，“CHECK 约束”。

+   删除现有命名为*`symbol`*的`CHECK`约束：

    ```sql
    ALTER TABLE *tbl_name*
        DROP CHECK *symbol*;
    ```

+   更改现有`CHECK`约束命名为*`symbol`*是否强制执行：

    ```sql
    ALTER TABLE *tbl_name*
        ALTER CHECK *symbol* [NOT] ENFORCED;
    ```

`DROP CHECK`和`ALTER CHECK`子句是 MySQL 对标准 SQL 的扩展。

从 MySQL 8.0.19 开始，`ALTER TABLE`允许更通用（符合 SQL 标准）的语法来删除和更改任何类型的现有约束，其中约束类型是根据约束名称确定的：

+   删除现有命名为*`symbol`*的约束：

    ```sql
    ALTER TABLE *tbl_name*
        DROP CONSTRAINT *symbol*;
    ```

    如果启用了`sql_require_primary_key`系统变量，则尝试删除主键会产生错误。

+   更改现有命名为*`symbol`*的约束是否强制执行：

    ```sql
    ALTER TABLE *tbl_name*
        ALTER CONSTRAINT *symbol* [NOT] ENFORCED;
    ```

    只有`CHECK`约束可以更改为不强制执行。所有其他约束类型始终强制执行。

SQL 标准规定所有类型的约束（主键、唯一索引、外键、检查）属于同一命名空间。在 MySQL 中，每种约束类型在每个模式中都有自己的命名空间。因此，每种约束类型的名称在每个模式中必须是唯一的，但不同类型的约束可以具有相同的名称。当多个约束具有相同名称时，`DROP CONSTRAINT`和`ADD CONSTRAINT`是模棱两可的，会导致错误。在这种情况下，必须使用特定于约束的语法来修改约束。例如，使用`DROP PRIMARY KEY`或`DROP FOREIGN KEY`来删除主键或外键。

如果表更改导致违反强制执行的`CHECK`约束，则会发生错误，表不会被修改。导致错误的操作示例：

+   尝试向用于`CHECK`约束的列添加`AUTO_INCREMENT`属性。

+   尝试添加强制执行的`CHECK`约束或强制执行现有行违反约束条件的非强制执行的`CHECK`约束。

+   尝试修改、重命名或删除用于`CHECK`约束的列，除非在同一语句中也删除该约束。例外：如果`CHECK`约束仅引用单个列，则删除该列会自动删除约束。

`ALTER TABLE *`tbl_name`* RENAME *`new_tbl_name`*` 会更改内部生成和用户定义的以字符串“*`tbl_name`*_chk_”开头的`CHECK`约束名称，以反映新表名。MySQL 将以字符串“*`tbl_name`*_chk_”开头的`CHECK`约束名称解释为内部生成的名称。

#### 更改字符集

要更改表的默认字符集和所有字符列(`CHAR`, `VARCHAR`, `TEXT`)为新字符集，使用如下语句：

```sql
ALTER TABLE *tbl_name* CONVERT TO CHARACTER SET *charset_name*;
```

该语句还会更改所有字符列的排序规则。如果未指定`COLLATE`子句以指示使用哪种排序规则，则该语句将使用字符集的默认排序规则。如果此排序规则不适用于预期的表使用（例如，如果从区分大小写的排序规则更改为不区分大小写的排序规则），请明确指定排序规则。

对于数据类型为`VARCHAR`或`TEXT`之一的列，`CONVERT TO CHARACTER SET`会根据需要更改数据类型，以确保新列足够长，可以存储与原始列相同数量的字符。例如，`TEXT`列有两个长度字节，用于存储列中值的字节长度，最多为 65,535。对于`latin1`的`TEXT`列，每个字符需要一个字节，因此该列最多可以存储 65,535 个字符。如果将该列转换为`utf8mb4`，每个字符可能需要多达 4 个字节，因此最大可能长度为 4 × 65,535 = 262,140 字节。该长度不适合`TEXT`列的长度字节，因此 MySQL 将数据类型转换为`MEDIUMTEXT`，这是长度字节可以记录值为 262,140 的最小字符串类型。类似地，`VARCHAR`列可能会转换为`MEDIUMTEXT`。

为避免发生刚才描述的数据类型更改，不要使用`CONVERT TO CHARACTER SET`。而是使用`MODIFY`来更改单个列。例如：

```sql
ALTER TABLE t MODIFY latin1_text_col TEXT CHARACTER SET utf8mb4;
ALTER TABLE t MODIFY latin1_varchar_col VARCHAR(*M*) CHARACTER SET utf8mb4;
```

如果指定 `CONVERT TO CHARACTER SET binary`，则 `CHAR`、`VARCHAR` 和 `TEXT` 列将转换为它们对应的二进制字符串类型（`BINARY`、`VARBINARY`、`BLOB`）。这意味着这些列不再具有字符集，并且随后的 `CONVERT TO` 操作不适用于它们。

如果 `CONVERT TO CHARACTER SET` 操作中的 *`charset_name`* 为 `DEFAULT`，则使用由 `character_set_database` 系统变量命名的字符集。

警告

`CONVERT TO` 操作在原始字符集和命名字符集之间转换列值。如果您有一个列使用一个字符集（如 `latin1`），但存储的值实际上使用另一个不兼容的字符集（如 `utf8mb4`），那么这不是您想要的。在这种情况下，您必须针对每个这样的列执行以下操作：

```sql
ALTER TABLE t1 CHANGE c1 c1 BLOB;
ALTER TABLE t1 CHANGE c1 c1 TEXT CHARACTER SET utf8mb4;
```

这样做的原因是在转换到或从 `BLOB` 列时不进行转换。

要仅更改表的 *默认* 字符集，请使用此语句：

```sql
ALTER TABLE *tbl_name* DEFAULT CHARACTER SET *charset_name*;
```

单词 `DEFAULT` 是可选的。如果您在以后向表添加列时未指定字符集，则默认字符集是使用的字符集（例如，使用 `ALTER TABLE ... ADD column`）。

当启用 `foreign_key_checks` 系统变量时（默认设置），不允许在包含在外键约束中使用字符串列的表上进行字符集转换。解决方法是在执行字符集转换之前禁用 `foreign_key_checks`。在重新启用 `foreign_key_checks` 之前，必须对涉及外键约束的两个表执行转换。如果在只转换其中一个表后重新启用 `foreign_key_checks`，由于在这些操作期间发生的隐式转换，`ON DELETE CASCADE` 或 `ON UPDATE CASCADE` 操作可能会损坏引用表中的数据（Bug #45290，Bug #74816）。

#### 导入 InnoDB 表

在其自己的 file-per-table 表空间中创建的 `InnoDB` 表可以使用 `DISCARD TABLESPACE` 和 `IMPORT TABLESPACE` 子句从备份或另一个 MySQL 服务器实例导入。参见 Section 17.6.1.3, “Importing InnoDB Tables”。

#### MyISAM 表的行顺序

`ORDER BY`使您能够按特定顺序创建新表中的行。此选项主要在您知道大多数情况下以特定顺序查询行时很有用。通过在对表进行重大更改后使用此选项，您可能能够获得更高的性能。在某些情况下，如果表按照稍后要按其排序的列的顺序排列，可能会使 MySQL 更容易进行排序。

注意

表在插入和删除后不会保持指定的顺序。

`ORDER BY`语法允许指定一个或多个列名进行排序，每个列名后面可以选择跟随`ASC`或`DESC`以指示升序或降序排序顺序。默认为升序。只允许列名作为排序标准；不允许任意表达式。此子句应在任何其他子句之后给出。

对于`InnoDB`表，`ORDER BY`没有意义，因为`InnoDB`始终根据聚簇索引对表行进行排序。

当在分区表上使用`ALTER TABLE ... ORDER BY`时，仅对每个分区内的行进行排序。

#### 分区选项

*`partition_options`*表示可用于分区表的选项，用于重新分区、添加、删除、丢弃、导入、合并和拆分分区，以及执行分区维护。

`ALTER TABLE`语句可以包含`PARTITION BY`或`REMOVE PARTITIONING`子句以及其他修改规范，但`PARTITION BY`或`REMOVE PARTITIONING`子句必须在任何其他规范之后指定。`ADD PARTITION`、`DROP PARTITION`、`DISCARD PARTITION`、`IMPORT PARTITION`、`COALESCE PARTITION`、`REORGANIZE PARTITION`、`EXCHANGE PARTITION`、`ANALYZE PARTITION`、`CHECK PARTITION`和`REPAIR PARTITION`选项不能与单个`ALTER TABLE`中的其他修改规范组合，因为列出的选项仅对单个分区起作用。

有关分区选项的更多信息，请参见第 15.1.20 节，“CREATE TABLE Statement”和第 15.1.9.1 节，“ALTER TABLE Partition Operations”。有关`ALTER TABLE ... EXCHANGE PARTITION`语句的信息和示例，请参见第 26.3.3 节，“Exchanging Partitions and Subpartitions with Tables”。
