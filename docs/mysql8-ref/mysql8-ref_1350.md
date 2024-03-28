> 原文：[`dev.mysql.com/doc/refman/8.0/en/replication-gtids-restrictions.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-gtids-restrictions.html)

#### 19.1.3.7 GTID 复制的限制

因为基于 GTID 的复制依赖于事务，所以在使用时不支持 MySQL 中其他可用的一些功能。本节提供有关使用 GTID 进行复制的限制和限制的信息。

**涉及非事务性存储引擎的更新。** 在使用 GTID 时，对使用非事务性存储引擎（如`MyISAM`）的表进行更新不能与对使用事务性存储引擎（如`InnoDB`）的表进行更新在同一语句或事务中进行。

这个限制是因为在同一事务中对使用非事务性存储引擎的表进行更新与对使用事务性存储引擎的表进行更新可能导致将多个 GTID 分配给同一事务。

当源和副本为相同表的不同版本使用不同的存储引擎时，可能会出现这些问题，其中一个存储引擎是事务性的，另一个不是。还要注意，对非事务表进行操作的触发器可能是这些问题的原因。

在上述任何情况下，事务和 GTID 之间的一对一对应关系被打破，导致基于 GTID 的复制无法正常运行。

**CREATE TABLE ... SELECT 语句。** 在 MySQL 8.0.21 之前，当使用基于 GTID 的复制时，不允许使用 `CREATE TABLE ... SELECT` 语句。当 `binlog_format` 设置为 `STATEMENT` 时，`CREATE TABLE ... SELECT` 语句在二进制日志中记录为一个具有一个 GTID 的事务，但如果使用 `ROW` 格式，则该语句将记录为具有两个 GTID 的两个事务。如果源使用 `STATEMENT` 格式，副本使用 `ROW` 格式，则副本将无法正确处理事务，因此为了防止这种情况，不允许使用 GTID 的 `CREATE TABLE ... SELECT` 语句。在支持原子 DDL 的存储引擎上，此限制在 MySQL 8.0.21 中解除。在这种情况下，`CREATE TABLE ... SELECT` 在二进制日志中记录为一个事务。有关更多信息，请参见 Section 15.1.1, “原子数据定义语句支持”。

**临时表。** 当`binlog_format`设置为`STATEMENT`时，在服务器上使用 GTIDs 时（即当`enforce_gtid_consistency`系统变量设置为`ON`时），不能在事务、存储过程、函数和触发器中使用`CREATE TEMPORARY TABLE`和`DROP TEMPORARY TABLE`语句。当使用 GTIDs 时，可以在这些上下文之外使用它们，前提是设置了`autocommit=1`。从 MySQL 8.0.13 开始，当`binlog_format`设置为`ROW`或`MIXED`时，在使用 GTIDs 时，可以在事务、存储过程、函数或触发器中使用`CREATE TEMPORARY TABLE`和`DROP TEMPORARY TABLE`语句。这些语句不会写入二进制日志，因此不会被复制到副本。使用基于行的复制意味着副本保持同步，无需复制临时表。如果从事务中删除这些语句导致事务为空，该事务不会写入二进制日志。

**阻止执行不支持的语句。** 为防止导致基于 GTID 的复制失败的语句执行，启用 GTIDs 时必须在所有服务器上使用`--enforce-gtid-consistency`选项启动。这将导致本节前面讨论的任何类型的语句执行失败并显示错误。

请注意，只有在为语句启用二进制日志记录时，`--enforce-gtid-consistency`才会生效。如果服务器上禁用了二进制日志记录，或者如果语句未写入二进制日志因为它们被过滤器移除，那么对于未记录的语句，GTID 一致性不会被检查或强制执行。

有关启用 GTIDs 时其他必需的启动选项的信息，请参阅 Section 19.1.3.4, “Setting Up Replication Using GTIDs”。

**跳过事务。** 当使用基于 GTID 的复制时，`sql_replica_skip_counter`或`sql_slave_skip_counter`不可用。如果需要跳过事务，请使用源的`gtid_executed`变量的值。如果已经在复制通道上启用了 GTID 分配，使用`CHANGE REPLICATION SOURCE TO`语句的`ASSIGN_GTIDS_TO_ANONYMOUS_TRANSACTIONS`选项，则`sql_replica_skip_counter`或`sql_slave_skip_counter`是可用的。更多信息，请参见第 19.1.7.3 节，“跳过事务”。

**忽略服务器。** 在使用 GTIDs 时，`CHANGE REPLICATION SOURCE TO` | `CHANGE MASTER TO`语句的 IGNORE_SERVER_IDS 选项已被弃用，因为已经应用的事务会自动被忽略。在开始基于 GTID 的复制之前，请检查并清除之前在涉及的服务器上设置的所有被忽略的服务器 ID 列表。可以为单个通道发出的`SHOW REPLICA STATUS`语句显示被忽略的服务器 ID 列表（如果有）。如果没有列表，则`Replicate_Ignore_Server_Ids`字段为空。

**GTID 模式和 mysql_upgrade。** 在 MySQL 8.0.16 之前，当服务器启用全局事务标识符（GTIDs）（`gtid_mode=ON`）时，请不要通过**mysql_upgrade**（`--write-binlog`选项）启用二进制日志记录。从 MySQL 8.0.16 开始，服务器执行整个 MySQL 升级过程，但在升级过程中禁用二进制日志记录，因此没有问题。
