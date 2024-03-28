> 原文：[`dev.mysql.com/doc/refman/8.0/en/group-replication-responses-failure-rejoin.html`](https://dev.mysql.com/doc/refman/8.0/en/group-replication-responses-failure-rejoin.html)

#### 20.7.7.3 自动重新加入

`group_replication_autorejoin_tries`系统变量从 MySQL 8.0.16 开始提供，使被驱逐或达到不可达多数超时的成员尝试自动重新加入组。在 MySQL 8.0.20 之前，系统变量的值默认为 0，因此默认情况下不会激活自动重新加入。从 MySQL 8.0.21 开始，系统变量的值默认为 3，这意味着成员会自动尝试重新加入组，每次尝试之间间隔 5 分钟，最多尝试 3 次。

当未激活自动重新加入时，成员一旦恢复通信即接受其驱逐，并继续执行由`group_replication_exit_state_action`系统变量指定的操作。之后，需要手动干预才能将成员带回组中。如果您可以容忍可能出现过时读取的可能性，并希望最小化手动干预的需求，特别是在瞬时网络问题经常导致成员被驱逐的情况下，使用自动重新加入功能是合适的。

使用自动重新加入时，当成员被驱逐或达到不可达多数超时时，它会尝试重新加入（使用当前插件选项值），然后继续进行进一步的自动重新加入尝试，直到达到指定的尝试次数为止。在一次不成功的自动重新加入尝试后，成员在下一次尝试之前等待 5 分钟。自动重新加入尝试和它们之间的时间称为自动重新加入过程。如果在成员重新加入或停止之前耗尽了指定的尝试次数，则成员将执行由`group_replication_exit_state_action`系统变量指定的操作。

在自动重新加入尝试期间和之间，成员保持在超级只读模式，并在其复制组视图上显示`ERROR`状态。在此期间，成员不接受写入。但是，随着时间的推移，成员上的读取仍然可以进行，但随着时间的推移，读取变得越来越可能过时。如果您确实希望在自动重新加入过程中干预以使成员脱机，可以随时使用`STOP GROUP_REPLICATION`语句或关闭服务器来手动停止成员。如果您无法容忍任何时间段内可能出现过时读取的可能性，请将`group_replication_autorejoin_tries`系统变量设置为 0。

你可以使用性能模式监视自动重新加入过程。在自动重新加入过程进行时，性能模式表`events_stages_current`显示事件“正在进行自动重新加入过程”，显示到目前为止在此过程实例中尝试的重试次数（在`WORK_COMPLETED`字段中）。`events_stages_summary_global_by_event_name`表显示服务器实例启动自动重新加入过程的次数（在`COUNT_STAR`字段中）。`events_stages_history_long`表显示每个自动重新加入过程完成的时间（在`TIMER_END`字段中）。当成员重新加入复制组时，在组完成兼容性检查并接受其为成员之前，其状态可能显示为`OFFLINE`或`ERROR`。当成员正在赶上组的事务时，其状态为`RECOVERING`。
