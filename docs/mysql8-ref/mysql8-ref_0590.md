# 10.6 MyISAM 表优化

> 原文：[`dev.mysql.com/doc/refman/8.0/en/optimizing-myisam.html`](https://dev.mysql.com/doc/refman/8.0/en/optimizing-myisam.html)

10.6.1 优化 MyISAM 查询

10.6.2 MyISAM 表的批量数据加载

10.6.3 优化 REPAIR TABLE 语句

`MyISAM` 存储引擎在处理读多写少的数据或低并发操作时表现最佳，因为表锁限制了同时进行更新的能力。在 MySQL 中，默认的存储引擎是 `InnoDB` 而不是 `MyISAM`。
