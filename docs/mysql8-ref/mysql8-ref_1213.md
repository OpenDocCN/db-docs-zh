# 17.8.9 清理配置

> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-purge-configuration.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-purge-configuration.html)

当您使用 SQL 语句删除行时，`InnoDB` 不会立即从数据库中物理删除行。只有在 `InnoDB` 丢弃为删除编写的撤销日志记录时，行及其索引记录才会被物理删除。此删除操作仅在行不再需要用于多版本并发控制（MVCC）或回滚后发生，称为清理。

清理定期运行。它解析并处理来自历史列表的撤销日志页，历史列表是由 `InnoDB` 事务系统维护的已提交事务的撤销日志页列表。清理在处理完撤销日志页后会释放它们。

#### 配置清理线程

清理操作由一个或多个清理线程在后台执行。清理线程的数量由`innodb_purge_threads` 变量控制。默认值为 4。

如果 DML 操作集中在单个表上，则该表的清理操作由单个清理线程执行，这可能导致清理操作减慢、清理延迟增加，并且如果 DML 操作涉及大对象值，则表空间文件大小会增加。从 MySQL 8.0.26 开始，如果超过了`innodb_max_purge_lag` 设置，清理工作会自动在可用的清理线程之间重新分配。在这种情况下，过多的活动清理线程可能会与用户线程发生争用，因此请相应地管理`innodb_purge_threads` 设置。`innodb_max_purge_lag` 变量默认设置为 0，这意味着默认情况下没有最大清理延迟。

如果 DML 操作集中在少数表上，请将`innodb_purge_threads` 设置较低，以便线程不会因争夺对繁忙表的访问而相互竞争。如果 DML 操作分布在许多表上，请考虑设置更高的`innodb_purge_threads`。清理线程的最大数量为 32。

`innodb_purge_threads` 设置是允许的最大清理线程数。清理系统会自动调整使用的清理线程数。

#### 配置清理批量大小

`innodb_purge_batch_size`变量定义了清除器在一个批处理中解析和处理的撤销日志页数。默认值为 300。在多线程清除配置中，协调员清除线程将`innodb_purge_batch_size`除以`innodb_purge_threads`，并将该数量的页分配给每个清除线程。

清除系统还释放不再需要的撤销日志页。它每 128 次迭代通过撤销日志这样做。除了定义在批处理中解析和处理的撤销日志页数外，`innodb_purge_batch_size`变量还定义了在每 128 次迭代中清除系统释放的撤销日志页数。

`innodb_purge_batch_size`变量用于高级性能调优和实验。大多数用户不需要更改`innodb_purge_batch_size`的默认值。

#### 配置最大清除延迟

`innodb_max_purge_lag`变量定义了期望的最大清除延迟。当清除延迟超过`innodb_max_purge_lag`阈值时，将对`INSERT`、`UPDATE`和`DELETE`操作施加延迟，以便清除操作赶上。默认值为 0，表示没有最大清除延迟和延迟。

`InnoDB`事务系统维护一个由`UPDATE`或`DELETE`操作删除标记的索引记录的事务列表。列表的长度即为清除延迟。在 MySQL 8.0.14 之前，清除延迟延迟是通过以下公式计算的，结果是最小延迟为 5000 微秒：

```sql
(purge lag/innodb_max_purge_lag - 0.5) * 10000
```

从 MySQL 8.0.14 开始，清除延迟延迟是通过以下修订后的公式计算的，将最小延迟减少到 5 微秒。5 微秒的延迟对于现代系统更为合适。

```sql
(purge_lag/innodb_max_purge_lag - 0.9995) * 10000
```

延迟是在清除批处理的开始时计算的。

对于有问题的工作负载，典型的`innodb_max_purge_lag`设置可能是 1000000（100 万），假设事务很小，只有 100 字节大小，并且可以有 100MB 的未清除表行。

清除延迟显示为`TRANSACTIONS`部分中的`History list length`值，在`SHOW ENGINE INNODB STATUS`输出中。

```sql
mysql> SHOW ENGINE INNODB STATUS;
...
------------
TRANSACTIONS
------------
Trx id counter 0 290328385
Purge done for trx's n:o < 0 290315608 undo n:o < 0 17
History list length 20
```

`History list length` 通常是一个较低的值，通常少于几千，但是写入密集的工作负载或长时间运行的事务可能会导致其增加，即使是只读事务也可能会增加。长时间运行的事务可能会导致 `History list length` 增加的原因是，在诸如 `REPEATABLE READ` 这样的一致性读事务隔离级别下，事务必须返回与创建该事务的读视图时相同的结果。因此，`InnoDB` 多版本并发控制（MVCC）系统必须保留数据的副本在撤销日志中，直到所有依赖于该数据的事务完成。以下是可能导致 `History list length` 增加的长时间运行事务的示例：

+   当存在大量并发的 DML 时，执行一个使用 `--single-transaction` 选项的 **mysqldump** 操作。

+   在禁用 `autocommit` 后运行 `SELECT` 查询，并忘记发出显式的 `COMMIT` 或 `ROLLBACK`。

为了防止在极端情况下出现过大的清理延迟，您可以通过设置 `innodb_max_purge_lag_delay` 变量来限制延迟。`innodb_max_purge_lag_delay` 变量指定了当超过 `innodb_max_purge_lag` 阈值时施加的延迟的最大延迟时间（以微秒为单位）。指定的 `innodb_max_purge_lag_delay` 值是由 `innodb_max_purge_lag` 公式计算的延迟时间的上限。

#### 清理和撤销表空间截断

清理系统还负责截断撤销表空间。您可以配置 `innodb_purge_rseg_truncate_frequency` 变量来控制清理系统查找要截断的撤销表空间的频率。有关更多信息，请参见 截断撤销表空间。
