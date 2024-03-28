# 7.4.6 服务器日志维护

> 原文：[`dev.mysql.com/doc/refman/8.0/en/log-file-maintenance.html`](https://dev.mysql.com/doc/refman/8.0/en/log-file-maintenance.html)

如 第 7.4 节，“MySQL 服务器日志” 中所述，MySQL 服务器可以创建几个不同的日志文件，帮助您查看正在发生的活动。然而，您必须定期清理这些文件，以确保日志不会占用太多磁盘空间。

当使用启用日志记录的 MySQL 时，您可能希望定期备份并删除旧的日志文件，并告诉 MySQL 开始记录到新文件中。参见 第 9.2 节，“数据库备份方法”。

在 Linux（Red Hat）安装中，您可以使用 `mysql-log-rotate` 脚本进行日志维护。如果您从 RPM 发行版安装了 MySQL，则此脚本应该已自动安装。如果您正在使用二进制日志进行复制，请小心使用此脚本。在确定所有副本都已处理其内容之前，不应删除二进制日志。

在其他系统上，您必须自己安装一个短脚本，然后从 **cron**（或其等效物）启动以处理日志文件。

二进制日志文件在服务器的二进制日志过期期间后会自动删除。文件的删除可以在启动时和二进制日志刷新时进行。默认的二进制日志过期期限为 30 天。要指定替代的过期期限，请使用 `binlog_expire_logs_seconds` 系统变量。如果您正在使用复制，您应该指定一个不低于副本可能滞后源的最长时间的过期期限。要按需删除二进制日志，请使用 `PURGE BINARY LOGS` 语句（参见 第 15.4.1.1 节，“PURGE BINARY LOGS 语句”）。

要强制 MySQL 开始使用新的日志文件，需要刷新日志。执行`FLUSH LOGS`语句或**mysqladmin flush-logs**、**mysqladmin refresh**、**mysqldump --flush-logs**或**mysqldump --master-data**命令时会发生日志刷新。参见第 15.7.8.3 节，“FLUSH 语句”、第 6.5.2 节，“mysqladmin — A MySQL Server Administration Program”和第 6.5.4 节，“mysqldump — A Database Backup Program”。此外，当当前二进制日志文件大小达到`max_binlog_size`系统变量的值时，服务器会自动刷新二进制日志。

`FLUSH LOGS`支持可选修饰符，以启用对单个日志的选择性刷新（例如，`FLUSH BINARY LOGS`）。参见第 15.7.8.3 节，“FLUSH 语句”。

日志刷新操作具有以下效果：

+   如果启用了二进制日志记录，服务器会关闭当前的二进制日志文件，并打开下一个序列号的新日志文件。

+   如果启用了一般查询日志或慢查询日志到日志文件的记录，服务器会关闭并重新打开日志文件。

+   如果服务器是使用`--log-error`选项启动的，以将错误日志写入文件，服务器会关闭并重新打开日志文件。

执行刷新日志语句或命令需要使用具有`RELOAD`权限的帐户连接到服务器。在 Unix 和类 Unix 系统上，刷新日志的另一种方法是向服务器发送信号，可以由`root`或拥有服务器进程的帐户执行。（参见第 6.10 节，“MySQL 中的 Unix 信号处理”。）信号使得可以在不连接到服务器的情况下执行日志刷新：

+   `SIGHUP`信号会刷新所有日志。然而，`SIGHUP`除了日志刷新之外还有其他可能不希望的附加效果。

+   截至 MySQL 8.0.19 版本，`SIGUSR1`会导致服务器刷新错误日志、一般查询日志和慢查询日志。如果只想刷新这些日志，`SIGUSR1`可以作为一个更“轻量级”的信号，不会产生与日志无关的`SIGHUP`效果。

如前所述，刷新二进制日志会创建一个新的二进制日志文件，而刷新一般查询日志、慢查询日志或错误日志只是关闭并重新打开日志文件。对于后者的日志，在 Unix 上，要在刷新之前重命名当前日志文件以创建一个新的日志文件。在刷新时，服务器会使用原始名称打开新的日志文件。例如，如果一般查询日志、慢查询日志和错误日志文件分别命名为`mysql.log`、`mysql-slow.log`和`err.log`，您可以在命令行中使用一系列命令如下：

```sql
cd *mysql-data-directory*
mv mysql.log mysql.log.old
mv mysql-slow.log mysql-slow.log.old
mv err.log err.log.old
mysqladmin flush-logs
```

在 Windows 上，使用**rename**而不是**mv**。

此时，您可以备份`mysql.log.old`、`mysql-slow.log.old`和`err.log.old`，然后将它们从磁盘中删除。

要在运行时重命名一般查询日志或慢查询日志，首先连接到服务器并禁用日志：

```sql
SET GLOBAL general_log = 'OFF';
SET GLOBAL slow_query_log = 'OFF';
```

禁用日志后，可以在外部重命名日志文件（例如，从命令行）。然后再次启用日志：

```sql
SET GLOBAL general_log = 'ON';
SET GLOBAL slow_query_log = 'ON';
```

这种方法适用于任何平台，不需要重新启动服务器。

注意

在您在外部重命名文件后，服务器重新创建给定日志文件时，文件位置必须可被服务器写入。这并非总是如此。例如，在 Linux 上，服务器可能将错误日志写入为`/var/log/mysqld.log`，其中`/var/log`由`root`拥有且不可写入**mysqld**。在这种情况下，日志刷新操作无法创建新的日志文件。

处理这种情况，您必须在重命名原始日志文件后手动创建具有适当所有权的新日志文件。例如，以`root`身份执行以下命令：

```sql
mv /var/log/mysqld.log /var/log/mysqld.log.old
install -omysql -gmysql -m0644 /dev/null /var/log/mysqld.log
```
