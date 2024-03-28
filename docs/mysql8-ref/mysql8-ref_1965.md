# 28.4.19 INFORMATION_SCHEMA INNODB_FT_INDEX_TABLE 表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/information-schema-innodb-ft-index-table-table.html`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-innodb-ft-index-table-table.html)

`INNODB_FT_INDEX_TABLE`表提供了关于用于处理针对`InnoDB`表的`FULLTEXT`索引的文本搜索的倒排索引的信息。

此表最初为空。在查询之前，请将`innodb_ft_aux_table`系统变量的值设置为包含`FULLTEXT`索引的表的名称（包括数据库名称）（例如，`test/articles`）。

有关相关用法信息和示例，请参见 Section 17.15.4，“InnoDB INFORMATION_SCHEMA FULLTEXT Index Tables”。

`INNODB_FT_INDEX_TABLE`表具有以下列：

+   `WORD`

    从`FULLTEXT`的文本中提取的一个单词。

+   `FIRST_DOC_ID`

    此单词在`FULLTEXT`索引中首次出现的文档 ID。

+   `LAST_DOC_ID`

    此单词在`FULLTEXT`索引中出现的最后一个文档 ID。

+   `DOC_COUNT`

    此单词在`FULLTEXT`索引中出现的行数。同一个单词可以在缓存表中出现多次，每次对应不同的`DOC_ID`和`POSITION`值组合。

+   `DOC_ID`

    包含该单词的行的文档 ID。此值可能反映您为基础表定义的 ID 列的值，或者当表不包含合适的列时，它可以是`InnoDB`生成的序列值。

+   `POSITION`

    该单词在由`DOC_ID`值标识的相关文档中的特定实例的位置。

#### 注意事项

+   这个表最初是空的。在查询之前，将`innodb_ft_aux_table`系统变量的值设置为包含`FULLTEXT`索引的表的名称（包括数据库名称）（例如，`test/articles`）。以下示例演示了如何使用`innodb_ft_aux_table`系统变量显示指定表的`FULLTEXT`索引信息。在新插入行的信息出现在`INNODB_FT_INDEX_TABLE`之前，必须将`FULLTEXT`索引缓存刷新到磁盘。这可以通过在启用`innodb_optimize_fulltext_only`系统变量的情况下对带有索引的表运行`OPTIMIZE TABLE`操作来实现。（示例在最后再次禁用该变量，因为它只打算暂时启用。）

    ```sql
    mysql> USE test;

    mysql> CREATE TABLE articles (
             id INT UNSIGNED AUTO_INCREMENT NOT NULL PRIMARY KEY,
             title VARCHAR(200),
             body TEXT,
             FULLTEXT (title,body)
           ) ENGINE=InnoDB;

    mysql> INSERT INTO articles (title,body) VALUES
           ('MySQL Tutorial','DBMS stands for DataBase ...'),
           ('How To Use MySQL Well','After you went through a ...'),
           ('Optimizing MySQL','In this tutorial we show ...'),
           ('1001 MySQL Tricks','1\. Never run mysqld as root. 2\. ...'),
           ('MySQL vs. YourSQL','In the following database comparison ...'),
           ('MySQL Security','When configured properly, MySQL ...');

    mysql> SET GLOBAL innodb_optimize_fulltext_only=ON;

    mysql> OPTIMIZE TABLE articles;
    +---------------+----------+----------+----------+
    | Table         | Op       | Msg_type | Msg_text |
    +---------------+----------+----------+----------+
    | test.articles | optimize | status   | OK       |
    +---------------+----------+----------+----------+

    mysql> SET GLOBAL innodb_ft_aux_table = 'test/articles';

    mysql> SELECT WORD, DOC_COUNT, DOC_ID, POSITION
           FROM INFORMATION_SCHEMA.INNODB_FT_INDEX_TABLE LIMIT 5;
    +------------+-----------+--------+----------+
    | WORD       | DOC_COUNT | DOC_ID | POSITION |
    +------------+-----------+--------+----------+
    | 1001       |         1 |      4 |        0 |
    | after      |         1 |      2 |       22 |
    | comparison |         1 |      5 |       44 |
    | configured |         1 |      6 |       20 |
    | database   |         2 |      1 |       31 |
    +------------+-----------+--------+----------+

    mysql> SET GLOBAL innodb_optimize_fulltext_only=OFF;
    ```

+   你必须拥有`PROCESS`权限才能查询这个表。

+   使用`INFORMATION_SCHEMA` `COLUMNS`表或`SHOW COLUMNS`语句查看有关这个表的列的其他信息，包括数据类型和默认值。

+   有关`InnoDB` `FULLTEXT`搜索的更多信息，请参见 Section 17.6.2.4, “InnoDB Full-Text Indexes”，以及 Section 14.9, “Full-Text Search Functions”。
