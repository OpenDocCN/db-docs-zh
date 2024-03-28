> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-information-schema-understanding-innodb-locking.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-information-schema-understanding-innodb-locking.html)

#### 17.15.2.2 InnoDB 锁和锁等待信息

注意

本节描述了由 Performance Schema `data_locks` 和 `data_lock_waits` 表公开的锁信息，它们在 MySQL 8.0 中取代了`INFORMATION_SCHEMA` `INNODB_LOCKS` 和 `INNODB_LOCK_WAITS` 表。对于以旧的`INFORMATION_SCHEMA`表为基础编写的类似讨论，请参阅 InnoDB 锁和锁等待信息，在 MySQL 5.7 参考手册中。

当一个事务更新表中的一行，或者用`SELECT FOR UPDATE`锁定它时，`InnoDB`会为该行建立一个锁的列表或队列。同样，`InnoDB`为表级锁维护一个锁的列表。如果第二个事务想要更新一个已被先前事务以不兼容模式锁定的行，或者锁定一个已被先前事务锁定的表，`InnoDB`会为该行添加一个锁请求到相应的队列中。为了让一个事务获取锁，所有之前为该行或表输入的不兼容锁请求必须被移除（这发生在持有或请求这些锁的事务提交或回滚时）。

一个事务可能对不同的行或表发出任意数量的锁请求。在任何给定时间，一个事务可能请求另一个事务持有的锁，此时它会被那个事务阻塞。请求锁的事务必须等待持有阻塞锁的事务提交或回滚。如果一个事务没有在等待锁，它处于`RUNNING`状态。如果一个事务在等待锁，它处于`LOCK WAIT`状态。（`INFORMATION_SCHEMA` `INNODB_TRX` 表显示事务状态值。）

Performance Schema `data_locks` 表为每个`LOCK WAIT`事务保存一行或多行，指示阻止其进展的任何锁请求。该表还包含描述每个锁的一行，该锁在等待给定行或表的锁队列中。Performance Schema `data_lock_waits` 表显示事务已持有的锁阻塞了其他事务请求的锁。
