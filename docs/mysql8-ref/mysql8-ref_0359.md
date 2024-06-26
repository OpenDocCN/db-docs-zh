# 8.1.7 客户端编程安全准则

> 原文：[`dev.mysql.com/doc/refman/8.0/en/secure-client-programming.html`](https://dev.mysql.com/doc/refman/8.0/en/secure-client-programming.html)

访问 MySQL 的客户端应用程序应遵循以下准则，以避免错误解释外部数据或暴露敏感信息。

+   正确处理外部数据

+   正确处理 MySQL 错误消息

#### 正确处理外部数据

访问 MySQL 的应用程序不应信任用户输入的任何数据，用户可以尝试通过在网络表单、URL 或您构建的任何应用程序中输入特殊或转义字符序列来欺骗您的代码。确保如果用户尝试通过在表单中输入类似`；删除数据库 mysql；`的内容来执行 SQL 注入，您的应用程序仍然保持安全。这是一个极端的例子，但如果您不为此做好准备，可能会发生大规模的安全漏洞和数据丢失。

一个常见的错误是仅保护字符串数据值。请记得检查数值数据。如果一个应用程序生成类似`SELECT * FROM table WHERE ID=234`的查询，当用户输入值`234`时，用户可以输入值`234 OR 1=1`来导致应用程序生成查询`SELECT * FROM table WHERE ID=234 OR 1=1`。结果，服务器检索表中的每一行。这暴露了每一行并导致服务器负载过大。保护免受此类攻击的最简单方法是在数值常量周围使用单引号：`SELECT * FROM table WHERE ID='234'`。如果用户输入额外信息，它都将成为字符串的一部分。在数值上下文中，MySQL 会自动将此字符串转换为数字并剥离其中的任何尾随非数字字符。

有时人们认为如果数据库只包含公开可用的数据，则无需保护。这是不正确的。即使可以显示数据库中的任何行，您仍应防范拒绝服务攻击（例如，基于前一段中导致服务器浪费资源的技术）。否则，您的服务器将对合法用户不响应。

检查清单：

+   启用严格的 SQL 模式，告诉服务器更严格地接受哪些数据值。请参阅第 7.1.11 节，“服务器 SQL 模式”。

+   尝试在所有的网络表单中输入单引号和双引号（`'` 和 `"`）。如果出现任何 MySQL 错误，请立即调查问题。

+   尝试通过向动态 URL 添加`%22`（`"`）、`%23`（`#`）和`%27`（`'`）来修改它们。

+   尝试将动态 URL 中的数据类型从数字类型修改为字符类型，使用前面示例中显示的字符。您的应用程序应对这些攻击和类似攻击保持安全。

+   尝试在数字字段中输入字符、空格和特殊符号而不是数字。您的应用程序应在传递给 MySQL 之前将它们删除，否则会生成错误。将未经检查的值传递给 MySQL 非常危险！

+   在将数据传递给 MySQL 之前检查数据的大小。

+   让应用程序使用与您用于管理目的不同的用户名连接到数据库。不要给予应用程序任何不需要的访问权限。

许多应用程序编程接口提供了一种方法来转义数据值中的特殊字符。正确使用可以防止应用程序用户输入导致应用程序生成具有不同效果的语句：

+   MySQL SQL 语句：使用 SQL 预处理语句，并仅通过占位符接受数据值；参见 第 15.5 节，“预处理语句”。

+   MySQL C API：使用 `mysql_real_escape_string_quote()` API 调用。或者，使用 C API 预处理语句接口，并仅通过占位符接受数据值；参见 C API 预处理语句接口。

+   MySQL++：对查询流使用 `escape` 和 `quote` 修饰符。

+   PHP：使用 `mysqli` 或 `pdo_mysql` 扩展，而不是旧的 `ext/mysql` 扩展。首选的 API 支持改进的 MySQL 认证协议和密码，以及带有占位符的预处理语句。另请参阅 MySQL 和 PHP。

    如果必须使用旧的 `ext/mysql` 扩展，则用 `mysql_real_escape_string_quote()` 函数进行转义，而不是 `mysql_escape_string()` 或 `addslashes()`，因为只有 `mysql_real_escape_string_quote()` 是字符集感知的；在使用（无效的）多字节字符集时，其他函数可能会被“绕过”。

+   Perl DBI：使用占位符或 `quote()` 方法。

+   Java JDBC：使用 `PreparedStatement` 对象和占位符。

其他编程接口可能具有类似的功能。

#### 适当处理 MySQL 错误消息

应用程序有责任拦截由执行与 MySQL 数据库服务器的 SQL 语句导致的错误，并适当处理这些错误。

MySQL 错误返回的信息并非毫无意义，因为这些信息对于使用应用程序调试 MySQL 至关重要。例如，要调试一个常见的 10 路连接`SELECT`语句，如果不提供涉及问题的数据库、表和其他对象的信息，几乎是不可能的。因此，MySQL 错误有时必须包含对这些对象名称的引用。

当应用程序从 MySQL 接收到这样的错误时，一个简单但不安全的方法是拦截并将其原样显示给客户端。然而，揭示错误信息是一种已知的应用程序漏洞类型（[CWE-209](http://cwe.mitre.org/data/definitions/209.html)），应用程序开发人员必须确保应用程序没有这种漏洞。

例如，一个显示如下消息的应用程序向客户端暴露了数据库名称和表名称，这是客户端可能会尝试利用的信息：

```sql
ERROR 1146 (42S02): Table 'mydb.mytable' doesn't exist
```

相反，当应用程序从 MySQL 接收到这样的错误时，正确的行为是记录适当的信息，包括错误信息，仅供信任人员访问的安全审计位置。应用程序可以向用户返回更通用的信息，比如“内部错误”。
