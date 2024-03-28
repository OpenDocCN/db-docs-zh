# 12.13.2 复杂字符集的字符串排序支持

> 原文：[`dev.mysql.com/doc/refman/8.0/en/string-collating.html`](https://dev.mysql.com/doc/refman/8.0/en/string-collating.html)

对于一个名为*`MYSET`*的简单字符集，排序规则在`*`MYSET`*.xml`配置文件中使用`<collation>`元素内的`<map>`数组元素指定。如果你的语言的排序规则过于复杂，无法用简单的数组处理，你必须在`strings`目录中的`ctype-*`MYSET`*.c`源文件中定义字符串排序函数。

现有的字符集提供了最好的文档和示例，展示了这些函数是如何实现的。查看`strings`目录中的`ctype-*.c`文件，比如`big5`、`czech`、`gbk`、`sjis`和`tis160`字符集的文件。查看`MY_COLLATION_HANDLER`结构体，了解它们是如何使用的。另请参阅`strings`目录中的`CHARSET_INFO.txt`文件获取更多信息。
