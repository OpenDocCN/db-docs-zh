> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-information-schema-internal-data.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-information-schema-internal-data.html)

#### 17.15.2.3 InnoDB 事务和锁信息的持久性和一致性

注意

本节描述了由性能模式 `data_locks` 和 `data_lock_waits` 表公开的锁信息，这些表在 MySQL 8.0 中取代了 `INFORMATION_SCHEMA` `INNODB_LOCKS` 和 `INNODB_LOCK_WAITS` 表。有关以旧的 `INFORMATION_SCHEMA` 表为基础的类似讨论，请参阅 InnoDB 事务和锁信息的持久性和一致性，在 MySQL 5.7 参考手册 中。

事务和锁定表（`INFORMATION_SCHEMA` `INNODB_TRX` 表，性能模式 `data_locks` 和 `data_lock_waits` 表）公开的数据代表了对快速变化数据的一瞥。这不像用户表，其中数据仅在应用程序发起的更新发生时才会更改。底层数据是内部系统管理的数据，可以非常快速地更改：

+   `INNODB_TRX`、`data_locks` 和 `data_lock_waits` 表之间的数据可能不一致。

    `data_locks` 和 `data_lock_waits` 表公开了来自 `InnoDB` 存储引擎的实时数据，提供有关 `INNODB_TRX` 表中事务的锁信息。从锁表中检索的数据存在于执行 `SELECT` 时，但在查询结果被客户端消耗时可能已经消失或更改。

    将`data_locks`与`data_lock_waits`连接可以显示在`data_lock_waits`中标识不再存在或尚不存在的`data_locks`中的父行的行。

+   事务和锁定表中的数据可能与`INFORMATION_SCHEMA` `PROCESSLIST`表或性能模式`threads`表中的数据不一致。

    例如，当比较`InnoDB`事务和锁定表中的数据与`PROCESSLIST`表中的数据时，应该小心。即使您发出单个`SELECT`（例如连接`INNODB_TRX`和`PROCESSLIST`），这些表的内容通常不一致。`INNODB_TRX`可能引用`PROCESSLIST`中不存在的行，或者当前执行的事务中`INNODB_TRX.TRX_QUERY`显示的 SQL 查询与`PROCESSLIST.INFO`中的查询不同。
