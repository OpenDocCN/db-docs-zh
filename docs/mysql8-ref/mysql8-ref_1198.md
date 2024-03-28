# 17.8.1 InnoDB 启动配置

> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-init-startup-configuration.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-init-startup-configuration.html)

关于 `InnoDB` 配置的首要决策涉及数据文件、日志文件、页面大小和内存缓冲区的配置，这些应在初始化 `InnoDB` 之前配置。在初始化 `InnoDB` 后修改配置可能涉及非平凡的程序。

本节提供了有关在配置文件中指定 `InnoDB` 设置、查看 `InnoDB` 初始化信息和重要存储考虑事项的信息。

+   在 MySQL 选项文件中指定选项

+   查看 InnoDB 初始化信息

+   重要的存储考虑

+   系统表空间数据文件配置

+   InnoDB 双写缓冲文件配置

+   重做日志配置

+   撤销表空间配置

+   全局临时表空间配置

+   会话临时表空间配置

+   页面大小配置

+   内存配置

#### 在 MySQL 选项文件中指定选项

因为 MySQL 使用数据文件、日志文件和页面大小设置来初始化 `InnoDB`，建议您在 MySQL 在启动时读取的选项文件中定义这些设置，以便在初始化 `InnoDB` 之前。通常情况下，当 MySQL 服务器首次启动时会初始化 `InnoDB`。

您可以将 `InnoDB` 选项放在服务器启动时读取的任何选项文件的 `[mysqld]` 组中。MySQL 选项文件的位置在 Section 6.2.2.2, “Using Option Files” 中有描述。

为了确保**mysqld**仅从特定文件（和`mysqld-auto.cnf`）中读取选项，请在启动服务器时将`--defaults-file`选项作为命令行中的第一个选项：

```sql
mysqld --defaults-file=*path_to_option_file*
```

#### 查看 InnoDB 初始化信息

要查看启动期间的`InnoDB`初始化信息，请从命令提示符启动**mysqld**，这会将初始化信息打印到控制台。

例如，在 Windows 上，如果**mysqld**位于`C:\Program Files\MySQL\MySQL Server 8.0\bin`，可以这样启动 MySQL 服务器：

```sql
C:\> "C:\Program Files\MySQL\MySQL Server 8.0\bin\mysqld" --console
```

在类 Unix 系统上，**mysqld**位于 MySQL 安装目录的`bin`目录中：

```sql
$> bin/mysqld --user=mysql &
```

如果您没有将服务器输出发送到控制台，请在启动后检查错误日志，查看在启动过程中打印的`InnoDB`初始化信息。

有关使用其他方法启动 MySQL 的信息，请参见 Section 2.9.5, “Starting and Stopping MySQL Automatically”。

注意

`InnoDB`在启动时不会打开所有用户表和相关数据文件。但是，`InnoDB`会检查数据字典中引用的表空间文件是否存在。如果找不到表空间文件，`InnoDB`会记录错误并继续启动序列。重做日志中引用的表空间文件可能在崩溃恢复期间打开以进行重做应用。

#### 重要的存储考虑事项

在继续进行启动配置之前，请查看以下与存储相关的考虑事项。

+   在某些情况下，通过将数据和日志文件放在不同的物理磁盘上，可以提高数据库性能。您还可以为`InnoDB`数据文件使用原始磁盘分区（原始设备），这可能加快 I/O 速度。请参阅 Using Raw Disk Partitions for the System Tablespace。

+   `InnoDB`是一个支持事务的（符合 ACID 标准）存储引擎，具有提交、回滚和崩溃恢复功能，以保护用户数据。**但是，如果底层操作系统或硬件不按照宣传的方式工作，它就无法做到**。许多操作系统或磁盘子系统可能会延迟或重新排序写操作以提高性能。在某些操作系统上，应该等待直到文件的所有未写入数据都已刷新的`fsync()`系统调用实际上可能会在数据刷新到稳定存储之前返回。由于这个原因，操作系统崩溃或停电可能会破坏最近提交的数据，或者在最坏的情况下，甚至损坏数据库，因为写操作已经被重新排序。如果数据完整性对您很重要，请在生产环境中使用任何内容之前执行“拔插头”测试。在 macOS 上，`InnoDB`使用一种特殊的`fcntl()`文件刷新方法。在 Linux 下，建议**禁用写回缓存**。

    在 ATA/SATA 硬盘驱动器上，像`hdparm -W0 /dev/hda`这样的命令可能可以禁用写回缓存。**请注意，一些驱动器或磁盘控制器可能无法禁用写回缓存。**

+   关于保护用户数据的`InnoDB`恢复能力，`InnoDB`使用一种涉及名为双写缓冲区的结构的文件刷新技术，默认情况下启用（`innodb_doublewrite=ON`）。双写缓冲区在意外退出或停电后的恢复过程中增加了安全性，并通过减少大多数 Unix 系统上`fsync()`操作的需求来提高性能。如果您关心数据完整性或可能的故障，请保持`innodb_doublewrite`选项启用。有关双写缓冲区的信息，请参阅第 17.11.1 节，“InnoDB 磁盘 I/O”。

+   在使用`InnoDB`与 NFS 之前，请查看使用 NFS 与 MySQL 中概述的潜在问题。

#### 系统表空间数据文件配置

`innodb_data_file_path`选项定义了`InnoDB`系统表空间数据文件的名称、大小和属性。如果在初始化 MySQL 服务器之前未配置此选项，则默认行为是创建一个稍大于 12MB 的单个自动扩展数据文件，名称为`ibdata1`：

```sql
mysql> SHOW VARIABLES LIKE 'innodb_data_file_path';
+-----------------------+------------------------+
| Variable_name         | Value                  |
+-----------------------+------------------------+
| innodb_data_file_path | ibdata1:12M:autoextend |
+-----------------------+------------------------+
```

完整的数据文件规范语法包括文件名、文件大小、`autoextend`属性和`max`属性：

```sql
*file_name*:*file_size*[:autoextend[:max:*max_file_size*]]
```

文件大小以 KB、MB 或 GB 为单位指定，通过在大小值后附加`K`、`M`或`G`来实现。如果以 KB 为单位指定数据文件大小，请以 1024 的倍数进行。否则，KB 值将四舍五入到最近的兆字节（MB）边界。文件大小之和必须至少略大于 12MB。

您可以使用分号分隔的列表指定多个数据文件。例如：

```sql
[mysqld]
innodb_data_file_path=ibdata1:50M;ibdata2:50M:autoextend
```

`autoextend`和`max`属性只能用于指定的最后一个数据文件。

当指定`autoextend`属性时，数据文件会根据需要自动增加 64MB 的增量。`innodb_autoextend_increment`变量控制增量大小。

要为自动扩展数据文件指定最大大小，请在`autoextend`属性后使用`max`属性。只有在约束磁盘使用量至关重要的情况下才使用`max`属性。以下配置允许`ibdata1`增长到 500MB 的限制：

```sql
[mysqld]
innodb_data_file_path=ibdata1:12M:autoextend:max:500M
```

为*第一个*系统表空间数据文件强制执行最小文件大小，以确保有足够的空间用于双写缓冲区页。以下表显示了每个`InnoDB`页大小的最小文件大小。默认的`InnoDB`页大小为 16384（16KB）。

| 页大小（innodb_page_size） | 最小文件大小 |
| --- | --- |
| 16384 (16KB) 或更少 | 3MB |
| 32768 (32KB) | 6MB |
| 65536 (64KB) | 12MB |

如果您的磁盘已满，可以在另一个磁盘上添加数据文件。有关说明，请参阅调整系统表空间大小。

个别文件的大小限制由您的操作系统确定。在支持大文件的操作系统上，您可以将文件大小设置为超过 4GB。您还可以将原始磁盘分区用作数据文件。请参阅使用原始磁盘分区作为系统表空间。

`InnoDB` 不了解文件系统的最大文件大小，因此在最大文件大小为 2GB 等较小值的文件系统上要小心。

默认情况下，系统表空间文件在数据目录中创建（`datadir`）。要指定替代位置，请使用`innodb_data_home_dir`选项。例如，要在名为`myibdata`的目录中创建系统表空间数据文件，请使用以下配置：

```sql
[mysqld]
innodb_data_home_dir = /myibdata/
innodb_data_file_path=ibdata1:50M:autoextend
```

在为`innodb_data_home_dir`指定值时，需要添加尾随斜杠。`InnoDB`不会创建目录，因此请确保在启动服务器之前指定的目录存在。还要确保 MySQL 服务器具有在目录中创建文件的适当访问权限。

`InnoDB`通过将`innodb_data_home_dir`的值与数据文件名文本连接来形成每个数据文件的目录路径。如果未定义`innodb_data_home_dir`，默认值为“./”，即数据目录。（当 MySQL 服务器开始执行时，它会将当前工作目录更改为数据目录。）

或者，您可以为系统表空间数据文件指定绝对路径。以下配置与前述配置等效：

```sql
[mysqld]
innodb_data_file_path=/myibdata/ibdata1:50M:autoextend
```

当您为`innodb_data_file_path`指定绝对路径时，设置不会与`innodb_data_home_dir`设置连接。系统表空间文件将在指定的绝对路径中创建。在启动服务器之前，指定的目录必须存在。

#### InnoDB 双写缓冲区文件配置

自 MySQL 8.0.20 起，双写缓冲区存储区域位于双写文件中，这提供了关于双写页存储位置的灵活性。在先前的版本中，双写缓冲区存储区域位于系统表空间中。`innodb_doublewrite_dir`变量定义了`InnoDB`在启动时创建双写文件的目录。如果未指定目录，则双写文件将在`innodb_data_home_dir`目录中创建，如果未指定，则默认为数据目录。

若要在`innodb_data_home_dir`目录之外的位置创建双写文件，请配置`innodb_doublewrite_dir`变量。例如：

```sql
innodb_doublewrite_dir=*/path/to/doublewrite_directory*
```

其他双写缓冲区变量允许定义双写文件数量、每个线程的页面数量以及双写批量大小。有关双写缓冲区配置的更多信息，请参阅第 17.6.4 节，“双写缓冲区”。

#### 重做日志配置

从 MySQL 8.0.30 开始，重做日志文件占用的磁盘空间量由`innodb_redo_log_capacity`变量控制，该变量可以在启动时或运行时设置；例如，要在选项文件中将变量设置为 8GB，请添加以下条目：

```sql
[mysqld]
innodb_redo_log_capacity = 8589934592
```

有关在运行时配置重做日志容量的信息，请参阅配置重做日志容量（MySQL 8.0.30 或更高版本）。

`innodb_redo_log_capacity`变量取代了已弃用的`innodb_log_file_size`和`innodb_log_files_in_group`变量。当定义了`innodb_redo_log_capacity`设置时，将忽略`innodb_log_file_size`和`innodb_log_files_in_group`设置；否则，这些设置用于计算`innodb_redo_log_capacity`设置（`innodb_log_files_in_group` * `innodb_log_file_size` = `innodb_redo_log_capacity`）。如果没有设置这些变量中的任何一个，`innodb_redo_log_capacity`将设置为默认值，即 104857600 字节（100MB）。最大设置为 128GB。

从 MySQL 8.0.30 开始，`InnoDB`尝试维护 32 个重做日志文件，每个文件大小为 1/32 * `innodb_redo_log_capacity`。重做日志文件位于数据目录中的`#innodb_redo`目录中，除非通过`innodb_log_group_home_dir`变量指定了不同的目录。如果定义了`innodb_log_group_home_dir`，则重做日志文件位于该目录中的`#innodb_redo`目录中。有关更多信息，请参见第 17.6.5 节，“重做日志”。

在 MySQL 8.0.30 之前，`InnoDB`默认在数据目录中创建两个 5MB 的重做日志文件，分别命名为`ib_logfile0`和`ib_logfile1`。您可以在初始化 MySQL 服务器实例时通过配置`innodb_log_files_in_group`和`innodb_log_file_size`变量来定义不同数量和大小的重做日志文件。

+   `innodb_log_files_in_group`定义了日志组中日志文件的数量。默认值和推荐值为 2。

+   `innodb_log_file_size`定义了日志组中每个日志文件的大小（以字节为单位）。组合日志文件大小（`innodb_log_file_size` * `innodb_log_files_in_group`）不能超过最大值，该值略小于 512GB。例如，一对 255GB 的日志文件接近极限但不超过。默认的日志文件大小为 48MB。通常，日志文件的组合大小应足够大，以使服务器能够平滑处理工作负载活动中的高峰和低谷，这通常意味着有足够的重做日志空间来处理超过一个小时的写入活动。较大的日志文件大小意味着在缓冲池中减少检查点刷新活动，从而减少磁盘 I/O。有关更多信息，请参见第 10.5.4 节，“优化 InnoDB 重做日志记录”。

`innodb_log_group_home_dir`定义了`InnoDB`日志文件的目录路径。您可以使用此选项将`InnoDB`重做日志文件放置在与`InnoDB`数据文件不同的物理存储位置，以避免潜在的 I/O 资源冲突；例如：

```sql
[mysqld]
innodb_log_group_home_dir = /dr3/iblogs
```

注意

`InnoDB`不会创建目录，因此在启动服务器之前确保日志目录存在。使用 Unix 或 DOS 的`mkdir`命令创建任何必要的目录。

确保 MySQL 服务器具有在日志目录中创建文件的适当访问权限。更一般地，服务器必须在需要创建文件的任何目录中具有访问权限。

#### 撤销表空间配置

默认情况下，撤销日志驻留在 MySQL 实例初始化时创建的两个撤销表空间中。

`innodb_undo_directory`变量定义了`InnoDB`创建默认撤销表空间的路径。如果该变量未定义，则默认的撤销表空间将在数据目录中创建。`innodb_undo_directory`变量不是动态的。配置它需要重新启动服务器。

撤销日志的 I/O 模式使撤销表空间成为 SSD 存储的良好选择。

有关配置额外撤销表空间的信息，请参见第 17.6.3.4 节，“撤销表空间”。

#### 全局临时表空间配置

全局临时表空间存储对用户创建的临时表所做更改的回滚段。

默认情况下，在`innodb_data_home_dir`目录中有一个名为`ibtmp1`的单个自动扩展全局临时表空间数据文件。初始文件大小略大于 12MB。

`innodb_temp_data_file_path` 选项指定了全局临时表空间数据文件的路径、文件名和文件大小。文件大小通过在大小值后附加 K、M 或 G 来指定为 KB、MB 或 GB。文件大小或组合文件大小必须略大于 12MB。

要指定全局临时表空间数据文件的替代位置，请在启动时配置 `innodb_temp_data_file_path` 选项。

#### 会话临时表空间配置

在 MySQL 8.0.15 及更早版本中，会话临时表空间存储用户创建的临时表和由优化器创建的内部临时表，当 `InnoDB` 被配置为内部临时表的磁盘存储引擎时（`internal_tmp_disk_storage_engine=InnoDB`）。从 MySQL 8.0.16 开始，`InnoDB` 总是被用作内部临时表的磁盘存储引擎。

`innodb_temp_tablespaces_dir` 变量定义了 `InnoDB` 创建会话临时表空间的位置。默认位置是数据目录中的 `#innodb_temp` 目录。

要指定会话临时表空间的替代位置，请在启动时配置 `innodb_temp_tablespaces_dir` 变量。允许使用完全限定路径或相对于数据目录的路径。

#### 页面大小配置

`innodb_page_size` 选项指定了 MySQL 实例中所有 `InnoDB` 表空间的页面大小。此值在实例创建时设置，并在此后保持不变。有效值为 64KB、32KB、16KB（默认值）、8KB 和 4KB。另外，您也可以按字节指定页面大小（65536、32768、16384、8192、4096）。

默认的 16KB 页面大小适用于各种工作负载，特别是涉及表扫描和涉及大量更新的 DML 操作的查询。对于涉及许多小写入的 OLTP 工作负载，较小的页面大小可能更有效，当单个页面包含许多行时，争用可能是一个问题。较小的页面对于通常使用小块大小的 SSD 存储设备也可能更有效。保持 `InnoDB` 页面大小接近存储设备块大小可以最小化被重写到磁盘的未更改数据量。

重要提示

`innodb_page_size` 只能在初始化数据目录时设置。有关此变量的更多信息，请参阅其描述。

#### 内存配置

MySQL 为各种缓存和缓冲区分配内存以提高数据库操作的性能。在为`InnoDB`分配内存时，始终考虑操作系统所需的内存、分配给其他应用程序的内存以及为其他 MySQL 缓冲区和缓存分配的内存。例如，如果您使用`MyISAM`表，请考虑为键缓冲区（`key_buffer_size`）分配的内存量。有关 MySQL 缓冲区和缓存的概述，请参阅第 10.12.3.1 节，“MySQL 如何使用内存”。

`InnoDB`特定的缓冲区使用以下参数进行配置：

+   `innodb_buffer_pool_size` 定义了缓冲池的大小，这是一个内存区域，用于保存`InnoDB`表、索引和其他辅助缓冲区的缓存数据。缓冲池的大小对系统性能很重要，通常建议将`innodb_buffer_pool_size` 配置为系统内存的 50%到 75%。默认的缓冲池大小为 128MB。有关更多指导，请参阅第 10.12.3.1 节，“MySQL 如何使用内存”。有关如何配置`InnoDB`缓冲池大小的信息，请参阅第 17.8.3.1 节，“配置 InnoDB 缓冲池大小”。缓冲池大小可以在启动时或动态配置。

    在具有大量内存的系统上，可以通过将缓冲池划分为多个缓冲池实例来提高并发性。缓冲池实例的数量由`innodb_buffer_pool_instances` 选项控制。默认情况下，`InnoDB`创建一个缓冲池实例。缓冲池实例的数量可以在启动时配置。有关更多信息，请参阅第 17.8.3.2 节，“配置多个缓冲池实例”。

+   `innodb_log_buffer_size` 定义了`InnoDB`用于向磁盘上的日志文件写入的缓冲区大小。默认大小为 16MB。较大的日志缓冲区使得大型事务在提交之前可以运行而不必将日志写入磁盘。如果您有更新、插入或删除许多行的事务，可以考虑增加日志缓冲区的大小以节省磁盘 I/O。`innodb_log_buffer_size` 可以在启动时配置。有关相关信息，请参阅第 10.5.4 节，“优化 InnoDB 重做日志记录”。

警告

在 32 位 GNU/Linux x86 上，如果内存使用量设置过高，`glibc`可能允许进程堆增长超过线程堆栈，导致服务器失败。如果为**mysqld**进程分配的全局和每个线程的缓冲区和缓存接近或超过 2GB，这是一个风险。

可以使用类似于以下计算 MySQL 全局和每个线程内存分配的公式来估算 MySQL 内存使用量。您可能需要修改公式以考虑您的 MySQL 版本和配置中的缓冲区和缓存。有关 MySQL 缓冲区和缓存的概述，请参见第 10.12.3.1 节，“MySQL 如何使用内存”。

```sql
innodb_buffer_pool_size
+ key_buffer_size
+ max_connections*(sort_buffer_size+read_buffer_size+binlog_cache_size)
+ max_connections*2MB
```

每个线程使用一个堆栈（通常为 2MB，但由 Oracle Corporation 提供的 MySQL 二进制文件中仅为 256KB），在最坏的情况下还会使用`sort_buffer_size + read_buffer_size`额外的内存。

在 Linux 上，如果内核启用了大页支持，`InnoDB`可以使用大页来为其缓冲池分配内存。参见第 10.12.3.3 节，“启用大页支持”。
