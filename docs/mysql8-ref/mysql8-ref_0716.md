# 12.11 字符集限制

> 原文：[`dev.mysql.com/doc/refman/8.0/en/charset-restrictions.html`](https://dev.mysql.com/doc/refman/8.0/en/charset-restrictions.html)

+   标识符存储在`mysql`数据库表（`user`、`db`等）中，使用`utf8mb3`，但标识符只能包含基本多文种平面（BMP）中的字符。标识符中不允许使用补充字符。

+   `ucs2`、`utf16`、`utf16le`和`utf32`字符集有以下限制：

    +   不能将它们中的任何一个用作客户端字符集。参见不允许的客户端字符集。

    +   目前无法使用`LOAD DATA`加载使用这些字符集的数据文件。

    +   不能在使用任何这些字符集的列上创建`FULLTEXT`索引。但是，您可以在没有索引的情况下对该列执行`IN BOOLEAN MODE`搜索。

+   `REGEXP`和`RLIKE`运算符以字节方式工作，因此它们不是多字节安全的，可能会在多字节字符集下产生意外结果。此外，这些运算符通过它们的字节值比较字符，即使给定的排序将它们视为相等，重音字符也可能不会被视为相等。
