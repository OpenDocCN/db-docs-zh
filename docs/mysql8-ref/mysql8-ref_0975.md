# 15.3.1 `START TRANSACTION`、`COMMIT`和`ROLLBACK`语句

> 原文：[`dev.mysql.com/doc/refman/8.0/en/commit.html`](https://dev.mysql.com/doc/refman/8.0/en/commit.html)

```sql
START TRANSACTION
    [*transaction_characteristic* [, *transaction_characteristic*] ...]

*transaction_characteristic*: {
    WITH CONSISTENT SNAPSHOT
  | READ WRITE
  | READ ONLY
}

BEGIN [WORK]
COMMIT [WORK] [AND [NO] CHAIN] [[NO] RELEASE]
ROLLBACK [WORK] [AND [NO] CHAIN] [[NO] RELEASE]
SET autocommit = {0 | 1}
```

这些语句提供了对事务的控制：

+   `START TRANSACTION` 或 `BEGIN` 开始一个新事务。

+   `COMMIT` 提交当前事务，使其更改永久生效。

+   `ROLLBACK` 回滚当前事务，取消其更改。

+   `SET autocommit` 禁用或启用当前会话的默认自动提交模式。

默认情况下，MySQL 运行时启用自动提交模式。这意味着，当不在事务内时，每个语句都是原子的，就好像被 `START TRANSACTION` 和 `COMMIT` 包围。您无法使用 `ROLLBACK` 撤消效果；但是，如果在执行语句时发生错误，则会回滚该语句。

要隐式禁用单个语句序列的自动提交模式，请使用 `START TRANSACTION` 语句：

```sql
START TRANSACTION;
SELECT @A:=SUM(salary) FROM table1 WHERE type=1;
UPDATE table2 SET summary=@A WHERE type=1;
COMMIT;
```

使用 `START TRANSACTION`，自动提交保持禁用，直到您使用 `COMMIT` 或 `ROLLBACK` 结束事务。然后自动提交模式恢复到先前的状态。

`START TRANSACTION` 允许多个修饰符控制事务特性。要指定多个修饰符，请用逗号分隔它们。

+   `WITH CONSISTENT SNAPSHOT` 修饰符为能够执行此操作的存储引擎启动一致性读取。这仅适用于 `InnoDB`。其效果与发出 `START TRANSACTION` 然后从任何 `InnoDB` 表中进行 `SELECT` 相同。请参阅 Section 17.7.2.3, “Consistent Nonlocking Reads”。`WITH CONSISTENT SNAPSHOT` 修饰符不会更改当前事务的隔离级别，因此仅在当前隔离级别允许一致性读取时才提供一致的快照。唯一允许一致性读取的隔离级别是 `REPEATABLE READ`。对于所有其他隔离级别，`WITH CONSISTENT SNAPSHOT` 子句将被忽略。当忽略 `WITH CONSISTENT SNAPSHOT` 子句时会生成警告。

+   `READ WRITE` 和 `READ ONLY` 修饰符设置事务访问模式。它们允许或禁止对事务中使用的表进行更改。`READ ONLY` 限制阻止事务修改或锁定对其他事务可见的事务和非事务表；事务仍然可以修改或锁定临时表。

    当事务已知为只读时，MySQL 在`InnoDB`表上的查询会启用额外的优化。指定`READ ONLY`可确保在无法自动确定只读状态的情况下应用这些优化。有关更多信息，请参阅第 10.5.3 节，“优化 InnoDB 只读事务”。

    如果未指定访问模式，则应用默认模式。除非默认值已更改，否则为读/写。在同一语句中不允许同时指定`READ WRITE`和`READ ONLY`。

    在只读模式下，仍然可以使用 DML 语句更改使用`TEMPORARY`关键字创建的表。使用 DDL 语句进行的更改是不允许的，就像永久表一样。

    有关事务访问模式的其他信息，包括更改默认模式的方法，请参阅第 15.3.7 节，“SET TRANSACTION 语句”。

    如果启用了`read_only`系统变量，则使用`START TRANSACTION READ WRITE`显式启动事务需要`CONNECTION_ADMIN`权限（或已弃用的`SUPER`权限）。

重要提示

许多用于编写 MySQL 客户端应用程序的 API（如 JDBC）提供了自己的启动事务方法，可以（有时应该）代替从客户端发送`START TRANSACTION`语句。有关更多信息，请参阅第三十一章，“连接器和 API”，或您的 API 文档。

要显式禁用自动提交模式，请使用以下语句：

```sql
SET autocommit=0;
```

将`autocommit`变量设置为零以禁用自动提交模式后，对事务安全表（例如`InnoDB`或`NDB`）的更改不会立即生效。您必须使用`COMMIT`将更改存储到磁盘，或使用`ROLLBACK`忽略更改。

`autocommit` 是一个会话变量，必须为每个会话设置。要为每个新连接禁用自动提交模式，请参阅`autocommit`系统变量在第 7.1.8 节，“服务器系统变量”的描述。

`BEGIN`和`BEGIN WORK`可作为`START TRANSACTION`的别名来启动事务。`START TRANSACTION`是标准 SQL 语法，是启动临时事务的推荐方式，并允许`BEGIN`不支持的修饰符。

`BEGIN`语句与使用`BEGIN`关键字开始`BEGIN ... END`复合语句的方式不同。后者不会开始事务。请参见第 15.6.1 节，“BEGIN ... END 复合语句”。

注意

在所有存储程序（存储过程和函数、触发器和事件）中，解析器将`BEGIN [WORK]`视为`BEGIN ... END`块的开始。在这种情况下使用`START TRANSACTION`开始一个事务。

可选的`WORK`关键字支持`COMMIT`和`ROLLBACK`，以及`CHAIN`和`RELEASE`子句。`CHAIN`和`RELEASE`可用于对事务完成进行额外控制。`completion_type`系统变量的值确定默认完成行为。请参见第 7.1.8 节，“服务器系统变量”。

`AND CHAIN`子句导致新事务在当前事务结束后立即开始，并且新事务具有与刚终止事务相同的隔离级别。新事务还使用与刚终止事务相同的访问模式（`READ WRITE`或`READ ONLY`）。`RELEASE`子句导致服务器在终止当前事务后断开当前客户端会话。包括`NO`关键字可以抑制`CHAIN`或`RELEASE`完成，如果`completion_type`系统变量设置为默认导致链接或释放完成时，这可能很有用。

开始事务会导致任何待处理的事务被提交。更多信息请参见第 15.3.3 节，“导致隐式提交的语句”。

开始事务还会导致使用`LOCK TABLES`获取的表锁被释放，就好像执行了`UNLOCK TABLES`一样。开始事务不会释放使用`FLUSH TABLES WITH READ LOCK`获取的全局读锁。

为了获得最佳结果，事务应仅使用单个事务安全存储引擎管理的表执行。否则，可能会出现以下问题：

+   如果使用来自多个事务安全存储引擎（如`InnoDB`）的表，并且事务隔离级别不是`SERIALIZABLE`，那么当一个事务提交时，使用相同表的另一个正在进行的事务可能只看到第一个事务所做的一部分更改。也就是说，混合引擎的事务的原子性不能得到保证，可能会导致不一致性。（如果混合引擎事务不频繁，您可以使用`SET TRANSACTION ISOLATION LEVEL`根据需要在每个事务基础上将隔离级别设置为`SERIALIZABLE`。）

+   如果在事务中使用非事务安全的表，那么对这些表的更改将立即存储，而不考虑自动提交模式的状态。

+   如果在事务中更新非事务表后发出`ROLLBACK`语句，将会出现`ER_WARNING_NOT_COMPLETE_ROLLBACK`警告。事务安全表的更改将被回滚，但非事务安全表的更改不会被回滚。

每个事务在`COMMIT`时以一个块的形式存储在二进制日志中。被回滚的事务不会被记录。（**例外**：对非事务表的修改无法回滚。如果被回滚的事务包括对非事务表的修改，则整个事务将在结尾处使用`ROLLBACK`语句记录，以确保对非事务表的修改被复制。）请参阅 Section 7.4.4, “The Binary Log”。

你可以使用`SET TRANSACTION`语句更改事务的隔离级别或访问模式。请参阅 Section 15.3.7, “SET TRANSACTION Statement”。

回滚可能是一个缓慢的操作，可能会在用户没有明确要求的情况下隐式发生（例如，当发生错误时）。因此，`SHOW PROCESSLIST`在会话的`State`列中显示`Rolling back`，不仅适用于使用`ROLLBACK`语句执行的显式回滚，还适用于隐式回滚。

注意

在 MySQL 8.0 中，`BEGIN`、`COMMIT`和`ROLLBACK`不受`--replicate-do-db`或`--replicate-ignore-db`规则的影响。

当`InnoDB`执行事务的完全回滚时，事务设置的所有锁都会被释放。如果事务中的一个 SQL 语句由于错误（如重复键错误）而回滚，那么该语句设置的锁将在事务保持活动状态时保留。这是因为`InnoDB`以一种格式存储行锁，以至于事后无法知道哪个锁是由哪个语句设置的。

如果事务中的一个`SELECT`语句调用了一个存储函数，并且存储函数中的一个语句失败，那么该语句将会回滚。如果随后为该事务执行`ROLLBACK`，整个事务将会回滚。
