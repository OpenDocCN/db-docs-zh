# 17.12.7 在线 DDL 失败条件

> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-online-ddl-failure-conditions.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-online-ddl-failure-conditions.html)

在线 DDL 操作失败通常是由以下条件之一导致的：

+   一个`ALGORITHM`子句指定了一个与特定类型的 DDL 操作或存储引擎不兼容的算法。

+   一个`LOCK`子句指定了一个低程度的锁定（`SHARED`或`NONE`），这与特定类型的 DDL 操作不兼容。

+   在等待对表的独占锁时发生超时，这可能在 DDL 操作的初始和最终阶段短暂需要。

+   当 MySQL 在索引创建过程中在磁盘上写入临时排序文件时，`tmpdir`或`innodb_tmpdir`文件系统的磁盘空间不足。有关更多信息，请参见第 17.12.3 节，“在线 DDL 空间要求”。

+   该操作需要很长时间，同时并发的 DML 修改了表，使得临时在线日志的大小超过了`innodb_online_alter_log_max_size`配置选项的值。这种情况会导致`DB_ONLINE_LOG_TOO_BIG`错误。

+   并发的 DML 对表进行了更改，这些更改在原始表定义中是允许的，但在新表定义中不允许。该操作仅在最后阶段失败，当 MySQL 尝试应用所有并发 DML 语句的更改时。例如，您可能在创建唯一索引时插入重复值，或者在创建主键索引时插入`NULL`值。并发 DML 所做的更改优先级更高，并且`ALTER TABLE`操作实际上被回滚。
