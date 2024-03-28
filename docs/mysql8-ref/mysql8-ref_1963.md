# 28.4.17 INFORMATION_SCHEMA INNODB_FT_DELETED 表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/information-schema-innodb-ft-deleted-table.html`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-innodb-ft-deleted-table.html)

`INNODB_FT_DELETED`表存储从`InnoDB`表的`FULLTEXT`索引中删除的行。为了避免在`InnoDB` `FULLTEXT`索引的 DML 操作期间进行昂贵的索引重组，新删除单词的信息被单独存储，当进行文本搜索时被过滤出搜索结果，并且只有在为`InnoDB`表发出`OPTIMIZE TABLE`时才从主搜索索引中删除。有关更多信息，请参见优化 InnoDB 全文索引。

这个表最初是空的。在查询之前，将`innodb_ft_aux_table`系统变量的值设置为包含`FULLTEXT`索引的表的名称（包括数据库名称）（例如，`test/articles`）。

有关相关用法信息和示例，请参见 Section 17.15.4, “InnoDB INFORMATION_SCHEMA FULLTEXT Index Tables”。

`INNODB_FT_DELETED`表具有以下列：

+   `DOC_ID`

    新删除行的文档 ID。这个值可能反映您为基础表定义的 ID 列的值，或者当表不包含合适的列时，它可以是`InnoDB`生成的序列值。在执行文本搜索时，此值用于跳过在`INNODB_FT_INDEX_TABLE`表中为已删除行物理删除`FULLTEXT`索引数据之前的行，通过`OPTIMIZE TABLE`语句。有关更多信息，请参见优化 InnoDB 全文索引。

#### 示例

```sql
mysql> SELECT * FROM INFORMATION_SCHEMA.INNODB_FT_DELETED;
+--------+
| DOC_ID |
+--------+
|      6 |
|      7 |
|      8 |
+--------+
```

#### 注意

+   你必须拥有`PROCESS`权限才能查询这个表。

+   使用`INFORMATION_SCHEMA` `COLUMNS`表或`SHOW COLUMNS`语句查看关于这个表的列的额外信息，包括数据类型和默认值。

+   关于`InnoDB`的`FULLTEXT`搜索的更多信息，请参阅 Section 17.6.2.4, “InnoDB Full-Text Indexes”和 Section 14.9, “Full-Text Search Functions”。
