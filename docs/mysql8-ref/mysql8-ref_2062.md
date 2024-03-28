> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-replication-group-member-stats-table.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-replication-group-member-stats-table.html)

#### 29.12.11.12 复制组成员统计表

此表显示复制组成员的统计信息。仅在运行 Group Replication 时填充。

`replication_group_member_stats`表具有以下列：

+   `CHANNEL_NAME`

    Group Replication 通道的名称

+   `VIEW_ID`

    此组的当前视图标识符。

+   `MEMBER_ID`

    成员服务器的 UUID。对于组中的每个成员，此值不同。这也作为一个键，因为对于每个成员它是唯一的。

+   `COUNT_TRANSACTIONS_IN_QUEUE`

    在队列中等待冲突检测的事务数。一旦事务经过冲突检测，如果通过检测，它们将排队等待应用。

+   `COUNT_TRANSACTIONS_CHECKED`

    已经检查冲突的事务数。

+   `COUNT_CONFLICTS_DETECTED`

    未通过冲突检测检查的事务数。

+   `COUNT_TRANSACTIONS_ROWS_VALIDATING`

    可用于认证的事务行数，但尚未被垃圾回收。可以认为是冲突检测数据库的当前大小，每个事务都会针对其进行认证。

+   `TRANSACTIONS_COMMITTED_ALL_MEMBERS`

    已在复制组的所有成员上成功提交的事务，显示为 GTID Sets。这在固定时间间隔内更新。

+   `LAST_CONFLICT_FREE_TRANSACTION`

    最后一个无冲突事务的事务标识符，已经经过检查。

+   `COUNT_TRANSACTIONS_REMOTE_IN_APPLIER_QUEUE`

    此成员从复制组接收的等待应用的事务数。

+   `COUNT_TRANSACTIONS_REMOTE_APPLIED`

    此成员从组中接收并应用的事务数。

+   `COUNT_TRANSACTIONS_LOCAL_PROPOSED`

    在此成员上起源并发送到组的事务数。

+   `COUNT_TRANSACTIONS_LOCAL_ROLLBACK`

    在此成员上起源并被组回滚的事务数。

`replication_group_member_stats`表没有索引。

`TRUNCATE TABLE`不允许用于`replication_group_member_stats`表。
