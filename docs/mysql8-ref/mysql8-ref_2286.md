# A.13 MySQL 8.0 FAQ：C API，libmysql

> 原文：[`dev.mysql.com/doc/refman/8.0/en/faqs-c-api.html`](https://dev.mysql.com/doc/refman/8.0/en/faqs-c-api.html)

关于 MySQL C API 和 libmysql 的常见问题。

A.13.1\. 什么是“MySQL 原生 C API”？典型的优势和用例是什么？

A.13.2\. 我应该使用哪个版本的 libmysql？

A.13.3\. 如果我想使用“NoSQL” X DevAPI 怎么办？

A.13.4\. 如何下载 libmysql？

A.13.5\. 文档在哪里？

A.13.6\. 如何报告错误？

A.13.7\. 是否可以自己编译库？

| **A.13.1.** | 什么是“MySQL 原生 C API”？典型的优势和用例是什么？ |
| --- | --- |
|  | libmysql 是一个基于 C 的 API，您可以在 C 应用程序中使用它与 MySQL 数据库服务器连接。它本身也被用作标准数据库 API（如 ODBC、Perl 的 DBI 和 Python 的 DB API）驱动程序的基础。 |
| **A.13.2.** | 我应该使用哪个版本的 libmysql？ |
|  | 对于 MySQL 8.0、5.7、5.6 和 5.5，我们推荐使用 libmysql 8.0。 |
| **A.13.3.** | 如果我想使用“NoSQL” X DevAPI 怎么办？ |
|  | 对于 MySQL 8.0 的 C 语言和 X DevApi 文档存储，我们推荐使用 MySQL Connector/C++。Connector/C++ 8.0 具有兼容的 C 头文件。（这不适用于 MySQL 5.7 或更早版本。） |
| **A.13.4.** | 如何下载 libmysql？ |
|  |

+   Linux：客户端实用程序包可从[MySQL 社区服务器](https://dev.mysql.com/downloads/mysql/)下载页面获取。

+   Repos：客户端实用程序包可从[Yum](https://dev.mysql.com/downloads/repo/yum/)、[APT](https://dev.mysql.com/downloads/repo/apt/)、[SuSE repositories](https://dev.mysql.com/downloads/repo/suse/)获取。

+   Windows：客户端实用程序包可从[Windows Installer](https://dev.mysql.com/downloads/installer/)获取。

|

| **A.13.5.** | 文档在哪里？ |
| --- | --- |
|  | 参阅 MySQL 8.0 C API 开发人员指南。 |
| **A.13.6.** | 如何报告错误？ |
|  | 请向我们的[Bugs Database](https://bugs.mysql.com/)报告您观察到的任何错误或不一致性。选择如图所示的 C API 客户端。 |
| **A.13.7.** | 是否可以自己编译库？ |
|  | 编译 MySQL Server 也会编译 libmysqlclient；没有办法只编译 libmysqlclient。有关相关信息，请参阅 MySQL C API 实现。 |
