> 原文：[`dev.mysql.com/doc/refman/8.0/en/optimize-table.html`](https://dev.mysql.com/doc/refman/8.0/en/optimize-table.html)

#### 15.7.3.4 OPTIMIZE TABLE 语句

```sql
OPTIMIZE [NO_WRITE_TO_BINLOG | LOCAL]
    TABLE *tbl_name* [, *tbl_name*] ...
```

`OPTIMIZE TABLE`重新组织表数据和相关索引数据的物理存储，以减少存储空间并在访问表时提高 I/O 效率。对每个表所做的确切更改取决于该表使用的存储引擎。

在这些情况下使用`OPTIMIZE TABLE`，取决于表的类型：

+   在对启用了`innodb_file_per_table`选项创建了自己的.ibd 文件的`InnoDB`表上进行大量插入、更新或删除操作之后。表和索引将重新组织，并且磁盘空间可以被回收供操作系统使用。

+   在对`InnoDB`表中的`FULLTEXT`索引的列进行大量插入、更新或删除操作之后。首先设置配置选项`innodb_optimize_fulltext_only=1`。为了保持索引维护时间在合理范围内，设置`innodb_ft_num_word_optimize`选项以指定要更新搜索索引中的单词数量，并运行一系列`OPTIMIZE TABLE`语句，直到搜索索引完全更新。

+   在删除`MyISAM`或`ARCHIVE`表的大部分内容，或对具有可变长度行的`MyISAM`或`ARCHIVE`表进行许多更改（具有`VARCHAR`、`VARBINARY`、`BLOB`或`TEXT`列的表）。已删除的行将保留在链表中，并且后续的`INSERT`操作将重用旧的行位置。您可以使用`OPTIMIZE TABLE`来回收未使用的空间并对数据文件进行碎片整理。在对表进行大量更改后，此语句有时也可以显著改善使用该表的语句的性能。

此语句需要表的`SELECT`和`INSERT`权限。

`OPTIMIZE TABLE`适用于`InnoDB`、`MyISAM`和`ARCHIVE`表。`OPTIMIZE TABLE`也支持内存中动态列的`NDB`表。它不适用于内存表的固定宽度列，也不适用于磁盘数据表。可以使用`--ndb-optimization-delay`来调整 NDB Cluster 表上`OPTIMIZE`的性能，该选项控制`OPTIMIZE TABLE`处理批处理行之间等待的时间长度。有关更多信息，请参见第 25.2.7.11 节，“NDB Cluster 8.0 中解决的以前的 NDB Cluster 问题”。

对于 NDB Cluster 表，`OPTIMIZE TABLE`可以被（例如）终止执行`OPTIMIZE`操作的 SQL 线程所中断。

默认情况下，`OPTIMIZE TABLE`不适用于使用任何其他存储引擎创建的表，并返回指示此不支持的结果。您可以通过使用`--skip-new`选项启动**mysqld**来使`OPTIMIZE TABLE`适用于其他存储引擎。在这种情况下，`OPTIMIZE TABLE`只是映射到`ALTER TABLE`。

此语句不适用于视图。

`OPTIMIZE TABLE`支持分区表。有关在分区表和表分区中使用此语句的信息，请参见第 26.3.4 节，“分区的维护”。

默认情况下，服务器会将`OPTIMIZE TABLE`语句写入二进制日志，以便在副本中复制。要禁止记录日志，请指定可选的`NO_WRITE_TO_BINLOG`关键字或其别名`LOCAL`。

+   OPTIMIZE TABLE 输出

+   InnoDB 详细信息

+   MyISAM 详细信息

+   其他考虑因素

##### OPTIMIZE TABLE 输出

`OPTIMIZE TABLE` 返回一个结果集，其中包含下表所示的列。

| 列 | 值 |
| --- | --- |
| `Table` | 表名 |
| `Op` | 始终为 `optimize` |
| `Msg_type` | `status`, `error`, `info`, `note`, 或 `warning` |
| `Msg_text` | 一个信息性消息 |

`OPTIMIZE TABLE` 表捕获并抛出在从旧文件复制表统计信息到新创建的文件时发生的任何错误。例如，如果`.MYD`或`.MYI`文件的所有者用户 ID 与 **mysqld** 进程的用户 ID 不同，`OPTIMIZE TABLE` 会生成“无法更改文件所有权”错误，除非 **mysqld** 是由 `root` 用户启动的。

##### InnoDB 详情

对于 `InnoDB` 表，`OPTIMIZE TABLE` 被映射为 `ALTER TABLE ... FORCE`，该操作重建表以更新索引统计信息并释放聚簇索引中未使用的空间。当你在 `InnoDB` 表上运行 `OPTIMIZE TABLE` 时，输出中会显示这一点：

```sql
mysql> OPTIMIZE TABLE foo;
+----------+----------+----------+-------------------------------------------------------------------+
| Table    | Op       | Msg_type | Msg_text                                                          |
+----------+----------+----------+-------------------------------------------------------------------+
| test.foo | optimize | note     | Table does not support optimize, doing recreate + analyze instead |
| test.foo | optimize | status   | OK                                                                |
+----------+----------+----------+-------------------------------------------------------------------+
```

`OPTIMIZE TABLE` 使用在线 DDL 来对常规和分区的 `InnoDB` 表进行重建，从而减少并发 DML 操作的停机时间。由 `OPTIMIZE TABLE` 触发的表重建是就地完成的。在操作的准备阶段和提交阶段仅短暂地获取独占表锁。在准备阶段，元数据被更新并创建一个中间表。在提交阶段，表元数据更改被提交。

在以下条件下，`OPTIMIZE TABLE` 使用表复制方法重建表：

+   当启用 `old_alter_table` 系统变量时。

+   当服务器使用 `--skip-new` 选项启动时。

使用在线 DDL 的 `OPTIMIZE TABLE` 不支持包含 `FULLTEXT` 索引的 `InnoDB` 表。而是使用表复制方法。

`InnoDB` 使用页面分配方法存储数据，并且不像传统存储引擎（如`MyISAM`）那样受到碎片化的影响。在考虑是否运行优化时，请考虑服务器预计要处理的事务工作负载：

+   一定程度的碎片化是可以预期的。`InnoDB`只将页面填充到 93%的容量，以便为更新留出空间，而无需分割页面。

+   删除操作可能会留下间隙，导致页面填充不足，这可能值得优化表格。

+   对行的更新通常会在同一页面内重写数据，取决于数据类型和行格式，在有足够空间的情况下。请参阅 Section 17.9.1.5, “How Compression Works for InnoDB Tables” 和 Section 17.10, “InnoDB Row Formats”。

+   高并发工作负载可能会随着时间的推移在索引中留下间隙，因为`InnoDB`通过其 MVCC 机制保留了相同数据的多个版本。请参阅 Section 17.3, “InnoDB Multi-Versioning”。

##### MyISAM 详细信息

对于`MyISAM`表，`OPTIMIZE TABLE` 的工作方式如下：

1.  如果表中有已删除或已分割的行，请修复表格。

1.  如果索引页面未排序，请对其进行排序。

1.  如果表格的统计数据不是最新的（且无法通过对索引进行排序来修复），请更新它们。

##### 其他考虑事项

`OPTIMIZE TABLE` 用于在线执行常规和分区的`InnoDB`表。否则，在运行`OPTIMIZE TABLE` 时，MySQL 会锁定表格。

`OPTIMIZE TABLE` 不会对 R-tree 索引进行排序，例如`POINT`列上的空间索引。（Bug #23578）
