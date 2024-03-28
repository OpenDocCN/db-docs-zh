# 1.2.2 MySQL 的主要特性

> 原文：[`dev.mysql.com/doc/refman/8.0/en/features.html`](https://dev.mysql.com/doc/refman/8.0/en/features.html)

本节描述了 MySQL 数据库软件的一些重要特性。在大多数方面，路线图适用于所有 MySQL 版本。有关在特定系列基础上引入 MySQL 的功能的信息，请参见适当手册的“简介”部分：

+   MySQL 8.0：第 1.3 节，“MySQL 8.0 中的新功能”

+   MySQL 5.7：MySQL 5.7 中的新功能

#### 内部结构和可移植性

+   用 C 和 C++ 编写。

+   使用各种不同编译器进行测试。

+   可在许多不同平台上运行。参见[`www.mysql.com/support/supportedplatforms/database.html`](https://www.mysql.com/support/supportedplatforms/database.html)。

+   为了可移植性，使用**CMake**进行配置。

+   使用 Purify（一种商业内存泄漏检测器）以及 Valgrind 进行测试，Valgrind 是一个 GPL 工具（[`valgrind.org/`](https://valgrind.org/)）。

+   使用具有独立模块的多层服务器设计。

+   设计为完全多线程，使用内核线程，以便在有多个 CPU 可用时轻松使用。

+   提供事务性和非事务性存储引擎。

+   使用非常快速的 B 树磁盘表（`MyISAM`），具有索引压缩。

+   设计为相对容易添加其他存储引擎。如果您想为内部数据库提供 SQL 接口，这将非常有用。

+   使用非常快速的基于线程的内存分配系统。

+   使用优化的嵌套循环连接执行非常快速的连接。

+   实现内存中的哈希表，用作临时表。

+   使用高度优化的类库实现 SQL 函数，应尽可能快速。通常在查询初始化后根本不需要任何内存分配。

+   将服务器作为单独的程序提供，用于客户端/服务器网络环境。

#### 数据类型

+   许多数据类型：有符号/无符号整数，长度分别为 1、2、3、4 和 8 字节，`FLOAT` - FLOAT, DOUBLE")，`DOUBLE` - FLOAT, DOUBLE")，`CHAR`，`VARCHAR`，`BINARY`，`VARBINARY`，`TEXT`，`BLOB`，`DATE`，`TIME`，`DATETIME`，`TIMESTAMP`，`YEAR`，`SET`，`ENUM` 和 OpenGIS 空间类型。参见第十三章，*数据类型*。

+   固定长度和可变长度的字符串类型。

#### 语句和函数

+   在查询的`SELECT`列表和`WHERE`子句中完整支持运算符和函数。例如：

    ```sql
    mysql> SELECT CONCAT(first_name, ' ', last_name)
        -> FROM citizen
        -> WHERE income/dependents > 10000 AND age > 30;
    ```

+   完全支持 SQL 的`GROUP BY`和`ORDER BY`子句。支持分组函数（`COUNT()`，`AVG()`，`STD()`，`SUM()`，`MAX()`，`MIN()` 和`GROUP_CONCAT()`)。

+   支持使用标准 SQL 和 ODBC 语法进行`LEFT OUTER JOIN`和`RIGHT OUTER JOIN`。

+   支持标准 SQL 所需的表和列的别名。

+   支持`DELETE`，`INSERT`，`REPLACE` 和`UPDATE`语句，以返回更改（受影响）的行数，或者通过在连接到服务器时设置标志来返回匹配的行数。

+   支持特定于 MySQL 的`SHOW`语句，用于检索有关数据库、存储引擎、表和索引的信息。支持根据标准 SQL 实现的`INFORMATION_SCHEMA`数据库。

+   一个`EXPLAIN`语句，显示优化器如何解析查询。

+   函数名称与表或列名称无关。例如，`ABS`是一个有效的列名。唯一的限制是对于函数调用，函数名称和随后的“`(`”之间不允许有空格。参见第 11.3 节，“关键字和保留字”。

+   您可以在同一语句中引用来自不同数据库的表。

#### 安全性

+   非常灵活和安全的权限和密码系统，支持基于主机的验证。

+   通过加密所有密码流量来实现密码安全性，当您连接到服务器时。

#### 可扩展性和限制

+   支持大型数据库。我们使用包含 5000 万条记录的 MySQL 服务器。我们还知道有用户使用具有 20 万个表和约 50 亿行的 MySQL 服务器。

+   每个表最多支持 64 个索引。每个索引可以由 1 到 16 列或列的部分组成。对于`InnoDB`表，最大索引宽度为 767 字节或 3072 字节。参见第 17.22 节，“InnoDB 限制”。对于`MyISAM`表，最大索引宽度为 1000 字节。参见第 18.2 节，“MyISAM 存储引擎”。索引可以使用列的前缀，适用于`CHAR`、`VARCHAR`、`BLOB`或`TEXT`列类型。

#### 连接性

+   客户端可以使用多种协议连接到 MySQL 服务器：

    +   客户端可以在任何平台上使用 TCP/IP 套接字进行连接。

    +   在 Windows 系统上，如果服务器启用了`named_pipe`系统变量，客户端可以使用命名管道进行连接。如果启用了`shared_memory`系统变量，Windows 服务器还支持共享内存连接。客户端可以通过使用`--protocol=memory`选项通过共享内存进行连接。

    +   在 Unix 系统上，客户端可以使用 Unix 域套接字文件进行连接。

+   MySQL 客户端程序可以用许多语言编写。为使用 C 或 C++编写的客户端或提供 C 绑定的任何语言提供了一个用 C 编写的客户端库。

+   提供了 C、C++、Eiffel、Java、Perl、PHP、Python、Ruby 和 Tcl 的 API，可以使用这些 API 在许多语言中编写 MySQL 客户端。参见第三十一章，*连接器和 API*。

+   Connector/ODBC（MyODBC）接口为使用 ODBC（开放数据库连接）连接的客户端程序提供 MySQL 支持。例如，您可以使用 MS Access 连接到您的 MySQL 服务器。客户端可在 Windows 或 Unix 上运行。Connector/ODBC 源代码可用。支持所有 ODBC 2.5 函数，以及许多其他函数。参见 MySQL Connector/ODBC 开发指南。

+   Connector/J 接口为使用 JDBC 连接的 Java 客户端程序提供 MySQL 支持。客户端可在 Windows 或 Unix 上运行。Connector/J 源代码可用。参见 MySQL Connector/J 8.0 开发指南。

+   MySQL Connector/NET 使开发人员能够轻松创建需要与 MySQL 进行安全、高性能数据连接的.NET 应用程序。它实现了所需的 ADO.NET 接口，并集成到了 ADO.NET 感知工具中。开发人员可以使用他们选择的.NET 语言构建应用程序。MySQL Connector/NET 是一个完全托管的 ADO.NET 驱动程序，使用 100%纯 C#编写。参见 MySQL Connector/NET 开发指南。

#### 本地化

+   服务器可以向客户端提供多种语言的错误消息。参见第 12.12 节，“设置错误消息语言”。

+   完全支持多种不同的字符集，包括`latin1`（cp1252）、`german`、`big5`、`ujis`、几种 Unicode 字符集等。例如，表和列名中允许使用斯堪的纳维亚字符“`å`”、“`ä`”和“`ö`”。

+   所有数据都保存在选择的字符集中。

+   排序和比较根据默认字符集和排序规则进行。在启动 MySQL 服务器时可以更改这一点（参见第 12.3.2 节，“服务器字符集和排序规则”）。要查看非常高级排序的示例，请查看捷克排序代码。MySQL 服务器支持许多不同的字符集，可以在编译时和运行时指定。

+   服务器时区可以动态更改，个别客户端可以指定自己的时区。参见第 7.1.15 节，“MySQL 服务器时区支持”。

#### 客户端和工具

+   MySQL 包括几个客户端和实用程序。这些包括命令行程序，如**mysqldump**和**mysqladmin**，以及图形程序，如 MySQL Workbench。

+   MySQL 服务器内置支持 SQL 语句来检查、优化和修复表格。这些语句可以通过命令行中的**mysqlcheck**客户端使用。MySQL 还包括**myisamchk**，一个非常快速的命令行实用程序，用于在`MyISAM`表上执行这些操作。参见第六章，*MySQL 程序*。

+   MySQL 程序可以通过`--help`或`-?`选项调用以获取在线帮助。
