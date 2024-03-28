> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-ps-trace-statement-digest.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-ps-trace-statement-digest.html)

#### 30.4.4.22 ps_trace_statement_digest() 过程

跟踪特定语句摘要的所有性能模式仪表板。

如果在性能模式`events_statements_summary_by_digest`表中找到感兴趣的语句，请将其`DIGEST`列的 MD5 值指定给该过程，并指示轮询持续时间和间隔。结果是在该间隔内跟踪的性能模式中的所有统计信息报告。

该过程还尝试在间隔期间执行`EXPLAIN`以解释摘要中运行时间最长的示例。这次尝试可能会失败，因为性能模式会截断较长的`SQL_TEXT`值。因此，由于解析错误，`EXPLAIN`失败。

该过程通过操纵`sql_log_bin`系统变量的会话值，在执行过程中禁用二进制日志记录。这是一个受限操作，因此该过程需要具有足够权限设置受限会话变量的权限。请参阅 Section 7.1.9.1, “System Variable Privileges”。

##### 参数

+   `in_digest VARCHAR(32)`：要分析的语句摘要标识符。

+   `in_runtime INT`：分析运行的持续时间，单位为秒。

+   `in_interval DECIMAL(2,2)`：尝试在秒内（可以是小数）进行快照的分析间隔。

+   `in_start_fresh BOOLEAN`：是否在开始之前截断性能模式`events_statements_history_long`和`events_stages_history_long`表。

+   `in_auto_enable BOOLEAN`：是否自动启用所需的消费者。

##### 示例

```sql
mysql> CALL sys.ps_trace_statement_digest('891ec6860f98ba46d89dd20b0c03652c', 10, 0.1, TRUE, TRUE);
+--------------------+
| SUMMARY STATISTICS |
+--------------------+
| SUMMARY STATISTICS |
+--------------------+
1 row in set (9.11 sec)

+------------+-----------+-----------+-----------+---------------+------------+------------+
| executions | exec_time | lock_time | rows_sent | rows_examined | tmp_tables | full_scans |
+------------+-----------+-----------+-----------+---------------+------------+------------+
|         21 | 4.11 ms   | 2.00 ms   |         0 |            21 |          0 |          0 |
+------------+-----------+-----------+-----------+---------------+------------+------------+
1 row in set (9.11 sec)

+------------------------------------------+-------+-----------+
| event_name                               | count | latency   |
+------------------------------------------+-------+-----------+
| stage/sql/statistics                     |    16 | 546.92 us |
| stage/sql/freeing items                  |    18 | 520.11 us |
| stage/sql/init                           |    51 | 466.80 us |
...
| stage/sql/cleaning up                    |    18 | 11.92 us  |
| stage/sql/executing                      |    16 | 6.95 us   |
+------------------------------------------+-------+-----------+
17 rows in set (9.12 sec)

+---------------------------+
| LONGEST RUNNING STATEMENT |
+---------------------------+
| LONGEST RUNNING STATEMENT |
+---------------------------+
1 row in set (9.16 sec)

+-----------+-----------+-----------+-----------+---------------+------------+-----------+
| thread_id | exec_time | lock_time | rows_sent | rows_examined | tmp_tables | full_scan |
+-----------+-----------+-----------+-----------+---------------+------------+-----------+
|    166646 | 618.43 us | 1.00 ms   |         0 |             1 |          0 |         0 |
+-----------+-----------+-----------+-----------+---------------+------------+-----------+
1 row in set (9.16 sec)

# Truncated for clarity...
+-----------------------------------------------------------------+
| sql_text                                                        |
+-----------------------------------------------------------------+
| select hibeventhe0_.id as id1382_, hibeventhe0_.createdTime ... |
+-----------------------------------------------------------------+
1 row in set (9.17 sec)

+------------------------------------------+-----------+
| event_name                               | latency   |
+------------------------------------------+-----------+
| stage/sql/init                           | 8.61 us   |
| stage/sql/init                           | 331.07 ns |
...
| stage/sql/freeing items                  | 30.46 us  |
| stage/sql/cleaning up                    | 662.13 ns |
+------------------------------------------+-----------+
18 rows in set (9.23 sec)

+----+-------------+--------------+-------+---------------+-----------+---------+-------------+------+-------+
| id | select_type | table        | type  | possible_keys | key       | key_len | ref         | rows | Extra |
+----+-------------+--------------+-------+---------------+-----------+---------+-------------+------+-------+
|  1 | SIMPLE      | hibeventhe0_ | const | fixedTime     | fixedTime | 775     | const,const |    1 | NULL  |
+----+-------------+--------------+-------+---------------+-----------+---------+-------------+------+-------+
1 row in set (9.27 sec)

Query OK, 0 rows affected (9.28 sec)
```
