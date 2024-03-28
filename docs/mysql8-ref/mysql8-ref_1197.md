# 17.8 InnoDB 配置

> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-configuration.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-configuration.html)

17.8.1 InnoDB 启动配置

17.8.2 配置 InnoDB 为只读操作

17.8.3 InnoDB 缓冲池配置

17.8.4 配置 InnoDB 的线程并发性

17.8.5 配置后台 InnoDB I/O 线程数量

17.8.6 在 Linux 上使用异步 I/O

17.8.7 配置 InnoDB 的 I/O 容量

17.8.8 配置自旋锁轮询

17.8.9 清理配置

17.8.10 配置 InnoDB 的优化器统计信息

17.8.11 配置索引页的合并阈值

17.8.12 启用专用 MySQL 服务器的自动配置

本节提供了有关`InnoDB`初始化、启动以及各种组件和特性的配置信息和流程。有关优化`InnoDB`表的数据库操作的信息，请参阅 Section 10.5, “Optimizing for InnoDB Tables”。
