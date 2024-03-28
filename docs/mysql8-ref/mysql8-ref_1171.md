> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-system-tablespace.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-system-tablespace.html)

#### 17.6.3.1 系统表空间

系统表空间是更改缓冲区的存储区域。如果表是在系统表空间而不是文件表或通用表空间中创建的，则还可能包含表和索引数据。在以前的 MySQL 版本中，系统表空间包含`InnoDB`数据字典。在 MySQL 8.0 中，`InnoDB`将元数据存储在 MySQL 数据字典中。请参见第十六章 *MySQL 数据字典*。在以前的 MySQL 版本中，系统表空间还包含双写缓冲区存储区域。从 MySQL 8.0.20 开始，此存储区域位于单独的双写文件中。请参见第 17.6.4 节，“双写缓冲区”。

系统表空间可以有一个或多个数据文件。默认情况下，在数据目录中创建一个名为`ibdata1`的系统表空间数据文件。系统表空间数据文件的大小和数量由`innodb_data_file_path`启动选项定义。有关配置信息，请参见系统表空间数据文件配置。

有关系统表空间的其他信息在本节的以下主题下提供：

+   调整系统表空间大小

+   使用原始磁盘分区作为系统表空间

##### 调整系统表空间大小

本节描述了如何增加或减少系统表空间的大小。

###### 增加系统表空间大小

增加系统表空间大小的最简单方法是将其配置为自动扩展。为此，请在`innodb_data_file_path`设置中为最后一个数据文件指定`autoextend`属性，并重新启动服务器。例如：

```sql
innodb_data_file_path=ibdata1:10M:autoextend
```

当指定`autoextend`属性时，数据文件会根据需要自动以 8MB 递增的大小增加。`innodb_autoextend_increment`变量控制增量大小。

您还可以通过添加另一个数据文件来增加系统表空间大小。要这样做：

1.  停止 MySQL 服务器。

1.  如果在 `innodb_data_file_path` 设置中的最后一个数据文件定义了 `autoextend` 属性，请将其删除，并修改大小属性以反映当前数据文件大小。要确定要指定的适当数据文件大小，请检查文件系统的文件大小，并将该值向下舍入到最接近的 MB 值，其中 1 MB 等于 1024 x 1024 字节。

1.  将一个新的数据文件追加到 `innodb_data_file_path` 设置中，可选择指定 `autoextend` 属性。`autoextend` 属性只能针对 `innodb_data_file_path` 设置中的最后一个数据文件指定。

1.  启动 MySQL 服务器。

例如，这个表空间有一个自动扩展的数据文件：

```sql
innodb_data_home_dir =
innodb_data_file_path = /ibdata/ibdata1:10M:autoextend
```

假设随着时间的推移，数据文件已经增长到 988MB。这是在修改大小属性以反映当前数据文件大小，并指定一个新的 50MB 自动扩展数据文件后的 `innodb_data_file_path` 设置：

```sql
innodb_data_home_dir =
innodb_data_file_path = /ibdata/ibdata1:988M;/disk2/ibdata2:50M:autoextend
```

当添加新的数据文件时，请不要指定现有的文件名。`InnoDB` 在启动服务器时会创建并初始化新的数据文件。

注意

不能通过更改其大小属性来增加现有系统表空间数据文件的大小。例如，将 `innodb_data_file_path` 设置从 `ibdata1:10M:autoextend` 更改为 `ibdata1:12M:autoextend` 在启动服务器时会产生以下错误：

```sql
[ERROR] [MY-012263] [InnoDB] The Auto-extending innodb_system
data file './ibdata1' is of a different size 640 pages (rounded down to MB) than
specified in the .cnf file: initial 768 pages, max 0 (relevant if non-zero) pages!
```

错误表明现有数据文件大小（以 `InnoDB` 页表示）与配置文件中指定的数据文件大小不同。如果遇到此错误，请恢复先前的 `innodb_data_file_path` 设置，并参考系统表空间调整大小的说明。

###### 减小 InnoDB 系统表空间的大小

不支持减小现有系统表空间的大小。实现较小系统表空间的唯一选项是从备份中恢复数据到一个新的 MySQL 实例，该实例创建时具有所需的系统表空间大小配置。

有关创建备份的信息，请参见 第 17.18.1 节，“InnoDB 备份”。

有关为新系统表空间配置数据文件的信息，请参见 系统表空间数据文件配置。

为避免大型系统表空间，考虑使用每表表空间或通用表空间存储您的数据。每表表空间是默认的表空间类型，在创建`InnoDB`表时隐式使用。与系统表空间不同，每表表空间在被截断或删除时将磁盘空间返回给操作系统。有关更多信息，请参见第 17.6.3.2 节，“每表表空间”。通用表空间是多表表空间，也可以用作系统表空间的替代方案。请参见第 17.6.3.3 节，“通用表空间”。

##### 使用原始磁盘分区作为系统表空间

原始磁盘分区可以用作系统表空间数据文件。这种技术可以在 Windows 和一些 Linux 和 Unix 系统上进行非缓冲 I/O，而无需文件系统开销。在您的系统上执行有和没有原始分区的测试，以验证它们是否提高了性能。

使用原始磁盘分区时，请确保运行 MySQL 服务器的用户 ID 对该分区具有读写权限。例如，如果将服务器作为`mysql`用户运行，则分区必须可读可写。如果使用`--memlock`选项运行服务器，则服务器必须以`root`身份运行，因此分区必须可读可写。

下面描述的过程涉及选项文件的修改。有关更多信息，请参见第 6.2.2.2 节，“使用选项文件”。

###### 在 Linux 和 Unix 系统上分配原始磁盘分区

1.  创建新数据文件时，在`innodb_data_file_path`选项的数据文件大小后立即指定关键字`newraw`。分区的大小必须至少与您指定的大小相同。请注意，在`InnoDB`中，1MB 是 1024 × 1024 字节，而在磁盘规范中，1MB 通常表示 1,000,000 字节。

    ```sql
    [mysqld]
    innodb_data_home_dir=
    innodb_data_file_path=/dev/hdd1:3Gnewraw;/dev/hdd2:2Gnewraw
    ```

1.  重新启动服务器。`InnoDB`注意到`newraw`关键字并初始化新分区。但是，不要创建或更改任何`InnoDB`表。否则，下次重新启动服务器时，`InnoDB`会重新初始化分区，您的更改将丢失。（作为安全措施，当指定任何带有`newraw`的分区时，`InnoDB`会阻止用户修改数据。）

1.  在`InnoDB`初始化新分区后，停止服务器，将数据文件规范中的`newraw`更改为`raw`：

    ```sql
    [mysqld]
    innodb_data_home_dir=
    innodb_data_file_path=/dev/hdd1:3Graw;/dev/hdd2:2Graw
    ```

1.  重新启动服务器。`InnoDB`现在允许进行更改。

###### 在 Windows 上分配原始磁盘分区

在 Windows 系统上，适用于 Linux 和 Unix 系统的相同步骤和相关指南适用，只是在 Windows 上`innodb_data_file_path`设置略有不同。

1.  在创建新数据文件时，在`innodb_data_file_path`选项后立即指定关键字`newraw`：

    ```sql
    [mysqld]
    innodb_data_home_dir=
    innodb_data_file_path=//./D::10Gnewraw
    ```

    `//./`对应于 Windows 访问物理驱动器的语法`\\.\`。在上面的示例中，`D:`是分区的驱动器号。

1.  重新启动服务器。`InnoDB`注意到`newraw`关键字并初始化新分区。

1.  在`InnoDB`初始化新分区后，停止服务器，将数据文件规范中的`newraw`更改为`raw`：

    ```sql
    [mysqld]
    innodb_data_home_dir=
    innodb_data_file_path=//./D::10Graw
    ```

1.  重新启动服务器。`InnoDB`现在允许进行更改。
