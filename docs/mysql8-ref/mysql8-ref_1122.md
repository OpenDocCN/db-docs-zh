> 原文：[`dev.mysql.com/doc/refman/8.0/en/flush.html`](https://dev.mysql.com/doc/refman/8.0/en/flush.html)

#### 15.7.8.3 FLUSH Statement

```sql
FLUSH [NO_WRITE_TO_BINLOG | LOCAL] {
    *flush_option* [, *flush_option*] ...
  | *tables_option*
}

*flush_option*: {
    BINARY LOGS
  | ENGINE LOGS
  | ERROR LOGS
  | GENERAL LOGS
  | HOSTS
  | LOGS
  | PRIVILEGES
  | OPTIMIZER_COSTS
  | RELAY LOGS [FOR CHANNEL *channel*]
  | SLOW LOGS
  | STATUS
  | USER_RESOURCES
}

*tables_option*: {
    *table_synonym*
  | *table_synonym* *tbl_name* [, *tbl_name*] ...
  | *table_synonym* WITH READ LOCK
  | *table_synonym* *tbl_name* [, *tbl_name*] ... WITH READ LOCK
  | *table_synonym* *tbl_name* [, *tbl_name*] ... FOR EXPORT
}

*table_synonym*: {
    TABLE
  | TABLES
}
```

`FLUSH`语句有几种变体形式，用于清除或重新加载各种内部缓存、刷新表或获取锁。每个`FLUSH`操作都需要其描述中指示的权限。

注意

在存储函数或触发器中不可能发出`FLUSH`语句。但是，您可以在存储过程中使用`FLUSH`，只要这些存储过程不是从存储函数或触发器中调用的。请参阅第 27.8 节，“存储程序的限制”。

默认情况下，服务器会将`FLUSH`语句写入二进制日志，以便它们复制到副本中。要禁止记录日志，请指定可选的`NO_WRITE_TO_BINLOG`关键字或其别名`LOCAL`。

注意

`FLUSH LOGS`、`FLUSH BINARY LOGS`、`FLUSH TABLES WITH READ LOCK`（带或不带表列表）、以及`FLUSH TABLES *`tbl_name`* ... FOR EXPORT`在任何情况下都不会写入二进制日志，因为如果复制到副本中会导致问题。

`FLUSH`语句会导致隐式提交。请参阅第 15.3.3 节，“导致隐式提交的语句”。

**mysqladmin**实用程序提供了一个命令行界面，用于执行一些刷新操作，使用命令如`flush-hosts`、`flush-logs`、`flush-privileges`、`flush-status`和`flush-tables`。请参阅第 6.5.2 节，“mysqladmin — MySQL 服务器管理程序”。

向服务器发送`SIGHUP`或`SIGUSR1`信号会导致发生几种刷新操作，类似于各种形式的`FLUSH`语句。信号可以由`root`系统帐户或拥有服务器进程的系统帐户发送。这使得刷新操作可以在不连接到服务器的情况下执行，而这需要一个具有足够权限的 MySQL 帐户进行这些操作。请参阅第 6.10 节，“MySQL 中的 Unix 信号处理”。

`重置`语句类似于`刷新`。有关在复制中使用`重置`的信息，请参见第 15.7.8.6 节，“重置语句”。

以下列表描述了允许的`刷新`语句*`flush_option`*值。有关允许的*`tables_option`*值的描述，请参见刷新表语法。

+   `刷新二进制日志`

    关闭并重新打开服务器正在写入的任何二进制日志文件。如果启用了二进制日志记录，则相对于上一个文件，二进制日志文件的序列号会增加一。

    此操作需要`RELOAD`权限。

+   `刷新引擎日志`

    关闭并重新打开任何可刷新的已安装存储引擎的日志。这会导致`InnoDB`将其日志刷新到磁盘。

    此操作需要`RELOAD`权限。

+   `刷新错误日志`

    关闭并重新打开服务器正在写入的任何错误日志文件。

    此操作需要`RELOAD`权限。

+   `刷新一般日志`

    关闭并重新打开服务器正在写入的任何一般查询日志文件。

    此操作需要`RELOAD`权限。

    此操作对用于一般查询日志的表没有影响（参见第 7.4.1 节，“选择一般查询日志和慢查询日志输出目的地”）。

+   `刷新主机`

    清空主机缓存和性能模式`host_cache`表，该表显示缓存内容，并解除任何被阻止的主机。

    此操作需要`RELOAD`权限。

    有关为什么可能建议或希望刷新主机缓存的信息，请参见第 7.1.12.3 节，“DNS 查找和主机缓存”。

    注意

    自 MySQL 8.0.23 起，`刷新主机`已弃用；预计将在未来的 MySQL 版本中删除。取而代之的是截断性能模式`host_cache`表：

    ```sql
    TRUNCATE TABLE performance_schema.host_cache;
    ```

    `TRUNCATE TABLE` 操作需要表的 `DROP` 权限，而不是 `RELOAD` 权限。您应该知道 `TRUNCATE TABLE` 语句不会被写入二进制日志。要从 `FLUSH HOSTS` 中获得相同的行为，请在语句中指定 `NO_WRITE_TO_BINLOG` 或 `LOCAL`。

+   `FLUSH LOGS`

    关闭并重新打开服务器正在写入的任何日志文件。

    此操作需要 `RELOAD` 权限。

    此操作的效果等同于这些操作的综合效果：

    ```sql
    FLUSH BINARY LOGS
    FLUSH ENGINE LOGS
    FLUSH ERROR LOGS
    FLUSH GENERAL LOGS
    FLUSH RELAY LOGS
    FLUSH SLOW LOGS
    ```

+   `FLUSH OPTIMIZER_COSTS`

    重新读取成本模型表，以便优化器开始使用其中存储的当前成本估算。

    此操作需要 `FLUSH_OPTIMIZER_COSTS` 或 `RELOAD` 权限。

    对于任何未识别的成本模型表条目，服务器会向错误日志写入警告。有关这些表的信息，请参见 Section 10.9.5, “The Optimizer Cost Model”。此操作仅影响在刷新后开始的会话。现有会话继续使用它们开始时的成本估算。

+   `FLUSH PRIVILEGES`

    重新从 `mysql` 系统模式中的授权表中读取权限。作为此操作的一部分，服务器会读取包含动态权限分配的 `global_grants` 表，并注册在那里找到的任何未注册的权限。

    重新加载授权表是必要的，以便仅在直接对授权表进行更改时才能启用对 MySQL 权限和用户的更新；对于像 `GRANT` 或 `REVOKE` 这样的帐户管理语句，它们会立即生效，不需要这样做。有关更多信息，请参见 Section 8.2.13, “When Privilege Changes Take Effect”。

    此操作需要 `RELOAD` 权限。

    如果在服务器启动时指定了 `--skip-grant-tables` 选项以禁用 MySQL 权限系统，则 `FLUSH PRIVILEGES` 提供了在运行时启用权限系统的方法。

    重置失败登录跟踪（或者如果服务器是使用 `--skip-grant-tables` 启动的，则启用它），并解锁任何临时锁定的帐户。参见 Section 8.2.15, “Password Management”。

    释放服务器缓存的内存，这是由`GRANT`、`CREATE USER`、`CREATE SERVER`和`INSTALL PLUGIN`语句导致的。这些内存不会被相应的`REVOKE`、`DROP USER`、`DROP SERVER`和`UNINSTALL PLUGIN`语句释放，因此对于执行许多导致缓存的语句实例的服务器，除非使用`FLUSH PRIVILEGES`释放，否则会增加缓存内存使用。

    清除`caching_sha2_password`认证插件使用的内存缓存。请参见 SHA-2 可插拔认证的缓存操作。

+   [`FLUSH RELAY LOGS [FOR CHANNEL *`channel`*]`](flush.html#flush-relay-logs)

    关闭并重新打开服务器正在写入的任何中继日志文件。如果启用了中继日志记录，则相对于上一个文件，中继日志文件的序列号将增加一。

    此操作需要`RELOAD`权限。

    `FOR CHANNEL *`channel`*`子句允许您指定操作应用于哪个复制通道。执行`FLUSH RELAY LOGS FOR CHANNEL *`channel`*`以刷新特定复制通道的中继日志。如果未命名通道且不存在额外的复制通道，则操作将应用于默认通道。如果未命名通道且存在多个复制通道，则操作将应用于所有复制通道。有关更多信息，请参见第 19.2.2 节，“复制通道”。

+   `FLUSH SLOW LOGS`

    关闭并重新打开服务器正在写入的任何慢查询日志文件。

    此操作需要`RELOAD`权限。

    此操作不会影响用于慢查询日志的表（请参见第 7.4.1 节，“选择一般查询日志和慢查询日志输出目的地”）。

+   `FLUSH STATUS`

    刷新状态指示器。

    此操作将当前线程的会话状态变量值添加到全局值中，并将会话值重置为零。一些全局变量也可能被重置为零。它还将键缓存（默认和命名）的计数器重置为零，并将 `Max_used_connections` 设置为当前打开连接数。在调试查询时，此信息可能会有用。请参阅 第 1.5 节，“如何报告错误或问题”。

    `FLUSH STATUS` 不受 `read_only` 或 `super_read_only` 的影响，并始终写入二进制日志。

    此操作需要 `FLUSH_STATUS` 或 `RELOAD` 权限。

+   `FLUSH USER_RESOURCES`

    将所有每小时用户资源指标重置为零。

    此操作需要 `FLUSH_USER_RESOURCES` 或 `RELOAD` 权限。

    重置资源指标使达到每小时连接、查询或更新限制的客户端可以立即恢复活动。 `FLUSH USER_RESOURCES` 不适用于由 `max_user_connections` 系统变量控制的最大同时连接数限制。请参阅 第 8.2.21 节，“设置帐户资源限制”。

##### FLUSH TABLES 语法

`FLUSH TABLES` 刷新表，并根据使用的变体获取锁。在 `FLUSH` 语句中使用的任何 `TABLES` 变体必须是唯一使用的选项。 `FLUSH TABLE` 是 `FLUSH TABLES` 的同义词。

注意

这里描述的指示通过关闭表来刷新表的描述对于 `InnoDB` 有所不同，它会将表内容刷新到磁盘，但保持表处于打开状态。这仍然允许在表处于打开状态时复制表文件，只要其他活动不修改它们。

+   `FLUSH TABLES`

    关闭所有打开的表，强制关闭所有正在使用的表，并刷新准备好的语句缓存。

    此操作需要 `FLUSH_TABLES` 或 `RELOAD` 权限。

    有关准备语句缓存的信息，请参阅 第 10.10.3 节，“准备语句和存储程序的缓存”。

    当存在活动的 `LOCK TABLES ... READ` 时，不允许执行 `FLUSH TABLES`。要刷新并锁定表，请使用 `FLUSH TABLES *`tbl_name`* ... WITH READ LOCK`。

+   [`FLUSH TABLES *`tbl_name`* [, *`tbl_name`*] ...`](flush.html#flush-tables-with-list)

    使用一个或多个逗号分隔的表名列表，此操作类似于不带名称的 `FLUSH TABLES`，只是服务器只刷新指定的表。如果指定的表不存在，不会发生错误。

    此操作需要 `FLUSH_TABLES` 或 `RELOAD` 权限。

+   `FLUSH TABLES WITH READ LOCK`

    关闭所有打开的表并为所有数据库的所有表加锁以获取全局读锁。

    此操作需要 `FLUSH_TABLES` 或 `RELOAD` 权限。

    如果您有像 Veritas 或 ZFS 这样可以在时间上进行快照的文件系统，这个操作是获取备份的非常方便的方法。使用 `UNLOCK TABLES` 来释放锁。

    `FLUSH TABLES WITH READ LOCK` 获取全局读锁而不是表锁，因此与表锁定和隐式提交相关的行为不受相同的影响：

    +   `UNLOCK TABLES` 会隐式提交任何已激活的事务，只有当任何表当前已被 `LOCK TABLES` 锁定时才会发生提交。对于跟随 `FLUSH TABLES WITH READ LOCK` 的 `UNLOCK TABLES` 不会发生提交，因为后者不会获取表锁。

    +   开始事务会导致使用 `LOCK TABLES` 获取的表锁被释放，就像执行了 `UNLOCK TABLES` 一样。开始事务不会释放使用 `FLUSH TABLES WITH READ LOCK` 获取的全局读锁。

    `FLUSH TABLES WITH READ LOCK`不会阻止服务器向日志表插入行（参见第 7.4.1 节，“选择通用查询日志和慢查询日志输出目的地”）。

+   [`FLUSH TABLES *`tbl_name`* [, *`tbl_name`*] ... WITH READ LOCK`](flush.html#flush-tables-with-read-lock-with-list)

    刷新并获取命名表的读锁。

    此操作需要`FLUSH_TABLES`或`RELOAD`权限。因为它获取表锁，所以还需要每个表的`LOCK TABLES`权限。

    该操作首先为表获取排他性元数据锁，因此它会等待那些打开这些表的事务完成。然后，操作会从表缓存中刷新表，重新打开表，获取表锁（类似于`LOCK TABLES ... READ`)，并将元数据锁从排他性降级为共享。在操作获取锁并降级元数据锁之后，其他会话可以读取但不能修改这些表。

    此操作仅适用于现有的基本（非`TEMPORARY)`表。如果名称指向基本表，则使用该表。如果它指向一个`TEMPORARY`表，则会被忽略。如果名称应用于视图，则会发生`ER_WRONG_OBJECT`错误。否则，会发生`ER_NO_SUCH_TABLE`错误。

    使用`UNLOCK TABLES`来释放锁，`LOCK TABLES`来释放锁并获取其他锁，或者使用`START TRANSACTION`来释放锁并开始一个新事务。

    这个`FLUSH TABLES`变体允许在单个操作中刷新和锁定表。它提供了一个解决方案，因为当存在活动的`LOCK TABLES ... READ`时，不允许执行`FLUSH TABLES`。

    此操作不会执行隐式的`UNLOCK TABLES`，因此如果在存在任何活动的`LOCK TABLES`的情况下执行操作，或者在释放已获取的锁之前第二次使用它，都会导致错误。

    如果刷新的表是使用`HANDLER`打开的，则处理程序会被隐式刷新并丢失其位置。

+   [`FLUSH TABLES *`tbl_name`* [, *`tbl_name`*] ... FOR EXPORT`](flush.html#flush-tables-for-export-with-list)

    这个`FLUSH TABLES`变体适用于`InnoDB`表。它确保对指定表的更改已刷新到磁盘，以便在服务器运行时可以进行二进制表副本的制作。

    此操作需要`FLUSH_TABLES`或`RELOAD`权限。因为它在准备导出表时获取表上的锁，所以还需要每个表的`LOCK TABLES`和`SELECT`权限。

    操作的工作方式如下：

    1.  它为命名表获取共享元数据锁。只要其他会话有活动事务修改了这些表或持有这些表的表锁，该操作就会阻塞。在获取锁之后，该操作会阻止试图更新表的事务，同时允许只读操作继续。

    1.  它检查表的所有存储引擎是否支持`FOR EXPORT`。如果有任何存储引擎不支持，将会发生`ER_ILLEGAL_HA`错误，操作将失败。

    1.  该操作通知每个表的存储引擎使表准备好导出。存储引擎必须确保任何待处理的更改都已写入磁盘。

    1.  该操作将会将会话置于锁表模式，以便在`FOR EXPORT`操作完成时不释放先前获取的元数据锁。

    此操作仅适用于现有的基本（非`TEMPORARY`）表。如果名称指的是基本表，则使用该表。如果指的是`TEMPORARY`表，则会被忽略。如果名称适用于视图，则会发生`ER_WRONG_OBJECT`错误。否则，会发生`ER_NO_SUCH_TABLE`错误。

    `InnoDB`支持对具有自己的`.ibd`文件文件的表进行`FOR EXPORT`（即启用了`innodb_file_per_table`设置的表）。`InnoDB`确保在`FOR EXPORT`操作通知时，任何更改都已刷新到磁盘。这允许在`FOR EXPORT`操作生效时制作表内容的二进制副本，因为`.ibd`文件是事务一致的，可以在服务器运行时复制。`FOR EXPORT`不适用于`InnoDB`系统表空间文件，也不适用于具有`FULLTEXT`索引的`InnoDB`表。

    `FLUSH TABLES ...FOR EXPORT`支持分区的`InnoDB`表。

    当收到`FOR EXPORT`通知时，`InnoDB`会将通常保存在内存中或在表空间文件之外的磁盘缓冲区中的某些类型数据写入磁盘。对于每个表，`InnoDB`还会在与表相同的数据库目录中生成一个名为`*`table_name`*.cfg`的文件。`.cfg`文件包含重新导入表空间文件所需的元数据，以便稍后重新导入到相同或不同的服务器中。

    当`FOR EXPORT`操作完成时，`InnoDB`已经将所有脏页刷新到表数据文件中。在刷新之前，任何更改缓冲区条目都会被合并。此时，表被锁定并处于静止状态：表在磁盘上处于事务一致状态，您可以将`.ibd`表空间文件与相应的`.cfg`文件一起复制，以获得这些表的一致快照。

    要重新导入复制的表数据到 MySQL 实例的过程，请参见第 17.6.1.3 节，“导入 InnoDB 表”。

    在处理完表之后，请使用`UNLOCK TABLES`释放锁定，`LOCK TABLES`释放锁定并获取其他锁定，或者使用`START TRANSACTION`释放锁定并开始新事务。

    在会话中执行这些语句之一时，尝试使用`FLUSH TABLES ... FOR EXPORT`会产生错误：

    ```sql
    FLUSH TABLES ... WITH READ LOCK
    FLUSH TABLES ... FOR EXPORT
    LOCK TABLES ... READ
    LOCK TABLES ... WRITE
    ```

    在会话中有效时，尝试使用任何这些语句会产生错误：`FLUSH TABLES ... FOR EXPORT`。

    ```sql
    FLUSH TABLES WITH READ LOCK
    FLUSH TABLES ... WITH READ LOCK
    FLUSH TABLES ... FOR EXPORT
    ```
