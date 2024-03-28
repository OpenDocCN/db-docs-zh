# 17.18.1 InnoDB 备份

> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-backup.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-backup.html)

安全数据库管理的关键是定期备份。根据您的数据量、MySQL 服务器数量和数据库工作量，您可以单独或结合使用以下备份技术：使用*MySQL Enterprise Backup*进行热备份；在 MySQL 服务器关闭时通过复制文件进行冷备份；使用**mysqldump**进行逻辑备份，适用于较小的数据量或记录模式对象的结构。热备份和冷备份是物理备份，复制实际数据文件，可以直接被**mysqld**服务器用于更快的恢复。

使用*MySQL Enterprise Backup*是备份`InnoDB`数据的推荐方法。

注意

`InnoDB`不支持使用第三方备份工具还原的数据库。

#### 热备份

**mysqlbackup**命令是 MySQL Enterprise Backup 组件的一部分，允许您备份运行中的 MySQL 实例，包括`InnoDB`表，在产生一致的数据库快照的同时最小化对操作的干扰。当**mysqlbackup**复制`InnoDB`表时，对`InnoDB`表的读写可以继续。MySQL Enterprise Backup 还可以创建压缩备份文件，并备份表和数据库的子集。结合 MySQL 二进制日志，用户可以执行按时间点恢复。MySQL Enterprise Backup 是 MySQL Enterprise 订阅的一部分。有关更多详细信息，请参见 Section 32.1, “MySQL Enterprise Backup Overview”。

#### 冷备份

如果可以关闭 MySQL 服务器，可以进行包含`InnoDB`用于管理其表的所有文件的物理备份。使用以下步骤：

1.  执行慢关闭 MySQL 服务器，并确保它在没有错误的情况下停止。

1.  将所有`InnoDB`数据文件（`ibdata`文件和`.ibd`文件）复制到安全位置。

1.  将所有`InnoDB`重做日志文件（MySQL 8.0.30 及更高版本中的`#ib_redo*`N`*`文件或早期版本中的`ib_logfile`文件）复制到安全位置。

1.  将您的`my.cnf`配置文件或文件复制到安全位置。

#### 使用 mysqldump 进行逻辑备份

除了物理备份之外，建议您定期使用**mysqldump**转储表格，创建逻辑备份。二进制文件可能会在您没有注意到的情况下损坏。转储的表格存储在人类可读的文本文件中，因此更容易发现表格损坏。此外，由于格式更简单，严重数据损坏的机会更小。**mysqldump**还具有`--single-transaction`选项，可以在不锁定其他客户端的情况下创建一致的快照。请参阅第 9.3.1 节，“建立备份策略”。

复制功能适用于`InnoDB`表格，因此您可以使用 MySQL 复制功能在需要高可用性的数据库站点保留数据库副本。请参阅第 17.19 节，“InnoDB 和 MySQL 复制”。
