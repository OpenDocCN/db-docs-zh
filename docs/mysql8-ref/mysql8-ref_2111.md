> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-log-status-table.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-log-status-table.html)

#### 29.12.21.5 `log_status`表

`log_status`表提供的信息使在线备份工具能够复制所需的日志文件，而不会在复制过程中锁定这些资源。

当查询`log_status`表时，服务器会阻止记录和相关的管理更改，只需足够的时间来填充表，然后释放资源。`log_status`表通知在线备份应该复制到源二进制日志和`gtid_executed`记录的哪个点，以及每个复制通道的中继日志。它还为各个存储引擎提供相关信息，例如`InnoDB`存储引擎的最后日志序列号（LSN）和最后一个检查点的 LSN。

`log_status`表具有以下列：

+   `SERVER_UUID`

    该服务器实例的服务器 UUID。这是只读系统变量`server_uuid`的生成唯一值。

+   `LOCAL`

    来自源的日志位置状态信息，以单个具有以下键的 JSON 对象提供：

    `binary_log_file`

    当前二进制日志文件的名称。

    `binary_log_position`

    访问`log_status`表时的当前二进制日志位置。

    `gtid_executed`

    在访问`log_status`表时全局服务器变量`gtid_executed`的当前值。此信息与`binary_log_file`和`binary_log_position`键一致。

+   `REPLICATION`

    一个包含以下信息的通道的 JSON 数组：

    `channel_name`

    复制通道的名称。默认复制通道的名称为空字符串（“”）。

    `relay_log_file`

    复制通道的当前中继日志文件的名称。

    `relay_log_pos`

    访问`log_status`表时的当前中继日志位置。

+   `STORAGE_ENGINES`

    从各个存储引擎提供的相关信息，以每个适用存储引擎的一个键的 JSON 对象。

`log_status`表没有索引。

访问`log_status`表需要`BACKUP_ADMIN`权限以及`SELECT`权限。

`TRUNCATE TABLE`不允许用于`log_status`表。
