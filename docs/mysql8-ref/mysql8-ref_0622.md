# 10.11.4 元数据锁定

> 原文：[`dev.mysql.com/doc/refman/8.0/en/metadata-locking.html`](https://dev.mysql.com/doc/refman/8.0/en/metadata-locking.html)

MySQL 使用元数据锁定来管理对数据库对象的并发访问，并确保数据一致性。元数据锁定不仅适用于表，还适用于模式、存储程序（过程、函数、触发器、计划事件）、表空间、使用 `GET_LOCK()` 函数获取的用户锁（参见 Section 14.14, “锁定函数”），以及在 Section 7.6.9.1, “锁定服务” 中描述的锁定服务中获取的锁。

Performance Schema `metadata_locks` 表公开了元数据锁信息，可以用于查看哪些会话持有锁，正在等待锁等情况。详情请参见 Section 29.12.13.3, “metadata_locks 表”。

元数据锁定确实会带来一些开销，随着查询量的增加而增加。当多个查询尝试访问相同对象时，元数据争用会增加。

元数据锁定不是表定义缓存的替代品，其互斥体和锁与 `LOCK_open` 互斥体不同。以下讨论提供了有关元数据锁定工作原理的一些信息。

+   元数据锁获取

+   元数据锁释放

#### 元数据锁获取

如果有多个等待者请求同一个锁，则最高优先级的锁请求首先得到满足，与 `max_write_lock_count` 系统变量相关的一个例外。写锁请求比读锁请求具有更高的优先级。但是，如果 `max_write_lock_count` 设置为较低值（比如，10），则读锁请求可能优先于待处理的写锁请求，如果读锁请求已经被忽略以优先处理 10 个写锁请求。通常情况下，这种行为不会发生，因为 `max_write_lock_count` 默认情况下具有非常大的值。

语句逐个获取元数据锁，而不是同时获取，并在此过程中执行死锁检测。

DML 语句通常按照语句中提到表的顺序获取锁。

DDL 语句、`LOCK TABLES`和其他类似语句尝试通过按名称顺序在显式命名的表上获取锁来减少并发 DDL 语句之间可能发生的死锁数量。对于隐式使用的表（例如外键关系中必须锁定的表），锁可能按不同顺序获取。

例如，`RENAME TABLE`是一个按名称顺序获取锁的 DDL 语句：

+   这个`RENAME TABLE`语句将`tbla`重命名为其他内容，并将`tblc`重命名为`tbla`：

    ```sql
    RENAME TABLE tbla TO tbld, tblc TO tbla;
    ```

    该语句按顺序在`tbla`、`tblc`和`tbld`上获取元数据锁（因为`tbld`在名称顺序中跟随`tblc`）。

+   这个稍有不同的语句还将`tbla`重命名为其他内容，并将`tblc`重命名为`tbla`：

    ```sql
    RENAME TABLE tbla TO tblb, tblc TO tbla;
    ```

    在这种情况下，语句按顺序在`tbla`、`tblb`和`tblc`上获取元数据锁（因为`tblb`在名称顺序中位于`tblc`之前）。

两个语句按顺序获取`tbla`和`tblc`上的锁，但在剩余表名的锁是在`tblc`之前还是之后获取有所不同。

当多个事务同时执行时，元数据锁获取顺序可能会影响操作结果，如下例所示。

从具有相同结构的两个表`x`和`x_new`开始。三个客户端发出涉及这些表的语句：

Client 1：

```sql
LOCK TABLE x WRITE, x_new WRITE;
```

该语句按名称顺序请求并获取`x`和`x_new`上的写锁。

Client 2：

```sql
INSERT INTO x VALUES(1);
```

该语句请求并在`x`上等待写锁。

Client 3：

```sql
RENAME TABLE x TO x_old, x_new TO x;
```

该语句按名称顺序请求`x`、`x_new`和`x_old`上的排他锁，但在等待`x`上的锁时被阻塞。

Client 1：

```sql
UNLOCK TABLES;
```

该语句释放了`x`和`x_new`上的写锁。Client 3 对`x`的排他锁请求比 Client 2 的写锁请求优先级更高，因此 Client 3 先获取了`x`的锁，然后也获取了`x_new`和`x_old`的锁，执行重命名操作，然后释放锁。然后 Client 2 获取了`x`的锁，执行插入操作，然后释放锁。

锁获取顺序导致`RENAME TABLE`在`INSERT`之前执行。插入发生的`x`是 Client 2 发出插入时命名为`x_new`，并由 Client 3 重命名为`x`的表：

```sql
mysql> SELECT * FROM x;
+------+
| i    |
+------+
|    1 |
+------+

mysql> SELECT * FROM x_old;
Empty set (0.01 sec)
```

现在改为以名称为`x`和`new_x`的表具有相同结构开始。再次，三个客户端发出涉及这些表的语句：

Client 1：

```sql
LOCK TABLE x WRITE, new_x WRITE;
```

该语句按名称顺序请求并获取`new_x`和`x`上的写锁。

Client 2：

```sql
INSERT INTO x VALUES(1);
```

该语句请求并在`x`上等待写锁。

Client 3：

```sql
RENAME TABLE x TO old_x, new_x TO x;
```

该语句按名称顺序请求`new_x`、`old_x`和`x`上的排他锁，但在等待`new_x`上的锁时被阻塞。

Client 1：

```sql
UNLOCK TABLES;
```

该语句释放了`x`和`new_x`的写锁。对于`x`，唯一的挂起请求来自客户端 2，因此客户端 2 获取其锁，执行插入操作，并释放锁。对于`new_x`，唯一的挂起请求来自客户端 3，允许其获取该锁（以及`old_x`的锁）。重命名操作仍然会因为在客户端 2 插入完成并释放其锁之前对`x`的锁而阻塞。然后客户端 3 获取`x`的锁，执行重命名操作，并释放其锁。

在这种情况下，锁获取顺序导致`INSERT`在`RENAME TABLE`之前执行。插入发生的`x`是原始的`x`，现在被重命名为`old_x`：

```sql
mysql> SELECT * FROM x;
Empty set (0.01 sec)

mysql> SELECT * FROM old_x;
+------+
| i    |
+------+
|    1 |
+------+
```

如果并发语句中锁获取顺序对操作结果有影响，如前面的例子，您可以调整表名以影响锁获取顺序。

为了防止相关表上的冲突 DML 和 DDL 操作同时执行，元数据锁会根据外键约束扩展到相关表。在更新父表时，会在更新外键元数据时在子表上获取元数据锁。外键元数据由子表拥有。

#### 元数据锁释放

为确保事务的可串行化，服务器不得允许一个会话在另一个会话中的未完成的显式或隐式启动的事务中对表执行数据定义语言（DDL）语句。服务器通过在事务中使用的表上获取元数据锁并推迟释放这些锁直到事务结束来实现这一点。表上的元数据锁阻止对表结构的更改。这种锁定方法意味着一个会话中正在使用的表在事务结束之前不能被其他会话用于 DDL 语句。

这个原则不仅适用于事务表，也适用于非事务表。假设一个会话开始一个事务，使用事务表`t`和非事务表`nt`如下：

```sql
START TRANSACTION;
SELECT * FROM t;
SELECT * FROM nt;
```

服务器在事务结束之前会持有表`t`和`nt`的元数据锁。如果另一个会话尝试在任一表上执行 DDL 或写锁操作，它会阻塞，直到事务结束时释放元数据锁。例如，如果第二个会话尝试执行以下任何操作，它会被阻塞：

```sql
DROP TABLE t;
ALTER TABLE t ...;
DROP TABLE nt;
ALTER TABLE nt ...;
LOCK TABLE t ... WRITE;
```

相同的行为也适用于`LOCK TABLES ... READ`。也就是说，显式或隐式启动的更新任何表（事务或非事务）的事务会被`LOCK TABLES ... READ`阻塞，并且被该表阻塞。

如果服务器为一个在语法上有效但在执行过程中失败的语句获取元数据锁，它不会提前释放锁。锁的释放仍然延迟到事务结束，因为失败的语句被写入二进制日志，而锁保护日志的一致性。

在自动提交模式下，每个语句实际上是一个完整的事务，因此为语句获取的元数据锁仅在语句结束时保持。

在`PREPARE`语句期间获取的元数据锁在语句准备完成后会被释放，即使准备过程发生在多语句事务中。

截至 MySQL 8.0.13 版本，对于处于`PREPARED`状态的 XA 事务，元数据锁在客户端断开连接和服务器重新启动时会被保留，直到执行`XA COMMIT`或`XA ROLLBACK`。
