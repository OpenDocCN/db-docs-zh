> 原文：[`dev.mysql.com/doc/refman/8.0/en/group-replication-functions-for-member-actions.html`](https://dev.mysql.com/doc/refman/8.0/en/group-replication-functions-for-member-actions.html)

#### 14.18.1.5 设置和重置组复制成员操作的函数

以下函数可用于启用和禁用组成员在指定情况下执行的操作，并将所有成员操作的配置重置为默认设置。只有具有`GROUP_REPLICATION_ADMIN`权限或已弃用的`SUPER`权限的管理员才能使用这些函数。

您可以使用`group_replication_enable_member_action`和`group_replication_disable_member_action`函数在组的主服务器上配置成员操作。然后，成员操作配置（包括所有成员操作以及它们是否启用或禁用）通过组复制的组消息传播到其他组成员和加入成员。这意味着当处于指定情况时，组成员将以相同的方式行事，您只需在主服务器上使用该函数。

只要安装了组复制插件，这些函数也可以用于不属于任何组的服务器。在这种情况下，成员操作配置不会传播到任何其他服务器。

`group_replication_reset_member_actions`函数只能在不属于任何组的服务器上使用。它将成员操作配置重置为默认设置，并重置其版本号。服务器必须是可写的（使用`read_only`系统变量设置为`OFF`），并且安装了组复制插件。

可用的成员操作如下：

`mysql_disable_super_read_only_if_primary`

该成员操作从 MySQL 8.0.26 版本开始提供。它在成员被选为组的主服务器后执行，这是事件`AFTER_PRIMARY_ELECTION`。该成员操作默认启用。您可以使用`group_replication_disable_member_action()`函数禁用它，并使用`group_replication_enable_member_action()`重新启用它。

当启用并执行此成员操作时，主节点上的超级只读模式将被禁用，因此主节点变为读写，并接受来自复制源服务器和客户端的更新。这是正常情况。

当禁用并未执行此成员操作时，选举后主节点仍处于超级只读模式。在此状态下，它不接受来自任何客户端的更新，即使是具有`CONNECTION_ADMIN`或`SUPER`权限的用户也不行。它仍然会接受由复制线程执行的更新。这种设置意味着当一个群组的目的是为另一个群组提供灾难容忍的次要备份时，您可以确保次要群组与第一个群组保持同步。

`mysql_start_failover_channels_if_primary`

此成员操作从 MySQL 8.0.27 版本开始提供。它在成员被选举为群组主节点后（即事件`AFTER_PRIMARY_ELECTION`）执行。该成员操作默认启用。您可以使用`group_replication_disable_member_action()`函数来禁用它，并使用`group_replication_enable_member_action()`函数重新启用它。

当启用此成员操作时，在群组复制主节点上的复制通道上设置`SOURCE_CONNECTION_AUTO_FAILOVER=1`语句时，副本的异步连接故障转移处于活动状态。当功能处于活动状态并正确配置时，如果正在复制的主节点下线或进入错误状态，则新的主节点在选举时会在同一通道上开始复制。这是正常情况。有关配置该功能的说明，请参见第 19.4.9.2 节，“副本的异步连接故障转移”。

当禁用此成员操作时，副本的异步连接故障转移不会发生。如果主节点下线或进入错误状态，则通道的复制将停止。请注意，如果有多个具有`SOURCE_CONNECTION_AUTO_FAILOVER=1`的通道，则成员操作将覆盖所有通道，因此不能通过此方法单独启用或禁用它们。设置`SOURCE_CONNECTION_AUTO_FAILOVER=0`以禁用单个通道。

有关成员操作及如何查看成员操作配置的更多信息，请参见第 20.5.1.5 节，“配置成员操作”。

+   `group_replication_disable_member_action()`

    禁用成员动作，以便成员在指定情况下不执行它。如果您使用该函数的服务器是组的一部分，则必须是单主模式下的当前主服务器，并且必须是大多数成员。更改的设置会传播到其他组成员和加入成员，因此当它们处于指定情况时，它们将以相同的方式行动，您只需在主服务器上使用该函数。

    语法：

    ```sql
    STRING group_replication_disable_member_action(*name*, *event*)
    ```

    参数：

    +   *`name`*：要禁用的成员动作的名称。

    +   *`event`*：触发成员动作的事件。

    返回值：

    包含操作结果的字符串，例如是否成功。

    示例：

    ```sql
    SELECT group_replication_disable_member_action("mysql_disable_super_read_only_if_primary", "AFTER_PRIMARY_ELECTION");
    ```

    有关更多信息，请参见 Section 20.5.1.5, “Configuring Member Actions”。

+   `group_replication_enable_member_action()`

    启用成员在指定情况下执行的动作。如果您使用该函数的服务器是组的一部分，则必须是单主模式下的当前主服务器，并且必须是大多数成员。更改的设置会传播到其他组成员和加入成员，因此当它们处于指定情况时，它们将以相同的方式行动，您只需在主服务器上使用该函数。

    语法：

    ```sql
    STRING group_replication_enable_member_action(*name*, *event*)
    ```

    参数：

    +   *`name`*：要启用的成员动作的名称。

    +   *`event`*：触发成员动作的事件。

    返回值：

    包含操作结果的字符串，例如是否成功。

    示例：

    ```sql
    SELECT group_replication_enable_member_action("mysql_disable_super_read_only_if_primary", "AFTER_PRIMARY_ELECTION");
    ```

    有关更多信息，请参见 Section 20.5.1.5, “Configuring Member Actions”。

+   `group_replication_reset_member_actions()`

    将成员动作配置重置为默认设置，并将其版本号重置为 1。

    `group_replication_reset_member_actions()`函数只能在当前不是组成员的服务器上使用。服务器必须是可写的（使用`read_only`系统变量设置为`OFF`），并且安装了 Group Replication 插件。如果您打算将其用作没有成员动作或不同成员动作的独立服务器，则可以使用此函数删除服务器在成为组成员时使用的成员动作配置。

    语法：

    ```sql
    STRING group_replication_reset_member_actions()
    ```

    参数：

    无。

    返回值：

    包含操作结果的字符串，例如是否成功。

    示例：

    ```sql
    SELECT group_replication_reset_member_actions();
    ```

    欲了解更多信息，请参阅第 20.5.1.5 节，“配置成员操作”。
