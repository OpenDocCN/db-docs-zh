> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-gr-memory-monitoring-ps-sample-queries.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-gr-memory-monitoring-ps-sample-queries.html)

#### 20.7.9.2 示例查询

本节描述了使用工具和事件监视组复制内存使用情况的示例查询。这些查询从`memory_summary_global_by_event_name`表中检索数据。

内存数据可以查询单个事件，例如：

```sql
SELECT * FROM performance_schema.memory_summary_global_by_event_name
WHERE EVENT_NAME = 'memory/group_rpl/write_set_encoded'\G

*************************** 1\. row ***************************
                  EVENT_NAME: memory/group_rpl/write_set_encoded
                 COUNT_ALLOC: 1
                  COUNT_FREE: 0
   SUM_NUMBER_OF_BYTES_ALLOC: 45
    SUM_NUMBER_OF_BYTES_FREE: 0
              LOW_COUNT_USED: 0
          CURRENT_COUNT_USED: 1
             HIGH_COUNT_USED: 1
    LOW_NUMBER_OF_BYTES_USED: 0
CURRENT_NUMBER_OF_BYTES_USED: 45
   HIGH_NUMBER_OF_BYTES_USED: 45
```

更多关于列的信息，请参阅第 29.12.20.10 节，“内存摘要表”。

您还可以定义查询，对各种事件求和，以提供特定内存使用领域的概述。

下面描述了以下示例：

+   用于捕获事务的内存

+   用于广播事务的内存

+   组复制中的总内存使用量

+   认证中使用的内存

+   认证中使用的内存

+   复制管道中使用的内存

+   一致性中使用的内存

+   交付消息服务中使用的内存

+   用于广播和接收事务的内存

##### 记忆用于捕获事务

用于捕获用户事务的内存分配是`write_set_encoded`、`write_set_extraction`和`Log_event`事件值的总和。例如：

```sql
 mysql> SELECT * FROM (
                   SELECT
                     (CASE
                        WHEN EVENT_NAME LIKE 'memory/group_rpl/write_set_encoded'
                        THEN 'memory/group_rpl/memory_gr'
                        WHEN EVENT_NAME = 'memory/sql/write_set_extraction'
                        THEN 'memory/group_rpl/memory_gr'
                        WHEN EVENT_NAME = 'memory/sql/Log_event'
                        THEN 'memory/group_rpl/memory_gr'
                        ELSE 'memory_gr_rest'
                     END) AS EVENT_NAME, SUM(COUNT_ALLOC), SUM(COUNT_FREE),
                   SUM(SUM_NUMBER_OF_BYTES_ALLOC),
                   SUM(SUM_NUMBER_OF_BYTES_FREE), SUM(LOW_COUNT_USED),
                   SUM(CURRENT_COUNT_USED), SUM(HIGH_COUNT_USED),
                   SUM(LOW_NUMBER_OF_BYTES_USED), SUM(CURRENT_NUMBER_OF_BYTES_USED),
                   SUM(HIGH_NUMBER_OF_BYTES_USED)
                 FROM performance_schema.memory_summary_global_by_event_name
                 GROUP BY (CASE
                              WHEN EVENT_NAME LIKE 'memory/group_rpl/write_set_encoded'
                              THEN 'memory/group_rpl/memory_gr'
                              WHEN EVENT_NAME = 'memory/sql/write_set_extraction'
                              THEN 'memory/group_rpl/memory_gr'
                              WHEN EVENT_NAME = 'memory/sql/Log_event'
                              THEN 'memory/group_rpl/memory_gr'
                              ELSE 'memory_gr_rest'
                            END)
      ) f
      WHERE f.EVENT_NAME != 'memory_gr_rest'
      *************************** 1\. row ***************************
                             EVENT_NAME: memory/group_rpl/memory_gr
                       SUM(COUNT_ALLOC): 127
                        SUM(COUNT_FREE): 117
         SUM(SUM_NUMBER_OF_BYTES_ALLOC): 54808
          SUM(SUM_NUMBER_OF_BYTES_FREE): 52051
                    SUM(LOW_COUNT_USED): 0
                SUM(CURRENT_COUNT_USED): 10
                   SUM(HIGH_COUNT_USED): 35
          SUM(LOW_NUMBER_OF_BYTES_USED): 0
      SUM(CURRENT_NUMBER_OF_BYTES_USED): 2757
         SUM(HIGH_NUMBER_OF_BYTES_USED): 15630
```

##### 记忆用于广播事务

用于广播事务的内存分配是`Gcs_message_data::m_buffer`、`transaction_data`和`GCS_XCom::xcom_cache`事件值的总和。例如：

```sql
 mysql> SELECT * FROM (
                  SELECT
                    (CASE
                       WHEN EVENT_NAME =  'memory/group_rpl/Gcs_message_data::m_buffer'
                       THEN 'memory/group_rpl/memory_gr'
                       WHEN EVENT_NAME = 'memory/group_rpl/GCS_XCom::xcom_cache'
                       THEN 'memory/group_rpl/memory_gr'
                       WHEN EVENT_NAME = 'memory/group_rpl/transaction_data'
                       THEN 'memory/group_rpl/memory_gr'
                       ELSE 'memory_gr_rest'
                    END) AS EVENT_NAME, SUM(COUNT_ALLOC), SUM(COUNT_FREE),
                    SUM(SUM_NUMBER_OF_BYTES_ALLOC),
                    SUM(SUM_NUMBER_OF_BYTES_FREE), SUM(LOW_COUNT_USED),
                    SUM(CURRENT_COUNT_USED), SUM(HIGH_COUNT_USED),
                    SUM(LOW_NUMBER_OF_BYTES_USED), SUM(CURRENT_NUMBER_OF_BYTES_USED),
                    SUM(HIGH_NUMBER_OF_BYTES_USED)
                  FROM performance_schema.memory_summary_global_by_event_name
                  GROUP BY (CASE
                              WHEN EVENT_NAME =  'memory/group_rpl/Gcs_message_data::m_buffer'
                              THEN 'memory/group_rpl/memory_gr'
                              WHEN EVENT_NAME = 'memory/group_rpl/GCS_XCom::xcom_cache'
                              THEN 'memory/group_rpl/memory_gr'
                              WHEN EVENT_NAME = 'memory/group_rpl/transaction_data'
                              THEN 'memory/group_rpl/memory_gr'
                              ELSE 'memory_gr_rest'
                            END)
       ) f
       WHERE f.EVENT_NAME != 'memory_gr_rest'\G
       *************************** 1\. row ***************************
                              EVENT_NAME: memory/group_rpl/memory_gr
                        SUM(COUNT_ALLOC): 84
                         SUM(COUNT_FREE): 31
          SUM(SUM_NUMBER_OF_BYTES_ALLOC): 1072324
           SUM(SUM_NUMBER_OF_BYTES_FREE): 7149
                     SUM(LOW_COUNT_USED): 0
                 SUM(CURRENT_COUNT_USED): 53
                    SUM(HIGH_COUNT_USED): 59
           SUM(LOW_NUMBER_OF_BYTES_USED): 0
       SUM(CURRENT_NUMBER_OF_BYTES_USED): 1065175
          SUM(HIGH_NUMBER_OF_BYTES_USED): 1065809
```

##### 在组复制中使用的总内存

用于发送和接收事务、认证和所有其他主要进程的内存分配。通过查询`memory/group_rpl/`组的所有事件来计算。例如：

```sql
 mysql> SELECT * FROM (
                  SELECT
                    (CASE
                       WHEN EVENT_NAME LIKE 'memory/group_rpl/%'
                       THEN 'memory/group_rpl/memory_gr'
                       ELSE 'memory_gr_rest'
                     END) AS EVENT_NAME, SUM(COUNT_ALLOC), SUM(COUNT_FREE),
                     SUM(SUM_NUMBER_OF_BYTES_ALLOC),
                     SUM(SUM_NUMBER_OF_BYTES_FREE), SUM(LOW_COUNT_USED),
                     SUM(CURRENT_COUNT_USED), SUM(HIGH_COUNT_USED),
                     SUM(LOW_NUMBER_OF_BYTES_USED), SUM(CURRENT_NUMBER_OF_BYTES_USED),
                     SUM(HIGH_NUMBER_OF_BYTES_USED)
                  FROM performance_schema.memory_summary_global_by_event_name
                  GROUP BY (CASE
                              WHEN EVENT_NAME LIKE 'memory/group_rpl/%'
                              THEN 'memory/group_rpl/memory_gr'
                              ELSE 'memory_gr_rest'
                            END)
       ) f
       WHERE f.EVENT_NAME != 'memory_gr_rest'\G
       *************************** 1\. row ***************************
                              EVENT_NAME: memory/group_rpl/memory_gr
                        SUM(COUNT_ALLOC): 190
                         SUM(COUNT_FREE): 127
          SUM(SUM_NUMBER_OF_BYTES_ALLOC): 1096370
           SUM(SUM_NUMBER_OF_BYTES_FREE): 28675
                     SUM(LOW_COUNT_USED): 0
                 SUM(CURRENT_COUNT_USED): 63
                    SUM(HIGH_COUNT_USED): 77
           SUM(LOW_NUMBER_OF_BYTES_USED): 0
       SUM(CURRENT_NUMBER_OF_BYTES_USED): 1067695
          SUM(HIGH_NUMBER_OF_BYTES_USED): 1069255
```

##### 认证中使用的内存

认证过程中的内存分配是`certification_data`、`certification_data_gc`和`certification_info`事件值的总和。例如：

```sql
 mysql> SELECT * FROM (
                  SELECT
                    (CASE
                       WHEN EVENT_NAME = 'memory/group_rpl/certification_data'
                       THEN 'memory/group_rpl/certification'
                       WHEN EVENT_NAME = 'memory/group_rpl/certification_data_gc'
                       THEN 'memory/group_rpl/certification'
                       WHEN EVENT_NAME = 'memory/group_rpl/certification_info'
                       THEN 'memory/group_rpl/certification'
                       ELSE 'memory_gr_rest'
                     END) AS EVENT_NAME, SUM(COUNT_ALLOC), SUM(COUNT_FREE),
                     SUM(SUM_NUMBER_OF_BYTES_ALLOC),
                     SUM(SUM_NUMBER_OF_BYTES_FREE), SUM(LOW_COUNT_USED),
                     SUM(CURRENT_COUNT_USED), SUM(HIGH_COUNT_USED),
                     SUM(LOW_NUMBER_OF_BYTES_USED), SUM(CURRENT_NUMBER_OF_BYTES_USED),
                     SUM(HIGH_NUMBER_OF_BYTES_USED)
                  FROM performance_schema.memory_summary_global_by_event_name
                  GROUP BY (CASE
                              WHEN EVENT_NAME = 'memory/group_rpl/certification_data'
                              THEN 'memory/group_rpl/certification'
                              WHEN EVENT_NAME = 'memory/group_rpl/certification_data_gc'
                              THEN 'memory/group_rpl/certification'
                              WHEN EVENT_NAME = 'memory/group_rpl/certification_info'
                              THEN 'memory/group_rpl/certification'
                              ELSE 'memory_gr_rest'
                           END)
       ) f
       WHERE f.EVENT_NAME != 'memory_gr_rest'\G
       *************************** 1\. row ***************************
                              EVENT_NAME: memory/group_rpl/certification
                        SUM(COUNT_ALLOC): 80
                         SUM(COUNT_FREE): 80
          SUM(SUM_NUMBER_OF_BYTES_ALLOC): 9442
           SUM(SUM_NUMBER_OF_BYTES_FREE): 9442
                     SUM(LOW_COUNT_USED): 0
                 SUM(CURRENT_COUNT_USED): 0
                    SUM(HIGH_COUNT_USED): 66
           SUM(LOW_NUMBER_OF_BYTES_USED): 0
       SUM(CURRENT_NUMBER_OF_BYTES_USED): 0
          SUM(HIGH_NUMBER_OF_BYTES_USED): 6561
```

##### 复制管道中使用的内存

复制管道的内存分配是`certification_data`和`transaction_data`事件值的总和。例如：

```sql
 mysql> SELECT * FROM (
                  SELECT
                    (CASE
                       WHEN EVENT_NAME LIKE 'memory/group_rpl/certification_data'
                       THEN 'memory/group_rpl/pipeline'
                       WHEN EVENT_NAME LIKE 'memory/group_rpl/transaction_data'
                       THEN 'memory/group_rpl/pipeline'
                       ELSE 'memory_gr_rest'
                     END) AS EVENT_NAME, SUM(COUNT_ALLOC), SUM(COUNT_FREE),
                     SUM(SUM_NUMBER_OF_BYTES_ALLOC),
                     SUM(SUM_NUMBER_OF_BYTES_FREE), SUM(LOW_COUNT_USED),
                     SUM(CURRENT_COUNT_USED), SUM(HIGH_COUNT_USED),
                     SUM(LOW_NUMBER_OF_BYTES_USED), SUM(CURRENT_NUMBER_OF_BYTES_USED),
                     SUM(HIGH_NUMBER_OF_BYTES_USED)
                   FROM performance_schema.memory_summary_global_by_event_name
                   GROUP BY (CASE
                              WHEN EVENT_NAME LIKE 'memory/group_rpl/certification_data'
                              THEN 'memory/group_rpl/pipeline'
                              WHEN EVENT_NAME LIKE 'memory/group_rpl/transaction_data'
                              THEN 'memory/group_rpl/pipeline'
                              ELSE 'memory_gr_rest'
                            END)
       ) f
       WHERE f.EVENT_NAME != 'memory_gr_rest'\G
       *************************** 1\. row ***************************
                         EVENT_NAME: memory/group_rpl/pipeline
                        COUNT_ALLOC: 17
                         COUNT_FREE: 13
          SUM_NUMBER_OF_BYTES_ALLOC: 2483
           SUM_NUMBER_OF_BYTES_FREE: 1668
                     LOW_COUNT_USED: 0
                 CURRENT_COUNT_USED: 4
                    HIGH_COUNT_USED: 4
           LOW_NUMBER_OF_BYTES_USED: 0
       CURRENT_NUMBER_OF_BYTES_USED: 815
          HIGH_NUMBER_OF_BYTES_USED: 815
```

##### 一致性中使用的内存

事务一致性保证的内存分配是`consistent_members_that_must_prepare_transaction`、`consistent_transactions`、`consistent_transactions_prepared`、`consistent_transactions_waiting`和`consistent_transactions_delayed_view_change`事件值的总和。例如：

```sql
 mysql> SELECT * FROM (
                  SELECT
                    (CASE
                       WHEN EVENT_NAME = 'memory/group_rpl/consistent_members_that_must_prepare_transaction'
                       THEN 'memory/group_rpl/consistency'
                       WHEN EVENT_NAME = 'memory/group_rpl/consistent_transactions'
                       THEN 'memory/group_rpl/consistency'
                       WHEN EVENT_NAME = 'memory/group_rpl/consistent_transactions_prepared'
                       THEN 'memory/group_rpl/consistency'
                       WHEN EVENT_NAME = 'memory/group_rpl/consistent_transactions_waiting'
                       THEN 'memory/group_rpl/consistency'
                       WHEN EVENT_NAME = 'memory/group_rpl/consistent_transactions_delayed_view_change'
                       THEN 'memory/group_rpl/consistency'
                       ELSE 'memory_gr_rest'
                     END) AS EVENT_NAME, SUM(COUNT_ALLOC), SUM(COUNT_FREE),
                    SUM(SUM_NUMBER_OF_BYTES_ALLOC),
                    SUM(SUM_NUMBER_OF_BYTES_FREE), SUM(LOW_COUNT_USED),
                    SUM(CURRENT_COUNT_USED), SUM(HIGH_COUNT_USED),
                    SUM(LOW_NUMBER_OF_BYTES_USED), SUM(CURRENT_NUMBER_OF_BYTES_USED),
                    SUM(HIGH_NUMBER_OF_BYTES_USED)
                  FROM performance_schema.memory_summary_global_by_event_name
                  GROUP BY (CASE
                              WHEN EVENT_NAME = 'memory/group_rpl/consistent_members_that_must_prepare_transaction'
                              THEN 'memory/group_rpl/consistency'
                              WHEN EVENT_NAME = 'memory/group_rpl/consistent_transactions'
                              THEN 'memory/group_rpl/consistency'
                              WHEN EVENT_NAME = 'memory/group_rpl/consistent_transactions_prepared'
                              THEN 'memory/group_rpl/consistency'
                              WHEN EVENT_NAME = 'memory/group_rpl/consistent_transactions_waiting'
                              THEN 'memory/group_rpl/consistency'
                              WHEN EVENT_NAME = 'memory/group_rpl/consistent_transactions_delayed_view_change'
                              THEN 'memory/group_rpl/consistency'
                              ELSE 'memory_gr_rest'
                            END)
      ) f
      WHERE f.EVENT_NAME != 'memory_gr_rest'\G
      *************************** 1\. row ***************************
                        EVENT_NAME: memory/group_rpl/consistency
                       COUNT_ALLOC: 16
                        COUNT_FREE: 6
         SUM_NUMBER_OF_BYTES_ALLOC: 1464
          SUM_NUMBER_OF_BYTES_FREE: 528
                    LOW_COUNT_USED: 0
                CURRENT_COUNT_USED: 10
                   HIGH_COUNT_USED: 11
          LOW_NUMBER_OF_BYTES_USED: 0
      CURRENT_NUMBER_OF_BYTES_USED: 936
         HIGH_NUMBER_OF_BYTES_USED: 1024
```

##### 交付消息服务中使用的内存

注意

此仪表适用于接收的数据，而不是发送的数据。

组复制交付消息服务的内存分配是`message_service_received_message`和`message_service_queue`事件值的总和。例如：

```sql
 mysql> SELECT * FROM (
                  SELECT
                    (CASE
                       WHEN EVENT_NAME = 'memory/group_rpl/message_service_received_message'
                       THEN 'memory/group_rpl/message_service'
                       WHEN EVENT_NAME = 'memory/group_rpl/message_service_queue'
                       THEN 'memory/group_rpl/message_service'
                       ELSE 'memory_gr_rest'
                     END) AS EVENT_NAME, SUM(COUNT_ALLOC), SUM(COUNT_FREE),
                    SUM(SUM_NUMBER_OF_BYTES_ALLOC),
                    SUM(SUM_NUMBER_OF_BYTES_FREE), SUM(LOW_COUNT_USED),
                    SUM(CURRENT_COUNT_USED), SUM(HIGH_COUNT_USED),
                    SUM(LOW_NUMBER_OF_BYTES_USED), SUM(CURRENT_NUMBER_OF_BYTES_USED),
                    SUM(HIGH_NUMBER_OF_BYTES_USED)
                  FROM performance_schema.memory_summary_global_by_event_name
                  GROUP BY (CASE
                              WHEN EVENT_NAME = 'memory/group_rpl/message_service_received_message'
                              THEN 'memory/group_rpl/message_service'
                              WHEN EVENT_NAME = 'memory/group_rpl/message_service_queue'
                              THEN 'memory/group_rpl/message_service'
                              ELSE 'memory_gr_rest'
                            END)
       ) f
       WHERE f.EVENT_NAME != 'memory_gr_rest'\G
       *************************** 1\. row ***************************
                         EVENT_NAME: memory/group_rpl/message_service
                        COUNT_ALLOC: 2
                         COUNT_FREE: 0
          SUM_NUMBER_OF_BYTES_ALLOC: 1048664
           SUM_NUMBER_OF_BYTES_FREE: 0
                     LOW_COUNT_USED: 0
                 CURRENT_COUNT_USED: 2
                    HIGH_COUNT_USED: 2
           LOW_NUMBER_OF_BYTES_USED: 0
       CURRENT_NUMBER_OF_BYTES_USED: 1048664
          HIGH_NUMBER_OF_BYTES_USED: 1048664
```

##### 用于广播和接收事务的内存

用于广播和接收网络中事务的内存分配是`wGcs_message_data::m_buffer`和`GCS_XCom::xcom_cache`事件值的总和。例如：

```sql
mysql> SELECT * FROM (
         SELECT
           (CASE
              WHEN EVENT_NAME = 'memory/group_rpl/Gcs_message_data::m_buffer'
              THEN 'memory/group_rpl/memory_gr'
              WHEN EVENT_NAME = 'memory/group_rpl/GCS_XCom::xcom_cache'
              THEN 'memory/group_rpl/memory_gr'
              ELSE 'memory_gr_rest'
            END) AS EVENT_NAME, SUM(COUNT_ALLOC), SUM(COUNT_FREE),
           SUM(SUM_NUMBER_OF_BYTES_ALLOC),
           SUM(SUM_NUMBER_OF_BYTES_FREE), SUM(LOW_COUNT_USED),
           SUM(CURRENT_COUNT_USED), SUM(HIGH_COUNT_USED),
           SUM(LOW_NUMBER_OF_BYTES_USED), SUM(CURRENT_NUMBER_OF_BYTES_USED),
           SUM(HIGH_NUMBER_OF_BYTES_USED)
         FROM performance_schema.memory_summary_global_by_event_name
         GROUP BY (CASE
                     WHEN EVENT_NAME = 'memory/group_rpl/Gcs_message_data::m_buffer'
                     THEN 'memory/group_rpl/memory_gr'
                     WHEN EVENT_NAME = 'memory/group_rpl/GCS_XCom::xcom_cache'
                     THEN 'memory/group_rpl/memory_gr'
                     ELSE 'memory_gr_rest'
                   END)
       ) f
       WHERE f.EVENT_NAME != 'memory_gr_rest'\G
       *************************** 1\. row ***************************
                              EVENT_NAME: memory/group_rpl/memory_gr
                        SUM(COUNT_ALLOC): 73
                         SUM(COUNT_FREE): 20
          SUM(SUM_NUMBER_OF_BYTES_ALLOC): 1070845
           SUM(SUM_NUMBER_OF_BYTES_FREE): 5670
                     SUM(LOW_COUNT_USED): 0
                 SUM(CURRENT_COUNT_USED): 53
                    SUM(HIGH_COUNT_USED): 56
           SUM(LOW_NUMBER_OF_BYTES_USED): 0
       SUM(CURRENT_NUMBER_OF_BYTES_USED): 1065175
          SUM(HIGH_NUMBER_OF_BYTES_USED): 1065175
```
