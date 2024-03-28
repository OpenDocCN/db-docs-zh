# 28.4.14 INNODB_FT_BEING_DELETED 表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/information-schema-innodb-ft-being-deleted-table.html`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-innodb-ft-being-deleted-table.html)

`INNODB_FT_BEING_DELETED` 表提供了 `INNODB_FT_DELETED` 表的快照；仅在 `OPTIMIZE TABLE` 维护操作期间使用。运行 `OPTIMIZE TABLE` 时，`INNODB_FT_BEING_DELETED` 表被清空，并且 `DOC_ID` 值从 `INNODB_FT_DELETED` 表中删除。由于 `INNODB_FT_BEING_DELETED` 的内容通常具有较短的生命周期，因此此表对于监视或调试具有有限的实用性。有关在具有 `FULLTEXT` 索引的表上运行 `OPTIMIZE TABLE` 的信息，请参见 第 14.9.6 节，“调整 MySQL 全文搜索”。

此表最初为空。在查询之前，请将 `innodb_ft_aux_table` 系统变量的值设置为包含 `FULLTEXT` 索引的表的名称（包括数据库名称）（例如，`test/articles`）。输出类似于为 `INNODB_FT_DELETED` 表提供的示例。

有关相关用法信息和示例，请参见 第 17.15.4 节，“InnoDB INFORMATION_SCHEMA FULLTEXT 索引表”。

`INNODB_FT_BEING_DELETED` 表包含以下列：

+   `DOC_ID`

    正在删除过程中的行的文档 ID。此值可能反映您为基础表定义的 ID 列的值，或者当表不包含适当列时，可以是`InnoDB`生成的序列值。在执行文本搜索时，此值用于在通过 `OPTIMIZE TABLE` 语句物理删除`FULLTEXT`索引中已删除行的数据之前，跳过`INNODB_FT_INDEX_TABLE` 表中的行。有关更多信息，请参见 优化 InnoDB 全文索引。

#### 注意

+   使用`INFORMATION_SCHEMA` `COLUMNS` 表或 `SHOW COLUMNS` 语句查看有关此表的列的其他信息，包括数据类型和默认值。

+   您必须具有`PROCESS`权限才能查询此表。

+   有关`InnoDB` `FULLTEXT`搜索的更多信息，请参见 Section 17.6.2.4, “InnoDB Full-Text Indexes”，以及 Section 14.9, “Full-Text Search Functions”。
