# 17.6.4 双写缓冲区

> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-doublewrite-buffer.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-doublewrite-buffer.html)

双写缓冲区是`InnoDB`在将页面写入其在`InnoDB`数据文件中正确位置之前，从缓冲池刷新的页面写入的存储区域。如果在页面写入过程中发生操作系统、存储子系统或意外的**mysqld**进程退出，`InnoDB`可以在崩溃恢复期间从双写缓冲区找到页面的良好副本。

尽管数据被写入两次，但双写缓冲区并不需要两倍的 I/O 开销或两倍的 I/O 操作。数据以大块顺序写入双写缓冲区，通过单个`fsync()`调用到操作系统（除非`innodb_flush_method`设置为`O_DIRECT_NO_FSYNC`）。

在 MySQL 8.0.20 之前，双写缓冲区存储区域位于`InnoDB`系统表空间中。从 MySQL 8.0.20 开始，双写缓冲区存储区域位于双写文件中。

提供以下变量用于双写缓冲区配置：

+   `innodb_doublewrite`

    `innodb_doublewrite`变量控制双写缓冲区是否启用。在大多数情况下，默认情况下启用双写缓冲区。要禁用双写缓冲区，请将`innodb_doublewrite`设置为`OFF`。例如，在执行基准测试时，如果您更关注性能而不是数据完整性，则考虑禁用双写缓冲区。

    从 MySQL 8.0.30 开始，`innodb_doublewrite`支持`DETECT_AND_RECOVER`和`DETECT_ONLY`设置。

    `DETECT_AND_RECOVER`设置与`ON`设置相同。使用此设置时，双写缓冲区完全启用，数据库页面内容被写入双写缓冲区，在恢复过程中访问以修复不完整的页面写入。

    使用`DETECT_ONLY`设置时，只有元数据被写入双写缓冲区。数据库页面内容不会被写入双写缓冲区，恢复过程也不会使用双写缓冲区来修复不完整的页面写入。这种轻量级设置仅用于检测不完整的页面写入。

    MySQL 8.0.30 及更高版本支持动态更改`innodb_doublewrite`设置，使双写缓冲区在`ON`、`DETECT_AND_RECOVER`和`DETECT_ONLY`之间切换。MySQL 不支持在启用双写缓冲区的设置和`OFF`之间进行动态更改，反之亦然。

    如果双写缓冲区位于支持原子写入的 Fusion-io 设备上，则双写缓冲区将自动禁用，并且数据文件写入将使用 Fusion-io 原子写入。但是，请注意`innodb_doublewrite`设置是全局的。当双写缓冲区被禁用时，所有数据文件都被禁用，包括那些不位于 Fusion-io 硬件上的文件。此功能仅受支持于 Fusion-io 硬件，并且仅在 Linux 上为 Fusion-io NVMFS 启用。为充分利用此功能，建议将`innodb_flush_method`设置为`O_DIRECT`。

+   `innodb_doublewrite_dir`

    `innodb_doublewrite_dir`变量（在 MySQL 8.0.20 中引入）定义了`InnoDB`创建双写文件的目录。如果未指定目录，则双写文件将在`innodb_data_home_dir`目录中创建，默认情况下为数据目录。

    自动在指定目录名称前加上井号'#'，以避免与模式名称冲突。但是，如果在目录名称中明确指定了'.'、'#'或'/'前缀，则不会在目录名称前加上井号'#'。

    理想情况下，双写目录应放置在最快的存储介质上。

+   `innodb_doublewrite_files`

    `innodb_doublewrite_files`变量定义了双写文件的数量。默认情况下，为每个缓冲池实例创建两个双写文件：一个刷新列表双写文件和一个 LRU 列表双写文件。

    刷新列表双写文件用于从缓冲池刷新列表刷新的页面。刷新列表双写文件的默认大小为`InnoDB`页面大小*双写页面字节。

    LRU 列表双写文件用于从缓冲池 LRU 列表刷新的页面。它还包含用于单页刷新的插槽。LRU 列表双写文件的默认大小为`InnoDB`页面大小*(双写页面+(512/缓冲池实例数))，其中 512 是为单页刷新保留的插槽总数。

    至少有两个双写文件。双写文件的最大数量是缓冲池实例数的两倍。（缓冲池实例数由`innodb_buffer_pool_instances`变量控制。）

    双写文件的命名格式如下：`#ib_*`page_size`*_*`file_number`*.dblwr`（或者使用`DETECT_ONLY`设置时为`.bdblwr`）。例如，对于一个`InnoDB`页大小为 16KB 且只有一个缓冲池的 MySQL 实例，会创建以下双写文件：

    ```sql
    #ib_16384_0.dblwr
    #ib_16384_1.dblwr
    ```

    `innodb_doublewrite_files`变量旨在用于高级性能调优。默认设置对大多数用户来说应该是合适的。

+   `innodb_doublewrite_pages`

    `innodb_doublewrite_pages`变量（MySQL 8.0.20 版本引入）控制了每个线程的最大双写页数。如果未指定值，`innodb_doublewrite_pages`将设置为`innodb_write_io_threads`的值。该变量旨在用于高级性能调优。默认值对大多数用户来说应该是合适的。

+   `innodb_doublewrite_batch_size`

    `innodb_doublewrite_batch_size`变量（MySQL 8.0.20 版本引入）控制了一批中要写入的双写页的数量。该变量旨在用于高级性能调优。默认值对大多数用户来说应该是合适的。

截至 MySQL 8.0.23 版本，`InnoDB`会自动加密属于加密表空间的双写文件页（参见第 17.13 节，“InnoDB 数据静态加密”）。同样，属于页压缩表空间的双写文件页会被压缩。因此，双写文件可以包含不同类型的页，包括未加密和未压缩的页，加密的页，压缩的页，以及既加密又压缩的页。
