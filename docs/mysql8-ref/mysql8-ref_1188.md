> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-consistent-read.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-consistent-read.html)

#### 17.7.2.3 一致性非锁定读取

一致性读取意味着`InnoDB`使用多版本控制向查询展示数据库在某个时间点的快照。查询看到在该时间点之前提交的事务所做的更改，而不会看到稍后或未提交事务所做的更改。这个规则的例外是查询看到同一事务中较早语句所做的更改。这个例外导致以下异常：如果您更新表中的某些行，`SELECT`会看到更新行的最新版本，但也可能看到任何行的旧版本。如果其他会话同时更新同一表，这种异常意味着您可能看到数据库中从未存在的表状态。

如果事务的隔离级别是`REPEATABLE READ`（默认级别），同一事务内的所有一致性读取都读取该事务中第一个这样的读取建立的快照。您可以通过提交当前事务，然后发出新查询来为您的查询获取更新的快照。

使用`READ COMMITTED`隔离级别时，事务内的每个一致性读取都会设置并读取自己的新快照。

一致性读取是`InnoDB`在`READ COMMITTED`和`REPEATABLE READ`隔离级别下处理`SELECT`语句的默认模式。一致性读取不会在访问的表上设置任何锁定，因此其他会话可以在执行一致性读取的同时自由修改这些表。

假设您正在使用默认的`REPEATABLE READ`隔离级别。当您发出一致性读取（即普通的`SELECT`语句）时，`InnoDB`根据您的查询看到数据库的时间点为您的事务分配一个时间点。如果另一个事务删除一行并在分配您的时间点后提交，则您不会看到该行已被删除。插入和更新的处理方式类似。

注意

数据库状态的快照适用于事务内的`SELECT`语句，而不一定适用于 DML 语句。如果你插入或修改了一些行然后提交该事务，另一个并发的`REPEATABLE READ`事务发出的`DELETE`或`UPDATE`语句可能会影响那些刚刚提交的行，尽管该会话无法查询它们。如果一个事务更新或删除了另一个事务提交的行，那些更改将对当前事务可见。例如，你可能会遇到以下情况：

```sql
SELECT COUNT(c1) FROM t1 WHERE c1 = 'xyz';
-- Returns 0: no rows match.
DELETE FROM t1 WHERE c1 = 'xyz';
-- Deletes several rows recently committed by other transaction.

SELECT COUNT(c2) FROM t1 WHERE c2 = 'abc';
-- Returns 0: no rows match.
UPDATE t1 SET c2 = 'cba' WHERE c2 = 'abc';
-- Affects 10 rows: another txn just committed 10 rows with 'abc' values.
SELECT COUNT(c2) FROM t1 WHERE c2 = 'cba';
-- Returns 10: this txn can now see the rows it just updated.
```

你可以通过提交事务然后执行另一个`SELECT`或`START TRANSACTION WITH CONSISTENT SNAPSHOT`来推进时间点。

这被称为多版本并发控制。

在下面的例子中，会话 A 只有在 B 提交插入并且 A 也提交后才能看到 B 插入的行，这样时间点就会超过 B 的提交。

```sql
 Session A              Session B

           SET autocommit=0;      SET autocommit=0;
time
|          SELECT * FROM t;
|          empty set
|                                 INSERT INTO t VALUES (1, 2);
|
v          SELECT * FROM t;
           empty set
                                  COMMIT;

           SELECT * FROM t;
           empty set

           COMMIT;

           SELECT * FROM t;
           ---------------------
 |    1    |    2    |
           ---------------------
```

如果你想查看数据库的“最新”状态，请使用`READ COMMITTED`隔离级别或者锁定读取。

```sql
SELECT * FROM t FOR SHARE;
```

使用`READ COMMITTED`隔离级别时，事务内的每个一致性读取都设置并读取自己的新快照。使用`FOR SHARE`时，会发生锁定读取：`SELECT`会阻塞，直到包含最新行的事务结束（参见第 17.7.2.4 节，“锁定读取”）。

一致性读取在某些 DDL 语句上不起作用：

+   一致性读取在`DROP TABLE`上不起作用，因为 MySQL 无法使用已删除的表，而`InnoDB`会销毁该表。

+   一致性读取在进行使原始表的临时副本并在构建临时副本时删除原始表的`ALTER TABLE`操作上不起作用。当在事务内重新发出一致性读取时，新表中的行不可见，因为这些行在事务的快照被拍摄时不存在。在这种情况下，事务会返回一个错误：`ER_TABLE_DEF_CHANGED`，“表定义已更改，请重试事务”。

对于像`INSERT INTO ... SELECT`，`UPDATE ... (SELECT)`和`CREATE TABLE ... SELECT`等子句中的选择，如果没有指定`FOR UPDATE`或`FOR SHARE`，则读取类型会有所不同：

+   默认情况下，`InnoDB`对这些语句使用更强的锁，并且`SELECT`部分的行为类似于`READ COMMITTED`，即每个一致性读取，即使在同一事务中，也会设置并读取自己的新快照。

+   要在这种情况下执行非锁定读取，将事务的隔离级别设置为`READ UNCOMMITTED`或`READ COMMITTED`，以避免对从所选表中读取的行设置锁。
