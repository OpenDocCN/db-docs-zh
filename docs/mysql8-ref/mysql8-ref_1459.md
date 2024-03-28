> 原文：[`dev.mysql.com/doc/refman/8.0/en/replication-features-temptables.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-features-temptables.html)

#### 19.5.1.31 复制和临时表

在 MySQL 8.0 中，当`binlog_format`设置为`ROW`或`MIXED`时，仅使用临时表的语句不会在源上记录，因此临时表不会被复制。涉及临时表和非临时表混合的语句仅在源上为非临时表的操作记录，临时表的操作不会记录。这意味着在副本发生意外关闭时，副本上永远不会有临时表丢失。有关基于行的复制和临时表的更多信息，请参阅基于行的临时表记录。

当`binlog_format`设置为`STATEMENT`时，涉及临时表的语句在源上记录并在副本上复制，前提是涉及临时表的语句可以安全地使用基于语句的格式记录。在这种情况下，在副本上丢失复制的临时表可能是一个问题。在基于语句的复制模式中，当服务器上使用 GTIDs 时（即，当`enforce_gtid_consistency`系统变量设置为`ON`时），不能在事务、过程、函数或触发器中使用`CREATE TEMPORARY TABLE`和`DROP TEMPORARY TABLE`语句。当使用 GTIDs 时，可以在这些上下文之外使用它们，前提是设置了`autocommit=1`。

由于基于行或混合复制模式与基于语句的复制模式在临时表行为上的差异，如果更改适用于包含任何打开临时表的上下文（全局或会话），则不能在运行时切换复制格式。有关更多详细信息，请参阅`binlog_format`选项的描述。

**在使用临时表时安全地关闭复制。** 在基于语句的复制模式下，临时表会被复制，除非您停止复制服务器（而不仅仅是复制线程），并且您已经复制了在副本上尚未执行的更新中使用的临时表。如果停止复制服务器，则在重新启动副本时，这些更新所需的临时表将不再可用。为了避免这个问题，请不要在副本有打开的临时表时关闭副本。而是使用以下过程：

1.  发出`STOP REPLICA SQL_THREAD`语句。

1.  使用`SHOW STATUS`来检查`Replica_open_temp_tables`或`Slave_open_temp_tables`状态变量的值。

1.  如果值不为 0，请使用`START REPLICA SQL_THREAD`重新启动复制 SQL 线程，稍后重复该过程。

1.  当值为 0 时，发出**mysqladmin shutdown**命令来停止复制。

**临时表和复制选项。** 默认情况下，使用基于语句的复制时，所有临时表都会被复制；无论是否存在任何匹配的`--replicate-do-db`，`--replicate-do-table`，或`--replicate-wild-do-table`选项。但是，对于临时表，会遵守`--replicate-ignore-table`和`--replicate-wild-ignore-table`选项。唯一的例外是，为了在会话结束时正确删除临时表，复制总是会复制`DROP TEMPORARY TABLE IF EXISTS`语句，而不管通常适用于指定表的任何排除规则。

在使用基于语句的复制时，建议指定一个专用前缀用于命名不希望被复制的临时表，然后使用`--replicate-wild-ignore-table`选项来匹配该前缀。例如，您可以给所有这样的表命名以`norep`开头（如`norepmytable`，`norepyourtable`等），然后使用`--replicate-wild-ignore-table=norep%`来阻止它们被复制。
