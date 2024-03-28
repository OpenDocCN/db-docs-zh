> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-cluster-locks.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-cluster-locks.html)

#### 25.6.16.6 `ndbinfo cluster_locks`表

`cluster_locks`表提供有关 NDB 集群中`NDB`表上持有和等待锁的当前锁请求的信息，并且旨在作为`cluster_operations`的伴随表。从`cluster_locks`表中获取的信息可能有助于调查停顿和死锁。

`cluster_locks`表包含以下列：

+   `node_id`

    报告节点的 ID

+   `block_instance`

    报告 LDM 实例的 ID

+   `tableid`

    包含此行的表的 ID

+   `fragmentid`

    包含锁定行的片段的 ID

+   `rowid`

    锁定行的 ID

+   `transid`

    事务 ID

+   `mode`

    锁请求模式

+   `state`

    锁定状态

+   `detail`

    是否是排队中第一个持有锁的

+   `op`

    操作类型

+   `duration_millis`

    等待或持有锁的毫秒数

+   `lock_num`

    锁对象的 ID

+   `waiting_for`

    等待具有此 ID 的锁

##### 备注

表 ID（`tableid`列）是内部分配的，与其他`ndbinfo`表中使用的相同。它也显示在**ndb_show_tables**的输出中。

事务 ID（`transid`列）是由 NDB API 为请求或持有当前锁的事务生成的标识符。

`mode`列显示锁模式；始终是`S`（表示共享锁）或`X`（独占锁）。如果事务对给定行持有独占锁，则该行上的所有其他锁都具有相同的事务 ID。

`state`列显示锁状态。其值始终是`H`（持有）或`W`（等待）。等待的锁请求等待由不同事务持有的锁。

当`detail`列包含`*`（星号字符）时，这意味着这个锁是受影响行锁队列中第一个持有的锁；否则，此列为空。此信息可用于帮助识别锁请求列表中的唯一条目。

`op`列显示请求锁的操作类型。始终是`READ`、`INSERT`、`UPDATE`、`DELETE`、`SCAN`或`REFRESH`中的一个值。

`duration_millis`列显示此锁请求等待或持有锁的毫秒数。当为等待请求授予锁时，此值将重置为 0。

锁 ID（`lockid`列）对于此节点和块实例是唯一的。

锁定状态显示在`lock_state`列中；如果是`W`，则表示锁正在等待授予，而`waiting_for`列显示此请求正在等待的锁对象的锁 ID。否则，`waiting_for`列为空。`waiting_for`只能指向同一行上的锁，由`node_id`、`block_instance`、`tableid`、`fragmentid`和`rowid`标识。
