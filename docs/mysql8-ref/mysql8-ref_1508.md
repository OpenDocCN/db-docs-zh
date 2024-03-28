> 原文：[`dev.mysql.com/doc/refman/8.0/en/group-replication-changing-group-mode.html`](https://dev.mysql.com/doc/refman/8.0/en/group-replication-changing-group-mode.html)

#### 20.5.1.2 更改组模式

本节解释了如何更改组运行的模式，即单主或多主。用于更改组模式的函数可以在任何成员上运行。

##### 切换到单主模式

使用`group_replication_switch_to_single_primary_mode()`函数通过执行以下命令将运行在多主模式下的组切换到单主模式：

```sql
SELECT group_replication_switch_to_single_primary_mode()
```

当您切换到单主模式时，所有组成员上也会禁用严格的一致性检查，这是单主模式所要求的（`group_replication_enforce_update_everywhere_checks=OFF`）。

如果没有传入字符串，在结果为单主组的新主要选举中，遵循 20.1.3.1 节，“单主模式”中描述的选举策略。要覆盖选举过程并在过程中配置多主组的特定成员作为新主要成员，请获取成员的`server_uuid`并将其传递给`group_replication_switch_to_single_primary_mode()`。例如，执行以下命令：

```sql
SELECT group_replication_switch_to_single_primary_mode(*member_uuid*);
```

如果在运行 MySQL Server 版本为 8.0.17 的成员上调用该函数，并且所有成员都运行 MySQL Server 版本为 8.0.17 或更高版本，则只能指定运行最低 MySQL Server 版本的新主要成员，基于补丁版本。这个保障是为了确保组保持与新功能的兼容性。如果不指定新的主要成员，选举过程将考虑组成员的补丁版本。

如果任何成员运行的 MySQL Server 版本介于 MySQL 8.0.13 和 MySQL 8.0.16 之间，则不会强制执行此保障，并且您可以指定任何新的主要成员，但建议选择运行组中最低 MySQL Server 版本的主要成员。如果不指定新的主要成员，选举过程仅考虑组成员的主要版本。

在操作运行时，您可以通过执行以下命令来检查其进度：

```sql
SELECT event_name, work_completed, work_estimated FROM performance_schema.events_stages_current WHERE event_name LIKE "%stage/group_rpl%";
+----------------------------------------------------------------------------+----------------+----------------+
| event_name                                                                 | work_completed | work_estimated |
+----------------------------------------------------------------------------+----------------+----------------+
| stage/group_rpl/Primary Switch: waiting for pending transactions to finish |              4 |             20 |
+----------------------------------------------------------------------------+----------------+----------------+
```

##### 切换到多主模式

使用`group_replication_switch_to_multi_primary_mode()`函数通过执行以下命令将运行在单主模式下的组切换到多主模式：

```sql
SELECT group_replication_switch_to_multi_primary_mode()
```

在执行一些协调的组操作以确保数据的安全性和一致性之后，所有属于该组的成员都变为主节点。

当您将在单主模式下运行的组更改为多主模式时，运行 MySQL 8.0.17 或更高版本的成员如果运行的 MySQL 服务器版本高于组中存在的最低版本，则会自动处于只读模式。运行 MySQL 8.0.16 或更低版本的成员不执行此检查，并始终处于读写模式。

当操作运行时，您可以通过发出以下命令来检查其进度：

```sql
SELECT event_name, work_completed, work_estimated FROM performance_schema.events_stages_current WHERE event_name LIKE "%stage/group_rpl%";
+----------------------------------------------------------------------+----------------+----------------+
| event_name                                                           | work_completed | work_estimated |
+----------------------------------------------------------------------+----------------+----------------+
| stage/group_rpl/Multi-primary Switch: applying buffered transactions |              0 |              1 |
+----------------------------------------------------------------------+----------------+----------------+
```
