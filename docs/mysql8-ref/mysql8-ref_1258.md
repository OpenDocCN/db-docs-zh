# 17.15.4 InnoDB INFORMATION_SCHEMA FULLTEXT 索引表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-information-schema-fulltext_index-tables.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-information-schema-fulltext_index-tables.html)

以下表格提供了 `FULLTEXT` 索引的元数据：

```sql
mysql> SHOW TABLES FROM INFORMATION_SCHEMA LIKE 'INNODB_FT%';
+-------------------------------------------+
| Tables_in_INFORMATION_SCHEMA (INNODB_FT%) |
+-------------------------------------------+
| INNODB_FT_CONFIG                          |
| INNODB_FT_BEING_DELETED                   |
| INNODB_FT_DELETED                         |
| INNODB_FT_DEFAULT_STOPWORD                |
| INNODB_FT_INDEX_TABLE                     |
| INNODB_FT_INDEX_CACHE                     |
+-------------------------------------------+
```

#### 表格概述

+   `INNODB_FT_CONFIG`：提供关于 `InnoDB` 表的 `FULLTEXT` 索引和相关处理的元数据。

+   `INNODB_FT_BEING_DELETED`：提供 `INNODB_FT_DELETED` 表的快照；仅在 `OPTIMIZE TABLE` 维护操作期间使用。运行 `OPTIMIZE TABLE` 时，`INNODB_FT_BEING_DELETED` 表会被清空，并且从 `INNODB_FT_DELETED` 表中删除 `DOC_ID` 值。由于 `INNODB_FT_BEING_DELETED` 的内容通常寿命较短，因此该表对于监控或调试具有有限的实用性。有关在具有 `FULLTEXT` 索引的表上运行 `OPTIMIZE TABLE` 的信息，请参见 第 14.9.6 节，“调整 MySQL 全文搜索”。

+   `INNODB_FT_DELETED`：存储从 `InnoDB` 表的 `FULLTEXT` 索引中删除的行。为了避免在 `InnoDB` `FULLTEXT` 索引的 DML 操作期间进行昂贵的索引重组，新删除的单词信息被单独存储，当进行文本搜索时会从搜索结果中过滤掉，并且仅当对 `InnoDB` 表发出 `OPTIMIZE TABLE` 语句时才从主搜索索引中��除。

+   `INNODB_FT_DEFAULT_STOPWORD`：保存在创建 `InnoDB` 表的 `FULLTEXT` 索引时默认使用的 停用词 列表。

    有关`INNODB_FT_DEFAULT_STOPWORD`表的信息，请参阅 Section 14.9.4, “全文停用词”。

+   `INNODB_FT_INDEX_TABLE`：提供有关用于处理对`InnoDB`表的`FULLTEXT`索引进行文本搜索的倒排索引的信息。

+   `INNODB_FT_INDEX_CACHE`：提供有关`FULLTEXT`索引中新插入行的标记信息。为避免在 DML 操作期间进行昂贵的索引重组，新索引单词的信息被单独存储，并仅在运行`OPTIMIZE TABLE`时，服务器关闭时，或者缓存大小超过由`innodb_ft_cache_size`或`innodb_ft_total_cache_size`系统变量定义的限制时，才与主搜索索引合并。

注意

除了`INNODB_FT_DEFAULT_STOPWORD`表外，这些表最初是空的。在查询任何这些表之前，将`innodb_ft_aux_table`系统变量的值设置为包含`FULLTEXT`索引的表的名称（包括数据库名称）（例如，`test/articles`）。

**示例 17.5 InnoDB FULLTEXT 索引 INFORMATION_SCHEMA 表**

本示例使用具有`FULLTEXT`索引的表来演示`FULLTEXT`索引`INFORMATION_SCHEMA`表中包含的数据。

1.  创建一个具有`FULLTEXT`索引的表并插入一些数据：

    ```sql
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
    ```

1.  将`innodb_ft_aux_table`变量设置为具有`FULLTEXT`索引的表的名称。如果未设置此变量，则`InnoDB` `FULLTEXT` `INFORMATION_SCHEMA`表为空，除了`INNODB_FT_DEFAULT_STOPWORD`。

    ```sql
    mysql> SET GLOBAL innodb_ft_aux_table = 'test/articles';
    ```

1.  查询`INNODB_FT_INDEX_CACHE`表，显示`FULLTEXT`索引中新插入行的信息。为避免在 DML 操作期间进行昂贵的索引重组，新插入行的数据仍保留在`FULLTEXT`索引缓存中，直到运行`OPTIMIZE TABLE`（或者直到服务器关闭或超过缓存限制）。

    ```sql
    mysql> SELECT * FROM INFORMATION_SCHEMA.INNODB_FT_INDEX_CACHE LIMIT 5;
    +------------+--------------+-------------+-----------+--------+----------+
    | WORD       | FIRST_DOC_ID | LAST_DOC_ID | DOC_COUNT | DOC_ID | POSITION |
    +------------+--------------+-------------+-----------+--------+----------+
    | 1001       |            5 |           5 |         1 |      5 |        0 |
    | after      |            3 |           3 |         1 |      3 |       22 |
    | comparison |            6 |           6 |         1 |      6 |       44 |
    | configured |            7 |           7 |         1 |      7 |       20 |
    | database   |            2 |           6 |         2 |      2 |       31 |
    +------------+--------------+-------------+-----------+--------+----------+
    ```

1.  启用 `innodb_optimize_fulltext_only` 系统变量，并在包含 `FULLTEXT` 索引的表上运行 `OPTIMIZE TABLE`。此操作将 `FULLTEXT` 索引缓存的内容刷新到主 `FULLTEXT` 索引中。`innodb_optimize_fulltext_only` 改变了 `OPTIMIZE TABLE` 语句在 `InnoDB` 表上的操作方式，并且旨在在具有 `FULLTEXT` 索引的 `InnoDB` 表上的维护操作期间临时启用。

    ```sql
    mysql> SET GLOBAL innodb_optimize_fulltext_only=ON;

    mysql> OPTIMIZE TABLE articles;
    +---------------+----------+----------+----------+
    | Table         | Op       | Msg_type | Msg_text |
    +---------------+----------+----------+----------+
    | test.articles | optimize | status   | OK       |
    +---------------+----------+----------+----------+
    ```

1.  查询 `INNODB_FT_INDEX_TABLE` 表，查看主 `FULLTEXT` 索引中的数据信息，包括刚刚从 `FULLTEXT` 索引缓存中刷新的数据信息。

    ```sql
    mysql> SELECT * FROM INFORMATION_SCHEMA.INNODB_FT_INDEX_TABLE LIMIT 5;
    +------------+--------------+-------------+-----------+--------+----------+
    | WORD       | FIRST_DOC_ID | LAST_DOC_ID | DOC_COUNT | DOC_ID | POSITION |
    +------------+--------------+-------------+-----------+--------+----------+
    | 1001       |            5 |           5 |         1 |      5 |        0 |
    | after      |            3 |           3 |         1 |      3 |       22 |
    | comparison |            6 |           6 |         1 |      6 |       44 |
    | configured |            7 |           7 |         1 |      7 |       20 |
    | database   |            2 |           6 |         2 |      2 |       31 |
    +------------+--------------+-------------+-----------+--------+----------+
    ```

    `INNODB_FT_INDEX_CACHE` 表现在为空，因为 `OPTIMIZE TABLE` 操作刷新了 `FULLTEXT` 索引缓存。

    ```sql
    mysql> SELECT * FROM INFORMATION_SCHEMA.INNODB_FT_INDEX_CACHE LIMIT 5;
    Empty set (0.00 sec)
    ```

1.  从 `test/articles` 表中删除一些记录。

    ```sql
    mysql> DELETE FROM test.articles WHERE id < 4;
    ```

1.  查询 `INNODB_FT_DELETED` 表。该表记录从 `FULLTEXT` 索引中删除的行。为了避免在 DML 操作期间进行昂贵的索引重组，新删除记录的信息被单独存储，当进行文本搜索时从搜索结果中过滤掉，并在运行 `OPTIMIZE TABLE` 时从主搜索索引中删除。

    ```sql
    mysql> SELECT * FROM INFORMATION_SCHEMA.INNODB_FT_DELETED;
    +--------+
    | DOC_ID |
    +--------+
    |      2 |
    |      3 |
    |      4 |
    +--------+
    ```

1.  运行 `OPTIMIZE TABLE` 来删除已删除的记录。

    ```sql
    mysql> OPTIMIZE TABLE articles;
    +---------------+----------+----------+----------+
    | Table         | Op       | Msg_type | Msg_text |
    +---------------+----------+----------+----------+
    | test.articles | optimize | status   | OK       |
    +---------------+----------+----------+----------+
    ```

    `INNODB_FT_DELETED` 表现在应该为空。

    ```sql
    mysql> SELECT * FROM INFORMATION_SCHEMA.INNODB_FT_DELETED;
    Empty set (0.00 sec)
    ```

1.  查询 `INNODB_FT_CONFIG` 表。该表包含关于 `FULLTEXT` 索引和相关处理的元数据：

    +   `optimize_checkpoint_limit`: 多少秒后运行 `OPTIMIZE TABLE` 停止。

    +   `synced_doc_id`: 下一个要发行的 `DOC_ID`。

    +   `stopword_table_name`: 用户定义的停用词表的 *`database/table`* 名称。如果没有用户定义的停用词表，则 `VALUE` 列为空。

    +   `use_stopword`: 指示是否使用停用词表，该表在创建 `FULLTEXT` 索引时定义。

    ```sql
    mysql> SELECT * FROM INFORMATION_SCHEMA.INNODB_FT_CONFIG;
    +---------------------------+-------+
    | KEY                       | VALUE |
    +---------------------------+-------+
    | optimize_checkpoint_limit | 180   |
    | synced_doc_id             | 8     |
    | stopword_table_name       |       |
    | use_stopword              | 1     |
    +---------------------------+-------+
    ```

1.  禁用`innodb_optimize_fulltext_only`，因为它只打算暂时启用：

    ```sql
    mysql> SET GLOBAL innodb_optimize_fulltext_only=OFF;
    ```
