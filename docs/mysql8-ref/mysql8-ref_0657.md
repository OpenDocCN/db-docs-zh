# 11.2 模式对象名称

> 原文：[`dev.mysql.com/doc/refman/8.0/en/identifiers.html`](https://dev.mysql.com/doc/refman/8.0/en/identifiers.html)

11.2.1 标识符长度限制

11.2.2 标识符限定符

11.2.3 标识符大小写敏感性

11.2.4 标识符映射到文件名

11.2.5 函数名称解析和解析

MySQL 中的某些对象，包括数据库、表、索引、列、别名、视图、存储过程、分区、表空间、资源组和其他对象名称被称为标识符。本节描述了 MySQL 中标识符的允许语法。第 11.2.1 节，“标识符长度限制”指示了每种类型标识符的最大长度。第 11.2.3 节，“标识符大小写敏感性”描述了哪些类型的标识符是区分大小写的以及在什么条件下。

标识符可以被引用或未引用。如果标识符包含特殊字符或是保留字，您在引用时*必须*对其进行引用。（例外情况：在限定名称中跟在句点后面的保留字必须是标识符，因此不需要引用。）保留字列在第 11.3 节，“关键字和保留字”中。

在内部，标识符被转换为并存储为 Unicode（UTF-8）。标识符中允许的 Unicode 字符是基本多文种平面（BMP）中的字符。不允许使用补充字符。因此，标识符可能包含这些字符：

+   未引用标识符中允许的字符：

    +   ASCII：[0-9,a-z,A-Z$_]（基本拉丁字母、数字 0-9、美元符号、下划线）

    +   扩展字符集：U+0080 .. U+FFFF

+   引用标识符中允许的字符包括完整的 Unicode 基本多文种平面（BMP），除了 U+0000：

    +   ASCII：U+0001 .. U+007F

    +   扩展字符集：U+0080 .. U+FFFF

+   ASCII NUL（U+0000）和补充字符（U+10000 及更高）不允许在引用或未引用的标识符中使用。

+   标识符可以以数字开头，但除非引用，不能完全由数字组成。

+   数据库、表和列名称不能以空格字符结尾。

+   从 MySQL 8.0.32 开始，将美元符号作为未引用的数据库、表、视图、列、存储程序或别名名称的第一个字符已被弃用，并会产生警告。这包括与限定符一起使用的名称（参见第 11.2.2 节，“标识符限定符”）。当按照本节后面给出的规则引用时，美元符号仍然可以用作此类标识符的前导字符。

��识符引用字符是反引号（```sql):

```

mysql> SELECT * FROM `select` WHERE `select`.id > 100;

```sql

If the `ANSI_QUOTES` SQL mode is enabled, it is also permissible to quote identifiers within double quotation marks:

```

mysql> CREATE TABLE "test" (col INT);

错误 1064：您的 SQL 语法中存在错误...

mysql> SET sql_mode='ANSI_QUOTES';

mysql> CREATE TABLE "test" (col INT);

查询成功，影响行数为 0（0.00 秒）

```sql

The `ANSI_QUOTES` mode causes the server to interpret double-quoted strings as identifiers. Consequently, when this mode is enabled, string literals must be enclosed within single quotation marks. They cannot be enclosed within double quotation marks. The server SQL mode is controlled as described in Section 7.1.11, “Server SQL Modes”.

Identifier quote characters can be included within an identifier if you quote the identifier. If the character to be included within the identifier is the same as that used to quote the identifier itself, then you need to double the character. The following statement creates a table named `a`b` that contains a column named `c"d`:

```

mysql> CREATE TABLE `a``b` (`c"d` INT);

```sql

In the select list of a query, a quoted column alias can be specified using identifier or string quoting characters:

```

mysql> SELECT 1 AS `one`, 2 AS 'two';

+-----+-----+

| one | two |
| --- | --- |

+-----+-----+

|   1 |   2 |
| --- | --- |

+-----+-----+

```

在语句的其他地方，对别名的引用必须使用标识符引用，否则引用将被视为字符串文字。

建议不要使用以 `*`M`*e` 或 `*`M`*e*`N`*` 开头的名称，其中 *`M`* 和 *`N`* 是整数。例如，避免使用 `1e` 作为标识符，因为诸如 `1e+3` 这样的表达式是模棱两可的。根据上下文，它可能被解释为表达式 `1e + 3` 或数字 `1e+3`。

在使用 `MD5()` 生成表名时要小心，因为它可能生成类似于刚才描述的非法或模棱两可的格式的名称。

还建议不要使用以 `!hidden!` 开头的列名，以确保新名称不会与现有隐藏列用于功能索引的名称发生冲突。

用户变量不能直接用作 SQL 语句中的标识符或标识符的一部分。请参阅第 11.4 节，“用户定义变量”，了解更多信息和解决方法的示例。

数据库和表名中的特殊字符在相应的文件系统名称中进行编码，如第 11.2.4 节，“标识符映射到文件名”中所述。
