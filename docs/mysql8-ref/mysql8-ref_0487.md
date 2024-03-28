# 9.3.1 建立备份策略

> 原文：[`dev.mysql.com/doc/refman/8.0/en/backup-policy.html`](https://dev.mysql.com/doc/refman/8.0/en/backup-policy.html)

为了有用，备份必须定期安排。在 MySQL 中可以使用几种工具进行完整备份（在某个时间点的数据快照）。例如，MySQL 企业版备份可以执行整个实例的物理备份，并进行优化以最小化开销并在备份`InnoDB`数据文件时避免中断；**mysqldump**提供在线逻辑备份。本讨论使用**mysqldump**。

假设我们在星期日下午 1 点，负载较低时，使用以下命令对所有数据库中的所有`InnoDB`表进行完全备份：

```sql
$> mysqldump --all-databases --master-data --single-transaction > backup_sunday_1_PM.sql
```

由**mysqldump**生成的`.sql`文件包含一组 SQL `INSERT`语句，可用于在以后重新加载转储的表。

此备份操作在转储开始时（使用`FLUSH TABLES WITH READ LOCK`）获取所有表的全局读锁。一旦获得此锁，就会读取二进制日志坐标并释放锁。如果在发出`FLUSH`语句时正在运行长时间的更新语句，则备份操作可能会停顿，直到这些语句完成。之后，转储变为无锁状态，不会干扰表的读写操作。

之前假设要备份的表是`InnoDB`表，因此`--single-transaction`使用一致性读取，并保证**mysqldump**看到的数据不会更改。（其他客户端对`InnoDB`表所做的更改不会被**mysqldump**进程看到。）如果备份操作包括非事务表，一致性要求在备份期间它们不会更改。例如，在`mysql`数据库中的`MyISAM`表，备份期间不能对 MySQL 帐户进行管理更改。

完整备份是必要的，但并不总是方便创建。它们产生大型备份文件并需要时间生成。从优化的角度来看，每次连续的完整备份都包含所有数据，即使是自上次完整备份以来未更改的部分。更有效的方法是进行初始完整备份，然后进行增量备份。增量备份更小，生成时间更短。权衡之处在于，在恢复时，你不能仅通过重新加载完整备份来恢复数据。你还必须处理增量备份以恢复增量更改。

要进行增量备份，我们需要保存增量更改。在 MySQL 中，这些更改在二进制日志中表示，因此 MySQL 服务器应始终使用`--log-bin`选项启动以启用该日志。启用二进制日志记录后，服务器在更新数据时将每个数据更改写入文件。查看运行了一些天的 MySQL 服务器的数据目录，我们会发现这些 MySQL 二进制日志文件：

```sql
-rw-rw---- 1 guilhem  guilhem   1277324 Nov 10 23:59 gbichot2-bin.000001
-rw-rw---- 1 guilhem  guilhem         4 Nov 10 23:59 gbichot2-bin.000002
-rw-rw---- 1 guilhem  guilhem        79 Nov 11 11:06 gbichot2-bin.000003
-rw-rw---- 1 guilhem  guilhem       508 Nov 11 11:08 gbichot2-bin.000004
-rw-rw---- 1 guilhem  guilhem 220047446 Nov 12 16:47 gbichot2-bin.000005
-rw-rw---- 1 guilhem  guilhem    998412 Nov 14 10:08 gbichot2-bin.000006
-rw-rw---- 1 guilhem  guilhem       361 Nov 14 10:07 gbichot2-bin.index
```

每次重新启动时，MySQL 服务器都会使用序列中的下一个数字创建一个新的二进制日志文件。在服务器运行时，您还可以告诉它通过发出`FLUSH LOGS` SQL 语句或使用**mysqladmin flush-logs**命令手动关闭当前的二进制日志文件并开始一个新的。**mysqldump**还有一个选项来刷新日志。数据目录中的`.index`文件包含目录中所有 MySQL 二进制日志的列表。

MySQL 二进制日志对于恢复很重要，因为它们形成了增量备份集。如果确保在进行完整备份时刷新日志，那么之后创建的二进制日志文件将包含自备份以来进行的所有数据更改。让我们稍微修改之前的**mysqldump**命令，以便在完整备份时刷新 MySQL 二进制日志，并使转储文件包含新当前二进制日志的名称：

```sql
$> mysqldump --single-transaction --flush-logs --master-data=2 \
         --all-databases > backup_sunday_1_PM.sql
```

执行此命令后，数据目录包含一个新的二进制日志文件，`gbichot2-bin.000007`，因为`--flush-logs`选项导致服务器刷新其日志。`--master-data`选项导致**mysqldump**将二进制日志信息写入其输出，因此生成的`.sql`转储文件包含这些行：

```sql
-- Position to start replication or point-in-time recovery from
-- CHANGE MASTER TO MASTER_LOG_FILE='gbichot2-bin.000007',MASTER_LOG_POS=4;
```

因为**mysqldump**命令进行了完整备份，这些行意味着两件事：

+   转储文件包含在写入`gbichot2-bin.000007`二进制日志文件或更高版本之前所做的所有更改。

+   备份后记录的所有数据更改都不包含在转储文件中，但包含在`gbichot2-bin.000007`二进制日志文件或更高版本中。

在星期一下午 1 点，我们可以通过刷新日志来开始一个新的二进制日志文件进行增量备份。例如，执行**mysqladmin flush-logs**命令会创建`gbichot2-bin.000008`。在星期日下午 1 点全量备份和星期一下午 1 点之间的所有更改都写入`gbichot2-bin.000007`。这个增量备份很重要，所以最好将其复制到一个安全的地方。（例如，将其备份到磁带或 DVD 上，或将其复制到另一台机器上。）在星期二下午 1 点，执行另一个**mysqladmin flush-logs**命令。在星期一下午 1 点和星期二下午 1 点之间的所有更改都写入`gbichot2-bin.000008`（也应该将其复制到安全的地方）。

MySQL 二进制日志占用磁盘空间。为了释放空间，定期清理它们。一种方法是删除不再需要的二进制日志，例如当我们进行全量备份时：

```sql
$> mysqldump --single-transaction --flush-logs --master-data=2 \
         --all-databases --delete-master-logs > backup_sunday_1_PM.sql
```

注意

使用**mysqldump --delete-master-logs**删除 MySQL 二进制日志可能会很危险，如果您的服务器是一个复制源服务器，因为副本可能尚未完全处理二进制日志的内容。`PURGE BINARY LOGS`语句的描述解释了在删除 MySQL 二进制日志之前应该验证的内容。请参阅 Section 15.4.1.1, “PURGE BINARY LOGS Statement”。
