# 14.9.2 布尔全文搜索

> 原文：[`dev.mysql.com/doc/refman/8.0/en/fulltext-boolean.html`](https://dev.mysql.com/doc/refman/8.0/en/fulltext-boolean.html)

MySQL 可以使用`IN BOOLEAN MODE`修饰符执行布尔全文搜索。使用此修饰符，搜索字符串中的某些字符在单词开头或结尾具有特殊含义。在以下查询中，`+`和`-`运算符表示单词必须存在或不存在，才能进行匹配。因此，该查询检索包含单词“MySQL”但*不包含*单词“YourSQL”的所有行：

```sql
mysql> SELECT * FROM articles WHERE MATCH (title,body)
 -> AGAINST ('+MySQL -YourSQL' IN BOOLEAN MODE);
+----+-----------------------+-------------------------------------+
| id | title                 | body                                |
+----+-----------------------+-------------------------------------+
|  1 | MySQL Tutorial        | DBMS stands for DataBase ...        |
|  2 | How To Use MySQL Well | After you went through a ...        |
|  3 | Optimizing MySQL      | In this tutorial, we show ...       |
|  4 | 1001 MySQL Tricks     | 1\. Never run mysqld as root. 2\. ... |
|  6 | MySQL Security        | When configured properly, MySQL ... |
+----+-----------------------+-------------------------------------+
```

注意

在实现此功能时，MySQL 使用所谓的隐含布尔逻辑，其中

+   `+`代表`AND`

+   `-`代表`NOT`

+   [*无运算符*]表示`OR`

布尔全文搜索具有以下特点：

+   它们不会自动按相关性降序对行进行排序。

+   `InnoDB`表在执行布尔查询时需要在`MATCH()`表达式的所有列上创建`FULLTEXT`索引。对`MyISAM`搜索索引执行布尔查询即使没有`FULLTEXT`索引也可以工作，尽管以这种方式执行的搜索速度会非常慢。

+   最小和最大单词长度全文参数适用于使用内置`FULLTEXT`解析器和 MeCab 解析器插件创建的`FULLTEXT`索引。`innodb_ft_min_token_size`和`innodb_ft_max_token_size`用于`InnoDB`搜索索引。`ft_min_word_len`和`ft_max_word_len`用于`MyISAM`搜索索引。

    最小和最大单词长度全文参数不适用于使用 ngram 解析器创建的`FULLTEXT`索引。 ngram 标记大小由`ngram_token_size`选项定义。

+   停用词列表适用于`InnoDB`搜索索引，由`innodb_ft_enable_stopword`、`innodb_ft_server_stopword_table`和`innodb_ft_user_stopword_table`控制，以及`MyISAM`索引由`ft_stopword_file`控制。

+   `InnoDB`全文搜索不支持在单个搜索词上使用多个运算符，例如：`'++apple'`。在单个搜索词上使用多个运算符会返回一个语法错误到标准输出。MyISAM 全文搜索成功处理相同的搜索，忽略除了紧邻搜索词的运算符之外的所有运算符。

+   `InnoDB`全文搜索仅支持前导加号或减号。例如，`InnoDB`支持`'+apple'`，但不支持`'apple+'`。指定尾随加号或减号会导致`InnoDB`报告语法错误。

+   `InnoDB`全文搜索不支持在通配符（`'+*'`）、加减号组合（`'+-'`）或前导加减号组合（`'+-apple'`）中使用前导加号。这些无效查询会返回语法错误。

+   `InnoDB`全文搜索不支持在布尔全文搜索中使用`@`符号。`@`符号保留供`@distance`接近搜索运算符使用。

+   它们不使用适用于`MyISAM`搜索索引的 50%阈值。

布尔全文搜索功能支持以下运算符：

+   `+`

    前导或尾随加号表示该单词*必须*出现在返回的每一行中。`InnoDB`仅支持前导加号。

+   `-`

    前导或尾随减号表示这个词必须*不*出现在返回的任何行中。`InnoDB`仅支持前导减号。

    注意：`-`运算符仅用于排除其他搜索项匹配的行。因此，仅包含由`-`前导的术语的布尔模式搜索会返回空结果。它不会返回“除了包含任何被排除术语的行之外的所有行”。

+   （无运算符）

    默认情况下（未指定`+`或`-`时），该单词是可选的，但包含它的行会被评分更高。这模仿了`MATCH() AGAINST()`在没有`IN BOOLEAN MODE`修饰符的情况下的行为。

+   `@*`distance`*`

    此运算符仅适用于`InnoDB`表。它测试两个或更多单词是否都在指定距离内开始，距离以单词为单位测量。在`@*`distance`*`运算符之前的双引号字符串中指定搜索词，例如，`MATCH(col1) AGAINST('"word1 word2 word3" @8' IN BOOLEAN MODE)`

+   `> <`

    这两个运算符用于改变单词对分配给行的相关性值的贡献。`>`运算符增加贡献，`<`运算符减少贡献。请参见此列表后面的示例。

+   `( )`

    括号将单词分组为子表达式。括号组可以嵌套。

+   `~`

    前导波浪号充当否定运算符，导致单词对行的相关性的贡献为负值。这对标记“噪音”词很有用。包含这样一个词的行评分低于其他行，但不会完全被排除，就像使用`-`运算符一样。

+   `*`

    星号用作截断（或通配符）运算符。与其他运算符不同，它*附加*到要受影响的单词之后。如果单词以`*`运算符之前的单词开头，则匹配。

    如果使用截断运算符指定了一个单词，则即使它太短或是停用词，也不会从布尔查询中删除。一个单词是否太短是根据`InnoDB`表的`innodb_ft_min_token_size`设置，或`MyISAM`表的`ft_min_word_len`来确定的。这些选项不适用于使用 ngram 解析器的`FULLTEXT`索引。

    通配符单词被视为必须出现在一个或多个单词的开头的前缀。如果最小单词长度为 4，搜索`'+*`word`* +the*'`可能返回的行比搜索`'+*`word`* +the'`更少，因为第二个查询忽略了太短的搜索词`the`。

+   `"`

    用双引号(`"`)括起来的短语仅匹配包含该短语的行*文字，就像输入的那样*。全文引擎将短语拆分为单词，并在`FULLTEXT`索引中搜索这些单词。非单词字符不需要完全匹配：短语搜索只要求匹配包含与短语完全相同的单词且顺序相同的内容。例如，`"test phrase"`匹配`"test, phrase"`。

    如果短语不包含索引中的任何单词，则结果为空。这些单词可能不在索引中，因为存在多种因素的组合：如果它们不存在于文本中，是停用词，或者比索引单词的最小长度更短。

以下示例演示了一些使用布尔全文搜索运算符的搜索字符串：

+   `'apple banana'`

    查找包含两个词中至少一个的行。

+   `'+apple +juice'`

    查找同时包含两个单词的行。

+   `'+apple macintosh'`

    查找包含单词“apple”的行，但如果它们还包含“macintosh”，则将其排名提高。

+   `'+apple -macintosh'`

    查找包含单词“apple”但不包含“macintosh”的行。

+   `'+apple ~macintosh'`

    查找包含单词“apple”的行，但如果该行还包含单词“macintosh”，则将其评级低于不包含该词的行。这比搜索`'+apple -macintosh'`“更柔和”，因为“macintosh”的存在会导致该行根本不返回。

+   `'+apple +(>turnover <strudel)'`

    查找包含单词“apple”和“turnover”，或“apple”和“strudel”（顺序不限），但将“apple turnover”排名高于“apple strudel”。

+   `'apple*'`

    查找包含诸如“apple”、“apples”、“applesauce”或“applet”等单词的行。

+   `'"some words"'`

    查找包含确切短语“some words”的行（例如，包含“some words of wisdom”但不包含“some noise words”的行）。请注意，包围短语的`"`字符是界定短语的操作符字符。它们不是包围搜索字符串本身的引号。

#### InnoDB 布尔模式搜索的相关性排名

`InnoDB`全文搜索是基于[Sphinx](http://sphinxsearch.com/)全文搜索引擎建模的，所使用的算法基于[BM25](http://en.wikipedia.org/wiki/Okapi_BM25)和[TF-IDF](http://en.wikipedia.org/wiki/TF-IDF)排名算法。因此，`InnoDB`布尔全文搜索的相关性排名可能与`MyISAM`的相关性排名不同。

`InnoDB`使用“词项频率-逆文档频率”（`TF-IDF`）加权系统的变体来为给定的全文搜索查询对文档的相关性进行排名。`TF-IDF`加权是基于一个词在文档中出现的频率，减去该词在整个文档集合中出现的频率。换句话说，一个词在文档中出现的频率越高，而在文档集合中出现的频率越低，文档的排名就越高。

##### 如何计算相关性排名

词项频率（`TF`）值是一个词在文档中出现的次数。一个词的逆文档频率（`IDF`）值是使用以下公式计算的，其中`total_records`是集合中的记录数，`matching_records`是搜索词出现在的记录数。

```sql
${IDF} = log10( ${total_records} / ${matching_records} )
```

当一个文档包含多次出现的单词时，IDF 值将乘以 TF 值：

```sql
${TF} * ${IDF}
```

使用`TF`和`IDF`值，文档的相关性排名是使用以下公式计算的：

```sql
${rank} = ${TF} * ${IDF} * ${IDF}
```

公式在以下示例中进行演示。

##### 单词搜索的相关性排名

本示例演示了单词搜索的相关性排名计算。

```sql
mysql> CREATE TABLE articles (
 ->   id INT UNSIGNED AUTO_INCREMENT NOT NULL PRIMARY KEY,
 ->   title VARCHAR(200),
 ->   body TEXT,
 ->   FULLTEXT (title,body)
 ->)  ENGINE=InnoDB;
Query OK, 0 rows affected (1.04 sec)

mysql> INSERT INTO articles (title,body) VALUES
 ->   ('MySQL Tutorial','This database tutorial ...'),
 ->   ("How To Use MySQL",'After you went through a ...'),
 ->   ('Optimizing Your Database','In this database tutorial ...'),
 ->   ('MySQL vs. YourSQL','When comparing databases ...'),
 ->   ('MySQL Security','When configured properly, MySQL ...'),
 ->   ('Database, Database, Database','database database database'),
 ->   ('1001 MySQL Tricks','1\. Never run mysqld as root. 2\. ...'),
 ->   ('MySQL Full-Text Indexes', 'MySQL fulltext indexes use a ..');
Query OK, 8 rows affected (0.06 sec)
Records: 8  Duplicates: 0  Warnings: 0

mysql> SELECT id, title, body, 
 ->   MATCH (title,body) AGAINST ('database' IN BOOLEAN MODE) AS score
 ->   FROM articles ORDER BY score DESC;
+----+------------------------------+-------------------------------------+---------------------+
| id | title                        | body                                | score               |
+----+------------------------------+-------------------------------------+---------------------+
|  6 | Database, Database, Database | database database database          |  1.0886961221694946 |
|  3 | Optimizing Your Database     | In this database tutorial ...       | 0.36289870738983154 |
|  1 | MySQL Tutorial               | This database tutorial ...          | 0.18144935369491577 |
|  2 | How To Use MySQL             | After you went through a ...        |                   0 |
|  4 | MySQL vs. YourSQL            | When comparing databases ...        |                   0 |
|  5 | MySQL Security               | When configured properly, MySQL ... |                   0 |
|  7 | 1001 MySQL Tricks            | 1\. Never run mysqld as root. 2\. ... |                   0 |
|  8 | MySQL Full-Text Indexes      | MySQL fulltext indexes use a ..     |                   0 |
+----+------------------------------+-------------------------------------+---------------------+
8 rows in set (0.00 sec)
```

总共有 8 条记录，其中有 3 条匹配“database”搜索词。第一条记录（`id 6`）包含搜索词 6 次，相关性排名为`1.0886961221694946`。此排名值是使用 TF 值为 6（“database”搜索词在记录`id 6`中出现 6 次）和 IDF 值为 0.42596873216370745 计算的，计算如下（其中 8 是总记录数，3 是搜索词出现在的记录数）：

```sql
${IDF} = LOG10( 8 / 3 ) = 0.42596873216370745
```

然后将`TF`和`IDF`值输入到排名公式中：

```sql
${rank} = ${TF} * ${IDF} * ${IDF}
```

在 MySQL 命令行客户端中进行计算返回一个排名值为 1.088696164686938。

```sql
mysql> SELECT 6*LOG10(8/3)*LOG10(8/3);
+-------------------------+
| 6*LOG10(8/3)*LOG10(8/3) |
+-------------------------+
|       1.088696164686938 |
+-------------------------+
1 row in set (0.00 sec)
```

注意

您可能会注意到`SELECT ... MATCH ... AGAINST`语句返回的排名值与 MySQL 命令行客户端返回的排名值之间存在轻微差异（`1.0886961221694946`与`1.088696164686938`）。这种差异是由于`InnoDB`内部执行整数和浮点数/双精度数之间的转换（以及相关的精度和舍入决策），以及在其他地方执行这些转换的方式，比如在 MySQL 命令行客户端或其他类型的计算器中。 

##### 多词搜索的相关性排名

本示例演示了基于`articles`表和前面示例中使用的数据进行多词全文搜索的相关性排名计算。

如果您搜索超过一个词，相关性排名值是每个词的相关性排名值的总和，如下所示：

```sql
${rank} = ${TF} * ${IDF} * ${IDF} + ${TF} * ${IDF} * ${IDF}
```

对两个术语（'mysql 教程'）进行搜索返回以下结果：

```sql
mysql> SELECT id, title, body, MATCH (title,body)  
 ->   AGAINST ('mysql tutorial' IN BOOLEAN MODE) AS score
 ->   FROM articles ORDER BY score DESC;
+----+------------------------------+-------------------------------------+----------------------+
| id | title                        | body                                | score                |
+----+------------------------------+-------------------------------------+----------------------+
|  1 | MySQL Tutorial               | This database tutorial ...          |   0.7405621409416199 |
|  3 | Optimizing Your Database     | In this database tutorial ...       |   0.3624762296676636 |
|  5 | MySQL Security               | When configured properly, MySQL ... | 0.031219376251101494 |
|  8 | MySQL Full-Text Indexes      | MySQL fulltext indexes use a ..     | 0.031219376251101494 |
|  2 | How To Use MySQL             | After you went through a ...        | 0.015609688125550747 |
|  4 | MySQL vs. YourSQL            | When comparing databases ...        | 0.015609688125550747 |
|  7 | 1001 MySQL Tricks            | 1\. Never run mysqld as root. 2\. ... | 0.015609688125550747 |
|  6 | Database, Database, Database | database database database          |                    0 |
+----+------------------------------+-------------------------------------+----------------------+
8 rows in set (0.00 sec)
```

在第一条记录（`id 8`）中，'mysql'出现一次，'tutorial'出现两次。有六条匹配记录为'mysql'，两条匹配记录为'tutorial'。当将这些值插入到多词搜索的排名公式中时，MySQL 命令行客户端返回了预期的排名值：

```sql
mysql> SELECT (1*log10(8/6)*log10(8/6)) + (2*log10(8/2)*log10(8/2));
+-------------------------------------------------------+
| (1*log10(8/6)*log10(8/6)) + (2*log10(8/2)*log10(8/2)) |
+-------------------------------------------------------+
|                                    0.7405621541938003 |
+-------------------------------------------------------+
1 row in set (0.00 sec)
```

注意

`SELECT ... MATCH ... AGAINST`语句和 MySQL 命令行客户端返回的排名值之间的轻微差异在前面的示例中有解释。
