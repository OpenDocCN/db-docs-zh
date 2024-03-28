# 1.1 关于本手册

> 原文：[`dev.mysql.com/doc/refman/8.0/en/manual-info.html`](https://dev.mysql.com/doc/refman/8.0/en/manual-info.html)

这是 MySQL 数据库系统的参考手册，版本 8.0，直至发布 8.0.36。MySQL 8.0 的次要版本之间的差异在本文中以发布号（8.0.*`x`*）的形式进行说明。有关许可信息，请参阅法律声明。

本手册不适用于 MySQL 软件的旧版本，因为 MySQL 8.0 与以前版本之间存在许多功能和其他差异。如果您使用的是 MySQL 软件的早期版本，请参考相应的手册。例如，*MySQL 5.7 参考手册*涵盖了 MySQL 软件 5.7 系列的发布。

因为本手册是作为参考使用的，所以不提供有关 SQL 或关系数据库概念的一般指导。它也不会教您如何使用操作系统或命令行解释器。

MySQL 数据库软件正在不断发展，参考手册也经常更新。手册的最新版本可在线搜索形式在`dev.mysql.com/doc/`上获取。其他格式也可在那里找到，包括可下载的 HTML 和 PDF 版本。

MySQL 本身的源代码包含使用 Doxygen 编写的内部文档。生成的 Doxygen 内容可从`dev.mysql.com/doc/index-other.html`获取。还可以使用 MySQL 源代码分发中的说明从本地生成此内容，方法请参见 Section 2.8.10，“生成 MySQL Doxygen 文档内容”。

如果您对使用 MySQL 有疑问，请加入[MySQL 社区 Slack](https://mysqlcommunity.slack.com/)。如果您对手册本身的添加或更正有建议，请发送至[`www.mysql.com/company/contact/`](http://www.mysql.com/company/contact/)。

### 排版和语法约定

本手册使用特定的排版约定：

+   `这种风格的文本`用于 SQL 语句；数据库、表和列名称；程序清单和源代码；以及环境变量。例如：“要重新加载授权表，请使用`FLUSH PRIVILEGES`语句。”

+   **`这种风格的文本`**表示您在示例中键入的输入。

+   **这种风格的文本**表示可执行程序和脚本的名称，例如**mysql**（MySQL 命令行客户端程序）和**mysqld**（MySQL 服务器可执行文件）。

+   *`这种风格的文本`*用于变量输入，您应该替换为自己选择的值。

+   *这种样式的文本* 用于强调。

+   **这种样式的文本** 用于表头和传达特别强烈的强调。

+   `这种样式的文本` 用于指示影响程序执行方式的程序选项，或提供程序需要以某种方式运行的信息。*示例*：“`--host` 选项（简写 `-h`）告诉 **mysql** 客户端程序应连接到的 MySQL 服务器的主机名或 IP 地址”。

+   文件名和目录名写成这样：“全局 `my.cnf` 文件位于 `/etc` 目录中。”

+   字符序列写成这样：“要指定通配符，请使用 ‘`%`’ 字符。”

当命令或语句以提示符为前缀时，我们使用这些：

```sql
$> type a command here
#> type a command as *root* here
C:\> type a command here (Windows only)
mysql> type a mysql statement here
```

命令在您的命令解释器中执行。在 Unix 上，这通常是诸如 **sh**、**csh** 或 **bash** 的程序。在 Windows 上，相应的程序是 **command.com** 或 **cmd.exe**，通常在控制台窗口中运行。以 `mysql` 为前缀的语句在 **mysql** 命令行客户端中执行。

注意

当您输入示例中显示的命令或语句时，请不要输入示例中显示的提示符。

在某些领域，不同的系统可能会区分彼此，以显示命令应在两个不同的环境中执行。例如，在处理复制时，命令可能会以 `source` 和 `replica` 为前缀：

```sql
source> type a mysql statement on the replication source here
replica> type a mysql statement on the replica here
```

数据库、表和列名通常需要替换到语句中。为了指示这种替换是必要的，本手册使用 *`db_name`*、*`tbl_name`* 和 *`col_name`*。例如，您可能会看到这样的语句：

```sql
mysql> SELECT *col_name* FROM *db_name*.*tbl_name*;
```

这意味着，如果您要输入类似的语句，您需要提供自己的数据库、表和列名，可能像这样：

```sql
mysql> SELECT author_name FROM biblio_db.author_list;
```

SQL 关键字不区分大小写，可以以任何大小写形式编写。本手册使用大写。

在语法描述中，方括号（“`[`”和“`]`”）表示可选的单词或从句。例如，在以下语句中，`IF EXISTS` 是可选的：

```sql
DROP TABLE [IF EXISTS] *tbl_name*
```

当语法元素由多个备选项组成时，备选项用竖线（“`|`”）分隔。当必须选择一组选择中的一个成员时，备选项列在方括号（“`[`”和“`]`”）中：

```sql
TRIM([[BOTH | LEADING | TRAILING] [*remstr*] FROM] *str*)
```

当必须选择一组选择中的一个成员时，备选项列在大括号（“`{`”和“`}`”）中：

```sql
{DESCRIBE | DESC} *tbl_name* [*col_name* | *wild*]
```

省略号（`...`）表示省略语句的一部分，通常是为了提供更复杂语法的简短版本。例如，`SELECT ... INTO OUTFILE` 是 `SELECT` 语句的缩写形式，其后跟有 `INTO OUTFILE` 子句。

省略号也可以表示语句中前面的语法元素可能重复。在下面的示例中，可以给出多个*`reset_option`*值，第一个之后的每个值前面都有逗号：

```sql
RESET *reset_option* [,*reset_option*] ...
```

使用 Bourne shell 语法显示设置 shell 变量的命令。例如，使用 Bourne shell 语法设置`CC`环境变量并运行**configure**命令的序列如下：

```sql
$> CC=gcc ./configure
```

如果您使用**csh**或**tcsh**，则必须以稍有不同的方式发出命令：

```sql
$> setenv CC gcc
$> ./configure
```

### 手册作者

参考手册源文件采用 DocBook XML 格式编写。HTML 版本和其他格式是自动生成的，主要使用 DocBook XSL 样式表。有关 DocBook 的信息，请参见[`docbook.org/`](http://docbook.org/)

本手册最初由 David Axmark 和 Michael “Monty” Widenius 编写。由 MySQL 文档团队维护，团队成员包括 Edward Gilmore、Stefan Hinz、David Hollis、Philip Olson、Daniel So 和 Jon Stephens。
