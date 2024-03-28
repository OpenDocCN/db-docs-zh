# 17.7.6 事务调度

> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-transaction-scheduling.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-transaction-scheduling.html)

`InnoDB`使用 Contenion-Aware Transaction Scheduling (CATS)算法来优先处理等待锁的事务。当多个事务等待同一对象上的锁时，CATS 算法确定哪个事务首先获得锁。

CATS 算法通过分配调度权重来优先处理等待事务，该权重是基于事务阻塞的事务数量计算的。例如，如果两个事务正在等待同一对象上的锁，那么阻塞最多事务的事务将被分配更大的调度权重。如果权重相等，则优先考虑等待时间最长的事务。

注意

在 MySQL 8.0.20 之前，`InnoDB`还使用先进先出（FIFO）算法来调度事务，并且 CATS 算法仅在锁争用严重时使用。MySQL 8.0.20 中的 CATS 算法增强使 FIFO 算法变得多余，允许其移除。MySQL 8.0.20 之后，由 FIFO 算法执行的事务调度由 CATS 算法执行。在某些情况下，此更改可能会影响事务获得锁的顺序。

您可以通过查询信息模式`INNODB_TRX`表中的`TRX_SCHEDULE_WEIGHT`列来查看事务调度权重。权重仅针对等待事务计算。等待事务是指处于`LOCK WAIT`事务执行状态的事务，如`TRX_STATE`列所报告的那样。不等待锁的事务报告 NULL 的`TRX_SCHEDULE_WEIGHT`值。

`INNODB_METRICS`提供了用于监视代码级事务调度事件的计数器。有关使用`INNODB_METRICS`计数器的信息，请参见第 17.15.6 节，“InnoDB INFORMATION_SCHEMA Metrics Table”。

+   `lock_rec_release_attempts`

    释放记录锁的尝试次数。一次尝试可能导致释放零个或多个记录锁，因为单个结构中可能存在零个或多个记录锁。

+   `lock_rec_grant_attempts`

    尝试授予记录锁的次数。一次尝试可能导致授予零个或多个记录锁。

+   `lock_schedule_refreshes`

    分析等待图以更新调度事务权重的次数。
