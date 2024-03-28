# 15.5 准备语句

> 原文：[`dev.mysql.com/doc/refman/8.0/en/sql-prepared-statements.html`](https://dev.mysql.com/doc/refman/8.0/en/sql-prepared-statements.html)

15.5.1 PREPARE 语句

15.5.2 EXECUTE 语句

15.5.3 DEALLOCATE PREPARE 语句

MySQL 8.0 支持服务器端准备语句。此支持利用了高效的客户端/服务器二进制协议。使用带有参数值占位符的准备语句具有以下优点：

+   每次执行语句时解析语句的开销较小。通常，数据库应用程序处理大量几乎相同的语句，只有在查询和删除的`WHERE`、更新的`SET`以及插入的`VALUES`等子句中的文字或变量值发生变化。

+   防止 SQL 注入攻击。参数值可以包含未转义的 SQL 引号和分隔符字符。

以下各节概述了准备语句的特性：

+   应用程序中的准备语句

+   SQL 脚本中的准备语句

+   PREPARE、EXECUTE 和 DEALLOCATE PREPARE 语句

+   准备语句中允许的 SQL 语法

### 应用程序中的准备语句

您可以通过客户端编程接口使用服务器端准备语句，包括用于 C 程序的 MySQL C API 客户端库，用于 Java 程序的 MySQL Connector/J，以及用于使用.NET 技术的程序的 MySQL Connector/NET。例如，C API 提供了一组构成其准备语句 API 的函数调用。请参阅 C API 准备语句接口。其他语言接口可以通过链接 C 客户端库提供支持，其中一个例子是[`mysqli`扩展](http://php.net/mysqli)，可在 PHP 5.0 及更高版本中使用。

### SQL 脚本中的准备语句

可用另一种 SQL 接口来处理准备语句。这种接口不如通过准备语句 API 使用二进制协议高效，但不需要编程，因为它直接在 SQL 级别提供：

+   当您无法使用编程接口时，可以使用它。

+   您可以从任何可以向服务器发送 SQL 语句以执行的程序中使用它，例如**mysql**客户端程序。

+   即使客户端使用旧版本的客户端库，也可以使用它。

准备好的语句的 SQL 语法旨在用于以下情况：

+   在编码之前测试准备好的语句在您的应用程序中的工作方式。

+   当您没有访问支持它们的编程 API 时使用准备好的语句。

+   与准备好的语句交互式地解决应用程序问题。

+   创建一个重现准备好的语句问题的测试用例，以便您可以提交 bug 报告。

### PREPARE、EXECUTE 和 DEALLOCATE PREPARE 语句

准备好的语句的 SQL 语法基于三个 SQL 语句：

+   `PREPARE` 为执行准备了一个语句（参见 Section 15.5.1, “PREPARE Statement”）。

+   `EXECUTE` 执行一个准备好的语句（参见 Section 15.5.2, “EXECUTE Statement”）。

+   `DEALLOCATE PREPARE` 释放一个准备好的语句（参见 Section 15.5.3, “DEALLOCATE PREPARE Statement”）。

以下示例展示了两种等效的准备好一个语句计算三角形的斜边长度的方法。

第一个示例展示了如何通过使用字符串文字来提供语句的文本来创建一个准备好的语句：

```sql
mysql> PREPARE stmt1 FROM 'SELECT SQRT(POW(?,2) + POW(?,2)) AS hypotenuse';
mysql> SET @a = 3;
mysql> SET @b = 4;
mysql> EXECUTE stmt1 USING @a, @b;
+------------+
| hypotenuse |
+------------+
|          5 |
+------------+
mysql> DEALLOCATE PREPARE stmt1;
```

第二个示例类似，但将语句的文本作为用户变量提供：

```sql
mysql> SET @s = 'SELECT SQRT(POW(?,2) + POW(?,2)) AS hypotenuse';
mysql> PREPARE stmt2 FROM @s;
mysql> SET @a = 6;
mysql> SET @b = 8;
mysql> EXECUTE stmt2 USING @a, @b;
+------------+
| hypotenuse |
+------------+
|         10 |
+------------+
mysql> DEALLOCATE PREPARE stmt2;
```

以下是另一个示例，演示如何在运行时选择要执行查询的表，方法是将表名存储为用户变量：

```sql
mysql> USE test;
mysql> CREATE TABLE t1 (a INT NOT NULL);
mysql> INSERT INTO t1 VALUES (4), (8), (11), (32), (80);

mysql> SET @table = 't1';
mysql> SET @s = CONCAT('SELECT * FROM ', @table);

mysql> PREPARE stmt3 FROM @s;
mysql> EXECUTE stmt3;
+----+
| a  |
+----+
|  4 |
|  8 |
| 11 |
| 32 |
| 80 |
+----+

mysql> DEALLOCATE PREPARE stmt3;
```

准备好的语句特定于创建它的会话。如果您终止一个会话而没有取消分配先前准备好的语句，服务器会自动取消分配它。

准备好的语句也是会话全局的。如果您在存储过程中创建了一个准备好的语句，当存储过程结束时不会取消分配它。

为防止同时创建太多准备好的语句，设置 `max_prepared_stmt_count` 系统变量。要防止使用准备好的语句，将值设置为 0。

### 允许在准备好的语句中使用的 SQL 语法

以下 SQL 语句可用作准备好的语句：

```sql
ALTER TABLE
ALTER USER
ANALYZE TABLE
CACHE INDEX
CALL
CHANGE MASTER
CHECKSUM {TABLE | TABLES}
COMMIT
{CREATE | DROP} INDEX
{CREATE | RENAME | DROP} DATABASE
{CREATE | DROP} TABLE
{CREATE | RENAME | DROP} USER
{CREATE | DROP} VIEW
DELETE
DO
FLUSH {TABLE | TABLES | TABLES WITH READ LOCK | HOSTS | PRIVILEGES
  | LOGS | STATUS | MASTER | SLAVE | USER_RESOURCES}
GRANT
INSERT
INSTALL PLUGIN
KILL
LOAD INDEX INTO CACHE
OPTIMIZE TABLE
RENAME TABLE
REPAIR TABLE
REPLACE
RESET {MASTER | SLAVE}
REVOKE
SELECT
SET
SHOW BINLOG EVENTS
SHOW CREATE {PROCEDURE | FUNCTION | EVENT | TABLE | VIEW}
SHOW {MASTER | BINARY} LOGS
SHOW {MASTER | SLAVE} STATUS
SLAVE {START | STOP}
TRUNCATE TABLE
UNINSTALL PLUGIN
UPDATE
```

不支持其他语句。

为了符合 SQL 标准，该标准规定诊断语句不可准备，MySQL 不支持以下作为准备好的语句：

+   `SHOW WARNINGS`，`SHOW COUNT(*) WARNINGS`

+   `SHOW ERRORS`，`SHOW COUNT(*) ERRORS`

+   包含对 `warning_count` 或 `error_count` 系统变量的任何引用的语句。

通常，SQL 预处理语句中不允许的语句在存储程序中也不允许。特殊情况在第 27.8 节，“存储程序的限制”中有说明。

当预处理语句引用的表或视图的元数据发生更改时，会检测到并在下次执行时自动重新准备该语句。更多信息，请参见第 10.10.3 节，“预处理语句和存储程序的缓存”。

使用预处理语句时，可以为`LIMIT`子句的参数使用占位符。参见第 15.2.13 节，“SELECT 语句”。

在与`PREPARE`和`EXECUTE`一起使用的预处理`CALL`语句中，从 MySQL 8.0 开始支持`OUT`和`INOUT`参数的占位符。请参见第 15.2.1 节，“CALL 语句”，以获取一个示例和早期版本的解决方法。无论版本如何，都可以为`IN`参数使用占位符。

预处理语句的 SQL 语法不能以嵌套方式使用。也就是说，传递给`PREPARE`的语句本身不能是`PREPARE`、`EXECUTE`或`DEALLOCATE PREPARE`语句。

预处理语句的 SQL 语法与使用预处理语句 API 调用是不同的。例如，你不能使用`mysql_stmt_prepare()` C API 函数来准备`PREPARE`、`EXECUTE`或`DEALLOCATE PREPARE`语句。

预处理语句的 SQL 语法可以在存储过程中使用，但不能在存储函数或触发器中使用。然而，不能对使用`PREPARE`和`EXECUTE`准备和执行的动态语句使用游标。游标的语句在游标创建时进行检查，因此语句不能是动态的。

预处理语句的 SQL 语法不支持多语句（即，在一个字符串中由`;`字符分隔的多个语句）。

要编写使用`CALL` SQL 语句执行包含准备语句的存储过程的 C 程序，必须启用`CLIENT_MULTI_RESULTS`标志。这是因为每个`CALL`都会返回一个结果以指示调用状态，除了存储过程内执行的语句可能返回的任何结果集。

当您调用`mysql_real_connect()`时，可以通过显式传递`CLIENT_MULTI_RESULTS`标志或隐式传递`CLIENT_MULTI_STATEMENTS`（也会启用`CLIENT_MULTI_RESULTS`）来启用`CLIENT_MULTI_RESULTS`。有关更多信息，请参见第 15.2.1 节，“CALL 语句”。
