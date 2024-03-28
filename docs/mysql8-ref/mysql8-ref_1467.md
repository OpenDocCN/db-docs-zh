> 原文：[`dev.mysql.com/doc/refman/8.0/en/replication-features-variables.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-features-variables.html)

#### 19.5.1.39 复制和变量

使用`STATEMENT`模式时，系统变量在使用会话范围时不会被正确复制，除了以下变量：

+   `auto_increment_increment`

+   `auto_increment_offset`

+   `character_set_client`

+   `character_set_connection`

+   `character_set_database`

+   `character_set_server`

+   `collation_connection`

+   `collation_database`

+   `collation_server`

+   `foreign_key_checks`

+   `identity`

+   `last_insert_id`

+   `lc_time_names`

+   `pseudo_thread_id`

+   `sql_auto_is_null`

+   `time_zone`

+   `timestamp`

+   `unique_checks`

当使用`MIXED`模式时，前述列表中的变量在会话范围内使用时会导致从基于语句的日志记录切换到基于行的日志记录。请参阅 Section 7.4.4.3, “Mixed Binary Logging Format”。

`sql_mode`也会被复制，除了`NO_DIR_IN_CREATE`模式；复制品始终保留自己的`NO_DIR_IN_CREATE`值，而不管源上对其进行了何种更改。这对所有复制格式都适用。

然而，当**mysqlbinlog**解析`SET @@sql_mode = *`mode`*`语句时，包括`NO_DIR_IN_CREATE`在内的完整*`mode`*值将传递给接收服务器。因此，在使用`STATEMENT`模式时，此类语句的复制可能不安全。

`default_storage_engine`系统变量不会被复制，无论日志记录模式如何；这旨在促进不同存储引擎之间的复制。

`read_only` 系统变量不会被复制。此外，启用此变量在不同的 MySQL 版本中对临时表、表锁定和 `SET PASSWORD` 语句产生不同的影响。

`max_heap_table_size` 系统变量不会被复制。在源上增加此变量的值而在副本上未这样做最终可能导致在副本上执行 `MEMORY` 表的 `INSERT` 语句时出现 Table is full 错误，因为源上的表被允许比副本上的表更大。有关更多信息，请参见 Section 19.5.1.21, “Replication and MEMORY Tables”。

在基于语句的复制中，当在更新表的语句中使用会话变量时，会话变量不会被正确复制。例如，以下语句序列在源和副本上不会插入相同的数据：

```sql
SET max_join_size=1000;
INSERT INTO mytable VALUES(@@max_join_size);
```

这不适用于常见的顺序：

```sql
SET time_zone=...;
INSERT INTO mytable VALUES(CONVERT_TZ(..., ..., @@time_zone));
```

当使用基于行的复制时，会话变量的复制不是问题，此时会话变量始终安全地被复制。参见 Section 19.2.1, “Replication Formats”。

以下会话变量被写入二进制日志，并在解析二进制日志时由副本进行尊重，无论日志格式如何：

+   `sql_mode`

+   `foreign_key_checks`

+   `unique_checks`

+   `character_set_client`

+   `collation_connection`

+   `collation_database`

+   `collation_server`

+   `sql_auto_is_null`

重要

尽管与字符集和校对有关的会话变量被写入二进制日志，但不支持不同字符集之间的复制。

为了减少可能的混淆，我们建议您始终在源和副本上使用相同的设置来配置 `lower_case_table_names` 系统变量，特别是当您在区分大小写的文件系统上运行 MySQL 时。`lower_case_table_names` 设置只能在初始化服务器时配置。
