# 17.7.1 InnoDB 锁定

> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-locking.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-locking.html)

本节描述了 `InnoDB` 使用的锁类型。

+   共享锁和独占锁

+   意向锁

+   记录锁

+   间隙锁

+   Next-Key 锁

+   插入意向锁

+   自增锁

+   空间索引的谓词锁

#### 共享锁和独占锁

`InnoDB` 实现标准的行级锁定，其中有两种类型的锁，共享 (`S`) 锁 和 独占 (`X`) 锁。

+   一个共享 (`S`) 锁 允许持有锁的事务读取一行。

+   一个独占 (`X`) 锁允许持有锁的事务更新或删除一行。

如果事务 `T1` 持有行 `r` 上的共享 (`S`) 锁，则来自某个不同事务 `T2` 对行 `r` 的锁的请求处理如下：

+   `T2` 请求 `S` 锁可以立即被授予。因此，`T1` 和 `T2` 都持有 `r` 上的 `S` 锁。

+   `T2` 对 `r` 的 `X` 锁请求无法立即被授予。

如果事务 `T1` 持有行 `r` 上的独占 (`X`) 锁，则来自某个不同事务 `T2` 对 `r` 上的任一类型锁的请求无法立即被授予。相反，事务 `T2` 必须等待事务 `T1` 释放 `r` 上的锁。

#### 意向锁

`InnoDB` 支持*多粒度锁定*，允许行锁和表锁共存。例如，像`LOCK TABLES ... WRITE` 这样的语句在指定的表上获取一个独占锁（`X` 锁）。为了使多粒度级别的锁定实用，`InnoDB` 使用意向锁。意向锁是表级锁，指示事务后续需要对表中的行请求哪种类型的锁（共享或独占）。有两种类型的意向锁：

+   一个意向共享锁 (`IS`) 表示事务打算在表中的各个行上设置*共享*锁。

+   意向排他锁（`IX`）表示一个事务打算在表中的单个行上设置排他锁。

例如，`SELECT ... FOR SHARE`设置一个`IS`锁，而`SELECT ... FOR UPDATE`设置一个`IX`锁。

意向锁定协议如下：

+   在事务可以在表中的行上获取共享锁之前，必须首先在表上获取一个`IS`锁或更强的锁。

+   在事务可以在表中的行上获取排他锁之前，必须首先在表上获取一个`IX`锁。

表级锁类型兼容性总结如下矩阵。

|  | `X` | `IX` | `S` | `IS` |
| --- | --- | --- | --- | --- |
| `X` | 冲突 | 冲突 | 冲突 | 冲突 |
| `IX` | 冲突 | 兼容 | 冲突 | 兼容 |
| `S` | 冲突 | 冲突 | 兼容 | 兼容 |
| `IS` | 冲突 | 兼容 | 兼容 | 兼容 |

如果请求的事务与现有锁兼容，则授予锁，但如果与现有锁冲突，则不授予锁。事务会等待，直到冲突的现有锁被释放。如果锁请求与现有锁冲突，并且由于会导致死锁而无法授予锁，则会发生错误。

意向锁不会阻塞任何东西，除了完整的表请求（例如，`LOCK TABLES ... WRITE`）。意向锁的主要目的是显示某人正在锁定一行，或者将要在表中锁定一行。

意向锁的事务数据在`SHOW ENGINE INNODB STATUS`和 InnoDB monitor 输出中看起来类似于以下内容：

```sql
TABLE LOCK table `test`.`t` trx id 10080 lock mode IX
```

#### 记录锁

记录锁是对索引记录的锁定。例如，`SELECT c1 FROM t WHERE c1 = 10 FOR UPDATE;`阻止任何其他事务插入、更新或删除`c1`值为`10`的行。

记录锁总是锁定索引记录，即使一个表被定义为没有索引。对于这种情况，`InnoDB`会创建一个隐藏的聚簇索引，并将此索引用于记录锁定。参见 Section 17.6.2.1, “Clustered and Secondary Indexes”。

记录锁的事务数据在`SHOW ENGINE INNODB STATUS`和 InnoDB monitor 输出中看起来类似于以下内容：

```sql
RECORD LOCKS space id 58 page no 3 n bits 72 index `PRIMARY` of table `test`.`t`
trx id 10078 lock_mode X locks rec but not gap
Record lock, heap no 2 PHYSICAL RECORD: n_fields 3; compact format; info bits 0
 0: len 4; hex 8000000a; asc     ;;
 1: len 6; hex 00000000274f; asc     'O;;
 2: len 7; hex b60000019d0110; asc        ;;
```

#### 间隙锁

间隙锁是索引记录之间的间隙上的锁，或者是第一个索引记录之前或最后一个索引记录之后的间隙上的锁。例如，`SELECT c1 FROM t WHERE c1 BETWEEN 10 and 20 FOR UPDATE;`阻止其他事务将值`15`插入到列`t.c1`中，无论该列中是否已经存在任何这样的值，因为范围内所有现有值之间的间隙都被锁定。

一个间隙可能跨越单个索引值，多个索引值，甚至为空。

间隙锁是性能和并发性之间的权衡的一部分，并且在某些事务隔离级别中使用，在其他事务隔离级别中不使用。

使用唯一索引锁定行以搜索唯一行的语句不需要间隙锁定。（这不包括搜索条件仅包含多列唯一索引的某些列的情况；在这种情况下，确实会发生间隙锁定。）例如，如果`id`列有一个唯一索引，下面的语句仅对具有`id`值 100 的行使用索引记录锁定，而其他会话是否在前面的间隙中插入行并不重要：

```sql
SELECT * FROM child WHERE id = 100;
```

如果`id`没有被索引或具有非唯一索引，则该语句确实锁定了前面的间隙。

还值得注意的是，不同事务可以在同一间隙上持有冲突的锁。例如，事务 A 可以在一个间隙上持有共享间隙锁（间隙 S 锁），而事务 B 可以在同一间隙上持有独占间隙锁（间隙 X 锁）。允许冲突的间隙锁的原因是，如果从索引中清除记录，则不同事务持有的记录上的间隙锁必须合并。

在`InnoDB`中，间隙锁是“纯粹抑制性的”，这意味着它们的唯一目的是防止其他事务向间隙中插入数据。间隙锁可以共存。一个事务获取的间隙锁不会阻止另一个事务在同一间隙上获取间隙锁。共享间隙锁和独占间隙锁之间没有区别。它们不会相互冲突，执行相同的功能。

间隙锁定可以被显式禁用。如果将事务隔离级别更改为`READ COMMITTED`，则会发生这种情况。在这种情况下，间隙锁定对搜索和索引扫描被禁用，仅用于外键约束检查和重复键检查。

使用`READ COMMITTED`隔离级别还有其他影响。对于不匹配行的记录锁在 MySQL 评估完`WHERE`条件后被释放。对于`UPDATE`语句，`InnoDB`执行“半一致性”读取，以便将最新提交的版本返回给 MySQL，以便 MySQL 可以确定行是否与`UPDATE`的`WHERE`条件匹配。

#### 下一个键锁

下一个键锁是在索引记录上设置记录锁和在索引记录之前的间隙上设置间隙锁的组合。

`InnoDB` 以一种使得当搜索或扫描表索引时，在遇到的索引记录上设置共享或排他锁的方式执行行级锁定。因此，行级锁实际上是索引记录锁。对索引记录的下一个键锁也会影响该索引记录之前的“间隙”。也就是说，下一个键锁是索引记录锁加上索引记录之前的间隙锁。如果一个会话在索引中的记录 `R` 上有共享或排他锁，则另一个会话不能在索引顺序中的 `R` 紧前的间隙中插入新的索引记录。

假设一个索引包含值 10、11、13 和 20。对于该索引的可能的下一个键锁覆盖以下区间，其中圆括号表示排除区间端点，方括号表示包含端点：

```sql
(negative infinity, 10]
(10, 11]
(11, 13]
(13, 20]
(20, positive infinity)
```

对于最后一个区间，下一个键锁锁定了索引中最大值上方的间隙和具有高于索引中任何实际值的“最大值”伪记录。最大值不是真正的索引记录，因此，实际上，这个下一个键锁只锁定了紧随最大索引值后的间隙。

默认情况下，`InnoDB` 在 `REPEATABLE READ` 事务隔离级别下运行。在这种情况下，`InnoDB` 对搜索和索引扫描使用下一个键锁，以防止幻影行（参见 第 17.7.4 节，“幻影行”）。

下一个键锁的事务数据在 `SHOW ENGINE INNODB STATUS` 和 InnoDB 监视器 输出中类似于以下内容：

```sql
RECORD LOCKS space id 58 page no 3 n bits 72 index `PRIMARY` of table `test`.`t`
trx id 10080 lock_mode X
Record lock, heap no 1 PHYSICAL RECORD: n_fields 1; compact format; info bits 0
 0: len 8; hex 73757072656d756d; asc supremum;;

Record lock, heap no 2 PHYSICAL RECORD: n_fields 3; compact format; info bits 0
 0: len 4; hex 8000000a; asc     ;;
 1: len 6; hex 00000000274f; asc     'O;;
 2: len 7; hex b60000019d0110; asc        ;;
```

#### 插入意向锁

插入意向锁是在 `INSERT` 操作插入行之前设置的一种间隙锁。此锁表示以一种插入的意图插入，即使多个插入到相同索引间隙的事务不需要等待对方，如果它们不是在间隙内的相同位置插入。假设存在值为 4 和 7 的索引记录。试图分别插入值为 5 和 6 的单独事务，在获取插入行的排他锁之前，都会在值为 4 和 7 之间的间隙上设置插入意向锁，但不会相互阻塞，因为这些行是非冲突的。

以下示例演示了一个事务在获取插入记录的排他锁之前获取插入意向锁。该示例涉及两个客户端，A 和 B。

客户端 A 创建一个包含两个索引记录（90 和 102）的表，然后启动一个事务，对 ID 大于 100 的索引记录进行排他锁定。在记录 102 之前，排他锁包括一个间隙锁：

```sql
mysql> CREATE TABLE child (id int(11) NOT NULL, PRIMARY KEY(id)) ENGINE=InnoDB;
mysql> INSERT INTO child (id) values (90),(102);

mysql> START TRANSACTION;
mysql> SELECT * FROM child WHERE id > 100 FOR UPDATE;
+-----+
| id  |
+-----+
| 102 |
+-----+
```

客户端 B 开始一个事务，向间隙中插入一条记录。该事务在等待获取排他锁时会获取插入意向锁。

```sql
mysql> START TRANSACTION;
mysql> INSERT INTO child (id) VALUES (101);
```

插入意向锁的事务数据在 `SHOW ENGINE INNODB STATUS` 和 InnoDB 监视器 输出中类似于以下内容：

```sql
RECORD LOCKS space id 31 page no 3 n bits 72 index `PRIMARY` of table `test`.`child`
trx id 8731 lock_mode X locks gap before rec insert intention waiting
Record lock, heap no 3 PHYSICAL RECORD: n_fields 3; compact format; info bits 0
 0: len 4; hex 80000066; asc    f;;
 1: len 6; hex 000000002215; asc     " ;;
 2: len 7; hex 9000000172011c; asc     r  ;;...
```

#### AUTO-INC 锁

`AUTO-INC` 锁是由插入具有 `AUTO_INCREMENT` 列的表的事务获取的特殊表级锁。在最简单的情况下，如果一个事务正在向表中插入值，任何其他事务必须等待进行自己的插入，以便由第一个事务插入的行接收连续的主键值。

`innodb_autoinc_lock_mode` 变量控制自增锁定的算法。它允许您选择如何在可预测的自增值序列和插入操作的最大并发性之间进行权衡。

更多信息，请参见 第 17.6.1.6 节，“InnoDB 中的 AUTO_INCREMENT 处理”。

#### 空间索引的谓词锁

`InnoDB` 支持包含空间数据的列的 `SPATIAL` 索引（参见 第 13.4.9 节，“优化空间分析”）。

为处理涉及 `SPATIAL` 索引的操作的锁定，下一个键锁定不适合支持 `REPEATABLE READ` 或 `SERIALIZABLE` 事务隔离级别。在多维数据中没有绝对排序概念，因此不清楚哪个是“下一个”键。

为了支持具有 `SPATIAL` 索引的表的隔离级别，`InnoDB` 使用谓词锁。`SPATIAL` 索引包含最小边界矩形（MBR）值，因此 `InnoDB` 通过在查询中使用的 MBR 值上设置谓词锁来强制对索引进行一致读取。其他事务无法插入或修改与查询条件匹配的行。
