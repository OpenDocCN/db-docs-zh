> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-statistics-estimation.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-statistics-estimation.html)

#### 17.8.10.2 配置非持久性优化器统计参数

本节描述了如何配置非持久性优化器统计。当`innodb_stats_persistent=OFF`或当使用`STATS_PERSISTENT=0`创建或更改单个表时，优化器统计不会持久保存到磁盘。相反，统计信息存储在内存中，在服务器关闭时丢失。统计信息也会定期由某些操作和在某些条件下更新。

优化器统计默认持久保存到磁盘，由`innodb_stats_persistent`配置选项启用。有关持久性优化器统计的信息，请参见第 17.8.10.1 节，“配置持久性优化器统计参数”。

##### 优化器统计更新

当非持久性优化器统计更新时：

+   运行`ANALYZE TABLE`。

+   运行`SHOW TABLE STATUS`，`SHOW INDEX`，或使用启用`innodb_stats_on_metadata`选项查询信息模式`TABLES`或`STATISTICS`表。

    `innodb_stats_on_metadata`的默认设置为`OFF`。启用`innodb_stats_on_metadata`可能会降低具有大量表或索引的模式的访问速度，并降低涉及`InnoDB`表的查询的执行计划的稳定性。通过使用`SET`语句在全局配置`innodb_stats_on_metadata`。

    ```sql
    SET GLOBAL innodb_stats_on_metadata=ON
    ```

    注意

    当优化器统计配置为非持久性时，`innodb_stats_on_metadata`才适用（当禁用`innodb_stats_persistent`时）。

+   使用启用了`--auto-rehash`选项的**mysql**客户端，这是默认设置。`auto-rehash`选项会导致所有`InnoDB`表被打开，并且打开表操作会导致统计信息被重新计算。

    为了提高**mysql**客户端的启动时间和更新统计信息，您可以使用`--disable-auto-rehash`选项关闭`auto-rehash`。`auto-rehash`功能使交互用户可以自动完成数据库、表和列名的名称补全。

+   首先打开一个表。

+   `InnoDB`检测到自上次更新统计信息以来表的 1/16 已被修改。

##### 配置采样页面数

MySQL 查询优化器使用关于键分布的估计统计信息来选择执行计划的索引，基于索引的相对选择性。当`InnoDB`更新优化器统计信息时，它会从表的每个索引中随机采样页面，以估算索引的基数。（这种技术被称为随机潜水。）

为了让您控制统计信息估计的质量（从而为查询优化器提供更好的信息），您可以使用参数`innodb_stats_transient_sample_pages`更改采样页面数。默认的采样页面数为 8，这可能不足以产生准确的估算，导致查询优化器做出错误的索引选择。这种技术对于大表和用于连接的表尤为重要。对于这些表进行不必要的全表扫描可能会成为一个重大的性能问题。请参阅 Section 10.2.1.23, “避免全表扫描”以获取调整此类查询的提示。`innodb_stats_transient_sample_pages`是一个可以在运行时设置的全局参数。

当`innodb_stats_persistent=0`时，`innodb_stats_transient_sample_pages`的值会影响所有`InnoDB`表和索引的索引采样。在更改索引采样大小时，请注意以下可能产生重大影响的情况：

+   像 1 或 2 这样的小值可能导致基数的估计不准确。

+   增加`innodb_stats_transient_sample_pages`的值可能需要更多的磁盘读取。比 8 大得多的值（比如 100），可能会导致打开表或执行`SHOW TABLE STATUS`的时间显著减慢。

+   优化器可能会根据不同的索引选择性估计选择非常不同的查询计划。

无论对于系统来说，`innodb_stats_transient_sample_pages`的哪个值效果最好，都要设置该选项并将其保持在该值。选择一个能够为数据库中所有表提供合理准确估计而又不需要过多 I/O 的值。因为统计数据会在执行`ANALYZE TABLE`之外的各种时间自动重新计算，所以增加索引样本大小，运行`ANALYZE TABLE`，然后再减小样本大小是没有意义的。

较小的表通常需要比较少的索引样本。如果您的数据库有许多大表，考虑使用比大多数较小表更高的值`innodb_stats_transient_sample_pages`。
