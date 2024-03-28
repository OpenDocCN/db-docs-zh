> 原文：[`dev.mysql.com/doc/refman/8.0/en/binary-log-mixed.html`](https://dev.mysql.com/doc/refman/8.0/en/binary-log-mixed.html)

#### 7.4.4.3 混合二进制日志格式

当在`MIXED`日志格式下运行时，服务器在以下情况下会自动从基于语句的日志切换到基于行的日志记录：

+   当函数包含`UUID()`时。

+   当更新一个或多个具有`AUTO_INCREMENT`列的表并调用触发器或存储函数时。与所有其他不安全语句一样，如果`binlog_format = STATEMENT`，则会生成警告。

    更多信息，请参见第 19.5.1.1 节，“复制和 AUTO_INCREMENT”。

+   当视图的主体需要基于行的复制时，创建视图的语句也会使用它。例如，当创建视图的语句使用`UUID()`函数时。

+   当涉及对可加载函数的调用时。

+   当使用`FOUND_ROWS()`或`ROW_COUNT()`时。（Bug #12092, Bug #30244）

+   当使用`USER()`、`CURRENT_USER()`或`CURRENT_USER`时。（Bug #28086）

+   当涉及的表之一是`mysql`数据库中的日志表时。

+   当使用`LOAD_FILE()`函数时。（Bug #39701）

+   当语句涉及一个或多个系统变量时。（Bug #31168）

    **异常。** 下列系统变量在会话范围（仅限）中使用时不会导致日志格式切换：

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

    有关确定系统变量范围的信息，请参见 Section 7.1.9, “Using System Variables”。

    有关复制如何处理`sql_mode`的信息，请参见 Section 19.5.1.39, “Replication and Variables”。

在早期版本中，当使用混合二进制日志格式时，如果一条语句被记录为行，并且执行该语句的会话有任何临时表，那么所有后续语句都被视为不安全，并以行为基础的格式记录，直到该会话中使用的所有临时表都被删除。从 MySQL 8.0 开始，对临时表的操作不会以混合二进制日��格式记录，并且会话中临时表的存在不会影响每条语句使用的日志模式。

注意

如果尝试使用基于语句的日志记录执行应该使用基于行的日志记录的语句，则会生成警告。警告会显示在客户端（在`SHOW WARNINGS`的输出中）和通过**mysqld**错误日志。每次执行这样的语句时，都会向`SHOW WARNINGS`表中添加一个警告。但是，为了防止日志淹没，每个客户端会话中生成警告的第一条语句才会写入错误日志。

除了上述决定外，各个引擎还可以确定在更新表中的信息时使用的日志格式。各个引擎的日志记录能力可以定义如下：

+   如果一个引擎支持基于行的日志记录，那么该引擎被称为支持行日志记录。

+   如果一个引擎支持基于语句的日志记录，那么该引擎被称为支持语句日志记录。

给定的存储引擎可以支持任一或两种日志记录格式。以下表格列出了每个引擎支持的格式。

| 存储引擎 | 支持行日志记录 | 支持语句日志记录 |
| --- | --- | --- |
| `ARCHIVE` | 是 | 是 |
| `BLACKHOLE` | 是 | 是 |
| `CSV` | 是 | 是 |
| `EXAMPLE` | 是 | 否 |
| `FEDERATED` | 是 | 是 |
| `HEAP` | 是 | 是 |
| `InnoDB` | 是 | 当事务隔离级别为`REPEATABLE READ`或`SERIALIZABLE`时为是；否则为否。 |
| `MyISAM` | 是 | 是 |
| `MERGE` | 是 | 是 |
| `NDB` | 是 | 否 |
| 存储引擎 | 支持行记录 | 支持语句记录 |

语句是否记录以及使用的记录模式是根据语句类型（安全、不安全或二进制注入）、二进制日志格式（`STATEMENT`、`ROW`或`MIXED`）以及存储引擎的记录能力（支持语句、支持行、两者都支持或两者都不支持）来确定的。（二进制注入指的是必须使用`ROW`格式记录的更改的记录。）

语句可能会被记录，有或没有警告；失败的语句不会被记录，但会在日志中生成错误。这在以下决策表中显示。**类型**，**binlog_format**，**SLC**和**RLC**列概述了条件，**错误/警告**和**记录为**列代表相应的操作。**SLC**代表“支持语句记录”，**RLC**代表“支持行记录”。

| 类型 | `binlog_format` | SLC | RLC | 错误/警告 | 记录为 |
| --- | --- | --- | --- | --- | --- |
| * | `*` | 否 | 否 | 错误：无法执行语句：由于至少有一个既不支持行也不支持语句的引擎参与其中，因此无法进行二进制日志记录。 | `-` |
| 安全 | `STATEMENT` | 是 | 否 | - | `STATEMENT` |
| 安全 | `MIXED` | 是 | 否 | - | `STATEMENT` |
| 安全 | `ROW` | 是 | 否 | 错误：无法执行语句：由于`BINLOG_FORMAT = ROW`且至少有一张表使用不支持基于行的日志记录的存储引擎，因此无法进行二进制日志记录。 | `-` |
| 不安全 | `STATEMENT` | 是 | 否 | 警告：由于`BINLOG_FORMAT = STATEMENT`，不安全语句以语句格式记录到二进制日志中 | `STATEMENT` |
| 不安全 | `MIXED` | 是 | 否 | 错误：无法执行语句：当存储引擎限制为基于语句的日志记录时，即使`BINLOG_FORMAT = MIXED`，也无法对不安全语句进行二进制日志记录。 | `-` |
| 不安全 | `ROW` | 是 | 否 | 错误：无法执行语句：由于`BINLOG_FORMAT = ROW`且至少有一张表使用不支持基于行的日志记录的存储引擎，因此无法进行二进制日志记录。 | - |
| 行注入 | `STATEMENT` | 是 | 否 | 错误：无法执行行注入：由于至少有一张表使用不支持基于行的日志记录的存储引擎，因此无法进行二进制日志记录。 | - |
| 行注入 | `MIXED` | 是 | 否 | 错误：无法执行行注入：由于至少有一张表使用不支持基于行的日志记录的存储引擎，因此无法进行二进制日志记录。 | - |
| 行注入 | `ROW` | 是 | 否 | 错误：无法执行行注入：由于至少有一张表使用不支持基于行的日志记录的存储引擎，因此无法进行二进制日志记录。 | - |
| 安全 | `STATEMENT` | 否 | 是 | 错误：无法执行语句：由于`BINLOG_FORMAT = STATEMENT`且至少有一张表使用不支持基于语句的日志记录的存储引擎，因此无法进行二进制日志记录。 | `-` |
| 安全 | `MIXED` | 否 | 是 | - | `ROW` |
| 安全 | `ROW` | 否 | 是 | - | `ROW` |
| 不安全 | `STATEMENT` | 否 | 是 | 错误：无法执行语句：由于 `BINLOG_FORMAT = STATEMENT`，并且至少有一个表使用不支持基于语句的日志记录的存储引擎，因此无法进行二进制日志记录。 | - |
| 不安全 | `MIXED` | 否 | 是 | - | `ROW` |
| 不安全 | `ROW` | 否 | 是 | - | `ROW` |
| 行注入 | `STATEMENT` | 否 | 是 | 错误：无法执行行注入：由于 `BINLOG_FORMAT = STATEMENT`，因此无法进行二进制日志记录。 | `-` |
| 行注入 | `MIXED` | 否 | 是 | - | `ROW` |
| 行注入 | `ROW` | 否 | 是 | - | `ROW` |
| 安全 | `STATEMENT` | 是 | 是 | - | `STATEMENT` |
| 安全 | `MIXED` | 是 | 是 | - | `STATEMENT` |
| 安全 | `ROW` | 是 | 是 | - | `ROW` |
| 不安全 | `STATEMENT` | 是 | 是 | 警告：由于 `BINLOG_FORMAT = STATEMENT`，不安全的语句以语句格式记录到二进制日志中。 | `STATEMENT` |
| 不安全 | `MIXED` | 是 | 是 | - | `ROW` |
| 不安全 | `ROW` | 是 | 是 | - | `ROW` |
| 行注入 | `STATEMENT` | 是 | 是 | 错误：无法执行行注入：由于 `BINLOG_FORMAT = STATEMENT`，因此无法进行二进制日志记录。 | - |
| 行注入 | `MIXED` | 是 | 是 | - | `ROW` |
| 行注入 | `ROW` | 是 | 是 | - | `ROW` |
| 类型 | `binlog_format` | SLC | RLC | 错误 / 警告 | 记录为 |

当决定产生警告时，会产生一个标准的 MySQL 警告（可以使用 `SHOW WARNINGS` 查看）。这些信息也会被写入到 **mysqld** 错误日志中。为了防止日志被淹没，每个客户端连接的每个错误实例只记录一个错误。日志消息包括尝试执行的 SQL 语句。

如果一个复制实例设置了 `log_error_verbosity` 以显示警告，那么复制实例会将消息打印到错误日志中，以提供有关其状态的信息，例如开始作业的二进制日志和中继日志坐标，切换到另一个中继日志时，重新连接后的情况，不适合基于语句的日志记录的语句等等。
