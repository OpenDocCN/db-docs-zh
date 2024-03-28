# 17.12.8 在线 DDL 限制

> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-online-ddl-limitations.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-online-ddl-limitations.html)

在线 DDL 操作有以下限制：

+   在 `TEMPORARY TABLE` 上创建索引时，表会被复制。

+   如果表上存在 `ON...CASCADE` 或 `ON...SET NULL` 约束，则不允许使用 `ALTER TABLE` 子句 `LOCK=NONE`。

+   在原地在线 DDL 操作完成之前，必须等待持有表上元数据锁的事务提交或回滚。在线 DDL 操作在执行阶段可能会短暂地需要表上的独占元数据锁，并且在更新表定义时的操作的最后阶段始终需要一个。因此，持有表上元数据锁的事务可能会导致在线 DDL 操作阻塞。持有表上元数据锁的事务可能在在线 DDL 操作之前或期间启动。持有表上元数据锁的长时间运行或不活动事务可能会导致在线 DDL 操作超时。

+   在运行原地在线 DDL 操作时，运行 `ALTER TABLE` 语句的线程会应用同时在其他连接线程上并发运行的 DML 操作的在线日志。当应用 DML 操作时，可能会遇到重复键入错误（ERROR 1062 (23000): Duplicate entry），即使重复条目只是临时的，并且会在在线日志中的后续条目中被撤销。这类似于 `InnoDB` 中外键约束检查的概念，在事务期间约束必须保持。

+   对于 `InnoDB` 表的 `OPTIMIZE TABLE` 被映射为一个 `ALTER TABLE` 操作，用于重建表并更新索引统计信息以及释放聚簇索引中未使用的空间。由于键按照它们在主键中出现的顺序插入，次要索引的创建效率不高。通过为重建常规和分区 `InnoDB` 表提供在线 DDL 支持，支持 `OPTIMIZE TABLE`。

+   在 MySQL 5.6 之前创建的包含时间列（`DATE`、`DATETIME` 或 `TIMESTAMP`）且未使用 `ALGORITHM=COPY` 重建的表不支持 `ALGORITHM=INPLACE`。在这种情况下，`ALTER TABLE ... ALGORITHM=INPLACE` 操作会返回以下错误：

    ```sql
    ERROR 1846 (0A000): ALGORITHM=INPLACE is not supported.
    Reason: Cannot change column type INPLACE. Try ALGORITHM=COPY.
    ```

+   在涉及重建表的大表的在线 DDL 操作中，通常适用以下限制：

    +   没有机制可以暂停在线 DDL 操作或限制在线 DDL 操作的 I/O 或 CPU 使用率。

    +   如果在线 DDL 操作失败，回滚操作可能会很昂贵。

    +   长时间运行的在线 DDL 操作可能导致复制延迟。在线 DDL 操作必须在源端完成运行后才能在副本上运行。此外，在源端并发处理的 DML 仅在副本上在副本上的 DDL 操作完成后才会处理。

    有关在大表上运行在线 DDL 操作的更多信息，请参见第 17.12.2 节，“在线 DDL 性能和并发性”。
