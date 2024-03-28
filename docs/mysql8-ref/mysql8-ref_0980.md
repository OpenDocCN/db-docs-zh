# 15.3.6 `LOCK TABLES` 和 `UNLOCK TABLES` 语句

> 原文：[`dev.mysql.com/doc/refman/8.0/en/lock-tables.html`](https://dev.mysql.com/doc/refman/8.0/en/lock-tables.html)

```sql
LOCK {TABLE | TABLES}
    *tbl_name* [[AS] *alias*] *lock_type*
    [, *tbl_name* [[AS] *alias*] *lock_type*] ...

*lock_type*: {
    READ [LOCAL]
  | [LOW_PRIORITY] WRITE
}

UNLOCK {TABLE | TABLES}
```

MySQL 允许客户端会话明确为协作访问表的其他会话或在会话需要独占访问表时阻止其他会话修改表而获取表锁。一个会话只能为自己获取或释放锁。一个会话不能为另一个会话获取锁或释放另一个会话持有的锁。

锁可以用于模拟事务或在更新表时获得更快的速度。这在 Table-Locking Restrictions and Conditions 中有更详细的解释。

`LOCK TABLES` 明确为当前客户端会话获取表锁。表锁可以用于基表或视图。你必须拥有 `LOCK TABLES` 权限，并且对每个要锁定的对象都必须拥有 `SELECT` 权限。

对于视图锁定，`LOCK TABLES` 会将视图中使用的所有基表添加到要锁定的表集合中，并自动锁定它们。对于任何被锁定视图下的表，`LOCK TABLES` 会检查视图定义者（对于 `SQL SECURITY DEFINER` 视图）或调用者（对于所有视图）对表具有适当的权限。

如果你使用 `LOCK TABLES` 明确锁定一个表，那么任何触发器中使用的表也会隐式锁定，如 LOCK TABLES and Triggers 中所述。

如果你使用 `LOCK TABLES` 明确锁定一个表，那么任何通过外键约束相关的表会被隐式打开并锁定。对于外键检查，相关表会被采取共享只读锁（`LOCK TABLES READ`）。对于级联更新，涉及操作的相关表会被采取共享无写锁（`LOCK TABLES WRITE`）。

`UNLOCK TABLES` 明确释放当前会话持有的任何表锁。在获取新锁之前，`LOCK TABLES` 会隐式释放当前会话持有的任何表锁。

`UNLOCK TABLES`的另一个用途是释放使用`FLUSH TABLES WITH READ LOCK`语句获取的全局读锁，该语句使您可以锁定所有数据库中的所有表。请参见第 15.7.8.3 节，“FLUSH 语句”。（如果您有可以在某个时间点进行快照的文件系统（如 Veritas），这是一个非常方便的备份方式。）

`LOCK TABLE`是`LOCK TABLES`的同义词；`UNLOCK TABLE`是`UNLOCK TABLES`的同义词。

表锁仅防止其他会话进行不当读取或写入。持有`WRITE`锁的会话可以执行诸如`DROP TABLE`或`TRUNCATE TABLE`等表级操作。对于持有`READ`锁的会话，不允许执行`DROP TABLE`和`TRUNCATE TABLE`操作。

以下讨论仅适用于非`TEMPORARY`表。对于`TEMPORARY`表，允许（但会被忽略）使用`LOCK TABLES`。该表可以在创建它的会话中自由访问，而不受其他锁定的影响。不需要锁定，因为没有其他会话可以看到该表。

+   表锁定获取

+   表锁定释放

+   表锁定和事务的交互

+   LOCK TABLES 和触发器

+   表锁定的限制和条件

#### 表锁定获取

要在当前会话中获取表锁，请使用`LOCK TABLES`语句，该语句获取元数据锁（参见第 10.11.4 节，“元数据锁定”）。

可用的锁类型如下：

`READ [LOCAL]`锁：

+   拥有锁的会话可以读取表（但不能写入）。

+   多个会话可以同时为表获取`READ`锁。

+   其他会话可以在不显式获取`READ`锁的情况下读取表。

+   `LOCAL` 修饰符允许其他会话在持有锁的情况下执行非冲突的 `INSERT` 语句（并发插入）。但是，如果在持有锁的同时使用外部进程操作数据库，则不能使用 `READ LOCAL`。对于 `InnoDB` 表，`READ LOCAL` 与 `READ` 相同。

`[LOW_PRIORITY] WRITE` 锁：

+   持有锁的会话可以读取和写入表。

+   只有持有锁的会话才能访问表。在锁被释放之前，其他会话无法访问它。

+   当持有 `WRITE` 锁时，其他会话对表的锁请求将被阻塞。

+   `LOW_PRIORITY` 修饰符没有影响。在 MySQL 的早期版本中，它会影响锁定行为，但现在不再有效。它现在已被弃用，使用 `WRITE` 而不带 `LOW_PRIORITY` 会产生警告。

`WRITE` 锁通常比 `READ` 锁具有更高的优先级，以确保更新尽快处理。这意味着如果一个会话获取了 `READ` 锁，然后另一个会话请求 `WRITE` 锁，后续的 `READ` 锁请求将等待，直到请求 `WRITE` 锁的会话获取并释放锁。（对于 `max_write_lock_count` 系统变量的小值，可能会有例外情况；请参见 Section 10.11.4, “Metadata Locking”.）

如果由于其他会话在任何表上持有的锁而导致 `LOCK TABLES` 语句必须等待，它将阻塞直到所有锁都可以被获取。

需要锁定表的会话必须在单个 `LOCK TABLES` 语句中获取所有需要的锁。在持有这些锁的情况下，会话只能访问被锁定的表。例如，在以下语句序列中，尝试访问 `t2` 会导致错误，因为它在 `LOCK TABLES` 语句中未被锁定：

```sql
mysql> LOCK TABLES t1 READ;
mysql> SELECT COUNT(*) FROM t1;
+----------+
| COUNT(*) |
+----------+
|        3 |
+----------+
mysql> SELECT COUNT(*) FROM t2;
ERROR 1100 (HY000): Table 't2' was not locked with LOCK TABLES
```

`INFORMATION_SCHEMA` 数据库中的表是一个例外。即使一个会话持有使用 `LOCK TABLES` 获取的表锁，也可以在不显式锁定的情况下访问它们。

不能在单个查询中多次引用具有相同名称的锁定表。请改用别名，并为表和每个别名获取单独的锁：

```sql
mysql> LOCK TABLE t WRITE, t AS t1 READ;
mysql> INSERT INTO t SELECT * FROM t;
ERROR 1100: Table 't' was not locked with LOCK TABLES
mysql> INSERT INTO t SELECT * FROM t AS t1;
```

第一个`INSERT`出现错误是因为对于一个被锁定的表有两个相同名称的引用。第二个`INSERT`成功是因为对表的引用使用了不同的名称。

如果您的语句通过别名引用表，那么必须使用相同的别名锁定表。不能不指定别名而锁定表：

```sql
mysql> LOCK TABLE t READ;
mysql> SELECT * FROM t AS myalias;
ERROR 1100: Table 'myalias' was not locked with LOCK TABLES
```

相反，如果您使用别名锁定表，那么在语句中必须使用该别名引用它：

```sql
mysql> LOCK TABLE t AS myalias READ;
mysql> SELECT * FROM t;
ERROR 1100: Table 't' was not locked with LOCK TABLES
mysql> SELECT * FROM t AS myalias;
```

#### 表锁定释放

当会话持有的表锁被释放时，它们会同时被释放。一个会话可以显式释放其锁，或者在某些条件下锁可能会隐式释放。

+   一个会话可以通过`UNLOCK TABLES`显式释放其锁。

+   如果一个会话在已经持有锁的情况下发出`LOCK TABLES`语句以获取锁，那么在授予新锁之前，现有的锁会被隐式释放。

+   如果一个会话开始一个事务（例如，使用`START TRANSACTION`），会执行一个隐式的`UNLOCK TABLES`，导致现有的锁被释放。（有关表锁定和事务之间的交互的更多信息，请参阅表锁定和事务的交互.）

如果客户端会话的连接终止，无论是正常还是异常，服务器会隐式释放会话持有的所有表锁（事务性和非事务性）。如果客户端重新连接，那么这些锁将不再生效。此外，如果客户端有一个活动事务，在断开连接时服务器会回滚该事务，如果重新连接发生，则新会话将以自动提交启用的方式开始。因此，客户端可能希望禁用自动重新连接。启用自动重新连接时，如果重新连接发生，客户端不会收到通知，但任何表锁或当前事务都会丢失。禁用自动重新连接时，如果连接断开，下一个发出的语句将出现错误。客户端可以检测错误并采取适当的措施，如重新获取锁或重做事务。请参阅自动重新连接控制。

注意

如果你在一个被锁定的表上使用 `ALTER TABLE`，它可能会变为未锁定状态。例如，如果你尝试第二次 `ALTER TABLE` 操作，结果可能是一个错误 `Table '*`tbl_name`*' was not locked with LOCK TABLES`。为了处理这个问题，在第二次修改之前再次锁定表。另请参阅 Section B.3.6.1, “ALTER TABLE 存在的问题”。

#### 表锁定和事务的交互

`LOCK TABLES` 和 `UNLOCK TABLES` 与事务的使用交互如下：

+   `LOCK TABLES` 不是事务安全的，并在尝试锁定表之前隐式提交任何活动事务。

+   `UNLOCK TABLES` 隐式提交任何活动事务，但仅在使用 `LOCK TABLES` 获取表锁时。例如，在以下一组语句中，`UNLOCK TABLES` 释放全局读锁但不提交事务，因为没有表锁生效：

    ```sql
    FLUSH TABLES WITH READ LOCK;
    START TRANSACTION;
    SELECT ... ;
    UNLOCK TABLES;
    ```

+   开始一个事务（例如，使用 `START TRANSACTION`）隐式提交任何当前事务并释放现有的表锁。

+   `FLUSH TABLES WITH READ LOCK` 获取全局读锁而不是表锁，因此它不受 `LOCK TABLES` 和 `UNLOCK TABLES` 关于表锁定和隐式提交的相同行为影响。例如，`START TRANSACTION` 不会释放全局读锁。请参见 Section 15.7.8.3, “FLUSH 语句”。

+   其他隐式导致事务提交的语句不会释放现有的表锁。有关此类语句的列表，请参见 Section 15.3.3, “导致隐式提交的语句”。

+   使用事务表（如`InnoDB`表）的正确方法是使用`SET autocommit = 0`（而不是`START TRANSACTION`）开始事务，然后使用`LOCK TABLES`，直到显式提交事务之前不要调用`UNLOCK TABLES`。例如，如果你需要向表`t1`写入并从表`t2`读取，你可以这样做：

    ```sql
    SET autocommit=0;
    LOCK TABLES t1 WRITE, t2 READ, ...;
    *... do something with tables t1 and t2 here ...* COMMIT;
    UNLOCK TABLES;
    ```

    当你调用`LOCK TABLES`时，`InnoDB`内部会获取自己的表锁，而 MySQL 会获取自己的表锁。`InnoDB`在下一次提交时释放其内部表锁，但要释放 MySQL 的表锁，你必须调用`UNLOCK TABLES`。你不应该设置`autocommit = 1`，因为这样`InnoDB`会在调用`LOCK TABLES`后立即释放其内部表锁，很容易发生死锁。如果设置`autocommit = 1`，`InnoDB`根本不会获取内部表锁，以帮助旧应用程序避免不必要的死锁。

+   `ROLLBACK`不会释放表锁。

#### 表锁和触发器

如果使用`LOCK TABLES`显式锁定一个表，那么触发器中使用的任何表也会隐式锁定：

+   这些锁与使用`LOCK TABLES`语句显式获取的锁同时获取。

+   触发器中使用的表的锁取决于表是否仅用于读取。如果是，那么读取锁就足够了。否则，会使用写入锁。

+   如果一个表被使用`LOCK TABLES`显式锁定以供读取，但由于可能在触发器中被修改而需要写入锁定，那么会获取写入锁定而不是读取锁定。（也就是说，由于表在触发器中的出现导致对表的显式读取锁请求被转换为写入锁请求。）

假设你使用这个语句锁定了两个表，`t1`和`t2`：

```sql
LOCK TABLES t1 WRITE, t2 READ;
```

如果`t1`或`t2`有任何触发器，触发器中使用的表也会被锁定。假设`t1`有一个定义如下的触发器：

```sql
CREATE TRIGGER t1_a_ins AFTER INSERT ON t1 FOR EACH ROW
BEGIN
  UPDATE t4 SET count = count+1
      WHERE id = NEW.id AND EXISTS (SELECT a FROM t3);
  INSERT INTO t2 VALUES(1, 2);
END;
```

`LOCK TABLES`语句的结果是，`t1`和`t2`被锁定，因为它们出现在语句中，而`t3`和`t4`被锁定，因为它们在触发器中使用：

+   根据`WRITE`锁请求，`t1`被锁定为写入。

+   即使请求是读取锁，`t2`也被锁定为写入。这是因为在触发器内插入了`t2`，所以读取请求被转换为写入请求。

+   因为只在触发器内部读取，所以`t3`被锁定为只读。

+   因为可能在触发器内更新，所以`t4`被锁定为写入。

#### 表锁定的限制和条件

您可以安全地使用`KILL`来终止等待表锁定的会话。请参阅第 15.7.8.4 节，“KILL 语句”。

不能在存储程序中使用`LOCK TABLES`和`UNLOCK TABLES`。

不能使用`LOCK TABLES`锁定`performance_schema`数据库中的表，除了`setup_*`xxx`*`表。

由`LOCK TABLES`生成的锁的范围是单个 MySQL 服务器。它与 NDB Cluster 不兼容，NDB Cluster 无法跨多个**mysqld**实例强制执行 SQL 级别的锁。您可以在 API 应用程序中强制执行锁定。有关更多信息，请参阅第 25.2.7.10 节，“与多个 NDB Cluster 节点相关的限制”。

在`LOCK TABLES`语句生效期间，禁止以下语句：`CREATE TABLE`，`CREATE TABLE ... LIKE`，`CREATE VIEW`，`DROP VIEW`以及对存储函数、存储过程和事件的 DDL 语句。

对于一些操作，必须访问`mysql`数据库中的系统表。例如，`HELP`语句需要服务器端帮助表的内容，而`CONVERT_TZ()`可能需要读取时区表。服务器会根据需要隐式锁定系统表以进行读取，因此您无需显式锁定它们。这些表被视为刚刚描述的方式：

```sql
mysql.help_category
mysql.help_keyword
mysql.help_relation
mysql.help_topic
mysql.time_zone
mysql.time_zone_leap_second
mysql.time_zone_name
mysql.time_zone_transition
mysql.time_zone_transition_type
```

如果您想要在任何表格上明确放置一个`WRITE`锁定，并使用`LOCK TABLES`语句，那么该表格必须是唯一被锁定的；没有其他表格可以与相同语句一起被锁定。

通常情况下，您不需要锁定表格，因为所有单个`UPDATE`语句都是原子的；没有其他会话可以干扰任何其他当前执行的 SQL 语句。然而，在一些情况下，锁定表格可能会带来优势：

+   如果您将在一组`MyISAM`表上运行许多操作，锁定您将要使用的表格会更快。锁定`MyISAM`表格会加快在其上插入、更新或删除的速度，因为 MySQL 在调用`UNLOCK TABLES`之前不会刷新被锁定表格的键缓存。通常情况下，每个 SQL 语句后都会刷新键缓存。

    锁定表格的缺点是，没有会话可以更新一个`READ`-锁定的表格（包括持有锁的表格），也没有会话可以访问一个`WRITE`-锁定的表格，除了持有锁的表格。

+   如果您正在使用非事务性存储引擎来创建表格，如果您希望确保在`SELECT`和`UPDATE`之间没有其他会话修改表格，您必须使用`LOCK TABLES`。这里展示的示例需要使用`LOCK TABLES`来安全执行：

    ```sql
    LOCK TABLES trans READ, customer WRITE;
    SELECT SUM(value) FROM trans WHERE customer_id=*some_id*;
    UPDATE customer
      SET total_value=*sum_from_previous_statement*
      WHERE customer_id=*some_id*;
    UNLOCK TABLES;
    ```

    没有`LOCK TABLES`，可能会导致另一个会话在执行`SELECT`和`UPDATE`语句之间插入新行到`trans`表中。

在许多情况下，您可以通过使用相对更新（`UPDATE customer SET *`value`*=*`value`*+*`new_value`*`）或`LAST_INSERT_ID()`函数来避免使用`LOCK TABLES`。

在某些情况下，您也可以通过使用用户级别的咨询锁函数`GET_LOCK()`和`RELEASE_LOCK()`来避免锁定表格。这些锁保存在服务器的哈希表中，并且使用`pthread_mutex_lock()`和`pthread_mutex_unlock()`实现高速。请参见第 14.14 节，“锁定函数”。

有关锁定策略的更多信息，请参见第 10.11.1 节，“内部锁定方法”。
