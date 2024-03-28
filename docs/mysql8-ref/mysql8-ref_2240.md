# 第三十一章 连接器和 API

> 原文：[`dev.mysql.com/doc/refman/8.0/en/connectors-apis.html`](https://dev.mysql.com/doc/refman/8.0/en/connectors-apis.html)

**目录**

31.1 MySQL Connector/C++

31.2 MySQL Connector/J

31.3 MySQL Connector/NET

31.4 MySQL Connector/ODBC

31.5 MySQL Connector/Python

31.6 MySQL Connector/Node.js

31.7 MySQL C API

31.8 MySQL PHP API

31.9 MySQL Perl API

31.10 MySQL Python API

31.11 MySQL Ruby APIs

31.11.1 MySQL/Ruby API

31.11.2 Ruby/MySQL API

31.12 MySQL Tcl API

31.13 MySQL Eiffel Wrapper

MySQL 连接器为客户端程序提供了与 MySQL 服务器的连接。API 使用经典的 MySQL 协议或 X 协议提供对 MySQL 资源的低级访问。连接器和 API 都使您能够从另一种语言或环境连接和执行 MySQL 语句，包括 ODBC、Java（JDBC）、C++、Python、Node.js、PHP、Perl、Ruby 和 C。

## MySQL 连接器

Oracle 开发了许多连接器：

+   Connector/C++ 使 C++应用程序能够连接到 MySQL。

+   Connector/J 提供了驱动程序支持，用于使用标准的 Java 数据库连接（JDBC）API 从 Java 应用程序连接到 MySQL。

+   Connector/NET 使开发人员能够创建连接到 MySQL 的.NET 应用程序。Connector/NET 实现了完全功能的 ADO.NET 接口，并提供了与 ADO.NET 感知工具一起使用的支持。使用 Connector/NET 的应用程序可以用任何支持的.NET 语言编写。

+   Connector/ODBC 提供了驱动程序支持，用于使用开放数据库连接（ODBC）API 连接到 MySQL。支持从 Windows、Unix 和 macOS 平台进行 ODBC 连接。

+   Connector/Python 提供了驱动程序支持，用于使用符合[Python DB API version 2.0](http://www.python.org/dev/peps/pep-0249/)的 API 从 Python 应用程序连接到 MySQL。不需要额外的 Python 模块或 MySQL 客户端库。

+   `Connector/Node.js`提供了一个异步 API，用于使用 X 协议从 Node.js 应用程序连接到 MySQL。Connector/Node.js 支持管理数据库会话和模式，处理 MySQL 文档存储集合，并使用原始 SQL 语句。

## MySQL C API

要直接在 C 应用程序中使用 MySQL，C API 通过`libmysqlclient`客户端库提供了对 MySQL 客户端/服务器协议的低级访问。这是连接到 MySQL 服务器实例的主要方法，被 MySQL 命令行客户端和这里详细介绍的许多 MySQL 连接器和第三方 API 所使用。

`libmysqlclient`包含在 MySQL 发行版中。

另请参阅 MySQL C API 实现。

要从 C 应用程序访问 MySQL，或者为本章中未支持的语言构建与 MySQL 的接口，C API 是起点。有许多程序员工具可用于帮助处理此过程；请参阅第 6.7 节，“程序开发工具”。

## 第三方 MySQL API

本章描述的其余 API 提供了从特定应用程序语言到 MySQL 的接口。这些第三方解决方案并非由 Oracle 开发或支持。这里仅提供它们的使用和能力的基本信息供参考。

所有第三方语言 API 都是使用两种方法之一开发的，使用`libmysqlclient`或通过实现本机驱动程序。这两种解决方案提供不同的好处：

+   使用*`libmysqlclient`*完全兼容 MySQL，因为它使用与 MySQL 客户端应用程序相同的库。但是，功能集仅限于通过`libmysqlclient`公开的实现和接口，性能可能较低，因为数据在本机语言和 MySQL API 组件之间进行复制。

+   *本机驱动程序*是完全在主机语言或环境中实现 MySQL 网络协议的驱动程序。本机驱动程序速度快，因为在组件之间的数据复制较少，并且可以提供标准 MySQL API 无法提供的高级功能。本机驱动程序也更容易供最终用户构建和部署，因为构建本机驱动程序组件不需要 MySQL 客户端库的副本。

表 31.1，“MySQL API 和接口”列出了许多可用于 MySQL 的库和接口。

**表 31.1 MySQL API 和接口**

| 环境 | API | 类型 | 备注 |
| --- | --- | --- | --- |
| Ada | GNU Ada MySQL 绑定 | `libmysqlclient` | 请参阅[GNU Ada 的 MySQL 绑定](http://gnade.sourceforge.net/) |
| C | C API | `libmysqlclient` | 请参阅 MySQL 8.0 C API 开发人员指南。 |
| C++ | 连接器/C++ | `libmysqlclient` | 请参阅 MySQL Connector/C++ 8.3 开发人员指南。 |
|  | MySQL++ | `libmysqlclient` | 请参阅[MySQL++网站](http://tangentsoft.net/mysql++/doc/)。 |
|  | MySQL wrapped | `libmysqlclient` | 查看[MySQL wrapped](http://www.alhem.net/project/mysql/)。 |
| Cocoa | MySQL-Cocoa | `libmysqlclient` | 与 Objective-C Cocoa 环境兼容。查看[`mysql-cocoa.sourceforge.net/`](http://mysql-cocoa.sourceforge.net/)。 |
| D | D 的 MySQL | `libmysqlclient` | 查看[D 的 MySQL](http://www.steinmole.de/d/)。 |
| Eiffel | Eiffel MySQL | `libmysqlclient` | 查看第 31.13 节，“MySQL Eiffel 包装器”。 |
| Erlang | `erlang-mysql-driver` | `libmysqlclient` | 查看[`erlang-mysql-driver`](http://code.google.com/p/erlang-mysql-driver/)。 |
| Haskell | Haskell MySQL 绑定 | 本地驱动程序 | 查看[Brian O'Sullivan 的纯 Haskell MySQL 绑定](http://www.serpentine.com/blog/software/mysql/)。 |
|  | `hsql-mysql` | `libmysqlclient` | 查看[Haskell 的 MySQL 驱动程序](http://hackage.haskell.org/cgi-bin/hackage-scripts/package/hsql-mysql-1.7)。 |
| Java/JDBC | Connector/J | 本地驱动程序 | 查看 MySQL Connector/J 8.0 开发者指南。 |
| Kaya | MyDB | `libmysqlclient` | 查看[MyDB](http://kayalang.org/library/latest/MyDB)。 |
| Lua | LuaSQL | `libmysqlclient` | 查看[LuaSQL](http://keplerproject.github.io/luasql/doc/us/)。 |
| .NET/Mono | Connector/NET | 本地驱动程序 | 查看 MySQL Connector/NET 开发者指南。 |
| Objective Caml | Objective Caml MySQL 绑定 | `libmysqlclient` | 查看[Objective Caml 的 MySQL 绑定](http://raevnos.pennmush.org/code/ocaml-mysql/)。 |
| Octave | GNU Octave 的数据库绑定 | `libmysqlclient` | 查看[GNU Octave 的数据库绑定](http://octave.sourceforge.net/database/index.html)。 |
| ODBC | Connector/ODBC | `libmysqlclient` | 查看 MySQL Connector/ODBC 开发者指南。 |
| Perl | `DBI`/`DBD::mysql` | `libmysqlclient` | 查看第 31.9 节，“MySQL Perl API”。 |
|  | `Net::MySQL` | 本地驱动程序 | 查看[`Net::MySQL`](http://search.cpan.org/dist/Net-MySQL/MySQL.pm)在 CPAN 上。 |
| PHP | `mysql`，`ext/mysql`接口（已弃用） | `libmysqlclient` | 查看 MySQL 和 PHP。 |
|  | `mysqli`，`ext/mysqli`接口 | `libmysqlclient` | 查看 MySQL 和 PHP。 |
|  | `PDO_MYSQL` | `libmysqlclient` | 查看 MySQL 和 PHP。 |
|  | PDO mysqlnd | 本地驱动程序 |  |
| Python | Connector/Python | 本地驱动程序 | 查看 MySQL Connector/Python 开发者指南。 |
| Python | Connector/Python C 扩展 | `libmysqlclient` | 查看 MySQL Connector/Python 开发者指南。 |
|  | MySQLdb | `libmysqlclient` | 查看第 31.10 节，“MySQL Python API”。 |
| Ruby | mysql2 | `libmysqlclient` | 使用`libmysqlclient`。查看第 31.11 节，“MySQL Ruby APIs”。 |
| Scheme | `Myscsh` | `libmysqlclient` | 查看[`Myscsh`](https://github.com/aehrisch/myscsh)。 |
| SPL | `sql_mysql` | `libmysqlclient` | 查看[SPL 中的`sql_mysql`](http://www.clifford.at/spl/spldoc/sql_mysql.html)。 |
| Tcl | MySQLtcl | `libmysqlclient` | 查看第 31.12 节，“MySQL Tcl API”。 |
| 环境 | API | 类型 | 备注 |
