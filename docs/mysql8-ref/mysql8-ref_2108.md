> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-error-log-table.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-error-log-table.html)

#### 29.12.21.2 错误日志表

MySQL 服务器维护的日志之一是错误日志，用于写入诊断消息（参见 Section 7.4.2, “错误日志”）。通常，服务器将诊断写入服务器主机上的文件或系统日志服务。从 MySQL 8.0.22 开始，根据错误日志配置，服务器还可以将最近的错误事件写入性能模式[`error_log`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-error-log-table.html)表。因此，授予[`SELECT`](https://dev.mysql.com/doc/refman/8.0/en/privileges-provided.html#priv_select)权限给[`error_log`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-error-log-table.html)表，使客户端和应用程序可以使用 SQL 查询访问错误日志内容，从而使 DBA 能够提供对日志的访问，而无需在服务器主机上允许直接文件系统访问。

[`error_log`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-error-log-table.html)表支持基于其更结构化列的专注查询。它还包括错误消息的完整文本，以支持更自由形式的分析。

表实现使用固定大小的内存环形缓冲区，旧事件会根据需要自动丢弃以为新事件腾出空间。

示例[`error_log`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-error-log-table.html)内容：

```sql
mysql> SELECT * FROM performance_schema.error_log\G
*************************** 1\. row ***************************
    LOGGED: 2020-08-06 09:25:00.338624
 THREAD_ID: 0
      PRIO: System
ERROR_CODE: MY-010116
 SUBSYSTEM: Server
      DATA: mysqld (mysqld 8.0.23) starting as process 96344
*************************** 2\. row ***************************
    LOGGED: 2020-08-06 09:25:00.363521
 THREAD_ID: 1
      PRIO: System
ERROR_CODE: MY-013576
 SUBSYSTEM: InnoDB
      DATA: InnoDB initialization has started.
...
*************************** 65\. row ***************************
    LOGGED: 2020-08-06 09:25:02.936146
 THREAD_ID: 0
      PRIO: Warning
ERROR_CODE: MY-010068
 SUBSYSTEM: Server
      DATA: CA certificate /var/mysql/sslinfo/cacert.pem is self signed.
...
*************************** 89\. row ***************************
    LOGGED: 2020-08-06 09:25:03.112801
 THREAD_ID: 0
      PRIO: System
ERROR_CODE: MY-013292
 SUBSYSTEM: Server
      DATA: Admin interface ready for connections, address: '127.0.0.1' port: 33062
```

[`error_log`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-error-log-table.html)表具有以下列。如描述中所示，除了`DATA`列外，所有列都对应于底层错误事件结构的字段，该结构在 Section 7.4.2.3, “错误事件字段”中有描述。

+   `LOGGED`

    事件时间戳，精确到微秒。`LOGGED`对应于错误事件的`time`字段，尽管可能存在某些潜在差异：

    +   错误日志中的`time`值根据[`log_timestamps`](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_log_timestamps)系统变量设置显示；参见早期启动日志输出格式。

    +   `LOGGED`列使用[`TIMESTAMP`](https://dev.mysql.com/doc/refman/8.0/en/datetime.html)数据类型存储值，其中值以 UTC 存储，但在检索时以当前会话时区显示；参见[Section 13.2.2, “日期、时间和时间戳类型”](https://dev.mysql.com/doc/refman/8.0/en/datetime.html)。

    要在与错误日志文件中显示的相同时区中显示`LOGGED`值，请首先设置会话时区如下：

    ```sql
    SET @@session.time_zone = @@global.log_timestamps;
    ```

    如果`log_timestamps`值为`UTC`，并且您的系统未安装命名时区支持（参见 Section 7.1.15, “MySQL Server Time Zone Support”），请像这样设置时区：

    ```sql
    SET @@session.time_zone = '+00:00';
    ```

+   `THREAD_ID`

    MySQL 线程 ID。`THREAD_ID`对应于错误事件的`thread`字段。

    在性能模式中，`error_log`表中的`THREAD_ID`列与`threads`表的`PROCESSLIST_ID`列最相似：

    +   对于前台线程，`THREAD_ID`和`PROCESSLIST_ID`表示连接标识符。这与`INFORMATION_SCHEMA` `PROCESSLIST`表中的`ID`列中显示的值相同，在`SHOW PROCESSLIST`输出中显示的`Id`列中显示，并在线程内由`CONNECTION_ID()`函数返回。

    +   对于后台线程，`THREAD_ID`为 0，`PROCESSLIST_ID`为`NULL`。

    除了`error_log`之外，许多性能模式表都有一个名为`THREAD_ID`的列，但在这些表中，`THREAD_ID`列是性能模式内部分配的值。

+   `PRIO`

    事件优先级。允许的值为`System`、`Error`、`Warning`、`Note`。`PRIO`列基于错误事件的`label`字段，该字段本身基于底层数字`prio`字段值。

+   `ERROR_CODE`

    数字事件错误代码。`ERROR_CODE`对应于错误事件的`error_code`字段。

+   `SUBSYSTEM`

    事件发生的子系统。`SUBSYSTEM`对应于错误事件的`subsystem`字段。

+   `DATA`

    错误事件的文本表示。此值的格式取决于生成`error_log`行的日志接收组件生成的格式。例如，如果日志接收组件是`log_sink_internal`或`log_sink_json`，`DATA`值分别表示传统格式或 JSON 格式的错误事件。（参见 Section 7.4.2.9, “Error Log Output Format”.）

    因为错误日志可以重新配置以更改提供行给`error_log`表的日志接收组件，并且不同的接收组件生成不同的输出格式，所以在不同时间写入`error_log`表的行可能具有不同的`DATA`格式。

`error_log`表具有以下索引：

+   在(`LOGGED`)上的主键

+   在(`THREAD_ID`)上的索引

+   在(`PRIO`)上的索引

+   在(`ERROR_CODE`)上的索引

+   在(`SUBSYSTEM`)上的索引

不允许对`error_log`表进行`TRUNCATE TABLE`操作。

##### `error_log`表的实现和配置

性能模式`error_log`表由写入表中的错误日志事件的错误日志接收组件填充，同时还写入格式化的错误事件到错误日志中。日志接收器通过性能模式支持有两个部分：

+   日志接收组件可以在发生时将新的错误事件写入`error_log`表。

+   日志接收组件可以提供用于提取先前编写的错误消息的解析器。这使得服务器实例能够读取前一个实例写入错误日志文件的消息，并将它们存储在`error_log`表中。前一个实例在关闭期间写入的消息可能有助于诊断关闭发生的原因。

目前，传统格式的`log_sink_internal`和 JSON 格式的`log_sink_json`接收器支持将新事件写入`error_log`表，并提供解析器用于读取先前编写的错误日志文件。

`log_error_services`系统变量控制着哪些日志组件用于错误日志记录。其值是一个管道，当发生错误事件时，按从左到右的顺序执行日志过滤器和日志接收组件。`log_error_services`的值与填充`error_log`表相关如下：

+   在启动时，服务器检查`log_error_services`的值，并从中选择最左边满足以下条件的日志接收器：

    +   支持`error_log`表并提供解析器的接收器。

    +   如果没有，支持`error_log`表但不提供解析器的接收器。

    如果没有日志接收器满足这些条件，则`error_log`表保持为空。否则，如果接收器提供解析器并且日志配置使先前写入的错误日志文件可以被找到，则服务器使用接收器解析器读取文件的最后部分，并将其中包含的旧事件写入表中。然后，接收器会在事件发生时将新的错误事件写入表中。

+   在运行时，如果`log_error_services`的值发生变化，服务器会再次检查它，这次寻找支持`error_log`表的最左边启用的日志接收器，无论它是否提供解析器。

    如果不存在这样的日志接收器，则不会将其他错误事件写入`error_log`表。否则，新配置的接收器会在事件发生时将新的错误事件写入表中。

任何影响写入错误日志的配置都会影响`error_log`表的内容。这包括诸如详细程度、消息抑制和消息过滤等设置。它还适用于从先前日志文件中启动时读取的信息。例如，在先前配置为低详细程度的服务器实例中未写入的消息，在当前配置为更高详细程度的实例读取文件时不会变得可用。

`error_log`表是一个固定大小的内存中的环形缓冲区的视图，旧事件会根据需要自动丢弃以腾出空间给新事件。如下表所示，几个状态变量提供有关进行中的`error_log`操作的信息。

| 状态变量 | 含义 |
| --- | --- |
| `Error_log_buffered_bytes` | 表中使用的字节 |
| `Error_log_buffered_events` | 表中存在的事件 |
| `Error_log_expired_events` | 从表中丢弃的事件 |
| `Error_log_latest_write` | 最后写入表的时间 |
