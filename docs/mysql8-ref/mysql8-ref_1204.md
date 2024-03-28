> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-performance-read_ahead.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-performance-read_ahead.html)

#### 17.8.3.4 配置 InnoDB 缓冲池预取（预读取）

一个预读取请求是一个异步的 I/O 请求，用于预取缓冲池中多个页面，以期望未来需要这些页面。这些请求一次带入一个 extent 中的所有页面。`InnoDB`使用两种预读取算法来提高 I/O 性能：

**线性**预读取是一种技术，根据缓冲池中按顺序访问的页面来预测哪些页面可能很快会被需要。您可以通过调整配置参数`innodb_read_ahead_threshold`来控制`InnoDB`何时执行预读取操作，该参数表示触发异步读取请求所需的连续页面访问次数。在添加此参数之前，`InnoDB`只会在读取当前 extent 的最后一页时计算是否发出整个下一个 extent 的异步预取请求。

配置参数`innodb_read_ahead_threshold`控制`InnoDB`在检测连续页面访问模式时的敏感度。如果从一个 extent 中连续读取的页面数量大于或等于`innodb_read_ahead_threshold`，`InnoDB`会启动整个后续 extent 的异步预读取操作。`innodb_read_ahead_threshold`的值可以设置为 0-64 之间的任何值。默认值为 56。值越高，访问模式检查越严格。例如，如果将值设置为 48，`InnoDB`仅在当前 extent 中连续访问了 48 页时才触发线性预读取请求。如果值为 8，即使在 extent 中连续访问了仅有 8 页，`InnoDB`也会触发异步预读取。您可以在 MySQL 的配置文件中设置此参数的值，或使用`SET GLOBAL`语句动态更改该值，这需要足够的权限来设置全局系统变量。参见 Section 7.1.9.1, “System Variable Privileges”。

**随机**预读是一种技术，根据缓冲池中已经存在的页面，预测哪些页面可能很快就会被需要，而不考虑这些页面的读取顺序。如果在缓冲池中找到来自同一范围的连续 13 个页面，`InnoDB`会异步发出请求，预取该范围的其余页面。要启用此功能，请将配置变量`innodb_random_read_ahead`设置为`ON`。

`SHOW ENGINE INNODB STATUS` 命令显示统计信息，帮助您评估预读算法的有效性。统计信息包括以下全局状态变量的计数器信息：

+   `Innodb_buffer_pool_read_ahead`

+   `Innodb_buffer_pool_read_ahead_evicted`

+   `Innodb_buffer_pool_read_ahead_rnd`

当微调`innodb_random_read_ahead`设置时，这些信息可能会有用。

有关 I/O 性能的更多信息，请参阅第 10.5.8 节，“优化 InnoDB 磁盘 I/O” 和 第 10.12.1 节，“优化磁盘 I/O”。
