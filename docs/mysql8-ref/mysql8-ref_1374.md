> 原文：[`dev.mysql.com/doc/refman/8.0/en/replication-administration-pausing.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-administration-pausing.html)

#### 19.1.7.2 在副本上暂停复制

你可以使用`STOP REPLICA`和`START REPLICA`语句在副本上停止和启动复制。从 MySQL 8.0.22 开始，`STOP SLAVE`和`START SLAVE`已被弃用，可以使用`STOP REPLICA`和`START REPLICA`代替。

要停止源的二进制日志的处理，请使用`STOP REPLICA`：

```sql
mysql> STOP SLAVE;
Or from MySQL 8.0.22:
mysql> STOP REPLICA;
```

当复制停止时，复制 I/O（接收器）线程停止从源二进制日志读取事件并将其写入中继日志，SQL 线程停止从中继日志读取事件并执行它们。您可以通过指定线程类型单独暂停 I/O（接收器）或 SQL（应用程序）线程：

```sql
mysql> STOP SLAVE IO_THREAD;
mysql> STOP SLAVE SQL_THREAD;
Or from MySQL 8.0.22:
mysql> STOP REPLICA IO_THREAD;
mysql> STOP REPLICA SQL_THREAD;
```

要再次开始执行，请使用`START REPLICA`语句：

```sql
mysql> START SLAVE;
Or from MySQL 8.0.22:
mysql> START REPLICA;
```

要启动特定线程，请指定线程类型：

```sql
mysql> START SLAVE IO_THREAD;
mysql> START SLAVE SQL_THREAD;
Or from MySQL 8.0.22:
mysql> START REPLICA IO_THREAD;
mysql> START REPLICA SQL_THREAD;
```

对于仅通过处理来自源的事件来执行更新的副本，如果要执行备份或其他任务，则仅停止 SQL 线程可能很有用。I/O（接收器）线程继续从源读取事件，但不执行它们。这使得在重新启动 SQL（应用程序）线程时更容易让副本赶上。

仅停止接收线程使中继日志中的事件能够被应用程序线程执行，直到中继日志结束的地方。当您想要暂停执行以赶上已从源接收的事件，当您想要在副本上执行管理操作但又确保它已处理到特定点的所有更新时，这可能很有用。此方法还可用于在您对源进行管理操作时暂停副本上的事件接收。停止接收线程但允许应用程序线程运行有助于确保在再次启动复制时不会有大量待执行的事件积压。
