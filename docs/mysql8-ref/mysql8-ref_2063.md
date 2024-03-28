> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-replication-group-member-actions-table.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-replication-group-member-actions-table.html)

#### 29.12.11.13 复制组成员操作表

此表列出了复制组成员操作的成员操作配置中包含的成员操作。仅在安装了组复制时才可用该表。您可以使用 `group_replication_reset_member_actions()` 函数重置成员操作配置。有关更多信息，请参见 20.5.1.5 节“配置成员操作”。

`replication_group_member_actions` 表具有以下列：

+   `名称`

    成员操作的名称。

+   `事件`

    触发成员操作的事件。

+   `启用`

    成员操作当前是否已启用。可以使用 `group_replication_enable_member_action()` 函数启用成员操作，并使用 `group_replication_disable_member_action()` 函数禁用成员操作。

+   `类型`

    成员操作的类型。`INTERNAL` 是由组复制插件提供的操作。

+   `优先级`

    成员操作的优先级。具有较低优先级值的操作首先执行。

+   `错误处理`

    如果在执行成员操作时发生错误，组复制将采取的操作。`IGNORE` 表示记录错误消息以指示成员操作失败，但不会采取进一步操作。`CRITICAL` 表示成员进入 `ERROR` 状态，并执行由 `group_replication_exit_state_action` 系统变量指定的操作。

`replication_group_member_actions` 表没有索引。

`TRUNCATE TABLE` 不允许用于 `replication_group_member_actions` 表。
