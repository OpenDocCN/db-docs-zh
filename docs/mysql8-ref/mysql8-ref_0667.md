# 11.7 注释

> 原文：[`dev.mysql.com/doc/refman/8.0/en/comments.html`](https://dev.mysql.com/doc/refman/8.0/en/comments.html)

MySQL 服务器支持三种注释样式：

+   从`#`字符到行尾。

+   从`-- `序列到行尾。在 MySQL 中，`-- `（双破折号）注释样式要求第二个破折号后至少跟随一个空格或控制字符（如空格、制表符、换行符等）。这种语法与标准 SQL 注释语法略有不同，如 1.6.2.4 节，“'--'作为注释的开始”中所讨论的。

+   从`/*`序列到后续的`*/`序列，就像在 C 编程语言中一样。这种语法使得注释可以跨越多行，因为开始和结束序列不需要在同一行上。

以下示例演示了所有三种注释样式：

```sql
mysql> SELECT 1+1;     # This comment continues to the end of line
mysql> SELECT 1+1;     -- This comment continues to the end of line
mysql> SELECT 1 /* this is an in-line comment */ + 1;
mysql> SELECT 1+
/*
this is a
multiple-line comment
*/
1;
```

嵌套注释不受支持，已被弃用；预计在未来的 MySQL 版本中将被移除。（在某些情况下，可能允许嵌套注释，但通常不建议使用。）

MySQL 服务器支持某些 C 风格注释的变体。这使您可以编写包含 MySQL 扩展的代码，但仍然是可移植的，通过使用以下形式的注释：

```sql
/*! *MySQL-specific code* */
```

在这种情况下，MySQL 服务器解析并执行注释中的代码，就像执行任何其他 SQL 语句一样，但其他 SQL 服务器应忽略这些扩展。例如，MySQL 服务器识别以下语句中的`STRAIGHT_JOIN`关键字，但其他服务器不应该：

```sql
SELECT /*! STRAIGHT_JOIN */ col1 FROM table1,table2 WHERE ...
```

如果在`!`字符后添加版本号，则仅当 MySQL 版本大于或等于指定版本号时才执行注释内的语法。以下注释中的`KEY_BLOCK_SIZE`关键字仅由 MySQL 5.1.10 或更高版本的服务器执行：

```sql
CREATE TABLE t1(a INT, KEY (a)) /*!50110 KEY_BLOCK_SIZE=1024 */;
```

版本号使用格式*`Mmmrr`*，其中*`M`*是主要版本，*`mm`*是两位数的次要版本，*`rr`*是两位数的发布号。例如：在仅由 MySQL 服务器版本 8.0.31 或更高版本运行的语句中，使用`80031`在注释中。

版本号后应跟至少一个空白字符（或注释的结尾）。从 MySQL 8.0.34 开始，如果不满足此条件，该语句将触发警告；预计这一要求将在未来的 MySQL 版本中得到严格执行。

刚才描述的注释语法适用于**mysqld**服务器解析 SQL 语句的方式。**mysql**客户端程序在将语句发送到服务器之前也会对一些语句进行解析。（它这样做是为了确定多语句输入行中的语句边界。）有关服务器和**mysql**客户端解析器之间的差异的信息，请参见 Section 6.5.1.6, “mysql Client Tips”。

以`/*!12345 ... */`格式的注释不会存储在服务器上。如果使用此格式对存储的程序进行注释，这些注释不会保留在程序主体中。

另一种 C 风格注释语法的变体用于指定优化器提示。提示注释包括在`/*`注释开头序列后跟一个`+`字符。示例：

```sql
SELECT /*+ BKA(t1) */ FROM ... ;
```

更多信息，请参见 Section 10.9.3, “Optimizer Hints”。

在多行`/* ... */`注释中使用诸如`\C`之类的**mysql**短格式命令是不受支持的。短格式命令可以在单行`/*! ... */`版本注释中工作，`/*+ ... */`优化器提示注释也可以工作，这些注释存储在对象定义中。如果担心优化器提示注释可能存储在对象定义中，以至于重新加载时使用`mysql`的转储文件会导致执行此类命令，要么使用`--binary-mode`选项调用**mysql**，要么使用除**mysql**之外的重新加载客户端。
