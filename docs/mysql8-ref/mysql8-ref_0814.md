# 14.9.8 ngram 全文解析器

> 原文：[`dev.mysql.com/doc/refman/8.0/en/fulltext-search-ngram.html`](https://dev.mysql.com/doc/refman/8.0/en/fulltext-search-ngram.html)

内置的 MySQL 全文解析器使用单词之间的空格作为分隔符来确定单词的起始和结束位置，这在处理不使用单词分隔符的表意语言时存在限制。为了解决这个限制，MySQL 提供了一个支持中文、日文和韩文（CJK）的 ngram 全文解析器。ngram 全文解析器支持与`InnoDB`和`MyISAM`一起使用。

注意

MySQL 还为日语提供了一个 MeCab 全文解析器插件，将文档标记为有意义的单词。有关更多信息，请参见第 14.9.9 节，“MeCab 全文解析器插件”。

ngram 是从给定文本序列中的连续*`n`*个字符序列。ngram 解析器将文本序列标记为连续的*`n`*个字符序列。例如，您可以使用 ngram 全文解析器为不同的*`n`*值对“abcd”进行标记。

```sql
n=1: 'a', 'b', 'c', 'd'
n=2: 'ab', 'bc', 'cd'
n=3: 'abc', 'bcd'
n=4: 'abcd'
```

ngram 全文解析器是一个内置的服务器插件。与其他内置的服务器插件一样，在服务器启动时会自动加载。

描述在第 14.9 节，“全文搜索函数”中的全文搜索语法适用于 ngram 解析器插件。本节描述了解析行为的差异。除了最小和最大单词长度选项（`innodb_ft_min_token_size`，`innodb_ft_max_token_size`，`ft_min_word_len`，`ft_max_word_len`)之外，也适用于与全文搜索相关的配置选项。

#### 配置 ngram Token 大小

ngram 解析器具有默认的 ngram token 大小为 2（bigram）。例如，使用大小为 2 的 token，ngram 解析器将字符串“abc def”解析为四个 token：“ab”，“bc”，“de”和“ef”。

ngram token 大小可通过`ngram_token_size`配置选项进行配置，最小值为 1，最大值为 10。

通常，`ngram_token_size` 被设置为您想要搜索的最大标记的大小。如果您只打算搜索单个字符，请将 `ngram_token_size` 设置为 1。较小的标记大小会产生较小的全文搜索索引，并且搜索速度更快。如果您需要搜索由多个字符组成的单词，请相应地设置 `ngram_token_size`。例如，“生日快乐”在简体中文中是“Happy Birthday”，其中“生日”是“birthday”，“快乐”翻译为“happy”。要搜索这样的两个字符单词，将 `ngram_token_size` 设置为 2 或更高的值。

作为只读变量，`ngram_token_size` 只能作为启动字符串的一部分或在配置文件中设置：

+   启动字符串：

    ```sql
    mysqld --ngram_token_size=2
    ```

+   配置文件：

    ```sql
    [mysqld]
    ngram_token_size=2
    ```

注意

对于使用 ngram 解析器的 `FULLTEXT` 索引，以下最小和最大单词长度配置选项将被忽略：`innodb_ft_min_token_size`, `innodb_ft_max_token_size`, `ft_min_word_len`, 和 `ft_max_word_len`。

#### 创建使用 ngram 解析器的 FULLTEXT 索引

要创建使用 ngram 解析器的 `FULLTEXT` 索引，请在 `CREATE TABLE`, `ALTER TABLE`, 或 `CREATE INDEX` 中指定 `WITH PARSER ngram`。

以下示例演示了创建具有 `ngram` `FULLTEXT` 索引的表，插入示例数据（简体中文文本）以及在信息模式 `INNODB_FT_INDEX_CACHE` 表中查看标记化数据。

```sql
mysql> USE test;

mysql> CREATE TABLE articles (
      id INT UNSIGNED AUTO_INCREMENT NOT NULL PRIMARY KEY,
      title VARCHAR(200),
      body TEXT,
      FULLTEXT (title,body) WITH PARSER ngram
    ) ENGINE=InnoDB CHARACTER SET utf8mb4;

mysql> SET NAMES utf8mb4;

INSERT INTO articles (title,body) VALUES
    ('数据库管理','在本教程中我将向你展示如何管理数据库'),
    ('数据库应用开发','学习开发数据库应用程序');

mysql> SET GLOBAL innodb_ft_aux_table="test/articles";

mysql> SELECT * FROM INFORMATION_SCHEMA.INNODB_FT_INDEX_CACHE ORDER BY doc_id, position;
```

要向现有表添加 `FULLTEXT` 索引，可以使用 `ALTER TABLE` 或 `CREATE INDEX`。例如：

```sql
CREATE TABLE articles (
      id INT UNSIGNED AUTO_INCREMENT NOT NULL PRIMARY KEY,
      title VARCHAR(200),
      body TEXT
     ) ENGINE=InnoDB CHARACTER SET utf8mb4;

ALTER TABLE articles ADD FULLTEXT INDEX ft_index (title,body) WITH PARSER ngram;

# Or:

CREATE FULLTEXT INDEX ft_index ON articles (title,body) WITH PARSER ngram;
```

#### ngram 解析器空格处理

ngram 解析器在解析时消除空格。例如：

+   “ab cd” 被解析为 “ab”, “cd”

+   “a bc” 被解析为 “bc”

#### ngram 解析器停用词处理

内置的 MySQL 全文解析器将单词与停用词列表中的条目进行比较。如果一个词等于停用词列表中的条目，则该词将从索引中排除。对于 ngram 解析器，停用词处理方式不同。它不是排除等于停用词列表中条目的标记，而是排除*包含*停用词的标记。例如，假设`ngram_token_size=2`，包含“a,b”的文档被解析为“a,”和“,b”。如果逗号（“,”）被定义为停用词，那么“a,”和“,b”都将因为包含逗号而被排除在索引之外。

默认情况下，ngram 解析器使用默认停用词列表，其中包含一组英语停用词。对于适用于中文、日文或韩文的停用词列表，您必须创建自己的停用词列表。有关创建停用词列表的信息，请参见第 14.9.4 节，“全文停用词”。

长度大于`ngram_token_size`的停用词将被忽略。

#### ngram 解析器术语搜索

对于*自然语言模式*搜索，搜索词被转换为 ngram 词的并集。例如，字符串“abc”（假设`ngram_token_size=2`）被转换为“ab bc”。给定两个文档，一个包含“ab”，另一个包含“abc”，搜索词“ab bc”匹配两个文档。

对于*布尔模式搜索*，搜索词被转换为 ngram 短语搜索。例如，字符串'abc'（假设`ngram_token_size=2`）被转换为'“ab bc”'。给定两个文档，一个包含'ab'，另一个包含'abc'，搜索短语'“ab bc”'仅匹配包含'abc'的文档。

#### ngram 解析器通配符搜索

因为 ngram `FULLTEXT`索引仅包含 ngrams，并不包含有关术语开头的信息，通配符搜索可能会返回意外结果。以下行为适用于使用 ngram `FULLTEXT`搜索索引进行通配符搜索的情况：

+   如果通配符搜索的前缀项短于 ngram 标记大小，查询将返回所有包含以前缀项开头的 ngram 标记的索引行。例如，假设`ngram_token_size=2`，搜索“a*”将返回所有以“a”开头的行。

+   如果通配符搜索的前缀项长于 ngram 标记大小，则前缀项被转换为 ngram 短语，通配符操作符被忽略。例如，假设`ngram_token_size=2`，一个“abc*”通配符搜索被转换为“ab bc”。

#### ngram 解析器短语搜索

短语搜索被转换为 ngram 短语搜索。例如，搜索短语“abc”被转换为“ab bc”，返回包含“abc”和“ab bc”的文档。

搜索短语“abc def”被转换为“ab bc de ef”，返回包含“abc def”和“ab bc de ef”的文档。不返回包含“abcdef”的文档。
