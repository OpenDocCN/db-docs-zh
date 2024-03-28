# 29.11 性能模式通用表特性

> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-table-characteristics.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-table-characteristics.html)

`performance_schema`数据库的名称是小写的，其中的表名也是小写的。查询应该以小写指定名称。

`performance_schema`数据库中的许多表是只读的，不能被修改：

```sql
mysql> TRUNCATE TABLE performance_schema.setup_instruments;
ERROR 1683 (HY000): Invalid performance_schema usage.
```

一些设置表具有可以修改以影响性能模式操作的列；有些还允许插入或删除行。允许截断以清除收集的事件，因此可以在包含这些类型信息的表上使用`TRUNCATE TABLE`，例如以`events_waits_`为前缀命名的表。

摘要表可以使用`TRUNCATE TABLE`进行截断。通常，效果是将摘要列重置为 0 或`NULL`，而不是删除行。这使您可以清除收集的值并重新开始聚合。例如，在进行运行时配置更改后，这可能很有用。个别摘要表部分中指出了此截断行为的异常情况。

权限与其他数据库和表相同：

+   要从`performance_schema`表中检索，您必须具有`SELECT`权限。

+   要更改可以修改的列，您必须具有`UPDATE`权限。

+   要截断可以截断的表，您必须具有`DROP`权限。

因为性能模式表只适用于有限的一组特权，尝试使用`GRANT ALL`作为在数据库或表级别授予权限的简写会导致错误：

```sql
mysql> GRANT ALL ON performance_schema.*
       TO 'u1'@'localhost';
ERROR 1044 (42000): Access denied for user 'root'@'localhost'
to database 'performance_schema'
mysql> GRANT ALL ON performance_schema.setup_instruments
       TO 'u2'@'localhost';
ERROR 1044 (42000): Access denied for user 'root'@'localhost'
to database 'performance_schema'
```

相反，授予确切所需的权限：

```sql
mysql> GRANT SELECT ON performance_schema.*
       TO 'u1'@'localhost';
Query OK, 0 rows affected (0.03 sec)

mysql> GRANT SELECT, UPDATE ON performance_schema.setup_instruments
       TO 'u2'@'localhost';
Query OK, 0 rows affected (0.02 sec)
```
