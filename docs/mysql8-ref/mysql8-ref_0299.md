# 7.6.5 ddl_rewriter 插件

> 原文：[`dev.mysql.com/doc/refman/8.0/en/ddl-rewriter.html`](https://dev.mysql.com/doc/refman/8.0/en/ddl-rewriter.html)

7.6.5.1 安装或卸载 ddl_rewriter

7.6.5.2 ddl_rewriter 插件选项

MySQL 8.0.16 及更高版本包括一个`ddl_rewriter`插件，该插件在服务器解析和执行之前修改接收到的`CREATE TABLE`语句。该插件删除`ENCRYPTION`、`DATA DIRECTORY`和`INDEX DIRECTORY`子句，这在从加密数据库或将表存储在数据目录之外的数据库创建的 SQL 转储文件中恢复表时可能会有所帮助。例如，该插件可以使这些转储文件能够恢复到未加密实例或在路径在数据目录之外不可访问的环境中。

在使用`ddl_rewriter`插件之前，请根据第 7.6.5.1 节“安装或卸载 ddl_rewriter”中提供的说明进行安装。

`ddl_rewriter`在服务器解析之前检查接收到的 SQL 语句，根据这些条件对其进行重写：

+   `ddl_rewriter`仅考虑`CREATE TABLE`语句，只有当它们是独立的语句，并且出现在输入行的开头或准备语句文本的开头时。`ddl_rewriter`不考虑存储程序定义中的`CREATE TABLE`语句。语句可以跨越多行。

+   在进行重写考虑的语句中，以下子句的实例将被重写，并且每个实例将被单个空格替换：

    +   `ENCRYPTION`

    +   `DATA DIRECTORY`（在表和分区级别）

    +   `INDEX DIRECTORY`（在表和分区级别）

+   重写不依赖于大小写。

如果`ddl_rewriter`重写了一个语句，它会生成一个警告：

```sql
mysql> CREATE TABLE t (i INT) DATA DIRECTORY '/var/mysql/data';
Query OK, 0 rows affected, 1 warning (0.03 sec)

mysql> SHOW WARNINGS\G
*************************** 1\. row ***************************
  Level: Note
   Code: 1105
Message: Query 'CREATE TABLE t (i INT) DATA DIRECTORY '/var/mysql/data''
         rewritten to 'CREATE TABLE t (i INT) ' by a query rewrite plugin 1 row in set (0.00 sec)
```

如果启用了一般查询日志或二进制日志，服务器会在任何`ddl_rewriter`重写后将语句写入其中。

安装后，`ddl_rewriter`会为跟踪插件内存使用情况暴露性能模式`memory/rewriter/ddl_rewriter`。请参阅第 29.12.20.10 节“内存摘要表”
