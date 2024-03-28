# 9.3 备份和恢复策略示例

> 原文：[`dev.mysql.com/doc/refman/8.0/en/backup-strategy-example.html`](https://dev.mysql.com/doc/refman/8.0/en/backup-strategy-example.html)

9.3.1 建立备份策略

9.3.2 使用备份进行恢复

9.3.3 备份策略摘要

本节讨论了一种备份执行程序，使您能够在几种崩溃后恢复数据：

+   操作系统崩溃

+   电源故障

+   文件系统崩溃

+   硬件问题（硬盘、主板等）

示例命令不包括像`--user`和`--password`这样的选项，用于**mysqldump**和**mysql**客户端程序。您应根据需要包含这些选项，以使客户端程序能够连接到 MySQL 服务器。

假设数据存储在支持事务和自动崩溃恢复的`InnoDB`存储引擎中。同时假设 MySQL 服务器在崩溃时处于负载状态。如果不是这样，就永远不需要恢复。

对于操作系统崩溃或电源故障的情况，我们可以假设 MySQL 的磁盘数据在重新启动后是可用的。由于崩溃，`InnoDB`数据文件可能不包含一致的数据，但`InnoDB`会读取其日志，并在其中找到未刷新到数据文件的待处理已提交和未提交事务列表。`InnoDB`会自动回滚那些未提交的事务，并将已提交的事务刷新到其数据文件中。关于此恢复过程的信息通过 MySQL 错误日志传达给用户。以下是一个示例日志摘录：

```sql
InnoDB: Database was not shut down normally.
InnoDB: Starting recovery from log files...
InnoDB: Starting log scan based on checkpoint at
InnoDB: log sequence number 0 13674004
InnoDB: Doing recovery: scanned up to log sequence number 0 13739520
InnoDB: Doing recovery: scanned up to log sequence number 0 13805056
InnoDB: Doing recovery: scanned up to log sequence number 0 13870592
InnoDB: Doing recovery: scanned up to log sequence number 0 13936128
...
InnoDB: Doing recovery: scanned up to log sequence number 0 20555264
InnoDB: Doing recovery: scanned up to log sequence number 0 20620800
InnoDB: Doing recovery: scanned up to log sequence number 0 20664692
InnoDB: 1 uncommitted transaction(s) which must be rolled back
InnoDB: Starting rollback of uncommitted transactions
InnoDB: Rolling back trx no 16745
InnoDB: Rolling back of trx no 16745 completed
InnoDB: Rollback of uncommitted transactions completed
InnoDB: Starting an apply batch of log records to the database...
InnoDB: Apply batch completed
InnoDB: Started
mysqld: ready for connections
```

对于文件系统崩溃或硬件问题的情况，我们可以假设在重新启动后 MySQL 磁盘数据*不可用*。这意味着 MySQL 无法成功启动，因为一些磁盘数据块不再可读。在这种情况下，需要重新格式化磁盘，安装新的磁盘，或者以其他方式纠正潜在问题。然后需要从备份中恢复我们的 MySQL 数据，这意味着备份必须已经制作好。为确保情况如此，设计并实施一个备份策略。
