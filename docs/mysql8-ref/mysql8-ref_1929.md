# 28.3.34 The INFORMATION_SCHEMA STATISTICS Table

> 原文：[`dev.mysql.com/doc/refman/8.0/en/information-schema-statistics-table.html`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-statistics-table.html)

`STATISTICS`表提供有关表索引的信息。

`STATISTICS`中表示表统计信息的列保存了缓存值。`information_schema_stats_expiry`系统变量定义了缓存表统计信息过期之前的时间段。默认值为 86400 秒（24 小时）。如果没有缓存的统计信息或统计信息已过期，则在查询表统计信息列时从存储引擎中检索统计信息。要随时更新给定表的缓存值，请使用`ANALYZE TABLE`。要始终直接从存储引擎中检索最新的统计信息，请设置`information_schema_stats_expiry=0`。有关更多信息，请参见 Section 10.2.3，“优化 INFORMATION_SCHEMA 查询”。

注意

如果启用了`innodb_read_only`系统变量，则可能因为无法更新使用`InnoDB`的数据字典中的统计信息表而导致`ANALYZE TABLE`失败。即使操作更新了表本身（例如，如果是`MyISAM`表），对更新键分布的`ANALYZE TABLE`操作也可能导致失败。要获取更新后的分布统计信息，请设置`information_schema_stats_expiry=0`。

`STATISTICS`表具有以下列：

+   `TABLE_CATALOG`

    包含索引的表所属的目录的名称。此值始终为`def`。

+   `TABLE_SCHEMA`

    包含索引的表所属的模式（数据库）的名称。

+   `TABLE_NAME`

    包含索引的表的名称。

+   `NON_UNIQUE`

    如果索引不能包含重复项，则为 0，如果可以则为 1。

+   `INDEX_SCHEMA`

    索引所属的模式（数据库）的名称。

+   `INDEX_NAME`

    索引的名称。如果索引是主键，则名称始终为`PRIMARY`。

+   `SEQ_IN_INDEX`

    索引中的列序号，从 1 开始。

+   `COLUMN_NAME`

    列名。另请参阅`EXPRESSION`列的描述。

+   `COLLATION`

    列在索引中的排序方式。这可以是`A`（升序），`D`（降序）或`NULL`（未排序）。

+   `CARDINALITY`

    索引中唯一值的估计数量。要更新此数字，请运行`ANALYZE TABLE`或（对于`MyISAM`表）**myisamchk -a**。

    `CARDINALITY`是基于存储为整数的统计数据计算的，因此即使对于小表，该值也不一定是精确的。基数越高，MySQL 在执行连接时使用索引的可能性就越大。

+   `SUB_PART`

    索引前缀。也就是，如果列仅部分索引，则索引字符数，如果整个列被索引，则为`NULL`。

    注意

    前缀*限制*以字节为单位。但是，在`CREATE TABLE`，`ALTER TABLE`和`CREATE INDEX`语句中的索引规范中，对于非二进制字符串类型（`CHAR`，`VARCHAR`，`TEXT`），前缀*长度*被解释为多字节字符集的字符数，对于二进制字符串类型（`BINARY`，`VARBINARY`，`BLOB`），前缀*长度*以字节为单位。在为使用多字节字符集的非二进制字符串列指定前缀长度时，请考虑这一点。

    有关索引前缀的其他信息，请参见第 10.3.5 节，“列索引”和第 15.1.15 节，“CREATE INDEX Statement”。

+   `PACKED`

    指示键如何打包。如果不是，则为`NULL`。

+   `NULLABLE`

    包含`YES`，如果列可能包含`NULL`值，`''`如果不包含。

+   `INDEX_TYPE`

    使用的索引方法（`BTREE`，`FULLTEXT`，`HASH`，`RTREE`）。

+   `COMMENT`

    关于索引的信息，未在其自己的列中描述，例如如果索引已禁用，则为`disabled`。

+   `INDEX_COMMENT`

    创建索引时使用`COMMENT`属性提供的索引的任何注释。

+   `IS_VISIBLE`

    索引是否对优化器可见。请参见第 10.3.12 节，“不可见索引”。

+   `EXPRESSION`

    MySQL 8.0.13 及更高版本支持功能键部分（参见功能键部分），这影响`COLUMN_NAME`和`EXPRESSION`列：

    +   对于非功能键部分，`COLUMN_NAME`指示由键部分索引的列，`EXPRESSION`为`NULL`。

    +   对于功能键部分，`COLUMN_NAME`列为`NULL`，而`EXPRESSION`表示键部分的表达式。

#### 注意

+   没有用于索引的标准`INFORMATION_SCHEMA`表。MySQL 列列表类似于 SQL Server 2000 返回的`sp_statistics`，只是`QUALIFIER`和`OWNER`分别替换为`CATALOG`和`SCHEMA`。

表索引的信息也可以从`SHOW INDEX`语句中获取。请参见第 15.7.7.22 节，“SHOW INDEX Statement”。以下语句是等效的：

```sql
SELECT * FROM INFORMATION_SCHEMA.STATISTICS
  WHERE table_name = '*tbl_name*'
  AND table_schema = '*db_name*'

SHOW INDEX
  FROM *tbl_name*
  FROM *db_name*
```

在 MySQL 8.0.30 及更高版本中，默认情况下，此表中显示有关生成的不可见主键列的信息。您可以通过设置`show_gipk_in_create_table_and_information_schema = OFF`来隐藏此类信息。有关更多信息，请参见第 15.1.20.11 节，“生成的不可见主键”。
