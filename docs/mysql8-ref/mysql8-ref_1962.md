# 28.4.16 INFORMATION_SCHEMA INNODB_FT_DEFAULT_STOPWORD 表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/information-schema-innodb-ft-default-stopword-table.html`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-innodb-ft-default-stopword-table.html)

`INNODB_FT_DEFAULT_STOPWORD`表保存了在`InnoDB`表上创建`FULLTEXT`索引时默认使用的停用词列表。有关默认`InnoDB`停用词列表以及如何定义自己的停用词列表的信息，请参见第 14.9.4 节，“Full-Text Stopwords”。

有关使用信息和示例，请参见第 17.15.4 节，“InnoDB INFORMATION_SCHEMA FULLTEXT Index Tables”。

`INNODB_FT_DEFAULT_STOPWORD`表具有以下列：

+   `value`

    作为`InnoDB`表上`FULLTEXT`索引的默认停用词使用的单词。如果使用`innodb_ft_server_stopword_table`或`innodb_ft_user_stopword_table`系统变量覆盖默认停用词处理，则不使用此单词。

#### 示例

```sql
mysql> SELECT * FROM INFORMATION_SCHEMA.INNODB_FT_DEFAULT_STOPWORD;
+-------+
| value |
+-------+
| a     |
| about |
| an    |
| are   |
| as    |
| at    |
| be    |
| by    |
| com   |
| de    |
| en    |
| for   |
| from  |
| how   |
| i     |
| in    |
| is    |
| it    |
| la    |
| of    |
| on    |
| or    |
| that  |
| the   |
| this  |
| to    |
| was   |
| what  |
| when  |
| where |
| who   |
| will  |
| with  |
| und   |
| the   |
| www   |
+-------+
36 rows in set (0.00 sec)
```

#### 注意

+   您必须具有`PROCESS`权限才能查询此表。

+   使用`INFORMATION_SCHEMA` `COLUMNS`表或`SHOW COLUMNS`语句查看有关此表的列的其他信息，包括数据类型和默认值。

+   有关`InnoDB` `FULLTEXT`搜索的更多信息，请参见第 17.6.2.4 节，“InnoDB Full-Text Indexes”和第 14.9 节，“Full-Text Search Functions”。
