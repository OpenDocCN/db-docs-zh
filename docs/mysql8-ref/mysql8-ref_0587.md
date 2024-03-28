# 10.5.8 优化 InnoDB 磁盘 I/O

> 原文：[`dev.mysql.com/doc/refman/8.0/en/optimizing-innodb-diskio.html`](https://dev.mysql.com/doc/refman/8.0/en/optimizing-innodb-diskio.html)

如果您遵循数据库设计的最佳实践和 SQL 操作的调优技术，但由于大量磁盘 I/O 活动而导致数据库仍然缓慢，请考虑这些磁盘 I/O 优化。如果 Unix 的 `top` 工具或 Windows 任务管理器显示您的工作负载的 CPU 使用率百分比低于 70%，那么您的工作负载可能受到磁盘限制。

+   增加缓冲池大小

    当表数据被缓存在 `InnoDB` 缓冲池中时，可以通过查询重复访问而无需进行任何磁盘 I/O。使用 `innodb_buffer_pool_size` 选项指定缓冲池的大小。这个内存区域非常重要，通常建议将 `innodb_buffer_pool_size` 配置为系统内存的 50 到 75%。有关更多信息，请参见 Section 10.12.3.1, “How MySQL Uses Memory”。

+   调整刷新方法

    在某些 GNU/Linux 和 Unix 版本中，使用 Unix 的 `fsync()` 调用（`InnoDB` 默认使用）和类似方法将文件刷新到磁盘的速度令人惊讶地慢。如果数据库写入性能成为问题，可以通过将 `innodb_flush_method` 参数设置为 `O_DSYNC` 进行基准测试。

+   配置操作系统刷新的阈值

    默认情况下，当 `InnoDB` 创建一个新的数据文件，例如新的日志文件或表空间文件时，该文件会在刷新到磁盘之前完全写入操作系统缓存，这可能会导致大量的磁盘写入活动一次性发生。为了强制操作系统缓存中的数据进行较小的周期性刷新，可以使用 `innodb_fsync_threshold` 变量定义一个阈值值，以字节为单位。当达到字节阈值时，操作系统缓存的内容将被刷新到磁盘。默认值为 0，强制默认行为，即仅在文件完全写入缓存后才将数据刷新到磁盘。

    指定阈值以强制进行较小的周期性刷新，在多个 MySQL 实例使用相同存储设备的情况下可能会有益。例如，创建一个新的 MySQL 实例及其关联的数据文件可能会导致大量的磁盘写入活动，影响使用相同存储设备的其他 MySQL 实例的性能。配置阈值有助于避免这种写入活动的激增。

+   使用 fdatasync() 替代 fsync()

    在支持 `fdatasync()` 系统调用的平台上，MySQL 8.0.26 中引入的 `innodb_use_fdatasync` 变量允许使用 `fdatasync()` 而不是 `fsync()` 进行操作系统刷新。`fdatasync()` 系统调用不会刷新文件元数据，除非需要进行后续数据检索，从而提供潜在的性能优势。

    一些 `innodb_flush_method` 设置的子集，如 `fsync`、`O_DSYNC` 和 `O_DIRECT` 使用 `fsync()` 系统调用。当使用这些设置时，`innodb_use_fdatasync` 变量适用。

+   在 Linux 上使用本机 AIO 时，请使用 noop 或 deadline I/O 调度程序

    `InnoDB` 在 Linux 上使用异步 I/O 子系统（本机 AIO）来执行数据文件页的预读和写入请求。此行为由 `innodb_use_native_aio` 配置选项控制，默认情况下已启用。使用本机 AIO 时，I/O 调度程序的类型对 I/O 性能有更大的影响。通常建议使用 noop 和 deadline I/O 调度程序。进行基准测试，以确定哪种 I/O 调度程序为您的工作负载和环境提供最佳结果。有关更多信息，请参见 第 17.8.6 节，“在 Linux 上使用异步 I/O”。

+   在 Solaris 10 的 x86_64 架构上使用直接 I/O

    在 Solaris 10 的 x86_64 架构（AMD Opteron）上使用 `InnoDB` 存储引擎时，为避免降低 `InnoDB` 性能，请对 `InnoDB` 相关文件使用直接 I/O。要为用于存储 `InnoDB` 相关文件的整个 UFS 文件系统使用直接 I/O，请使用 `forcedirectio` 选项挂载它；参见 `mount_ufs(1M)`。（Solaris 10/x86_64 的默认设置 *不* 使用此选项。）要仅对 `InnoDB` 文件操作应用直接 I/O 而不是整个文件系统，请设置 `innodb_flush_method = O_DIRECT`。使用此设置，`InnoDB` 调用 `directio()` 而不是 `fcntl()` 进行数据文件的 I/O（不适用于日志文件的 I/O）。

+   在 Solaris 2.6 或更高版本上为数据和日志文件使用原始存储

    在 Solaris 2.6 及更高版本和任何平台（sparc/x86/x64/amd64）上使用具有大 `innodb_buffer_pool_size` 值的 `InnoDB` 存储引擎时，使用如前所述的 `forcedirectio` 挂载选项在原始设备或单独的直接 I/O UFS 文件系统上进行 `InnoDB` 数据文件和日志文件的基准测试。如果要对日志文件使用直接 I/O，则需要使用挂载选项而不是设置 `innodb_flush_method`。使用 Veritas 文件系统 VxFS 的用户应使用 `convosync=direct` 挂载选项。

    不要将其他 MySQL 数据文件，如`MyISAM`表的文件，放在直接 I/O 文件系统上。可执行文件或库*不得*放在直接 I/O 文件系统上。

+   使用额外的存储设备

    可以使用额外的存储设备设置 RAID 配置。有关相关信息，请参见第 10.12.1 节，“优化磁��I/O”。

    或者，`InnoDB`表空间数据文件和日志文件可以放在不同的物理磁盘上。有关更多信息，请参考以下章节：

    +   第 17.8.1 节，“InnoDB 启动配置”

    +   第 17.6.1.2 节，“外部创建表”

    +   创建通用表空间

    +   第 17.6.1.4 节，“移动或复制 InnoDB 表”

+   考虑非旋转存储

    非旋转存储通常为随机 I/O 操作提供更好的性能；而旋转存储适用于顺序 I/O 操作。在将数据和日志文件分布在旋转和非旋转存储设备上时，请考虑每个文件上主要执行的 I/O 操作类型。

    随机 I/O 导向的文件通常包括 file-per-table 和 general tablespace 数据文件，undo tablespace 文件，以及 temporary tablespace 文件。顺序 I/O 导向的文件包括`InnoDB`system tablespace 文件（由于 MySQL 8.0.20 之前的 Doublewrite buffering 和 change buffering），MySQL 8.0.20 引入的 doublewrite 文件，以及诸如 binary log 文件和 redo log 文件等日志文件。

    使用非旋转存储时，请查看以下配置选项的设置：

    +   `innodb_checksum_algorithm`

        `crc32`选项使用更快的校验算法，建议用于快速存储系统。

    +   `innodb_flush_neighbors`

        为旋转存储设备优化 I/O。对于非旋转存储或旋转和非旋转存储的混合情况，请禁用它。默认情况下禁用。

    +   `innodb_idle_flush_pct`

        允许在空闲时期限制页面刷新，这有助于延长非旋转存储设备的寿命。在 MySQL 8.0.18 中引入。

    +   `innodb_io_capacity`

        默认设置为 200 通常足以满足低端非旋转存储设备的需求。对于高端、总线连接设备，请考虑更高的设置，例如 1000。

    +   `innodb_io_capacity_max`

        默认值为 2000 适用于使用非旋转存储的工作负载。对于高端、总线连接的非旋转存储设备，请考虑更高的设置，例如 2500。

    +   `innodb_log_compressed_pages`

        如果重做日志存储在非旋转存储上，请考虑禁用此选项以减少日志记录。参见禁用压缩页面的日志记录。

    +   `innodb_log_file_size`（在 MySQL 8.0.30 中已弃用）

        如果重做日志存储在非旋转存储上，请配置此选项以最大化缓存和写入组合。

    +   `innodb_redo_log_capacity`

        如果重做日志存储在非旋转存储上，请配置此选项以最大化缓存和写入组合。

    +   `innodb_page_size`

        考虑使用与磁盘内部扇区大小匹配的页面大小。早期的 SSD 设备通常具有 4KB 扇区大小。一些新设备具有 16KB 扇区大小。默认的`InnoDB`页面大小为 16KB。保持页面大小接近存储设备块大小可以最大程度地减少被重写到磁盘的未更改数据量。

    +   `binlog_row_image`

        如果二进制日志存储在非旋转存储上，并且所有表都有主键，请考虑将此选项设置为`minimal`以减少日志记录。

    确保您的操作系统已启用 TRIM 支持。通常情况下，默认情况下已启用。

+   增加 I/O 容量以避免积压

    如果由于`InnoDB`检查点操作而导致吞吐量周期性下降，请考虑增加`innodb_io_capacity`配置选项的值。较高的值会导致更频繁的刷新，避免积压的工作导致吞吐量下降。

+   如果刷新没有落后，请降低 I/O 容量

    如果系统没有落后于`InnoDB` 刷新操作，请考虑降低`innodb_io_capacity`配置选项的值。通常情况下，您应该尽可能保持此选项值较低，但不要太低以至于导致吞吐量周期性下降，如前述项目所述。在您可以降低选项值的典型情况下，您可能会在`SHOW ENGINE INNODB STATUS`的输出中看到类似以下组合：

    +   历史列表长度较低，低于几千。

    +   插入缓冲区合并接近插入的行数。

    +   缓冲池中修改的页面始终远低于`innodb_max_dirty_pages_pct`的缓冲池。（在服务器不进行大量插入操作时进行测量；在进行大量插入操作时，修改页面的百分比会显著上升是正常的。）

    +   `日志序列号 - 最后检查点`小于总`InnoDB` 日志文件大小的 7/8 或理想情况下小于 6/8。

+   将系统表空间文件存储在 Fusion-io 设备上

    您可以通过将包含双写存储区域的文件存储在支持原子写入的 Fusion-io 设备上，利用双写缓冲区相关的 I/O 优化。（在 MySQL 8.0.20 之前，双写缓冲区存储区域位于系统表空间数据文件中。从 MySQL 8.0.20 开始，存储区域位于双写文件中。请参阅 Section 17.6.4, “Doublewrite Buffer”。）当双写存储区域文件放置在支持原子写入的 Fusion-io 设备上时，双写缓冲区会自动禁用，并且 Fusion-io 原子写入会用于所有数据文件。此功能仅在 Fusion-io 硬件上受支持，并且仅在 Linux 上为 Fusion-io NVMFS 启用。为充分利用此功能，建议将`innodb_flush_method`设置为`O_DIRECT`。

    注意

    由于双写缓冲区设置是全局的，因此对于不位于 Fusion-io 硬件上的数据文件，双写缓冲区也被禁用。

+   禁用压缩页面的日志记录

    当使用`InnoDB`表的压缩功能时，重新压缩的页的图像在对压缩数据进行更改时会被写入重做日志。这种行为由`innodb_log_compressed_pages`控制，默认情况下启用，以防止在恢复过程中使用不同版本的`zlib`压缩算法导致的损坏。如果您确定`zlib`版本不会更改，请禁用`innodb_log_compressed_pages`以减少修改压缩数据的工作负载生成的重做日志。
