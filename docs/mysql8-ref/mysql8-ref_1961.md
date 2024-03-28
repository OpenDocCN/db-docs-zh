# 28.4.15 INFORMATION_SCHEMA INNODB_FT_CONFIG 表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/information-schema-innodb-ft-config-table.html`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-innodb-ft-config-table.html)

`INNODB_FT_CONFIG` 表提供有关`InnoDB`表的`FULLTEXT`索引和相关处理的元数据。

这个表最初是空的。在查询之前，将`innodb_ft_aux_table`系统变量的值设置为包含`FULLTEXT`索引的表的名称（包括数据库名称）（例如，`test/articles`）。

有关相关用法信息和示例，请参见 Section 17.15.4，“InnoDB INFORMATION_SCHEMA FULLTEXT Index Tables”。

`INNODB_FT_CONFIG` 表具有以下列：

+   `KEY`

    指定包含`FULLTEXT`索引的`InnoDB`表的元数据项的名称。

    此列的值可能会更改，具体取决于性能调整和调试`InnoDB`全文处理的需求。关键名称及其含义包括：

    +   `optimize_checkpoint_limit`：运行`OPTIMIZE TABLE`后停止的秒数。

    +   `synced_doc_id`：下一个要发行的`DOC_ID`。

    +   `stopword_table_name`：用户定义的停用词表的*`database/table`*名称。如果没有用户定义的停用词表，则`VALUE`列为空。

    +   `use_stopword`：指示是否使用停用词表，该表在创建`FULLTEXT`索引时定义。

+   `VALUE`

    与相应`KEY`列关联的值，反映`InnoDB`表的`FULLTEXT`索引的某个方面的限制或当前值。

#### 示例

```sql
mysql> SELECT * FROM INFORMATION_SCHEMA.INNODB_FT_CONFIG;
+---------------------------+-------------------+
| KEY                       | VALUE             |
+---------------------------+-------------------+
| optimize_checkpoint_limit | 180               |
| synced_doc_id             | 0                 |
| stopword_table_name       | test/my_stopwords |
| use_stopword              | 1                 |
+---------------------------+-------------------+
```

#### 注意

+   此表仅用于内部配置。不用于统计信息目的。

+   您必须具有`PROCESS`权限才能查询此表。

+   使用`INFORMATION_SCHEMA` `COLUMNS` 表或`SHOW COLUMNS`语句查看有关此表的列的其他信息，包括数据类型和默认值。

+   有关`InnoDB` `FULLTEXT`搜索的更多信息，请参见 Section 17.6.2.4，“InnoDB Full-Text Indexes”，以及 Section 14.9，“Full-Text Search Functions”。
