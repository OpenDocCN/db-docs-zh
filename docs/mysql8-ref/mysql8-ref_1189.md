> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-locking-reads.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-locking-reads.html)

#### 17.7.2.4 锁定读取

如果您在同一事务中查询数据然后插入或更新相关数据，常规的 `SELECT` 语句不提供足够的保护。其他事务可以更新或删除您刚刚查询的相同行。`InnoDB` 支持两种提供额��安全性的 锁定读取 类型：

+   `SELECT ... FOR SHARE`

    对读取的任何行设置共享模式锁定。其他会话可以读取这些行，但在您的事务提交之前不能修改它们。如果这些行中的任何行已被另一个尚未提交的事务更改，您的查询将等待该事务结束，然后使用最新值。

    注意

    `SELECT ... FOR SHARE` 是 `SELECT ... LOCK IN SHARE MODE` 的替代品，但 `LOCK IN SHARE MODE` 仍然可用于向后兼容。这两个语句是等效的。然而，`FOR SHARE` 支持 `OF *`table_name`*`、`NOWAIT` 和 `SKIP LOCKED` 选项。请参阅 Locking Read Concurrency with NOWAIT and SKIP LOCKED。

    在 MySQL 8.0.22 之前，`SELECT ... FOR SHARE` 需要 `SELECT` 权限和至少一个 `DELETE`、`LOCK TABLES` 或 `UPDATE` 权限。从 MySQL 8.0.22 开始，只需要 `SELECT` 权限。

    从 MySQL 8.0.22 开始，`SELECT ... FOR SHARE` 语句不会在 MySQL 授权表上获取读取锁。有关更多信息，请参阅 Grant Table Concurrency。

+   `SELECT ... FOR UPDATE`

    对于搜索遇到的索引记录，锁定行和任何相关的索引条目，就像您为这些行发出 `UPDATE` 语句一样。其他事务被阻止更新这些行，执行 `SELECT ... FOR SHARE`，或在某些事务隔离级别下读取数据。一致性读取会忽略在读取视图中存在的记录上设置的任何锁定。（旧版本的记录不能被锁定；它们通过在记录的内存副本上应用 undo logs 来重建。）

    `SELECT ... FOR UPDATE` 需要 `SELECT` 权限和至少一个 `DELETE`、`LOCK TABLES` 或 `UPDATE` 权限。

这些子句主要在处理树形结构或图形结构数据时非常有用，无论是在单个表中还是跨多个表中。您可以从一个地方遍历边缘或树枝到另一个地方，同时保留回来更改任何这些“指针”值的权利。

由`FOR SHARE`和`FOR UPDATE`查询设置的所有锁定在事务提交或回滚时释放。

注意

只有在禁用自动提交时（通过使用`START TRANSACTION`开始事务或将`autocommit`设置为 0）才能进行锁定读取。

外部语句中的锁定读取子句不会锁定嵌套子查询中表的行，除非在子查询中也指定了锁定读取子句。例如，以下语句不会锁定表`t2`中的行。

```sql
SELECT * FROM t1 WHERE c1 = (SELECT c1 FROM t2) FOR UPDATE;
```

要锁定表`t2`中的行，请向子查询添加一个锁定读取子句：

```sql
SELECT * FROM t1 WHERE c1 = (SELECT c1 FROM t2 FOR UPDATE) FOR UPDATE;
```

##### 锁定读取示例

假设您想要向表`child`中插入新行，并确保子行在表`parent`中有父行。您的应用程序代码可以确保在这一系列操作中保持引用完整性。

首先，使用一致性读取查询表`PARENT`并验证父行是否存在。您可以安全地将子行插入到`CHILD`表中吗？不可以，因为在您的`SELECT`和`INSERT`之间的某一时刻，其他会话可能会删除父行，而您却不知情。

为避免这种潜在问题，使用`FOR SHARE`执行`SELECT`：

```sql
SELECT * FROM parent WHERE NAME = 'Jones' FOR SHARE;
```

在`FOR SHARE`查询返回父记录`'Jones'`后，您可以安全地将子记录添加到`CHILD`表中并提交事务。任何试图在`PARENT`表中适用行中获取独占锁的事务都会等待，直到您完成，也就是说，直到所有表中的数据处于一致状态。

举个例子，考虑一个表`CHILD_CODES`中的整数计数器字段，用于为添加到`CHILD`表中的每个子记录分配唯一标识符。不要使用一致性读取或共享模式读取来读取计数器的当前值，因为数据库的两个用户可能会看到计数器的相同值，并且如果两个事务尝试向`CHILD`表中添加具有相同标识符的行，则会发生重复键错误。

在这里，`FOR SHARE`不是一个好的解决方案，因为如果两个用户同时读取计数器，其中至少一个在尝试更新计数器时会陷入死锁。

要实现对计数器的读取和递增操作，首先使用`FOR UPDATE`执行计数器的锁定读取，然后再递增计数器。例如：

```sql
SELECT counter_field FROM child_codes FOR UPDATE;
UPDATE child_codes SET counter_field = counter_field + 1;
```

`SELECT ... FOR UPDATE`读取最新可用数据，在读取的每一行上设置排他锁。因此，它设置了与搜索的 SQL `UPDATE`在行上设置的相同锁。

上述描述仅仅是演示了`SELECT ... FOR UPDATE`的工作方式的一个例子。在 MySQL 中，实际上可以仅通过一次访问表来完成生成唯一标识符的特定任务：

```sql
UPDATE child_codes SET counter_field = LAST_INSERT_ID(counter_field + 1);
SELECT LAST_INSERT_ID();
```

`SELECT`语句仅仅检索标识符信息（特定于当前连接）。它不访问任何表。

##### 使用`NOWAIT`和`SKIP LOCKED`进行锁定读取并发

如果一行被事务锁定，请求相同锁定行的`SELECT ... FOR UPDATE`或`SELECT ... FOR SHARE`事务必须等待阻塞事务释放行锁。这种行为防止了事务更新或删除被其他事务查询更新的行。然而，如果希望查询在请求的行被锁定时立即返回，或者接受将被锁定的行排除在结果集之外，则无需等待行锁被释放。

为了避免等待其他事务释放行锁，可以在`SELECT ... FOR UPDATE`或`SELECT ... FOR SHARE`锁定读取语句中使用`NOWAIT`和`SKIP LOCKED`选项。

+   `NOWAIT`

    使用`NOWAIT`的锁定读取永远不会等待获取行锁。查询立即执行，如果请求的行被锁定，则失败并返回错误。

+   `SKIP LOCKED`

    使用`SKIP LOCKED`的锁定读取永远不会等待获取行锁。查询立即执行，从结果集中移除被锁定的行。

    注意

    跳过被锁定行的查询返回数据的不一致视图。因此，`SKIP LOCKED`不适用于一般的事务工作。然而，当多个会话访问相同的类似队列的表时，可以用于避免锁争用。

`NOWAIT`和`SKIP LOCKED`仅适用于行级锁。

使用`NOWAIT`或`SKIP LOCKED`的语句对基于语句的复制是不安全的。

以下示例演示了`NOWAIT`和`SKIP LOCKED`。会话 1 启动一个事务，锁定单个记录的行。会话 2 尝试使用`NOWAIT`选项对相同记录进行锁定读取。由于请求的行被会话 1 锁定，锁定读取立即返回错误。在会话 3 中，使用`SKIP LOCKED`的锁定读取返回请求的行，除了被会话 1 锁定的行。

```sql
# Session 1:

mysql> CREATE TABLE t (i INT, PRIMARY KEY (i)) ENGINE = InnoDB;

mysql> INSERT INTO t (i) VALUES(1),(2),(3);

mysql> START TRANSACTION;

mysql> SELECT * FROM t WHERE i = 2 FOR UPDATE;
+---+
| i |
+---+
| 2 |
+---+

# Session 2:

mysql> START TRANSACTION;

mysql> SELECT * FROM t WHERE i = 2 FOR UPDATE NOWAIT;
ERROR 3572 (HY000): Do not wait for lock.

# Session 3: 
mysql> START TRANSACTION;

mysql> SELECT * FROM t FOR UPDATE SKIP LOCKED;
+---+
| i |
+---+
| 1 |
| 3 |
+---+
```
