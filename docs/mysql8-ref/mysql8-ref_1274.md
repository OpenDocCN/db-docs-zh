# 17.20 InnoDB memcached 插件

> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-memcached.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-memcached.html)

17.20.1 InnoDB memcached 插件的优势

17.20.2 InnoDB memcached 架构

17.20.3 设置 InnoDB memcached 插件

17.20.4 InnoDB memcached 多个 get 和范围查询支持

17.20.5 InnoDB memcached 插件的安全考虑

17.20.6 为 InnoDB memcached 插件编写应用程序

17.20.7 InnoDB memcached 插件和复制

17.20.8 InnoDB memcached 插件内部

17.20.9 InnoDB memcached 插件故障排除

重要提示

`InnoDB` **memcached**插件在 MySQL 8.3.0 中被移除，在 MySQL 8.0.22 中被弃用。

`InnoDB` **memcached**插件(`daemon_memcached`)提供了一个集成的**memcached**守护程序，自动从`InnoDB`表中存储和检索数据，将 MySQL 服务器转变为快速的“键-值存储”。您可以使用简单的`get`、`set`和`incr`操作来代替在 SQL 中制定查询，避免与 SQL 解析和构建查询优化计划相关的性能开销。您也可以通过 SQL 访问相同的`InnoDB`表，以便进行方便、复杂的查询、批量操作和传统数据库软件的其他优势。

这种“NoSQL 风格”的接口使用**memcached** API 加速数据库操作，让`InnoDB`使用其缓冲池机制处理内存缓存。通过**memcached**操作修改的数据，如`add`、`set`和`incr`，会存储到磁盘中的`InnoDB`表中。**memcached**的简单性与`InnoDB`的可靠性和一致性的结合，为用户提供了最佳的两全其美，如第 17.20.1 节“InnoDB memcached 插件的优势”中所解释的。有关架构概述，请参阅第 17.20.2 节“InnoDB memcached 架构”。
