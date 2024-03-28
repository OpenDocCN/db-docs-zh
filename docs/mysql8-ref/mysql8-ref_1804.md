> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-server-locks.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-server-locks.html)

#### 25.6.16.53 ndbinfo server_locks 表

`server_locks`表的结构类似于`cluster_locks`表，并提供后者中找到的信息的子集，但是特定于 SQL 节点（MySQL 服务器）的信息（`cluster_locks`表提供有关集群中所有锁的信息）。更确切地说，`server_locks`包含由属于当前**mysqld**实例的线程请求的锁的信息，并作为`server_operations`的伴随表。这可能有助于将锁定模式与特定的 MySQL 用户会话、查询或用例相关联。

`server_locks`表包含以下列：

+   `mysql_connection_id`

    MySQL 连接 ID

+   `node_id`

    报告节点的 ID

+   `block_instance`

    报告 LDM 实例的 ID

+   `tableid`

    包含此行的表的 ID

+   `fragmentid`

    包含被锁定行的片段的 ID

+   `rowid`

    被锁定行的 ID

+   `transid`

    事务 ID

+   `mode`

    锁请求模式

+   `state`

    锁状态

+   `detail`

    是否是行锁队列中的第一个持有锁

+   `op`

    操作类型

+   `duration_millis`

    等待或持有锁的毫秒数

+   `lock_num`

    锁对象的 ID

+   `waiting_for`

    等待具有此 ID 的锁

##### 备注

`mysql_connection_id`列显示由`SHOW PROCESSLIST`显示的 MySQL 连接或线程 ID。

`block_instance`指的是内核块的一个实例。与块名称一起，此数字可用于在`threadblocks`表中查找给定实例。

`tableid`由`NDB`分配给表；在其他`ndbinfo`表中以及**ndb_show_tables**的输出中使用相同的 ID 来表示此表。

`transid`列中显示的事务 ID 是由 NDB API 为请求或持有当前锁的事务生成的标识符。

`mode`列显示锁模式，始终为`S`（共享锁）或`X`（排他锁）。如果事务对给定行有排他锁，则该行上的所有其他锁都具有相同的事务 ID。

`state`列显示锁状态。其值始终为`H`（持有）或`W`（等待）。等待锁请求等待由不同事务持有的锁。

`detail`列指示此锁是否是受影响行的锁队列中持有锁的第一个锁，如果是，则包含一个`*`（星号字符）；否则，此列为空。此信息可用于帮助识别锁请求列表中的唯一条目。

`op`列显示请求锁的操作类型。这始终是`READ`、`INSERT`、`UPDATE`、`DELETE`、`SCAN`或`REFRESH`中的一个值。

`duration_millis`列显示此锁请求等待或持有锁的毫秒数。当为等待请求授予锁时，此值将重置为 0。

锁 ID（`lockid`列）对于此节点和块实例是唯一的。

如果`lock_state`列的值为`W`，则此锁正在等待授予，并且`waiting_for`列显示此请求正在等待的锁对象的锁 ID。否则，`waiting_for`为空。`waiting_for`只能引用相同行上的锁（由`node_id`、`block_instance`、`tableid`、`fragmentid`和`rowid`标识）。
