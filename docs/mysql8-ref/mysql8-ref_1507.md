> 原文：[`dev.mysql.com/doc/refman/8.0/en/group-replication-change-primary.html`](https://dev.mysql.com/doc/refman/8.0/en/group-replication-change-primary.html)

#### 20.5.1.1 更改主节点

本节解释了如何更改单一主节点组中的主节点，使用`group_replication_set_as_primary()`函数，该函数可以在组的任何成员上运行。执行此操作后，当前主节点将成为只读辅助节点，指定的组成员将成为读写主节点；这取代了第 20.1.3.1 节，“单一主节点模式”中描述的通常的主节点选举过程。

如果标准的源到副本复制通道正在现有的主节点上运行，除了组复制通道之外，您必须在更改主节点之前停止该复制通道。您可以使用性能模式表`replication_group_members`中的`MEMBER_ROLE`列，或者`group_replication_primary_member`状态变量来识别当前的主节点。

如果所有成员未运行相同的 MySQL 服务器版本，则只能指定运行组中最低 MySQL 服务器版本的新主节点。此保护措施旨在确保组与新功能保持兼容。这适用于所有 MySQL 版本，并从 MySQL 8.0.17 开始强制执行。

组正在等待的任何未提交事务必须在操作完成之前提交、回滚或终止。在 MySQL 8.0.29 之前，该函数会等待现有主节点上的所有活动事务结束，包括在使用该函数后启动的传入事务。从 MySQL 8.0.29 开始，您可以为在使用该函数时正在运行的事务指定从 0 秒（立即）到 3600 秒（60 分钟）的超时时间。为使超时生效，组的所有成员必须运行 MySQL 8.0.29 或更高版本。超时没有默认设置，因此如果您不设置它，等待时间没有上限，新事务可以在此期间启动。

当超时到期时，对于尚未达到提交阶段的任何事务，客户端会被断开连接，以防事务继续进行。已达到提交阶段的事务将被允许完成。设置超时还会阻止从那时起在主服务器上启动新事务。即使它们不修改任何数据，显式定义的事务（使用`START TRANSACTION`或`BEGIN`语句）也会受到超时、断开连接和传入事务阻止的影响。为了允许在函数运行时检查主服务器，允许执行不修改数据的单个语句，如一致性规则下允许的查询中列出的语句。

通过发出以下语句，传递成员的`server_uuid`，该成员将成为组的新主要成员：

```sql
SELECT group_replication_set_as_primary(*member_uuid*);
```

在 MySQL 8.0.29 及更高版本中，您可以添加一个超时，如下所示：

```sql
SELECT group_replication_set_as_primary(‘00371d66-3c45-11ea-804b-080027337932’, 300)
```

要检查超时的状态，请使用性能模式`threads`表中的`PROCESSLIST_INFO`列，如下所示：

```sql
mysql> SELECT NAME, PROCESSLIST_INFO FROM performance_schema.threads 
 -> WHERE NAME="thread/group_rpl/THD_transaction_monitor"\G
*************************** 1\. row ***************************
            NAME: thread/group_rpl/THD_transaction_monitor
PROCESSLIST_INFO: Group replication transaction monitor: Stopped client connections
```

状态显示了事务监控线程何时被创建，新事务何时被停止，带有未提交事务的客户端连接何时被断开，最后，进程何时完成并允许新事务再次进行。

在操作运行时，您可以通过发出以下语句来检查其进度：

```sql
mysql> SELECT event_name, work_completed, work_estimated 
 -> FROM performance_schema.events_stages_current 
 -> WHERE event_name LIKE "%stage/group_rpl%"\G
*************************** 1\. row ***************************
    EVENT_NAME: stage/group_rpl/Primary Election: Waiting for members to turn on super_read_only
WORK_COMPLETED: 3
WORK_ESTIMATED: 5
```
