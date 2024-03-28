# 15.1.6 ALTER LOGFILE GROUP Statement

> 原文：[`dev.mysql.com/doc/refman/8.0/en/alter-logfile-group.html`](https://dev.mysql.com/doc/refman/8.0/en/alter-logfile-group.html)

```sql
ALTER LOGFILE GROUP *logfile_group*
    ADD UNDOFILE '*file_name*'
    [INITIAL_SIZE [=] *size*]
    [WAIT]
    ENGINE [=] *engine_name*
```

此语句向现有日志文件组 *`logfile_group`* 添加一个名为 '*`file_name`*' 的 `UNDO` 文件。`ALTER LOGFILE GROUP` 语句只能有一个 `ADD UNDOFILE` 子句。当前不支持 `DROP UNDOFILE` 子句。

注意

所有 NDB 集群磁盘数据对象共享相同的命名空间。这意味着*每个磁盘数据对象*必须具有唯一名称（而不仅仅是给定类型的每个磁盘数据对象）。例如，您不能拥有同名的表空间和撤销日志文件，或者同名的撤销日志文件和数据文件。

可选的 `INITIAL_SIZE` 参数设置 `UNDO` 文件的初始大小（以字节为单位）；如果未指定，初始大小默认为 134217728（128 MB）。您可以选择在 *`size`* 后跟一个表示数量级的字母缩写，类似于 `my.cnf` 中使用的缩写。通常，这是 `M`（兆字节）或 `G`（千兆字节）中的一个字母。 (Bug #13116514, Bug #16104705, Bug #62858)

在 32 位系统上，`INITIAL_SIZE` 的最大支持值为 4294967296（4 GB）。 (Bug #29186)

`INITIAL_SIZE` 的最小允许值为 1048576（1 MB）。 (Bug #29574)

注意

`WAIT` 被解析但被忽略。此关键字目前没有任何效果，预计用于未来扩展。

`ENGINE` 参数（必需）确定此日志文件组使用的存储引擎，*`engine_name`* 是存储引擎的名称。目前，*`engine_name`* 的唯一接受值为“`NDBCLUSTER`”和“`NDB`”。这两个值是等效的。

这里有一个示例，假设日志文件组 `lg_3` 已经使用 `CREATE LOGFILE GROUP` 创建（参见 Section 15.1.16, “CREATE LOGFILE GROUP Statement”）：

```sql
ALTER LOGFILE GROUP lg_3
    ADD UNDOFILE 'undo_10.dat'
    INITIAL_SIZE=32M
    ENGINE=NDBCLUSTER;
```

当使用 `ALTER LOGFILE GROUP` 与 `ENGINE = NDBCLUSTER`（或者 `ENGINE = NDB`）时，在每个 NDB 集群数据节点上创建一个 `UNDO` 日志文件。您可以通过查询信息模式 `FILES` 表来验证 `UNDO` 文件是否已创建并获取有关它们的信息。例如：

```sql
mysql> SELECT FILE_NAME, LOGFILE_GROUP_NUMBER, EXTRA
 -> FROM INFORMATION_SCHEMA.FILES
 -> WHERE LOGFILE_GROUP_NAME = 'lg_3';
+-------------+----------------------+----------------+
| FILE_NAME   | LOGFILE_GROUP_NUMBER | EXTRA          |
+-------------+----------------------+----------------+
| newdata.dat |                    0 | CLUSTER_NODE=3 |
| newdata.dat |                    0 | CLUSTER_NODE=4 |
| undo_10.dat |                   11 | CLUSTER_NODE=3 |
| undo_10.dat |                   11 | CLUSTER_NODE=4 |
+-------------+----------------------+----------------+
4 rows in set (0.01 sec)
```

(参见 Section 28.3.15, “The INFORMATION_SCHEMA FILES Table”.)

内存用于`UNDO_BUFFER_SIZE`来自全局池，其大小由`SharedGlobalMemory`数据节点配置参数的值确定。这包括由`InitialLogFileGroup`数据节点配置参数的设置隐含的任何默认值。

`ALTER LOGFILE GROUP`仅适用于 NDB Cluster 的磁盘数据存储。更多信息，请参见 Section 25.6.11, “NDB Cluster Disk Data Tables”。
