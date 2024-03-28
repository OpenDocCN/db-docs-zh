# 10.5.4 优化 InnoDB 重做日志

> 原文：[`dev.mysql.com/doc/refman/8.0/en/optimizing-innodb-logging.html`](https://dev.mysql.com/doc/refman/8.0/en/optimizing-innodb-logging.html)

考虑以下优化重做日志记录的准则：

+   增加重做日志文件的大小。当 `InnoDB` 写满重做日志文件时，必须将缓冲池的修改内容写入磁盘中的一个检查点。较小的重做日志文件会导致许多不必要的磁盘写入。

    从 MySQL 8.0.30 开始，重做日志文件大小由 `innodb_redo_log_capacity` 设置确定。`InnoDB` 尝试维护相同大小的 32 个重做日志文件，每个文件大小等于 1/32 * `innodb_redo_log_capacity`。因此，更改 `innodb_redo_log_capacity` 设置会改变重做日志文件的大小。

    在 MySQL 8.0.30 之前，重做日志文件的大小和数量是使用 `innodb_log_file_size` 和 `innodb_log_files_in_group` 变量进行配置的。

    有关修改重做日志文件配置的信息，请参见 第 17.6.5 节，“重做日志”。

+   考虑增加 日志缓冲区 的大小。较大的日志缓冲区使大型事务能够在提交之前运行而无需将日志写入磁盘。因此，如果您有更新、插入或删除许多行的事务，使日志缓冲区更大可以节省磁盘 I/O。日志缓冲区大小通过 `innodb_log_buffer_size` 配置选项进行配置，在 MySQL 8.0 中可以动态配置。

+   配置 `innodb_log_write_ahead_size` 配置选项以避免“读写”。此选项定义了重做日志的预写块大小。将 `innodb_log_write_ahead_size` 设置为匹配操作系统或文件系统缓存块大小。读写发生在由于重做日志的预写块大小与操作系统或文件系统缓存块大小不匹配而导致重做日志块未完全缓存到操作系统或文件系统时。

    `innodb_log_write_ahead_size` 的有效值是 `InnoDB` 日志文件块大小的倍数（2^n）。最小值为 `InnoDB` 日志文件块大小（512）。当指定最小值时，不会发生预写。最大值等于 `innodb_page_size` 的值。如果指定的值大于 `innodb_page_size` 的值，则 `innodb_log_write_ahead_size` 设置将被截断为 `innodb_page_size` 的值。

    如果将 `innodb_log_write_ahead_size` 的值设置得太低，与操作系统或文件系统缓存块大小相比，会导致读写。将值设置得太高可能会对日志文件写入的 `fsync` 性能产生轻微影响，因为会一次写入多个块。

+   MySQL 8.0.11 引入了专用的日志写入线程，用于将重做日志记录从日志缓冲区写入系统缓冲区并将系统缓冲区刷新到重做日志文件。以前，各个用户线程负责这些任务。从 MySQL 8.0.22 开始，您可以使用 `innodb_log_writer_threads` 变量启用或禁用日志写入线程。专用的日志写入线程可以提高高并发系统的性能，但对于低并发系统，禁用专用的日志写入线程可以提供更好的性能。

+   通过用户线程优化等待刷新重做的自旋延迟的使用。自旋延迟有助于减少延迟。在低并发期间，减少延迟可能不是首要任务，避免在这些时期使用自旋延迟可能会降低能耗。在高并发期间，您可能希望避免在自旋延迟上消耗处理能力，以便用于其他工作。以下系统变量允许设置定义自旋延迟使用边界的高水位线和低水位线值。

    +   `innodb_log_wait_for_flush_spin_hwm`：定义了超过该值的最大平均日志刷新时间，用户线程在等待刷新的重做时不再自旋。默认值为 400 微秒。

    +   `innodb_log_spin_cpu_abs_lwm`: 定义了在刷新重做时等待时，用户线程不再自旋的最低 CPU 使用量。该值表示为 CPU 核使用量的总和。例如，默认值为 80，即一个 CPU 核的 80%。在具有多核处理器的系统上，值为 150 表示一个 CPU 核的 100% 使用量加上第二个 CPU 核的 50% 使用量。

    +   `innodb_log_spin_cpu_pct_hwm`: 定义了在刷新重做时等待时，用户线程不再自旋的最大 CPU 使用量。该值表示为所有 CPU 核的总处理能力的百分比。默认值为 50%。例如，在具有四个 CPU 核的服务器上，两个 CPU 核的 100% 使用量是所有 CPU 处理能力的 50%。

        `innodb_log_spin_cpu_pct_hwm` 配置选项遵守处理器亲和性。例如，如果服务器有 48 个核心，但 **mysqld** 进程只固定在四个 CPU 核心上，其他 44 个 CPU 核心将被忽略。
