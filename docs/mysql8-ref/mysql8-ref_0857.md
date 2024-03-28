> 原文：[`dev.mysql.com/doc/refman/8.0/en/group-replication-functions-for-communication-protocol.html`](https://dev.mysql.com/doc/refman/8.0/en/group-replication-functions-for-communication-protocol.html)

#### 14.18.1.4 检查和设置 Group Replication 通信协议版本的函数

以下函数使您能够检查和配置复制组使用的 Group Replication 通信协议版本。

+   MySQL 5.7.14 版本开始允许消息压缩（参见第 20.7.4 节，“消息压缩”）。

+   MySQL 8.0.16 版本还允许消息分段（参见第 20.7.5 节，“消息分段”）。

+   MySQL 8.0.27 版本还允许在组处于单主模式且 `group_replication_paxos_single_leader` 设置为 true 时，组通信引擎操作具有单一共识领导者（参见第 20.7.3 节，“单一共识领导者”）。

+   `group_replication_get_communication_protocol()`

    检查当前组正在使用的 Group Replication 通信协议版本。

    语法：

    ```sql
    STRING group_replication_get_communication_protocol()
    ```

    此函数没有参数。

    返回值：

    可加入此组并使用组通信协议的最旧 MySQL Server 版本。请注意，`group_replication_get_communication_protocol()` 函数返回组支持的最低 MySQL 版本，这可能与传递给 `group_replication_set_communication_protocol()` 的版本号不同，并且可能与使用该函数的成员上安装的 MySQL Server 版本不同。

    如果无法检查协议，因为此服务器实例不属于复制组，则返回错误字符串。

    示例：

    ```sql
    SELECT group_replication_get_communication_protocol();
    +------------------------------------------------+
    | group_replication_get_communication_protocol() |
    +------------------------------------------------+
    | 8.0.36                                          |
    +------------------------------------------------+
    ```

    有关更多信息，请参见第 20.5.1.4 节，“设置组的通信协议版本”。

+   `group_replication_set_communication_protocol()`

    降级组复制通信协议版本，以便较早版本的成员可以加入，或者在所有成员升级 MySQL 服务器后升级组复制通信协议版本。使用此功能需要`GROUP_REPLICATION_ADMIN`权限，并且在发出语句时，所有现有组成员必须在线，没有多数成员丢失。

    注意：

    对于 MySQL InnoDB 集群，通信协议版本在使用 AdminAPI 操作更改集群拓扑时会自动管理。您不必自己为 InnoDB 集群使用这些功能。

    语法：

    ```sql
    STRING group_replication_set_communication_protocol(*version*)
    ```

    参数：

    +   *`version`*：对于降级，指定具有最旧安装服务器版本的潜在组成员的 MySQL 服务器版本。在这种情况下，如果可能的话，该命令会使组回退到与该服务器版本兼容的通信协议。您可以指定的最小服务器版本是 MySQL 5.7.14。对于升级，请指定现有组成员已升级到的新 MySQL 服务器版本。

    返回值：

    包含操作结果的字符串，例如操作是否成功。

    示例：

    ```sql
    SELECT group_replication_set_communication_protocol("5.7.25");
    ```

    更多信息，请参见 Section 20.5.1.4, “Setting a Group's Communication Protocol Version”。
