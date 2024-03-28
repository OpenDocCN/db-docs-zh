# 25.6.15 NDB API 统计计数器和变量

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndb-api-statistics.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndb-api-statistics.html)

有多种类型的统计计数器与由`Ndb`对象执行或影响的操作相关联。这些操作包括启动和关闭（或中止）事务；主键和唯一键操作；表、范围和修剪扫描；线程在等待各种操作完成时被阻塞；以及由`NDBCLUSTER`发送和接收的数据和事件。在进行 NDB API 调用或将数据发送到数据节点或接收数据时，这些计数器在 NDB 内核内部递增。**mysqld**将这些计数器公开为系统状态变量；它们的值可以在`SHOW STATUS`的输出中读取，或通过查询性能模式`session_status`或`global_status`表来读取。通过比较操作`NDB`表的语句执行前后的值，您可以观察在 API 级别执行的相应操作，从而了解执行语句的成本。

您可以使用以下`SHOW STATUS`语句列出所有这些状态变量：

```sql
mysql> SHOW STATUS LIKE 'ndb_api%';
+----------------------------------------------+-----------+
| Variable_name                                | Value     |
+----------------------------------------------+-----------+
| Ndb_api_wait_exec_complete_count             | 297       |
| Ndb_api_wait_scan_result_count               | 0         |
| Ndb_api_wait_meta_request_count              | 321       |
| Ndb_api_wait_nanos_count                     | 228438645 |
| Ndb_api_bytes_sent_count                     | 33988     |
| Ndb_api_bytes_received_count                 | 66236     |
| Ndb_api_trans_start_count                    | 148       |
| Ndb_api_trans_commit_count                   | 148       |
| Ndb_api_trans_abort_count                    | 0         |
| Ndb_api_trans_close_count                    | 148       |
| Ndb_api_pk_op_count                          | 151       |
| Ndb_api_uk_op_count                          | 0         |
| Ndb_api_table_scan_count                     | 0         |
| Ndb_api_range_scan_count                     | 0         |
| Ndb_api_pruned_scan_count                    | 0         |
| Ndb_api_scan_batch_count                     | 0         |
| Ndb_api_read_row_count                       | 147       |
| Ndb_api_trans_local_read_row_count           | 37        |
| Ndb_api_adaptive_send_forced_count           | 3         |
| Ndb_api_adaptive_send_unforced_count         | 294       |
| Ndb_api_adaptive_send_deferred_count         | 0         |
| Ndb_api_event_data_count                     | 0         |
| Ndb_api_event_nondata_count                  | 0         |
| Ndb_api_event_bytes_count                    | 0         |
| Ndb_api_wait_exec_complete_count_slave       | 0         |
| Ndb_api_wait_scan_result_count_slave         | 0         |
| Ndb_api_wait_meta_request_count_slave        | 0         |
| Ndb_api_wait_nanos_count_slave               | 0         |
| Ndb_api_bytes_sent_count_slave               | 0         |
| Ndb_api_bytes_received_count_slave           | 0         |
| Ndb_api_trans_start_count_slave              | 0         |
| Ndb_api_trans_commit_count_slave             | 0         |
| Ndb_api_trans_abort_count_slave              | 0         |
| Ndb_api_trans_close_count_slave              | 0         |
| Ndb_api_pk_op_count_slave                    | 0         |
| Ndb_api_uk_op_count_slave                    | 0         |
| Ndb_api_table_scan_count_slave               | 0         |
| Ndb_api_range_scan_count_slave               | 0         |
| Ndb_api_pruned_scan_count_slave              | 0         |
| Ndb_api_scan_batch_count_slave               | 0         |
| Ndb_api_read_row_count_slave                 | 0         |
| Ndb_api_trans_local_read_row_count_slave     | 0         |
| Ndb_api_adaptive_send_forced_count_slave     | 0         |
| Ndb_api_adaptive_send_unforced_count_slave   | 0         |
| Ndb_api_adaptive_send_deferred_count_slave   | 0         |
| Ndb_api_wait_exec_complete_count_replica     | 0         |
| Ndb_api_wait_scan_result_count_replica       | 0         |
| Ndb_api_wait_meta_request_count_replica      | 0         |
| Ndb_api_wait_nanos_count_replica             | 0         |
| Ndb_api_bytes_sent_count_replica             | 0         |
| Ndb_api_bytes_received_count_replica         | 0         |
| Ndb_api_trans_start_count_replica            | 0         |
| Ndb_api_trans_commit_count_replica           | 0         |
| Ndb_api_trans_abort_count_replica            | 0         |
| Ndb_api_trans_close_count_replica            | 0         |
| Ndb_api_pk_op_count_replica                  | 0         |
| Ndb_api_uk_op_count_replica                  | 0         |
| Ndb_api_table_scan_count_replica             | 0         |
| Ndb_api_range_scan_count_replica             | 0         |
| Ndb_api_pruned_scan_count_replica            | 0         |
| Ndb_api_scan_batch_count_replica             | 0         |
| Ndb_api_read_row_count_replica               | 0         |
| Ndb_api_trans_local_read_row_count_replica   | 0         |
| Ndb_api_adaptive_send_forced_count_replica   | 0         |
| Ndb_api_adaptive_send_unforced_count_replica | 0         |
| Ndb_api_adaptive_send_deferred_count_replica | 0         |
| Ndb_api_event_data_count_injector            | 0         |
| Ndb_api_event_nondata_count_injector         | 0         |
| Ndb_api_event_bytes_count_injector           | 0         |
| Ndb_api_wait_exec_complete_count_session     | 0         |
| Ndb_api_wait_scan_result_count_session       | 0         |
| Ndb_api_wait_meta_request_count_session      | 0         |
| Ndb_api_wait_nanos_count_session             | 0         |
| Ndb_api_bytes_sent_count_session             | 0         |
| Ndb_api_bytes_received_count_session         | 0         |
| Ndb_api_trans_start_count_session            | 0         |
| Ndb_api_trans_commit_count_session           | 0         |
| Ndb_api_trans_abort_count_session            | 0         |
| Ndb_api_trans_close_count_session            | 0         |
| Ndb_api_pk_op_count_session                  | 0         |
| Ndb_api_uk_op_count_session                  | 0         |
| Ndb_api_table_scan_count_session             | 0         |
| Ndb_api_range_scan_count_session             | 0         |
| Ndb_api_pruned_scan_count_session            | 0         |
| Ndb_api_scan_batch_count_session             | 0         |
| Ndb_api_read_row_count_session               | 0         |
| Ndb_api_trans_local_read_row_count_session   | 0         |
| Ndb_api_adaptive_send_forced_count_session   | 0         |
| Ndb_api_adaptive_send_unforced_count_session | 0         |
| Ndb_api_adaptive_send_deferred_count_session | 0         |
+----------------------------------------------+-----------+
90 rows in set (0.01 sec)
```

这些状态变量也可以从性能模式`session_status`和`global_status`表中获取，如下所示：

```sql
mysql> SELECT * FROM performance_schema.session_status
 ->   WHERE VARIABLE_NAME LIKE 'ndb_api%';
+----------------------------------------------+----------------+
| VARIABLE_NAME                                | VARIABLE_VALUE |
+----------------------------------------------+----------------+
| Ndb_api_wait_exec_complete_count             | 617            |
| Ndb_api_wait_scan_result_count               | 0              |
| Ndb_api_wait_meta_request_count              | 649            |
| Ndb_api_wait_nanos_count                     | 335663491      |
| Ndb_api_bytes_sent_count                     | 65764          |
| Ndb_api_bytes_received_count                 | 86940          |
| Ndb_api_trans_start_count                    | 308            |
| Ndb_api_trans_commit_count                   | 308            |
| Ndb_api_trans_abort_count                    | 0              |
| Ndb_api_trans_close_count                    | 308            |
| Ndb_api_pk_op_count                          | 311            |
| Ndb_api_uk_op_count                          | 0              |
| Ndb_api_table_scan_count                     | 0              |
| Ndb_api_range_scan_count                     | 0              |
| Ndb_api_pruned_scan_count                    | 0              |
| Ndb_api_scan_batch_count                     | 0              |
| Ndb_api_read_row_count                       | 307            |
| Ndb_api_trans_local_read_row_count           | 77             |
| Ndb_api_adaptive_send_forced_count           | 3              |
| Ndb_api_adaptive_send_unforced_count         | 614            |
| Ndb_api_adaptive_send_deferred_count         | 0              |
| Ndb_api_event_data_count                     | 0              |
| Ndb_api_event_nondata_count                  | 0              |
| Ndb_api_event_bytes_count                    | 0              |
| Ndb_api_wait_exec_complete_count_slave       | 0              |
| Ndb_api_wait_scan_result_count_slave         | 0              |
| Ndb_api_wait_meta_request_count_slave        | 0              |
| Ndb_api_wait_nanos_count_slave               | 0              |
| Ndb_api_bytes_sent_count_slave               | 0              |
| Ndb_api_bytes_received_count_slave           | 0              |
| Ndb_api_trans_start_count_slave              | 0              |
| Ndb_api_trans_commit_count_slave             | 0              |
| Ndb_api_trans_abort_count_slave              | 0              |
| Ndb_api_trans_close_count_slave              | 0              |
| Ndb_api_pk_op_count_slave                    | 0              |
| Ndb_api_uk_op_count_slave                    | 0              |
| Ndb_api_table_scan_count_slave               | 0              |
| Ndb_api_range_scan_count_slave               | 0              |
| Ndb_api_pruned_scan_count_slave              | 0              |
| Ndb_api_scan_batch_count_slave               | 0              |
| Ndb_api_read_row_count_slave                 | 0              |
| Ndb_api_trans_local_read_row_count_slave     | 0              |
| Ndb_api_adaptive_send_forced_count_slave     | 0              |
| Ndb_api_adaptive_send_unforced_count_slave   | 0              |
| Ndb_api_adaptive_send_deferred_count_slave   | 0              |
| Ndb_api_wait_exec_complete_count_replica     | 0              |
| Ndb_api_wait_scan_result_count_replica       | 0              |
| Ndb_api_wait_meta_request_count_replica      | 0              |
| Ndb_api_wait_nanos_count_replica             | 0              |
| Ndb_api_bytes_sent_count_replica             | 0              |
| Ndb_api_bytes_received_count_replica         | 0              |
| Ndb_api_trans_start_count_replica            | 0              |
| Ndb_api_trans_commit_count_replica           | 0              |
| Ndb_api_trans_abort_count_replica            | 0              |
| Ndb_api_trans_close_count_replica            | 0              |
| Ndb_api_pk_op_count_replica                  | 0              |
| Ndb_api_uk_op_count_replica                  | 0              |
| Ndb_api_table_scan_count_replica             | 0              |
| Ndb_api_range_scan_count_replica             | 0              |
| Ndb_api_pruned_scan_count_replica            | 0              |
| Ndb_api_scan_batch_count_replica             | 0              |
| Ndb_api_read_row_count_replica               | 0              |
| Ndb_api_trans_local_read_row_count_replica   | 0              |
| Ndb_api_adaptive_send_forced_count_replica   | 0              |
| Ndb_api_adaptive_send_unforced_count_replica | 0              |
| Ndb_api_adaptive_send_deferred_count_replica | 0              |
| Ndb_api_event_data_count_injector            | 0              |
| Ndb_api_event_nondata_count_injector         | 0              |
| Ndb_api_event_bytes_count_injector           | 0              |
| Ndb_api_wait_exec_complete_count_session     | 0              |
| Ndb_api_wait_scan_result_count_session       | 0              |
| Ndb_api_wait_meta_request_count_session      | 0              |
| Ndb_api_wait_nanos_count_session             | 0              |
| Ndb_api_bytes_sent_count_session             | 0              |
| Ndb_api_bytes_received_count_session         | 0              |
| Ndb_api_trans_start_count_session            | 0              |
| Ndb_api_trans_commit_count_session           | 0              |
| Ndb_api_trans_abort_count_session            | 0              |
| Ndb_api_trans_close_count_session            | 0              |
| Ndb_api_pk_op_count_session                  | 0              |
| Ndb_api_uk_op_count_session                  | 0              |
| Ndb_api_table_scan_count_session             | 0              |
| Ndb_api_range_scan_count_session             | 0              |
| Ndb_api_pruned_scan_count_session            | 0              |
| Ndb_api_scan_batch_count_session             | 0              |
| Ndb_api_read_row_count_session               | 0              |
| Ndb_api_trans_local_read_row_count_session   | 0              |
| Ndb_api_adaptive_send_forced_count_session   | 0              |
| Ndb_api_adaptive_send_unforced_count_session | 0              |
| Ndb_api_adaptive_send_deferred_count_session | 0              |
+----------------------------------------------+----------------+
90 rows in set (0.01 sec)

mysql> SELECT * FROM performance_schema.global_status
 ->     WHERE VARIABLE_NAME LIKE 'ndb_api%';
+----------------------------------------------+----------------+
| VARIABLE_NAME                                | VARIABLE_VALUE |
+----------------------------------------------+----------------+
| Ndb_api_wait_exec_complete_count             | 741            |
| Ndb_api_wait_scan_result_count               | 0              |
| Ndb_api_wait_meta_request_count              | 777            |
| Ndb_api_wait_nanos_count                     | 373888309      |
| Ndb_api_bytes_sent_count                     | 78124          |
| Ndb_api_bytes_received_count                 | 94988          |
| Ndb_api_trans_start_count                    | 370            |
| Ndb_api_trans_commit_count                   | 370            |
| Ndb_api_trans_abort_count                    | 0              |
| Ndb_api_trans_close_count                    | 370            |
| Ndb_api_pk_op_count                          | 373            |
| Ndb_api_uk_op_count                          | 0              |
| Ndb_api_table_scan_count                     | 0              |
| Ndb_api_range_scan_count                     | 0              |
| Ndb_api_pruned_scan_count                    | 0              |
| Ndb_api_scan_batch_count                     | 0              |
| Ndb_api_read_row_count                       | 369            |
| Ndb_api_trans_local_read_row_count           | 93             |
| Ndb_api_adaptive_send_forced_count           | 3              |
| Ndb_api_adaptive_send_unforced_count         | 738            |
| Ndb_api_adaptive_send_deferred_count         | 0              |
| Ndb_api_event_data_count                     | 0              |
| Ndb_api_event_nondata_count                  | 0              |
| Ndb_api_event_bytes_count                    | 0              |
| Ndb_api_wait_exec_complete_count_slave       | 0              |
| Ndb_api_wait_scan_result_count_slave         | 0              |
| Ndb_api_wait_meta_request_count_slave        | 0              |
| Ndb_api_wait_nanos_count_slave               | 0              |
| Ndb_api_bytes_sent_count_slave               | 0              |
| Ndb_api_bytes_received_count_slave           | 0              |
| Ndb_api_trans_start_count_slave              | 0              |
| Ndb_api_trans_commit_count_slave             | 0              |
| Ndb_api_trans_abort_count_slave              | 0              |
| Ndb_api_trans_close_count_slave              | 0              |
| Ndb_api_pk_op_count_slave                    | 0              |
| Ndb_api_uk_op_count_slave                    | 0              |
| Ndb_api_table_scan_count_slave               | 0              |
| Ndb_api_range_scan_count_slave               | 0              |
| Ndb_api_pruned_scan_count_slave              | 0              |
| Ndb_api_scan_batch_count_slave               | 0              |
| Ndb_api_read_row_count_slave                 | 0              |
| Ndb_api_trans_local_read_row_count_slave     | 0              |
| Ndb_api_adaptive_send_forced_count_slave     | 0              |
| Ndb_api_adaptive_send_unforced_count_slave   | 0              |
| Ndb_api_adaptive_send_deferred_count_slave   | 0              |
| Ndb_api_wait_exec_complete_count_replica     | 0              |
| Ndb_api_wait_scan_result_count_replica       | 0              |
| Ndb_api_wait_meta_request_count_replica      | 0              |
| Ndb_api_wait_nanos_count_replica             | 0              |
| Ndb_api_bytes_sent_count_replica             | 0              |
| Ndb_api_bytes_received_count_replica         | 0              |
| Ndb_api_trans_start_count_replica            | 0              |
| Ndb_api_trans_commit_count_replica           | 0              |
| Ndb_api_trans_abort_count_replica            | 0              |
| Ndb_api_trans_close_count_replica            | 0              |
| Ndb_api_pk_op_count_replica                  | 0              |
| Ndb_api_uk_op_count_replica                  | 0              |
| Ndb_api_table_scan_count_replica             | 0              |
| Ndb_api_range_scan_count_replica             | 0              |
| Ndb_api_pruned_scan_count_replica            | 0              |
| Ndb_api_scan_batch_count_replica             | 0              |
| Ndb_api_read_row_count_replica               | 0              |
| Ndb_api_trans_local_read_row_count_replica   | 0              |
| Ndb_api_adaptive_send_forced_count_replica   | 0              |
| Ndb_api_adaptive_send_unforced_count_replica | 0              |
| Ndb_api_adaptive_send_deferred_count_replica | 0              |
| Ndb_api_event_data_count_injector            | 0              |
| Ndb_api_event_nondata_count_injector         | 0              |
| Ndb_api_event_bytes_count_injector           | 0              |
| Ndb_api_wait_exec_complete_count_session     | 0              |
| Ndb_api_wait_scan_result_count_session       | 0              |
| Ndb_api_wait_meta_request_count_session      | 0              |
| Ndb_api_wait_nanos_count_session             | 0              |
| Ndb_api_bytes_sent_count_session             | 0              |
| Ndb_api_bytes_received_count_session         | 0              |
| Ndb_api_trans_start_count_session            | 0              |
| Ndb_api_trans_commit_count_session           | 0              |
| Ndb_api_trans_abort_count_session            | 0              |
| Ndb_api_trans_close_count_session            | 0              |
| Ndb_api_pk_op_count_session                  | 0              |
| Ndb_api_uk_op_count_session                  | 0              |
| Ndb_api_table_scan_count_session             | 0              |
| Ndb_api_range_scan_count_session             | 0              |
| Ndb_api_pruned_scan_count_session            | 0              |
| Ndb_api_scan_batch_count_session             | 0              |
| Ndb_api_read_row_count_session               | 0              |
| Ndb_api_trans_local_read_row_count_session   | 0              |
| Ndb_api_adaptive_send_forced_count_session   | 0              |
| Ndb_api_adaptive_send_unforced_count_session | 0              |
| Ndb_api_adaptive_send_deferred_count_session | 0              |
+----------------------------------------------+----------------+
90 rows in set (0.01 sec)
```

每个`Ndb`对象都有自己的计数器。NDB API 应用程序可以读取这些计数器的值，用于优化或监控。对于同时使用多个`Ndb`对象的多线程客户端，还可以从属于给定`Ndb_cluster_connection`的所有`Ndb`对象获取计数器的总和视图。

有四组这些计数器可供查看。一组适用于当前会话；另外 3 组是全局的。*尽管它们的值可以作为**mysql**客户端的会话或全局状态变量获得*。这意味着在`SHOW STATUS`中指定`SESSION`或`GLOBAL`关键字对 NDB API 统计状态变量报告的值没有影响，每个变量的值无论是从`session_status`表的等效列还是从`global_status`表获得，其值都是相同的。

+   *会话计数器（特定会话）*

    会话计数器与当前会话中使用的`Ndb`对象相关。其他 MySQL 客户端对这些对象的使用不会影响这些计数。

    为了最大限度地减少与标准 MySQL 会话变量的混淆，我们将与这些 NDB API 会话计数器对应的变量称为“`_session`变量”，带有前导下划线。

+   *复制计数器（全局）*

    这组计数器与复制 SQL 线程（如果有）使用的`Ndb`对象相关。如果此**mysqld**不充当复制品，或不使用`NDB`表，则所有这些计数都为 0。

    我们将相关的状态变量称为“`_slave`变量”（带有前导下划线）。

+   *注入器计数器（全局）*

    注入器计数器与二进制日志注入器线程监听集群事件所使用的`Ndb`对象相关。即使不写入二进制日志，附加到 NDB 集群的**mysqld**进程仍会继续监听某些事件，如模式更改。

    我们将与 NDB API 注入器计数器对应的状态变量称为“`_injector`变量”（带有前导下划线）。

+   *服务器（全局）计数器（全局）*

    这组计数器与当前**mysqld**使用的所有`Ndb`对象相关。这包括所有 MySQL 客户端应用程序、复制 SQL 线程（如果有）、二进制日志注入器和`NDB`实用程序线程。

    我们将与这些计数器对应的状态变量称为“全局变量”或“**mysqld**级别变量”。

你可以通过在变量名中额外过滤子字符串 `session`、`slave` 或 `injector`（以及共同前缀 `Ndb_api`）来获取特定一组变量的值。对于 `_session` 变量，可以按照这里所示进行操作：

```sql
mysql> SHOW STATUS LIKE 'ndb_api%session';
+--------------------------------------------+---------+
| Variable_name                              | Value   |
+--------------------------------------------+---------+
| Ndb_api_wait_exec_complete_count_session   | 2       |
| Ndb_api_wait_scan_result_count_session     | 0       |
| Ndb_api_wait_meta_request_count_session    | 1       |
| Ndb_api_wait_nanos_count_session           | 8144375 |
| Ndb_api_bytes_sent_count_session           | 68      |
| Ndb_api_bytes_received_count_session       | 84      |
| Ndb_api_trans_start_count_session          | 1       |
| Ndb_api_trans_commit_count_session         | 1       |
| Ndb_api_trans_abort_count_session          | 0       |
| Ndb_api_trans_close_count_session          | 1       |
| Ndb_api_pk_op_count_session                | 1       |
| Ndb_api_uk_op_count_session                | 0       |
| Ndb_api_table_scan_count_session           | 0       |
| Ndb_api_range_scan_count_session           | 0       |
| Ndb_api_pruned_scan_count_session          | 0       |
| Ndb_api_scan_batch_count_session           | 0       |
| Ndb_api_read_row_count_session             | 1       |
| Ndb_api_trans_local_read_row_count_session | 1       |
+--------------------------------------------+---------+
18 rows in set (0.50 sec)
```

要获取 NDB API **mysqld** 级别状态变量的列表，请过滤以 `ndb_api` 开头且以 `_count` 结尾的变量名，就像这样：

```sql
mysql> SELECT * FROM performance_schema.session_status
 ->     WHERE VARIABLE_NAME LIKE 'ndb_api%count';
+------------------------------------+----------------+
| VARIABLE_NAME                      | VARIABLE_VALUE |
+------------------------------------+----------------+
| NDB_API_WAIT_EXEC_COMPLETE_COUNT   | 4              |
| NDB_API_WAIT_SCAN_RESULT_COUNT     | 3              |
| NDB_API_WAIT_META_REQUEST_COUNT    | 28             |
| NDB_API_WAIT_NANOS_COUNT           | 53756398       |
| NDB_API_BYTES_SENT_COUNT           | 1060           |
| NDB_API_BYTES_RECEIVED_COUNT       | 9724           |
| NDB_API_TRANS_START_COUNT          | 3              |
| NDB_API_TRANS_COMMIT_COUNT         | 2              |
| NDB_API_TRANS_ABORT_COUNT          | 0              |
| NDB_API_TRANS_CLOSE_COUNT          | 3              |
| NDB_API_PK_OP_COUNT                | 2              |
| NDB_API_UK_OP_COUNT                | 0              |
| NDB_API_TABLE_SCAN_COUNT           | 1              |
| NDB_API_RANGE_SCAN_COUNT           | 0              |
| NDB_API_PRUNED_SCAN_COUNT          | 0              |
| NDB_API_SCAN_BATCH_COUNT           | 0              |
| NDB_API_READ_ROW_COUNT             | 2              |
| NDB_API_TRANS_LOCAL_READ_ROW_COUNT | 2              |
| NDB_API_EVENT_DATA_COUNT           | 0              |
| NDB_API_EVENT_NONDATA_COUNT        | 0              |
| NDB_API_EVENT_BYTES_COUNT          | 0              |
+------------------------------------+----------------+
21 rows in set (0.09 sec)
```

并非所有计数器都反映在所有 4 组状态变量中。对于事件计数器 `DataEventsRecvdCount`、`NondataEventsRecvdCount` 和 `EventBytesRecvdCount`，仅有 `_injector` 和 **mysqld** 级别的 NDB API 状态变量可用：

```sql
mysql> SHOW STATUS LIKE 'ndb_api%event%';
+--------------------------------------+-------+
| Variable_name                        | Value |
+--------------------------------------+-------+
| Ndb_api_event_data_count_injector    | 0     |
| Ndb_api_event_nondata_count_injector | 0     |
| Ndb_api_event_bytes_count_injector   | 0     |
| Ndb_api_event_data_count             | 0     |
| Ndb_api_event_nondata_count          | 0     |
| Ndb_api_event_bytes_count            | 0     |
+--------------------------------------+-------+
6 rows in set (0.00 sec)
```

对于任何其他 NDB API 计数器，都未实现 `_injector` 状态变量，如下所示：

```sql
mysql> SHOW STATUS LIKE 'ndb_api%injector%';
+--------------------------------------+-------+
| Variable_name                        | Value |
+--------------------------------------+-------+
| Ndb_api_event_data_count_injector    | 0     |
| Ndb_api_event_nondata_count_injector | 0     |
| Ndb_api_event_bytes_count_injector   | 0     |
+--------------------------------------+-------+
3 rows in set (0.00 sec)
```

状态变量的名称可以轻松与相应计数器的名称关联起来。每个 NDB API 统计计数器在以下表中列出，包括描述以及与该计数器对应的任何 MySQL 服务器状态变量的名称。

**表 25.67 NDB API 统计计数器**

| 计数器名称 | 描述 | 状态变量（按统计类型）：

+   会话

+   从库（副本）

+   注入器

+   服务器

|

| `WaitExecCompleteCount` | 线程在等待操作完成时被阻塞的次数。包括所有 `execute()` 调用，以及对 blob 操作和对客户端不可见的自增操作的隐式执行。 |
| --- | --- |

+   `Ndb_api_wait_exec_complete_count_session`

+   `Ndb_api_wait_exec_complete_count_slave`

+   [无]

+   `Ndb_api_wait_exec_complete_count`

|

| `WaitScanResultCount` | 线程在等待扫描信号时被阻塞的次数，例如等待额外结果或等待扫描关闭。 |
| --- | --- |

+   `Ndb_api_wait_scan_result_count_session`

+   `Ndb_api_wait_scan_result_count_slave`

+   [无]

+   `Ndb_api_wait_scan_result_count`

|

| `WaitMetaRequestCount` | 线程被阻塞等待基于元数据的信号的次数；当等待 DDL 操作或等待开始（或结束）时代时，可能会发生这种情况。 |
| --- | --- |

+   `Ndb_api_wait_meta_request_count_session`

+   `Ndb_api_wait_meta_request_count_slave`

+   [无]

+   `Ndb_api_wait_meta_request_count`

|

| `WaitNanosCount` | 从数据节点等待某种信号所花费的总时间（以纳秒为单位）。 |
| --- | --- |

+   `Ndb_api_wait_nanos_count_session`

+   `Ndb_api_wait_nanos_count_slave`

+   [none]

+   `Ndb_api_wait_nanos_count`

|

| `BytesSentCount` | 发送到数据节点的数据量（以字节为单位）。 |
| --- | --- |

+   `Ndb_api_bytes_sent_count_session`

+   `Ndb_api_bytes_sent_count_slave`

+   [none]

+   `Ndb_api_bytes_sent_count`

|

| `BytesRecvdCount` | 从数据节点接收的数据量（以字节为单位）。 |
| --- | --- |

+   `Ndb_api_bytes_received_count_session`

+   `Ndb_api_bytes_received_count_slave`

+   [none]

+   `Ndb_api_bytes_received_count`

|

| `TransStartCount` | 开始的事务数。 |
| --- | --- |

+   `Ndb_api_trans_start_count_session`

+   `Ndb_api_trans_start_count_slave`

+   [none]

+   `Ndb_api_trans_start_count`

|

| `TransCommitCount` | 提交的事务数。 |
| --- | --- |

+   `Ndb_api_trans_commit_count_session`

+   `Ndb_api_trans_commit_count_slave`

+   [none]

+   `Ndb_api_trans_commit_count`

|

| `TransAbortCount` | 中止的事务数。 |
| --- | --- |

+   `Ndb_api_trans_abort_count_session`

+   `Ndb_api_trans_abort_count_slave`

+   [none]

+   `Ndb_api_trans_abort_count`

|

| `TransCloseCount` | 中止的事务数。（此值可能大于`TransCommitCount`和`TransAbortCount`的总和。） |
| --- | --- |

+   `Ndb_api_trans_close_count_session`

+   `Ndb_api_trans_close_count_slave`

+   [none]

+   `Ndb_api_trans_close_count`

|

| `PkOpCount` | 基于或使用主键的操作次数。此计数包括 blob-part 表操作、隐式解锁操作和自增操作，以及通常对 MySQL 客户端可见的主键操作。 |
| --- | --- |

+   `Ndb_api_pk_op_count_session`

+   `Ndb_api_pk_op_count_slave`

+   [无]

+   `Ndb_api_pk_op_count`

|

| `UkOpCount` | 基于或使用唯一键的操作次数。 |
| --- | --- |

+   `Ndb_api_uk_op_count_session`

+   `Ndb_api_uk_op_count_slave`

+   [无]

+   `Ndb_api_uk_op_count`

|

| `TableScanCount` | 已启动的表扫描次数。这包括对内部表的扫描。 |
| --- | --- |

+   `Ndb_api_table_scan_count_session`

+   `Ndb_api_table_scan_count_slave`

+   [无]

+   `Ndb_api_table_scan_count`

|

| `RangeScanCount` | 已启动的范围扫描次数。 |
| --- | --- |

+   `Ndb_api_range_scan_count_session`

+   `Ndb_api_range_scan_count_slave`

+   [无]

+   `Ndb_api_range_scan_count`

|

| `PrunedScanCount` | 已被剪枝为单个分区的扫描次数。 |
| --- | --- |

+   `Ndb_api_pruned_scan_count_session`

+   `Ndb_api_pruned_scan_count_slave`

+   [无]

+   `Ndb_api_pruned_scan_count`

|

| `ScanBatchCount` | 接收的行批次数。 （在此上下文中，批次是来自单个片段的扫描结果集。） |
| --- | --- |

+   `Ndb_api_scan_batch_count_session`

+   `Ndb_api_scan_batch_count_slave`

+   [无]

+   `Ndb_api_scan_batch_count`

|

| `ReadRowCount` | 已读取的总行数。包括使用主键、唯一键和扫描操作读取的行。 |
| --- | --- |

+   `Ndb_api_read_row_count_session`

+   `Ndb_api_read_row_count_slave`

+   [无]

+   `Ndb_api_read_row_count`

|

| `TransLocalReadRowCount` | 从运行事务的同一节点上读取的行数。 |
| --- | --- |

+   `Ndb_api_trans_local_read_row_count_session`

+   `Ndb_api_trans_local_read_row_count_slave`

+   [无]

+   `Ndb_api_trans_local_read_row_count`

|

| `DataEventsRecvdCount` | 接收的行更改事件数。 |
| --- | --- |

+   [无]

+   [无]

+   `Ndb_api_event_data_count_injector`

+   `Ndb_api_event_data_count`

|

| `NondataEventsRecvdCount` | 接收的事件数，除了行更改事件。 |
| --- | --- |

+   [无]

+   [无]

+   `Ndb_api_event_nondata_count_injector`

+   `Ndb_api_event_nondata_count`

|

| `EventBytesRecvdCount` | 接收的事件字节数。 |
| --- | --- |

+   [无]

+   [无]

+   `Ndb_api_event_bytes_count_injector`

+   `Ndb_api_event_bytes_count`

|

| 计数器名称 | 描述 | 状态变量（按统计类型）：

+   会话

+   从属（副本）

+   注入器

+   服务器

|

要查看所有已提交事务的计数，即所有`TransCommitCount`计数器状态变量，您可以将`SHOW STATUS`的结果过滤为子字符串`trans_commit_count`，就像这样：

```sql
mysql> SHOW STATUS LIKE '%trans_commit_count%';
+------------------------------------+-------+
| Variable_name                      | Value |
+------------------------------------+-------+
| Ndb_api_trans_commit_count_session | 1     |
| Ndb_api_trans_commit_count_slave   | 0     |
| Ndb_api_trans_commit_count         | 2     |
+------------------------------------+-------+
3 rows in set (0.00 sec)
```

从中您可以确定当前**mysql**客户端会话中已提交了 1 个事务，并且自上次重新启动以来，此**mysqld**上已提交了 2 个事务。

您可以通过比较执行语句前后相应`_session`状态变量的值来查看给定 SQL 语句如何递增各种 NDB API 计数器。在此示例中，在从`SHOW STATUS`获取初始值后，我们在`test`数据库中创建一个名为`t`的`NDB`表，该表只有一列：

```sql
mysql> SHOW STATUS LIKE 'ndb_api%session%';
+--------------------------------------------+--------+
| Variable_name                              | Value  |
+--------------------------------------------+--------+
| Ndb_api_wait_exec_complete_count_session   | 2      |
| Ndb_api_wait_scan_result_count_session     | 0      |
| Ndb_api_wait_meta_request_count_session    | 3      |
| Ndb_api_wait_nanos_count_session           | 820705 |
| Ndb_api_bytes_sent_count_session           | 132    |
| Ndb_api_bytes_received_count_session       | 372    |
| Ndb_api_trans_start_count_session          | 1      |
| Ndb_api_trans_commit_count_session         | 1      |
| Ndb_api_trans_abort_count_session          | 0      |
| Ndb_api_trans_close_count_session          | 1      |
| Ndb_api_pk_op_count_session                | 1      |
| Ndb_api_uk_op_count_session                | 0      |
| Ndb_api_table_scan_count_session           | 0      |
| Ndb_api_range_scan_count_session           | 0      |
| Ndb_api_pruned_scan_count_session          | 0      |
| Ndb_api_scan_batch_count_session           | 0      |
| Ndb_api_read_row_count_session             | 1      |
| Ndb_api_trans_local_read_row_count_session | 1      |
+--------------------------------------------+--------+
18 rows in set (0.00 sec)

mysql> USE test;
Database changed
mysql> CREATE TABLE t (c INT) ENGINE NDBCLUSTER;
Query OK, 0 rows affected (0.85 sec)
```

现在，您可以执行一个新的`SHOW STATUS`语句并观察更改，如下所示（在输出中突出显示更改的行）：

```sql
mysql> SHOW STATUS LIKE 'ndb_api%session%';
+--------------------------------------------+-----------+
| Variable_name                              | Value     |
+--------------------------------------------+-----------+ *| Ndb_api_wait_exec_complete_count_session   | 8         |* | Ndb_api_wait_scan_result_count_session     | 0         |
*| Ndb_api_wait_meta_request_count_session    | 17        |*
*| Ndb_api_wait_nanos_count_session           | 706871709 |*
*| Ndb_api_bytes_sent_count_session           | 2376      |*
*| Ndb_api_bytes_received_count_session       | 3844      |*
*| Ndb_api_trans_start_count_session          | 4         |*
*| Ndb_api_trans_commit_count_session         | 4         |*
| Ndb_api_trans_abort_count_session          | 0         |
*| Ndb_api_trans_close_count_session          | 4         |*
*| Ndb_api_pk_op_count_session                | 6         |*
| Ndb_api_uk_op_count_session                | 0         |
| Ndb_api_table_scan_count_session           | 0         |
| Ndb_api_range_scan_count_session           | 0         |
| Ndb_api_pruned_scan_count_session          | 0         |
| Ndb_api_scan_batch_count_session           | 0         |
*| Ndb_api_read_row_count_session             | 2         |*
| Ndb_api_trans_local_read_row_count_session | 1         |
+--------------------------------------------+-----------+
18 rows in set (0.00 sec)
```

同样，您可以看到通过向`t`插入一行导致的 NDB API 统计计数器的变化：插入行，然后运行与前一个示例中使用的相同的`SHOW STATUS`语句，如下所示：

```sql
mysql> INSERT INTO t VALUES (100);
Query OK, 1 row affected (0.00 sec)

mysql> SHOW STATUS LIKE 'ndb_api%session%';
+--------------------------------------------+-----------+
| Variable_name                              | Value     |
+--------------------------------------------+-----------+
*| Ndb_api_wait_exec_complete_count_session   | 11        |*
*| Ndb_api_wait_scan_result_count_session     | 6         |*
*| Ndb_api_wait_meta_request_count_session    | 20        |*
*| Ndb_api_wait_nanos_count_session           | 707370418 |*
*| Ndb_api_bytes_sent_count_session           | 2724      |*
*| Ndb_api_bytes_received_count_session       | 4116      |*
*| Ndb_api_trans_start_count_session          | 7         |*
*| Ndb_api_trans_commit_count_session         | 6         |*
| Ndb_api_trans_abort_count_session          | 0         |
*| Ndb_api_trans_close_count_session          | 7         |*
*| Ndb_api_pk_op_count_session                | 8         |*
| Ndb_api_uk_op_count_session                | 0         |
*| Ndb_api_table_scan_count_session           | 1         |*
| Ndb_api_range_scan_count_session           | 0         |
| Ndb_api_pruned_scan_count_session          | 0         |
| Ndb_api_scan_batch_count_session           | 0         |
*| Ndb_api_read_row_count_session             | 3         |*
*| Ndb_api_trans_local_read_row_count_session | 2         |*
+--------------------------------------------+-----------+
18 rows in set (0.00 sec)
```

我们可以从这些结果中得出许多观察结果：

+   虽然我们创建了`t`，没有明确的主键，但在此过程中执行了 5 次主键操作（`Ndb_api_pk_op_count_session`的“之前”和“之后”值的差异，或者 6 减 1）。这反映了隐藏主键的创建，这是所有使用`NDB`存储引擎的表的特点。

+   通过比较`Ndb_api_wait_nanos_count_session`的连续值，我们可以看到实现`CREATE TABLE`语句的 NDB API 操作等待的时间要长得多（706871709 - 820705 = 706051004 纳秒，或大约 0.7 秒），以获取来自数据节点的响应，比执行`INSERT`的操作（707370418 - 706871709 = 498709 ns，或大约 0.0005 秒）要长。在**mysql**客户端中报告的执行时间与这些数字大致相关。

    在没有足够（纳秒）时间分辨率的平台上，由于执行非常快的 SQL 语句而导致`WaitNanosCount` NDB API 计数器值的微小变化，可能不总是在`Ndb_api_wait_nanos_count_session`、`Ndb_api_wait_nanos_count_slave`或`Ndb_api_wait_nanos_count`的值中可见。

+   `INSERT`语句增加了`ReadRowCount`和`TransLocalReadRowCount` NDB API 统计计数器，反映在`Ndb_api_read_row_count_session`和`Ndb_api_trans_local_read_row_count_session`的增加值。
