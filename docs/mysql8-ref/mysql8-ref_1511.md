> 原文：[`dev.mysql.com/doc/refman/8.0/en/group-replication-member-actions.html`](https://dev.mysql.com/doc/refman/8.0/en/group-replication-member-actions.html)

#### 20.5.1.5 配置成员操作

从 MySQL 8.0.26 开始，Group Replication 具有设置组成员在指定情况下采取的操作的能力。可以使用函数单独启用和禁用成员操作。服务器的成员操作配置在离开组后也可以重置为默认值。

管理员（具有`GROUP_REPLICATION_ADMIN`权限）可以使用`group_replication_enable_member_action`或`group_replication_disable_member_action`函数在组的主服务器上配置成员操作。然后，成员操作配置（包括所有成员操作以及它们是否启用或禁用）通过 Group Replication 的组消息传播到其他组成员和加入成员。因此，所有组成员都具有相同的成员操作配置。您还可以在不属于任何组的服务器上配置成员操作，只要安装了 Group Replication 插件。在这种情况下，成员操作配置不会传播到任何其他服务器。

如果在使用函数配置成员操作的服务器是组的一部分，则它必须是单主模式下的当前主服务器，并且必须是大多数的一部分。配置更改在 Group Replication 内部被跟踪，但不会被赋予 GTID，并且不会被写入二进制日志，因此不会传播到任何组外的服务器，如下游复制。每次启用或禁用成员操作时，Group Replication 都会增加其成员操作配置的版本号。

成员操作配置传播给成员的方式如下：

+   在启动组时，引导组的服务器的成员操作配置成为组的配置。

+   如果组的最低 MySQL Server 版本支持成员操作，则加入成员在加入时进行状态交换过程中接收组的成员操作配置。在这种情况下，加入成员将自己的成员操作配置替换为组的配置。

+   如果支持成员操作的加入成员加入一个最低 MySQL Server 版本不支持成员操作的组，则加入时不会接收成员操作配置。在这种情况下，加入成员将自己的配置重置为默认值。

不支持成员操作的成员无法加入具有成员操作配置的组，因为其 MySQL Server 版本低于现有组成员运行的最低版本。

Performance Schema 表`replication_group_member_actions`列出了配置中可用的成员操作、触发它们的事件以及它们当前是否启用。成员操作的优先级从 1 到 100，较低的值首先执行。如果在执行成员操作时发生错误，则可以记录成员操作的失败，但否则忽略。如果认为成员操作的失败是关键的，则可以根据`group_replication_exit_state_action`系统变量指定的策略进行处理。

可以使用 Performance Schema 表`replication_group_configuration_version`查看的`mysql.replication_group_configuration_version`表记录了成员操作配置的当前版本。每当使用函数启用或禁用成员操作时，版本号都会递增。

`group_replication_reset_member_actions`函数只能在不属于任何组的服务器上使用。它将成员操作配置重置为默认设置，并将其版本号重置为 1。服务器必须可写（使用`read_only`系统变量设置为`OFF`），并安装了 Group Replication 插件。您可以使用此函数删除服务器在成为组的一部分时使用的成员操作配置，如果您打算将其用作没有成员操作或具有不同成员操作的独立服务器。

##### 成员操作：`mysql_disable_super_read_only_if_primary`

成员操作`mysql_disable_super_read_only_if_primary`可以配置为使单主模式下的组在选举新主时保持在超级只读模式，以便该组仅接受复制事务，不接受来自客户端的直接写入。这种设置意味着当一个组的目的是为另一个组提供灾难容忍的次要备份时，您可以确保次要组与第一个组保持同步。

默认情况下，当选为主服务器时，超级只读模式在主服务器上被禁用，以便主服务器变为读写，并接受来自复制源服务器和客户端的更新。这是当成员操作`mysql_disable_super_read_only_if_primary`被启用时的情况，这是其默认设置。如果您使用`group_replication_disable_member_action`函数将操作设置为禁用，主服务器在选举后仍然保持在超级只读模式。在这种状态下，它不接受来自任何客户端的更新，甚至是具有`CONNECTION_ADMIN`或`SUPER`权限的用户。它仍然会接受由复制线程执行的更新。
