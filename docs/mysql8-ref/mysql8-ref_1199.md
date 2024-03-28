# 17.8.2 配置 InnoDB 为只读操作

> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-read-only-instance.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-read-only-instance.html)

通过在服务器启动时启用`--innodb-read-only`配置选项，可以查询`InnoDB`表，其中 MySQL 数据目录位于只读介质上。

#### 如何启用

为了准备一个实例进行只读操作，在将其存储在只读介质上之前，确保所有必要的信息都被刷新到数据文件中。运行服务器时禁用更改缓冲（`innodb_change_buffering=0`），并进行慢关闭。

要为整个 MySQL 实例启用只读模式，请在服务器启动时指定以下配置选项：

+   `--innodb-read-only=1`

+   如果实例在只读介质上，如 DVD 或 CD，或者`/var`目录不可被所有用户写入：`--pid-file=*`path_on_writeable_media`*` 和 `--event-scheduler=disabled`

+   `--innodb-temp-data-file-path`。此选项指定了`InnoDB`临时表空间数据文件的路径、文件名和文件大小。默认设置为`ibtmp1:12M:autoextend`，这将在数据目录中创建`ibtmp1`临时表空间数据文件。为了准备一个实例进行只读操作，将`innodb_temp_data_file_path`设置为数据目录之外的位置。路径必须相对于数据目录。例如：

    ```sql
    --innodb-temp-data-file-path=../../../tmp/ibtmp1:12M:autoextend
    ```

从 MySQL 8.0 开始，启用`innodb_read_only`会阻止所有存储引擎的表创建和删除操作。这些操作修改了`mysql`系统数据库中的数据字典表，但这些表使用了`InnoDB`存储引擎，当启用`innodb_read_only`时无法修改。相同的限制也适用于任何修改数据字典表的操作，如`ANALYZE TABLE` 和 `ALTER TABLE *`tbl_name`* ENGINE=*`engine_name`*`。

此外，MySQL 8.0 中的`mysql`系统数据库中的其他表使用`InnoDB`存储引擎。将这些表设置为只读会导致对修改它们的操作的限制。例如，在只读模式下不允许`CREATE USER`、`GRANT`、`REVOKE`和`INSTALL PLUGIN`操作。

#### 使用场景

这种操作模式适用于以下情况：

+   在只读存储介质（如 DVD 或 CD）上分发 MySQL 应用程序或一组 MySQL 数据。

+   多个同时查询相同数据目录的 MySQL 实例，通常在数据仓库配置中。您可以使用这种技术来避免在负载较重的 MySQL 实例中可能出现的瓶颈，或者您可以为各个实例使用不同的配置选项，为每个实例调整特定类型的查询。

+   查询已被放入只读状态以确保安全性或数据完整性的数据，例如已存档的备份数据。

注意

此功能主要用于在分发和部署方面提供灵活性，而不是基于只读方面的原始性能。请参阅第 10.5.3 节，“优化 InnoDB 只读事务”以了解如何调整只读查询的性能，这不需要使整个服务器变为只读。

#### 工作原理

当服务器通过`--innodb-read-only`选项以只读模式运行时，某些`InnoDB`功能和组件会减少或完全关闭：

+   不进行更改缓冲，特别是不进行来自更改缓冲区的合并。为确保在准备实例进行只读操作时更改缓冲区为空，请禁用更改缓冲（`innodb_change_buffering=0`）并首先进行慢关闭。

+   启动时没有崩溃恢复阶段。实例必须在被置于只读状态之前执行慢关闭。

+   因为重做日志在只读操作中不被使用，您可以在将实例设置为只读之前将`innodb_log_file_size`设置为可能的最小大小（1 MB）。

+   大多数后台线程被关闭。I/O 读线程保持开启，以及 I/O 写线程和用于写入临时文件的页面刷新协调器线程，这在只读模式下是允许的。一个缓冲池调整线程也保持活动状态，以便在线调整缓冲池大小。

+   关于死锁、监视器输出等信息不会写入临时文件。因此，`SHOW ENGINE INNODB STATUS`不会产生任何输出。

+   当服务器处于只读模式时，通常会更改写操作行为的配置选项设置更改不会产生任何效果。

+   用于执行隔离级别的 MVCC 处理被关闭。所有查询读取记录的最新版本，因为更新和删除是不可能的。

+   撤销日志未被使用。禁用`innodb_undo_tablespaces`和`innodb_undo_directory`配置选项的任何设置。
