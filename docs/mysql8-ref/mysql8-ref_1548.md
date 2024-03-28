# 20.7.9 使用 Performance Schema 内存仪表化监控 Group Replication 内存使用情况

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-gr-memory-monitoring-ps-instruments.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-gr-memory-monitoring-ps-instruments.html)

20.7.9.1 启用或禁用 Group Replication 仪表化

20.7.9.2 示例查询

从 MySQL 8.0.30 开始，Performance Schema 提供了用于监控 Group Replication 内存使用情况的仪表化。要查看可用的 Group Replication 仪表，执行以下查询：

```sql
mysql> SELECT NAME,ENABLED FROM performance_schema.setup_instruments
       WHERE NAME LIKE 'memory/group_rpl/%';
+-------------------------------------------------------------------+---------+
| NAME                                                              | ENABLED |
+-------------------------------------------------------------------+---------+
| memory/group_rpl/write_set_encoded                                | YES     |
| memory/group_rpl/certification_data                               | YES     |
| memory/group_rpl/certification_data_gc                            | YES     |
| memory/group_rpl/certification_info                               | YES     |
| memory/group_rpl/transaction_data                                 | YES     |
| memory/group_rpl/sql_service_command_data                         | YES     |
| memory/group_rpl/mysql_thread_queued_task                         | YES     |
| memory/group_rpl/message_service_queue                            | YES     |
| memory/group_rpl/message_service_received_message                 | YES     |
| memory/group_rpl/group_member_info                                | YES     |
| memory/group_rpl/consistent_members_that_must_prepare_transaction | YES     |
| memory/group_rpl/consistent_transactions                          | YES     |
| memory/group_rpl/consistent_transactions_prepared                 | YES     |
| memory/group_rpl/consistent_transactions_waiting                  | YES     |
| memory/group_rpl/consistent_transactions_delayed_view_change      | YES     |
| memory/group_rpl/GCS_XCom::xcom_cache                             | YES     |
| memory/group_rpl/Gcs_message_data::m_buffer                       | YES     |
+-------------------------------------------------------------------+---------+
```

有关 Performance Schema 内存仪表化和事件的更多信息，请参阅 第 29.12.20.10 节，“内存摘要表”。

Performance Schema Group Replication 为 Group Replication 的内存分配提供仪表化。

`memory/group_rpl/` Performance Schema 仪表化在 8.0.30 中进行了更新，以扩展对 Group Replication 内存使用情况的监控。`memory/group_rpl/` 包含以下仪表：

+   `write_set_encoded`: 在广播到组成员之前对写入集进行编码分配的内存。

+   `Gcs_message_data::m_buffer`: 为发送到网络的事务数据负载分配的内存。

+   `certification_data`: 为传入事务的认证分配的内存。

+   `certification_data_gc`: 为每个成员发送的 GTID_EXECUTED 进行垃圾回收分配的内存。

+   `certification_info`: 为解决并发事务之间冲突分配的认证信息存储内存。

+   `transaction_data`: 为排队等待插件管道的传入事务分配的内存。

+   `message_service_received_message`: 为从 Group Replication 传递消息服务接收消息分配的内存。

+   `sql_service_command_data`: 为处理内部 SQL 服务命令队列分配的内存。

+   `mysql_thread_queued_task`: 当将 MySQL 线程相关任务添加到处理队列时分配的内存。

+   `message_service_queue`: 为 Group Replication 传递消息服务的排队消息分配的内存。

+   `GCS_XCom::xcom_cache`: 为组成员之间作为共识协议的一部分交换的消息和元数据分配的 XCOM 缓存内存。

+   `consistent_members_that_must_prepare_transaction`: 为保存为 Group Replication 事务一致性保证准备事务的成员列表分配的内存。

+   `consistent_transactions`: 为保存事务和必须为 Group Replication 事务一致性保证准备该事务的成员列表分配的内存。

+   `consistent_transactions_prepared`: 为保存为 Group Replication 事务一致性保证准备的事务信息列表分配的内存。

+   `consistent_transactions_waiting`：用于保存在处理具有`AFTER`和`BEFORE_AND_AFTER`一致性的准备事务之前的事务列表信息的内存分配。

+   `consistent_transactions_delayed_view_change`：用于保存由于准备一致性事务等待准备确认而延迟的视图更改事件（`view_change_log_event`）列表的内存分配。

+   `group_member_info`：用于保存组成员属性的内存分配。属性如主机名、端口、成员权重和角色等。

`memory/sql/` 分组中的以下工具也用于监视组复制内存：

+   `Log_event`：用于将事务数据编码为二进制日志格式的内存分配；这是组复制传输数据的相同格式。

+   `write_set_extraction`：在提交之前为事务生成的写入集分配的内存。

+   `Gtid_set::to_string`：用于存储 GTID 集合的字符串表示的内存分配。

+   `Gtid_set::Interval_chunk`：用于存储 GTID 对象的内存分配。
