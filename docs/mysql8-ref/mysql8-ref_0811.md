# 14.9.5 全文限制

> 原文：[`dev.mysql.com/doc/refman/8.0/en/fulltext-restrictions.html`](https://dev.mysql.com/doc/refman/8.0/en/fulltext-restrictions.html)

+   仅支持对`InnoDB`和`MyISAM`表进行全文搜索。

+   不支持对分区表进行全文搜索。请参阅第 26.6 节“分区的限制和限制”。

+   大多数多字节字符集都可以与全文搜索一起使用。唯一的例外是对于 Unicode，可以使用`utf8mb3`或`utf8mb4`字符集，但不能使用`ucs2`字符集。虽然无法使用`ucs2`列上的`FULLTEXT`索引，但可以在没有此类索引的`ucs2`列上执行`IN BOOLEAN MODE`搜索。

    `utf8mb3`的备注也适用于`utf8mb4`，`ucs2`的备注也适用于`utf16`、`utf16le`和`utf32`。

+   汉语和日语等表意语言没有词分隔符。因此，内置全文解析器*无法确定这些语言中单词的起始和结束位置*。

    提供了支持中文、日文和韩文（CJK）的基于字符的 ngram 全文解析器，以及支持日文的基于词的 MeCab 解析器插件，可用于`InnoDB`和`MyISAM`表。

+   尽管支持在单个表中使用多个字符集，但`FULLTEXT`索引中的所有列必须使用相同的字符集和排序规则。

+   `MATCH()`列列表必须与表的某个`FULLTEXT`索引定义中的列列表完全匹配，除非这个`MATCH()`在`MyISAM`表上是`IN BOOLEAN MODE`。对于`MyISAM`表，布尔模式搜索可以在非索引列上进行，尽管可能会很慢。

+   `AGAINST()`的参数必须是在查询评估期间保持不变的字符串值。例如，表列就不符合要求，因为每行可能不同。

    截至 MySQL 8.0.28 版本，`MATCH()`的参数不能使用汇总列。

+   对于`FULLTEXT`搜索，索引提示比非`FULLTEXT`搜索更受限制。请参阅第 10.9.4 节“索引提示”。

+   对于`InnoDB`，涉及具有全文索引的列的所有 DML 操作（`INSERT`、`UPDATE`、`DELETE`）在事务提交时处理。例如，对于`INSERT`操作，插入的字符串被标记化并分解为单词。当事务提交时，这些单词将被添加到全文索引表中。因此，全文搜索仅返回已提交的数据。

+   '%' 字符不是全文搜索的支持通配符字符。
