# 15.3.5 LOCK INSTANCE FOR BACKUP 和 UNLOCK INSTANCE 语句

> 原文：[`dev.mysql.com/doc/refman/8.0/en/lock-instance-for-backup.html`](https://dev.mysql.com/doc/refman/8.0/en/lock-instance-for-backup.html)

```sql
LOCK INSTANCE FOR BACKUP

UNLOCK INSTANCE
```

`LOCK INSTANCE FOR BACKUP`获取一个实例级别的*备份锁*，允许在在线备份期间进行 DML 操作，同时阻止可能导致不一致快照的操作。

执行`LOCK INSTANCE FOR BACKUP`语句需要`BACKUP_ADMIN`权限。当从早期版本升级到 MySQL 8.0 时，具有`RELOAD`权限的用户会自动被授予`BACKUP_ADMIN`权限。

多个会话可以同时持有备份锁。

`UNLOCK INSTANCE`释放当前会话持有的备份锁。如果会话终止，会话持有的备份锁也会被释放。

`LOCK INSTANCE FOR BACKUP`阻止创建、重命名或删除文件。`REPAIR TABLE` `TRUNCATE TABLE`、`OPTIMIZE TABLE`和账户管理语句被阻止。参见第 15.7.1 节，“账户管理语句”。还会阻止修改未记录在`InnoDB`重做日志中的`InnoDB`文件的操作。

`LOCK INSTANCE FOR BACKUP`允许执行仅影响用户创建的临时表的 DDL 操作。实际上，在持有备份锁时，可以创建、重命名或删除属于用户创建的临时表的文件。也允许创建二进制日志文件。

在实例上有效的`LOCK INSTANCE FOR BACKUP`语句生效期间，不应发出`PURGE BINARY LOGS`语句，因为这违反了备份锁的规则，会从服务器中删除文件。从 MySQL 8.0.28 开始，这是不允许的。

由`LOCK INSTANCE FOR BACKUP`获取的备份锁独立于事务锁和由[`FLUSH TABLES *`tbl_name`* [, *`tbl_name`*] ... WITH READ LOCK`](flush.html#flush-tables-with-read-lock-with-list)获取的锁，并允许以下语句序列：

```sql
LOCK INSTANCE FOR BACKUP;
FLUSH TABLES *tbl_name* [, *tbl_name*] ... WITH READ LOCK;
UNLOCK TABLES;
UNLOCK INSTANCE;
```

```sql
FLUSH TABLES *tbl_name* [, *tbl_name*] ... WITH READ LOCK;
LOCK INSTANCE FOR BACKUP;
UNLOCK INSTANCE;
UNLOCK TABLES;
```

`lock_wait_timeout`设置定义了`LOCK INSTANCE FOR BACKUP`语句在放弃之前等待获取锁的时间。
