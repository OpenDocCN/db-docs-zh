> 原文：[`dev.mysql.com/doc/refman/8.0/en/temporary-files.html`](https://dev.mysql.com/doc/refman/8.0/en/temporary-files.html)

#### B.3.3.5 MySQL 存储临时文件的位置

在 Unix 上，MySQL 使用`TMPDIR`环境变量的值作为存储临时文件的目录路径名。如果未设置`TMPDIR`，MySQL 使用系统默认值，通常为`/tmp`、`/var/tmp`或`/usr/tmp`。

在 Windows 上，MySQL 按顺序检查`TMPDIR`、`TEMP`和`TMP`环境变量的值。对于设置的第一个值，MySQL 使用它并不再检查剩下的值。如果`TMPDIR`、`TEMP`或`TMP`都未设置，则 MySQL 使用 Windows 系统默认值，通常为`C:\windows\temp\`。

如果包含临时文件目录的文件系统太小，可以使用**mysqld** `--tmpdir`选项指定一个文件系统中有足够空间的目录。

`--tmpdir`选项可以设置为一个以轮询方式使用的多个路径列表。在 Unix 上应使用冒号字符（`:`）分隔路径，在 Windows 上应使用分号字符（`;`）分隔路径。

注意

为了有效地分散负载，这些路径应位于不同的*物理*磁盘上，而不是同一磁盘的不同分区上。

如果 MySQL 服务器充当副本，则可以设置系统变量`replica_load_tmpdir`（从 MySQL 8.0.26 开始）或`slave_load_tmpdir`（在 MySQL 8.0.26 之前）来指定一个单独的目录，用于保存在复制`LOAD DATA`语句时使用的临时文件。此目录应位于基于磁盘的文件系统（而不是基于内存的文件系统）中，以便用于复制 LOAD DATA 的临时文件可以在机器重新启动时保留。该目录也不应该是在系统启动过程中被操作系统清除的目录。但是，如果临时文件已被删除，则复制现在可以在重新启动后继续。

MySQL 安排在**mysqld**终止时删除临时文件。在支持的平台上（如 Unix），这是通过在打开文件后取消链接文件来完成的。这样做的缺点是文件名不会出现在目录列表中，您看不到一个填满临时文件目录所在文件系统的大临时文件。（在这种情况下，**lsof +L1**可能有助于识别与**mysqld**关联的大文件。）

在排序（`ORDER BY`或`GROUP BY`）时，MySQL 通常使用一个或两个临时文件。所需的最大磁盘空间由以下表达式确定：

```sql
(length of what is sorted + sizeof(row pointer))
* number of matched rows
* 2
```

行指针大小通常为四个字节，但将来可能会增长以适应真正大的表。

对于某些语句，MySQL 会创建临时的 SQL 表，这些表不是隐藏的，名称以`#sql`开头。

一些`SELECT`查询会创建临时的 SQL 表来保存中间结果。

重建表的 DDL 操作，如果不使用`ALGORITHM=INPLACE`技术在线执行，则会在与原始表相同的目录中创建原始表的临时副本。

在线 DDL 操作可能会使用临时日志文件来记录并发 DML，创建索引时使用临时排序文件，以及在重建表时使用临时中间表文件。更多信息，请参见 Section 17.12.3, “Online DDL Space Requirements”。

`InnoDB`用户创建的临时表和磁盘上的内部临时表会在名为`ibtmp1`的 MySQL 数据目录中的临时表空间文件中创建。更多信息，请参见 Section 17.6.3.5, “Temporary Tablespaces”。

另请参见 Section 17.15.7, “InnoDB INFORMATION_SCHEMA Temporary Table Info Table”。

可选的`EXTENDED`修饰符会导致`SHOW TABLES`列出由失败的`ALTER TABLE`语句创建的隐藏表。参见 Section 15.7.7.39, “SHOW TABLES Statement”。
