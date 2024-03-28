# 28.4.18 INFORMATION_SCHEMA INNODB_FT_INDEX_CACHE 表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/information-schema-innodb-ft-index-cache-table.html`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-innodb-ft-index-cache-table.html)

`INNODB_FT_INDEX_CACHE` 表提供了关于`FULLTEXT`索引中新插入行的标记信息。为了避免在 DML 操作期间进行昂贵的索引重组，新索引单词的信息被单独存储，并且仅在运行`OPTIMIZE TABLE`、服务器关闭或缓存大小超过由`innodb_ft_cache_size`或`innodb_ft_total_cache_size`系统变量定义的限制时，才与主搜索索引合并。

这个表最初是空的。在查询之前，将`innodb_ft_aux_table`系统变量的值设置为包含`FULLTEXT`索引的表的名称（包括数据库名称）（例如，`test/articles`）。

有关相关用法信息和示例，请参见第 17.15.4 节，“InnoDB INFORMATION_SCHEMA FULLTEXT Index Tables”。

`INNODB_FT_INDEX_CACHE` 表具有以下列：

+   `WORD`

    从新插入行的文本中提取的一个单词。

+   `FIRST_DOC_ID`

    这个单词在`FULLTEXT`索引中首次出现的文档 ID。

+   `LAST_DOC_ID`

    这个单词在`FULLTEXT`索引中最后一次出现的文档 ID。

+   `DOC_COUNT`

    这个单词在`FULLTEXT`索引中出现的行数。同一个单词可以在缓存表中出现多次，每次都对应`DOC_ID`和`POSITION`值的组合。

+   `DOC_ID`

    新插入行的文档 ID。这个值可能反映了您为基础表定义的 ID 列的值，或者当表不包含合适的列时，它可以是`InnoDB`生成的序列值。

+   `POSITION`

    这个单词在由`DOC_ID`值标识的相关文档中的特定实例的位置。该值不代表绝对位置；它是添加到该单词上一个实例的`POSITION`的偏移量。

#### 注意

+   此表最初为空。在查询之前，请将`innodb_ft_aux_table`系统变量的值设置为包含`FULLTEXT`索引的表的名称（包括数据库名称）（例如`test/articles`）。以下示例演示了如何使用`innodb_ft_aux_table`系统变量来显示指定表的`FULLTEXT`索引的信息。

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

    mysql> SET GLOBAL innodb_ft_aux_table = 'test/articles';

    mysql> SELECT WORD, DOC_COUNT, DOC_ID, POSITION
           FROM INFORMATION_SCHEMA.INNODB_FT_INDEX_CACHE LIMIT 5;
    +------------+-----------+--------+----------+
    | WORD       | DOC_COUNT | DOC_ID | POSITION |
    +------------+-----------+--------+----------+
    | 1001       |         1 |      4 |        0 |
    | after      |         1 |      2 |       22 |
    | comparison |         1 |      5 |       44 |
    | configured |         1 |      6 |       20 |
    | database   |         2 |      1 |       31 |
    +------------+-----------+--------+----------+
    ```

+   您必须具有`PROCESS`权限才能查询此表。

+   使用`INFORMATION_SCHEMA` `COLUMNS`表或`SHOW COLUMNS`语句查看有关此表的列的其他信息，包括数据类型和默认值。

+   关于`InnoDB` `FULLTEXT`搜索的更多信息，请参阅 Section 17.6.2.4, “InnoDB Full-Text Indexes”和 Section 14.9, “Full-Text Search Functions”。
