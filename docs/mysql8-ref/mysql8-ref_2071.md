> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-data-locks-table.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-data-locks-table.html)

#### 29.12.13.1 数据锁表

`data_locks`表显示已持有和请求的数据锁。有关哪些锁请求被哪些持有的锁阻塞的信息，请参见 Section 29.12.13.2, “The data_lock_waits Table”。

示例数据锁信息：

```sql
mysql> SELECT * FROM performance_schema.data_locks\G
*************************** 1\. row ***************************
               ENGINE: INNODB
       ENGINE_LOCK_ID: 139664434886512:1059:139664350547912
ENGINE_TRANSACTION_ID: 2569
            THREAD_ID: 46
             EVENT_ID: 12
        OBJECT_SCHEMA: test
          OBJECT_NAME: t1
       PARTITION_NAME: NULL
    SUBPARTITION_NAME: NULL
           INDEX_NAME: NULL
OBJECT_INSTANCE_BEGIN: 139664350547912
            LOCK_TYPE: TABLE
            LOCK_MODE: IX
          LOCK_STATUS: GRANTED
            LOCK_DATA: NULL
*************************** 2\. row ***************************
               ENGINE: INNODB
       ENGINE_LOCK_ID: 139664434886512:2:4:1:139664350544872
ENGINE_TRANSACTION_ID: 2569
            THREAD_ID: 46
             EVENT_ID: 12
        OBJECT_SCHEMA: test
          OBJECT_NAME: t1
       PARTITION_NAME: NULL
    SUBPARTITION_NAME: NULL
           INDEX_NAME: GEN_CLUST_INDEX
OBJECT_INSTANCE_BEGIN: 139664350544872
            LOCK_TYPE: RECORD
            LOCK_MODE: X
          LOCK_STATUS: GRANTED
            LOCK_DATA: supremum pseudo-record
```

与大多数性能模式数据收集不同，没有用于控制是否收集数据锁信息或用于控制数据锁表大小的仪器。性能模式收集服务器中已经可用的信息，因此生成此信息不会产生内存或 CPU 开销，也不需要控制其收集的参数。

使用`data_locks`表来帮助诊断在高并发负载时发生的性能问题。对于`InnoDB`，请参阅 Section 17.15.2, “InnoDB INFORMATION_SCHEMA Transaction and Locking Information”中关于此主题的讨论。

`data_locks`表具有以下列：

+   `ENGINE`

    持有或请求锁的存储引擎。

+   `ENGINE_LOCK_ID`

    存储引擎持有或请求的锁的 ID。(`ENGINE_LOCK_ID`, `ENGINE`)值的元组是唯一的。

    锁 ID 格式是内部的，并随时可能更改。应用程序不应依赖于锁 ID 具有特定格式。

+   `ENGINE_TRANSACTION_ID`

    请求锁的事务的存储引擎内部 ID。这可以被视为锁的所有者，尽管锁可能仍处于挂起状态，实际上尚未被授予（`LOCK_STATUS='WAITING'`）。

    如果事务尚未执行任何写操作（仍被视为只读），则该列包含用户不应尝试解释的内部数据。否则，该列是事务 ID。

    对于`InnoDB`，要获取有关事务的详细信息，请将此列与`INFORMATION_SCHEMA` `INNODB_TRX`表的`TRX_ID`列连接。

+   `THREAD_ID`

    创建锁的会话的线程 ID。要获取有关线程的详细信息，请将此列与性能模式`threads`表的`THREAD_ID`列连接。

    `THREAD_ID`可以与`EVENT_ID`一起使用，以确定在内存中创建锁数据结构的事件。 （如果数据结构用于存储多个锁，则此事件可能发生在此特定锁请求发生之前。）

+   `EVENT_ID`

    导致锁定的性能模式事件。(`THREAD_ID`, `EVENT_ID`)值的元组在其他性能模式表中隐式标识父事件：

    +   `events_waits_*`xxx`*`表中的父等待事件

    +   `events_stages_*`xxx`*`表中的父阶段事件

    +   `events_statements_*`xxx`*`表中的父语句事件

    +   `events_transactions_current`表中的父事务事件

    要获取有关父事件的详细信息，请将`THREAD_ID`和`EVENT_ID`列与适当的父事件表中同名的列进行连接。参见 Section 29.19.2, “Obtaining Parent Event Information”。

+   `OBJECT_SCHEMA`

    包含被锁定表的模式。

+   `OBJECT_NAME`

    被锁定表的名称。

+   `PARTITION_NAME`

    被锁定的分区的名称，如果有的话；否则为`NULL`。

+   `SUBPARTITION_NAME`

    被锁定的子分区的名称，如果有的话；否则为`NULL`。

+   `INDEX_NAME`

    被锁定的索引的名称，如果有的话；否则为`NULL`。

    在实践中，`InnoDB`总是创建一个索引（`GEN_CLUST_INDEX`），因此对于`InnoDB`表，`INDEX_NAME`不是`NULL`。

+   `OBJECT_INSTANCE_BEGIN`

    锁的内存地址。

+   `LOCK_TYPE`

    锁的类型。

    该值取决于存储引擎。对于`InnoDB`，允许的值是`RECORD`（行级锁）和`TABLE`（表级锁）。

+   `LOCK_MODE`

    锁是如何请求的。

    该值取决于存储引擎。对于`InnoDB`，允许的值是`S[,GAP]`，`X[,GAP]`，`IS[,GAP]`，`IX[,GAP]`，`AUTO_INC`和`UNKNOWN`。 除了`AUTO_INC`和`UNKNOWN`之外的锁模式表示存在间隙锁。 有关`S`，`X`，`IS`，`IX`和间隙锁的信息，请参阅 Section 17.7.1, “InnoDB Locking”。

+   `LOCK_STATUS`

    锁请求的状态。

    该值取决于存储引擎。对于`InnoDB`，允许的值是`GRANTED`（持有锁）和`WAITING`（正在等待锁）。

+   `LOCK_DATA`

    如果有锁相关的数据，则显示该数据。该值取决于存储引擎。对于`InnoDB`，如果`LOCK_TYPE`是`RECORD`，则显示一个值，否则该值为`NULL`。对于放置在主键索引上的锁，显示被锁定记录的主键值。对于放置在辅助索引上的锁，显示附加的主键值。如果没有主键，`LOCK_DATA`显示所选唯一索引的键值或唯一的`InnoDB`内部行 ID 号，根据`InnoDB`聚簇索引使用规则（参见 Section 17.6.2.1, “Clustered and Secondary Indexes”）。对于在伪记录上放置的锁，`LOCK_DATA`报告“supremum 伪记录”。如果包含被锁定记录的页面不在缓冲池中，因为在持有锁时将其写入磁盘，`InnoDB`不会从磁盘获取页面。相反，`LOCK_DATA`报告`NULL`。

`data_locks`表具有以下索引：

+   主键在(`ENGINE_LOCK_ID`, `ENGINE`)

+   索引在(`ENGINE_TRANSACTION_ID`, `ENGINE`)

+   索引在(`THREAD_ID`, `EVENT_ID`)

+   索引在(`OBJECT_SCHEMA`, `OBJECT_NAME`, `PARTITION_NAME`, `SUBPARTITION_NAME`)

`TRUNCATE TABLE`不允许用于`data_locks`表。

注意

在 MySQL 8.0.1 之前，类似于性能模式`data_locks`表中的信息可在`INFORMATION_SCHEMA.INNODB_LOCKS`表中找到，该表提供有关每个`InnoDB`事务请求但尚未获取的锁以及每个由阻止另一个事务的事务持有的锁的信息。`INFORMATION_SCHEMA.INNODB_LOCKS`已被弃用，并在 MySQL 8.0.1 中移除。应改用`data_locks`。

`INNODB_LOCKS`和`data_locks`之间的区别：

+   如果一个事务持有锁，`INNODB_LOCKS`仅在另一个事务正在等待时显示该锁。`data_locks`无论是否有事务在等待，都会显示该锁。

+   `data_locks`表没有与`LOCK_SPACE`、`LOCK_PAGE`或`LOCK_REC`对应的列。

+   `INNODB_LOCKS` 表需要全局 `PROCESS` 权限。`data_locks` 表需要通常的 Performance Schema 权限，在表上具有 `SELECT` 权限才能被选择。

以下表格显示了从 `INNODB_LOCKS` 列到 `data_locks` 列的映射。使用这些信息将应用程序从一个表迁移到另一个表。

**表 29.4 从 INNODB_LOCKS 到 data_locks 列的映射**

| INNODB_LOCKS 列 | data_locks 列 |
| --- | --- |
| `LOCK_ID` | `ENGINE_LOCK_ID` |
| `LOCK_TRX_ID` | `ENGINE_TRANSACTION_ID` |
| `LOCK_MODE` | `LOCK_MODE` |
| `LOCK_TYPE` | `LOCK_TYPE` |
| `LOCK_TABLE`（合并的模式/表名） | `OBJECT_SCHEMA`（模式名），`OBJECT_NAME`（表名） |
| `LOCK_INDEX` | `INDEX_NAME` |
| `LOCK_SPACE` | 无 |
| `LOCK_PAGE` | 无 |
| `LOCK_REC` | 无 |
| `LOCK_DATA` | `LOCK_DATA` |
| INNODB_LOCKS 列 | data_locks 列 |
