# 17.7 InnoDB 锁定和事务模型

> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-locking-transaction-model.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-locking-transaction-model.html)

17.7.1 InnoDB 锁定

17.7.2 InnoDB 事务模型

17.7.3 InnoDB 中由不同 SQL 语句设置的锁

17.7.4 幻影行

17.7.5 InnoDB 中的死锁

17.7.6 事务调度

要实现大规模、繁忙或高可靠性的数据库应用程序，从不同数据库系统移植大量代码，或调整 MySQL 性能，了解`InnoDB`锁定和`InnoDB`事务模型是非常重要的。

本节讨论了几个与`InnoDB`锁定和`InnoDB`事务模型相关的主题，您应该熟悉。

+   第 17.7.1 节，“InnoDB 锁定” 描述了`InnoDB`使用的锁类型。

+   第 17.7.2 节，“InnoDB 事务模型” 描述了事务隔离级别和每种锁定策略的使用。它还讨论了`autocommit`的使用，一致的非锁定读取和锁定读取。

+   第 17.7.3 节，“InnoDB 中由不同 SQL 语句设置的锁” 讨论了在`InnoDB`中为各种语句设置的特定类型的锁。

+   第 17.7.4 节，“幻影行” 描述了`InnoDB`如何使用下一个键锁定来避免幻影行。

+   第 17.7.5 节，“InnoDB 中的死锁” 提供了一个死锁示例，讨论了死锁检测，并提供了最小化和处理`InnoDB`中死锁的提示。
