> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-deadlock-detection.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-deadlock-detection.html)

#### 17.7.5.2 死锁检测

当启用死锁检测（默认情况下），`InnoDB`会自动检测事务死锁并回滚一个或多个事务以打破死锁。`InnoDB`尝试选择要回滚的小事务，事务的大小由插入、更新或删除的行数确定。

当`innodb_table_locks = 1`（默认）且`autocommit = 0`时，`InnoDB`会意识到表锁，并且 MySQL 层知道行级锁。否则，`InnoDB`无法检测到由 MySQL `LOCK TABLES`语句设置的表锁或由`InnoDB`之外的存储引擎设置的锁所涉及的死锁。通过设置`innodb_lock_wait_timeout`系统变量的值来解决这些情况。

如果`InnoDB`监视器输出的`LATEST DETECTED DEADLOCK`部分包含一条消息，指出在锁表等待图中搜索过深或过长，我们将回滚以下事务，则表示等待列表上的事务数量已达到 200 的限制。超过 200 个事务的等待列表被视为死锁，并且试图检查等待列表的事务将被回滚。如果锁定线程必须查看等待列表上的超过 1,000,000 个事务拥有的锁，则也可能发生相同的错误。

有关组织数据库操作以避免死锁的技术，请参阅第 17.7.5 节，“InnoDB 中的死锁”。

##### 禁用死锁检测

在高并发系统中，死锁检测可能会导致大量线程等待同一锁时出现减速。有时，禁用死锁检测并依赖于`innodb_lock_wait_timeout`设置在死锁发生时进行事务回滚可能更有效。可以使用`innodb_deadlock_detect`变量禁用死锁检测。
