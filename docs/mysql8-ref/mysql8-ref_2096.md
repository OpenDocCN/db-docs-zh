> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-statement-summary-tables.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-statement-summary-tables.html)

#### 29.12.20.3 语句摘要表

Performance Schema 维护用于收集当前和最近语句事件的表，并在摘要表中聚合该信息。第 29.12.6 节，“Performance Schema 语句事件表” 描述了语句摘要的基础事件。查看该讨论以获取有关语句事件内容、当前和历史语句事件表以及如何控制语句事件收集的信息，默认情况下部分禁用。

示例语句事件摘要信息：

```sql
mysql> SELECT *
       FROM performance_schema.events_statements_summary_global_by_event_name\G
*************************** 1\. row ***************************
                 EVENT_NAME: statement/sql/select
                 COUNT_STAR: 54
             SUM_TIMER_WAIT: 38860400000
             MIN_TIMER_WAIT: 52400000
             AVG_TIMER_WAIT: 719600000
             MAX_TIMER_WAIT: 12631800000
              SUM_LOCK_TIME: 88000000
                 SUM_ERRORS: 0
               SUM_WARNINGS: 0
          SUM_ROWS_AFFECTED: 0
              SUM_ROWS_SENT: 60
          SUM_ROWS_EXAMINED: 120
SUM_CREATED_TMP_DISK_TABLES: 0
     SUM_CREATED_TMP_TABLES: 21
       SUM_SELECT_FULL_JOIN: 16
 SUM_SELECT_FULL_RANGE_JOIN: 0
           SUM_SELECT_RANGE: 0
     SUM_SELECT_RANGE_CHECK: 0
            SUM_SELECT_SCAN: 41
      SUM_SORT_MERGE_PASSES: 0
             SUM_SORT_RANGE: 0
              SUM_SORT_ROWS: 0
              SUM_SORT_SCAN: 0
          SUM_NO_INDEX_USED: 22
     SUM_NO_GOOD_INDEX_USED: 0
               SUM_CPU_TIME: 0
      MAX_CONTROLLED_MEMORY: 2028360
           MAX_TOTAL_MEMORY: 2853429
            COUNT_SECONDARY: 0
...
```

每个语句摘要表都有一个或多个分组列，指示表如何聚合事件。事件名称指的是`setup_instruments` 表中事件工具的名称：

+   `events_statements_summary_by_account_by_event_name` 包含 `EVENT_NAME`、`USER` 和 `HOST` 列。每行总结了给定帐户（用户和主机组合）和事件名称的事件。

+   `events_statements_summary_by_digest` 包含 `SCHEMA_NAME` 和 `DIGEST` 列。每行总结了每个模式和摘要值的事件。（`DIGEST_TEXT` 列包含相应的标准化语句摘要文本，但不是分组或摘要列。`QUERY_SAMPLE_TEXT`、`QUERY_SAMPLE_SEEN` 和 `QUERY_SAMPLE_TIMER_WAIT` 列也不是分组或摘要列；它们支持语句抽样。）

    表中的最大行数在服务器启动时自动调整大小。要明确设置此最大值，请在服务器启动时设置 `performance_schema_digests_size` 系统变量。

+   `events_statements_summary_by_host_by_event_name` 包含 `EVENT_NAME` 和 `HOST` 列。每行总结了给定主机和事件名称的事件。

+   `events_statements_summary_by_program` 包含 `OBJECT_TYPE`、`OBJECT_SCHEMA` 和 `OBJECT_NAME` 列。每行总结了给定存储程序（存储过程或函数、触发器或事件）的事件。

+   `events_statements_summary_by_thread_by_event_name` 包含 `THREAD_ID` 和 `EVENT_NAME` 列。每行总结了给定线程和事件名称的事件。

+   `events_statements_summary_by_user_by_event_name`具有`EVENT_NAME`和`USER`列。每一行总结了给定用户和事件名称的事件。

+   `events_statements_summary_global_by_event_name`具有一个`EVENT_NAME`列。每一行总结了给定事件名称的事件。

+   `prepared_statements_instances`具有一个`OBJECT_INSTANCE_BEGIN`列。每一行总结了给定准备语句的事件。

每个语句摘要表都有这些包含聚合值的摘要列（除非另有说明）：

+   `COUNT_STAR`、`SUM_TIMER_WAIT`、`MIN_TIMER_WAIT`、`AVG_TIMER_WAIT`、`MAX_TIMER_WAIT`

    这些列类似于等待事件摘要表中相同名称的列（参见第 29.12.20.1 节，“等待事件摘要表”），不同之处在于语句摘要表聚合了来自`events_statements_current`而不是`events_waits_current`的事件。

    `prepared_statements_instances`表没有这些列。

+   `SUM_*`xxx`*`

    对应于`events_statements_current`表中的*`xxx`*列的聚合。例如，语句摘要表中的`SUM_LOCK_TIME`和`SUM_ERRORS`列是`events_statements_current`表中`LOCK_TIME`和`ERRORS`列的聚合。

+   `MAX_CONTROLLED_MEMORY`

    报告执行过程中语句使用的最大受控内存量。

    这一列是在 MySQL 8.0.31 中添加的。

+   `MAX_TOTAL_MEMORY`

    报告执行过程中语句使用的最大内存量。

    这一列是在 MySQL 8.0.31 中添加的。

+   `COUNT_SECONDARY`

    查询在`SECONDARY`引擎上处理的次数。用于 MySQL HeatWave 服务和 HeatWave，其中`PRIMARY`引擎是`InnoDB`，`SECONDARY`引擎是 HeatWave（`RAPID`）。对于 MySQL 社区版服务器、MySQL 企业版服务器（本地）和没有 HeatWave 的 MySQL HeatWave 服务，查询始终在`PRIMARY`引擎上处理，这意味着在这些 MySQL 服务器上该值始终为 0。`COUNT_SECONDARY`列是在 MySQL 8.0.29 中添加的。

`events_statements_summary_by_digest`表具有以下额外的摘要列：

+   `FIRST_SEEN`，`LAST_SEEN`

    指示具有给定摘要值的语句何时首次看到和最近看到的时间戳。

+   `QUANTILE_95`：语句延迟的第 95 百分位数，单位为皮秒。这个百分位数是一个高估值，是从收集的直方图数据计算得出的。换句话说，对于给定的摘要，有 95%的语句延迟低于`QUANTILE_95`。

    要访问直方图数据，请使用第 29.12.20.4 节，“语句直方图摘要表”中描述的表。

+   `QUANTILE_99`：类似于`QUANTILE_95`，但针对第 99 百分位数。

+   `QUANTILE_999`：类似于`QUANTILE_95`，但针对 99.9 百分位数。

`events_statements_summary_by_digest`表包含以下列。这些既不是分组列也不是摘要列；它们支持语句抽样：

+   `QUERY_SAMPLE_TEXT`

    生成行中摘要值的样本 SQL 语句。此列使应用程序能够访问为给定摘要值而由服务器看到的实际语句。其中一个用途可能是对频繁出现的摘要相关的代表性语句运行`EXPLAIN`以检查执行计划。

    当`QUERY_SAMPLE_TEXT`列被赋值时，`QUERY_SAMPLE_SEEN`和`QUERY_SAMPLE_TIMER_WAIT`列也被赋值。

    语句显示的最大空间默认为 1024 字节。要更改此值，请在服务器启动时设置`performance_schema_max_sql_text_length`系统变量。（更改此值也会影响其他性能模式表中的列。请参阅第 29.10 节，“性能模式语句摘要和抽样”。）

    有关语句抽样的信息，请参阅第 29.10 节，“性能模式语句摘要和抽样”。

+   `QUERY_SAMPLE_SEEN`

    指示`QUERY_SAMPLE_TEXT`列中的语句何时被看到的时间戳。

+   `QUERY_SAMPLE_TIMER_WAIT`

    `QUERY_SAMPLE_TEXT`列中样本语句的等待时间。

`events_statements_summary_by_program`表具有以下额外的摘要列：

+   `COUNT_STATEMENTS`, `SUM_STATEMENTS_WAIT`, `MIN_STATEMENTS_WAIT`, `AVG_STATEMENTS_WAIT`, `MAX_STATEMENTS_WAIT`

    关于存储程序执行期间调用的嵌套语句的统计信息。

`prepared_statements_instances` 表具有以下额外的摘要列：

+   `COUNT_EXECUTE`, `SUM_TIMER_EXECUTE`, `MIN_TIMER_EXECUTE`, `AVG_TIMER_EXECUTE`, `MAX_TIMER_EXECUTE`

    准备语句执行的聚合统计信息。

语句摘要表具有以下索引：

+   `events_transactions_summary_by_account_by_event_name`:

    +   主键为 (`USER`, `HOST`, `EVENT_NAME`)

+   `events_statements_summary_by_digest`:

    +   主键为 (`SCHEMA_NAME`, `DIGEST`)

+   `events_transactions_summary_by_host_by_event_name`:

    +   主键为 (`HOST`, `EVENT_NAME`)

+   `events_statements_summary_by_program`:

    +   主键为 (`OBJECT_TYPE`, `OBJECT_SCHEMA`, `OBJECT_NAME`)

+   `events_statements_summary_by_thread_by_event_name`:

    +   主键为 (`THREAD_ID`, `EVENT_NAME`)

+   `events_transactions_summary_by_user_by_event_name`:

    +   主键为 (`USER`, `EVENT_NAME`)

+   `events_statements_summary_global_by_event_name`:

    +   主键为 (`EVENT_NAME`)

允许对语句摘要表执行 `TRUNCATE TABLE`。其效果如下：

+   对于 `events_statements_summary_by_digest`，它会删除行。

+   对于其他未按帐户、主机或用户聚合的摘要表，截断会将摘要列重置为零，而不是删除行。

+   对于其他按帐户、主机或用户聚合的摘要表，截断会删除没有连接的帐户、主机或用户的行，并将剩余行的摘要列重置为零。

此外，按账户、主机、用户或线程汇总的每个语句摘要表都是通过截断其依赖的连接表或截断`events_statements_summary_global_by_event_name`而隐式截断的。详情请参见第 29.12.8 节，“性能模式连接表”。

此外，截断`events_statements_summary_by_digest`隐式截断`events_statements_histogram_by_digest`，截断`events_statements_summary_global_by_event_name`隐式截断`events_statements_histogram_global`。

##### 语句摘要聚合规则

如果启用了`statements_digest`消费者，则在语句完成时将聚合到`events_statements_summary_by_digest`中。聚合基于为语句计算的`DIGEST`值。

+   如果已存在具有刚完成语句的摘要值的`events_statements_summary_by_digest`行，则将该语句的统计信息聚合到该行。`LAST_SEEN`列将更新为当前时间。

+   如果没有行具有刚完成语句的摘要值，并且表未满，则为该语句创建一个新行。`FIRST_SEEN`和`LAST_SEEN`列将使用当前时间进行初始化。

+   如果没有行具有刚完成语句的语句摘要值，并且表已满，则将刚完成语句的统计信息添加到一个特殊的“全捕获”行中，该行具有`DIGEST` = `NULL`，如有必要则创建。如果创建了该行，则`FIRST_SEEN`和`LAST_SEEN`列将使用当前时间进行初始化。否则，`LAST_SEEN`列将使用当前时间进行更新。

具有`DIGEST` = `NULL`的行是保留的，因为性能模式表由于内存限制具有最大大小。`DIGEST` = `NULL`行允许不匹配其他行的摘要即使摘要表已满也能计数，使用一个常见的“其他”桶。此行帮助您估计摘要是否具有代表性：

+   一个 `DIGEST` = `NULL` 行，其 `COUNT_STAR` 值代表所有摘要的 5%，表明摘要汇总表非常具有代表性；其他行涵盖了所见语句的 95%。

+   一个 `DIGEST` = `NULL` 行，其 `COUNT_STAR` 值代表所有摘要的 50%，表明摘要汇总表并不是非常具有代表性；其他行仅涵盖了所见语句的一半。很可能数据库管理员应该增加最大表大小，以便更多在 `DIGEST` = `NULL` 行中计数的行可以使用更具体的行进行计数。默认情况下，表是自动调整大小的，但如果此大小太小，请在服务器启动时将 `performance_schema_digests_size` 系统变量设置为较大的值。

##### 存储程序仪表化行为

对于在 `setup_objects` 表中启用了仪表化的存储程序类型，`events_statements_summary_by_program` 维护存储程序的统计信息如下：

+   当对象首次在服务器中使用时，将为该对象添加一行。

+   当对象被删除时，对象的行将被移除。

+   统计数据在对象的行中进行聚合，随着其执行而进行。

参见 第 29.4.3 节，“事件预过滤”。
