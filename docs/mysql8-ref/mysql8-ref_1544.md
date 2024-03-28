> 原文：[`dev.mysql.com/doc/refman/8.0/en/group-replication-responses-failure-partition.html`](https://dev.mysql.com/doc/refman/8.0/en/group-replication-responses-failure-partition.html)

#### 20.7.7.2 不可达多数超时

默认情况下，由于网络分区而处于少数派的成员不会自动离开群组。您可以使用系统变量`group_replication_unreachable_majority_timeout`设置成员在与大多数群组成员失去联系后等待的秒数，然后退出群组。设置超时意味着您无需主动监视网络分区后处于少数派群组的服务器，并且可以避免由于不当干预而创建群组成员的两个版本导致的分裂脑情况。

当`group_replication_unreachable_majority_timeout`指定的超时时间到期时，已由该成员和少数派群组中的其他成员处理的所有待处理事务都将被回滚，并且该群组中的服务器将移至`ERROR`状态。您可以使用从 MySQL 8.0.16 开始提供的`group_replication_autorejoin_tries`系统变量，使成员在此时自动尝试重新加入群组。从 MySQL 8.0.21 开始，默认情况下激活此功能，并且成员将进行三次自动重新加入尝试。如果自动重新加入过程不成功或未尝试，则少数派成员将按照`group_replication_exit_state_action`指定的退出操作进行。

在决定是否设置不可达多数超时时，请考虑以下几点：

+   在对称群组中，例如具有两个或四个服务器的群组，如果两个分区包含相等数量的服务器，则两个群组都认为自己处于少数派并进入`ERROR`状态。在这种情况下，群组没有功能性分区。

+   尽管存在少数派群组，但由少数派群组处理的任何事务都会被接受，但由于少数服务器无法达到法定人数，这些事务被阻塞，直到在这些服务器上发出`STOP GROUP_REPLICATION`或达到不可达多数超时为止。

+   如果不设置不可达多数超时，则少数派群组中的服务器永远不会自动进入`ERROR`状态，您必须手动停止它们。

+   如果在检测到大多数丢失后在少数派群组的服务器上设置不可达多数超时，则不会产生任何影响。

如果您不使用`group_replication_unreachable_majority_timeout`系统变量，则在网络分区事件中进行操作发明的过程在第 20.7.8 节，“处理网络分区和失去法定人数”中描述。该过程涉及检查哪些服务器正在运行，并在必要时强制进行新的组成员资格。
