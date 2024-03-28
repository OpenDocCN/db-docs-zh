# 14.9.1 自然语言全文搜索

> 原文：[`dev.mysql.com/doc/refman/8.0/en/fulltext-natural-language.html`](https://dev.mysql.com/doc/refman/8.0/en/fulltext-natural-language.html)

默认情况下或使用`IN NATURAL LANGUAGE MODE`修饰符时，`MATCH()`函数针对文本集合执行自然语言搜索。集合是包含在`FULLTEXT`索引中的一个或多个列的集合。搜索字符串作为参数传递给`AGAINST()`。对于表中的每一行，`MATCH()`返回一个相关性值；即，搜索字符串与列中文本之间的相似度量，在`MATCH()`列表中命名的列中。

```sql
mysql> CREATE TABLE articles (
 ->   id INT UNSIGNED AUTO_INCREMENT NOT NULL PRIMARY KEY,
 ->   title VARCHAR(200),
 ->   body TEXT,
 ->   FULLTEXT (title,body)
 -> ) ENGINE=InnoDB;
Query OK, 0 rows affected (0.08 sec)

mysql> INSERT INTO articles (title,body) VALUES
 ->   ('MySQL Tutorial','DBMS stands for DataBase ...'),
 ->   ('How To Use MySQL Well','After you went through a ...'),
 ->   ('Optimizing MySQL','In this tutorial, we show ...'),
 ->   ('1001 MySQL Tricks','1\. Never run mysqld as root. 2\. ...'),
 ->   ('MySQL vs. YourSQL','In the following database comparison ...'),
 ->   ('MySQL Security','When configured properly, MySQL ...');
Query OK, 6 rows affected (0.01 sec)
Records: 6  Duplicates: 0  Warnings: 0

mysql> SELECT * FROM articles
 -> WHERE MATCH (title,body)
 -> AGAINST ('database' IN NATURAL LANGUAGE MODE);
+----+-------------------+------------------------------------------+
| id | title             | body                                     |
+----+-------------------+------------------------------------------+
|  1 | MySQL Tutorial    | DBMS stands for DataBase ...             |
|  5 | MySQL vs. YourSQL | In the following database comparison ... |
+----+-------------------+------------------------------------------+
2 rows in set (0.00 sec)
```

默认情况下，搜索是以不区分大小写的方式执行的。要执行区分大小写的全文搜索，请为索引列使用区分大小写或二进制排序规则。例如，使用`utf8mb4`字符集的列可以分配`utf8mb4_0900_as_cs`或`utf8mb4_bin`排序规则，以使其对全文搜索区分大小写。

当`MATCH()`在`WHERE`子句中使用时，如前面示例所示，只要满足以下条件，返回的行将自动按相关性最高的顺序排序：

+   不得有显式的`ORDER BY`子句。

+   搜索必须使用全文索引扫描，而不是表扫描。

+   如果查询涉及表连接，则全文索引扫描必须是连接中最左边的非常量表。

鉴于刚刚列出的条件，当需要或希望时，通常更容易通过使用`ORDER BY`指定显式排序顺序。

相关性值是非负浮点数。零相关性表示没有相似性。相关性是基于行中的单词数、行中唯一单词数、集合中的总单词数以及包含特定单词的行数进行计算的。

注意

“文档”一词可以与“行”一词互换使用，两个术语都指的是行的索引部分。“集合”一词指的是索引列，并包括所有行。

要简单计算匹配项，可以使用如下查询：

```sql
mysql> SELECT COUNT(*) FROM articles
 -> WHERE MATCH (title,body)
 -> AGAINST ('database' IN NATURAL LANGUAGE MODE);
+----------+
| COUNT(*) |
+----------+
|        2 |
+----------+
1 row in set (0.00 sec)
```

你可能会发现将查询重写为以下形式更快：

```sql
mysql> SELECT
 -> COUNT(IF(MATCH (title,body) AGAINST ('database' IN NATURAL LANGUAGE MODE), 1, NULL))
 -> AS count
 -> FROM articles;
+-------+
| count |
+-------+
|     2 |
+-------+
1 row in set (0.03 sec)
```

第一个查询做了一些额外的工作（按相关性对结果进行排序），但也可以根据`WHERE`子句使用索引查找。如果搜索匹配的行数较少，索引查找可能会使第一个查询更快。第二个查询执行全表扫描，如果搜索词在大多数行中存在，可能比索引查找更快。

对于自然语言全文搜索，`MATCH()`函数中命名的列必须与表中某个`FULLTEXT`索引中包含的相同列相同。对于上述查询，请注意`MATCH()`函数中命名的列（`title`和`body`）与`article`表的`FULLTEXT`索引定义中命名的列相同。要分别搜索`title`或`body`，您需要为每个列创建单独的`FULLTEXT`索引。

你还可以执行布尔搜索或带有查询扩展的搜索。这些搜索类型在第 14.9.2 节，“布尔全文搜索”和第 14.9.3 节，“带有查询扩展的全文搜索”中有描述。

使用索引的全文搜索在`MATCH()`子句中只能命名来自单个表的列，因为索引不能跨越多个表。对于`MyISAM`表，如果没有索引，可以执行布尔搜索（尽管速度较慢），在这种情况下，可以命名来自多个表的列。

上面的示例是一个基本示例，展示了如何在返回的行按相关性递减的顺序中使用`MATCH()`函数。下一个示例展示了如何显式检索相关性值。返回的行没有排序，因为`SELECT`语句中既没有`WHERE`也没有`ORDER BY`子句：

```sql
mysql> SELECT id, MATCH (title,body)
 -> AGAINST ('Tutorial' IN NATURAL LANGUAGE MODE) AS score
 -> FROM articles;
+----+---------------------+
| id | score               |
+----+---------------------+
|  1 | 0.22764469683170319 |
|  2 |                   0 |
|  3 | 0.22764469683170319 |
|  4 |                   0 |
|  5 |                   0 |
|  6 |                   0 |
+----+---------------------+
6 rows in set (0.00 sec)
```

下面的示例更为复杂。该查询返回相关性值，并按相关性递减的顺序对行进行排序。为了实现这个结果，在`SELECT`列表中指定`MATCH()`两次：一次在`SELECT`列表中，一次在`WHERE`子句中。这不会增加额外开销，因为 MySQL 优化器注意到两次`MATCH()`调用是相同的，并且只调用一次全文搜索代码。

```sql
mysql> SELECT id, body, MATCH (title,body)
 ->   AGAINST ('Security implications of running MySQL as root'
 ->   IN NATURAL LANGUAGE MODE) AS score
 -> FROM articles
 ->   WHERE MATCH (title,body) 
 ->   AGAINST('Security implications of running MySQL as root'
 ->   IN NATURAL LANGUAGE MODE);
+----+-------------------------------------+-----------------+
| id | body                                | score           |
+----+-------------------------------------+-----------------+
|  4 | 1\. Never run mysqld as root. 2\. ... | 1.5219271183014 |
|  6 | When configured properly, MySQL ... | 1.3114095926285 |
+----+-------------------------------------+-----------------+
2 rows in set (0.00 sec)
```

用双引号(`"`)括起来的短语仅匹配包含该短语的行*文字，就像它被输入的那样*。全文引擎将短语拆分为单词，并在`FULLTEXT`索引中为这些单词执行搜索。非单词字符不需要完全匹配：短语搜索只需要匹配包含与短语完全相同的单词并且顺序相同的匹配项。例如，`"test phrase"`匹配`"test, phrase"`。如果短语不包含索引中的任何单词，则结果为空。例如，如果所有单词要么是停用词要么比索引单词的最小长度短，结果为空。

MySQL 的`FULLTEXT`实现将任何真实单词字符序列（字母，数字和下划线）视为一个单词。该序列也可以包含撇号（`'`），但不能连续超过一个。这意味着`aaa'bbb`被视为一个单词，但`aaa''bbb`被视为两个单词。单词开头或结尾的撇号将被`FULLTEXT`解析器去除；`'aaa'bbb'`会被解析为`aaa'bbb`。

内置的`FULLTEXT`解析器通过查找特定的分隔符字符来确定单词的起始和结束位置；例如，（空格），`,`（逗号）和`.`（句号）。如果单词没有被分隔符（例如，中文）分隔，内置的`FULLTEXT`解析器无法确定单词的起始或结束位置。为了能够将这些语言中的单词或其他索引术语添加到使用内置`FULLTEXT`解析器的`FULLTEXT`索引中，您必须预处理它们，以便它们被某种任意的分隔符分隔。或者，您可以使用 ngram 解析器插件（用于中文，日文或韩文）或 MeCab 解析器插件（用于日文）创建`FULLTEXT`索引。

可以编写一个插件来替换内置的全文解析器。有关详细信息，请参阅 MySQL 插件 API。例如解析器插件源代码，请查看 MySQL 源代码分发的`plugin/fulltext`目录。

在全文搜索中会忽略一些词：

+   任何太短的单词都会被忽略。通过全文搜索找到的单词的默认最小长度为`InnoDB`搜索索引为三个字符，或者`MyISAM`为四个字符。您可以通过在创建索引之前设置配置选项来控制截断：`InnoDB`搜索索引的`innodb_ft_min_token_size`配置选项，或者`MyISAM`的`ft_min_word_len`。

    注意

    这种行为不适用于使用 ngram 解析器的`FULLTEXT`索引。对于 ngram 解析器，标记长度由`ngram_token_size`选项定义。

+   停用词列表中的单词会被忽略。停用词是一些如“the”或“some”这样常见以至于被认为没有语义价值的单词。有一个内置的停用词列表，但可以被用户定义的列表覆盖。停用词列表和相关的配置选项对`InnoDB`搜索索引和`MyISAM`索引是不同的。停用词处理由配置选项`innodb_ft_enable_stopword`，`innodb_ft_server_stopword_table`和`innodb_ft_user_stopword_table`控制`InnoDB`搜索索引，以及`ft_stopword_file`控制`MyISAM`索引。

参见第 14.9.4 节，“全文停用词”查看默认停用词列表以及如何更改它们。默认最小单词长度可以按照第 14.9.6 节，“调整 MySQL 全文搜索”中描述的方式进行更改。

集合和查询中的每个正确单词根据其在集合或查询中的重要性进行加权。因此，出现在许多文档中的单词权重较低，因为在这个特定集合中它的语义价值较低。相反，如果单词很少见，它将获得更高的权重。单词的权重组合在一起计算行的相关性。这种技术在大型集合中效果最佳。

MyISAM 限制

对于非常小的表，单词分布不能充分反映它们的语义价值，这种模型有时可能会为`MyISAM`表上的搜索索引产生奇怪的结果。例如，尽管单词“MySQL”出现在先前显示的`articles`表的每一行中，但在`MyISAM`搜索索引中搜索该单词却没有结果：

```sql
mysql> SELECT * FROM articles
 -> WHERE MATCH (title,body)
 -> AGAINST ('MySQL' IN NATURAL LANGUAGE MODE);
Empty set (0.00 sec)
```

由于单词“MySQL”至少出现在 50%的行中，搜索结果为空，因此实际上被视为停用词。这种过滤技术更适用于大数据集，您可能不希望结果集从 1GB 表中返回每一秒的行，而不适用于小数据集，因为这可能导致热门术语的结果不佳。

当您首次尝试全文搜索以查看其工作原理时，50%的阈值可能会让您感到惊讶，并使`InnoDB`表更适合用于全文搜索的实验。如果您创建一个`MyISAM`表并只插入一两行文本，那么文本中的每个单词至少在 50%的行中出现。因此，在表包含更多行之前，任何搜索都不会返回任何结果。需要绕过 50%限制的用户可以在`InnoDB`表上构建搜索索引，或者使用第 14.9.2 节，“布尔全文搜索”中解释的布尔搜索模式。
