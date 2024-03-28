# 29.4.10 确定仪器化的内容

> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-instrumentation-checking.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-instrumentation-checking.html)

通过检查`setup_instruments`表，始终可以确定性能模式包含哪些仪器。例如，要查看为`InnoDB`存储引擎仪器化的文件相关事件，请使用以下查询：

```sql
mysql> SELECT NAME, ENABLED, TIMED
       FROM performance_schema.setup_instruments
       WHERE NAME LIKE 'wait/io/file/innodb/%';
+-------------------------------------------------+---------+-------+
| NAME                                            | ENABLED | TIMED |
+-------------------------------------------------+---------+-------+
| wait/io/file/innodb/innodb_tablespace_open_file | YES     | YES   |
| wait/io/file/innodb/innodb_data_file            | YES     | YES   |
| wait/io/file/innodb/innodb_log_file             | YES     | YES   |
| wait/io/file/innodb/innodb_temp_file            | YES     | YES   |
| wait/io/file/innodb/innodb_arch_file            | YES     | YES   |
| wait/io/file/innodb/innodb_clone_file           | YES     | YES   |
+-------------------------------------------------+---------+-------+
```

此文档未详细描述仪器化的内容，原因有几个：

+   仪器化的是服务器代码。对这些代码的更改经常发生，这也会影响仪器的集合。

+   不可能列出所有的仪器，因为它们有数百个。

+   如前所述，可以通过查询`setup_instruments`表来找到。这些信息始终是针对您的 MySQL 版本最新的，还包括您可能安装的不属于核心服务器的仪器化插件的仪器化，并可被自动化工具使用。
