# 10.3.2 主键优化

> 原文：[`dev.mysql.com/doc/refman/8.0/en/primary-key-optimization.html`](https://dev.mysql.com/doc/refman/8.0/en/primary-key-optimization.html)

表的主键代表您在最关键的查询中使用的列或列集。它有一个关联的索引，用于快速查询性能。查询性能受`NOT NULL`优化的益处，因为它不能包含任何`NULL`值。使用`InnoDB`存储引擎，表数据被物理组织以便根据主键列或列进行超快速查找和排序。

如果您的表很大且重要，但没有明显的列或列集可用作主键，您可以创建一个带有自增值的单独列用作主键。这些唯一的 ID 可以在使用外键连接表时作为指向其他表中相应行的指针。
