# 6.7 程序开发工具

> 原文：[`dev.mysql.com/doc/refman/8.0/en/programs-development.html`](https://dev.mysql.com/doc/refman/8.0/en/programs-development.html)

6.7.1 mysql_config — 显示用于编译客户端的选项

6.7.2 my_print_defaults — 显示选项文件中的选项

本节描述了在开发 MySQL 程序时可能会有用的一些实用工具。

在 shell 脚本中，您可以使用 **my_print_defaults** 程序来解析选项文件，并查看给定程序将使用哪些选项。以下示例显示了当要求显示在 `[client]` 和 `[mysql]` 组中找到的选项时，**my_print_defaults** 可能产生的输出：

```sql
$> my_print_defaults client mysql
--port=3306
--socket=/tmp/mysql.sock
--no-auto-rehash
```

开发者注意：选项文件处理是通过在处理适当组或组中的所有选项之前处理所有选项来实现的。这对于使用指定多次的选项的最后一个实例的程序非常有效。如果您有一个处理多次指定选项的 C 或 C++ 程序，但不读取选项文件，您只需添加两行代码即可赋予其这种能力。查看任何标准 MySQL 客户端的源代码以了解如何实现。

MySQL 的几种其他语言接口基于 C 客户端库，其中一些提供了访问选项文件内容的方法。这些包括 Perl 和 Python。有关详细信息，请参阅您首选接口的文档。
