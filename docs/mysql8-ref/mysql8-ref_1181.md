# 17.6.5 重做日志

> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-redo-log.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-redo-log.html)

重做日志是一种基于磁盘的数据结构，在崩溃恢复期间用于纠正由不完整事务写入的数据。在正常操作期间，重做日志对由 SQL 语句或低级 API 调用导致的更改表数据的请求进行编码。在意外关闭之前未完成更新数据文件的修改在初始化期间和在接受连接之前自动重放。有关重做日志在崩溃恢复中的作用的信息，请参见第 17.18.2 节，“InnoDB 恢复”。

重做日志在磁盘上以重做日志文件的形式物理表示。写入重做日志文件的数据以受影响的记录为编码，这些数据被统称为重做。数据通过重做日志文件的传递由一个不断增加的 LSN 值表示。重做日志数据在数据修改发生时追加，而最旧的数据在检查点进展时被截断。

有关重做日志的信息和程序在以下主题中描述：

+   配置重做日志容量（MySQL 8.0.30 或更高版本）

+   配置重做日志容量（MySQL 8.0.30 之前）

+   自动重做日志容量配置

+   重做日志归档

+   禁用重做日志记录

+   相关主题

#### 配置重做日志容量（MySQL 8.0.30 或更高版本）

从 MySQL 8.0.30 开始，`innodb_redo_log_capacity`系统变量控制重做日志文件占用的磁盘空间量。您可以在启动时或运行时使用`SET GLOBAL`语句在选项文件中设置此变量；例如，以下语句将重做日志容量设置为 8GB：

```sql
SET GLOBAL innodb_redo_log_capacity = 8589934592;
```

在运行时设置时，配置更改会立即生效，但新限制可能需要一些时间才能完全实施。如果重做日志文件占用的空间少于指定值，则从缓冲池中更积极地刷新脏页到表空间数据文件，最终增加重做日志文件占用的磁盘空间。如果重做日志文件占用的空间多于指定值，则更积极地刷新脏页，最终减少重做日志文件占用的磁盘空间。

`innodb_redo_log_capacity`变量取代了已弃用的`innodb_log_files_in_group`和`innodb_log_file_size`变量。当定义了`innodb_redo_log_capacity`设置时，`innodb_log_files_in_group`和`innodb_log_file_size`设置将被忽略；否则，这些设置将用于计算`innodb_redo_log_capacity`设置（`innodb_log_files_in_group` * `innodb_log_file_size` = `innodb_redo_log_capacity`）。如果没有设置这些变量中的任何一个，重做日志容量将设置为`innodb_redo_log_capacity`的默认值，即 104857600 字节（100MB）。最大重做日志容量为 128GB。

重做日志文件位于数据目录中的`#innodb_redo`目录中，除非通过`innodb_log_group_home_dir`变量指定了不同的目录。如果定义了`innodb_log_group_home_dir`，则重做日志文件位于该目录中的`#innodb_redo`目录中。有两种类型的重做日志文件，普通和备用。普通的重做日志文件是正在使用的。备用的重做日志文件是等待使用的。`InnoDB`尝试总共维护 32 个重做日志文件，每个文件的大小等于 1/32 * `innodb_redo_log_capacity`；但是，在修改`innodb_redo_log_capacity`设置后，文件大小可能会有所不同。

重做日志文件使用`#ib_redo*`N`*`的命名约定，其中*`N`*是重做日志文件编号。备用的重做日志文件以`_tmp`后缀表示。以下示例显示了一个`#innodb_redo`目录中的重做日志文件，其中有 21 个活动重做日志文件和 11 个备用重做日志文件，按顺序编号。

```sql
'#ib_redo582'  '#ib_redo590'  '#ib_redo598'      '#ib_redo606_tmp'
'#ib_redo583'  '#ib_redo591'  '#ib_redo599'      '#ib_redo607_tmp'
'#ib_redo584'  '#ib_redo592'  '#ib_redo600'      '#ib_redo608_tmp'
'#ib_redo585'  '#ib_redo593'  '#ib_redo601'      '#ib_redo609_tmp'
'#ib_redo586'  '#ib_redo594'  '#ib_redo602'      '#ib_redo610_tmp'
'#ib_redo587'  '#ib_redo595'  '#ib_redo603_tmp'  '#ib_redo611_tmp'
'#ib_redo588'  '#ib_redo596'  '#ib_redo604_tmp'  '#ib_redo612_tmp'
'#ib_redo589'  '#ib_redo597'  '#ib_redo605_tmp'  '#ib_redo613_tmp'
```

每个普通的重做日志文件与特定范围的 LSN 值相关联；例如，以下查询显示了前面示例中列出的活动重做日志文件的`START_LSN`和`END_LSN`值：

```sql
mysql> SELECT FILE_NAME, START_LSN, END_LSN FROM performance_schema.innodb_redo_log_files;
+----------------------------+--------------+--------------+
| FILE_NAME                  | START_LSN    | END_LSN      |
+----------------------------+--------------+--------------+
| ./#innodb_redo/#ib_redo582 | 117654982144 | 117658256896 |
| ./#innodb_redo/#ib_redo583 | 117658256896 | 117661531648 |
| ./#innodb_redo/#ib_redo584 | 117661531648 | 117664806400 |
| ./#innodb_redo/#ib_redo585 | 117664806400 | 117668081152 |
| ./#innodb_redo/#ib_redo586 | 117668081152 | 117671355904 |
| ./#innodb_redo/#ib_redo587 | 117671355904 | 117674630656 |
| ./#innodb_redo/#ib_redo588 | 117674630656 | 117677905408 |
| ./#innodb_redo/#ib_redo589 | 117677905408 | 117681180160 |
| ./#innodb_redo/#ib_redo590 | 117681180160 | 117684454912 |
| ./#innodb_redo/#ib_redo591 | 117684454912 | 117687729664 |
| ./#innodb_redo/#ib_redo592 | 117687729664 | 117691004416 |
| ./#innodb_redo/#ib_redo593 | 117691004416 | 117694279168 |
| ./#innodb_redo/#ib_redo594 | 117694279168 | 117697553920 |
| ./#innodb_redo/#ib_redo595 | 117697553920 | 117700828672 |
| ./#innodb_redo/#ib_redo596 | 117700828672 | 117704103424 |
| ./#innodb_redo/#ib_redo597 | 117704103424 | 117707378176 |
| ./#innodb_redo/#ib_redo598 | 117707378176 | 117710652928 |
| ./#innodb_redo/#ib_redo599 | 117710652928 | 117713927680 |
| ./#innodb_redo/#ib_redo600 | 117713927680 | 117717202432 |
| ./#innodb_redo/#ib_redo601 | 117717202432 | 117720477184 |
| ./#innodb_redo/#ib_redo602 | 117720477184 | 117723751936 |
+----------------------------+--------------+--------------+
```

在执行检查点时，`InnoDB`将检查点 LSN 存储在包含此 LSN 的文件的头部。在恢复过程中，所有重做日志文件都会被检查，恢复从最新的检查点 LSN 开始。

提供了几个状态变量用于监视重做日志和重做日志容量调整操作；例如，您可以查询`Innodb_redo_log_resize_status`以查看调整操作的状态：

```sql
mysql> SHOW STATUS LIKE 'Innodb_redo_log_resize_status';
+-------------------------------+-------+
| Variable_name                 | Value |
+-------------------------------+-------+
| Innodb_redo_log_resize_status | OK    |
+-------------------------------+-------+
```

`Innodb_redo_log_capacity_resized`状态变量显示当前重做日志容量限制：

```sql
mysql> SHOW STATUS LIKE 'Innodb_redo_log_capacity_resized';
 +----------------------------------+-----------+
| Variable_name                    | Value     |
+----------------------------------+-----------+
| Innodb_redo_log_capacity_resized | 104857600 |
+----------------------------------+-----------+
```

其他适用的状态变量包括：

+   `Innodb_redo_log_checkpoint_lsn`

+   `Innodb_redo_log_current_lsn`

+   `Innodb_redo_log_flushed_to_disk_lsn`

+   `Innodb_redo_log_logical_size`

+   `Innodb_redo_log_physical_size`

+   `Innodb_redo_log_read_only`

+   `Innodb_redo_log_uuid`

请参考状态变量描述以获取更多信息。

您可以通过查询`innodb_redo_log_files`性能模式表查看有关活动重做日志文件的信息。以下查询检索所有表列的数据：

```sql
SELECT FILE_ID, START_LSN, END_LSN, SIZE_IN_BYTES, IS_FULL, CONSUMER_LEVEL 
FROM performance_schema.innodb_redo_log_files;
```

#### 配置重做日志容量（MySQL 8.0.30 之前）

在 MySQL 8.0.30 之前，默认情况下，`InnoDB`在数据目录中创建两个重做日志文件，命名为`ib_logfile0`和`ib_logfile1`，并以循环方式写入这些文件。

修改重做日志容量需要更改重做日志文件的数量或大小，或两者兼而有之。

1.  停止 MySQL 服务器，并确保它在没有错误的情况下关闭。

1.  编辑`my.cnf`以更改重做日志文件配置。要更改重做日志文件大小，请配置`innodb_log_file_size`。要增加重做日志文件数量，请配置`innodb_log_files_in_group`。

1.  再次启动 MySQL 服务器。

如果`InnoDB`检测到`innodb_log_file_size`与重做日志文件大小不同，它会写入日志检查点，关闭并删除旧日志文件，以请求的大小创建新日志文件，并打开新日志文件。

#### 自动重做日志容量配置

当启用`innodb_dedicated_server`时，`InnoDB`会自动配置某些`InnoDB`参数，包括重做日志容量。自动配置适用于驻留在专用于 MySQL 的服务器上的 MySQL 实例，其中 MySQL 服务器可以使用所有可用的系统资源。有关更多信息，请参见第 17.8.12 节，“为专用 MySQL 服务器启用自动配置”。

#### 重做日志归档

备份工具有时可能在备份操作进行时未能跟上重做日志生成的速度，导致由于这些记录被覆盖而丢失重做日志记录。这个问题在备份操作期间有大量 MySQL 服务器活动，并且重做日志文件存储介质的运行速度比备份存储介质快时最常发生。MySQL 8.0.17 中引入的重做日志归档功能通过将重做日志记录顺序写入归档文件来解决这个问题，除了重做日志文件外。备份工具可以根据需要从归档文件中复制重做日志记录，从而避免数据的潜在丢失。

如果在服务器上配置了重做日志归档，MySQL 企业备份，可用于[MySQL 企业版](https://www.mysql.com/products/enterprise/)，在备份 MySQL 服务器时使用重做日志归档功能。

在服务器上启用重做日志归档需要为`innodb_redo_log_archive_dirs`系统变量设置一个值。该值被指定为带标签的重做日志归档目录的分号分隔列表。`*`label:directory`*`对由冒号(`:`)分隔。例如：

```sql
mysql> SET GLOBAL innodb_redo_log_archive_dirs='*label1*:*directory_path1*[;*label2*:*directory_path2*;…]';
```

*`label`*是归档目录的任意标识符。它可以是任何字符的字符串，但不允许使用冒号(:)。空标签也是允许的，但在这种情况下仍然需要冒号(:)。必须指定*`directory_path`*。在激活重做日志归档时，用于重做日志归档文件的目录必须存在，否则会返回错误。路径可以包含冒号(':')，但不允许使用分号(;)。

必须在激活重做日志归档之前配置`innodb_redo_log_archive_dirs`变量。默认值为`NULL`，不允许激活重做日志归档。

注意

您指定的归档目录必须满足以下要求。（这些要求在激活重做日志归档时会被强制执行。）：

+   目录必须存在。目录不会被重做日志归档过程创建。否则，将返回以下错误：

    错误 3844 (HY000)：重做日志归档目录'*`directory_path1`*'不存在���不是目录

+   目录不能对所有用户开放访问。这是为了防止重做日志数据暴露给系统上的未经授权用户。否则，将返回以下错误：

    错误 3846 (HY000)：重做日志归档目录'*`directory_path1`*'对所有操作系统用户可访问

+   目录不能是由`datadir`、`innodb_data_home_dir`、`innodb_directories`、`innodb_log_group_home_dir`、`innodb_temp_tablespaces_dir`、`innodb_tmpdir`、`innodb_undo_directory`或`secure_file_priv`定义的目录，也不能是这些目录的父目录或子目录。否则，将返回类似以下的错误：

    错误 3845 (HY000)：重做日志归档目录'*`directory_path1`*'在服务器目录'datadir' - '*`/path/to/data_directory`*'中、下或上

当支持重做日志归档的备份实用程序启动备份时，备份实用程序通过调用`innodb_redo_log_archive_start()`函数激活重做日志归档。

如果您没有使用支持重做日志归档的备份实用程序，也可以手动激活重做日志归档，如下所示：

```sql
mysql> SELECT innodb_redo_log_archive_start('*label*', '*subdir*');
+------------------------------------------+
| innodb_redo_log_archive_start('*label*') |
+------------------------------------------+
| 0                                        |
+------------------------------------------+
```

或：

```sql
mysql> DO innodb_redo_log_archive_start('*label*', '*subdir*');
Query OK, 0 rows affected (0.09 sec)
```

注意

激活重做日志归档的 MySQL 会话（使用`innodb_redo_log_archive_start()`）必须保持打开状态以进行归档。相同的会话必须停用重做日志归档（使用`innodb_redo_log_archive_stop()`）。如果会话在显式停用重做日志归档之前终止，则服务器会隐式停用重做日志归档并删除重做日志归档文件。

其中*`label`*是由`innodb_redo_log_archive_dirs`定义的标签；`subdir`是用于指定保存归档文件的*`label`*标识的目录的子目录的可选参数；它必须是一个简单的目录名称（不允许斜杠（/）、反斜杠（\）或冒号（:））。`subdir`可以为空，为 null，或者可以省略。

只有具有`INNODB_REDO_LOG_ARCHIVE`权限的用户才能通过调用`innodb_redo_log_archive_start()`激活重做日志归档，或使用`innodb_redo_log_archive_stop()`停用它。运行备份实用程序的 MySQL 用户或手动激活和停用重做日志归档的 MySQL 用户必须具有此权限。

重做日志归档文件路径为`*`directory_identified_by_label`*/[*`subdir`*/]archive.*`serverUUID`*.000001.log`，其中`*`directory_identified_by_label`*`是由`innodb_redo_log_archive_start()`的`*`label`*`参数标识的归档目录。`*`subdir`*`是用于`innodb_redo_log_archive_start()`的可选参数。

例如，重做日志归档文件的完整路径和名称类似于以下内容：

```sql
/*directory_path*/*subdirectory*/archive.e71a47dc-61f8-11e9-a3cb-080027154b4d.000001.log
```

备份工具完成复制`InnoDB`数据文件后，通过调用`innodb_redo_log_archive_stop()`函数来停用重做日志归档。

如果您没有使用支持重做日志归档的备份工具，也可以手动停用重做日志归档，如下所示：

```sql
mysql> SELECT innodb_redo_log_archive_stop();
+--------------------------------+
| innodb_redo_log_archive_stop() |
+--------------------------------+
| 0                              |
+--------------------------------+
```

或者：

```sql
mysql> DO innodb_redo_log_archive_stop();
Query OK, 0 rows affected (0.01 sec)
```

在停止函数成功完成后，备份工具会从归档文件中查找相关部分的重做日志数据，并将其复制到备份中。

备份工具完成复制重做日志数据并不再需要重做日志归档文件后，会删除该归档文件。

在正常情况下，归档文件的删除是备份工具的责任。但是，如果重做日志归档操作在调用`innodb_redo_log_archive_stop()`之前意外退出，则 MySQL 服务器会删除该文件。

##### 性能考虑

激活重做日志归档通常会因为额外的写入活动而产生轻微的性能成本。

在 Unix 和类 Unix 操作系统上，性能影响通常较小，假设没有持续高速更新。在 Windows 上，性能影响通常略高，假设情况相同。

如果更新速率持续较高且重做日志归档文件与重做日志文件位于相同存储介质上，则由于复合写入活动，性能影响可能更为显著。

如果更新速率持续较高且重做日志归档文件位于比重做日志文件更慢的存储介质上，则性能会受到任意影响。

写入重做日志归档文件不会妨碍正常的事务日志记录，除非重做日志归档文件存储介质的速率远远低于重做日志文件存储介质，并且有大量持久化的重做日志块等待写入重做日志归档文件。在这种情况下，事务日志记录速率会降低到可以由重做日志归档文件所在的较慢存储介质管理的水平。

#### 禁用重做日志记录

截至 MySQL 8.0.21 版本，您可以使用`ALTER INSTANCE DISABLE INNODB REDO_LOG`语句禁用重做日志记录。此功能旨在将数据加载到新的 MySQL 实例中。禁用重做日志记录通过避免重做日志写入和双写缓冲来加快数据加载速度。

警告

此功能仅用于将数据加载到新的 MySQL 实例中。*不要在生产系统上禁用重做日志*。允许在禁用重做日志时关闭和重新启动服务器，但在禁用重做日志时发生意外服务器停止可能会导致数据丢失和实例损坏。

在禁用重做日志后尝试重新启动服务器会被拒绝，并显示以下错误：

```sql
[ERROR] [MY-013598] [InnoDB] Server was killed when Innodb Redo 
logging was disabled. Data files could be corrupt. You can try 
to restart the database with innodb_force_recovery=6
```

在这种情况下，初始化一个新的 MySQL 实例并重新启动数据加载过程。

启用和禁用重做日志需要`INNODB_REDO_LOG_ENABLE`权限。

`Innodb_redo_log_enabled`状态变量允许监视重做日志状态。

在禁用重做日志时，不允许克隆操作和重做日志归档，反之亦然。

[`ALTER INSTANCE [ENABLE|DISABLE] INNODB REDO_LOG`](alter-instance.html "15.1.5 ALTER INSTANCE 语句")操作需要独占备份元数据锁，这会阻止其他`ALTER INSTANCE`操作并发执行。其他`ALTER INSTANCE`操作必须等待锁释放后才能执行。

以下过程演示了在将数据加载到新的 MySQL 实例时如何禁用重做日志。

1.  在新的 MySQL 实例上，向负责禁用重做日志的用户帐户授予`INNODB_REDO_LOG_ENABLE`权限。

    ```sql
    mysql> GRANT INNODB_REDO_LOG_ENABLE ON *.* to 'data_load_admin';
    ```

1.  作为`data_load_admin`用户，禁用重做日志：

    ```sql
    mysql> ALTER INSTANCE DISABLE INNODB REDO_LOG;
    ```

1.  检查`Innodb_redo_log_enabled`状态变量，确保重做日志已禁用。

    ```sql
    mysql> SHOW GLOBAL STATUS LIKE 'Innodb_redo_log_enabled';
    +-------------------------+-------+
    | Variable_name           | Value |
    +-------------------------+-------+
    | Innodb_redo_log_enabled | OFF   |
    +-------------------------+-------+
    ```

1.  运行数据加载操作。

1.  作为`data_load_admin`用户，在数据加载操作完成后启用重做日志：

    ```sql
    mysql> ALTER INSTANCE ENABLE INNODB REDO_LOG;
    ```

1.  检查`Innodb_redo_log_enabled`状态变量，确保重做日志已启用。

    ```sql
    mysql> SHOW GLOBAL STATUS LIKE 'Innodb_redo_log_enabled';
    +-------------------------+-------+
    | Variable_name           | Value |
    +-------------------------+-------+
    | Innodb_redo_log_enabled | ON    |
    +-------------------------+-------+
    ```

#### 相关主题

+   重做日志配置

+   第 10.5.4 节，“优化 InnoDB 重做日志”

+   重做日志加密
