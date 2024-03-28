# 17.8.7 配置 InnoDB I/O 容量

> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-configuring-io-capacity.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-configuring-io-capacity.html)

`InnoDB`主线程和其他线程在后台执行各种任务，其中大部分与 I/O 相关，例如从缓冲池中刷新脏页并将更改从更改缓冲区写入适当的辅助索引。`InnoDB`试图以不会对服务器正常工作产生不利影响的方式执行这些任务。它试图估计可用的 I/O 带宽，并调整其活动以利用可用容量。

`innodb_io_capacity`变量定义了`InnoDB`可用的整体 I/O 容量。它应该设置为系统每秒可以执行的 I/O 操作数（IOPS）的大致数量。当设置了`innodb_io_capacity`时，`InnoDB`根据设置的值估计可用于后台任务的 I/O 带宽。

你可以将`innodb_io_capacity`设置为 100 或更高的值。默认值为`200`。通常，约 100 左右的值适用于消费级存储设备，例如转速为 7200 转/分的硬盘。速度更快的硬盘、RAID 配置和固态硬盘（SSD）受益于更高的值。

理想情况下，保持设置尽可能低，但不要太低以至于后台活动落后。如果值设置过高，数据会从缓冲池和更改缓冲区中被过快地移除，使得缓存无法提供显著的好处。对于能够处理更高 I/O 速率的繁忙系统，你可以设置更高的值来帮助服务器处理与高频率行更改相关的后台维护工作。通常，你可以根据用于`InnoDB` I/O 的驱动器数量的函数增加该值。例如，你可以在使用多个磁盘或 SSD 的系统上增加该值。

默认设置的 200 通常对于低端 SSD 足够。对于高端总线连接的 SSD，考虑更高的设置，例如 1000。对于使用单独的 5400 转/分或 7200 转/分驱动器的系统，你可能会将值降低到 100，这代表了每秒 I/O 操作（IOPS）的估计比例，适用于可以执行约 100 IOPS 的旧一代磁盘驱动器。

虽然你可以指定一个很高的值，比如一百万，但实际上这样大的值很少有好处。通常，建议不要将值设定为 20000 以上，除非你确定较低的值对你的工作负载不足。

在调整`innodb_io_capacity`时考虑写入工作负载。具有大量写入工作负载的系统可能会受益于更高的设置。对于写入工作负载较小的系统，较低的设置可能已经足够。

`innodb_io_capacity`设置不是每个缓冲池实例的设置。可用的 I/O 容量在刷新活动中均匀分配给缓冲池实例。

您可以在 MySQL 选项文件（`my.cnf`或`my.ini`）中设置`innodb_io_capacity`的值，或者使用`SET GLOBAL`语句在运行时修改它，这需要足够的权限来设置全局系统变量。请参阅第 7.1.9.1 节，“系统变量权限”。

#### 在检查点忽略 I/O 容量

`innodb_flush_sync`变量默认启用，导致在发生 I/O 活动突发时，会忽略`innodb_io_capacity`设置，这种活动发生在检查点。要遵守`innodb_io_capacity`设置定义的 I/O 速率，请禁用`innodb_flush_sync`。

您可以在 MySQL 选项文件（`my.cnf`或`my.ini`）中设置`innodb_flush_sync`的值，或者使用`SET GLOBAL`语句在运行时修改它，这需要足够的权限来设置全局系统变量。请参阅第 7.1.9.1 节，“系统变量权限”。

#### 配置 I/O 容量最大值

如果刷新活动落后，`InnoDB`可以以比`innodb_io_capacity`变量定义的更高的 IOPS（每秒 I/O 操作数）更积极地刷新。`innodb_io_capacity_max`变量定义了`InnoDB`在这种情况下执行的后台任务的最大 IOPS 数量。

如果在启动时指定了`innodb_io_capacity`设置，但未为`innodb_io_capacity_max`指定值，则`innodb_io_capacity_max`默认为`innodb_io_capacity`值的两倍或 2000，以较大值为准。

在配置`innodb_io_capacity_max`时，通常将`innodb_io_capacity`的两倍作为起点是一个不错的选择。默认值为 2000 适用于使用 SSD 或多个常规磁盘驱动器的工作负载。对于不使用 SSD 或多个磁盘驱动器的工作负载，设置为 2000 可能过高，可能会导致过多的刷新。对于单个常规磁盘驱动器，建议设置在 200 到 400 之间。对于高端、总线连接的 SSD，考虑设置更高的值，如 2500。与`innodb_io_capacity`设置一样，保持设置尽可能低，但不要太低以至于`InnoDB`无法足够扩展 IOPS 的速率超过`innodb_io_capacity`设置。

在调整`innodb_io_capacity_max`时，请考虑写入工作负载。具有大量写入工作负载的系统可能会受益于更高的设置。对于具有较小写入工作负载的系统，较低的设置可能足够。

`innodb_io_capacity_max`不能设置为低于`innodb_io_capacity`值。

使用`SET`语句将`innodb_io_capacity_max`设置为`DEFAULT`（`SET GLOBAL innodb_io_capacity_max=DEFAULT`）将`innodb_io_capacity_max`设置为最大值。

`innodb_io_capacity_max` 限制适用于所有缓冲池实例。这不是每个缓冲池实例的设置。
