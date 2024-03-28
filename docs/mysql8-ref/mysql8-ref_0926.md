# 第 15.1.28 节 DROP LOGFILE GROUP 语句

> 原文：[`dev.mysql.com/doc/refman/8.0/en/drop-logfile-group.html`](https://dev.mysql.com/doc/refman/8.0/en/drop-logfile-group.html)

```sql
DROP LOGFILE GROUP *logfile_group*
    ENGINE [=] *engine_name*
```

此语句删除名为*`logfile_group`*的日志文件组。日志文件组必须已经存在，否则将出现错误。（有关创建日志文件组的信息，请参见第 15.1.16 节，“CREATE LOGFILE GROUP 语句”。）

重要提示

在删除日志文件组之前，您必须删除所有使用该日志文件组进行`UNDO`日志记录的表空间。

必需的`ENGINE`子句提供了要删除的日志文件组所使用的存储引擎的名称。目前，*`engine_name`*的唯一允许值为`NDB`和`NDBCLUSTER`。

`DROP LOGFILE GROUP`仅适用于 NDB Cluster 的 Disk Data 存储。请参阅第 25.6.11 节，“NDB Cluster Disk Data Tables”。
