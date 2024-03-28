# 10.9 控制查询优化器

> 原文：[`dev.mysql.com/doc/refman/8.0/en/controlling-optimizer.html`](https://dev.mysql.com/doc/refman/8.0/en/controlling-optimizer.html)

10.9.1 控制查询计划评估

10.9.2 可切换的优化

10.9.3 优化器提示

10.9.4 索引提示

10.9.5 优化器成本模型

10.9.6 优化器统计信息

MySQL 通过影响查询计划如何评估的系统变量、可切换的优化、优化器和索引提示以及优化器成本模型提供优化器控制。

服务器在`column_statistics`数据字典表中维护关于列值的直方图统计信息（参见第 10.9.6 节，“优化器统计信息”）。像其他数据字典表一样，用户无法直接访问此表。相反，您可以通过查询`INFORMATION_SCHEMA.COLUMN_STATISTICS`来获取直方图信息，该表实现为数据字典表上的视图。您还可以使用`ANALYZE TABLE`语句执行直方图管理。
