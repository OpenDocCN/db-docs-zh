# 30.2 使用 sys 模式

> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-schema-usage.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-schema-usage.html)

您可以将 `sys` 模式设置为默认模式，以便引用其对象时无需使用模式名称限定：

```sql
mysql> USE sys;
Database changed
mysql> SELECT * FROM version;
+-------------+---------------+
| sys_version | mysql_version |
+-------------+---------------+
| 2.1.1       | 8.0.26-debug  |
+-------------+---------------+
```

（`version` 视图显示 `sys` 模式和 MySQL 服务器版本。）

要在默认情况下访问 `sys` 模式对象（或者只是明确），请使用模式名称限定对象引用：

```sql
mysql> SELECT * FROM sys.version;
+-------------+---------------+
| sys_version | mysql_version |
+-------------+---------------+
| 2.1.1       | 8.0.26-debug  |
+-------------+---------------+
```

`sys` 模式包含许多以不同方式总结性能模式表的视图。其中大多数视图都是成对出现的，其中一对的名称与另一对成员的名称相同，只是前面有一个 `x$` 前缀。例如，`host_summary_by_file_io` 视图按主机分组总结文件 I/O，并显示从皮秒转换为更易读值（带单位）的延迟；

```sql
mysql> SELECT * FROM sys.host_summary_by_file_io;
+------------+-------+------------+
| host       | ios   | io_latency |
+------------+-------+------------+
| localhost  | 67570 | 5.38 s     |
| background |  3468 | 4.18 s     |
+------------+-------+------------+
```

`x$host_summary_by_file_io` 视图总结相同数据，但显示未格式化的皮秒延迟：

```sql
mysql> SELECT * FROM sys.x$host_summary_by_file_io;
+------------+-------+---------------+
| host       | ios   | io_latency    |
+------------+-------+---------------+
| localhost  | 67574 | 5380678125144 |
| background |  3474 | 4758696829416 |
+------------+-------+---------------+
```

没有 `x$` 前缀的视图旨在提供更用户友好且更易于人类阅读的输出。带有 `x$` 前缀的视图显示相同值的原始形式，更适用于其他对数据执行自己处理的工具。有关非 `x$` 和 `x$` 视图之间差异的更多信息，请参阅 第 30.4.3 节“sys 模式视图”。

要检查 `sys` 模式对象定义，请使用适当的 `SHOW` 语句或 `INFORMATION_SCHEMA` 查询。例如，要检查 `session` 视图和 `format_bytes()` 函数") 函数的定义，请使用以下语句：

```sql
mysql> SHOW CREATE VIEW sys.session;
mysql> SHOW CREATE FUNCTION sys.format_bytes;
```

然而，这些语句以相对未格式化的形式显示定义。要查看具有更易读格式的对象定义，请访问 MySQL 源分发中 `scripts/sys_schema` 下找到的各个 `.sql` 文件。在 MySQL 8.0.18 之前，这些源代码是在一个单独的分发中维护的，可以从 `sys` 模式开发网站 [`github.com/mysql/mysql-sys`](https://github.com/mysql/mysql-sys) 获取。

无论是**mysqldump**还是**mysqlpump**默认情况下都不会转储`sys`模式。要生成一个转储文件，请在命令行上明确命名`sys`模式，使用以下任一命令：

```sql
mysqldump --databases --routines sys > sys_dump.sql
mysqlpump sys > sys_dump.sql
```

要从转储文件重新安装模式，请使用以下命令：

```sql
mysql < sys_dump.sql
```
