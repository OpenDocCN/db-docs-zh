> [`dev.mysql.com/doc/refman/8.0/en/group-replication-communication-protocol.html`](https://dev.mysql.com/doc/refman/8.0/en/group-replication-communication-protocol.html)

#### 20.5.1.4 设置组的通信协议版本

从 MySQL 8.0.16 开始，Group Replication 具有组的通信协议的概念。Group Replication 通信协议版本可以被明确管理，并设置为适应您希望组支持的最旧 MySQL Server 版本。这使得可以从不同 MySQL Server 版本的成员组成组，同时确保向后兼容性。

+   MySQL 5.7.14 版本允许消息的压缩（参见 Section 20.7.4, “Message Compression”）。

+   MySQL 8.0.16 版本还允许消息的分段（参见 Section 20.7.5, “Message Fragmentation”）。

+   MySQL 8.0.27 版本还允许在单主模式下，当`group_replication_paxos_single_leader`设置为 true 时，组通信引擎可以与单一一致性领导者一起运行（参见 Section 20.7.3, “Single Consensus Leader”）。

所有组成员必须使用相同的通信协议版本，以便组成员可以处于不同的 MySQL Server 版本，但只发送所有组成员都能理解的消息。

MySQL 服务器在版本 X 下，只有当复制组的通信协议版本小于或等于 X 时，才能加入并达到`ONLINE`状态。当新成员加入复制组时，它会检查现有成员所宣布的通信协议版本。如果加入成员支持该版本，则加入该组并使用组宣布的通信协议，即使该成员支持额外的通信能力。如果加入成员不支持通信协议版本，则会被从组中驱逐。

如果两个成员尝试在同一成员变更事件中加入，只有当两个成员的通信协议版本已经与组的通信协议版本兼容时，它们才能加入。与组不同通信协议版本的成员必须独立加入。例如：

+   一个 MySQL Server 8.0.16 实例可以成功加入使用通信协议版本 5.7.24 的组。

+   一个 MySQL Server 5.7.24 实例无法成功加入使用通信协议版本 8.0.16 的组。

+   两个 MySQL Server 8.0.16 实例不能同时加入使用通信协议版本 5.7.24 的组。

+   两个 MySQL Server 8.0.16 实例可以同时加入使用通信协议版本 8.0.16 的组。

您可以使用`group_replication_get_communication_protocol()`函数检查组正在使用的通信协议，该函数返回组支持的最旧 MySQL Server 版本。组的所有现有成员返回相同的通信协议版本。例如：

```sql
SELECT group_replication_get_communication_protocol();
+------------------------------------------------+
| group_replication_get_communication_protocol() |
+------------------------------------------------+
| 8.0.16                                         |
+------------------------------------------------+
```

请注意，`group_replication_get_communication_protocol()`函数返回组支持的最低 MySQL 版本，这可能与传递给`group_replication_set_communication_protocol()`函数的版本号不同，并且可能与您在使用该函数的成员上安装的 MySQL Server 版本不同。

如果您需要更改组的通信协议版本，以便早期版本的成员可以加入，请使用`group_replication_set_communication_protocol()`函数指定您希望允许的最旧成员的 MySQL Server 版本。如果可能的话，这将使组回退到兼容的通信协议版本。使用此函数需要`GROUP_REPLICATION_ADMIN`权限，并且在发出语句时，所有现有组成员必须在线，没有多数成员丢失。例如：

```sql
SELECT group_replication_set_communication_protocol("5.7.25");
```

如果您将复制组的所有成员升级到新的 MySQL Server 版本，则组的通信协议版本不会自动升级以匹配。如果您不再需要支持早期版本的成员，您可以使用`group_replication_set_communication_protocol()`函数将通信协议版本设置为您已经升级成员的新 MySQL Server 版本。例如：

```sql
SELECT group_replication_set_communication_protocol("8.0.16");
```

`group_replication_set_communication_protocol()` 函数被实现为一个组操作，因此在组的所有成员上同时执行。组操作开始缓冲消息，并等待任何正在进行中的传出消息的传递完成，然后更改通信协议版本并发送缓冲消息。如果在更改通信协议版本后的任何时间点，有成员尝试加入组，组成员会宣布新的协议版本。

MySQL InnoDB 集群在使用 AdminAPI 操作更改集群拓扑结构时，自动透明地管理其成员的通信协议版本。InnoDB 集群始终使用当前所有实例支持的最新通信协议版本，这些实例当前是集群的一部分或正在加入其中。有关详细信息，请参阅 InnoDB 集群和组复制协议。
