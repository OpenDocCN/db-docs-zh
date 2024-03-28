# 17.12 InnoDB 和在线 DDL

> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-online-ddl.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-online-ddl.html)

17.12.1 在线 DDL 操作

17.12.2 在线 DDL 性能和并发性

17.12.3 在线 DDL 空间要求

17.12.4 在线 DDL 内存管理

17.12.5 配置在线 DDL 操作的并行线程

17.12.6 简化 DDL 语句的在线 DDL

17.12.7 在线 DDL 失败条件

17.12.8 在线 DDL 限制

在线 DDL 功能支持即时和原地表更改以及并发 DML。此功能的好处包括：

+   在繁忙的生产环境中提高响应性和可用性，使表在几分钟或几小时内不可用是不切实际的。

+   对于原地操作，在 DDL 操作期间使用 `LOCK` 子句调整性能和并发性之间的平衡。参见 LOCK 子句。

+   比表复制方法使用更少的磁盘空间和 I/O 开销。

注意

`ALGORITHM=INSTANT` 支持在 MySQL 8.0.12 中的 `ADD COLUMN` 和其他操作中使用。

通常，您不需要采取任何特殊措施来启用在线 DDL。默认情况下，MySQL 尽可能以立即或原地的方式执行操作，并尽量减少锁定。

您可以使用`ALTER TABLE`语句的 `ALGORITHM` 和 `LOCK` 子句来控制 DDL 操作的各个方面。这些子句位于语句末尾，与表和列规范用逗号分隔。例如：

```sql
ALTER TABLE *tbl_name* ADD PRIMARY KEY (*column*), ALGORITHM=INPLACE;
```

`LOCK` 子句可用于原地执行的操作，并且在操作期间对表的并发访问程度进行微调非常有用。仅支持 `LOCK=DEFAULT` 用于立即执行的操作。`ALGORITHM` 子句主要用于性能比较，并作为旧表复制行为的后备，以防遇到任何问题。例如：

+   为了避免在原地`ALTER TABLE`操作期间意外使表对读取、写入或两者都不可用，请在`ALTER TABLE`语句上指定一个子句，如 `LOCK=NONE`（允许读取和写入）或 `LOCK=SHARED`（允许读取）。如果所请求的并发级别不可用，则操作立即停止。

+   为了比较算法之间的性能，请运行带有`ALGORITHM=INSTANT`、`ALGORITHM=INPLACE`和`ALGORITHM=COPY`的语句。您还可以运行带有启用`old_alter_table`配置选项的语句，以强制使用`ALGORITHM=COPY`。

+   为了避免使用`ALTER TABLE`操作拷贝表格而占用服务器资源，请包含`ALGORITHM=INSTANT`或`ALGORITHM=INPLACE`。如果无法使用指定的算法，该语句将立即停止。
