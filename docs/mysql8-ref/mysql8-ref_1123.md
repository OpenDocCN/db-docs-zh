> 原文：[`dev.mysql.com/doc/refman/8.0/en/kill.html`](https://dev.mysql.com/doc/refman/8.0/en/kill.html)

#### 15.7.8.4 KILL Statement

```sql
KILL [CONNECTION | QUERY] *processlist_id*
```

每个连接到**mysqld**的运行在一个单独的线程中。您可以使用`KILL *`processlist_id`*`语句终止一个线程。

线程进程列表标识符可以从`INFORMATION_SCHEMA` `PROCESSLIST`表的`ID`列，`SHOW PROCESSLIST`输出的`Id`列以及性能模式`threads`表的`PROCESSLIST_ID`列中确定。当前线程的值由`CONNECTION_ID()`函数返回。

`KILL`允许使用可选的`CONNECTION`或`QUERY`修饰符：

+   `KILL CONNECTION`与没有修饰符的`KILL`相同：它终止与给定*`processlist_id`*相关联的连接，在终止连接正在执行的任何语句之后。

+   `KILL QUERY`终止连接当前正在执行的语句，但保持连接本身不变。

查看可终止的线程取决于`PROCESS`权限：

+   没有`PROCESS`，您只能看到自己的线程。

+   使用`PROCESS`，您可以看到所有线程。

终止线程和语句的能力取决于`CONNECTION_ADMIN`权限和已弃用的`SUPER`权限：

+   没有`CONNECTION_ADMIN`或`SUPER`，您只能终止自己的线程和语句。

+   使用`CONNECTION_ADMIN`或`SUPER`，您可以终止所有线程和语句，但要影响正在使用`SYSTEM_USER`权限执行的线程或语句，您自己的会话还必须具有`SYSTEM_USER`权限。

您还可以使用**mysqladmin processlist**和**mysqladmin kill**命令来检查和终止线程。

当您使用`KILL`时，会为线程设置特定于线程的 kill 标志。在大多数情况下，线程可能需要一些时间才能终止，因为 kill 标志仅在特定间隔检查：

+   在`SELECT`操作期间，对于`ORDER BY`和`GROUP BY`循环，每次读取一块行后都会检查标志。如果设置了 kill 标志，则会中止语句。

+   进行使表复制的`ALTER TABLE`操作会定期检查 kill 标志，以便从原始表中读取每几行复制的行。如果设置了 kill 标志，则会中止语句并删除临时表。

    `KILL`语句会立即返回而不等待确认，但 kill 标志检查会在相当短的时间内中止操作。中止操作以执行任何必要的清理也需要一些时间。

+   在`UPDATE`或`DELETE`操作期间，每次读取块和每次更新或删除行后都会检查 kill 标志。如果设置了 kill 标志，则会中止语句。如果您没有使用事务，则更改不会回滚。

+   `GET_LOCK()`会中止并返回`NULL`。

+   如果线程在表锁处理程序中（状态：`Locked`），则表锁会被快速中止。

+   如果线程在写调用中等待空闲磁盘空间，则会用“磁盘已满”错误消息中止写操作。

+   `EXPLAIN ANALYZE`会中止并打印输出的第一行。这适用于 MySQL 8.0.20 及更高版本。

警告

在`MyISAM`表上终止`REPAIR TABLE`或`OPTIMIZE TABLE`操作会导致表损坏且无法使用。在您再次优化或修复它之前（无中断），对这样的表的任何读取或写入都会失败。
