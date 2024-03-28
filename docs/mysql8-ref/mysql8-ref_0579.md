# 10.5 优化 InnoDB 表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/optimizing-innodb.html`](https://dev.mysql.com/doc/refman/8.0/en/optimizing-innodb.html)

10.5.1 优化 InnoDB 表的存储布局

10.5.2 优化 InnoDB 事务管理

10.5.3 优化 InnoDB 只读事务

10.5.4 优化 InnoDB 重做日志记录

10.5.5 InnoDB 表的批量数据加载

10.5.6 优化 InnoDB 查询

10.5.7 优化 InnoDB DDL 操作

10.5.8 优化 InnoDB 磁盘 I/O

10.5.9 优化 InnoDB 配置变量

10.5.10 优化具有多个表的 InnoDB 系统

`InnoDB` 是 MySQL 客户通常在生产数据库中使用的存储引擎，可靠性和并发性非常重要。`InnoDB` 是 MySQL 的默认存储引擎。本节解释了如何为 `InnoDB` 表优化数据库操作。
