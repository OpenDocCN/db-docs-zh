> 原文：[`dev.mysql.com/doc/refman/8.0/en/binary-log-setting.html`](https://dev.mysql.com/doc/refman/8.0/en/binary-log-setting.html)

#### 7.4.4.2 设置二进制日志格式

您可以通过使用`--binlog-format=*`type`*`启动 MySQL 服务器来明确选择二进制日志记录格式。*`type`*支持的值为：

+   `STATEMENT`导致记录以语句为基础。

+   `ROW`导致记录以行为基础。这是默认设置。

+   `MIXED`导致记录使用混合格式。

设置二进制日志记录格式不会激活服务器的二进制日志记录。该设置仅在服务器启用二进制日志记录时生效，当`log_bin`系统变量设置为`ON`时为此情况。从 MySQL 8.0 开始，默认情况下启用二进制日志记录，只有在启动时指定`--skip-log-bin`或`--disable-log-bin`选项时才会禁用。

记录格式也可以在运行时切换，尽管请注意，在本节后面讨论的一些情况下，您无法这样做。设置全局值`binlog_format`系统变量以指定更改后连接的客户端的格式：

```sql
mysql> SET GLOBAL binlog_format = 'STATEMENT';
mysql> SET GLOBAL binlog_format = 'ROW';
mysql> SET GLOBAL binlog_format = 'MIXED';
```

通过设置`binlog_format`的会话值，个别客户端可以控制其自己语句的记录格式：

```sql
mysql> SET SESSION binlog_format = 'STATEMENT';
mysql> SET SESSION binlog_format = 'ROW';
mysql> SET SESSION binlog_format = 'MIXED';
```

更改全局`binlog_format`值需要具有足够权限来设置全局系统变量。更改会话`binlog_format`值需要具有足够权限来设置受限会话系统变量。参见 Section 7.1.9.1, “System Variable Privileges”。

有几个原因可能导致客户端希望按会话设置二进制日志记录：

+   对数据库进行许多小更改的会话可能希望使用基于行的记录。

+   执行更新匹配`WHERE`子句中许多行的会话可能希望使用基于语句的记录，因为记录少量语句比记录许多行更有效率。

+   一些语句在源端执行时需要大量时间，但只会修改少量行。因此，使用基于行的记录可能是有益的。

在运行时无法切换复制格式的情况有例外：

+   无法从存储函数或触发器内部更改复制格式。

+   如果启用了`NDB`存储引擎。

+   如果会话中有打开的临时表，则无法为该会话更改复制格式（`SET @@SESSION.binlog_format`）。

+   如果任何复制通道有打开的临时表，则无法全局更改复制格式（`SET @@GLOBAL.binlog_format` 或 `SET @@PERSIST.binlog_format`）。

+   如果任何复制通道应用程序线程当前正在运行，则无法全局更改复制格式（`SET @@GLOBAL.binlog_format` 或 `SET @@PERSIST.binlog_format`）。

在这些情况下尝试切换复制格式（或尝试设置当前复制格式）会导致错误。但是，您可以随时使用 `PERSIST_ONLY`（`SET @@PERSIST_ONLY.binlog_format`）更改复制格式，因为此操作不会修改运行时全局系统变量值，并且仅在服务器重新启动后生效。

不建议在存在任何临时表时在运行时切换复制格式，因为仅在使用基于语句的复制时才记录临时表，而在基于行的复制和混合复制中，它们不会被记录。

当复制正在进行时切换复制格式也可能会导致问题。每个 MySQL 服务器都可以设置自己的二进制日志格式（无论 `binlog_format` 是以全局还是会话范围设置）。这意味着在复制源服务器上更改日志格式不会导致副本更改其日志格式以匹配。在使用 `STATEMENT` 模式时，`binlog_format` 系统变量不会被复制。在使用 `MIXED` 或 `ROW` 日志模式时，它会被复制，但副本会忽略它。

副本无法将以 `ROW` 日志格式接收的二进制日志条目转换为 `STATEMENT` 格式以在其自己的二进制日志中使用。因此，如果源使用 `ROW` 或 `MIXED` 格式，则副本必须使用 `ROW` 或 `MIXED` 格式。在复制仍在进行的情况下，将源上的二进制日志格式从 `STATEMENT` 更改为 `ROW` 或 `MIXED` 到具有 `STATEMENT` 格式的副本可能会导致复制失败，出现错误，例如执行行事件时出错：'无法执行语句：由于语句处于行格式且 BINLOG_FORMAT = STATEMENT，因此无法写入二进制日志。' 当源仍在使用 `MIXED` 或 `ROW` 格式时，将副本上的二进制日志格式更改为 `STATEMENT` 格式也会导致相同类型的复制失败。要安全更改格式，必须停止复制，并确保在源和副本上都进行相同的更改。

如果您正在使用 `InnoDB` 表，并且事务隔离级别为 `READ COMMITTED` 或 `READ UNCOMMITTED`，则只能使用基于行的日志记录。可以*可能*将日志格式更改为 `STATEMENT`，但在运行时这样做会非常快速地导致错误，因为 `InnoDB` 无法再执行插入操作。

将二进制日志格式设置为 `ROW` 后，许多更改将以基于行的格式写入二进制日志。然而，仍然有一些更改使用基于语句的格式。例如，所有 DDL（数据定义语言）语句，如 `CREATE TABLE`, `ALTER TABLE`, 或 `DROP TABLE`。

当使用基于行的二进制日志记录时，`binlog_row_event_max_size` 系统变量及其对应的启动选项 `--binlog-row-event-max-size` 设置了行事件的最大大小的软限制。默认值为 8192 字节，该值只能在服务器启动时更改。在可能的情况下，存储在二进制日志中的行被分组为大小不超过此设置值的事件。如果事件无法分割，则最大大小可能会超过。

`--binlog-row-event-max-size` 选项适用于能够进行基于行的复制的服务器。行以不超过此选项值的字节大小的块存储在二进制日志中。该值必须是 256 的倍数。默认值为 8192。

警告

当为复制使用*基于语句的日志记录*时，如果语句设计为数据修改是非确定性的，即由查询优化器决定，则源和副本上的数据可能会变得不同。一般来说，即使在复制之外，这也不是一个好的做法。有关此问题的详细解释，请参见 Section B.3.7, “MySQL 中的已知问题”。
