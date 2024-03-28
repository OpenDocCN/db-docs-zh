# 12.13.3 复杂字符集的多字节字符支持

> 原文：[`dev.mysql.com/doc/refman/8.0/en/multibyte-characters.html`](https://dev.mysql.com/doc/refman/8.0/en/multibyte-characters.html)

如果您想要为一个名为*`MYSET`*的新字符集添加多字节字符支持，您必须在`strings`目录中的`ctype-*`MYSET`*.c`源文件中使用多字节字符函数。

现有的字符集提供了最佳的文档和示例，展示了这些函数是如何实现的。查看`strings`目录中的`ctype-*.c`文件，例如`euc_kr`、`gb2312`、`gbk`、`sjis`和`ujis`字符集的文件。查看`MY_CHARSET_HANDLER`结构以了解它们的使用方式。另请参阅`strings`目录中的`CHARSET_INFO.txt`文件获取更多信息。
