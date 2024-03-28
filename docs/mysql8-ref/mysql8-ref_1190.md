# 17.7.3 InnoDB 中不同 SQL 语句设置的锁

> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-locks-set.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-locks-set.html)

锁定读取、`UPDATE` 或 `DELETE` 通常会在处理 SQL 语句时扫描的每个索引记录上设置记录锁。语句中是否有 `WHERE` 条件排除行并不重要。`InnoDB` 不记得确切的 `WHERE` 条件，只知道扫描了哪些索引范围。这些锁通常是下一个键锁，还会阻止插入到记录之前的“间隙”。但是，可以显式禁用间隙锁定，这会导致不使用下一个键锁。有关更多信息，请参见第 17.7.1 节，“InnoDB 锁定”。事务隔离级别也会影响设置的锁；请参见第 17.7.2.1 节，“事务隔离级别”。

如果在搜索中使用了辅助索引，并且要设置的索引记录锁是排他的，则 `InnoDB` 还会检索相应的聚簇索引记录并对其设置锁。

如果您的语句没有适合的索引，MySQL 必须扫描整个表来处理语句，那么表的每一行都会被锁定，从而阻止其他用户向表中插入数据。重要的是要创建良好的索引，以便您的查询不会扫描比必要更多的行。

`InnoDB` 设置特定类型的锁如下。

+   `SELECT ... FROM` 是一致的读取，读取数据库的快照并不设置锁，除非事务隔离级别设置为 `SERIALIZABLE`。对于 `SERIALIZABLE` 级别，搜索会在遇到的索引记录上设置共享的下一个键锁。然而，只有使用唯一索引锁定行以搜索唯一行的语句需要索引记录锁。

+   使用唯一索引的`SELECT ... FOR UPDATE`和`SELECT ... FOR SHARE`语句会为扫描的行获取锁，并释放不符合结果集包含条件的行的锁（例如，如果它们不符合`WHERE`子句中给定的条件）。然而，在某些情况下，行可能不会立即解锁，因为在查询执行期间结果行与其原始来源之间的关系丢失。例如，在`UNION`中，从表中扫描（并锁定）的行可能会在评估它们是否符合结果集之前插入临时表中。在这种情况下，临时表中的行与原始表中的行之间的关系丢失，直到查询执行结束，后者的行才会解锁。

+   对于锁定读取（带有`FOR UPDATE`或`FOR SHARE`的`SELECT`，`UPDATE`和`DELETE`语句，所采取的锁取决于语句是否使用具有唯一搜索条件或范围类型搜索条件的唯一索引。

    +   对于具有唯一搜索条件的唯一索引，`InnoDB`仅锁定找到的索引记录，而不是其前面的间隙。

    +   对于其他搜索条件和非唯一索引，`InnoDB`会锁定扫描的索引范围，使用间隙锁或下一个键锁来阻止其他会话在范围覆盖的间隙中插入。有关间隙锁和下一个键锁的信息，请参见第 17.7.1 节，“InnoDB 锁定”。

+   对于索引记录，搜索遇到的`SELECT ... FOR UPDATE`会阻止其他会话执行`SELECT ... FOR SHARE`或在某些事务隔离级别下读取。一致性读取会忽略读取视图中存在的记录上设置的任何锁。

+   `UPDATE ... WHERE ...`在搜索遇到的每个记录上设置一个独占的下一个键锁。然而，仅对使用唯一索引锁定行以搜索唯一行的语句需要索引记录锁。

+   当`UPDATE`修改聚集索引记录时，会对受影响的次要索引记录采取隐式锁。在插入新的次要索引记录之前执行重复检查扫描时，`UPDATE`操作还会对受影响的次要索引记录采取共享锁，并在插入新的次要索引记录时也会采取共享锁。

+   `DELETE FROM ... WHERE ...`对搜索遇到的每条记录设置排他的 next-key 锁。然而，对于使用唯一索引锁定行以搜索唯一行的语句，只需要一个索引记录锁。

+   `INSERT`对插入的行设置排他锁。这个锁是一个索引记录锁，而不是一个 next-key 锁（也就是说，没有间隙锁），不会阻止其他会话在插入行之前插入到间隙中。

    在插入行之前，会设置一种称为插入意向间隙锁的间隙锁类型。这个锁表示插入的意图，以便多个插入相同索引间隙的事务不必等待彼此，如果它们不在间隙内的相同位置插入。假设存在值为 4 和 7 的索引记录。尝试插入值为 5 和 6 的单独事务在获得插入行的排他锁之前，会在 4 和 7 之间的间隙上设置插入意向锁，但不会相互阻塞，因为这些行是不冲突的。

    如果发生重复键错误，则会设置对重复索引记录的共享锁。这种共享锁的使用可能导致死锁，如果有多个会话尝试插入相同的行，而另一个会话已经具有排他锁。如果另一个会话删除了该行，则可能会发生这种情况。假设一个`InnoDB`表`t1`具有以下结构：

    ```sql
    CREATE TABLE t1 (i INT, PRIMARY KEY (i)) ENGINE = InnoDB;
    ```

    现在假设三个会话按顺序执行以下操作：

    第 1 节：

    ```sql
    START TRANSACTION;
    INSERT INTO t1 VALUES(1);
    ```

    第 2 节：

    ```sql
    START TRANSACTION;
    INSERT INTO t1 VALUES(1);
    ```

    第 3 节：

    ```sql
    START TRANSACTION;
    INSERT INTO t1 VALUES(1);
    ```

    第 1 节：

    ```sql
    ROLLBACK;
    ```

    会话 1 的第一个操作会为该行获取排他锁。会话 2 和 3 的操作都导致重复键错误，并且它们都请求该行的共享锁。当会话 1 回滚时，它释放了该行的排他锁，并且为会话 2 和 3 排队的共享锁请求被授予。此时，会话 2 和 3 发生死锁：由于彼此持有的共享锁，它们都无法获取该行的排他锁。

    如果表中已经包含具有键值 1 的行，并且三个会话按顺序执行以下操作，则会发生类似的情况：

    第 1 节：

    ```sql
    START TRANSACTION;
    DELETE FROM t1 WHERE i = 1;
    ```

    第 2 节：

    ```sql
    START TRANSACTION;
    INSERT INTO t1 VALUES(1);
    ```

    第 3 节：

    ```sql
    START TRANSACTION;
    INSERT INTO t1 VALUES(1);
    ```

    第 1 节：

    ```sql
    COMMIT;
    ```

    会话 1 的第一个操作会为该行获取排他锁。会话 2 和 3 的操作都导致重复键错误，并且它们都请求该行的共享锁。当会话 1 提交时，它释放了该行的排他锁，并且为会话 2 和 3 排队的共享锁请求被授予。此时，会话 2 和 3 发生死锁：由于彼此持有的共享锁，它们都无法获取该行的排他锁。

+   `INSERT ... ON DUPLICATE KEY UPDATE` 与简单的 `INSERT` 不同，当发生重复键错误时，会在要更新的行上放置一个独占锁而不是共享锁。对于重复的主键值，会采取独占的索引记录锁。对于重复的唯一键值，会采取独占的 next-key 锁。

+   `REPLACE` 如果在唯一键上没有冲突，则类似于 `INSERT`。否则，会在要替换的行上放置一个独占的 next-key 锁。

+   `INSERT INTO T SELECT ... FROM S WHERE ...` 会在插入到 `T` 中的每一行上设置一个独占的索引记录锁（不包括间隙锁）。如果事务隔离级别是 `READ COMMITTED`，`InnoDB` 会将 `S` 上的搜索作为一致性读取（无锁）进行。否则，`InnoDB` 会在来自 `S` 的行上设置共享的 next-key 锁。在后一种情况下，`InnoDB` 必须设置锁：在使用基于语句的二进制日志进行前滚恢复时，必须以与最初执行时完全相同的方式执行每个 SQL 语句。

    `CREATE TABLE ... SELECT ...` 使用共享的 next-key 锁或一致性读取执行 `SELECT`，就像 `INSERT ... SELECT` 一样。

    当在构造 `REPLACE INTO t SELECT ... FROM s WHERE ...` 或 `UPDATE t ... WHERE col IN (SELECT ... FROM s ...)` 中使用 `SELECT` 时，`InnoDB` 会在来自表 `s` 的行上设置共享的 next-key 锁。

+   在初始化表上先前指定的 `AUTO_INCREMENT` 列时，`InnoDB` 会在与 `AUTO_INCREMENT` 列关联的索引末尾设置一个��占锁。

    使用 `innodb_autoinc_lock_mode=0`，`InnoDB` 使用一种特殊的 `AUTO-INC` 表锁模式，在访问自增计数器时获得并保持锁直到当前 SQL 语句结束（而不是整个事务结束）。在持有 `AUTO-INC` 表锁时，其他客户端无法向表中插入数据。对于具有 `innodb_autoinc_lock_mode=1` 的“批量插入”，也会发生相同的行为。不使用表级 `AUTO-INC` 锁与 `innodb_autoinc_lock_mode=2`。更多信息，请参阅 第 17.6.1.6 节，“InnoDB 中的 AUTO_INCREMENT 处理”。

    `InnoDB` 在不设置任何锁的情况下获取先前初始化的 `AUTO_INCREMENT` 列的值。

+   如果在表上定义了`FOREIGN KEY`约束，任何需要检查约束条件的插入、更新或删除都会在查看以检查约束的记录上设置共享记录级锁。如果约束失败，`InnoDB`也会设置这些锁。

+   `LOCK TABLES`设置表锁，但是在`InnoDB`层上方的更高 MySQL 层设置这些锁。如果`innodb_table_locks = 1`（默认）且`autocommit = 0`，那么`InnoDB`会意识到表锁，而在`InnoDB`上方的 MySQL 层知道行级锁。

    否则，`InnoDB`的自动死锁检测无法检测涉及此类表锁的死锁。此外，因为在这种情况下更高的 MySQL 层不知道行级锁，所以可能在另一个会话当前具有行级锁的表上获取表锁。然而，这并不会危及事务完整性，如第 17.7.5.2 节“死锁检测”中所讨论的。

+   如果`innodb_table_locks=1`（默认），`LOCK TABLES`在每个表上获取两个锁。除了 MySQL 层上的表锁外，它还会获取一个`InnoDB`表锁。要避免获取`InnoDB`表锁，设置`innodb_table_locks=0`。如果没有获取`InnoDB`表锁，即使某些表的记录被其他事务锁定，`LOCK TABLES`也会完成。

    在 MySQL 8.0 中，`innodb_table_locks=0`对使用`LOCK TABLES ... WRITE`显式锁定的表没有影响。对于通过触发器隐式（例如，通过触发器）或通过`LOCK TABLES ... READ`读取或写入的表，它会产生影响。

+   当事务提交或中止时，所有`InnoDB`锁都会被释放。因此，在`autocommit=1`模式下对`InnoDB`表调用`LOCK TABLES`并没有太多意义，因为获取的`InnoDB`表锁会立即释放。

+   你不能在事务中间锁定额外的表，因为`LOCK TABLES`会执行隐式的`COMMIT`和`UNLOCK TABLES`。
