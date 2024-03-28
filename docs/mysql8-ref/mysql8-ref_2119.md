# 29.15 性能模式系统变量

> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-system-variables.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-system-variables.html)

性能模式实现了几个提供配置信息的系统变量：

```sql
mysql> SHOW VARIABLES LIKE 'perf%';
+----------------------------------------------------------+-------+
| Variable_name                                            | Value |
+----------------------------------------------------------+-------+
| performance_schema                                       | ON    |
| performance_schema_accounts_size                         | -1    |
| performance_schema_digests_size                          | 10000 |
| performance_schema_events_stages_history_long_size       | 10000 |
| performance_schema_events_stages_history_size            | 10    |
| performance_schema_events_statements_history_long_size   | 10000 |
| performance_schema_events_statements_history_size        | 10    |
| performance_schema_events_transactions_history_long_size | 10000 |
| performance_schema_events_transactions_history_size      | 10    |
| performance_schema_events_waits_history_long_size        | 10000 |
| performance_schema_events_waits_history_size             | 10    |
| performance_schema_hosts_size                            | -1    |
| performance_schema_max_cond_classes                      | 80    |
| performance_schema_max_cond_instances                    | -1    |
| performance_schema_max_digest_length                     | 1024  |
| performance_schema_max_file_classes                      | 50    |
| performance_schema_max_file_handles                      | 32768 |
| performance_schema_max_file_instances                    | -1    |
| performance_schema_max_index_stat                        | -1    |
| performance_schema_max_memory_classes                    | 320   |
| performance_schema_max_metadata_locks                    | -1    |
| performance_schema_max_mutex_classes                     | 350   |
| performance_schema_max_mutex_instances                   | -1    |
| performance_schema_max_prepared_statements_instances     | -1    |
| performance_schema_max_program_instances                 | -1    |
| performance_schema_max_rwlock_classes                    | 40    |
| performance_schema_max_rwlock_instances                  | -1    |
| performance_schema_max_socket_classes                    | 10    |
| performance_schema_max_socket_instances                  | -1    |
| performance_schema_max_sql_text_length                   | 1024  |
| performance_schema_max_stage_classes                     | 150   |
| performance_schema_max_statement_classes                 | 192   |
| performance_schema_max_statement_stack                   | 10    |
| performance_schema_max_table_handles                     | -1    |
| performance_schema_max_table_instances                   | -1    |
| performance_schema_max_table_lock_stat                   | -1    |
| performance_schema_max_thread_classes                    | 50    |
| performance_schema_max_thread_instances                  | -1    |
| performance_schema_session_connect_attrs_size            | 512   |
| performance_schema_setup_actors_size                     | -1    |
| performance_schema_setup_objects_size                    | -1    |
| performance_schema_users_size                            | -1    |
+----------------------------------------------------------+-------+
```

可以在命令行或选项文件中设置性能模式系统变量，并且许多变量可以在运行时设置。请参见第 29.13 节，“性能模式选项和变量参考”。

如果未显式设置，性能模式会在服务器启动时自动调整几个参数的值。更多信息，请参见第 29.3 节，“性能模式启动配置”。

性能模式系统变量具有以下含义：

+   `performance_schema`

    | 命令行格式 | `--performance-schema[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `performance_schema` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `ON` |

    此变量的值为`ON`或`OFF`，表示性能模式是否已启用。默认情况下，该值为`ON`。在服务器启动时，您可以指定此变量不带值或带`ON`或 1 的值来启用它，或者带`OFF`或 0 的值来禁用它。

    即使性能模式被禁用，它仍会继续填充`global_variables`、`session_variables`、`global_status`和`session_status`表。这是必要的，以便从这些表中获取`SHOW VARIABLES`和`SHOW STATUS`语句的结果。性能模式在禁用时也会填充一些复制表。

+   `performance_schema_accounts_size`

    | 命令行格式 | `--performance-schema-accounts-size=#` |
    | --- | --- |
    | 系统变量 | `performance_schema_accounts_size` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `-1`（表示自动缩放；不要分配此文字值） |
    | 最小值 | `-1`（表示自动缩放；不要分配此文字值） |
    | 最大值 | `1048576` |

    `accounts`表中的行数。如果此变量为 0，则性能模式不会在`accounts`表中维护连接统计信息，也不会在`status_by_account`表中维护状态变量信息。

+   `performance_schema_digests_size`

    | 命令行格式 | `--performance-schema-digests-size=#` |
    | --- | --- |
    | 系统变量 | `performance_schema_digests_size` |
    | 范围 | 全局 |
    | Dynamic | No |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | Default Value | `-1`（表示自动调整大小；不要分配此文字值） |
    | 最小值 | `-1`（表示自动缩放；不要分配此文字值） |
    | 最大值 | `1048576` |

    `events_statements_summary_by_digest`表中的最大行数。如果超过此最大值，以至于无法对摘要进行检测，则性能模式会增加`Performance_schema_digest_lost`状态变量。

    有关语句摘要的更多信息，请参见第 29.10 节，“性能模式语句摘要和抽样”。

+   `performance_schema_error_size`

    | 命令行格式 | `--performance-schema-error-size=#` |
    | --- | --- |
    | 系统变量 | `performance_schema_error_size` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | Default Value | `服务器错误代码的数量` |
    | 最小值 | `0` |
    | 最大值 | `1048576` |

    被检测的服务器错误代码的数量。默认值是实际的服务器错误代码数量。虽然该值可以设置为从 0 到其最大值的任何值，但预期用法是将其设置为默认值（以检测所有错误）或 0（不检测任何错误）。

    错误信息在摘要表中汇总；请参阅第 29.12.20.11 节，“错误摘要表”。如果发生未被检测的错误，该事件的信息将被汇总到每个摘要表的`NULL`行中；也就是说，到具有`ERROR_NUMBER=0`、`ERROR_NAME=NULL`和`SQLSTATE=NULL`的行。

+   `performance_schema_events_stages_history_long_size`

    | 命令行格式 | `--performance-schema-events-stages-history-long-size=#` |
    | --- | --- |
    | 系统变量 | `performance_schema_events_stages_history_long_size` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `-1`（表示自动调整大小；不要分配此文字值） |
    | 最小值 | `-1`（表示自动缩放；不要分配此文字值） |
    | 最大值 | `1048576` |

    `events_stages_history_long` 表中的行数。

+   `performance_schema_events_stages_history_size`

    | 命令行格式 | `--performance-schema-events-stages-history-size=#` |
    | --- | --- |
    | 系统变量 | `performance_schema_events_stages_history_size` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `-1`（表示自动调整大小；不要分配此文字值） |
    | 最小值 | `-1`（表示自动缩放；不要分配此文字值） |
    | 最大值 | `1024` |

    每个线程在`events_stages_history`表中的行数。

+   `performance_schema_events_statements_history_long_size`

    | 命令行格式 | `--performance-schema-events-statements-history-long-size=#` |
    | --- | --- |
    | 系统变量 | `performance_schema_events_statements_history_long_size` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `-1`（表示自动调整大小；不要分配此文字值） |
    | 最小值 | `-1`（表示自动缩放；不要分配此文字值） |
    | 最大值 | `1048576` |

    `events_statements_history_long` 表中的行数��

+   `performance_schema_events_statements_history_size`

    | 命令行格式 | `--performance-schema-events-statements-history-size=#` |
    | --- | --- |
    | 系统变量 | `performance_schema_events_statements_history_size` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `-1`（表示自动调整大小；不要分配此文字值） |
    | 最小值 | `-1`（表示自动缩放；不要分配此文字值） |
    | 最大值 | `1024` |

    `events_statements_history` 表中每个线程的行数。

+   `performance_schema_events_transactions_history_long_size`

    | 命令行格式 | `--performance-schema-events-transactions-history-long-size=#` |
    | --- | --- |
    | 系统变量 | `performance_schema_events_transactions_history_long_size` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `-1`（表示自动调整大小；不要分配此文字值） |
    | 最小值 | `-1`（表示自动缩放；不要分配此文字值） |
    | 最大值 | `1048576` |

    `events_transactions_history_long` 表中的行数。

+   `performance_schema_events_transactions_history_size`

    | 命令行格式 | `--performance-schema-events-transactions-history-size=#` |
    | --- | --- |
    | 系统变量 | `performance_schema_events_transactions_history_size` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `-1`（表示自动调整大小；不要分配此文字值） |
    | 最小值 | `-1`（表示自动缩放；不要分配此文字值） |
    | 最大值 | `1024` |

    在`events_transactions_history`表中每个线程的行数。

+   `performance_schema_events_waits_history_long_size`

    | 命令行格式 | `--performance-schema-events-waits-history-long-size=#` |
    | --- | --- |
    | 系统变量 | `performance_schema_events_waits_history_long_size` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `-1`（表示自动调整大小；不要分配此文字值） |
    | 最小值 | `-1`（表示自动缩放；不要分配此文字值） |
    | 最大值 | `1048576` |

    在`events_waits_history_long`表中的行数。

+   `performance_schema_events_waits_history_size`

    | 命令行格式 | `--performance-schema-events-waits-history-size=#` |
    | --- | --- |
    | 系统变量 | `performance_schema_events_waits_history_size` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `-1`（表示自动调整大小；不要分配此文字值） |
    | 最小值 | `-1`（表示自动缩放；不要分配此文字值） |
    | 最大值 | `1024` |

    在`events_waits_history`表中每个线程的行数。

+   `performance_schema_hosts_size`

    | 命令行格式 | `--performance-schema-hosts-size=#` |
    | --- | --- |
    | 系统变量 | `performance_schema_hosts_size` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `-1`（表示自动缩放；不要分配此文字值） |
    | 最小值 | `-1`（表示自动缩放；不要分配此文字值） |
    | 最大值 | `1048576` |

    `hosts` 表中的行数。如果此变量为 0，则性能模式不会在 `hosts` 表中维护连接统计信息，也不会在 `status_by_host` 表中维护状态变量信息。

+   `performance_schema_max_cond_classes`

    | 命令行格式 | `--performance-schema-max-cond-classes=#` |
    | --- | --- |
    | 系统变量 | `performance_schema_max_cond_classes` |
    | 作用域 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值（≥ 8.0.27） | `150` |
    | 默认值（≥ 8.0.13, ≤ 8.0.26） | `100` |
    | 默认值（≤ 8.0.12） | `80` |
    | 最小值 | `0` |
    | 最大值（≥ 8.0.12） | `1024` |
    | 最大值（8.0.11） | `256` |

    条件仪器的最大数量。有关如何设置和使用此变量的信息，请参见 第 29.7 节，“性能模式状态监控”。

+   `performance_schema_max_cond_instances`

    | 命令行格式 | `--performance-schema-max-cond-instances=#` |
    | --- | --- |
    | 系统变量 | `performance_schema_max_cond_instances` |
    | 作用域 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `-1`（表示自动缩放；不要赋予此字面值） |
    | 最小值 | `-1`（表示自动缩放；不要赋予此字面��） |
    | 最大值 | `1048576` |

    仪器化条件对象的最大数量。有关如何设置和使用此变量的信息，请参见 第 29.7 节，“性能模式状态监控”。

+   `performance_schema_max_digest_length`

    | 命令行格式 | `--performance-schema-max-digest-length=#` |
    | --- | --- |
    | 系统变量 | `performance_schema_max_digest_length` |
    | 作用域 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `1024` |
    | 最小值 | `0` |
    | 最大值 | `1048576` |
    | 单位 | 字节 |

    在性能模式中，每个语句用于计算规范语句摘要值的内存保留的最大字节数。此变量与`max_digest_length`有关；请参阅第 7.1.8 节，“服务器系统变量”中对该变量的描述。

    有关语句摘要的更多信息，包括有关内存使用的考虑，请参阅第 29.10 节，“性能模式语句摘要和采样”。

+   `performance_schema_max_digest_sample_age`

    | 命令行格式 | `--performance-schema-max-digest-sample-age=#` |
    | --- | --- |
    | 系统变量 | `performance_schema_max_digest_sample_age` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `60` |
    | 最小值 | `0` |
    | 最大值 | `1048576` |
    | 单位 | 秒 |

    此变量影响`events_statements_summary_by_digest`表的语句采样。当插入新的表行时，生成行摘要值的语句将存储为与摘要相关联的当前样本语句。此后，当服务器看到具有相同摘要值的其他语句时，它会确定是否使用新语句替换当前样本语句（即是否重新采样）。重新采样策略基于当前样本语句和新语句的比较等待时间，以及可选的当前样本语句的年龄：

    +   基于等待时间的重新采样：如果新语句的等待时间大于当前样本语句的等待时间，则新语句将成为当前样本语句。

    +   基于年龄的重新采样：如果`performance_schema_max_digest_sample_age`系统变量的值大于零，并且当前样本语句的年龄超过该时间（以秒为单位），则当前语句被视为“太旧”，新语句将替换它。即使新语句的等待时间小于当前样本语句的等待时间，也会发生这种情况。

    有关语句采样的信息，请参阅第 29.10 节，“性能模式语句摘要和采样”。

+   `performance_schema_max_file_classes`

    | 命令行格式 | `--performance-schema-max-file-classes=#` |
    | --- | --- |
    | 系统变量 | `performance_schema_max_file_classes` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `80` |
    | 最小值 | `0` |
    | 最大值 (≥ 8.0.12) | `1024` |
    | 最大值 (8.0.11) | `256` |

    文件工具的最大数量。有关如何设置和使用此变量的信息，请参见 第 29.7 节，“性能模式状态监控”。

+   `performance_schema_max_file_handles` 

    | 命令行格式 | `--performance-schema-max-file-handles=#` |
    | --- | --- |
    | 系统变量 | `performance_schema_max_file_handles` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `32768` |
    | 最小值 | `0` |
    | 最大值 | `1048576` |

    打开文件对象的最大数量。有关如何设置和使用此变量的信息，请参见 第 29.7 节，“性能模式状态监控”。

    `performance_schema_max_file_handles` 的值应大于 `open_files_limit` 的值：`open_files_limit` 影响服务器支持的最大打开文件句柄数，而 `performance_schema_max_file_handles` 影响可以被检测的这些文件句柄的数量。

+   `performance_schema_max_file_instances` 

    | 命令行格式 | `--performance-schema-max-file-instances=#` |
    | --- | --- |
    | 系统变量 | `performance_schema_max_file_instances` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `-1`（表示自动缩放；不要分配此文字值） |
    | 最小值 | `-1`（表示自动缩放；不要分配此文字值） |
    | 最大值 | `1048576` |

    文件对象的受检测最大数量。有关如何设置和使用此变量的信息，请参见第 29.7 节，“性能模式状态监控”。

+   `performance_schema_max_index_stat`

    | 命令行格式 | `--performance-schema-max-index-stat=#` |
    | --- | --- |
    | 系统变量 | `performance_schema_max_index_stat` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `-1`（表示自动调整大小；不要分配此字面值） |
    | 最小值 | `-1`（表示自动缩放；不要分配此字面值） |
    | 最大值 | `1048576` |

    Performance Schema 维护统计信息的最大索引数。如果超过此最大值，导致索引统计丢失，Performance Schema 会增加`Performance_schema_index_stat_lost`状态变量。默认值是使用`performance_schema_max_table_instances`的值进行自动调整。

+   `performance_schema_max_memory_classes`

    | 命令行格式 | `--performance-schema-max-memory-classes=#` |
    | --- | --- |
    | 系统变量 | `performance_schema_max_memory_classes` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `450` |
    | 最小值 | `0` |
    | 最大值 | `1024` |

    内存工具的最大数量。有关如何设置和使用此变量的信息，请参见第 29.7 节，“性能模式状态监控”。

+   `performance_schema_max_metadata_locks`

    | 命令行格式 | `--performance-schema-max-metadata-locks=#` |
    | --- | --- |
    | 系统变量 | `performance_schema_max_metadata_locks` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `-1`（表示自动缩放；不要分配此字面值） |
    | 最小值 | `-1`（表示自动缩放；不要分配此字面值） |
    | 最大值 | `10485760` |

    元数据锁仪器的最大数量。此值控制`metadata_locks`表的大小。如果超过此最大值，以至于无法对元数据锁进行仪器化，则性能模式会增加`Performance_schema_metadata_lock_lost`状态变量。

+   `performance_schema_max_mutex_classes`

    | 命令行格式 | `--performance-schema-max-mutex-classes=#` |
    | --- | --- |
    | 系统变量 | `performance_schema_max_mutex_classes` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 (≥ 8.0.27) | `350` |
    | 默认值 (≥ 8.0.12, ≤ 8.0.26) | `300` |
    | 默认值 (8.0.11) | `250` |
    | 最小值 | `0` |
    | 最大值 (≥ 8.0.12) | `1024` |
    | 最大值 (8.0.11) | `256` |

    互斥锁仪器的最大数量。有关如何设置和使用此变量的信息，请参见第 29.7 节，“性能模式状态监控”。

+   `performance_schema_max_mutex_instances`

    | 命令行格式 | `--performance-schema-max-mutex-instances=#` |
    | --- | --- |
    | 系统变量 | `performance_schema_max_mutex_instances` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `-1` (表示自动缩放；不要分配此文字值) |
    | 最小值 | `-1` (表示自动缩放；不要分配此文字值) |
    | 最大值 | `104857600` |

    仪器化互斥对象的最大数量。有关如何设置和使用此变量的信息，请参见第 29.7 节，“性能模式状态监控”。

+   `performance_schema_max_prepared_statements_instances`

    | 命令行格式 | `--performance-schema-max-prepared-statements-instances=#` |
    | --- | --- |
    | 系统变量 | `performance_schema_max_prepared_statements_instances` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `-1` (表示自动缩放；不要分配此文字值) |
    | 最小值 | `-1` (表示自动缩放；不要分配此文字值) |
    | 最大值 | `4194304` |

    `prepared_statements_instances` 表中的最大行数。如果超过此最大值，使得无法对准备语句进行仪器化，性能模式会增加 `Performance_schema_prepared_statements_lost` 状态变量。有关如何设置和使用此变量的信息，请参见第 29.7 节，“性能模式状态监控”。

    此变量的默认值是基于 `max_prepared_stmt_count` 系统变量的值自动调整的。

+   `performance_schema_max_rwlock_classes`

    | 命令行格式 | `--performance-schema-max-rwlock-classes=#` |
    | --- | --- |
    | 系统变量 | `performance_schema_max_rwlock_classes` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 (≥ 8.0.12) | `100` |
    | 默认值 (8.0.11) | `60` |
    | 最小值 | `0` |
    | 最大值 (≥ 8.0.12) | `1024` |
    | 最大值 (8.0.11) | `256` |

    最大的 rwlock 仪器数量。有关如何设置和使用此变量的信息，请参见第 29.7 节，“性能模式状态监控”。

+   `performance_schema_max_program_instances`

    | 命令行格式 | `--performance-schema-max-program-instances=#` |
    | --- | --- |
    | 系统变量 | `performance_schema_max_program_instances` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `-1` (表示自动缩放；不要分配此文字值) |
    | 最小值 | `-1` (表示自动缩放；不要分配此文字值) |
    | 最大值 | `1048576` |

    存储程序的最大数量，性能模式维护统计信息。如果超过此最大值，性能模式会增加`Performance_schema_program_lost`状态变量。有关如何设置和使用此变量的信息，请参见第 29.7 节，“性能模式状态监控”。

+   `performance_schema_max_rwlock_instances`

    | 命令行格式 | `--performance-schema-max-rwlock-instances=#` |
    | --- | --- |
    | 系统变量 | `performance_schema_max_rwlock_instances` |
    | 作用范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `-1`（表示自动调整大小；不要分配此字面值） |
    | 最小值 | `-1`（表示自动调整大小；不要分配此字面值） |
    | 最大值 | `104857600` |

    最大锁对象数量。有关如何设置和使用此变量的信息，请参见第 29.7 节，“性能模式状态监控”。

+   `performance_schema_max_socket_classes`

    | 命令行格式 | `--performance-schema-max-socket-classes=#` |
    | --- | --- |
    | 系统变量 | `performance_schema_max_socket_classes` |
    | 作用范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `10` |
    | 最小值 | `0` |
    | 最大值（≥ 8.0.12） | `1024` |
    | 最大值（8.0.11） | `256` |

    最大套接字工具数量。有关如何设置和使用此变量的信息，请参见第 29.7 节，“性能模式状态监控”。

+   `performance_schema_max_socket_instances`

    | 命令行格式 | `--performance-schema-max-socket-instances=#` |
    | --- | --- |
    | 系统变量 | `performance_schema_max_socket_instances` |
    | 作用范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `-1`（表示自动调整大小；不要分配此字面值） |
    | 最小值 | `-1`（表示自动缩放；不要赋予此文字值） |
    | 最大值 | `1048576` |

    受监视的套接字对象的最大数量。有关如何设置和使用此变量的信息，请参见 第 29.7 节“性能模式状态监视”。

+   `performance_schema_max_sql_text_length`

    | 命令行格式 | `--performance-schema-max-sql-text-length=#` |
    | --- | --- |
    | 系统变量 | `performance_schema_max_sql_text_length` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `1024` |
    | 最小值 | `0` |
    | 最大值 | `1048576` |
    | 单位 | 字节 |

    用于存储 SQL 语句的最大字节数。该值适用于以下列所需的存储空间：

    +   `events_statements_current`、`events_statements_history` 和 `events_statements_history_long` 语句事件表的 `SQL_TEXT` 列。

    +   `events_statements_summary_by_digest` 摘要表的 `QUERY_SAMPLE_TEXT` 列。

    超过 `performance_schema_max_sql_text_length` 的任何字节都将被丢弃，并不会出现在列中。只有在那么多初始字节之后有所不同的语句在列中是无法区分的。

    减小 `performance_schema_max_sql_text_length` 值会减少内存使用，但会导致更多语句在仅在结尾处���所不同时无法区分。增加该值会增加内存使用，但允许区分更长的语句。

+   `performance_schema_max_stage_classes`

    | 命令行格式 | `--performance-schema-max-stage-classes=#` |
    | --- | --- |
    | 系统变量 | `performance_schema_max_stage_classes` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值（≥ 8.0.13） | `175` |
    | 默认值（≤ 8.0.12） | `150` |
    | 最小值 | `0` |
    | 最大值（≥ 8.0.12） | `1024` |
    | 最大值（8.0.11） | `256` |

    阶段仪器的最大数量。有关如何设置和使用此变量的信息，请参见第 29.7 节“性能模式状态监控”。

+   `performance_schema_max_statement_classes`

    | 命令行格式 | `--performance-schema-max-statement-classes=#` |
    | --- | --- |
    | 系统变量 | `performance_schema_max_statement_classes` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 最小值 | `0` |
    | 最大值 | `256` |

    语句仪器的最大数量。有关如何设置和使用此变量的信息，请参见第 29.7 节“性能模式状态监控”。

    默认值是在服务器构建时根据客户端/服务器协议中的命令数量和服务器支持的 SQL 语句类型数量计算的。

    除非将其设置为 0 以禁用所有语句仪表化并保存与之关联的所有内存，否则不应更改此变量。将变量设置为非默认值没有任何好处；特别是，大于默认值的值会导致分配更多的内存。

+   `performance_schema_max_statement_stack`

    | 命令行格式 | `--performance-schema-max-statement-stack=#` |
    | --- | --- |
    | 系统变量 | `performance_schema_max_statement_stack` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `10` |
    | 最小值 | `1` |
    | 最大值 | `256` |

    Performance Schema 维护统计信息的最大嵌套存储过程调用深度。当超过此最大值时，Performance Schema 会为执行的每个存储过程语句递增`Performance_schema_nested_statement_lost`状态变量。

+   `performance_schema_max_table_handles`

    | 命令行格式 | `--performance-schema-max-table-handles=#` |
    | --- | --- |
    | 系统变量 | `performance_schema_max_table_handles` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `-1`（表示自动缩放；不要分配此文字值） |
    | 最小值 | `-1`（表示自动缩放；不要分配此文字值） |
    | 最大值 | `1048576` |

    最大打开表对象数量。此值控制`table_handles`表的大小。如果超过此最大值，导致无法对表句柄进行仪表化，性能模式会增加`Performance_schema_table_handles_lost`状态变量。有关如何设置和使用此变量的信息，请参见第 29.7 节，“性能模式状态监控”。

+   `performance_schema_max_table_instances`

    | 命令行格式 | `--performance-schema-max-table-instances=#` |
    | --- | --- |
    | 系统变量 | `performance_schema_max_table_instances` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `-1`（表示自动缩放；不要分配此文字值） |
    | 最小值 | `-1`（表示自动缩放；不要分配此文字值） |
    | 最大值 | `1048576` |

    最大仪表化表对象数量。有关如何设置和使用此变量的信息，请参见第 29.7 节，“性能模式状态监控”。

+   `performance_schema_max_table_lock_stat`

    | 命令行格式 | `--performance-schema-max-table-lock-stat=#` |
    | --- | --- |
    | 系统变量 | `performance_schema_max_table_lock_stat` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `-1`（表示自动调整大小；不要分配此文字值） |
    | 最小值 | `-1`（表示自动缩放；不要分配此文字值） |
    | 最大值 | `1048576` |

    性能模式维护锁统计信息的表的最大数量。如果超过了这个最大值，导致表锁统计信息丢失，性能模式会增加`Performance_schema_table_lock_stat_lost`状态变量。

+   `performance_schema_max_thread_classes`

    | 命令行格式 | `--performance-schema-max-thread-classes=#` |
    | --- | --- |
    | 系统变量 | `performance_schema_max_thread_classes` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `100` |
    | 最小值 | `0` |
    | 最大值（≥ 8.0.12） | `1024` |
    | 最大值（8.0.11） | `256` |

    线程仪器的最大数量。有关如何设置和使用此变量的信息，请参见第 29.7 节“性能模式状态监控”。

+   `performance_schema_max_thread_instances`

    | 命令行格式 | `--performance-schema-max-thread-instances=#` |
    | --- | --- |
    | 系统变量 | `performance_schema_max_thread_instances` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `-1`（表示自动调整大小；不要分配此文字值） |
    | 最小值 | `-1`（表示自动缩放；不要分配此文字值） |
    | 最大值 | `1048576` |

    最大仪器化线程对象的数量。该值控制了`threads`表的大小。如果超过了这个最大值，导致无法对线程进行仪器化，性能模式会增加`Performance_schema_thread_instances_lost`状态变量。有关如何设置和使用此变量的信息，请参见第 29.7 节“性能模式状态监控”。

    `max_connections`系统变量影响服务器中可以运行的线程数量。`performance_schema_max_thread_instances`影响可以对这些运行线程进行仪器化的数量。

    `variables_by_thread` 和 `status_by_thread` 表仅包含关于前台线程的系统和状态变量信息。如果性能模式未对所有线程进行仪表化，此表将缺少一些行。在这种情况下，`Performance_schema_thread_instances_lost` 状态变量大于零。

+   `performance_schema_session_connect_attrs_size`

    | 命令行格式 | `--performance-schema-session-connect-attrs-size=#` |
    | --- | --- |
    | 系统变量 | `performance_schema_session_connect_attrs_size` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `-1`（表示自动调整大小；不要分配此文字值） |
    | 最小值 | `-1`（表示自动调整大小；不要分配此文字值） |
    | 最大值 | `1048576` |
    | 单位 | 字节 |

    每个线程预分配的内存量，用于保存连接属性键值对。如果客户端发送的连接属性数据的总大小大于此数量，则性能模式会截断属性数据，增加 `Performance_schema_session_connect_attrs_lost` 状态变量，并在错误日志中写入一条消息指示发生了截断，如果 `log_error_verbosity` 系统变量大于 1。如果属性缓冲区有足够的空间，还会向会话属性添加一个 `_truncated` 属性，其中包含丢失的字节数，这样性能模式就可以在连接属性表中公��每个连接的截断信息。这样就可以在不必检查错误日志的情况下检查这些信息。

    `performance_schema_session_connect_attrs_size` 的默认值在服务器启动时自动调整大小。这个值可能很小，所以如果发生截断（`Performance_schema_session_connect_attrs_lost` 变为非零），您可能希望将 `performance_schema_session_connect_attrs_size` 明确设置为较大的值。

    尽管`performance_schema_session_connect_attrs_size`的最大允许值为 1MB，但有效最大值为 64KB，因为服务器对其接受的连接属性数据的总大小施加了 64KB 的限制。如果客户端尝试发送超过 64KB 的属性数据，服务器将拒绝连接。更多信息，请参见第 29.12.9 节，“性能模式连接属性表”。

+   `performance_schema_setup_actors_size`

    | 命令行格式 | `--performance-schema-setup-actors-size=#` |
    | --- | --- |
    | 系统变量 | `performance_schema_setup_actors_size` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `-1`（表示自动缩放；不要分配此文字值） |
    | 最小值 | `-1`（表示自动调整大小；不要分配此文字值） |
    | 最大值 | `1048576` |

    `setup_actors`表中的行数。

+   `performance_schema_setup_objects_size`

    | 命令行格式 | `--performance-schema-setup-objects-size=#` |
    | --- | --- |
    | 系统变量 | `performance_schema_setup_objects_size` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `-1`（表示自动缩放；不要分配此文字值） |
    | 最小值 | `-1`（表示自动缩放；不要分配此文字值） |
    | 最大值 | `1048576` |

    `setup_objects`表中的行数。

+   `performance_schema_show_processlist`

    | 命令行格式 | `--performance-schema-show-processlist[={OFF&#124;ON}]` |
    | --- | --- |
    | 引入版本 | 8.0.22 |
    | 已弃用 | 8.0.35 |
    | 系统变量 | `performance_schema_show_processlist` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔 |
    | 默认值 | `OFF` |

    `SHOW PROCESSLIST`语句通过从所有活动线程收集线程数据提供进程信息。`performance_schema_show_processlist`变量确定要使用哪种`SHOW PROCESSLIST`实现：

    +   默认实现在持有全局互斥锁的同时，从线程管理器中遍历活动线程。这对于繁忙系统会产生负面的性能后果。

    +   另一种`SHOW PROCESSLIST`实现基于性能模式`processlist`表。该实现从性能模式而不是线程管理器查询活动线程数据，并且不需要互斥锁。

    要启用替代实现，请启用`performance_schema_show_processlist`系统变量。为确保默认和替代实现提供相同的信息，必须满足某些配置要求；参见第 29.12.21.7 节，“The processlist Table”。

+   `performance_schema_users_size`

    | 命令行格式 | `--performance-schema-users-size=#` |
    | --- | --- |
    | 系统变量 | `performance_schema_users_size`` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `-1`（表示自动缩放；不要分配此文字值） |
    | 最小值 | `-1`（表示自动缩放；不要分配此文字值） |
    | 最大值 | `1048576` |

    `users`表中的行数。如果此变量为 0，则性能模式不会在`users`表中维护连接统计信息，也不会在`status_by_user`表中维护状态变量信息。
