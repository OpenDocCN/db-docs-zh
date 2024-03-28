> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-undo-tablespaces.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-undo-tablespaces.html)

#### 17.6.3.4 撤销表空间

撤销表空间包含撤销日志，这些日志是包含有关如何撤消事务对聚簇索引记录的最新更改的信息的记录集。

本节中以下主题中描述了撤销表空间：

+   默认撤销表空间

+   撤销表空间大小

+   添加撤销表空间

+   删除撤销表空间

+   移动撤销表空间

+   配置回滚段的数量

+   截断撤销表空间

+   撤销表空间状态变量

##### 默认撤销表空间

当初始化 MySQL 实例时会创建两个默认的撤销表空间。默认的撤销表空间在初始化时创建，以提供回滚段的位置，这些回滚段必须在 SQL 语句被接受之前存在。支持自动截断撤销表空间需要至少两个撤销表空间。请参阅截断撤销表空间。

默认的撤销表空间是在由`innodb_undo_directory`变量定义的位置创建的。如果`innodb_undo_directory`变量未定义，则默认的撤销表空间将在数据目录中创建。默认的撤销表空间数据文件命名为`undo_001`和`undo_002`。数据字典中定义的相应撤销表空间名称为`innodb_undo_001`和`innodb_undo_002`。

截至 MySQL 8.0.14，可以使用 SQL 在运行时创建额外的撤销表空间。请参阅添加撤销表空间。

##### 撤销表空间大小

在 MySQL 8.0.23 之前，撤销表空间的初始大小取决于`innodb_page_size`的值。对于默认的 16KB 页大小，初始撤销表空间文件大小为 10MiB。对于 4KB、8KB、32KB 和 64KB 页大小，初始撤销表空间文件大小分别为 7MiB、8MiB、20MiB 和 40MiB。从 MySQL 8.0.23 开始，初始撤销表空间大小通常为 16MiB。当通过截断操作创建新的撤销表空间时，初始大小可能会有所不同。在这种情况下，如果文件扩展大小大于 16MB，并且上一个文件扩展发生在最近一秒内，则新的撤销表空间将以`innodb_max_undo_log_size`变量定义的四分之一大小创建。

在 MySQL 8.0.23 之前，撤销表空间每次扩展四个区段。从 MySQL 8.0.23 开始，撤销表空间至少扩展 16MB。为了处理激进的增长，如果上一个文件扩展发生在不到 0.1 秒之前，则文件扩展大小加倍。文件扩展大小可以多次加倍，最多达到 256MB。如果上一个文件扩展发生在超过 0.1 秒之前，则文件扩展大小减半，这也可以多次发生，最少为 16MB。如果为撤销表空间定义了`AUTOEXTEND_SIZE`选项，则它将按照上述逻辑确定的扩展大小和`AUTOEXTEND_SIZE`设置中的较大值进行扩展。有关`AUTOEXTEND_SIZE`选项的信息，请参见 Section 17.6.3.9, “Tablespace AUTOEXTEND_SIZE Configuration”。

##### 添加撤销表空间

由于长时间运行的事务可能导致撤销日志变得很大，创建额外的撤销表空间可以帮助防止单个撤销表空间变得过大。从 MySQL 8.0.14 开始，可以使用`CREATE UNDO TABLESPACE`语法在运行时创建额外的撤销表空间。

```sql
CREATE UNDO TABLESPACE *tablespace_name* ADD DATAFILE '*file_name*.ibu';
```

撤销表空间文件名必须具有`.ibu`扩展名。在定义撤销表空间文件名时，不允许指定相对路径。允许使用完全限定路径，但路径必须为`InnoDB`所知。已知路径是由`innodb_directories`变量定义的路径。建议使用唯一的撤销表空间文件名，以避免在移动或克隆数据时出现潜在的文件名冲突。

注意

在复制环境中，源和每个副本必须有自己的撤销表空间文件目录。将撤销表空间文件的创建复制到公共目录会导致文件名冲突。

在启动时，由`innodb_directories`变量定义的目录将被扫描以查找撤销表空间文件。（扫描还会遍历子目录。）由`innodb_data_home_dir`、`innodb_undo_directory`和`datadir`变量定义的目录将自动附加到`innodb_directories`值中，无论`innodb_directories`变量是否被显式定义。因此，撤销表空间可以位于任何这些变量定义的路径中。

如果撤销表空间文件名不包含路径，则撤销表空间将创建在由`innodb_undo_directory`变量定义的目录中。如果该变量未定义，则撤销表空间将创建在数据目录中。

注意

`InnoDB`恢复过程要求撤销表空间文件位于已知目录中。在重做恢复和其他数据文件打开之前，必须发现并打开撤销表空间文件，以允许未提交的事务和数据字典更改被回滚。在恢复之前找不到的撤销表空间无法使用，这可能导致数据库不一致。如果数据字典中已知的撤销表空间未找到，则在启动时会报告错误消息。已知目录要求还支持撤销表空间的可移植性。请参阅移动撤销表空间。

要在相对于数据目录的路径中创建撤销表空间，请将`innodb_undo_directory`变量设置为相对路径，并在创建撤销表空间时仅指定文件名。

要查看撤销表空间的名称和路径，请查询`INFORMATION_SCHEMA.FILES`：

```sql
SELECT TABLESPACE_NAME, FILE_NAME FROM INFORMATION_SCHEMA.FILES
  WHERE FILE_TYPE LIKE 'UNDO LOG';
```

一个 MySQL 实例支持最多 127 个撤销表空间，包括 MySQL 实例初始化时创建的两个默认撤销表空间。

注意

在 MySQL 8.0.14 之前，通过配置`innodb_undo_tablespaces`启动变量来创建额外的撤销表空间。从 MySQL 8.0.14 开始，此变量已被弃用，不再可配置。

在 MySQL 8.0.14 之前，增加`innodb_undo_tablespaces`设置会���建指定数量的撤销表空间，并将它们添加到活动撤销表空间列表中。减少`innodb_undo_tablespaces`设置会从活动撤销表空间列表中删除撤销表空间。从活动列表中删除的撤销表空间会保持活动状态，直到它们不再被现有事务使用。可以使用`SET`语句在运行时配置`innodb_undo_tablespaces`变量，也��以在配置文件中定义。

在 MySQL 8.0.14 之前，停用的撤销表空间无法删除。在缓慢关闭后可以手动删除撤销表空间文件，但不建议这样做，因为停用的撤销表空间在服务器重新启动后可能会在一段时间内包含活动的撤销日志，如果在关闭服务器时存在未完成的事务。从 MySQL 8.0.14 开始，可以使用`DROP UNDO TABALESPACE`语法删除撤销表空间。请参阅 Dropping Undo Tablespaces。

##### 删除撤销表空间

截至 MySQL 8.0.14 版本，使用`CREATE UNDO TABLESPACE`语法创建的撤销表空间可以使用`DROP UNDO TABALESPACE`语法在运行时删除。

在删除撤销表空间之前，撤销表空间必须为空。要清空撤销表空间，必须首先使用`ALTER UNDO TABLESPACE`语法将撤销表空间标记为非活动状态，以便该表空间不再用于为新事务分配回滚段。

```sql
ALTER UNDO TABLESPACE *tablespace_name* SET INACTIVE;
```

将撤销表空间标记为非活动状态后，当前在撤销表空间中使用回滚段的事务被允许完成，以及在这些事务完成之前启动的任何事务。事务完成后，清除系统释放撤销表空间中的回滚段，并将撤销表空间截断为其初始大小。（在截断撤销表空间时使用相同的过程。请参阅 Truncating Undo Tablespaces。）一旦撤销表空间为空，就可以删除它。

```sql
DROP UNDO TABLESPACE *tablespace_name*;
```

注意

或者，如果需要，可以将撤销表空间保留为空状态，并在以后重新激活，方法是发出`ALTER UNDO TABLESPACE *`tablespace_name`* SET ACTIVE`语句。

撤销表空间的状态可以通过查询信息模式`INNODB_TABLESPACES`表来监视。

```sql
SELECT NAME, STATE FROM INFORMATION_SCHEMA.INNODB_TABLESPACES
  WHERE NAME LIKE '*tablespace_name*';
```

一个`inactive`状态表示回滚段在撤销表空间中不再被新事务使用。一个`empty`状态表示一个撤销表空间是空的，可以被删除，或者可以使用`ALTER UNDO TABLESPACE *`tablespace_name`* SET ACTIVE`语句重新激活。尝试删除一个非空的撤销表空间会返回错误。

当 MySQL 实例初始化时创建的默认撤销表空间（`innodb_undo_001`和`innodb_undo_002`）不能被删除。但是，它们可以通过使用`ALTER UNDO TABLESPACE *`tablespace_name`* SET INACTIVE`语句设置为非活动状态。在默认撤销表空间可以设置为非活动状态之前，必须有一个撤销表空间来替代它。始终需要至少两个活动的撤销表空间来支持自动截断撤销表空间。

##### 移动撤销表空间

使用`CREATE UNDO TABLESPACE`语法创建的撤销表空间可以在服务器离线状态下移动到任何已知目录。已知目录是由`innodb_directories`变量定义的目录。由`innodb_data_home_dir`、`innodb_undo_directory`和`datadir`定义的目录会自动附加到`innodb_directories`值中，无论`innodb_directories`变量是否被显式定义。这些目录及其子目录在启动时会被扫描以查找撤销表空间文件。移动到这些目录中的撤销表空间文件在启动时会被发现，并假定为已移动的撤销表空间。

当 MySQL 实例初始化时创建的默认撤销表空间（`innodb_undo_001`和`innodb_undo_002`）必须位于由`innodb_undo_directory`变量定义的目录中。如果`innodb_undo_directory`变量未定义，则默认撤销表空间位于数据目录中。如果在服务器离线状态下移动默认撤销表空间，则必须使用配置为新目录的`innodb_undo_directory`变量启动服务器。

撤销日志的 I/O 模式使撤销表空间成为 SSD 存储的良好选择。

##### 配置回滚段的数量

`innodb_rollback_segments`变量定义了分配给每个撤销表空间和全局临时表空间的回滚段的数量。`innodb_rollback_segments`变量可以在启动时或服务器运行时进行配置。

`innodb_rollback_segments`的默认设置为 128，这也是最大值。有关回滚段支持的事务数量的信息，请参见第 17.6.6 节，“撤销日志”。

##### 截断撤销表空间

有两种截断撤销表空间的方法，可以单独使用或结合使用来管理撤销表空间的大小。一种方法是自动的，使用配置变量启用。另一种方法是手动的，使用 SQL 语句执行。

自动方法不需要监视撤销表空间的大小，并且一旦启用，它会执行撤销表空间的停用、截断和重新激活，无需手动干预。如果您希望控制何时将撤销表空间脱机进行截断，则可能更喜欢手动截断方法。例如，您可能希望避免在高负载时间截断撤销表空间。

###### 自动截断

自动截断撤销表空间需要至少两个活动撤销表空间，这确保了一个撤销表空间保持活动状态，而另一个被脱机进行截断。默认情况下，在初始化 MySQL 实例时会创建两个撤销表空间。

要自动截断撤销表空间，请启用`innodb_undo_log_truncate`变量。例如：

```sql
mysql> SET GLOBAL innodb_undo_log_truncate=ON;
```

当启用`innodb_undo_log_truncate`变量时，超过`innodb_max_undo_log_size`变量定义的大小限制的撤销表空间将被截断。`innodb_max_undo_log_size`变量是动态的，默认值为 1073741824 字节（1024 MiB）。

```sql
mysql> SELECT @@innodb_max_undo_log_size;
+----------------------------+
| @@innodb_max_undo_log_size |
+----------------------------+
|                 1073741824 |
+----------------------------+
```

当启用`innodb_undo_log_truncate`变量时：

1.  超过`innodb_max_undo_log_size`设置的默认和用户定义的撤销表空间将被标记为截断。选择要截断的撤销表空间是循环进行的，以避免每次截断相同的撤销表空间。

1.  位于所选撤销表空间中的回滚段被设置为不活动，以便它们不分配给新事务。当前正在使用回滚段的现有事务被允许完成。

1.  清除系统通过释放不再使用的撤销日志来清空回滚段。

1.  在撤销表空间中的所有回滚段被释放后，截断操作运行并将撤销表空间截断至其初始大小。

    截断操作后的撤销表空间大小可能比初始大小大，因为操作完成后立即使用。

    `innodb_undo_directory`变量定义默认撤销表空间文件的位置。如果未定义`innodb_undo_directory`变量，则默认撤销表空间位于数据目录中。包括使用`CREATE UNDO TABLESPACE`语法创建的用户定义撤销表空间在内的所有撤销表空间文件的位置可以通过查询信息模式`FILES`表来确定：

    ```sql
    SELECT TABLESPACE_NAME, FILE_NAME FROM INFORMATION_SCHEMA.FILES WHERE FILE_TYPE LIKE 'UNDO LOG';
    ```

1.  回滚段被重新激活，以便分配给新事务。

###### 手动截断

手动截断撤销表空间需要至少三个活动的撤销表空间。始终需要两个活动的撤销表空间来支持自动截断的可能性。三个撤销表空间的最低要求满足此要求，同时允许手动将撤销表空间下线。

要手动启动撤销表空间的截断，请发出以下语句：

```sql
ALTER UNDO TABLESPACE *tablespace_name* SET INACTIVE;
```

撤销表空间标记为不活动后，当前正在使用撤销表空间中回滚段的事务被允许完成，以及在这些事务完成之前启动的任何事务。事务完成后，清除系统释放撤销表空间中的回滚段，撤销表空间被截断至其初始大小，并且撤销表空间状态从`inactive`变为`empty`。

注意

当`ALTER UNDO TABLESPACE *`tablespace_name`* SET INACTIVE`语句停用撤销表空间时，清除线程会在下一个机会查找该撤销表空间。一旦找到撤销表空间并标记为截断，清除线程会以增加的频率返回，以快速清空和截断撤销表空间。

要检查撤销表空间的状态，请查询信息模式`INNODB_TABLESPACES`表。

```sql
SELECT NAME, STATE FROM INFORMATION_SCHEMA.INNODB_TABLESPACES
  WHERE NAME LIKE '*tablespace_name*';
```

一旦撤销表空间处于`empty`状态，可以通过发出以下语句重新激活：

```sql
ALTER UNDO TABLESPACE *tablespace_name* SET ACTIVE;
```

处于`空`状态的撤消表空间也可以被删除。请参阅删除撤消表空间。

###### 加快自动截断撤消表空间

清除线程负责清空和截断撤消表空间。默认情况下，清除线程在调用清除的 128 次中查找撤消表空间以截断一次。清除线程查找撤消表空间以截断的频率由`innodb_purge_rseg_truncate_frequency`变量控制，默认设置为 128。

```sql
mysql> SELECT @@innodb_purge_rseg_truncate_frequency;
+----------------------------------------+
| @@innodb_purge_rseg_truncate_frequency |
+----------------------------------------+
|                                    128 |
+----------------------------------------+
```

要增加频率，请降低`innodb_purge_rseg_truncate_frequency`设置。例如，为了使清除线程在调用清除的 32 次中查找撤消表空间一次，将`innodb_purge_rseg_truncate_frequency`设置为 32。

```sql
mysql> SET GLOBAL innodb_purge_rseg_truncate_frequency=32;
```

###### 截断撤消表空间文件的性能影响

当撤消表空间被截断时，撤消表空间中的回滚段将被停用。其他撤消表空间中的活动回滚段将负责整个系统负载，这可能导致轻微的性能下降。性能受影响的程度取决于多个因素：

+   撤消表空间数量

+   撤消日志数量

+   撤消表空间大小

+   I/O 子系统的速度

+   现有的长时间运行事务

+   系统负载

避免潜在性能影响的最简单方法是增加撤消表空间的数量。

###### 监控撤消表空间截断

从 MySQL 8.0.16 开始，提供了用于监视与撤消日志截断相关的后台活动的`undo`和`purge`子系统计数器。有关计数器名称和描述，请查询信息模式`INNODB_METRICS`表。

```sql
SELECT NAME, SUBSYSTEM, COMMENT FROM INFORMATION_SCHEMA.INNODB_METRICS WHERE NAME LIKE '%truncate%';
```

有关启用计数器和查询计数器数据的信息，请参阅第 17.15.6 节，“InnoDB INFORMATION_SCHEMA Metrics Table”。

###### 撤消表空间截断限制

从 MySQL 8.0.21 开始，同一撤消表空间在检查点之间的截断操作次数限制为 64 次。该限制防止由于在繁忙系统上设置了过低的`innodb_max_undo_log_size`而导致的过多撤消表空间截断操作可能引起的潜在问题。如果超过限制，撤消表空间仍然可以变为非活动状态，但直到下一个检查点之后才会被截断。在 MySQL 8.0.22 中，该限制从 64 提高到了 50,000。

###### 撤消表空间截断恢复

撤销表空间截断操作会在服务器日志目录中创建一个临时`undo_*`space_number`*_trunc.log`文件。该日志目录由`innodb_log_group_home_dir`定义。如果在截断操作期间发生系统故障，临时日志文件允许启动过程识别正在被截断的撤销表空间，并继续操作。

##### 撤销表空间状态变量

以下状态变量允许跟踪总撤销表空间数、隐式（`InnoDB`创建的）撤销表空间数、显式（用户创建的）撤销表空间数以及活动撤销表空间数：

```sql
mysql> SHOW STATUS LIKE 'Innodb_undo_tablespaces%';
+----------------------------------+-------+
| Variable_name                    | Value |
+----------------------------------+-------+
| Innodb_undo_tablespaces_total    | 2     |
| Innodb_undo_tablespaces_implicit | 2     |
| Innodb_undo_tablespaces_explicit | 0     |
| Innodb_undo_tablespaces_active   | 2     |
+----------------------------------+-------+
```

查看状态变量描述，请参阅第 7.1.10 节，“服务器状态变量”。
