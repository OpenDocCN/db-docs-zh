# 17.12.3 在线 DDL 空间需求

> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-online-ddl-space-requirements.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-online-ddl-space-requirements.html)

在线 DDL 操作的磁盘空间需求如下所述。这些需求不适用于立即执行的操作。

+   临时日志文件：

    临时日志文件记录了在线 DDL 操作创建索引或修改表时的并发 DML。临时日志文件根据`innodb_sort_buffer_size`的值进行扩展，最大扩展到`innodb_online_alter_log_max_size`指定的最大值。如果操作花费很长时间，且并发 DML 修改表的量超过临时日志文件的大小超过`innodb_online_alter_log_max_size`的值，那么在线 DDL 操作将因为`DB_ONLINE_LOG_TOO_BIG`错误而失败，并且未提交的并发 DML 操作将被回滚。设置较大的`innodb_online_alter_log_max_size`允许在线 DDL 操作期间进行更多的 DML，但也会延长 DDL 操作结束时表被锁定以应用已记录 DML 的时间。

    `innodb_sort_buffer_size` 变量还定义了临时日志文件读取缓冲区和写入缓冲区的大小。

+   临时排序文件：

    重建表的在线 DDL 操作在索引创建过程中将临时排序文件写入 MySQL 临时目录（Unix 上的`$TMPDIR`，Windows 上的`%TEMP%`，或由`--tmpdir`指定的目录）。临时排序文件不会在包含原始表的目录中创建。每个临时排序文件足够大以容纳一列数据，并且在其数据合并到最终表或索引时将删除每个排序文件。涉及临时排序文件的操作可能需要临时空间，等于表中数据加索引的量。如果在线 DDL 操作使用了数据目录所在文件系统上所有可用的磁盘空间，则会报告错误。

    如果 MySQL 临时目录不足以容纳排序文件，请将`tmpdir`设置为另一个目录。或者，使用`innodb_tmpdir`为在线 DDL 操作定义一个单独的临时目录。此选项旨在帮助避免由于大型临时排序文件导致的临时目录溢出。

+   中间表文件：

    一些在线 DDL 操作重新构建表时会在与原始表相同的目录中创建一个临时中间表文件。中间表文件可能需要与原始表大小相等的空间。中间表文件的名称以`#sql-ib`前缀开头，在在线 DDL 操作期间只会短暂出现。

    `innodb_tmpdir`选项不适用于中间表文件。
