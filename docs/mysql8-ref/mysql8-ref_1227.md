> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-performance-compression-oltp.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-performance-compression-oltp.html)

#### 17.9.1.6 OLTP 工作负载的压缩

传统上，`InnoDB`的压缩功能主要推荐用于只读或读最多的工作负载，例如在数据仓库配置中。随着 SSD 存储设备的兴起，这些设备速度快但相对较小且昂贵，使压缩对于`OLTP`工作负载也变得有吸引力：高流量、互动式网站可以通过使用频繁进行`INSERT`、`UPDATE`和`DELETE`操作的应用程序与压缩表来减少其存储需求和每秒的 I/O 操作（IOPS）。

这些配置选项允许您调整压缩方式，以适应特定的 MySQL 实例，重点放在写入密集型操作的性能和可伸缩性上：

+   `innodb_compression_level` 允许您调整压缩程度。较高的值可以让您在存储设备上容纳更多数据，但会增加压缩时的 CPU 开销。较低的值可以减少 CPU 开销，当存储空间不是关键问题，或者您预计数据不容易压缩时。

+   `innodb_compression_failure_threshold_pct` 指定在更新压缩表时的压缩失败的截止点。当超过此阈值时，MySQL 开始在每个新的压缩页面内留下额外的空闲空间，动态调整空闲空间的量，直到达到`innodb_compression_pad_pct_max`指定的页面大小百分比。

+   `innodb_compression_pad_pct_max` 允许您调整每个页内保留的空间量，用于记录压缩行的更改，而无需重新压缩整个页面。值越高，可以记录更多更改而无需重新压缩页面。当运行时指定的压缩操作“失败”达到指定百分比时，MySQL 为每个压缩表内的页面使用可变数量的空闲空间，需要昂贵的操作来拆分压缩页面。

+   `innodb_log_compressed_pages` 允许您禁用将重新压缩的页面的图像写入重做日志。当对压缩数据进行更改时，可能会发生重新压缩。默认情况下启用此选项，以防止在恢复过程中使用不同版本的`zlib`压缩算法时可能发生的损坏。如果您确定`zlib`版本不会更改，请禁用`innodb_log_compressed_pages`以减少修改压缩数据的工作负载生成的重做日志。

因为使用压缩数据有时需要同时在内存中保留页面的压缩和未压缩版本，所以在使用压缩与 OLTP 风格的工作负载时，准备增加`innodb_buffer_pool_size`配置选项的值。
