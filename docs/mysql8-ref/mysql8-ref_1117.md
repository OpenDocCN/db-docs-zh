> 原文：[`dev.mysql.com/doc/refman/8.0/en/show-variables.html`](https://dev.mysql.com/doc/refman/8.0/en/show-variables.html)

#### 15.7.7.41 显示变量语句

```sql
SHOW [GLOBAL | SESSION] VARIABLES
    [LIKE '*pattern*' | WHERE *expr*]
```

`SHOW VARIABLES`显示 MySQL 系统变量的值（参见第 7.1.8 节，“服务器系统变量”）。此语句不需要任何特权。只需要连接到服务器的能力。

系统变量信息也可以从以下来源获取：

+   性能模式表。参见第 29.12.14 节，“性能模式系统变量表”。

+   **mysqladmin variables**命令。参见第 6.5.2 节，“mysqladmin — MySQL 服务器管理程序”。

对于`SHOW VARIABLES`，如果存在`LIKE`子句，则指示匹配哪些变量名。可以使用`WHERE`子句选择使用更一般条件的行，如第 28.8 节，“SHOW 语句的扩展”中讨论的。

`SHOW VARIABLES`接受可选的`GLOBAL`或`SESSION`变量范围修饰符：

+   使用`GLOBAL`修饰符，该语句显示全局系统变量值。这些值用于初始化 MySQL 新连接的相应会话变量。如果变量没有全局值，则不显示任何值。

+   使用`SESSION`修饰符，该语句显示当前连接中生效的系统变量值。如果变量没有会话值，则显示全局值。`LOCAL`是`SESSION`的同义词。

+   如果没有修饰符，则默认为`SESSION`。

每个系统变量的范围在第 7.1.8 节，“服务器系统变量”中列出。

`SHOW VARIABLES`受版本相关的显示宽度限制。对于值非常长且未完全显示的变量，可以使用`SELECT`作为解决方法。例如：

```sql
SELECT @@GLOBAL.innodb_data_file_path;
```

大多数系统变量可以在服务器启动时设置（只读变量如`version_comment`是例外）。许多可以通过`SET`语句在运行时更改。参见第 7.1.9 节，“使用系统变量”，以及第 15.7.6.1 节，“变量赋值的 SET 语法”。

这里显示了部分输出。名称和值的列表可能因您的服务器而异。第 7.1.8 节，“服务器系统变量”描述了每个变量的含义，第 7.1.1 节，“配置服务器”提供了有关调整它们的信息。

```sql
mysql> SHOW VARIABLES;
+--------------------------------------------+------------------------------+
| Variable_name                              | Value                        |
+--------------------------------------------+------------------------------+
| activate_all_roles_on_login                | OFF                          |
| auto_generate_certs                        | ON                           |
| auto_increment_increment                   | 1                            |
| auto_increment_offset                      | 1                            |
| autocommit                                 | ON                           |
| automatic_sp_privileges                    | ON                           |
| avoid_temporal_upgrade                     | OFF                          |
| back_log                                   | 151                          |
| basedir                                    | /usr/                        |
| big_tables                                 | OFF                          |
| bind_address                               | *                            |
| binlog_cache_size                          | 32768                        |
| binlog_checksum                            | CRC32                        |
| binlog_direct_non_transactional_updates    | OFF                          |
| binlog_error_action                        | ABORT_SERVER                 |
| binlog_expire_logs_seconds                 | 2592000                      |
| binlog_format                              | ROW                          |
| binlog_group_commit_sync_delay             | 0                            |
| binlog_group_commit_sync_no_delay_count    | 0                            |
| binlog_gtid_simple_recovery                | ON                           |
| binlog_max_flush_queue_time                | 0                            |
| binlog_order_commits                       | ON                           |
| binlog_row_image                           | FULL                         |
| binlog_row_metadata                        | MINIMAL                      |
| binlog_row_value_options                   |                              |
| binlog_rows_query_log_events               | OFF                          |
| binlog_stmt_cache_size                     | 32768                        |
| binlog_transaction_dependency_history_size | 25000                        |
| binlog_transaction_dependency_tracking     | COMMIT_ORDER                 |
| block_encryption_mode                      | aes-128-ecb                  |
| bulk_insert_buffer_size                    | 8388608                      |

...

| max_allowed_packet                         | 67108864                     |
| max_binlog_cache_size                      | 18446744073709547520         |
| max_binlog_size                            | 1073741824                   |
| max_binlog_stmt_cache_size                 | 18446744073709547520         |
| max_connect_errors                         | 100                          |
| max_connections                            | 151                          |
| max_delayed_threads                        | 20                           |
| max_digest_length                          | 1024                         |
| max_error_count                            | 1024                         |
| max_execution_time                         | 0                            |
| max_heap_table_size                        | 16777216                     |
| max_insert_delayed_threads                 | 20                           |
| max_join_size                              | 18446744073709551615         |

...

| thread_handling                            | one-thread-per-connection    |
| thread_stack                               | 286720                       |
| time_zone                                  | SYSTEM                       |
| timestamp                                  | 1530906638.765316            |
| tls_version                                | TLSv1.2,TLSv1.3              |
| tmp_table_size                             | 16777216                     |
| tmpdir                                     | /tmp                         |
| transaction_alloc_block_size               | 8192                         |
| transaction_allow_batching                 | OFF                          |
| transaction_isolation                      | REPEATABLE-READ              |
| transaction_prealloc_size                  | 4096                         |
| transaction_read_only                      | OFF                          |
| transaction_write_set_extraction           | XXHASH64                     |
| unique_checks                              | ON                           |
| updatable_views_with_limit                 | YES                          |
| version                                    | 8.0.36                       |
| version_comment                            | MySQL Community Server - GPL |
| version_compile_machine                    | x86_64                       |
| version_compile_os                         | Linux                        |
| version_compile_zlib                       | 1.2.11                       |
| wait_timeout                               | 28800                        |
| warning_count                              | 0                            |
| windowing_use_high_precision               | ON                           |
+--------------------------------------------+------------------------------+
```

使用`LIKE`子句，该语句仅显示那些名称与模式匹配的变量的行。要获取特定变量的行，请使用如下所示的`LIKE`子句：

```sql
SHOW VARIABLES LIKE 'max_join_size';
SHOW SESSION VARIABLES LIKE 'max_join_size';
```

要获取名称与模式匹配的变量列表，请在`LIKE`子句中使用`%`通配符：

```sql
SHOW VARIABLES LIKE '%size%';
SHOW GLOBAL VARIABLES LIKE '%size%';
```

通配符可以在要匹配的模式中的任何位置使用。严格来说，因为`_`是一个匹配任意单个字符的通配符，你应该将其转义为`\_`以确实匹配它。在实践中，这很少是必要的。
