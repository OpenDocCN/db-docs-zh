# A.3 MySQL 8.0 FAQ：服务器 SQL 模式

> 原文：[`dev.mysql.com/doc/refman/8.0/en/faqs-sql-modes.html`](https://dev.mysql.com/doc/refman/8.0/en/faqs-sql-modes.html)

A.3.1\. 什么是服务器 SQL 模式？

A.3.2\. 服务器 SQL 模式有多少种？

A.3.3\. 如何确定服务器 SQL 模式？

A.3.4\. 模式是否依赖于数据库或连接？

A.3.5\. 严格模式的规则可以扩展吗？

A.3.6\. 严格模式会影响性能吗？

A.3.7\. 当安装 MySQL 8.0 时，默认的服务器 SQL 模式是什么？

| **A.3.1.** | 什么是服务器 SQL 模式？ |
| --- | --- |
|  | 服务器 SQL 模式定义了 MySQL 应支持的 SQL 语法以及应执行的数据验证检查类型。这使得在不同环境中使用 MySQL 以及与其他数据库服务器一起使用 MySQL 变得更加容易。MySQL 服务器将这些模式分别应用于不同的客户端。更多信息，请参阅第 7.1.11 节，“服务器 SQL 模式”。 |
| **A.3.2.** | 服务器 SQL 模式有多少种？ |
|  | 每种模式都可以独立开启和关闭。查看第 7.1.11 节，“服务器 SQL 模式”，获取可用模式的完整列表。 |
| **A.3.3.** | 如何确定服务器 SQL 模式？ |
|  | 您可以使用`--sql-mode`选项设置默认的 SQL 模式（用于**mysqld**启动）。使用语句[`SET [GLOBAL&#124;SESSION] sql_mode='*`modes`*'`](set-variable.html "15.7.6.1 变量赋值的 SET 语法")，您可以在连接内部更改设置，可以在连接本地设置，也可以全局生效。您可以通过发出`SELECT @@sql_mode`语句来检索当前模式。 |
| **A.3.4.** | 模式是否依赖于数据库或连接？ |
|  | 模式与特定数据库无关。模式可以在会话（连接）中本地设置，也可以在服务器全局设置。您可以使用[`SET [GLOBAL&#124;SESSION] sql_mode='*`modes`*'`](set-variable.html "15.7.6.1 变量赋值的 SET 语法")来更改这些设置。 |
| **A.3.5.** | 严格模式的规则可以扩展吗？ |
|  | 当我们提到*严格模式*时，我们指的是至少启用了`TRADITIONAL`、`STRICT_TRANS_TABLES`或`STRICT_ALL_TABLES`中的一个模式。选项可以组合，因此您可以向模式添加限制。有关更多信息，请参见第 7.1.11 节，“服务器 SQL 模式”。 |
| **A.3.6.** | 严格模式会影响性能吗？ |
|  | 一些设置对输入数据进行的密集验证需要比不进行验证时花费更多的时间。虽然性能影响并不那么大，如果您不需要这样的验证（也许您的应用程序已经处理了所有这些），那么 MySQL 可以让您选择禁用严格模式。但是，如果您需要它，严格模式可以提供这样的验证。 |
| **A.3.7.** | MySQL 8.0 安装时的默认服务器 SQL 模式是什么？ |
|  | MySQL 8.0 中的默认 SQL 模式包括这些模式：`ONLY_FULL_GROUP_BY`、`STRICT_TRANS_TABLES`、`NO_ZERO_IN_DATE`、`NO_ZERO_DATE`、`ERROR_FOR_DIVISION_BY_ZERO`和`NO_ENGINE_SUBSTITUTION`。有关所有可用模式和默认 MySQL 行为的信息，请参见第 7.1.11 节，“服务器 SQL 模式”。 |
