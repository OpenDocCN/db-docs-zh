# B.2 错误信息接口

> 原文：[`dev.mysql.com/doc/refman/8.0/en/error-interfaces.html`](https://dev.mysql.com/doc/refman/8.0/en/error-interfaces.html)

错误消息可能源自服务器端或客户端端，每个错误消息包括错误代码、SQLSTATE 值和消息字符串，如第 B.1 节，“错误消息来源和元素”中所述。有关服务器端、客户端端和全局（服务器和客户端共享）错误的列表，请参阅 MySQL 8.0 错误消息参考。

在程序内部进行错误检查时，请使用错误代码编号或符号，而不是错误消息字符串。消息字符串不经常更改，但有可能。此外，如果数据库管理员更改语言设置，那将影响消息字符串的语言；请参阅第 12.12 节，“设置错误消息语言”。

MySQL 中的错误信息可在服务器错误日志、SQL 级别、客户端程序内部以及命令行中获取。

+   错误日志

+   SQL 错误消息接口

+   客户端错误消息接口

+   命令行错误消息接口

### 错误日志

在服务器端，一些消息旨在写入错误日志。有关配置服务器写入日志的位置和方式的信息，请参阅第 7.4.2 节，“错误日志”。

其他服务器端错误消息旨在发送给客户端程序，并可按照客户端错误消息接口中描述的方式获取。

特定错误代码所在范围决定服务器是否将错误消息写入错误日志或发送给客户端。有关这些范围的信息，请参阅错误代码范围。

### SQL 错误消息接口

在 SQL 级别，MySQL 中有几个错误信息来源：

+   SQL 语句的警告和错误信息可以通过`SHOW WARNINGS`和`SHOW ERRORS`语句获得。`warning_count`系统变量表示错误、警告和注释的数量（如果`sql_notes`系统变量被禁用，则排除注释）。`error_count`系统变量表示错误的数量。其值不包括警告和注释。

+   `GET DIAGNOSTICS`语句可用于检查诊断区域中的诊断信息。参见 Section 15.6.7.3, “GET DIAGNOSTICS Statement”。

+   `SHOW SLAVE STATUS`语句输出包含了关于在复制服务器上发生的复制错误的信息。

+   `SHOW ENGINE INNODB STATUS`语句输出包含了关于最近的外键错误的信息，如果为`InnoDB`表的`CREATE TABLE`语句失败。

### 客户端错误消息接口

客户端程序从两个来源接收错误：

+   源自于 MySQL 客户端库内部的客户端端错误。

+   源自服务器端并由服务器发送给客户端的错误。这些错误在客户端库内接收，从而使它们可供主机客户端程序使用。

特定错误代码所处的范围确定了它是源自客户端库内部还是由客户端从服务器接收。有关这些范围的信息，请参阅错误代码范围。

无论错误是源自客户端库内部还是从服务器接收，MySQL 客户端程序通过调用客户端库中的 C API 函数获得错误代码、SQLSTATE 值、消息字符串和其他相关信息：

+   `mysql_errno()`返回 MySQL 错误代码。

+   `mysql_sqlstate()`返回 SQLSTATE 值。

+   `mysql_error()`返回消息字符串。

+   `mysql_stmt_errno()`、`mysql_stmt_sqlstate()`和`mysql_stmt_error()`是预处理语句的相应错误函数。

+   `mysql_warning_count()`返回最近语句的错误、警告和注释数量。

对于客户端库错误函数的描述，请参阅 MySQL 8.0 C API 开发人员指南。

MySQL 客户端程序可能以不同方式响应错误。客户端可能显示错误消息，以便用户采取纠正措施，内部尝试解决或重试失败的操作，或采取其他操作。例如，（使用**mysql**客户端），连接到服务器失败可能导致此消息：

```sql
$> mysql -h no-such-host
ERROR 2005 (HY000): Unknown MySQL server host 'no-such-host' (-2)
```

### 命令行错误消息接口

**perror**程序提供关于错误编号的命令行信息。请参阅 Section 6.8.2, “perror — 显示 MySQL 错误消息信息”。

```sql
$> perror 1231
MySQL error code MY-001231 (ER_WRONG_VALUE_FOR_VAR): Variable '%-.64s'
can't be set to the value of '%-.200s'
```

对于 MySQL NDB Cluster 错误，请使用**ndb_perror**。请参阅 Section 25.5.16, “ndb_perror — 获取 NDB 错误消息信息”。

```sql
$> ndb_perror 323
NDB error code 323: Invalid nodegroup id, nodegroup already existing:
Permanent error: Application error
```
