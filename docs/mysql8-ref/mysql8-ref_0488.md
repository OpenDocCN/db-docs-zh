# 9.3.2 使用备份进行恢复

> 原文：[`dev.mysql.com/doc/refman/8.0/en/recovery-from-backups.html`](https://dev.mysql.com/doc/refman/8.0/en/recovery-from-backups.html)

现在，假设我们在周三早上 8 点发生了灾难性的意外退出，需要从备份中恢复。为了恢复，首先我们恢复我们拥有的最后一个完整备份（即周日下午 1 点的备份）。完整备份文件只是一组 SQL 语句，因此恢复它非常容易：

```sql
$> mysql < backup_sunday_1_PM.sql
```

此时，数据已恢复到周日下午 1 点的状态。要恢复自那时以来所做的更改，我们必须使用增量备份；也就是说，`gbichot2-bin.000007`和`gbichot2-bin.000008`二进制日志文件。如有必要，从备份位置获取文件，然后像这样处理它们的内容：

```sql
$> mysqlbinlog gbichot2-bin.000007 gbichot2-bin.000008 | mysql
```

现在，我们已将数据恢复到周二下午 1 点的状态，但仍然缺少从那天到崩溃日期的更改。为了不丢失它们，我们需要让 MySQL 服务器将其 MySQL 二进制日志存储到一个安全位置（RAID 磁盘，SAN，...），与存储数据文件的位置不同，以便这些日志不在被破坏的磁盘上。（也就是说，我们可以使用`--log-bin`选项启动服务器，指定一个与数据目录所在的物理设备不同的位置。这样，即使包含目录的设备丢失，日志也是安全的。）如果我们这样做了，我们将手头上有`gbichot2-bin.000009`文件（以及任何后续文件），我们可以使用**mysqlbinlog**和**mysql**应用它们，恢复最近的数据更改，直到崩溃时刻，而无需丢失：

```sql
$> mysqlbinlog gbichot2-bin.000009 ... | mysql
```

有关使用**mysqlbinlog**处理二进制日志文件的更多信息，请参阅第 9.5 节，“时间点（增量）恢复” Recovery")。
