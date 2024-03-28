# 10.5.3 优化 InnoDB 只读事务

> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-performance-ro-txn.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-performance-ro-txn.html)

`InnoDB` 可以避免为已知为只读的事务设置事务 ID（`TRX_ID` 字段）所带来的开销。只有可能执行写操作或锁定读取（如 `SELECT ... FOR UPDATE`）的事务才需要事务 ID。消除不必要的事务 ID 可减少每次查询或数据更改语句构建读视图时所查询的内部数据结构的大小。

当：

+   事务是通过 `START TRANSACTION READ ONLY` 语句启动的。在这种情况下，尝试对数据库进行更改（对于 `InnoDB`、`MyISAM` 或其他类型的表）会导致错误，并且事务将继续处于只读状态：

    ```sql
    ERROR 1792 (25006): Cannot execute statement in a READ ONLY transaction.
    ```

    在只读事务中，您仍然可以对会话特定的临时表进行更改，或者为其发出锁定查询，因为这些更改和锁定对任何其他事务都不可见。

+   `autocommit` 设置已打开，因此可以保证事务是单个语句，并且组成事务的单个语句是“非锁定” `SELECT` 语句。也就是说，不使用 `FOR UPDATE` 或 `LOCK IN SHARED MODE` 子句的 `SELECT`。

+   事务是在没有 `READ ONLY` 选项的情况下启动的，但尚未执行更新或明确锁定行的语句。在需要更新或明确锁定之前，事务保持在只读模式下。

因此，对于像报表生成器这样的读密集型应用程序，您可以通过将它们组合在 `START TRANSACTION READ ONLY` 和 `COMMIT` 中，或者在运行 `SELECT` 语句之前打开 `autocommit` 设置，或者简单地避免在查询中插入任何数据更改语句来调整一系列 `InnoDB` 查询。

有关 `START TRANSACTION` 和 `autocommit` 的信息，请参见 Section 15.3.1, “START TRANSACTION, COMMIT, and ROLLBACK Statements”。

注意

符合自动提交、非锁定和只读（AC-NL-RO）条件的事务将被排除在某些内部`InnoDB`数据结构之外，因此不会在`SHOW ENGINE INNODB STATUS`输出中列出。
