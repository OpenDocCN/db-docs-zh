# 15.1.16 CREATE LOGFILE GROUP Statement

> 原文：[`dev.mysql.com/doc/refman/8.0/en/create-logfile-group.html`](https://dev.mysql.com/doc/refman/8.0/en/create-logfile-group.html)

```sql
CREATE LOGFILE GROUP *logfile_group*
    ADD UNDOFILE '*undo_file*'
    [INITIAL_SIZE [=] *initial_size*]
    [UNDO_BUFFER_SIZE [=] *undo_buffer_size*]
    [REDO_BUFFER_SIZE [=] *redo_buffer_size*]
    [NODEGROUP [=] *nodegroup_id*]
    [WAIT]
    [COMMENT [=] '*string*']
    ENGINE [=] *engine_name*
```

此语句创建一个名为*`logfile_group`*的新日志文件组，其中包含一个名为'*`undo_file`*'的单个`UNDO`文件。`CREATE LOGFILE GROUP`语句只有一个`ADD UNDOFILE`子句。有关日志文件组命名规则，请参见第 11.2 节，“模式对象名称”。

注意

所有 NDB Cluster 磁盘数据对象共享相同的命名空间。这意味着*每个磁盘数据对象*必须具有唯一的名称（而不仅仅是给定类型的每个磁盘数据对象）。例如，您不能拥有具有相同名称的表空间和日志文件组，或者具有相同名称的表空间和数据文件。

在任何给定时间，每个 NDB Cluster 实例只能有一个日志文件组。

可选的`INITIAL_SIZE`参数设置`UNDO`文件的初始大小；如果未指定，则默认为`128M`（128 兆字节）。可选的`UNDO_BUFFER_SIZE`参数设置日志文件组的`UNDO`缓冲区使用的大小；`UNDO_BUFFER_SIZE`的默认值为`8M`（八兆字节）；此值不能超过系统内存的数量。这两个参数都以字节为单位指定。您可以选择在这两个参数中的任一个或两个后面跟随一个表示数量级的单个字母缩写，类似于`my.cnf`中使用的那些字母之一。通常，这是`M`（表示兆字节）或`G`（表示千兆字节）中的一个。

用于`UNDO_BUFFER_SIZE`的内存来自由`SharedGlobalMemory`数据节点配置参数的值确定的全局池。这包括由`InitialLogFileGroup`数据节点配置参数的设置隐含的此选项的任何默认值。

`UNDO_BUFFER_SIZE`的最大允许值为 629145600（600 MB）。

在 32 位系统上，`INITIAL_SIZE`的最大支持值为 4294967296（4 GB）。 （Bug＃29186）

`INITIAL_SIZE`的最小允许值为 1048576（1 MB）。

`ENGINE` 选项确定了该日志文件组要使用的存储引擎，*`engine_name`* 是存储引擎的名称。在 MySQL 8.0 中，这必须是 `NDB`（或 `NDBCLUSTER`）。如果未设置 `ENGINE`，MySQL 尝试使用由 `default_storage_engine` 服务器系统变量（以前是 `storage_engine`）指定的引擎。无论如何，如果未指定引擎为 `NDB` 或 `NDBCLUSTER`，`CREATE LOGFILE GROUP` 语句看起来成功了，但实际上未能创建日志文件组，如下所示：

```sql
mysql> CREATE LOGFILE GROUP lg1
 ->     ADD UNDOFILE 'undo.dat' INITIAL_SIZE = 10M;
Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql> SHOW WARNINGS;
+-------+------+------------------------------------------------------------------------------------------------+
| Level | Code | Message                                                                                        |
+-------+------+------------------------------------------------------------------------------------------------+
| Error | 1478 | Table storage engine 'InnoDB' does not support the create option 'TABLESPACE or LOGFILE GROUP' |
+-------+------+------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)

mysql> DROP LOGFILE GROUP lg1 ENGINE = NDB;
ERROR 1529 (HY000): Failed to drop LOGFILE GROUP 
mysql> CREATE LOGFILE GROUP lg1
 ->     ADD UNDOFILE 'undo.dat' INITIAL_SIZE = 10M
 ->     ENGINE = NDB;
Query OK, 0 rows affected (2.97 sec)
```

`CREATE LOGFILE GROUP` 语句在命名非`NDB`存储引擎时实际上并不会返回错误，而是看起来成功了，这是一个已知问题，我们希望在未来的 NDB Cluster 版本中解决。

*`REDO_BUFFER_SIZE`*、`NODEGROUP`、`WAIT` 和 `COMMENT` 被解析但被忽略，在 MySQL 8.0 中没有任何效果。这些选项是为了未来的扩展而设计的。

当与 `ENGINE [=] NDB` 一起使用时，在每个 Cluster 数据节点上创建一个日志文件组和相关的 `UNDO` 日志文件。您可以通过查询信息模式 `FILES` 表来验证 `UNDO` 文件是否已创建并获取有关它们的信息。例如：

```sql
mysql> SELECT LOGFILE_GROUP_NAME, LOGFILE_GROUP_NUMBER, EXTRA
 -> FROM INFORMATION_SCHEMA.FILES
 -> WHERE FILE_NAME = 'undo_10.dat';
+--------------------+----------------------+----------------+
| LOGFILE_GROUP_NAME | LOGFILE_GROUP_NUMBER | EXTRA          |
+--------------------+----------------------+----------------+
| lg_3               |                   11 | CLUSTER_NODE=3 |
| lg_3               |                   11 | CLUSTER_NODE=4 |
+--------------------+----------------------+----------------+
2 rows in set (0.06 sec)
```

`CREATE LOGFILE GROUP` 仅在 NDB Cluster 的 Disk Data 存储中有用。请参阅 第 25.6.11 节，“NDB Cluster Disk Data Tables”。
