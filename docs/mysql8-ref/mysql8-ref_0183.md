> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-server-side-help.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-server-side-help.html)

#### 6.5.1.4 mysql 客户端服务器端帮助

```sql
mysql> help *search_string*
```

如果您向`help`命令提供参数，**mysql**将其用作搜索字符串，以从 MySQL 参考手册的内容中访问服务器端帮助。该命令的正确操作要求`mysql`数据库中的帮助表已初始化为帮助主题信息（请参阅 Section 7.1.17, “服务器端帮助支持”）。

如果搜索字符串没有匹配项，搜索失败：

```sql
mysql> help me

Nothing found
Please try to run 'help contents' for a list of all accessible topics
```

使用**帮助目录**查看帮助类别列表：

```sql
mysql> help contents
You asked for help about help category: "Contents"
For more information, type 'help <item>', where <item> is one of the
following categories:
   Account Management
   Administration
   Data Definition
   Data Manipulation
   Data Types
   Functions
   Functions and Modifiers for Use with GROUP BY
   Geographic Features
   Language Structure
   Plugins
   Storage Engines
   Stored Routines
   Table Maintenance
   Transactions
   Triggers
```

如果搜索字符串匹配多个项目，**mysql**将显示匹配主题的列表：

```sql
mysql> help logs
Many help items for your request exist.
To make a more specific request, please type 'help <item>',
where <item> is one of the following topics:
   SHOW
   SHOW BINARY LOGS
   SHOW ENGINE
   SHOW LOGS
```

使用一个主题作为搜索字符串，查看该主题的帮助条目：

```sql
mysql> help show binary logs
Name: 'SHOW BINARY LOGS'
Description:
Syntax:
SHOW BINARY LOGS
SHOW MASTER LOGS

Lists the binary log files on the server. This statement is used as
part of the procedure described in [purge-binary-logs], that shows how
to determine which logs can be purged.
```

```sql
mysql> SHOW BINARY LOGS;
+---------------+-----------+-----------+
| Log_name      | File_size | Encrypted |
+---------------+-----------+-----------+
| binlog.000015 |    724935 | Yes       |
| binlog.000016 |    733481 | Yes       |
+---------------+-----------+-----------+
```

搜索字符串可以包含通配符字符`%`和`_`。这些字符的含义与使用`LIKE`运算符执行的模式匹配操作相同。例如，`HELP rep%`返回以`rep`开头的主题列表：

```sql
mysql> HELP rep%
Many help items for your request exist.
To make a more specific request, please type 'help <item>',
where <item> is one of the following
topics:
   REPAIR TABLE
   REPEAT FUNCTION
   REPEAT LOOP
   REPLACE
   REPLACE FUNCTION
```
