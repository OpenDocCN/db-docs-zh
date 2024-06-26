# 17.3 InnoDB 多版本

> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-multi-versioning.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-multi-versioning.html)

`InnoDB`是一个多版本存储引擎。它保留有关已更改行的旧版本的信息，以支持事务功能，如并发和回滚。此信息存储在称为回滚段的数据结构中的撤销表空间中。请参见第 17.6.3.4 节，“撤销表空间”。`InnoDB`使用回滚段中的信息执行事务回滚所需的撤销操作。它还使用信息为一致读取构建行的早期版本。请参见第 17.7.2.3 节，“一致非锁定读取”。

在内部，`InnoDB`为存储在数据库中的每一行添加了三个字段：

+   一个 6 字节的`DB_TRX_ID`字段表示最后一次插入或更新行的事务标识符。此外，删除在内部被视为将一个特殊位设置为标记删除的更新。

+   一个名为 roll pointer 的 7 字节的`DB_ROLL_PTR`字段。roll pointer 指向写入回滚段的撤销日志记录。如果行已更新，则撤销日志记录包含重建行内容所需的信息，使其恢复到更新之前的状态。

+   一个 6 字节的`DB_ROW_ID`字段包含一个行 ID，随着插入新行而单调递增。如果`InnoDB`自动生成聚簇索引，则索引包含行 ID 值。否则，`DB_ROW_ID`列不会出现在任何索引中。

回滚段中的撤销日志分为插入和更新撤销日志。插入撤销日志仅在事务回滚中需要，并且可以在事务提交后立即丢弃。更新撤销日志也用于一致读取，但只有在没有事务存在时才能丢弃，对于这些事务，`InnoDB`已分配了一个快照，一致读取可能需要更新撤销日志中的信息来构建数据库行的早期版本。有关撤销日志的其他信息，请参见第 17.6.6 节，“撤销日志”。

建议您定期提交事务，包括仅发出一致读取的事务。否则，`InnoDB`无法从更新撤销日志中丢弃数据，回滚段可能会变得过大，填满其所在的撤销表空间。有关管理撤销表空间的信息，请参见第 17.6.3.4 节，“撤销表空间”。

回滚段中撤销日志记录的物理大小通常小于相应的插入或更新行。您可以使用此信息来计算回滚段所需的空间。

在`InnoDB`多版本方案中，当使用 SQL 语句删除行时，行不会立即从数据库中物理删除。只有当`InnoDB`丢弃为删除编写的更新撤销日志记录时，才会物理删除相应的行和其索引记录。这个删除操作称为清理，通常非常快，通常与执行删除操作的 SQL 语句花费的时间相同。

如果在表中以大致相同的速率批量插入和删除行，清理线程可能会开始滞后，表会因为所有“死”行而变得越来越大，使得所有操作都受限于磁盘，非常缓慢。在这种情况下，限制新行操作的速度，并通过调整`innodb_max_purge_lag`系统变量为清理线程分配更多资源。更多信息，请参见第 17.8.9 节，“清理配置”。

### 多版本和辅助索引

`InnoDB` 多版本并发控制（MVCC）对待辅助索引与聚簇索引的方式不同。聚簇索引中的记录会原地更新，并且它们的隐藏系统列指向撤销日志条目，可以从中重建记录的早期版本。与聚簇索引记录不同，辅助索引记录不包含隐藏系统列，也不会原地更新。

当更新辅助索引列时，旧的辅助索引记录会被标记为删除，新记录会被插入，而标记为删除的记录最终会被清除。当辅助索引记录被标记为删除或辅助索引页被新事务更新时，`InnoDB`会在聚簇索引中查找数据库记录。在聚簇索引中，记录的`DB_TRX_ID`会被检查，如果记录在读取事务启动后被修改，则从撤销日志中检索记录的正确版本。

如果辅助索引记录被标记为删除或辅助索引页被新事务更新，不会使用覆盖索引技术。`InnoDB`不会从索引结构返回值，而是在聚簇索引中查找记录。

然而，如果启用了 index condition pushdown (ICP)优化，并且`WHERE`条件的部分可以仅使用索引字段进行评估，MySQL 服务器仍然会将`WHERE`条件的这部分推送到存储引擎，在那里使用索引进行评估。如果找不到匹配的记录，则避免聚簇索引查找。如果找到匹配的记录，即使在标记为删除的记录中，`InnoDB`也会在聚簇索引中查找记录。
