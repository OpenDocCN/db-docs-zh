# 17.8.10 为 InnoDB 配置优化器统计信息

> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-performance-optimizer-statistics.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-performance-optimizer-statistics.html)

17.8.10.1 配置持久性优化器统计参数

17.8.10.2 配置非持久性优化器统计参数

17.8.10.3 为 InnoDB 表估算 ANALYZE TABLE 复杂度

本节描述了如何为`InnoDB`表配置持久性和非持久性优化器统计信息。

持久性优化器统计信息在服务器重启后保留，可以实现更高的计划稳定性和更一致的查询性能。持久性优化器统计信息还提供了以下额外的控制和灵活性：

+   你可以使用`innodb_stats_auto_recalc`配置选项来控制在表发生重大更改后是否自动更新统计信息。

+   你可以在`CREATE TABLE`和`ALTER TABLE`语句中使用`STATS_PERSISTENT`、`STATS_AUTO_RECALC`和`STATS_SAMPLE_PAGES`子句来为单个表配置优化器统计信息。

+   你可以在`mysql.innodb_table_stats`和`mysql.innodb_index_stats`表中查询优化器统计数据。

+   你可以查看`mysql.innodb_table_stats`和`mysql.innodb_index_stats`表的`last_update`列，以查看统计信息上次更新的时间。

+   你可以手动修改`mysql.innodb_table_stats`和`mysql.innodb_index_stats`表，以强制执行特定的查询优化计划或测试替代计划，而不修改数据库。

持久性优化器统计功能默认启用（`innodb_stats_persistent=ON`）。

非持久性优化器统计信息在每次服务器重启和其他一些操作后被清除，并在下一次访问表时重新计算。因此，在重新计算统计信息时可能会产生不同的估计值，导致执行计划的不同选择和查询性能的变化。

本节还提供了关于估算`ANALYZE TABLE`复杂度的信息，这在尝试在准确统计信息和`ANALYZE TABLE`执行时间之间取得平衡时可能会有用。
