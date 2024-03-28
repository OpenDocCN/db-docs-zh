# 17.15.2 InnoDB INFORMATION_SCHEMA 事务和锁定信息

> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-information-schema-transactions.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-information-schema-transactions.html)

17.15.2.1 使用 InnoDB 事务和锁定信息

17.15.2.2 InnoDB 锁和锁等待信息

17.15.2.3 InnoDB 事务和锁定信息的持久性和一致性

注意

本节描述由 Performance Schema `data_locks` 和 `data_lock_waits` 表公开的锁定信息，这些表在 MySQL 8.0 中取代了`INFORMATION_SCHEMA` `INNODB_LOCKS` 和 `INNODB_LOCK_WAITS` 表。有关以旧的`INFORMATION_SCHEMA`表为基础撰写的类似讨论，请参阅 InnoDB INFORMATION_SCHEMA 事务和锁定信息，在 MySQL 5.7 参考手册中。

一个`INFORMATION_SCHEMA`表和两个 Performance Schema 表使您能够监视`InnoDB`事务并诊断潜在的锁定问题：

+   `INNODB_TRX`：这个`INFORMATION_SCHEMA`表提供有关每个当前在`InnoDB`中执行的事务的信息，包括事务状态（例如，它是正在运行还是等待锁），事务开始时间以及事务正在执行的特定 SQL 语句。

+   `data_locks`：这个 Performance Schema 表包含每个持有锁和每个被阻塞等待持有锁释放的锁请求的行：

    +   对于每个持有的锁都有一行，无论持有锁的事务的状态如何（`INNODB_TRX.TRX_STATE`为`RUNNING`，`LOCK WAIT`，`ROLLING BACK`或`COMMITTING`）。

    +   每个在 InnoDB 中等待另一个事务释放锁的事务（`INNODB_TRX.TRX_STATE`为`LOCK WAIT`）都被一个阻塞的锁请求所阻塞。该阻塞的锁请求是由另一个事务以不兼容模式持有的行或表锁引起的。锁请求的模式总是与阻止请求的持有锁的模式不兼容（读 vs. 写，共享 vs. 独占）。

        被阻塞的事务在另一个事务提交或回滚后才能继续，从而释放所请求的锁。对于每个被阻塞的事务，`data_locks` 包含一行，描述了事务请求的每个锁以及等待的锁。

+   `data_lock_waits`：这个性能模式表指示哪些事务正在等待特定的锁，或者哪个锁正在等待特定的事务。对于每个被阻塞的事务，这个表包含一个或多个行，指示它请求的锁以及阻止该请求的任何锁。`REQUESTING_ENGINE_LOCK_ID` 值指的是事务请求的锁，`BLOCKING_ENGINE_LOCK_ID` 值指的是（由另一个事务持有的）阻止第一个事务继续的锁。对于任何被阻塞的事务，`data_lock_waits` 中的所有行都具有相同的 `REQUESTING_ENGINE_LOCK_ID` 值，但 `BLOCKING_ENGINE_LOCK_ID` 值不同。

有关上述表的更多信息，请参见 第 28.4.28 节，“INFORMATION_SCHEMA INNODB_TRX 表”，第 29.12.13.1 节，“data_locks 表”，以及 第 29.12.13.2 节，“data_lock_waits 表”。
