> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-events-statements-current-table.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-events-statements-current-table.html)

#### 29.12.6.1 events_statements_current 表

[`events_statements_current`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-events-statements-current-table.html)表包含当前语句事件。该表为每个线程存储一行，显示线程最近监视语句事件的当前状态，因此没有用于配置表大小的系统变量。

在包含语句事件行的表中，[`events_statements_current`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-events-statements-current-table.html)是最基本的。其他包含语句事件行的表从当前事件逻辑派生。例如，`events_statements_history`和`events_statements_history_long`表是最近已结束的语句事件的集合，每个线程最多一定数量的行，并在全局范围内跨所有线程。

有关三个`events_statements_*`xxx`*`事件表之间关系的更多信息，请参见第 29.9 节，“当前和历史事件的性能模式表”。

有关配置是否收集语句事件的信息，请参见第 29.12.6 节，“性能模式语句事件表”。

[`events_statements_current`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-events-statements-current-table.html)表具有以下列：

+   `THREAD_ID`，`EVENT_ID`

    与事件相关联的线程和事件开始时的线程当前事件编号。`THREAD_ID`和`EVENT_ID`值组合在一起唯一标识行。没有两行具有相同的数值对。

+   `END_EVENT_ID`

    当事件开始时，此列设置为`NULL`，并在事件结束时更新为线程当前事件编号。

+   `EVENT_NAME`

    从中收集事件的仪器的名称。这是来自`setup_instruments`表的`NAME`值。仪器名称可能有多个部分并形成层次结构，如第 29.6 节，“性能模式仪器命名约定”中所讨论的那样。

    对于 SQL 语句，`EVENT_NAME`值最初为`statement/com/Query`，直到语句被解析，然后根据需要更改为更合适的值，如第 29.12.6 节，“性能模式语句事件表”中所述。

+   `SOURCE`

    包含产生事件的仪器代码的源文件的名称以及发生仪器化的文件中的行号。这使您可以检查源代码以确定涉及的确切代码。

+   `TIMER_START`，`TIMER_END`，`TIMER_WAIT`

    事件的时间信息。这些值的单位为皮秒（万亿分之一秒）。`TIMER_START`和`TIMER_END`值指示事件计时开始和结束的时间。`TIMER_WAIT`是事件经过的时间（持续时间）。

    如果事件尚未完成，则`TIMER_END`是当前计时器值，`TIMER_WAIT`是到目前为止经过的时间（`TIMER_END` − `TIMER_START`）。

    如果事件是由`TIMED = NO`的仪器产生的，则不会收集时间信息，`TIMER_START`，`TIMER_END`和`TIMER_WAIT`都是`NULL`。

    有关事件时间单位为皮秒和影响时间值的因素的讨论，请参见第 29.4.1 节，“性能模式事件定时”。

+   `LOCK_TIME`

    等待表锁的时间。此值以微秒计算，但为了与其他性能模式计时器更容易比较，已标准化为皮秒。

+   `SQL_TEXT`

    SQL 语句的文本。对于与 SQL 语句不相关的命令，该值为`NULL`。

    语句显示的最大空间默认为 1024 字节。要更改此值，请在服务器启动时设置`performance_schema_max_sql_text_length`系统变量。（更改此值还会影响其他性能模式表中的列。请参见第 29.10 节，“性能模式语句摘要和抽样”。）

+   `DIGEST`

    语句摘要 SHA-256 值，以 64 个十六进制字符的字符串表示，如果`statements_digest`消费者为`no`，则为`NULL`。有关语句摘要的更多信息，请参见第 29.10 节，“性能模式语句摘要和抽样”。

+   `DIGEST_TEXT`

    标准化的语句摘要文本，或者如果`statements_digest`消费者为`no`，则为`NULL`。有关语句摘要的更多信息，请参见第 29.10 节，“性能模式语句摘要和抽样”。

    `performance_schema_max_digest_length`系统变量确定每个会话用于摘要值存储的最大字节数。然而，由于在摘要缓冲区中对语句元素（如关键字和文字值）进行编码，语句摘要的显示长度可能超过可用缓冲区大小。因此，从语句事件表的`DIGEST_TEXT`列中选择的值可能会超过`performance_schema_max_digest_length`的值。

+   `CURRENT_SCHEMA`

    语句的默认数据库，如果没有则为`NULL`。

+   `OBJECT_SCHEMA`，`OBJECT_NAME`，`OBJECT_TYPE`

    对于嵌套语句（存储过程），这些列包含有关父语句的信息。否则，它们为`NULL`。

+   `OBJECT_INSTANCE_BEGIN`

    此列标识语句。该值是内存中对象的地址。

+   `MYSQL_ERRNO`

    语句错误编号，来自语句诊断区域。

+   `RETURNED_SQLSTATE`

    语句的 SQLSTATE 值，来自语句诊断区域。

+   `MESSAGE_TEXT`

    语句错误消息，来自语句诊断区域。

+   `ERRORS`

    语句是否发生错误。如果 SQLSTATE 值以`00`（完成）或`01`（警告）开头，则值为 0。如果 SQLSTATE 值为其他值，则值为 1。

+   `WARNINGS`

    来自语句诊断区域的警告数量。

+   `ROWS_AFFECTED`

    语句影响的行数。有关“受影响”的含义，请参阅 mysql_affected_rows()。

+   `ROWS_SENT`

    语句返回的行数。

+   `ROWS_EXAMINED`

    服务器层检查的行数（不包括存储引擎内部处理）。

+   `CREATED_TMP_DISK_TABLES`

    类似于`Created_tmp_disk_tables`状态变量，但特定于语句。

+   `CREATED_TMP_TABLES`

    类似于`Created_tmp_tables`状态变量，但特定于语句。

+   `SELECT_FULL_JOIN`

    类似于`Select_full_join`状态变量，但特定于语句。

+   `SELECT_FULL_RANGE_JOIN`

    类似于`Select_full_range_join`状态变量，但特定于语句。

+   `SELECT_RANGE`

    类似于`Select_range`状态变量，但特定于语句。

+   `SELECT_RANGE_CHECK`

    类似于`Select_range_check`状态变量，但特定于语句。

+   `SELECT_SCAN`

    类似于`Select_scan`状态变量，但特定于语句。

+   `SORT_MERGE_PASSES`

    类似于 `Sort_merge_passes` 状态变量，但特定于语句。

+   `SORT_RANGE`

    类似于 `Sort_range` 状态变量，但特定于语句。

+   `SORT_ROWS`

    类似于 `Sort_rows` 状态变量，但特定于语句。

+   `SORT_SCAN`

    类似于 `Sort_scan` 状态变量，但特定于语句。

+   `NO_INDEX_USED`

    如果语句执行了不使用索引的表扫描，则为 1，否则为 0。

+   `NO_GOOD_INDEX_USED`

    如果服务器找不到适合语句的好索引，则为 1，否则为 0。有关更多信息，请参见 第 10.8.2 节，“EXPLAIN 输出格式” 中 `EXPLAIN` 输出的 `Range checked for each record` 值的 `Extra` 列描述。

+   `NESTING_EVENT_ID`、`NESTING_EVENT_TYPE`、`NESTING_EVENT_LEVEL`

    这三列与其他列一起用于为顶层（非嵌套）语句和嵌套语句（在存储过程中执行）提供以下信息。

    对于顶层语句：

    ```sql
    OBJECT_TYPE = NULL
    OBJECT_SCHEMA = NULL
    OBJECT_NAME = NULL
    NESTING_EVENT_ID = the parent transaction EVENT_ID
    NESTING_EVENT_TYPE = 'TRANSACTION'
    NESTING_LEVEL = 0
    ```

    对于嵌套语句：

    ```sql
    OBJECT_TYPE = the parent statement object type
    OBJECT_SCHEMA = the parent statement object schema
    OBJECT_NAME = the parent statement object name
    NESTING_EVENT_ID = the parent statement EVENT_ID
    NESTING_EVENT_TYPE = 'STATEMENT'
    NESTING_LEVEL = the parent statement NESTING_LEVEL plus one
    ```

+   `STATEMENT_ID`

    服务器在 SQL 级别维护的查询 ID。该值对于服务器实例是唯一的，因为这些 ID 是使用原子递增的全局计数器生成的。该列在 MySQL 8.0.14 中添加。

+   `CPU_TIME`

    当前线程在 CPU 上花费的时间，以皮秒表示。该列在 MySQL 8.0.28 中添加。

+   `MAX_CONTROLLED_MEMORY`

    报告语句在执行过程中使用的最大受控内存量。

    该列在 MySQL 8.0.31 中添加。

+   `MAX_TOTAL_MEMORY`

    报告语句在执行过程中使用的最大内存量。

    该列在 MySQL 8.0.31 中添加。

+   `EXECUTION_ENGINE`

    查询执行引擎。该值为 `PRIMARY` 或 `SECONDARY`。用于 MySQL HeatWave 服务和 HeatWave，其中 `PRIMARY` 引擎为 `InnoDB`，`SECONDARY` 引擎为 HeatWave（`RAPID`）。对于 MySQL Community Edition Server、MySQL Enterprise Edition Server（本地）和没有 HeatWave 的 MySQL HeatWave 服务，该值始终为 `PRIMARY`。该列在 MySQL 8.0.29 中添加。

`events_statements_current` 表具有以下索引：

+   主键为 (`THREAD_ID`, `EVENT_ID`)。

`TRUNCATE TABLE` 允许用于 `events_statements_current` 表。它会删除行。
