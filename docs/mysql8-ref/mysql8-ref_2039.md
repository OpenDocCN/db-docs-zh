> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-events-transactions-current-table.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-events-transactions-current-table.html)

#### 29.12.7.1 事件事务当前表

[`events_transactions_current`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-events-transactions-current-table.html) 表包含当前事务事件。该表存储每个线程的一行，显示线程最近监视的事务事件的当前状态，因此没有用于配置表大小的系统变量。例如：

```sql
mysql> SELECT *
       FROM performance_schema.events_transactions_current LIMIT 1\G
*************************** 1\. row ***************************
                      THREAD_ID: 26
                       EVENT_ID: 7
                   END_EVENT_ID: NULL
                     EVENT_NAME: transaction
                          STATE: ACTIVE
                         TRX_ID: NULL
                           GTID: 3E11FA47-71CA-11E1-9E33-C80AA9429562:56
                            XID: NULL
                       XA_STATE: NULL
                         SOURCE: transaction.cc:150
                    TIMER_START: 420833537900000
                      TIMER_END: NULL
                     TIMER_WAIT: NULL
                    ACCESS_MODE: READ WRITE
                ISOLATION_LEVEL: REPEATABLE READ
                     AUTOCOMMIT: NO
           NUMBER_OF_SAVEPOINTS: 0
NUMBER_OF_ROLLBACK_TO_SAVEPOINT: 0
    NUMBER_OF_RELEASE_SAVEPOINT: 0
          OBJECT_INSTANCE_BEGIN: NULL
               NESTING_EVENT_ID: 6
             NESTING_EVENT_TYPE: STATEMENT
```

包含事务事件行的表中，[`events_transactions_current`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-events-transactions-current-table.html) 是最基础的。其他包含事务事件行的表从当前事件逻辑派生而来。例如，[`events_transactions_history`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-events-transactions-history-table.html) 和 [`events_transactions_history_long`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-events-transactions-history-long-table.html) 表分别是最近已结束的事务事件的集合，每个线程最多一定数量的行和全局跨所有线程。

有关三个事务事件表之间关系的更多信息，请参阅 [第 29.9 节“当前和历史事件的性能模式表”](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-event-tables.html)。

有关配置是否收集事务事件的信息，请参阅 [第 29.12.7 节“性能模式事务表”](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-transaction-tables.html)。

[`events_transactions_current`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-events-transactions-current-table.html) 表具有以下列：

+   `THREAD_ID`，`EVENT_ID`

    与事件关联的线程和事件开始时的线程当前事件编号。`THREAD_ID` 和 `EVENT_ID` 值一起唯一标识行。没有两行具有相同的值对。

+   `END_EVENT_ID`

    当事件开始时，此列设置为 `NULL`，当事件结束时更新为线程当前事件编号。

+   `EVENT_NAME`

    事件收集的仪器名称。这是来自 [`setup_instruments`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-setup-instruments-table.html) 表的 `NAME` 值。仪器名称可能有多个部分并形成层次结构，如 [第 29.6 节“性能模式仪器命名约定”](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-instrument-naming.html) 中讨论的那样。

+   `STATE`

    当前事务状态。其值为`ACTIVE`（在`START TRANSACTION`或`BEGIN`之后）、`COMMITTED`（在`COMMIT`之后）或`ROLLED BACK`（在`ROLLBACK`之后）。

+   `TRX_ID`

    未使用。

+   `GTID`

    GTID 列包含`gtid_next`的值，可以是`ANONYMOUS`、`AUTOMATIC`或使用格式`UUID:NUMBER`的 GTID。对于使用`gtid_next=AUTOMATIC`的事务，即所有正常客户端事务，当事务提交并分配实际 GTID 时，GTID 列会发生变化。如果`gtid_mode`为`ON`或`ON_PERMISSIVE`，GTID 列会变为事务的 GTID。如果`gtid_mode`为`OFF`或`OFF_PERMISSIVE`，GTID 列会变为`ANONYMOUS`。

+   `XID_FORMAT_ID`、`XID_GTRID` 和 `XID_BQUAL`

    XA 事务标识符的元素。其格式如第 15.3.8.1 节，“XA Transaction SQL Statements”中所述。

+   `XA_STATE`

    XA 事务的状态。其值为`ACTIVE`（在`XA START`之后）、`IDLE`（在`XA END`之后）、`PREPARED`（在`XA PREPARE`之后）、`ROLLED BACK`（在`XA ROLLBACK`之后）或`COMMITTED`（在`XA COMMIT`之后）。

    在副本上，同一 XA 事务可以在不同线程上以不同状态出现在`events_transactions_current`表中。这是因为在 XA 事务准备完成后，它会从副本的应用程序线程中分离出来，并且可以由副本上的任何线程提交或回滚。`events_transactions_current`表显示了线程上最近监视的事务事件的当前状态，并且在线程空闲时不会更新此状态。因此，即使在被另一个线程处理后，XA 事务仍然可以在原始应用程序线程中以`PREPARED`状态显示。要明确识别仍处于`PREPARED`状态且需要恢复的 XA 事务，请使用`XA RECOVER`语句，而不是性能模式事务表。

+   `SOURCE`

    包含产生事件的被检测代码的源文件名称和代码插入的文件中的行号。这使您可以检查源代码以确定涉及的确切代码。

+   `TIMER_START`，`TIMER_END`，`TIMER_WAIT`

    事件的时间信息。这些值的单位为皮秒（万亿分之一秒）。`TIMER_START`和`TIMER_END`值表示事件计时开始和结束的时间。`TIMER_WAIT`是事件经过的时间（持续时间）。

    如果事件尚未完成，`TIMER_END`是当前计时器值，`TIMER_WAIT`是到目前为止经过的时间（`TIMER_END` − `TIMER_START`）。

    如果事件是由`TIMED = NO`的工具产生的，则不会收集时间信息，`TIMER_START`，`TIMER_END`和`TIMER_WAIT`都为`NULL`。

    有关皮秒作为事件时间单位以及影响时间值的因素的讨论，请参阅第 29.4.1 节，“性能模式事件计时”。

+   `ACCESS_MODE`

    事务访问模式。其取值为`READ WRITE`或`READ ONLY`。

+   `ISOLATION_LEVEL`

    事务隔离级别。其取值为`REPEATABLE READ`，`READ COMMITTED`，`READ UNCOMMITTED`，或`SERIALIZABLE`。

+   `AUTOCOMMIT`

    事务启动时是否启用了自动提交模式。

+   `NUMBER_OF_SAVEPOINTS`，`NUMBER_OF_ROLLBACK_TO_SAVEPOINT`，`NUMBER_OF_RELEASE_SAVEPOINT`

    在事务期间发出的 `SAVEPOINT`、`ROLLBACK TO SAVEPOINT` 和 `RELEASE SAVEPOINT` 语句的数量。

+   `OBJECT_INSTANCE_BEGIN`

    未使用。

+   `NESTING_EVENT_ID`

    该事件所嵌套在其中的事件的 `EVENT_ID` 值。

+   `NESTING_EVENT_TYPE`

    嵌套事件类型。值可以是 `TRANSACTION`、`STATEMENT`、`STAGE` 或 `WAIT`。（`TRANSACTION` 不会出现，因为事务不能嵌套。）

`events_transactions_current` 表具有以下索引：

+   主键为 (`THREAD_ID`, `EVENT_ID`)

`TRUNCATE TABLE` 允许用于 `events_transactions_current` 表。它会删除行。
