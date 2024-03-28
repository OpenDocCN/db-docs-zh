# 17.20.6 为 InnoDB memcached 插件编写应用程序

> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-memcached-developing.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-memcached-developing.html)

17.20.6.1 为 InnoDB memcached 插件调整现有 MySQL 模式

17.20.6.2 为 InnoDB memcached 插件调整 memcached 应用程序

17.20.6.3 调整 InnoDB memcached 插件性能

17.20.6.4 控制 InnoDB memcached 插件的事务行为

17.20.6.5 将 DML 语句调整为 memcached 操作

17.20.6.6 在底层 InnoDB 表上执行 DML 和 DDL 语句

通常，为`InnoDB` **memcached**插件编写应用程序涉及对使用 MySQL 或**memcached** API 的现有代码进行一定程度的重写或调整。

+   使用`daemon_memcached`插件，你不再需要在低性能机器上运行许多传统的**memcached**服务器，而是将与 MySQL 服务器数量相同的**memcached**服务器运行在具有大量磁盘存储和内存的相对高性能机器上。你可能会重用一些与**memcached** API 一起工作的现有代码，但由于不同的服务器配置，可能需要进行适应。

+   通过`daemon_memcached`插件存储的数据存储在`VARCHAR`、`TEXT`或`BLOB`列中，并且必须转换为执行数值操作。你可以在应用程序端执行转换，或者在查询中使用`CAST()`函数执行转换。

+   如果你来自数据库背景，你可能习惯于具有许多列的通用 SQL 表。**memcached**代码访问的表可能只有几列，甚至只有一个列用于保存数据值。

+   你可能需要调整应用程序的部分部分，以执行单行查询、插入、更新或删除操作，以提高代码关键部分的性能。通过`InnoDB` **memcached**接口执行的查询（读取）和 DML（写入）操作在性能上可以显著提高。写入操作的性能改进通常大于读取操作的性能改进，因此你可能需要专注于调整记录日志或记录网站上交互选择的代码。

以下部分将更详细地探讨这些要点。
