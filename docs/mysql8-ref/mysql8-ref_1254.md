> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-information-schema-examples.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-information-schema-examples.html)

#### 17.15.2.1 使用 InnoDB 事务和锁定信息

注意

本节描述了由性能模式`data_locks`和`data_lock_waits`表公开的锁定信息，这些表在 MySQL 8.0 中取代了`INFORMATION_SCHEMA`中的`INNODB_LOCKS`和`INNODB_LOCK_WAITS`表。有关以旧的`INFORMATION_SCHEMA`表为基础的类似讨论，请参阅使用 InnoDB 事务和锁定信息，在 MySQL 5.7 参考手册中。

##### 识别阻塞事务

有时候确定哪个事务阻塞另一个是有帮助的。包含有关`InnoDB`事务和数据锁的信息的表使您能够确定哪个事务正在等待另一个事务，以及正在请求哪个资源。（有关这些表的描述，请参见第 17.15.2 节，“InnoDB INFORMATION_SCHEMA 事务和锁定信息”。）

假设有三个会话同时运行。每个会话对应一个 MySQL 线程，并在另一个之后执行一个事务。考虑当这些会话发出以下语句但尚未提交其事务时系统的状态：

+   会话 A：

    ```sql
    BEGIN;
    SELECT a FROM t FOR UPDATE;
    SELECT SLEEP(100);
    ```

+   会话 B：

    ```sql
    SELECT b FROM t FOR UPDATE;
    ```

+   会话 C：

    ```sql
    SELECT c FROM t FOR UPDATE;
    ```

在这种情况下，使用以下查询查看哪些事务正在等待，哪些事务正在阻塞它们：

```sql
SELECT
  r.trx_id waiting_trx_id,
  r.trx_mysql_thread_id waiting_thread,
  r.trx_query waiting_query,
  b.trx_id blocking_trx_id,
  b.trx_mysql_thread_id blocking_thread,
  b.trx_query blocking_query
FROM       performance_schema.data_lock_waits w
INNER JOIN information_schema.innodb_trx b
  ON b.trx_id = w.blocking_engine_transaction_id
INNER JOIN information_schema.innodb_trx r
  ON r.trx_id = w.requesting_engine_transaction_id;
```

或者更简单地，使用`sys`模式的`innodb_lock_waits`视图：

```sql
SELECT
  waiting_trx_id,
  waiting_pid,
  waiting_query,
  blocking_trx_id,
  blocking_pid,
  blocking_query
FROM sys.innodb_lock_waits;
```

如果阻塞查询报告了 NULL 值，请参阅在发出会话变为空闲后识别阻塞查询。

| 等待 trx id | 等待线程 | 等待查询 | 阻塞 trx id | 阻塞线程 | 阻塞查询 |
| --- | --- | --- | --- | --- | --- |
| `A4` | `6` | `SELECT b FROM t FOR UPDATE` | `A3` | `5` | `SELECT SLEEP(100)` |
| `A5` | `7` | `SELECT c FROM t FOR UPDATE` | `A3` | `5` | `SELECT SLEEP(100)` |
| `A5` | `7` | `SELECT c FROM t FOR UPDATE` | `A4` | `6` | `SELECT b FROM t FOR UPDATE` |

在上表中，您可以通过“等待查询”或“阻塞查询”列来识别会话。正如您所看到的：

+   会话 B（trx id `A4`，线程`6`）和会话 C（trx id `A5`，线程`7`）都在等待会话 A（trx id `A3`，线程`5`）。

+   会话 C 正在等待会话 B 以及会话 A。

你可以在`INFORMATION_SCHEMA`的`INNODB_TRX`表以及性能模式的`data_locks`和`data_lock_waits`表中查看底层数据。

以下表格显示了`INNODB_TRX`表的一些示例内容。

| 事务 ID | 事务状态 | 事务开始时间 | 请求锁 ID | 等待开始时间 | 权重 | MySQL 线程 ID | 查询语句 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `A3` | `RUN­NING` | `2008-01-15 16:44:54` | `NULL` | `NULL` | `2` | `5` | `SELECT SLEEP(100)` |
| `A4` | `LOCK WAIT` | `2008-01-15 16:45:09` | `A4:1:3:2` | `2008-01-15 16:45:09` | `2` | `6` | `SELECT b FROM t FOR UPDATE` |
| `A5` | `LOCK WAIT` | `2008-01-15 16:45:14` | `A5:1:3:2` | `2008-01-15 16:45:14` | `2` | `7` | `SELECT c FROM t FOR UPDATE` |

以下表格显示了`data_locks`表的一些示例内容。

| 锁 ID | 锁事务 ID | 锁模式 | 锁类型 | 锁模式 | 锁表 | 锁索引 | 锁数据 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `A3:1:3:2` | `A3` | `X` | `RECORD` | `test` | `t` | `PRIMARY` | `0x0200` |
| `A4:1:3:2` | `A4` | `X` | `RECORD` | `test` | `t` | `PRIMARY` | `0x0200` |
| `A5:1:3:2` | `A5` | `X` | `RECORD` | `test` | `t` | `PRIMARY` | `0x0200` |

以下表格显示了`data_lock_waits`表的一些示例内容。

| 请求事务 ID | 请求锁 ID | 阻塞事务 ID | 阻塞锁 ID |
| --- | --- | --- | --- |
| `A4` | `A4:1:3:2` | `A3` | `A3:1:3:2` |
| `A5` | `A5:1:3:2` | `A3` | `A3:1:3:2` |
| `A5` | `A5:1:3:2` | `A4` | `A4:1:3:2` |

##### 在发出会话变为空闲后识别阻塞查询

在识别阻塞事务时，如果发出查询的会话已经变为空闲，则会报告阻塞查询的 NULL 值。在这种情况下，使用以下步骤确定阻塞查询：

1.  确定阻塞事务的进程列表 ID。在`sys.innodb_lock_waits`表中，阻塞事务的进程列表 ID 是`blocking_pid`值。

1.  使用`blocking_pid`，查询 MySQL 性能模式的`threads`表以确定阻塞事务的`THREAD_ID`。例如，如果`blocking_pid`为 6，则发出以下查询：

    ```sql
    SELECT THREAD_ID FROM performance_schema.threads WHERE PROCESSLIST_ID = 6;
    ```

1.  使用`THREAD_ID`，查询性能模式`events_statements_current`表以确定线程执行的最后一个查询。例如，如果`THREAD_ID`为 28，则发出此查询：

    ```sql
    SELECT THREAD_ID, SQL_TEXT FROM performance_schema.events_statements_current
    WHERE THREAD_ID = 28\G
    ```

1.  如果线程执行的最后一个查询不足以确定为何保持锁定，则可以查询性能模式`events_statements_history`表，查看线程执行的最后 10 个语句。

    ```sql
    SELECT THREAD_ID, SQL_TEXT FROM performance_schema.events_statements_history
    WHERE THREAD_ID = 28 ORDER BY EVENT_ID;
    ```

##### 将 InnoDB 事务与 MySQL 会话相关联

有时将内部`InnoDB`锁定信息与 MySQL 维护的会话级信息相关联是有用的。例如，您可能想要知道，对于给定的`InnoDB`事务 ID，持有锁定并因此阻止其他事务的 MySQL 会话 ID 和会话名称。

下面来自`INFORMATION_SCHEMA` `INNODB_TRX`表和性能模式`data_locks`和`data_lock_waits`表的输出来自一个负载较重的系统。可以看到，有几个事务正在运行。

下面的`data_locks`和`data_lock_waits`表显示：

+   事务`77F`（执行`INSERT` 和 `INNODB_TRX` 表中显示的查询可能存在不一致。有关解释，请参见 第 17.15.2.3 节，“InnoDB 事务和锁定信息的持久性和一致性”。

以下表格显示了运行重 工作负载 系统的 `PROCESSLIST` 表的内容。

| ID | 用户 | 主机 | 数据库 | 命令 | 时间 | 状态 | 信息 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `384` | `root` | `localhost` | `test` | `Query` | `10` | `update` | `INSERT INTO t2 VALUES …` |
| `257` | `root` | `localhost` | `test` | `Query` | `3` | `update` | `INSERT INTO t2 VALUES …` |
| `130` | `root` | `localhost` | `test` | `Query` | `0` | `update` | `INSERT INTO t2 VALUES …` |
| `61` | `root` | `localhost` | `test` | `Query` | `1` | `update` | `INSERT INTO t2 VALUES …` |
| `8` | `root` | `localhost` | `test` | `Query` | `1` | `update` | `INSERT INTO t2 VALUES …` |
| `4` | `root` | `localhost` | `test` | `Query` | `0` | `preparing` | `SELECT * FROM PROCESSLIST` |
| `2` | `root` | `localhost` | `test` | `Sleep` | `566` |  | `NULL` |

以下表格显示了运行重 工作负载 系统的 `INNODB_TRX` 表的内容。

| 事务 ID | 事务状态 | 事务开始时间 | 事务请求锁 ID | 事务等待开始时间 | 事务权重 | 事务 MySQL 线程 ID | 事务查询 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `77F` | `LOCK WAIT` | `2008-01-15 13:10:16` | `77F` | `2008-01-15 13:10:16` | `1` | `876` | `INSERT INTO t09 (D, B, C) VALUES …` |
| `77E` | `LOCK WAIT` | `2008-01-15 13:10:16` | `77E` | `2008-01-15 13:10:16` | `1` | `875` | `INSERT INTO t09 (D, B, C) VALUES …` |
| `77D` | `LOCK WAIT` | `2008-01-15 13:10:16` | `77D` | `2008-01-15 13:10:16` | `1` | `874` | `INSERT INTO t09 (D, B, C) VALUES …` |
| `77B` | `LOCK WAIT` | `2008-01-15 13:10:16` | `77B:733:12:1` | `2008-01-15 13:10:16` | `4` | `873` | `INSERT INTO t09 (D, B, C) VALUES …` |
| `77A` | `RUN­NING` | `2008-01-15 13:10:16` | `NULL` | `NULL` | `4` | `872` | `SELECT b, c FROM t09 WHERE …` |
| `E56` | `LOCK WAIT` | `2008-01-15 13:10:06` | `E56:743:6:2` | `2008-01-15 13:10:06` | `5` | `384` | `INSERT INTO t2 VALUES …` |
| `E55` | `LOCK WAIT` | `2008-01-15 13:10:06` | `E55:743:38:2` | `2008-01-15 13:10:13` | `965` | `257` | `INSERT INTO t2 VALUES …` |
| `19C` | `RUN­NING` | `2008-01-15 13:09:10` | `NULL` | `NULL` | `2900` | `130` | `INSERT INTO t2 VALUES …` |
| `E15` | `运行中` | `2008-01-15 13:08:59` | `NULL` | `NULL` | `5395` | `61` | `INSERT INTO t2 VALUES …` |
| `51D` | `运行中` | `2008-01-15 13:08:47` | `NULL` | `NULL` | `9807` | `8` | `INSERT INTO t2 VALUES …` |
| 事务标识 | 事务状态 | 事务开始时间 | 事务请求的锁标识 | 事务等待开始时间 | 事务权重 | 事务 MySQL 线程标识 | 事务查询 |

下表显示了运行重 工作负载 系统的 `data_lock_waits` 表的内容。

| 请求事务标识 | 请求的锁标识 | 阻塞事务标识 | 阻塞的锁标识 |
| --- | --- | --- | --- |
| `77F` | `77F:806` | `77E` | `77E:806` |
| `77F` | `77F:806` | `77D` | `77D:806` |
| `77F` | `77F:806` | `77B` | `77B:806` |
| `77E` | `77E:806` | `77D` | `77D:806` |
| `77E` | `77E:806` | `77B` | `77B:806` |
| `77D` | `77D:806` | `77B` | `77B:806` |
| `77B` | `77B:733:12:1` | `77A` | `77A:733:12:1` |
| `E56` | `E56:743:6:2` | `E55` | `E55:743:6:2` |
| `E55` | `E55:743:38:2` | `19C` | `19C:743:38:2` |

下表显示了运行重 工作负载 系统的 `data_locks` 表的内容。

| 锁标识 | 锁事务标识 | 锁模式 | 锁类型 | 锁模式 | 锁表 | 锁索引 | 锁数据 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `77F:806` | `77F` | `AUTO_INC` | `TABLE` | `test` | `t09` | `NULL` | `NULL` |
| `77E:806` | `77E` | `AUTO_INC` | `TABLE` | `test` | `t09` | `NULL` | `NULL` |
| `77D:806` | `77D` | `AUTO_INC` | `TABLE` | `test` | `t09` | `NULL` | `NULL` |
| `77B:806` | `77B` | `AUTO_INC` | `TABLE` | `test` | `t09` | `NULL` | `NULL` |
| `77B:733:12:1` | `77B` | `X` | `RECORD` | `test` | `t09` | `PRIMARY` | `supremum pseudo-record` |
| `77A:733:12:1` | `77A` | `X` | `RECORD` | `test` | `t09` | `PRIMARY` | `supremum pseudo-record` |
| `E56:743:6:2` | `E56` | `S` | `RECORD` | `test` | `t2` | `PRIMARY` | `0, 0` |
| `E55:743:6:2` | `E55` | `X` | `RECORD` | `test` | `t2` | `PRIMARY` | `0, 0` |
| `E55:743:38:2` | `E55` | `S` | `RECORD` | `test` | `t2` | `PRIMARY` | `1922, 1922` |
| `19C:743:38:2` | `19C` | `X` | `RECORD` | `test` | `t2` | `PRIMARY` | `1922, 1922` |
| 锁标识 | 锁事务标识 | 锁模式 | 锁类型 | 锁模式 | 锁表 | 锁索引 | 锁数据 |
