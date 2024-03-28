> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-data-lock-waits-table.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-data-lock-waits-table.html)

#### 29.12.13.2 数据锁等待表

`data_lock_waits`表实现了一个多对多的关系，显示了`data_locks`表中的哪些数据锁请求被`data_locks`表中的哪些持有数据锁所阻塞。在`data_lock_waits`表中，只有当持有的锁阻止某些锁请求时，`data_locks`中的持有锁才会出现。

这些信息可以帮助您了解会话之间的数据锁依赖关系。该表不仅展示了会话或事务正在等待的锁，还展示了当前持有该锁的会话或事务。

数据锁等待信息示例：

```sql
mysql> SELECT * FROM performance_schema.data_lock_waits\G
*************************** 1\. row ***************************
                          ENGINE: INNODB
       REQUESTING_ENGINE_LOCK_ID: 140211201964816:2:4:2:140211086465800
REQUESTING_ENGINE_TRANSACTION_ID: 1555
            REQUESTING_THREAD_ID: 47
             REQUESTING_EVENT_ID: 5
REQUESTING_OBJECT_INSTANCE_BEGIN: 140211086465800
         BLOCKING_ENGINE_LOCK_ID: 140211201963888:2:4:2:140211086459880
  BLOCKING_ENGINE_TRANSACTION_ID: 1554
              BLOCKING_THREAD_ID: 46
               BLOCKING_EVENT_ID: 12
  BLOCKING_OBJECT_INSTANCE_BEGIN: 140211086459880
```

与大多数性能模式数据收集不同，没有用于控制是否收集数据锁信息或用于控制数据锁表大小的系统变量。性能模式收集服务器中已经可用的信息，因此生成此信息不会产生内存或 CPU 开销，也不需要控制其收集的参数。

使用[`data_lock_waits`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-data-lock-waits-table.html)表来帮助诊断在高并发负载时发生的性能问题。对于`InnoDB`，请参阅 Section 17.15.2, “InnoDB INFORMATION_SCHEMA Transaction and Locking Information”中关于此主题的讨论。

因为`data_lock_waits`表中的列与`data_locks`表中的列类似，所以这里的列描述是缩写的。有关更详细的列描述，请参阅 Section 29.12.13.1, “The data_locks Table”。

`data_lock_waits`表具有以下列：

+   `ENGINE`

    请求锁的存储引擎。

+   `REQUESTING_ENGINE_LOCK_ID`

    存储引擎请求的锁的 ID。要获取有关锁的详细信息，请将此列与`data_locks`表的`ENGINE_LOCK_ID`列进行连接。

+   `REQUESTING_ENGINE_TRANSACTION_ID`

    请求锁的事务的存储引擎内部 ID。

+   `REQUESTING_THREAD_ID`

    请求锁的会话的线程 ID。

+   `REQUESTING_EVENT_ID`

    导致请求锁的会话中的锁请求的性能模式事件。

+   `REQUESTING_OBJECT_INSTANCE_BEGIN`

    请求锁的内存地址。

+   `BLOCKING_ENGINE_LOCK_ID`

    阻塞锁的 ID。要获取有关锁的详细信息，请将此列与 `data_locks` 表的 `ENGINE_LOCK_ID` 列进行连接。

+   `BLOCKING_ENGINE_TRANSACTION_ID`

    持有阻塞锁的事务的存储引擎内部 ID。

+   `BLOCKING_THREAD_ID`

    持有阻塞锁的会话的线程 ID。

+   `BLOCKING_EVENT_ID`

    导致持有阻塞锁的会话中的性能模式事件。

+   `BLOCKING_OBJECT_INSTANCE_BEGIN`

    阻塞锁的内存地址。

`data_lock_waits` 表具有以下索引：

+   索引为 (`REQUESTING_ENGINE_LOCK_ID`, `ENGINE`)

+   索引为 (`BLOCKING_ENGINE_LOCK_ID`, `ENGINE`)

+   索引为 (`REQUESTING_ENGINE_TRANSACTION_ID`, `ENGINE`)

+   索引为 (`BLOCKING_ENGINE_TRANSACTION_ID`, `ENGINE`)

+   索引为 (`REQUESTING_THREAD_ID`, `REQUESTING_EVENT_ID`)

+   索引为 (`BLOCKING_THREAD_ID`, `BLOCKING_EVENT_ID`)

`TRUNCATE TABLE` 不允许用于 `data_lock_waits` 表。

注意

在 MySQL 8.0.1 之前，类似于性能模式 `data_lock_waits` 表中的信息可在 `INFORMATION_SCHEMA.INNODB_LOCK_WAITS` 表中找到，该表提供有关每个被阻塞的 `InnoDB` 事务的信息，指示其请求的锁以及阻止该请求的任何锁。`INFORMATION_SCHEMA.INNODB_LOCK_WAITS` 已被弃用，并在 MySQL 8.0.1 中移除。应改用 `data_lock_waits`。

这两个表在所需的权限上有所不同：`INNODB_LOCK_WAITS` 表需要全局 `PROCESS` 权限。`data_lock_waits` 表需要通常的从表中选择的性能模式权限 `SELECT`。

下表显示了从 `INNODB_LOCK_WAITS` 列到 `data_lock_waits` 列的映射。使用此信息将应用程序从一个表迁移到另一个表。

**表 29.5 从 INNODB_LOCK_WAITS 到 data_lock_waits 列的映射**

| INNODB_LOCK_WAITS 列 | data_lock_waits 列 |
| --- | --- |
| `REQUESTING_TRX_ID` | `REQUESTING_ENGINE_TRANSACTION_ID` |
| `REQUESTED_LOCK_ID` | `REQUESTING_ENGINE_LOCK_ID` |
| `BLOCKING_TRX_ID` | `BLOCKING_ENGINE_TRANSACTION_ID` |
| `BLOCKING_LOCK_ID` | `BLOCKING_ENGINE_LOCK_ID` |
