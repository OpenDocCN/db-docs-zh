# 28.3.38 The INFORMATION_SCHEMA TABLES Table

> 原文：[`dev.mysql.com/doc/refman/8.0/en/information-schema-tables-table.html`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-tables-table.html)

[`TABLES`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-tables-table.html)表提供有关��据库中表的信息。

[`TABLES`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-tables-table.html)中表示表统计信息的列保存了缓存值。[`information_schema_stats_expiry`](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_information_schema_stats_expiry)系统变量定义了缓存表统计信息过期之前的时间段。默认值为 86400 秒（24 小时）。如果没有缓存统计信息或统计信息已过期，在查询表统计信息列时，将从存储引擎中检索统计信息。要随时更新给定表的缓存值，请使用[`ANALYZE TABLE`](https://dev.mysql.com/doc/refman/8.0/en/analyze-table.html)语句。要始终直接从存储引擎中检索最新统计信息，请将[`information_schema_stats_expiry`](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_information_schema_stats_expiry)设置为`0`。更多信息，请参见[Section 10.2.3, “Optimizing INFORMATION_SCHEMA Queries”](https://dev.mysql.com/doc/refman/8.0/en/information-schema-optimization.html)。

注意

如果启用了[`innodb_read_only`](https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html#sysvar_innodb_read_only)系统变量，则[`ANALYZE TABLE`](https://dev.mysql.com/doc/refman/8.0/en/analyze-table.html)可能会失败，因为它无法更新使用`InnoDB`的数据字典中的统计表。对于更新键分布的[`ANALYZE TABLE`](https://dev.mysql.com/doc/refman/8.0/en/analyze-table.html)操作，即使操作更新了表本身（例如，如果它是`MyISAM`表），也可能会发生失败。要获取更新后的分布统计信息，请将[`information_schema_stats_expiry=0`](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_information_schema_stats_expiry)。

[`TABLES`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-tables-table.html)表具有以下列：

+   `TABLE_CATALOG`

    表所属的目录名称。该值始终为`def`。

+   `TABLE_SCHEMA`

    表所属的模式（数据库）的名称。

+   `TABLE_NAME`

    表的名称。

+   `TABLE_TYPE`

    表的`BASE TABLE`，视图的`VIEW`，或`INFORMATION_SCHEMA`表的`SYSTEM VIEW`。

    [`TABLES`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-tables-table.html)表不列出`TEMPORARY`表。

+   `ENGINE`

    表的存储引擎。请参阅[Chapter 17, *The InnoDB Storage Engine*](https://dev.mysql.com/doc/refman/8.0/en/innodb-storage-engine.html)，以及[Chapter 18, *Alternative Storage Engines*](https://dev.mysql.com/doc/refman/8.0/en/storage-engines.html)。

    对于分区表，`ENGINE`显示所有分区使用的存储引擎的名称。

+   `VERSION`

    此列未使用。随着 MySQL 8.0 中`.frm`文件的移除，此列现在报告一个硬编码值`10`，这是 MySQL 5.7 中使用的最后一个`.frm`文件版本。

+   `ROW_FORMAT`

    行存储格式（`Fixed`，`Dynamic`，`Compressed`，`Redundant`，`Compact`）。对于`MyISAM`表，`Dynamic`对应于**myisamchk -dvv**报告的`Packed`。

+   `TABLE_ROWS`

    行数。一些存储引擎，如`MyISAM`，存储确切的计数。对于其他存储引擎，如`InnoDB`，这个值是一个近似值，可能与实际值相差 40%至 50%。在这种情况下，使用`SELECT COUNT(*)`来获得准确的计数。

    对于`INFORMATION_SCHEMA`表，`TABLE_ROWS`为`NULL`。

    对于`InnoDB`表，行计数仅是 SQL 优化中使用的粗略估计。（如果`InnoDB`表被分区，这也是正确的。）

+   `AVG_ROW_LENGTH`

    平均行长度。

+   `DATA_LENGTH`

    对于`MyISAM`，`DATA_LENGTH`是数据文件的长度，以字节为单位。

    对于`InnoDB`，`DATA_LENGTH`是近似为聚簇索引分配的空间量，以字节为单位。具体来说，它是聚簇索引大小（以页为单位）乘以`InnoDB`页大小。

    有关其他存储引擎的信息，请参考本节末尾的注释。

+   `MAX_DATA_LENGTH`

    对于`MyISAM`，`MAX_DATA_LENGTH`是数据文件的最大长度。这是可以存储在表中的数据字节数总数，考虑到使用的数据指针大小。

    对于`InnoDB`未使用。

    有关其他存储引擎的信息，请参考本节末尾的注释。

+   `INDEX_LENGTH`

    对于`MyISAM`，`INDEX_LENGTH`是索引文件的长度，以字节为单位。

    对于`InnoDB`，`INDEX_LENGTH`是非聚簇索引分配的空间量的近似值，以字节为单位。具体来说，它是非聚簇索引大小（以页为单位）的总和，乘以`InnoDB`页大小。

    有关其他存储引擎的信息，请参考本节末尾的注释。

+   `DATA_FREE`

    已分配但未使用字节的数量。

    `InnoDB`表报告表所属表空间的空闲空间。对于位于共享表空间中的表，这是共享表空间的空闲空间。如果您使用多个表空间并且表有自己的表空间，则空闲空间仅适用于该表。空闲空间表示完全空闲范围中的字节数减去安全边界。即使空闲空间显示为 0，也可能可以插入行，只要不需要分配新的范围。

    对于 NDB Cluster，`DATA_FREE`显示在磁盘上为磁盘数据表或片段分配但未使用的空间。（内存数据资源使用由`DATA_LENGTH`列报告。）

    对于分区表，此值仅为估计值，可能不完全正确。在这种情况下，获取此信息的更准确方法是查询`INFORMATION_SCHEMA` `PARTITIONS`表，如下例所示：

    ```sql
    SELECT SUM(DATA_FREE)
        FROM  INFORMATION_SCHEMA.PARTITIONS
        WHERE TABLE_SCHEMA = 'mydb'
        AND   TABLE_NAME   = 'mytable';
    ```

    更多信息，请参见 Section 28.3.21, “The INFORMATION_SCHEMA PARTITIONS Table”。

+   `AUTO_INCREMENT`

    下一个`AUTO_INCREMENT`值。

+   `CREATE_TIME`

    表创建的时间。

+   `UPDATE_TIME`

    表上次更新的时间。对于某些存储引擎，此值为`NULL`。即使使用每个`InnoDB`表在单独的`.ibd`文件中的 file-per-table 模式，change buffering 也可能延迟对数据文件的写入，因此文件修改时间与最后插入、更新或删除的时间不同。对于`MyISAM`，使用数据文件时间戳；但在 Windows 上，时间戳不会被更新，因此值不准确。

    `UPDATE_TIME`显示了对未分区的`InnoDB`表执行的最后一次`UPDATE`、`INSERT`或`DELETE`的时间戳值。对于 MVCC，时间戳值反映了`COMMIT`时间，被视为最后更新时间。当服务器重新启动或表从`InnoDB`数据字典缓存中删除时，时间戳不会被持久化。

+   `CHECK_TIME`

    上次检查表的时间。并非所有存储引擎都会更新此时间，如果不更新，则值始终为`NULL`。

    对于分区的`InnoDB`表，`CHECK_TIME`始终为`NULL`。

+   `TABLE_COLLATION`

    表的默认排序规则。输出中没有明确列出表的默认字符集，但排序规则名称以字符集名称开头。

+   `CHECKSUM`

    活动校验和值（如果有）。

+   `CREATE_OPTIONS`

    与`CREATE TABLE`一起使用的额外选项。

    `CREATE_OPTIONS`显示为分区表。

    在 MySQL 8.0.16 之前，`CREATE_OPTIONS` 显示了为在每个文件表空间中创建的表指定的 `ENCRYPTION` 子句。从 MySQL 8.0.16 开始，如果表已加密或指定的加密与模式加密不同，则显示每个文件表空间的加密子句。对于在一般表空间中创建的表，不显示加密子句。要识别加密的每个文件表空间和一般表空间，请查询 `INNODB_TABLESPACES` 的 `ENCRYPTION` 列。

    在创建表时禁用 严格模式，如果指定的行格式不受支持，则使用存储引擎的默认行格式。表的实际行格式在 `ROW_FORMAT` 列中报告。`CREATE_OPTIONS` 显示在 `CREATE TABLE` 语句中指定的行格式。

    当更改表的存储引擎时，不适用于新存储引擎的表选项保留在表定义中，以便在必要时将具有先前定义选项的表还原为原始存储引擎。`CREATE_OPTIONS` 列可能显示保留的选项。

+   `TABLE_COMMENT`

    创建表时使用的注释（或者为什么 MySQL 无法访问表信息的信息）。

#### 注意

+   对于 `NDB` 表，此语句的输出显示 `AVG_ROW_LENGTH` 和 `DATA_LENGTH` 列的适当值，但不考虑 `BLOB` 列。

+   对于 `NDB` 表，`DATA_LENGTH` 仅包括存储在主内存中的数据；`MAX_DATA_LENGTH` 和 `DATA_FREE` 列适用于磁盘数据。

+   对于 NDB 集群磁盘数据表，`MAX_DATA_LENGTH` 显示为磁盘数据表或片段的磁盘部分分配的空间。（内存数据资源使用由 `DATA_LENGTH` 列报告。）

+   对于 `MEMORY` 表，`DATA_LENGTH`、`MAX_DATA_LENGTH` 和 `INDEX_LENGTH` 的值近似于实际分配的内存量。分配算法会大量保留内存以减少分配操作的数量。

+   对于视图，大多数 `TABLES` 列为 0 或 `NULL`，除了 `TABLE_NAME` 指示视图名称，`CREATE_TIME` 指示创建时间，`TABLE_COMMENT` 显示 `VIEW`。

表信息也可以通过`SHOW TABLE STATUS`和`SHOW TABLES`语句获取。请参阅 Section 15.7.7.38, “SHOW TABLE STATUS Statement”和 Section 15.7.7.39, “SHOW TABLES Statement”。以下语句是等效的：

```sql
SELECT
    TABLE_NAME, ENGINE, VERSION, ROW_FORMAT, TABLE_ROWS, AVG_ROW_LENGTH,
    DATA_LENGTH, MAX_DATA_LENGTH, INDEX_LENGTH, DATA_FREE, AUTO_INCREMENT,
    CREATE_TIME, UPDATE_TIME, CHECK_TIME, TABLE_COLLATION, CHECKSUM,
    CREATE_OPTIONS, TABLE_COMMENT
  FROM INFORMATION_SCHEMA.TABLES
  WHERE table_schema = '*db_name*'
  [AND table_name LIKE '*wild*']

SHOW TABLE STATUS
  FROM *db_name*
  [LIKE '*wild*']
```

以下语句是等效的：

```sql
SELECT
  TABLE_NAME, TABLE_TYPE
  FROM INFORMATION_SCHEMA.TABLES
  WHERE table_schema = '*db_name*'
  [AND table_name LIKE '*wild*']

SHOW FULL TABLES
  FROM *db_name*
  [LIKE '*wild*']
```
