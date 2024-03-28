> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-statement-performance-analyzer.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-statement-performance-analyzer.html)

#### 30.4.4.25 `statement_performance_analyzer()`过程

创建一个报告，显示服务器上正在运行的语句。视图是基于整体和/或增量活动计算的。

在执行此过程期间，通过操纵`sql_log_bin`系统变量的会话值来禁用二进制日志记录。这是一个受限制的操作，因此该过程需要具有足够权限以设置受限制会话变量的权限。请参阅第 7.1.9.1 节，“系统变量权限”。

##### 参数

+   `in_action ENUM('snapshot', 'overall', 'delta', 'create_tmp', 'create_table', 'save', 'cleanup')`: 执行的操作。允许使用以下数值：

    +   `snapshot`: 存储一个快照。默认情况下，会对性能模式`events_statements_summary_by_digest`表的当前内容进行快照。通过设置`in_table`，可以覆盖为复制指定表的内容。快照存储在`sys`模式下的`tmp_digests`临时表中。

    +   `overall`: 基于由`in_table`指定的表的内容生成分析。对于整体分析，`in_table`可以是`NOW()`以使用最新快照。这会覆盖现有快照。对于`in_table`为`NULL`的情况，使用现有快照。如果`in_table`为`NULL`且不存在快照，则会创建一个新的快照。`in_views`参数和`statement_performance_analyzer.limit`配置选项会影响此过程的操作。

    +   `delta`: 生成增量分析。增量是在由`in_table`指定的参考表和必须存在的快照之间计算的。此操作使用`sys`模式下的`tmp_digests_delta`临时表。`in_views`参数和`statement_performance_analyzer.limit`配置选项会影响此过程的操作。

    +   `create_table`: 创建一个适合存储快照以供以后使用的常规表（例如，用于计算增量）。

    +   `create_tmp`: 创建一个适合存储快照以供以后使用的临时表（例如，用于计算增量）。

    +   `save`: 将快照保存在由`in_table`指定的表中。表必须存在并具有正确的结构。如果不存在快照，则会创建一个新的快照。

    +   `cleanup`: 删除用于快照和增量的临时表。

+   `in_table VARCHAR(129)`: 用于由 `in_action` 参数指定的某些操作的表参数。使用格式 *`db_name.tbl_name`* 或 *`tbl_name`*，不使用任何反引号 (```sql) identifier-quoting characters. Periods (`.`) are not supported in database and table names.

    The meaning of the `in_table` value for each `in_action` value is detailed in the individual `in_action` value descriptions.

*   `in_views SET ('with_runtimes_in_95th_percentile', 'analysis', 'with_errors_or_warnings', 'with_full_table_scans', 'with_sorting', 'with_temp_tables', 'custom')`: Which views to include. This parameter is a `SET` value, so it can contain multiple view names, separated by commas. The default is to include all views except `custom`. The following values are permitted:

    *   `with_runtimes_in_95th_percentile`: Use the `statements_with_runtimes_in_95th_percentile` view.

    *   `analysis`: Use the `statement_analysis` view.

    *   `with_errors_or_warnings`: Use the `statements_with_errors_or_warnings` view.

    *   `with_full_table_scans`: Use the `statements_with_full_table_scans` view.

    *   `with_sorting`: Use the `statements_with_sorting` view.

    *   `with_temp_tables`: Use the `statements_with_temp_tables` view.

    *   `custom`: Use a custom view. This view must be specified using the `statement_performance_analyzer.view` configuration option to name a query or an existing view.

##### Configuration Options

`statement_performance_analyzer()` Procedure") operation can be modified using the following configuration options or their corresponding user-defined variables (see Section 30.4.2.1, “The sys_config Table”):

*   `debug`, `@sys.debug`

    If this option is `ON`, produce debugging output. The default is `OFF`.

*   `statement_performance_analyzer.limit`, `@sys.statement_performance_analyzer.limit`

    The maximum number of rows to return for views that have no built-in limit. The default is 100.

*   `statement_performance_analyzer.view`, `@sys.statement_performance_analyzer.view`

    The custom query or view to be used. If the option value contains a space, it is interpreted as a query. Otherwise, it must be the name of an existing view that queries the Performance Schema `events_statements_summary_by_digest` table. There cannot be any `LIMIT` clause in the query or view definition if the `statement_performance_analyzer.limit` configuration option is greater than 0\. If specifying a view, use the same format as for the `in_table` parameter. The default is `NULL` (no custom view defined).

##### Example

To create a report with the queries in the 95th percentile since the last truncation of `events_statements_summary_by_digest` and with a one-minute delta period:

1.  Create a temporary table to store the initial snapshot.

2.  Create the initial snapshot.

3.  Save the initial snapshot in the temporary table.

4.  Wait one minute.

5.  Create a new snapshot.

6.  Perform analysis based on the new snapshot.

7.  Perform analysis based on the delta between the initial and new snapshots.

```

mysql> 调用 sys.statement_performance_analyzer('create_tmp', 'mydb.tmp_digests_ini', NULL);

查询成功，0 行受影响 (0.08 秒)

mysql> 调用 sys.statement_performance_analyzer('snapshot', NULL, NULL);

查询成功，0 行受影响 (0.02 秒)

mysql> 调用 sys.statement_performance_analyzer('save', 'mydb.tmp_digests_ini', NULL);

查询成功，0 行受影响 (0.00 秒)

mysql> 休眠 60 秒;

查询成功，0 行受影响 (1 分 0.00 秒)

mysql> 调用 sys.statement_performance_analyzer('snapshot', NULL, NULL);

查询成功，0 行受影响 (0.02 秒)

mysql> 调用 sys.statement_performance_analyzer('overall', NULL, 'with_runtimes_in_95th_percentile');

+-----------------------------------------+

| 下一个输出                             |
| --- |

+-----------------------------------------+

| 运行时间在第 95 百分位数的查询 |
| --- |

+-----------------------------------------+

1 行受影响 (0.05 秒)

...

mysql> 调用 sys.statement_performance_analyzer('delta', 'mydb.tmp_digests_ini', 'with_runtimes_in_95th_percentile');

+-----------------------------------------+

| 下一个输出                             |
| --- |

+-----------------------------------------+

| 运行时间在第 95 百分位数的查询 |
| --- |

+-----------------------------------------+

1 行受影响 (0.03 秒)

...

```sql

Create an overall report of the 95th percentile queries and the top 10 queries with full table scans:

```

mysql> 调用 sys.statement_performance_analyzer('snapshot', NULL, NULL);

查询成功，0 行受影响 (0.01 秒)

mysql> 设�� @sys.statement_performance_analyzer.limit = 10;

查询成功，0 行受影响 (0.00 秒)

mysql> 调用 sys.statement_performance_analyzer('overall', NULL, 'with_runtimes_in_95th_percentile,with_full_table_scans');

+-----------------------------------------+

| 下一个输出                             |
| --- |

+-----------------------------------------+

| 运行时间在第 95 百分位数的查询 |
| --- |

+-----------------------------------------+

1 行受影响 (0.01 秒)

...

+-------------------------------------+

| 下一个输出                         |
| --- |

+-------------------------------------+

| 具有全表扫描的前 10 个查询 |
| --- |

+-------------------------------------+

1 行受影响 (0.09 秒)

...

```sql

Use a custom view showing the top 10 queries sorted by total execution time, refreshing the view every minute using the **watch** command in Linux:

```

mysql> 创建或替换视图 mydb.my_statements AS

    SELECT sys.format_statement(DIGEST_TEXT) AS query,

            SCHEMA_NAME AS db,

            COUNT_STAR AS exec_count,

            sys.format_time(SUM_TIMER_WAIT) AS total_latency,

            sys.format_time(AVG_TIMER_WAIT) AS avg_latency,

            ROUND(IFNULL(SUM_ROWS_SENT / NULLIF(COUNT_STAR, 0), 0)) AS rows_sent_avg,

            ROUND(IFNULL(SUM_ROWS_EXAMINED / NULLIF(COUNT_STAR, 0), 0)) AS rows_examined_avg,

            ROUND(IFNULL(SUM_ROWS_AFFECTED / NULLIF(COUNT_STAR, 0), 0)) AS rows_affected_avg,

            DIGEST AS digest

        FROM performance_schema.events_statements_summary_by_digest

    按 SUM_TIMER_WAIT 降序排列;

查询成功，0 行受影响 (0.10 秒)

mysql> 调用 sys.statement_performance_analyzer('create_table', 'mydb.digests_prev', NULL);

Query OK, 0 rows affected (0.10 sec)

$> watch -n 60 "mysql sys --table -e \"

> SET @sys.statement_performance_analyzer.view = 'mydb.my_statements';
> 
> SET @sys.statement_performance_analyzer.limit = 10;
> 
> CALL statement_performance_analyzer('snapshot', NULL, NULL);
> 
> CALL statement_performance_analyzer('delta', 'mydb.digests_prev', 'custom');
> 
> CALL statement_performance_analyzer('save', 'mydb.digests_prev', NULL);
> 
> \""

每隔 60.0 秒: mysql sys --table -e "        ...  Mon Dec 22 10:58:51 2014

+----------------------------------+

| 下一个输出                      |
| --- |

+----------------------------------+

| 前 10 个使用自定义视图的查询 |
| --- |

+----------------------------------+

+-------------------+-------+------------+---------------+-------------+---------------+-------------------+-------------------+----------------------------------+

| 查询             | 数据库    | 执行次数 | 总延迟 | 平均延迟 | 平均发送行数 | 平均检查行数 | 平均影响行数 | 摘要                           |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |

+-------------------+-------+------------+---------------+-------------+---------------+-------------------+-------------------+----------------------------------+

...

```
