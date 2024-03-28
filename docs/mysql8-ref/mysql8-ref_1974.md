# 28.4.28 INFORMATION_SCHEMA INNODB_TRX 表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/information-schema-innodb-trx-table.html`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-innodb-trx-table.html)

`INNODB_TRX`表提供有关当前在`InnoDB`内部执行的每个事务的信息，包括事务是否正在等待锁定，事务开始时间以及事务正在执行的 SQL 语句（如果有）。

用法信息，请参见第 17.15.2.1 节，“使用 InnoDB 事务和锁定信息”。

`INNODB_TRX`表具有以下列：

+   `TRX_ID`

    一个在`InnoDB`内部唯一的事务 ID 号。这些 ID 不会为只读和非锁定的事务创建。详情请参见第 10.5.3 节，“优化 InnoDB 只读事务”。

+   `TRX_WEIGHT`

    事务的权重，反映（但不一定是准确的）事务修改的行数和被事务锁定的行数。为了解决死锁，`InnoDB`选择具有最小权重的事务作为“受害者”进行回滚。已更改非事务表的事务被认为比其他事务更重，无论修改和锁定的行数如何。

+   `TRX_STATE`

    事务执行状态。允许的值为`RUNNING`、`LOCK WAIT`、`ROLLING BACK`和`COMMITTING`。

+   `TRX_STARTED`

    事务开始时间。

+   `TRX_REQUESTED_LOCK_ID`

    事务当前正在等待的锁的 ID，如果`TRX_STATE`为`LOCK WAIT`；否则为`NULL`。要获取有关锁的详细信息，请将此列与性能模式`data_locks`表的`ENGINE_LOCK_ID`列连接。

+   `TRX_WAIT_STARTED`

    事务开始等待锁的时间，如果`TRX_STATE`为`LOCK WAIT`；否则为`NULL`。

+   `TRX_MYSQL_THREAD_ID`

    MySQL 线程 ID。要获取有关线程的详细信息，请将此列与`INFORMATION_SCHEMA` `PROCESSLIST`表的`ID`列连接，但请参见第 17.15.2.3 节，“InnoDB 事务和锁定信息的持久性和一致性”。

+   `TRX_QUERY`

    正在被事务执行的 SQL 语句。

+   `TRX_OPERATION_STATE`

    事务的当前操作，如果有的话；否则为`NULL`。

+   `TRX_TABLES_IN_USE`

    在处理当前事务的 SQL 语句时使用的`InnoDB`表的数量。

+   `TRX_TABLES_LOCKED`

    当前 SQL 语句在哪些`InnoDB`表上有行锁。（因为这些是行锁，而不是表锁，所以尽管某些行被锁定，表通常仍然可以被多个事务读取和写入。）

+   `TRX_LOCK_STRUCTS`

    事务保留的锁数。

+   `TRX_LOCK_MEMORY_BYTES`

    此事务在内存中锁结构所占用的总大小。

+   `TRX_ROWS_LOCKED`

    此事务锁定的行数的近似值。该值可能包括物理上存在但对事务不可见的删除标记行。

+   `TRX_ROWS_MODIFIED`

    这个事务中修改和插入的行数。

+   `TRX_CONCURRENCY_TICKETS`

    表示当前事务在被交换出之前可以完成多少工作的值，由`innodb_concurrency_tickets`系统变量指定。

+   `TRX_ISOLATION_LEVEL`

    当前事务的隔离级别。

+   `TRX_UNIQUE_CHECKS`

    当前事务是否打开或关闭唯一性检查。例如，在大量数据加载期间可能会关闭它们。

+   `TRX_FOREIGN_KEY_CHECKS`

    当前事务是否打开或关闭外键检查。例如，在大量数据加载期间可能会关闭它们。

+   `TRX_LAST_FOREIGN_KEY_ERROR`

    如果有的话，最后一个外键错误的详细错误消息；否则为`NULL`。

+   `TRX_ADAPTIVE_HASH_LATCHED`

    当前事务是否锁定自适应哈希索引。当自适应哈希索引搜索系统被分区时，单个事务不会锁定整个自适应哈希索引。自适应哈希索引分区由`innodb_adaptive_hash_index_parts`控制，默认设置为 8。

+   `TRX_ADAPTIVE_HASH_TIMEOUT`

    是否立即放弃自适应哈希索引的搜索锁，还是在 MySQL 的调用之间保留它。当没有自适应哈希索引争用时，此值保持为零，并且语句保留锁直到完成。在争用时，它倒计时到零，并且语句在每次行查找后立即释放锁。当自适应哈希索引搜索系统被分区（由`innodb_adaptive_hash_index_parts`控制）时，该值保持为 0。

+   `TRX_IS_READ_ONLY`

    值为 1 表示事务是只读的。

+   `TRX_AUTOCOMMIT_NON_LOCKING`

    值为 1 表示事务是一个不使用`FOR UPDATE`或`LOCK IN SHARED MODE`子句的`SELECT`语句，并且在启用`autocommit`的情况下执行，因此事务仅包含此一个语句。当此列和`TRX_IS_READ_ONLY`都为 1 时，`InnoDB`会优化事务以减少与更改表数据的事务相关的开销。

+   `TRX_SCHEDULE_WEIGHT`

    由内容感知事务调度（CATS）算法分配给等待锁的事务的事务调度权重。该值相对于其他事务的值。较高的值具有更大的权重。仅为处于`LOCK WAIT`状态的事务计算值，如`TRX_STATE`列所报告的那样。对于不等待锁的事务，将报告 NULL 值。`TRX_SCHEDULE_WEIGHT`值与由不同算法为不同目的计算的`TRX_WEIGHT`值不同。

#### 示例

```sql
mysql> SELECT * FROM INFORMATION_SCHEMA.INNODB_TRX\G
*************************** 1\. row ***************************
                    trx_id: 1510
                 trx_state: RUNNING
               trx_started: 2014-11-19 13:24:40
     trx_requested_lock_id: NULL
          trx_wait_started: NULL
                trx_weight: 586739
       trx_mysql_thread_id: 2
                 trx_query: DELETE FROM employees.salaries WHERE salary > 65000
       trx_operation_state: updating or deleting
         trx_tables_in_use: 1
         trx_tables_locked: 1
          trx_lock_structs: 3003
     trx_lock_memory_bytes: 450768
           trx_rows_locked: 1407513
         trx_rows_modified: 583736
   trx_concurrency_tickets: 0
       trx_isolation_level: REPEATABLE READ
         trx_unique_checks: 1
    trx_foreign_key_checks: 1
trx_last_foreign_key_error: NULL
 trx_adaptive_hash_latched: 0
 trx_adaptive_hash_timeout: 10000
          trx_is_read_only: 0
trx_autocommit_non_locking: 0
       trx_schedule_weight: NULL
```

#### 注意事项

+   使用此表格帮助诊断在高并发负载时发生的性能问题。其内容如 Section 17.15.2.3, “InnoDB 事务和锁信息的持久性和一致性”所述更新。

+   您必须具有`PROCESS`权限才能查询此表。

+   使用`INFORMATION_SCHEMA` `COLUMNS`表或`SHOW COLUMNS`语句查看有关此表的列的其他信息，包括数据类型和默认值。
