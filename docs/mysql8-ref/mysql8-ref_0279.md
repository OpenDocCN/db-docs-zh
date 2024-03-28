# 7.4.5 慢查询日志

> 原文：[`dev.mysql.com/doc/refman/8.0/en/slow-query-log.html`](https://dev.mysql.com/doc/refman/8.0/en/slow-query-log.html)

慢查询日志包含执行时间超过`long_query_time`秒且需要至少检查`min_examined_row_limit`行的 SQL 语句。慢查询日志可用于查找执行时间长且需要优化的查询。然而，检查长时间的慢查询日志可能是一项耗时的任务。为了简化这个过程，可以使用**mysqldumpslow**命令处理慢查询日志文件并总结其内容。参见第 6.6.10 节，“mysqldumpslow — Summarize Slow Query Log Files”。

获取初始锁的时间不计入执行时间。**mysqld**在执行完语句并释放所有锁之后将其写入慢查询日志，因此日志顺序可能与执行顺序不同。

+   慢查询日志参数

+   慢查询日志内容

#### 慢查询日志参数

`long_query_time`的最小和默认值分别为 0 和 10。该值可以以微秒为分辨率进行指定。

默认情况下，不会记录管理语句，也不会记录未使用索引进行查找的查询。可以通过`log_slow_admin_statements`和`log_queries_not_using_indexes`进行更改，稍后会详细描述。

默认情况下，慢查询日志是禁用的。要明确指定初始慢查询日志状态，请使用[`--slow_query_log[={0|1}]`](server-system-variables.html#sysvar_slow_query_log)。没有参数或参数为 1 时，`--slow_query_log`启用日志。参数为 0 时，此选项禁用日志。要指定日志文件名，请使用`--slow_query_log_file=*`file_name`*`。要指定日志目标，请使用`log_output`系统变量（如第 7.4.1 节，“选择通用查询日志和慢查询日志输出目标”中所述）。

注意

如果指定`TABLE`日志目的地，请参阅 Log Tables and “Too many open files” Errors。

如果未为慢查询日志文件指定名称，则默认名称为`*`host_name`*-slow.log`。服务器将在数据目录中创建该文件，除非给定绝对路径名以指定不同的目录。

要在运行时禁用或启用慢查询日志，或更改日志文件名，请使用全局`slow_query_log`和`slow_query_log_file`系统变量。将`slow_query_log`设置为 0 以禁用日志，设置为 1 以启用日志。将`slow_query_log_file`设置为指定日志文件的名称。如果已经打开了日志文件，则关闭该文件并打开新文件。

如果使用`--log-short-format`选项，服务器将向慢查询日志写入较少的信息。

要在慢查询日志中包含慢管理语句，请启用`log_slow_admin_statements`系统变量。管理语句包括`ALTER TABLE`、`ANALYZE TABLE`、`CHECK TABLE`、`CREATE INDEX`、`DROP INDEX`、`OPTIMIZE TABLE`和`REPAIR TABLE`。

要在慢查询日志中写入不使用索引进行行查找的查询语句，请启用`log_queries_not_using_indexes`系统变量。（即使启用了该变量，由于表少于两行，服务器也不会记录不受索引影响的查询。）

当记录不使用索引的查询时，慢查询日志可能会迅速增长。可以通过设置`log_throttle_queries_not_using_indexes`系统变量对这些查询设置速率限制。默认情况下，此变量为 0，表示没有限制。正值对不使用索引的查询的日志记录施加每分钟限制。第一个这样的查询打开一个 60 秒的窗口，在此窗口内，服务器记录达到给定限制的查询，然后抑制额外的查询。如果在窗口结束时有被抑制的查询，服务器将记录一个摘要，指示有多少个查询以及在其中花费的总时间。当服务器记录下一个不使用索引的查询时，下一个 60 秒的窗口开始。

服务器按照以下顺序使用控制参数来确定是否将查询写入慢查询日志：

1.  查询必须不是管理语句，或者必须启用`log_slow_admin_statements`。

1.  查询必须至少花费`long_query_time`秒，或者必须启用`log_queries_not_using_indexes`，并且查询没有使用索引进行行查找。

1.  查询必须至少检查了`min_examined_row_limit`行。

1.  查询必须根据`log_throttle_queries_not_using_indexes`设置而不被抑制。

`log_timestamps`系统变量控制写入慢查询日志文件（以及一般查询日志文件和错误日志）中消息的时间戳的时区。它不影响写入日志表的一般查询日志和慢查询日志消息的时区，但是可以通过`CONVERT_TZ()`或设置会话`time_zone`系统变量，将从这些表中检索的行从本地系统时区转换为任何所需的时区。

默认情况下，副本不会将复制的查询写入慢查询日志。要更改此设置，请启用系统变量`log_slow_replica_statements`（从 MySQL 8.0.26 开始）或`log_slow_slave_statements`（在 MySQL 8.0.26 之前）。请注意，如果使用基于行的复制（`binlog_format=ROW`），这些系统变量不起作用。仅当以语句格式在二进制日志中记录时，即当设置了`binlog_format=STATEMENT`时，或者当设置了`binlog_format=MIXED`并且语句以语句格式记录时，才会将查询添加到副本的慢查询日志中。即使启用了`log_slow_replica_statements`或`log_slow_slave_statements`，当以行格式记录慢查询时，即使设置了`binlog_format=MIXED`，或者当设置了`binlog_format=ROW`，也不会将其添加到副本的慢查询日志中。

#### 慢查询日志内容

当慢查询日志被启用时，服务器会将输出写入由`log_output`系统变量指定的任何目的地。如果启用了日志，服务器会打开日志文件并将启动消息写入其中。然而，除非选择了`FILE`日志目的地，否则不会进一步将查询记录到文件中。如果目的地是`NONE`，即使启用了慢查询日志，服务器也不会写入任何查询。如果未选择`FILE`作为输出目的地，则设置日志文件名不会影响日志记录。

如果启用了慢查询日志并选择了`FILE`作为输出目的地，则写入日志的每个语句前面都会有一行以`#`字符开头并具有以下字段（所有字段都在一行上）：

+   `Query_time: *`持续时间`*`

    语句执行时间（秒）。

+   `Lock_time: *`持续时间`*`

    获取锁所需的时间（秒）。

+   `Rows_sent: *`N`*`

    发送到客户端的行数。

+   `Rows_examined:`

    `服务器层检查的行数（不包括存储引擎内部处理的任何处理）。`

`启用`log_slow_extra`系统变量（MySQL 8.0.14 起可用）会导致服务器将以下额外字段写入`FILE`输出，除了刚刚列出的字段外（`TABLE`输出不受影响）。一些字段描述涉及状态变量名称。有关更多信息，请参考状态变量描述。但是，在慢查询日志中，计数器是每个语句的值，而不是每个会话的累积值。

+   `Thread_id: *`ID`*`

    语句线程标识符。

+   `Errno: *`错误编号`*`

    语句错误编号，如果没有错误则为 0。

+   `Killed: *`N`*`

    如果语句被终止，表示为什么的错误编号，如果语句正常终止则为 0。

+   `Bytes_received: *`N`*`

    `Bytes_received`语句的值。

+   `Bytes_sent: *`N`*`

    `Bytes_sent`语句的值。

+   `Read_first: *`N`*`

    `Handler_read_first`语句的值。

+   `Read_last: *`N`*`

    `Handler_read_last`语句的值。

+   `Read_key: *`N`*`

    `Handler_read_key`语句的值。

+   `Read_next: *`N`*`

    `Handler_read_next`语句的值。

+   `Read_prev: *`N`*`

    `Handler_read_prev`语句的值。

+   `Read_rnd: *`N`*`

    `Handler_read_rnd`语句的值。

+   `Read_rnd_next: *`N`*`

    `Handler_read_rnd_next`语句的值。

+   `Sort_merge_passes: *`N`*`

    `Sort_merge_passes`语句的值。

+   `Sort_range_count: *`N`*`

    `Sort_range`语句的值。

+   `Sort_rows: *`N`*`

    `Sort_rows`语句的值。

+   `Sort_scan_count: *`N`*`

    `Sort_scan`语句的值。

+   `Created_tmp_disk_tables: *`N`*`

    `Created_tmp_disk_tables`语句的值。

+   `Created_tmp_tables: *`N`*`

    `Created_tmp_tables`语句的值。

+   `Start: *`时间戳`*`

    语句执行开始时间。

+   `End: *`时间戳`*`

    语句执行结束时间。

给定的慢查询日志文件可能包含启用`log_slow_extra`后添加了额外字段的行和没有额外字段的行。日志文件分析器可以通过字段计数确定一行是否包含额外字段。

每条写入慢查询日志文件的语句前面都有一个包含时间戳的`SET`语句。从 MySQL 8.0.14 开始，时间戳表示慢语句开始执行的时间。在 8.0.14 之前，时间戳表示慢语句被记录的时间（这发生在语句执行完成后）。

写入慢查询日志的语句中的密码会被服务器重写，以避免明文出现。参见第 8.1.2.3 节，“密码和日志记录”。

从 MySQL 8.0.29 开始，由于语法错误等原因无法解析的语句不会被写入慢查询日志。
