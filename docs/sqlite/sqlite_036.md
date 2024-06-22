# 1\. 概述

> 原文：[`sqlite.com/csv.html`](https://sqlite.com/csv.html)

CSV 虚拟表读取[RFC 4180](https://www.ietf.org/rfc/rfc4180.txt)格式的逗号分隔值，并将该内容作为 SQL 表的行和列返回。

CSV 虚拟表对需要批量加载大量逗号分隔内容的应用程序非常有用。CSV 虚拟表还可以作为实现其他虚拟表的模板源文件。

CSV 虚拟表没有内置到 SQLite 总汇中。它作为一个[单独的源文件](https://www.sqlite.org/src/artifact?ci=trunk&filename=ext/misc/csv.c)可用，可以编译成可加载扩展。从命令行 shell 中典型使用 CSV 虚拟表的方式可能如下：

```sql
.load ./csv
CREATE VIRTUAL TABLE temp.t1 USING csv(filename='thefile.csv');
SELECT * FROM t1;

```

上述脚本的第一行导致命令行 shell 读取并激活 CSV 的运行时可加载扩展。对于应用程序，等效的 C 语言 API 是 sqlite3_load_extension()。注意，扩展文件名中的扩展名（例如：".dll"、".so" 或 ".dylib"）被省略了。省略扩展名并非必需，但有助于使脚本跨平台。SQLite 将自动附加适当的扩展名。

上述第二行创建了一个名为 "t1" 的虚拟表，该表读取参数中指定文件的内容。列的数量和名称通过读取内容的第一行自动确定。CSV 虚拟表的其他选项允许从字符串而不是单独的文件中获取 CSV 内容，并且给程序员更多控制列的数量和名称。下面详细介绍了这些选项。CSV 虚拟表通常作为临时表创建，因此仅存在于当前数据库连接中，并不成为数据库模式的永久部分。请注意，SQLite 中没有 "CREATE TEMP VIRTUAL TABLE" 命令。而是将虚拟表的名称前缀 "temp." 。

示例的第三行显示了虚拟表的使用，以读取 CSV 文件的所有内容。这可能是使用虚拟表的最简单可能方式。CSV 虚拟表可用于任何可以使用普通虚拟表的地方。可以在子查询、公共表达式中使用 CSV 虚拟表，或根据需要添加 WHERE、GROUP BY、HAVING、ORDER BY 和 LIMIT 子句。

# 2\. Arguments

上面的示例展示了 CSV 虚拟表的单个 **filename='thefile.csv'** 参数。但还有其他可能的参数。

+   **filename=***FILENAME*

    **filename=** 参数指定从中读取 CSV 内容的外部文件。每个 CSV 虚拟表必须有一个 **filename=** 参数或一个 **data=** 参数，而不能同时存在两者。

+   **data=***TEXT*

    **data=**参数指定*TEXT*为 CSV 文件的字面内容。

+   **schema=***SCHEMA*

    **schema=**参数指定一个 CREATE TABLE 语句，CSV 虚拟表将其传递给 sqlite3_declare_vtab()接口，以定义虚拟表中列的名称。

+   **columns=***N*

    **columns=***N*参数指定 CSV 文件中的列数。如果输入数据包含的列数超过此值，则多余的列将被忽略。如果输入数据包含的列数少于此值，则额外的列将填充为 NULL。如果省略**columns=***N*参数，则读取 CSV 文件的第一行以确定列的数量。

+   **header=***BOOLEAN*

    或者只是

    **header**

    如果**header**参数为 true，则将 CSV 文件的第一行视为标题而不是数据。CSV 文件的第二行成为内容的第一行。如果省略**schema=**选项，则 CSV 文件的第一行确定列的名称。

# 3\. 列名

虚拟表的列名主要由**schema=**参数确定。如果省略**schema=**参数但**header**为 true，则 CSV 文件第一行的值成为列名。如果省略**schema=**参数且**header**为 false，则列名为"c0"、"c1"、"c2"等。
