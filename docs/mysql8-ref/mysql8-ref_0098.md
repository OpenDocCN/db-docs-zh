# 2.10 Perl 安装注意事项

> 原文：[`dev.mysql.com/doc/refman/8.0/en/perl-support.html`](https://dev.mysql.com/doc/refman/8.0/en/perl-support.html)

2.10.1 在 Unix 上安装 Perl

2.10.2 在 Windows 上安装 ActiveState Perl

2.10.3 使用 Perl DBI/DBD 接口时出现的问题

Perl `DBI` 模块提供了一个通用的数据库访问接口。您可以编写一个适用于许多不同数据库引擎的 `DBI` 脚本，而无需更改。要使用 `DBI`，您必须安装 `DBI` 模块，以及每种类型的数据库服务器的 DataBase Driver (DBD) 模块。对于 MySQL，这个驱动程序是 `DBD::mysql` 模块。

注意

MySQL 发行版不包含 Perl 支持。您可以从[`search.cpan.org`](http://search.cpan.org)获取必要的模块，Unix 系统可以使用 ActiveState **ppm**程序在 Windows 上获取。以下部分描述了如何执行此操作。

`DBI`/`DBD` 接口需要 Perl 5.6.0，最好使用 5.6.1 或更高版本。如果您使用较旧版本的 Perl，`DBI` *无法工作*。您应该使用`DBD::mysql` 4.009 或更高版本。尽管早期版本可用，但它们不支持 MySQL 8.0 的全部功能。
