> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-transaction-isolation-levels.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-transaction-isolation-levels.html)

#### 17.7.2.1 事务隔离级别

事务隔离是数据库处理的基础之一。隔离是缩写 ACID 中的 I；隔离级别是在多个事务同时进行更改和执行查询时，微调性能和可靠性、一致性和结果可重现性之间平衡的设置。

`InnoDB`提供了 SQL:1992 标准描述的四种事务隔离级别：`READ UNCOMMITTED`, `READ COMMITTED`, `REPEATABLE READ`和`SERIALIZABLE`。`InnoDB`的默认隔离级别是`REPEATABLE READ`。

用户可以使用`SET TRANSACTION`语句更改单个会话或所有后续连接的隔离级别。要为所有连接设置服务器的默认隔离级别，请在命令行或选项文件中使用`--transaction-isolation`选项。有关隔离级别和级别设置语法的详细信息，请参见 Section 15.3.7, “SET TRANSACTION Statement”。

`InnoDB`使用不同的锁定策略支持这里描述的每个事务隔离级别。您可以使用默认的`REPEATABLE READ`级别强制执行高度一致性，用于关键数据的操作，其中 ACID 合规性很重要。或者您可以通过`READ COMMITTED`甚至`READ UNCOMMITTED`放宽一致性规则，用于诸如大量报告之类的情况，其中精确一致性和可重复结果不如最小化锁定开销重要。`SERIALIZABLE`比`REPEATABLE READ`甚至更严格，主要用于专门情况，例如 XA 事务和用于解决并发和死锁问题。

以下列表描述了 MySQL 支持不同事务级别的方式。列表从最常用的级别到最不常用的级别。

+   `REPEATABLE READ`

    这是`InnoDB`的默认隔离级别。在同一事务中进行一致性读取时，读取的是第一次读取时建立的快照。这意味着如果您在同一事务中发出几个普通（非锁定）`SELECT`语句，这些`SELECT`语句在彼此之间也是一致的。请参阅 Section 17.7.2.3, “Consistent Nonlocking Reads”。

    对于锁定读取（带有`FOR UPDATE`或`FOR SHARE`的`SELECT`）、`UPDATE`和`DELETE`语句，锁定取决于语句是否使用具有唯一搜索条件的唯一索引，或范围类型的搜索条件。

    +   对于具有唯一搜索条件的唯一索引，`InnoDB`仅锁定找到的索引记录，而不是其前面的间隙。

    +   对于其他搜索条件，`InnoDB`锁定扫描的索引范围，使用间隙锁或下一个键锁来阻止其他会话在范围覆盖的间隙中插入。有关间隙锁和下一个键锁的信息，请参阅 Section 17.7.1, “InnoDB Locking”。

+   `READ COMMITTED`

    每个一致性读取，即使在同一事务中，也会设置并读取自己的新快照。有关一致性读取的信息，请参阅 Section 17.7.2.3, “Consistent Nonlocking Reads”。

    对于锁定读取（带有`FOR UPDATE`或`FOR SHARE`的`SELECT`）、`UPDATE`和`DELETE`语句，`InnoDB`仅锁定索引记录，而不是它们之前的间隙，因此允许在锁定记录旁边自由插入新记录。间隙锁仅用于外键约束检查和重复键检查。

    由于间隙锁定已禁用，可能会出现幻影行问题，因为其他会话可以在间隙中插入新行。有关幻影行的信息，请参阅 Section 17.7.4, “Phantom Rows”。

    仅支持基于行的二进制日志记录与`READ COMMITTED`隔离级别。如果您在`binlog_format=MIXED`下使用`READ COMMITTED`，服务器会自动使用基于行的日志记录。

    使用`READ COMMITTED`还有额外的影响：

    +   对于`UPDATE`或`DELETE`语句，`InnoDB`仅为更新或删除的行保持锁定。对于不匹配的行，MySQL 在评估`WHERE`条件后释放记录锁。这极大地降低了死锁的概率，但仍然可能发生。

    +   对于`UPDATE`语句，如果一行已被锁定，`InnoDB`执行“半一致性”读取，将最新提交版本返回给 MySQL，以便 MySQL 确定该行是否与`UPDATE`的`WHERE`条件匹配。如果行匹配（必须更新），MySQL 再次读取该行，这时`InnoDB`要么锁定它，要么等待锁定。

    考虑以下示例，从这个表开始：

    ```sql
    CREATE TABLE t (a INT NOT NULL, b INT) ENGINE = InnoDB;
    INSERT INTO t VALUES (1,2),(2,3),(3,2),(4,3),(5,2);
    COMMIT;
    ```

    在这种情况下，表没有索引，因此搜索和索引扫描使用隐藏的聚簇索引进行记录锁定（参见第 17.6.2.1 节，“聚簇索引和辅助索引”），而不是索引列。

    假设一个会话执行以下语句进行`UPDATE`：

    ```sql
    # Session A
    START TRANSACTION;
    UPDATE t SET b = 5 WHERE b = 3;
    ```

    还假设第二个会话执行以下语句来执行`UPDATE`，紧随第一个会话之后：

    ```sql
    # Session B
    UPDATE t SET b = 4 WHERE b = 2;
    ```

    当`InnoDB`执行每个`UPDATE`时，首先为每一行获取独占锁，然后确定是否修改它。如果`InnoDB`不修改该行，则释放锁。否则，`InnoDB`将保留该锁直到事务结束。这会影响事务处理如下。

    当使用默认的`REPEATABLE READ`隔离级别时，第一个`UPDATE`在读取每一行时获取 x 锁，并且不释放任何一个：

    ```sql
    x-lock(1,2); retain x-lock
    x-lock(2,3); update(2,3) to (2,5); retain x-lock
    x-lock(3,2); retain x-lock
    x-lock(4,3); update(4,3) to (4,5); retain x-lock
    x-lock(5,2); retain x-lock
    ```

    第二个`UPDATE`在尝试获取任何锁定时立即阻塞（因为第一个更新已保留了所有行的锁定），并且在第一个`UPDATE`提交或回滚之前不会继续进行：

    ```sql
    x-lock(1,2); block and wait for first UPDATE to commit or roll back
    ```

    如果改用`READ COMMITTED`，第一个`UPDATE`在读取每一行时获取 x 锁，并释放那些它不修改的行：

    ```sql
    x-lock(1,2); unlock(1,2)
    x-lock(2,3); update(2,3) to (2,5); retain x-lock
    x-lock(3,2); unlock(3,2)
    x-lock(4,3); update(4,3) to (4,5); retain x-lock
    x-lock(5,2); unlock(5,2)
    ```

    对于第二个`UPDATE`，`InnoDB`进行了“半一致性”读取，将其读取的每一行的最新提交版本返回给 MySQL，以便 MySQL 确定该行是否与`UPDATE`的`WHERE`条件匹配：

    ```sql
    x-lock(1,2); update(1,2) to (1,4); retain x-lock
    x-lock(2,3); unlock(2,3)
    x-lock(3,2); update(3,2) to (3,4); retain x-lock
    x-lock(4,3); unlock(4,3)
    x-lock(5,2); update(5,2) to (5,4); retain x-lock
    ```

    然而，如果`WHERE`条件包括一个索引列，并且`InnoDB`使用了该索引，那么在获取和保留记录锁时只考虑索引列。在下面的例子中，第一个`UPDATE`在 b = 2 的每一行上获取并保留 x 锁。当第二个`UPDATE`尝试在相同记录上获取 x 锁时，由于它也使用了在列 b 上定义的索引，它会被阻塞。

    ```sql
    CREATE TABLE t (a INT NOT NULL, b INT, c INT, INDEX (b)) ENGINE = InnoDB;
    INSERT INTO t VALUES (1,2,3),(2,2,4);
    COMMIT;

    # Session A
    START TRANSACTION;
    UPDATE t SET b = 3 WHERE b = 2 AND c = 3;

    # Session B
    UPDATE t SET b = 4 WHERE b = 2 AND c = 4;
    ```

    `READ COMMITTED`隔离级别可以在启动时设置或在运行时更改。在运行时，它可以全局设置给所有会话，或者针对每个会话单独设置。

+   `READ UNCOMMITTED`

    `SELECT`语句以非锁定方式执行，但可能使用较早版本的行。因此，在这个隔离级别下，这样的读取是不一致的。这也被称为脏读。否则，这个隔离级别的工作方式类似于`READ COMMITTED`。

+   `SERIALIZABLE`

    这个级别类似于`REPEATABLE READ`，但如果`autocommit`被禁用，`InnoDB`会隐式地将所有普通的`SELECT`语句转换为`SELECT ... FOR SHARE`。如果`autocommit`被启用，`SELECT`就是它自己的事务。因此，它被认为是只读的，如果作为一致的（非锁定的）读取执行，可以进行序列化，不需要为其他事务阻塞。（要强制普通的`SELECT`在其他事务修改了所选行时阻塞，禁用`autocommit`。）

    注意

    从 MySQL 8.0.22 开始，从 MySQL 授权表中读取数据的 DML 操作（通过连接列表或子查询）但不修改它们的操作不会获取 MySQL 授权表的读锁，无论隔离级别如何。更多信息，请参阅授权表并发性。
