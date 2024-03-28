> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-schema-table-lock-waits.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-schema-table-lock-waits.html)

#### 30.4.3.28 The schema_table_lock_waits and x$schema_table_lock_waits Views

这些视图显示了哪些会话因等待元数据锁而被阻塞，以及是什么在阻塞它们。

这里的列描述很简要。有关更多信息，请参阅性能模式 `metadata_locks` 表的描述，链接在 Section 29.12.13.3, “The metadata_locks Table”。

`schema_table_lock_waits` 和 `x$schema_table_lock_waits` 视图具有以下列：

+   `object_schema`

    包含要锁定对象的模式。

+   `object_name`

    被检测对象的名称。

+   `waiting_thread_id`

    等待锁的线程 ID。

+   `waiting_pid`

    等待锁的线程的进程列表 ID。

+   `waiting_account`

    与等待锁的会话相关联的帐户。

+   `waiting_lock_type`

    等待锁的类型。

+   `waiting_lock_duration`

    等待锁已经等待多长时间。

+   `waiting_query`

    等待锁的语句。

+   `waiting_query_secs`

    语句等待的时间，以秒为单位。

+   `waiting_query_rows_affected`

    语句受影响的行数。

+   `waiting_query_rows_examined`

    语句从存储引擎中读取的行数。

+   `blocking_thread_id`

    阻塞等待锁的线程的线程 ID。

+   `blocking_pid`

    阻塞等待锁的线程的进程列表 ID。

+   `blocking_account`

    与阻塞等待锁的线程相关联的帐户。

+   `blocking_lock_type`

    阻塞等待锁的类型。

+   `blocking_lock_duration`

    阻塞锁已经持有多长时间。

+   `sql_kill_blocking_query`

    `KILL` 语句用于执行以终止阻塞语句。

+   `sql_kill_blocking_connection`

    `KILL` 语句用于执行以终止运行阻塞语句的会话。
