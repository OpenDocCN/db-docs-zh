# 6.5.1 mysql — MySQL 命令行客户端

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql.html)

6.5.1.1 mysql 客户端选项

6.5.1.2 mysql 客户端命令

6.5.1.3 mysql 客户端日志记录

6.5.1.4 mysql 客户端服务器端帮助

6.5.1.5 从文本文件执行 SQL 语句

6.5.1.6 mysql 客户端技巧

**mysql** 是一个带有输入行编辑功能的简单 SQL shell。它支持交互和非交互使用。在交互使用时，查询结果以 ASCII 表格格式呈现。在非交互使用时（例如作为过滤器），结果以制表符分隔的格式呈现。可以使用命令选项更改输出格式。

如果由于结果集过大而导致内存不足而出现问题，请使用 `--quick` 选项。这会强制**mysql**逐行从服务器检索结果，而不是检索整个结果集并在显示之前将其缓冲在内存中。这是通过在客户端/服务器库中使用 `mysql_use_result()` C API 函数而不是 `mysql_store_result()` 来返回结果集来完成的。

注意

或者，MySQL Shell 提供对 X DevAPI 的访问。详情请参阅 MySQL Shell 8.0。

使用**mysql**非常简单。请在命令解释器的提示符下按以下方式调用它：

```sql
mysql *db_name*
```

或者：

```sql
mysql --user=*user_name* --password *db_name*
```

在这种情况下，您需要根据**mysql** 显示的提示输入密码：

```sql
Enter password: *your_password*
```

然后输入一个 SQL 语句，以 `;`、`\g` 或 `\G` 结尾，然后按 Enter 键。

输入 **Control+C** 可以中断当前语句（如果有的话），或者取消任何部分输入行。

您可以像这样在脚本文件（批处理文件）中执行 SQL 语句：

```sql
mysql *db_name* < *script.sql* > *output.tab*
```

在 Unix 上，**mysql** 客户端会将交互执行的语句记录到历史文件中。请参阅 6.5.1.3 “mysql 客户端日志记录”。
