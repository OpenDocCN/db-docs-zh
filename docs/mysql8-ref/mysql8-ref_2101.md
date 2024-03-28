> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-table-wait-summary-tables.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-table-wait-summary-tables.html)

#### 29.12.20.8 表 I/O 和锁等待摘要表

以下部分描述了表 I/O 和锁等待摘要表：

+   `按索引使用汇总的表 I/O 等待摘要`：每个索引的表 I/O 等待

+   按表汇总的表 I/O 等待摘要：每个表的表 I/O 等待

+   `按表汇总的表锁等待摘要`：每个表的表锁等待

##### 29.12.20.8.1 按表汇总的表 I/O 等待摘要表

`按表汇总的表 I/O 等待摘要`表汇总了所有表 I/O 等待事件，由`wait/io/table/sql/handler`工具生成。按表进行分组。

`按表汇总的表 I/O 等待摘要`表具有以下分组列，用于指示表如何汇总事件：`OBJECT_TYPE`、`OBJECT_SCHEMA`和`OBJECT_NAME`。这些列的含义与`events_waits_current`表中的含义相同。它们标识适用于哪个表的行。

`按表汇总的表 I/O 等待摘要`具有以下包含汇总值的摘要列。如列描述中所示，一些列更为通用，其值与更细粒度列的值之和相同。例如，汇总所有写入的列包含聚合插入、更新和删除的相应列的总和。通过这种方式，高级别的聚合直接可用，无需用户定义的视图来汇总低级别列���

+   `COUNT_STAR`, `SUM_TIMER_WAIT`, `MIN_TIMER_WAIT`, `AVG_TIMER_WAIT`, `MAX_TIMER_WAIT`

    这些列汇总了所有 I/O 操作。它们与相应的`*`xxx`*_READ`和`*`xxx`*_WRITE`列的总和相同。

+   `COUNT_READ`, `SUM_TIMER_READ`, `MIN_TIMER_READ`, `AVG_TIMER_READ`, `MAX_TIMER_READ`

    这些列汇总了所有读操作。它们与相应的`*`xxx`*_FETCH`列的总和相同。

+   `COUNT_WRITE`, `SUM_TIMER_WRITE`, `MIN_TIMER_WRITE`, `AVG_TIMER_WRITE`, `MAX_TIMER_WRITE`

    这些列汇总了所有写操作。它们与相应的`*`xxx`*_INSERT`、`*`xxx`*_UPDATE`和`*`xxx`*_DELETE`列的总和��同。

+   `COUNT_FETCH`, `SUM_TIMER_FETCH`, `MIN_TIMER_FETCH`, `AVG_TIMER_FETCH`, `MAX_TIMER_FETCH`

    这些列汇总了所有提取操作。

+   `COUNT_INSERT`, `SUM_TIMER_INSERT`, `MIN_TIMER_INSERT`, `AVG_TIMER_INSERT`, `MAX_TIMER_INSERT`

    这些列汇总了所有插入操作。

+   `COUNT_UPDATE`, `SUM_TIMER_UPDATE`, `MIN_TIMER_UPDATE`, `AVG_TIMER_UPDATE`, `MAX_TIMER_UPDATE`

    这些列汇总了所有更新操作。

+   `COUNT_DELETE`, `SUM_TIMER_DELETE`, `MIN_TIMER_DELETE`, `AVG_TIMER_DELETE`, `MAX_TIMER_DELETE`

    这些列汇总了所有删除操作。

`table_io_waits_summary_by_table`表具有以下索引：

+   (`OBJECT_TYPE`, `OBJECT_SCHEMA`, `OBJECT_NAME`)上的唯一索引

`TRUNCATE TABLE`允许用于表 I/O 汇总表。它将汇总列重置为零，而不是删除行。截断此表还会截断`table_io_waits_summary_by_index_usage`表。

##### 29.12.20.8.2 table_io_waits_summary_by_index_usage 表

`table_io_waits_summary_by_index_usage`表汇总了所有表索引 I/O 等待事件，由`wait/io/table/sql/handler`工具生成。分组是按表索引进行的。

`table_io_waits_summary_by_index_usage`的列几乎与`table_io_waits_summary_by_table`相同。唯一的区别是额外的分组列`INDEX_NAME`，对应于记录表 I/O 等待事件时使用的索引名称：

+   `PRIMARY`的值表示表 I/O 使用了主索引。

+   `NULL`的值表示表 I/O 未使用索引。

+   插入计入`INDEX_NAME = NULL`。

`table_io_waits_summary_by_index_usage` 表具有这些索引：

+   在 (`OBJECT_TYPE`, `OBJECT_SCHEMA`, `OBJECT_NAME`, `INDEX_NAME`) 上的唯一索引

对于表 I/O 摘要表，允许 `TRUNCATE TABLE`。它将摘要列重置为零，而不是删除行。此表也通过截断 `table_io_waits_summary_by_table` 表而被截断。更改表的索引结构的 DDL 操作可能会导致每个索引的统计信息被重置。

##### 29.12.20.8.3 table_lock_waits_summary_by_table 表

`table_lock_waits_summary_by_table` 表聚合了所有表锁等待事件，由 `wait/lock/table/sql/handler` 工具生成。分组是按表进行的。

此表包含有关内部和外部锁的信息：

+   内部锁对应于 SQL 层中的锁。目前通过调用 `thr_lock()` 实现。在事件行中，这些锁由 `OPERATION` 列区分，该列具有以下值之一：

    ```sql
    read normal
    read with shared locks
    read high priority
    read no insert
    write allow write
    write concurrent insert
    write delayed
    write low priority
    write normal
    ```

+   外部锁对应于存储引擎层中的锁。目前通过调用 `handler::external_lock()` 实现。在事件行中，这些锁由 `OPERATION` 列区分，该列具有以下值之一：

    ```sql
    read external
    write external
    ```

`table_lock_waits_summary_by_table` 表具有这些分组列，用于指示表如何聚合事件：`OBJECT_TYPE`、`OBJECT_SCHEMA` 和 `OBJECT_NAME`。这些列的含义与 `events_waits_current` 表中的含义相同。它们标识适用于哪个表的行。

`table_lock_waits_summary_by_table` 包含以下汇总列，其中包含聚合值。 如列描述中所示，某些列更一般，其值与更细粒度列的值之和相同。 例如，聚合所有锁的列包含聚合读锁和写锁的相应列之和。 通过这种方式，高级别的聚合可直接使用，无需用户定义的视图来汇总较低级别的列。

+   `COUNT_STAR`, `SUM_TIMER_WAIT`, `MIN_TIMER_WAIT`, `AVG_TIMER_WAIT`, `MAX_TIMER_WAIT`

    这些列聚合所有锁操作。 它们与相应的 `*`xxx`*_READ` 和 `*`xxx`*_WRITE` 列的和相同。

+   `COUNT_READ`, `SUM_TIMER_READ`, `MIN_TIMER_READ`, `AVG_TIMER_READ`, `MAX_TIMER_READ`

    这些列聚合所有读锁操作。 它们与相应的 `*`xxx`*_READ_NORMAL`, `*`xxx`*_READ_WITH_SHARED_LOCKS`, `*`xxx`*_READ_HIGH_PRIORITY`, 和 `*`xxx`*_READ_NO_INSERT` 列的和相同。

+   `COUNT_WRITE`, `SUM_TIMER_WRITE`, `MIN_TIMER_WRITE`, `AVG_TIMER_WRITE`, `MAX_TIMER_WRITE`

    这些列聚合所有写锁操作。 它们与相应的 `*`xxx`*_WRITE_ALLOW_WRITE`, `*`xxx`*_WRITE_CONCURRENT_INSERT`, `*`xxx`*_WRITE_LOW_PRIORITY`, 和 `*`xxx`*_WRITE_NORMAL` 列的和相同。

+   `COUNT_READ_NORMAL`, `SUM_TIMER_READ_NORMAL`, `MIN_TIMER_READ_NORMAL`, `AVG_TIMER_READ_NORMAL`, `MAX_TIMER_READ_NORMAL`

    这些列聚合内部读锁。

+   `COUNT_READ_WITH_SHARED_LOCKS`, `SUM_TIMER_READ_WITH_SHARED_LOCKS`, `MIN_TIMER_READ_WITH_SHARED_LOCKS`, `AVG_TIMER_READ_WITH_SHARED_LOCKS`, `MAX_TIMER_READ_WITH_SHARED_LOCKS`

    这些列聚合内部读锁。

+   `COUNT_READ_HIGH_PRIORITY`, `SUM_TIMER_READ_HIGH_PRIORITY`, `MIN_TIMER_READ_HIGH_PRIORITY`, `AVG_TIMER_READ_HIGH_PRIORITY`, `MAX_TIMER_READ_HIGH_PRIORITY`

    这些列聚合内部读锁。

+   `COUNT_READ_NO_INSERT`, `SUM_TIMER_READ_NO_INSERT`, `MIN_TIMER_READ_NO_INSERT`, `AVG_TIMER_READ_NO_INSERT`, `MAX_TIMER_READ_NO_INSERT`

    这些列聚合内部读锁。

+   `COUNT_READ_EXTERNAL`, `SUM_TIMER_READ_EXTERNAL`, `MIN_TIMER_READ_EXTERNAL`, `AVG_TIMER_READ_EXTERNAL`, `MAX_TIMER_READ_EXTERNAL`

    这些列聚合外部读锁。

+   `COUNT_WRITE_ALLOW_WRITE`, `SUM_TIMER_WRITE_ALLOW_WRITE`, `MIN_TIMER_WRITE_ALLOW_WRITE`, `AVG_TIMER_WRITE_ALLOW_WRITE`, `MAX_TIMER_WRITE_ALLOW_WRITE`

    这些列聚合内部写锁。

+   `COUNT_WRITE_CONCURRENT_INSERT`, `SUM_TIMER_WRITE_CONCURRENT_INSERT`, `MIN_TIMER_WRITE_CONCURRENT_INSERT`, `AVG_TIMER_WRITE_CONCURRENT_INSERT`, `MAX_TIMER_WRITE_CONCURRENT_INSERT`

    这些列聚合内部写锁。

+   `COUNT_WRITE_LOW_PRIORITY`, `SUM_TIMER_WRITE_LOW_PRIORITY`, `MIN_TIMER_WRITE_LOW_PRIORITY`, `AVG_TIMER_WRITE_LOW_PRIORITY`, `MAX_TIMER_WRITE_LOW_PRIORITY`

    这些列聚合了内部写锁。

+   `COUNT_WRITE_NORMAL`, `SUM_TIMER_WRITE_NORMAL`, `MIN_TIMER_WRITE_NORMAL`, `AVG_TIMER_WRITE_NORMAL`, `MAX_TIMER_WRITE_NORMAL`

    这些列聚合了内部写锁。

+   `COUNT_WRITE_EXTERNAL`, `SUM_TIMER_WRITE_EXTERNAL`, `MIN_TIMER_WRITE_EXTERNAL`, `AVG_TIMER_WRITE_EXTERNAL`, `MAX_TIMER_WRITE_EXTERNAL`

    这些列聚合了外部写锁。

`table_lock_waits_summary_by_table` 表具有以下索引：

+   (`OBJECT_TYPE`, `OBJECT_SCHEMA`, `OBJECT_NAME`) 上的唯一索引

`截断表` 允许用于表锁摘要表。它将摘要列重置为零，而不是删除行。
