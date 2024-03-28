# 14.18.3 异步复制通道故障转移函数

> 原文：[`dev.mysql.com/doc/refman/8.0/en/replication-functions-async-failover.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-functions-async-failover.html)

以下函数从 MySQL 8.0.22 开始适用于标准源到副本复制，从 MySQL 8.0.23 开始适用于 Group Replication，使您能够向复制通道的源列表中添加和删除复制源服务器。从 MySQL 8.0.27 开始，您还可以清除服务器的源列表。

**表 14.27 故障转移通道函数**

| 名称 | 描述 | 引入版本 |
| --- | --- | --- |
| [`asynchronous_connection_failover_add_managed()`](https://dev.mysql.com/doc/refman/8.0/en/replication-functions-async-failover.html#function_asynchronous-connection-failover-add-managed) | 将组成员源服务器配置信息添加到复制通道源列表 | 8.0.23 |
| [`asynchronous_connection_failover_add_source()`](https://dev.mysql.com/doc/refman/8.0/en/replication-functions-async-failover.html#function_asynchronous-connection-failover-add-source) | 将源服务器配置信息添加到复制通道源列表 | 8.0.22 |
| [`asynchronous_connection_failover_delete_managed()`](https://dev.mysql.com/doc/refman/8.0/en/replication-functions-async-failover.html#function_asynchronous-connection-failover-delete-managed) | 从复制通道源列表中删除托管组 | 8.0.23 |
| [`asynchronous_connection_failover_delete_source()`](https://dev.mysql.com/doc/refman/8.0/en/replication-functions-async-failover.html#function_asynchronous-connection-failover-delete-source) | 从复制通道源列表中删除源服务器 | 8.0.22 |
| [`asynchronous_connection_failover_reset()`](https://dev.mysql.com/doc/refman/8.0/en/replication-functions-async-failover.html#function_asynchronous-connection-failover-reset) | 删除与组复制异步故障转移相关的所有设置 | 8.0.27 |

异步连接故障转移机制在复制连接（源到副本）失败后，自动从适当列表中为新源建立异步连接。从 MySQL 8.0.23 开始，如果当前连接的源不是组中具有最高加权优先级的源，则连接也会更改。对于作为托管组的一部分定义的 Group Replication 源服务器，如果当前连接的源离开组或不再占多数，连接也会切换到另一个组成员。有关该机制的更多信息，请参见[第 19.4.9 节，“使用异步连接故障转移切换源和副本”](https://dev.mysql.com/doc/refman/8.0/en/replication-asynchronous-connection-failover.html "19.4.9 使用异步连接故障转移切换源和副本")。

源列表存储在`mysql.replication_asynchronous_connection_failover`和`mysql.replication_asynchronous_connection_failover_managed`表中，并且可以在性能模式`replication_asynchronous_connection_failover`表中查看。

如果复制通道位于启用了副本之间故障切换的组的 Group Replication 主服务器上，则当它们加入或通过任何方法更新时，源列表将广播给所有组成员。副本之间的故障切换由`mysql_start_failover_channels_if_primary`成员操作控制，默认情况下启用，并且可以使用`group_replication_disable_member_action`函数禁用。

+   `asynchronous_connection_failover_add_managed()`

    为受管组（Group Replication 组成员）的复制源服务器添加配置信息到复制通道的源列表中。您只需要添加一个组成员。副本会自动从当前组成员中添加其余成员，然后根据成员变化保持源列表更新。

    语法：

    ```sql
    asynchronous_connection_failover_add_managed(*channel*, *managed_type*, *managed_name*, *host*, *port*, *network_namespace*, *primary_weight*, *secondary_weight*)
    ```

    参数：

    +   *`channel`*: 此复制源服务器所属的复制通道。

    +   *`managed_type`*: 异步连接故障转移机制必须为此服务器提供的受管服务类型。当前唯一接受的值是`GroupReplication`。

    +   *`managed_name`*: 服务器所属的受管组的标识符。对于`GroupReplication`受管服务，标识符是`group_replication_group_name`系统变量的值。

    +   *`host`*: 此复制源服务器的主机名。

    +   *`port`*: 此复制源服务器的端口号。

    +   *`network_namespace`*: 此复制源服务器的网络命名空间。指定空字符串，因为此参数保留供将来使用。

    +   *`primary_weight`*: 当作为受管组的主服务器时，此复制源服务器在复制通道源列表中的优先级。权重范围为 1 到 100，100 为最高。对于主服务器，80 是一个合适的权重。如果当前连接的源不是组中权重最高的源，则异步连接故障转移机制会激活。假设您设置了受管组，为主服务器分配较高的权重，为次要服务器分配较低的权重，当主服务器更改时，其权重增加，副本会切换到连接到主服务器。

    +   *`secondary_weight`*: 当作为托管组中的次要角色时，此复制源服务器在复制通道源列表中的优先级。权重范围为 1 到 100，100 为最高。对于次要角色，60 是一个合适的权重。

    返回值：

    包含操作结果的字符串，例如操作是否成功。

    示例：

    ```sql
    SELECT asynchronous_connection_failover_add_managed('channel2', 'GroupReplication', 'aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa', '127.0.0.1', 3310, '', 80, 60);
    +----------------------------------------------------------------------------------------------------------------------------------------------------+
    | asynchronous_connection_failover_add_source('channel2', 'GroupReplication', 'aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa', '127.0.0.1', 3310, '', 80, 60) |
    +----------------------------------------------------------------------------------------------------------------------------------------------------+
    | Source managed configuration details successfully inserted.                                                                                        |
    +----------------------------------------------------------------------------------------------------------------------------------------------------+
    ```

    更多信息，请参见 Section 19.4.9, “使用异步连接故障转移切换源和副本”。

+   `asynchronous_connection_failover_add_source()`

    为复制通道的源列表添加复制源服务器的配置信息。

    语法：

    ```sql
    asynchronous_connection_failover_add_source(*channel*, *host*, *port*, *network_namespace*, *weight*)
    ```

    参数：

    +   *`channel`*: 此复制源服务器所属源列表的复制通道。

    +   *`host`*: 此复制源服务器的主机名。

    +   *`port`*: 此复制源服务器的端口号。

    +   *`network_namespace`*: 此复制源服务器的网络命名空间。请指定一个空字符串，因为此参数保留供将来使用。

    +   *`weight`*: 此复制源服务器在复制通道源列表中的优先级。优先级范围为 1 到 100，100 为最高，50 为默认值。当异步连接故障转移机制激活时，通道源列表中具有最高优先级设置的备用源将被选择用于第一次连接尝试。如果此尝试不起作用，则副本将按优先级降序尝试所有列出的源，然后从最高优先级源重新开始。如果多个源具有相同的优先级，则副本会随机排序它们。从 MySQL 8.0.23 开始，如果当前连接的源不是组中权重最高的源，则异步连接故障转移机制将激活。

    返回值：

    包含操作结果的字符串，例如操作是否成功。

    示例：

    ```sql
    SELECT asynchronous_connection_failover_add_source('channel2', '127.0.0.1', 3310, '', 80);
    +-------------------------------------------------------------------------------------------------+
    | asynchronous_connection_failover_add_source('channel2', '127.0.0.1', 3310, '', 80)              |
    +-------------------------------------------------------------------------------------------------+
    | Source configuration details successfully inserted.                                             |
    +-------------------------------------------------------------------------------------------------+
    ```

    更多信息，请参见 Section 19.4.9, “使用异步连接故障转移切换源和副本”。

+   `asynchronous_connection_failover_delete_managed()`

    从复制通道的源列表中删除整个托管组。使用此函数时，托管组中定义的所有复制源服务器都将从通道的源列表中移除。

    语法：

    ```sql
    asynchronous_connection_failover_delete_managed(*channel*, *managed_name*)
    ```

    参数：

    +   *`channel`*: 此复制源服务器曾经是其所属源列表的复制通道。

    +   *`managed_name`*：服务器所属的托管组的标识符。对于`GroupReplication`托管服务，标识符是`group_replication_group_name`系统变量的值。

    返回值：

    包含操作结果的字符串，例如操作是否成功。

    示例：

    ```sql
    SELECT asynchronous_connection_failover_delete_managed('channel2', 'aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa');
    +-----------------------------------------------------------------------------------------------------+
    | asynchronous_connection_failover_delete_managed('channel2', 'aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa') |
    +-----------------------------------------------------------------------------------------------------+
    | Source managed configuration details successfully deleted.                                          |
    +-----------------------------------------------------------------------------------------------------+
    ```

    有关更多信息，请参见第 19.4.9 节，“使用异步连接故障转移切换源和副本”。

+   `asynchronous_connection_failover_delete_source()`

    从复制通道的源列表中删除复制源服务器的配置信息。

    语法：

    ```sql
    asynchronous_connection_failover_delete_source(*channel*, *host*, *port*, *network_namespace*)
    ```

    参数：

    +   *`channel`*：此复制源服务器所属源列表的复制通道。

    +   *`host`*：此复制源服务器的主机名。

    +   *`port`*：此复制源服务器的端口号。

    +   *`network_namespace`*：此复制源服务器的网络命名空间。指定空字符串，因为此参数保留供将来使用。

    返回值：

    包含操作结果的字符串，例如操作是否成功。

    示例：

    ```sql
    SELECT asynchronous_connection_failover_delete_source('channel2', '127.0.0.1', 3310, '');
    +------------------------------------------------------------------------------------------------+
    | asynchronous_connection_failover_delete_source('channel2', '127.0.0.1', 3310, '')              |
    +------------------------------------------------------------------------------------------------+
    | Source configuration details successfully deleted.                                             |
    +------------------------------------------------------------------------------------------------+
    ```

    有关更多信息，请参见第 19.4.9 节，“使用异步连接故障转移切换源和副本”。

+   `asynchronous_connection_failover_reset()`

    删除与异步连接故障转移机制相关的所有设置。该函数清除性能模式表`replication_asynchronous_connection_failover`和`replication_asynchronous_connection_failover_managed`。

    `asynchronous_connection_failover_reset()`只能在当前不属于任何组且没有任何复制通道运行的服务器上使用。您可以使用此函数清理不再在托管组中使用的服务器。

    语法：

    ```sql
    STRING asynchronous_connection_failover_reset()
    ```

    参数：

    无。

    返回值：

    包含操作结果的字符串，例如操作是否成功。

    示例：

    ```sql
    mysql> SELECT asynchronous_connection_failover_reset();
    +-------------------------------------------------------------------------+
    | asynchronous_connection_failover_reset()                                |
    +-------------------------------------------------------------------------+
    | The UDF asynchronous_connection_failover_reset() executed successfully. |
    +-------------------------------------------------------------------------+
    1 row in set (0.00 sec)
    ```

    欲了解更多信息，请参阅第 19.4.9 节，“使用异步连接故障转移切换源和副本”。
