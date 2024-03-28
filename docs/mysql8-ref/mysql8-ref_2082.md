> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-tp-thread-state-table.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-tp-thread-state-table.html)

#### 29.12.16.3 tp_thread_state 表

注意

此处描述的性能模式表可在 MySQL 8.0.14 及更高版本中使用。在 MySQL 8.0.14 之前，请改用相应的`INFORMATION_SCHEMA`表；参见 Section 28.5.4, “INFORMATION_SCHEMA TP_THREAD_STATE 表”。

`tp_thread_state`表每个由线程池创建的用于处理连接的线程都有一行。

`tp_thread_state`表具有以下列：

+   `TP_GROUP_ID`

    线程组 ID。

+   `TP_THREAD_NUMBER`

    线程在其线程组内的 ID。`TP_GROUP_ID`和`TP_THREAD_NUMBER`一起在表中提供唯一键。

+   `PROCESS_COUNT`

    该线程当前正在执行的语句所使用的 10ms 间隔。0 表示没有语句正在执行，1 表示在第一个 10ms 内，依此类推。

+   `WAIT_TYPE`

    线程的等待类型。`NULL`表示线程未被阻塞。否则，线程被`thd_wait_begin()`调用阻塞，值指定等待类型。`tp_thread_group_stats`表的`*`xxx`*_WAIT`列累积每种等待类型的计数。

    `WAIT_TYPE`值是描述等待类型的字符串，如下表所示。

    **表 29.6 tp_thread_state 表 WAIT_TYPE 值**

    | 等待类型 | 含义 |
    | --- | --- |
    | `THD_WAIT_SLEEP` | 等待睡眠 |
    | `THD_WAIT_DISKIO` | 等待磁盘 IO |
    | `THD_WAIT_ROW_LOCK` | 等待行锁 |
    | `THD_WAIT_GLOBAL_LOCK` | 等待全局锁 |
    | `THD_WAIT_META_DATA_LOCK` | 等待元数据锁 |
    | `THD_WAIT_TABLE_LOCK` | 等待表锁 |
    | `THD_WAIT_USER_LOCK` | 等待用户锁 |
    | `THD_WAIT_BINLOG` | 等待 binlog |
    | `THD_WAIT_GROUP_COMMIT` | 等待组提交 |
    | `THD_WAIT_SYNC` | 等待 fsync |
    | 等待类型 | 含义 |

+   `TP_THREAD_TYPE`

    线程的类型。此列中显示的值是`CONNECTION_HANDLER_WORKER_THREAD`、`LISTENER_WORKER_THREAD`、`QUERY_WORKER_THREAD`或`TIMER_WORKER_THREAD`之一。

    此列是在 MySQL 8.0.32 中添加的。

+   `THREAD_ID`

    此线程的唯一标识符。该值与性能模式`threads`表的`THREAD_ID`列中使用的值相同。

    此列是在 MySQL 8.0.32 中添加的。

`tp_thread_state`表具有以下索引：

+   (`TP_GROUP_ID`, `TP_THREAD_NUMBER`)上的唯一索引

`TRUNCATE TABLE` 不允许用于 `tp_thread_state` 表。
