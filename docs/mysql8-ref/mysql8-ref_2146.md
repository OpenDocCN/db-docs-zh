> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-innodb-lock-waits.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-innodb-lock-waits.html)

#### 30.4.3.9 `innodb_lock_waits` 和 `x$innodb_lock_waits` 视图

这些视图总结了事务正在等待的`InnoDB`锁。默认情况下，按照锁��年龄降序排序。

`innodb_lock_waits` 和 `x$innodb_lock_waits` 视图具有以下列：

+   `wait_started`

    等待锁的开始时间。

+   `wait_age`

    等待锁已等待的时间，作为`TIME`值。

+   `wait_age_secs`

    等待锁已等待的时间，以秒为单位。

+   `locked_table_schema`

    包含锁定表的模式。

+   `locked_table_name`

    锁定表的名称。

+   `locked_table_partition`

    锁定分区的名称（如果有）；否则为`NULL`。

+   `locked_table_subpartition`

    锁定子分区的名称（如果有）；否则为`NULL`。

+   `locked_index`

    锁定索引的名称。

+   `locked_type`

    等待锁的类型。

+   `waiting_trx_id`

    等待事务的 ID。

+   `waiting_trx_started`

    等待事务开始的时间。

+   `waiting_trx_age`

    等待事务已等待的时间，作为`TIME`值。

+   `waiting_trx_rows_locked`

    等待事务锁定的行数。

+   `waiting_trx_rows_modified`

    等待事务修改的行数。

+   `waiting_pid`

    等待事务的进程列表 ID。

+   `waiting_query`

    等待锁的语句。

+   `waiting_lock_id`

    等待锁的 ID。

+   `waiting_lock_mode`

    等待锁的模式。

+   `blocking_trx_id`

    阻塞等待锁的事务的 ID。

+   `blocking_pid`

    阻塞事务的进程列表 ID。

+   `blocking_query`

    阻塞事务正在执行的语句。如果发出阻塞查询的会话变为空闲，则此字段报告为 NULL。有关更多信息，请参阅在发出会话变为空闲后识别阻塞查询。

+   `blocking_lock_id`

    阻塞等待锁的锁的 ID。

+   `blocking_lock_mode`

    阻塞等待锁的锁的模式。

+   `blocking_trx_started`

    阻塞事务开始的时间。

+   `blocking_trx_age`

    阻塞事务已执行的时间，作为`TIME`值。

+   `blocking_trx_rows_locked`

    阻塞事务锁定的行数。

+   `blocking_trx_rows_modified`

    阻塞事务修改的行数。

+   `sql_kill_blocking_query`

    需要执行的`KILL`语句以终止阻塞语句。

+   `sql_kill_blocking_connection`

    需要执行的`KILL`语句以终止运行阻塞语句的会话。
