> 原文：[`dev.mysql.com/doc/refman/8.0/en/group-replication-responses-failure-exit.html`](https://dev.mysql.com/doc/refman/8.0/en/group-replication-responses-failure-exit.html)

#### 20.7.7.4 退出操作

`group_replication_exit_state_action`系统变量，从 MySQL 8.0.12 和 MySQL 5.7.24 开始提供，指定了当成员由于错误或问题意外离开组时，Group Replication 应该执行的操作，以及在自动重新加入失败或不尝试重新加入时的操作。请注意，在被驱逐成员的情况下，成员在重新连接到组之前并不知道自己被驱逐，因此只有在成员成功重新连接或成员对自己产生怀疑并将自己驱逐时，才会执行指定的操作。

按影响顺序，退出操作如下：

1.  如果`READ_ONLY`是退出操作，实例将通过将系统变量`super_read_only`设置为`ON`，将 MySQL 切换到超级只读模式。当成员处于超级只读模式时，客户端无法进行任何更新操作，即使他们拥有`CONNECTION_ADMIN`权限（或已弃用的`SUPER`权限）。然而，客户端仍然可以读取数据，由于不再进行更新，随着时间的推移，存在过时读取的可能性会增加。因此，您需要主动监视服务器以防止故障。从 MySQL 8.0.15 开始，这是默认的退出操作。在执行此退出操作后，成员的状态将在组的视图中显示为`ERROR`。

1.  如果`OFFLINE_MODE`是退出操作，则通过将系统变量`offline_mode`设置为`ON`，实例将将 MySQL 切换到离线模式。当成员处于离线模式时，连接的客户端用户在下一次请求时将被断开连接，不再接受连接，但具有`CONNECTION_ADMIN`权限（或已弃用的`SUPER`权限）的客户端用户除外。Group Replication 还将系统变量`super_read_only`设置为`ON`，因此客户端无法进行任何更新，即使他们已经使用`CONNECTION_ADMIN`或`SUPER`权限连接。此退出操作阻止了更新和过时读取（除了具有上述权限的客户端用户进行读取），并使代理工具（如 MySQL Router）能够识别服务器不可用并重定向客户端连接。它还保持实例运行，以便管理员可以尝试解决问题而不关闭 MySQL。此退出操作从 MySQL 8.0.18 版本开始提供。在执行此退出操作后，成员的状态在组的视图中显示为`ERROR`（而不是`OFFLINE`，这意味着成员具有 Group Replication 功能，但当前不属于任何组）。

1.  如果`ABORT_SERVER`是退出操作，则实例将关闭 MySQL。指示成员关闭自身可以防止所有过时读取和客户端更新，但这意味着 MySQL Server 实例不可用，必须重新启动，即使问题可以在不进行此步骤的情况下解决。此退出操作是从 MySQL 8.0.12 版本（添加系统变量时）到 MySQL 8.0.15 版本（含）的默认操作。在执行此退出操作后，成员将从组的视图中的服务器列表中删除。

请注意，无论设置了哪种退出操作，都需要操作员干预，因为已耗尽自动重新加入尝试（或从未有过）并已被组驱逐的前成员不允许重新加入而无需重新启动 Group Replication。退出操作仅影响客户端是否仍然可以在无法重新加入组的服务器上读取数据，以及服务器是否保持运行。

重要提示

如果在成员成功加入组之前发生故障，则不会执行`group_replication_exit_state_action`指定的退出操作。这种情况发生在本地配置检查期间发生故障，或者加入成员的配置与组的配置不匹配时。在这些情况下，`super_read_only`系统变量保持其原始值，服务器不会关闭 MySQL。为了确保当 Group Replication 未启动时服务器无法接受更新，我们建议在服务器启动时在配置文件中设置`super_read_only=ON`，Group Replication 在成功启动后将其更改为`OFF`。当服务器配置为在服务器启动时启动 Group Replication（`group_replication_start_on_boot=ON`）时，此保护措施尤为重要，但在使用`START GROUP_REPLICATION`命令手动启动 Group Replication 时也很有用。

如果成员成功加入组后发生故障，则执行指定的退出操作。以下情况会发生：

1.  *应用程序错误* - 复制应用程序中存在错误。此问题无法恢复。

1.  *无法进行分布式恢复* - 存在问题，导致 Group Replication 的分布式恢复过程（使用远程克隆操作和从二进制日志进行状态传输）无法完成。Group Replication 在适当的情况下会自动重试分布式恢复，但如果没有更多选项来完成该过程，则会停止。有关详细信息，请参见第 20.5.4.4 节，“分布式恢复的容错性”。

1.  *组配置更改错误* - 在使用函数进行组范围配置更改时发生错误，如第 20.5.1 节，“配置在线组”中所述。

1.  *主选举错误* - 在单主模式下为组选择新主成员时发生错误，如第 20.1.3.1 节，“单主模式”中所述。

1.  *无法访问的多数超时* - 成员与大多数组成员失去联系，因此处于少数，并且由 `group_replication_unreachable_majority_timeout` 系统变量设置的超时已经过期。

1.  *成员被组驱逐* - 对成员提出了怀疑，并且由 `group_replication_member_expel_timeout` 系统变量设置的任何超时已经过期，成员已恢复与组的通信并发现自己已被驱逐。

1.  *超出自动重新加入尝试次数* - `group_replication_autorejoin_tries` 系统变量被设置为指定在失去多数或被驱逐后的自动重新加入尝试次数，成员完成了这些尝试次数但未成功。

以下表格总结了每种情况下的失败场景和操作：

**表 20.3 组复制失败情况下的退出操作**

| 失败情况 | 使用 `START GROUP_REPLICATION` 启动组复制 | 使用 `group_replication_start_on_boot =ON` 启动组复制 |
| --- | --- | --- |
| 成员未通过本地配置检查加入成员与组配置不匹配 | `super_read_only` 和 `offline_mode` 未更改 MySQL 继续运行在启动时设置 `super_read_only=ON` 以防止更新 | `super_read_only` 和 `offline_mode` 未更改 MySQL 继续运行在启动时设置 `super_read_only=ON` 以防止更新（重要） |
| 应用程序错误在成员分布式恢复不可能组配置更改错误主要选举错误无法访问的多数超时成员被组驱逐超出自动重新加入尝试次数 | `super_read_only` 设置为 `ON` 或 `offline_mode` 和 `super_read_only` 设置为 `ON` 或 MySQL 关闭 | `super_read_only` 设置为 `ON` 或 `offline_mode` 和 `super_read_only` 设置为 `ON` 或 MySQL 关闭 |
