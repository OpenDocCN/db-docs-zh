# 14.9 全文搜索函数

> 原文：[`dev.mysql.com/doc/refman/8.0/en/fulltext-search.html`](https://dev.mysql.com/doc/refman/8.0/en/fulltext-search.html)

14.9.1 自然语言全文搜索

14.9.2 布尔全文搜索

14.9.3 带有查询扩展的全文搜索

14.9.4 全文搜索停用词

14.9.5 全文搜索限制

14.9.6 调整 MySQL 全文搜索

14.9.7 为全文索引添加用户定义的排序规则

14.9.8 ngram 全文解析器

14.9.9 MeCab 全文解析器插件

[`MATCH (*`col1`*,*`col2`*,...) AGAINST (*`expr`* [*`search_modifier`*])`](fulltext-search.html#function_match)

```sql
*search_modifier:*
  {
       IN NATURAL LANGUAGE MODE
     | IN NATURAL LANGUAGE MODE WITH QUERY EXPANSION
     | IN BOOLEAN MODE
     | WITH QUERY EXPANSION
  }
```

MySQL 支持全文索引和搜索：

+   MySQL 中的全文索引是`FULLTEXT`类型的索引。

+   全文索引只能用于`InnoDB`或`MyISAM`表，并且只能为`CHAR`、`VARCHAR`或`TEXT`列创建。

+   MySQL 提供了内置的全文 ngram 解析器，支持中文、日文和韩文（CJK），以及用于日文的可安装的 MeCab 全文解析器插件。解析差异在第 14.9.8 章“ngram 全文解析器”和第 14.9.9 章“MeCab 全文解析器插件”中有详细说明。

+   在创建表时，可以在`CREATE TABLE`语句中给出`FULLTEXT`索引定义，或者稍后使用`ALTER TABLE`或`CREATE INDEX`添加。

+   对于大型数据集，将数据加载到没有`FULLTEXT`索引的表中，然后在此之后创建索引，比将数据加载到已有`FULLTEXT`索引的表中要快得多。

使用`MATCH() AGAINST()`语法执行全文搜索。`MATCH()`接受一个逗号分隔的列表，列出要搜索的列。`AGAINST`接受要搜索的字符串，以及一个可选的修饰符，指示要执行的搜索类型。搜索字符串必须是在查询评估期间保持不变的字符串值。例如，表列就不符合这个规则，因为每行可能不同。

以前，MySQL 允许在`MATCH()`中使用一个 rollup 列，但使用这种结构的查询性能不佳且结果不可靠。（这是因为`MATCH()`不是根据其参数的函数实现，而是根据基表的底层扫描中当前行的行 ID 的函数实现。）从 MySQL 8.0.28 开始，MySQL 不再允许这样的查询；更具体地说，任何符合此处列出的所有标准的查询都将被拒绝，并显示`ER_FULLTEXT_WITH_ROLLUP`：

+   `MATCH()`出现在查询块的`SELECT`列表、`GROUP BY`子句、`HAVING`子句或`ORDER BY`子句中。

+   查询块包含一个`GROUP BY ... WITH ROLLUP`子句。

+   调用`MATCH()`函数的参数是分组列之一。

这里显示了一些此类查询的示例：

```sql
# MATCH() in SELECT list...
SELECT MATCH (a) AGAINST ('abc') FROM t GROUP BY a WITH ROLLUP;
SELECT 1 FROM t GROUP BY a, MATCH (a) AGAINST ('abc') WITH ROLLUP;

# ...in HAVING clause...
SELECT 1 FROM t GROUP BY a WITH ROLLUP HAVING MATCH (a) AGAINST ('abc');

# ...and in ORDER BY clause
SELECT 1 FROM t GROUP BY a WITH ROLLUP ORDER BY MATCH (a) AGAINST ('abc');
```

在`WHERE`子句中允许使用带有 rollup 列的`MATCH()`。

有三种类型的全文搜索：

+   自然语言搜索将搜索字符串解释为自然人类语言中的短语（自由文本中的短语）。除了双引号（"）字符外，没有特殊运算符。停用词列表适用。有关停用词列表的更多信息，请参见第 14.9.4 节，“全文停用词”。

    如果给定`IN NATURAL LANGUAGE MODE`修饰符或未给定修饰符，则全文搜索是自然语言搜索。有关更多信息，请参见第 14.9.1 节，“自然语言全文搜索”。

+   布尔搜索使用特殊查询语言的规则解释搜索字符串。字符串包含要搜索的单词。它还可以包含指定要求的运算符，例如一个单词必须存在或不存在于匹配行中，或者它应该比通常更高或更低权重。某些常见单词（停用词）被省略在搜索索引中，如果在搜索字符串中存在则不匹配。`IN BOOLEAN MODE`修饰符指定布尔搜索。有关更多信息，请参见第 14.9.2 节，“布尔全文搜索”。

+   查询扩展搜索是自然语言搜索的修改。搜索字符串用于执行自然语言搜索。然后从搜索返回的最相关行中添加单词到搜索字符串，并再次进行搜索。查询返回第二次搜索的行。`IN NATURAL LANGUAGE MODE WITH QUERY EXPANSION`或`WITH QUERY EXPANSION`修饰符指定查询扩展搜索。有关更多信息，请参见第 14.9.3 节，“带查询扩展的全文搜索”。

有关`FULLTEXT`查询性能的信息，请参见 Section 10.3.5, “Column Indexes”。

关于`InnoDB` `FULLTEXT`索引的更多信息，请参见 Section 17.6.2.4, “InnoDB Full-Text Indexes”。

全文搜索的限制条件列在 Section 14.9.5, “Full-Text Restrictions”。

**myisam_ftdump**实用程序会转储`MyISAM`全文索引的内容。这对于调试全文查询可能会有所帮助。请参见 Section 6.6.3, “myisam_ftdump — Display Full-Text Index information”。
