# 10.5.10 优化具有多个表的 InnoDB 系统

> 原文：[`dev.mysql.com/doc/refman/8.0/en/optimizing-innodb-many-tables.html`](https://dev.mysql.com/doc/refman/8.0/en/optimizing-innodb-many-tables.html)

+   如果您已配置了非持久化优化器统计信息（非默认配置），`InnoDB`在首次访问表时计算索引基数值，而不是将这些值存储在表中。在将数据分区到多个表的系统上，这一步可能需要较长时间。由于这种开销仅适用于初始表打开操作，为了“预热”表以供以后使用，立即在启动后访问它，例如通过发出类似`SELECT 1 FROM *`tbl_name`* LIMIT 1`的语句。

    优化器统计信息默认情况下会持久化到磁盘，通过`innodb_stats_persistent`配置选项启用。有关持久化优化器统计信息的信息，请参阅第 17.8.10.1 节，“配置持久化优化器统计参数”。
