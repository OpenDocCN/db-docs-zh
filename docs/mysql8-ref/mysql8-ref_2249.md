# 31.9 MySQL Perl API

> 原文：[`dev.mysql.com/doc/refman/8.0/en/apis-perl.html`](https://dev.mysql.com/doc/refman/8.0/en/apis-perl.html)

Perl `DBI` 模块提供了一个通用的数据库访问接口。您可以编写一个可以与许多不同数据库引擎一起使用的 DBI 脚本，而无需更改。要与 MySQL 一起使用 DBI，请安装以下内容：

1.  `DBI` 模块。

1.  `DBD::mysql` 模块。这是 Perl 的 DataBase Driver (DBD) 模块。

1.  可选地，用于访问其他类型数据库服务器的 DBD 模块。

Perl DBI 是推荐的 Perl 接口。它取代了一个名为 `mysqlperl` 的旧接口，应被视为过时。

这些部分包含有关在 Perl 中与 MySQL 一起使用以及在 Perl 中编写 MySQL 应用程序的信息：

+   有关 Perl DBI 支持的安装说明，请参见 第 2.10 节，“Perl 安装说明”。

+   有关从选项文件中读取选项的示例，请参见 第 7.8.4 节，“在多服务器环境中使用客户端程序”。

+   有关安全编码提示，请参见 第 8.1.1 节，“安全指南”。

+   有关调试提示，请参见 第 7.9.1.4 节，“在 gdb 下调试 mysqld”。

+   有关一些 Perl 特定的环境变量，请参见 第 6.9 节，“环境变量”。

+   关于在 macOS 上运行的考虑，请参见 第 2.4 节，“在 macOS 上安装 MySQL”。

+   有关引用字符串文字的方法，请参见 第 11.1.1 节，“字符串文字”。

DBI 信息可以在命令行、在线或印刷形式中获得：

+   一旦安装了 `DBI` 和 `DBD::mysql` 模块，您可以使用 `perldoc` 命令在命令行获取有关它们的信息：

    ```sql
    $> perldoc DBI
    $> perldoc DBI::FAQ
    $> perldoc DBD::mysql
    ```

    您还可以使用 `pod2man`、`pod2html` 等工具将此信息转换为其他格式。

+   有关 Perl DBI 的在线信息，请访问 DBI 网站 [`dbi.perl.org/`](http://dbi.perl.org/)。该网站托管了一个通用的 DBI 邮件列表。

+   有关印刷信息，官方的 DBI 书籍是 *Programming the Perl DBI*（Alligator Descartes 和 Tim Bunce，O'Reilly & Associates，2000）。有关该书的信息可在 DBI 网站 [`dbi.perl.org/`](http://dbi.perl.org/) 上找到。
