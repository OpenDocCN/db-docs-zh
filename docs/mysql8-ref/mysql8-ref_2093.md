# 29.12.20 性能模式摘要表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-summary-tables.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-summary-tables.html)

29.12.20.1 等待事件摘要表

29.12.20.2 阶段摘要表

29.12.20.3 语句摘要表

29.12.20.4 语句直方图摘要表

29.12.20.5 事务摘要表

29.12.20.6 对象等待摘要表

29.12.20.7 文件 I/O 摘要表

29.12.20.8 表 I/O 和锁等待摘要表

29.12.20.9 套接字摘要表

29.12.20.10 内存摘要表

29.12.20.11 错误摘要表

29.12.20.12 状态变量摘要表

摘要表提供了随时间终止事件的聚合信息。此组中的表以不同方式总结事件数据。

每个摘要表都有分组列，用于确定如何对数据进行聚合，并包含聚合值的摘要列。通常，以类似方式总结事件的表具有类似的摘要列集，并且仅在用于确定如何聚合事件的分组列方面有所不同。

摘要表可以使用`TRUNCATE TABLE`进行截断。通常，效果是将摘要列重置为 0 或`NULL`，而不是删除行。这使您可以清除收集的值并重新开始聚合。例如，在进行运行时配置更改后，这可能很有用。个别摘要表部分中指出的截断行为的例外情况。

#### 等待事件摘要

**表 29.7 性能模式等待事件摘要表**

| 表名 | 描述 |
| --- | --- |
| `events_waits_summary_by_account_by_event_name` | 每个帐户和事件名称的等待事件 |
| `events_waits_summary_by_host_by_event_name` | 每个主机名和事件名称的等待事件 |
| `events_waits_summary_by_instance` | 每个实例的等待事件 |
| `events_waits_summary_by_thread_by_event_name` | 每个线程和事件名称的等待事件 |
| `events_waits_summary_by_user_by_event_name` | 按用户名称和事件名统计等待事件 |
| `events_waits_summary_global_by_event_name` | 按事件名统计等待事件 |

#### 阶段摘要

**表 29.8 性能模式阶段事件摘要表**

| 表名 | 描述 |
| --- | --- |
| `events_stages_summary_by_account_by_event_name` | 按账户和事件名统计阶段事件 |
| `events_stages_summary_by_host_by_event_name` | 按主机名和事件名统计阶段事件 |
| `events_stages_summary_by_thread_by_event_name` | 按线程和事件名统计阶段等待 |
| `events_stages_summary_by_user_by_event_name` | 按用户名称和事件名统计阶段事件 |
| `events_stages_summary_global_by_event_name` | 按事件名统计阶段等待 |

#### 语句摘要

**表 29.9 性能模式语句事件摘要表**

| 表名 | 描述 |
| --- | --- |
| `events_statements_histogram_by_digest` | 按模式和摘要值统计语句直方图 |
| `events_statements_histogram_global` | 全局汇总的语句直方图 |
| `events_statements_summary_by_account_by_event_name` | 按账户和事件名统计语句事件 |
| `events_statements_summary_by_digest` | 按模式和摘要值统计语句事件 |
| `events_statements_summary_by_host_by_event_name` | 按主机名和事件名统计语句事件 |
| `events_statements_summary_by_program` | 按存储程序统计语句事件 |
| `events_statements_summary_by_thread_by_event_name` | 按线程和事件名统计语句事件 |
| `events_statements_summary_by_user_by_event_name` | 按用户名称和事件名统计语句事件 |
| `events_statements_summary_global_by_event_name` | 按事件名称分类的语句事件 |
| `prepared_statements_instances` | 准备语句实例和统计信息 |
| 表名 | 描述 |

#### 事务摘要

**表 29.10 性能模式事务事件摘要表**

| 表名 | 描述 |
| --- | --- |
| `events_transactions_summary_by_account_by_event_name` | 按帐户和事件名称分类的事务事件 |
| `events_transactions_summary_by_host_by_event_name` | 按主机名和事件名称分类的事务事件 |
| `events_transactions_summary_by_thread_by_event_name` | 按线程和事件名称分类的事务事件 |
| `events_transactions_summary_by_user_by_event_name` | 按用户名和事件名称分类的事务事件 |
| `events_transactions_summary_global_by_event_name` | 按事件名称分类的事务事件 |

#### 对象等待摘要

**表 29.11 性能模式对象事件摘要表**

| 表名 | 描述 |
| --- | --- |
| `objects_summary_global_by_type` | 对象摘要 |

#### 文件 I/O 摘要

**表 29.12 性能模式文件 I/O 事件摘要表**

| 表名 | 描述 |
| --- | --- |
| `file_summary_by_event_name` | 按事件名称分类的文件事件 |
| `file_summary_by_instance` | 按文件实例分类的文件事件 |

#### 表 I/O 和锁等待摘要

**表 29.13 性能模式表 I/O 和锁等待事件摘要表**

| 表名 | 描述 |
| --- | --- |
| `table_io_waits_summary_by_index_usage` | 按索引分类的表 I/O 等待 |
| `table_io_waits_summary_by_table` | 按表分类的表 I/O 等待 |
| `table_lock_waits_summary_by_table` | 每个表的表锁等待 |

#### 套接字摘要

**表 29.14 性能模式套接字事件摘要表**

| 表名称 | 描述 |
| --- | --- |
| `socket_summary_by_event_name` | 每个事件名称的套接字等待和 I/O |
| `socket_summary_by_instance` | 每个实例的套接字等待和 I/O |

#### 内存摘要

**表 29.15 性能模式内存操作摘要表**

| 表名称 | 描述 |
| --- | --- |
| `memory_summary_by_account_by_event_name` | 每个帐户和事件名称的内存操作 |
| `memory_summary_by_host_by_event_name` | 每个主机和事件名称的内存操作 |
| `memory_summary_by_thread_by_event_name` | 每个线程和事件名称的内存操作 |
| `memory_summary_by_user_by_event_name` | 每个用户和事件名称的内存操作 |
| `memory_summary_global_by_event_name` | 每个事件名称的全局内存操作 |

#### 错误摘要

**表 29.16 性能模式错误摘要表**

| 表名称 | 描述 |
| --- | --- |
| `events_errors_summary_by_account_by_error` | 每个帐户和错误代码的错误 |
| `events_errors_summary_by_host_by_error` | 每个主机和错误代码的错误 |
| `events_errors_summary_by_thread_by_error` | 每个线程和错误代码的错误 |
| `events_errors_summary_by_user_by_error` | 每个用户和错误代码的错误 |
| `events_errors_summary_global_by_error` | 每个错误代码的错误 |

#### 状态变量摘要

**表 29.17 性能模式错误状态变量摘要表**

| 表名称 | 描述 |
| --- | --- |
| `status_by_account` | 每个帐户的会话状态变量 |
| `status_by_host` | 每个主机名称的会话状态变量 |
| `status_by_user` | 每个用户名称的会话状态变量 |
