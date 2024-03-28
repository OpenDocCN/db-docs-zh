# 20.9.1 Group Replication 系统变量

> 原文：[`dev.mysql.com/doc/refman/8.0/en/group-replication-system-variables.html`](https://dev.mysql.com/doc/refman/8.0/en/group-replication-system-variables.html)

本节列出了特定于 Group Replication 插件的系统变量。

每个 Group Replication 系统变量的名称都以`group_replication_`为前缀。

注意

InnoDB Cluster 使用 Group Replication，但 Group Replication 系统变量的默认值可能与本节中记录的默认值不同。例如，在 InnoDB Cluster 中，`group_replication_communication_stack`的默认值是`MYSQL`，而不是默认 Group Replication 实现的`XCOM`。

更多信息，请参阅 MySQL InnoDB Cluster。

一些 Group Replication 组成员的系统变量，包括一些 Group Replication 特定的系统变量和一些通用的系统变量，都是组范围的配置设置。这些系统变量在所有组成员上必须具有相同的值，并且需要对组进行完全重启（由具有`group_replication_bootstrap_group=ON`的服务器引导）才能使值更改生效。有关在所有成员已停止的组中重新启动的说明，请参阅 Section 20.5.2, “Restarting a Group”。

如果运行中的组对于组范围的配置设置有一个值设置，而加入的成员对于该系统变量有不同的值设置，则加入的成员在值匹配之前无法加入该组。如果组对于这些系统变量中的一个设置了一个值，而加入的成员不支持该系统变量，则无法加入该组。

以下系统变量是组范围的配置设置：

+   `group_replication_single_primary_mode`

+   `group_replication_enforce_update_everywhere_checks`

+   `group_replication_gtid_assignment_block_size`

+   `group_replication_view_change_uuid`

+   `group_replication_paxos_single_leader`

+   `group_replication_communication_stack`（这是一个特殊情况，不受 Group Replication 自身检查的限制；有关详细信息，请参阅系统变量描述）

+   `default_table_encryption`

+   `lower_case_table_names`

+   `transaction_write_set_extraction`（从 MySQL 8.0.26 开始已弃用）

在 Group Replication 运行时无法通过通常的方法更改组范围的配置设置。然而，从 MySQL 8.0.16 开始，您可以使用`group_replication_switch_to_single_primary_mode()`和`group_replication_switch_to_multi_primary_mode()`函数在组仍在运行时更改`group_replication_single_primary_mode`和`group_replication_enforce_update_everywhere_checks`的值。更多信息，请参阅 Section 20.5.1.2, “Changing the Group Mode”。

大多数 Group Replication 的系统变量在不同的组成员上可以有不同的值。对于以下系统变量，建议在组的所有成员上设置相同的值，以避免事务不必要的回滚、消息传递失败或消息恢复失败：

+   `group_replication_auto_increment_increment`

+   `group_replication_communication_max_message_size`

+   `group_replication_compression_threshold`

+   `group_replication_message_cache_size`

+   `group_replication_transaction_size_limit`

大多数 Group Replication 的系统变量被描述为动态的，它们的值可以在服务器运行时更改。然而，在大多数情况下，更改只有在您使用`STOP GROUP_REPLICATION`语句停止并重新启动组复制后才会生效，随后是一个`START GROUP_REPLICATION`语句。以下系统变量的更改在不停止和重新启动 Group Replication 的情况下生效：

+   `group_replication_advertise_recovery_endpoints`

+   `group_replication_autorejoin_tries`

+   `group_replication_consistency`

+   `group_replication_exit_state_action`

+   `group_replication_flow_control_applier_threshold`

+   `group_replication_flow_control_certifier_threshold`

+   `group_replication_flow_control_hold_percent`

+   `group_replication_flow_control_max_quota`

+   `group_replication_flow_control_member_quota_percent`

+   `group_replication_flow_control_min_quota`

+   `group_replication_flow_control_min_recovery_quota`

+   `group_replication_flow_control_mode`

+   `group_replication_flow_control_period`

+   `group_replication_flow_control_release_percent`

+   `group_replication_force_members`

+   `group_replication_ip_allowlist`

+   `group_replication_ip_whitelist`

+   `group_replication_member_expel_timeout`

+   `group_replication_member_weight`

+   `group_replication_transaction_size_limit`

+   `group_replication_unreachable_majority_timeout`

当您更改任何 Group Replication 系统变量的值时，请记住，如果在每个成员同时通过 `STOP GROUP_REPLICATION` 语句或系统关闭停止 Group Replication 的某一点，那么必须像第一次启动一样通过引导重新启动组。有关安全执行此操作的说明，请参见 Section 20.5.2, “重新启动组”。对于整个组的配置设置，这是必需的，但如果您正在更改其他设置，请尽量确保至少有一个成员始终在运行。

重要

+   如果将一些 Group Replication 的系统变量作为命令行参数传递给服务器，则在服务器启动期间，一些 Group Replication 的系统变量在启动时并未完全验证。这些系统变量包括 `group_replication_group_name`、`group_replication_single_primary_mode`、`group_replication_force_members`、SSL 变量和流量控制系统变量。它们只在服务器启动后完全验证。

+   指定组成员的 IP 地址或主机名的 Group Replication 系统变量在发出 `START GROUP_REPLICATION` 语句之前不会被验证。直到那时，Group Replication 的组通信系统 (GCS) 才可用于验证这些值。

专用于 Group Replication 插件的系统变量如下：

+   `group_replication_advertise_recovery_endpoints`

    | 命令行格式 | `--group-replication-advertise-recovery-endpoints=value` |
    | --- | --- |
    | 引入版本 | 8.0.21 |
    | 系统变量 | `group_replication_advertise_recovery_endpoints` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 字符串 |
    | 默认值 | `DEFAULT` |

    此系统变量的值可以在 Group Replication 运行时更改。更改立即在成员上生效。但是，已经接收到系统变量先前值的加入成员将继续使用该值。只有在值更改后加入的成员才会接收新值。

    `group_replication_advertise_recovery_endpoints`指定加入成员如何为分布式恢复的状态传输与现有成员建立连接。该连接用于远程克隆操作和从捐赠者的二进制日志进行状态传输。

    值为`DEFAULT`，即默认设置，表示加入成员使用现有成员的标准 SQL 客户端连接，如 MySQL Server 的`hostname`和`port`系统变量所指定。如果`report_port`系统变量指定了替代端口号，则使用该端口号。性能模式表`replication_group_members`在`MEMBER_HOST`和`MEMBER_PORT`字段中显示此连接的地址和端口号。这是截至 MySQL 8.0.20 版本的组成员的行为。

    您可以指定一个或多个分布式恢复端点，而不是`DEFAULT`，现有成员向加入成员广告这些端点供其使用。提供分布式恢复端点可以让管理员单独控制分布式恢复流量，与对组成员的常规 MySQL 客户端连接分开。加入成员按照列表上指定的顺序依次尝试每个端点。

    将分布式恢复端点指定为逗号分隔的 IP 地址和端口号列表，例如：

    ```sql
    group_replication_advertise_recovery_endpoints= "127.0.0.1:3306,127.0.0.1:4567,[::1]:3306,localhost:3306"
    ```

    IPv4 和 IPv6 地址以及主机名可以任意组合使用。IPv6 地址必须在方括号中指定。主机名必须解析为本地 IP 地址。不能使用通配符地址格式，也不能指定空列表。请注意，标准 SQL 客户端连接不会自动包含在分布式恢复端点列表中。如果要将其用作端点，必须在列表中明确包含它。

    有关如何选择 IP 地址和端口作为分布式恢复端点，以及加入成员如何使用它们的详细信息，请参见 Section 20.5.4.1.1，“选择分布式恢复端点的地址”。要求的摘要如下：

    +   IP 地址不必为 MySQL Server 配置，但必须分配给服务器。

    +   端口必须使用`port`、`report_port`或`admin_port`系统变量为 MySQL Server 配置。

    +   如果使用`admin_port`，则需要为分布式恢复的复制用户授予适当的权限。

    +   IP 地址不需要添加到由`group_replication_ip_allowlist`或`group_replication_ip_whitelist`系统变量指定的组复制允许列表中。

    +   连接的 SSL 要求由`group_replication_recovery_ssl_*`选项指定。

+   `group_replication_allow_local_lower_version_join`

    | 命令行格式 | `--group-replication-allow-local-lower-version-join[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `group_replication_allow_local_lower_version_join` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    此系统变量的值可以在运行 Group Replication 时更改，但更改仅在您停止并重新启动组成员上的 Group Replication 后生效。

    `group_replication_allow_local_lower_version_join`允许当前服务器即使运行低于组的 MySQL 服务器版本也加入组。默认设置为`OFF`，如果运行低于现有组成员的版本，则不允许服务器加入复制组。此标准策略确保组的所有成员能够交换消息并应用事务。请注意，运行 MySQL 8.0.17 或更高版本的成员在检查兼容性时考虑发布的补丁版本。运行 MySQL 8.0.16 或更低版本，或 MySQL 5.7 的成员只考虑主要版本。

    仅在以下情况下将`group_replication_allow_local_lower_version_join`设置为`ON`：

    +   必须在紧急情况下将服务器添加到组中以提高组的容错能力，且只有旧版本可用。

    +   您希望为一个或多个复制组成员回滚升级，而无需关闭整个组并重新引导。

    警告

    将此选项设置为`ON`并不会使新成员与组兼容，并允许其加入组而没有任何防范措施防止现有成员的不兼容行为。为确保新成员的正确操作，请采取*以下两项*预防措施：

    1.  在运行较低版本的服务器加入组之前，请停止该服务器上的所有写操作。

    1.  从运行较低版本的服务器加入组的那一点开始，在组中的其他服务器上停止所有写操作。

    如果没有这些预防措施，运行较低版本的服务器很可能会遇到困难，并以错误终止。

+   `group_replication_auto_increment_increment`

    | 命令行格式 | `--group-replication-auto-increment-increment=#` |
    | --- | --- |
    | 系统变量 | `group_replication_auto_increment_increment` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `7` |
    | 最小值 | `1` |
    | 最大值 | `65535` |

    所有组成员的此系统变量应具有相同的值。在 Group Replication 运行时，您不能更改此系统变量的值。您必须停止 Group Replication，更改系统变量的值，然后在每个组成员上重新启动 Group Replication。在此过程中，系统变量的值允许在组成员之间有所不同，但是一些组成员上的事务可能会被回滚。

    `group_replication_auto_increment_increment`确定在此服务器实例上执行的事务的自增列的连续值之间的间隔。增加一个间隔可以避免在组成员上写入时选择重复的自增值，这会导致事务回滚。默认值 7 代表可用值的数量和复制组的允许最大大小（9 个成员）之间的平衡。如果您的组成员更多或更少，您可以在启动 Group Replication 之前将此系统变量设置为匹配预期的组成员数量。

    重要

    当`group_replication_single_primary_mode`为`ON`时，设置`group_replication_auto_increment_increment`不起作用。

    当在服务器实例上启动组复制时，服务器系统变量`auto_increment_increment`的值将更改为此值，服务器系统变量`auto_increment_offset`的值将更改为服务器 ID。当停止组复制时，这些更改将被还原。仅当`auto_increment_increment`和`auto_increment_offset`的默认值均为 1 时，才会进行这些更改和还原。如果它们的值已经从默认值修改过，则组复制不会更改它们。在 MySQL 8.0 中，当组复制处于单主模式时，系统变量也不会被修改，只有一个服务器进行写入。

+   `group_replication_autorejoin_tries`

    | 命令行格式 | `--group-replication-autorejoin-tries=#` |
    | --- | --- |
    | 引入版本 | 8.0.16 |
    | 系统变量 | `group_replication_autorejoin_tries` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值（≥ 8.0.21） | `3` |
    | 默认值（≤ 8.0.20） | `0` |
    | 最小值 | `0` |
    | 最大值 | `2016` |

    可以在组复制运行时更改此系统变量的值，并且更改会立即生效。当发生需要该行为的问题时，会读取系统变量的当前值。

    `group_replication_autorejoin_tries`指定成员在被驱逐或在达到`group_replication_unreachable_majority_timeout`设置之前无法联系到大多数组时，尝试自动重新加入组的次数。当成员被驱逐或无法联系到大多数组时，它会尝试重新加入（使用当前插件选项值），然后继续进行进一步的自动重新加入尝试，直到达到指定的尝试次数。在一次不成功的自动重新加入尝试后，成员会在下一次尝试之前等待 5 分钟。如果在没有成员重新加入或停止的情况下耗尽了指定的尝试次数，则成员将执行由`group_replication_exit_state_action`系统变量指定的操作。

    在 MySQL 8.0.20 版本之前，默认设置为 0，表示成员不会尝试自动重新加入。从 MySQL 8.0.21 开始，默认设置为 3，表示成员会自动尝试重新加入群组，每次间隔 5 分钟，最多可以指定 2016 次尝试。

    在自动重新加入尝试期间和之间，成员保持在超级只读模式，并且不接受写入，但仍然可以在成员上进行读取，随着时间的推移，过时读取的可能性会增加。如果无法容忍任何时间段内可能出现的过时读取，请将 `group_replication_autorejoin_tries` 设置为 0。有关自动重新加入功能的更多信息以及在选择此选项的值时的考虑事项，请参见 第 20.7.7.3 节，“自动重新加入”。

+   `group_replication_bootstrap_group`

    | 命令行格式 | `--group-replication-bootstrap-group[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `group_replication_bootstrap_group` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    `group_replication_bootstrap_group` 配置此服务器引导群组。此系统变量**仅**应在一个服务器上设置，并且**仅**在第一次启动群组或重新启动整个群组时设置。群组引导完成后，将此选项设置为 `OFF`。应在动态和配置文件中将其设置为 `OFF`。在运行群组时启动两个服务器或重新启动一个服务器并将此选项设置可能会导致人为的脑裂情况，即引导两个具有相同名称的独立群组。

    有关首次引导群组的说明，请参见 第 20.2.1.5 节，“引导群组”。有关在已执行和认证事务的情况下安全引导群组的说明，请参见 第 20.5.2 节，“重新启动群组”。

+   `group_replication_clone_threshold`

    | 命令行格式 | `--group-replication-clone-threshold=#` |
    | --- | --- |
    | 引入版本 | 8.0.17 |
    | 系统变量 | `group_replication_clone_threshold` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `9223372036854775807` |
    | 最小值 | `1` |
    | 最大值 | `9223372036854775807` |
    | 单位 | 事务 |

    在 Group Replication 运行时可以更改此系统变量的值，但更改只有在停止并重新启动组成员的 Group Replication 后才会生效。

    `group_replication_clone_threshold` 指定现有成员（捐赠者）和加入成员（接收者）之间的事务间隔，触发在分布式恢复过程中使用远程克隆操作将状态传输给加入成员。如果加入成员和合适的捐赠者之间的事务间隔超过阈值，则 Group Replication 将开始使用远程克隆操作进行分布式恢复。如果事务间隔低于阈值，或者远程克隆操作在技术上不可行，则 Group Replication 直接从捐赠者的二进制日志进行状态传输。

    警告

    在活动组中不要使用低设置的 `group_replication_clone_threshold`。如果在远程克隆操作正在进行时组中发生超过阈值的事务数量，加入成员在重新启动后会再次触发远程克隆操作，并可能无限继续。为避免这种情况，请确保将阈值设置为预期在远程克隆操作所需时间内组中可能发生的事务数量更高的数字。

    要使用此功能，捐赠者和加入成员必须事先设置为支持克隆。有关说明，请参见 Section 20.5.4.2, “分布式恢复的克隆”。当执行远程克隆操作时，Group Replication 会为您管理它，包括所需的服务器重新启动，前提是设置了 `group_replication_start_on_boot=ON`。如果没有设置，则必须手动重新启动服务器。远程克隆操作会替换加入成员上的现有数据字典，但如果加入成员有其他组成员上不存在的额外事务，则 Group Replication 会进行检查并不继续进行，因为这些事务将被克隆操作擦除。

    默认设置（即 GTID 中事务的最大允许序列号）意味着几乎总是尝试从捐赠者的二进制日志进行状态传输，而不是克隆。但是，请注意，无论您的阈值如何，Group Replication 始终尝试执行克隆操作，如果从捐赠者的二进制日志中无法进行状态传输，例如因为加入成员所需的事务在任何现有组成员的二进制日志中都不可用。如果您不希望在复制组中完全使用克隆，请不要在成员上安装克隆插件。

+   `group_replication_communication_debug_options`

    | 命令行格式 | `--group-replication-communication-debug-options=value` |
    | --- | --- |
    | 系统变量 | `group_replication_communication_debug_options` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 字符串 |
    | 默认值 | `GCS_DEBUG_NONE` |
    | 有效数值 | `GCS_DEBUG_NONE``GCS_DEBUG_BASIC``GCS_DEBUG_TRACE``XCOM_DEBUG_BASIC``XCOM_DEBUG_TRACE``GCS_DEBUG_ALL` |

    在 Group Replication 运行时可以更改此系统变量的值，并立即生效。

    `group_replication_communication_debug_options` 配置不同 Group Replication 组件（如 Group Communication System（GCS）和组通信引擎（XCom，一种 Paxos 变体））提供的调试消息级别。调试信息存储在数据目录中的`GCS_DEBUG_TRACE`文件中。

    可以组合作为字符串指定的可用选项集。以下选项可用：

    +   `GCS_DEBUG_NONE` 禁用 GCS 和 XCom 的所有调试级别。

    +   `GCS_DEBUG_BASIC` 在 GCS 中启用基本调试信息。

    +   `GCS_DEBUG_TRACE` 在 GCS 中启用跟踪信息。

    +   `XCOM_DEBUG_BASIC` 在 XCom 中启用基本调试信息。

    +   `XCOM_DEBUG_TRACE` 在 XCom 中启用跟��信息。

    +   `GCS_DEBUG_ALL` 在 GCS 和 XCom 中启用所有调试级别。

    将调试级别设置为`GCS_DEBUG_NONE`仅在没有任何其他选项的情况下提供时才生效。将调试级别设置为`GCS_DEBUG_ALL`会覆盖所有其他选项。

+   `group_replication_communication_max_message_size`

    | 命令行格式 | `--group-replication-communication-max-message-size=#` |
    | --- | --- |
    | 引入 | 8.0.16 |
    | 系统变量 | `group_replication_communication_max_message_size` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `10485760` |
    | 最小值 | `0` |
    | 最大值 | `1073741824` |
    | 单位 | 字节 |

    所有组成员应该具有相同的系统变量值。在组复制运行时，无法更改此系统变量的值。您必须停止组复制，更改系统变量的值，然后在每个组成员上重新启动组复制。在此过程中，系统变量的值允许在组成员之间有所不同，但某些组成员上的事务可能会被回滚。

    `group_replication_communication_max_message_size`指定了组复制通信的最大消息大小。超过此大小的消息会自动分割成片段单独发送，并由接收方重新组装。更多信息，请参见第 20.7.5 节，“消息分段”。

    默认情况下，最大消息大小为 10485760 字节（10 MiB），这意味着在 MySQL 8.0.16 版本中默认使用分段。最大允许值与`replica_max_allowed_packet`和`slave_max_allowed_packet`系统变量的最大值相同，即 1073741824 字节（1 GB）。`group_replication_communication_max_message_size`的设置必须小于`replica_max_allowed_packet`或`slave_max_allowed_packet`的设置，因为应用程序线程无法处理大于最大允许数据包大小的消息片段。要关闭分段，请为`group_replication_communication_max_message_size`指定零值。

    为了使复制组的成员使用分段，组的通信协议版本必须是 MySQL 8.0.16 或更高。使用`group_replication_get_communication_protocol()`函数查看组的通信协议版本。如果使用较低版本，则组成员不会分段消息。如果所有组成员支持，可以使用`group_replication_set_communication_protocol()`函数将组的通信协议设置为更高版本。更多信息，请参阅 Section 20.5.1.4, “设置组的通信协议版本”。

+   `group_replication_communication_stack`

    | 引入版本 | 8.0.27 |
    | --- | --- |
    | 系统变量 | `group_replication_communication_stack` |
    | 作用范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 字符串 |
    | 默认值 | `XCOM` |
    | 有效值 | `XCOM``MYSQL` |

    注意

    这个系统变量实际上是一个整个组的配置设置，更改生效需要对复制组进行完全重启。

    `group_replication_communication_stack`指定了是使用 XCom 通信栈还是 MySQL 通信栈来建立组成员之间的组通信连接。XCom 通信栈是 Group Replication 自己的实现，在 MySQL 8.0.27 之前的所有版本中始终使用，并且不支持认证或网络命名空间。MySQL 通信栈是 MySQL 服务器的本机实现，支持认证和网络命名空间，并在发布后立即访问新的安全功能。组的所有成员必须使用相同的通信栈。

    当你使用 MySQL 的通信栈代替 XCom 时，MySQL 服务器使用自己的认证和加密协议在组成员之间建立每个连接。

    注意

    如果你正在使用 InnoDB Cluster，`group_replication_communication_stack`的默认值是`MYSQL`。

    更多信息，请参阅 MySQL InnoDB Cluster。

    在设置组使用 MySQL 通信堆栈时需要进行额外配置；请参阅 第 20.6.1 节，“连接安全管理的通信堆栈”。

    `group_replication_communication_stack` 实际上是一个组范围的配置设置，所有组成员必须具有相同的设置。但是，Group Replication 对于组范围配置设置并不执行检查。具有与其余组不同值的成员无法与其他成员通信，因为通信协议不兼容，因此无法交换有关其配置设置的信息。

    这意味着虽然在 Group Replication 运行时可以更改系统变量的值，并在重新启动组成员的情况下生效，但成员仍然无法重新加入组，直到在所有成员上更改了设置。因此，您必须在所有成员上停止 Group Replication 并更改系统变量的值，然后才能重新启动组。由于所有成员都已停止，因此需要对组进行完全重启（由具有 `group_replication_bootstrap_group=ON` 的服务器引导）以使值更改生效。有关从一种通信堆栈迁移到另一种的说明，请参阅 第 20.6.1 节，“连接安全管理的通信堆栈”。

+   `group_replication_components_stop_timeout`

    | 命令行格式 | `--group-replication-components-stop-timeout=#` |
    | --- | --- |
    | 系统变量 | `group_replication_components_stop_timeout` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值（≥ 8.0.27） | `300` |
    | 默认值（≤ 8.0.26） | `31536000` |
    | 最小值 | `2` |
    | 最大值 | `31536000` |
    | 单位 | 秒 |

    此系统变量的值可以在 Group Replication 运行时更改，但更改只有在您停止并重新启动组成员上的 Group Replication 后才会生效。

    `group_replication_components_stop_timeout` 指定 Group Replication 在关闭时等待其各个模块完成正在进行的进程的时间，单位为秒。组件超时在发出 `STOP GROUP_REPLICATION` 语句后应用，该语句在服务器重新启动或自动重新加入时会自动执行。

    超时时间用于解决 Group Replication 组件无法正常停止的情况，这可能发生在成员处于错误状态时被驱逐出组，或者在诸如 MySQL Enterprise Backup 正在持有成员上的表的全局锁时。在这种情况下，成员无法停止应用程序线程或完成分布式恢复过程以重新加入。`STOP GROUP_REPLICATION` 不会完成，直到情况得到解决（例如，通过释放锁），或者组件超时到期并且模块无论其状态如何都会被关闭。

    在 MySQL 8.0.27 之前，默认组件超时时间为 31536000 秒，即 365 天。在这种情况下，组件超时对于描述的情况并不起作用，因此建议设置一个较低的值。从 MySQL 8.0.27 开始，默认值为 300 秒，因此如果在此时间之前未解决问题，Group Replication 组件将在 5 分钟后停止，允许成员重新启动并重新加入。

+   `group_replication_compression_threshold`

    | 命令行格式 | `--group-replication-compression-threshold=#` |
    | --- | --- |
    | 系统变量 | `group_replication_compression_threshold` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `1000000` |
    | 最小值 | `0` |
    | 最大值 | `4294967295` |
    | 单位 | 字节 |

    在组成员之间发送的消息超过此阈值字节时应用压缩。如果此系统变量设置为零，则禁用压缩。`group_replication_compression_threshold` 的值应在所有组成员上保持一致。

    Group Replication 使用 LZ4 压缩算法来压缩组内发送的消息。请注意，LZ4 压缩算法支持的最大输入大小为 2113929216 字节。此限制低于 `group_replication_compression_threshold` 系统变量的最大可能值，该值与 XCom 接受的最大消息大小相匹配。使用 LZ4 压缩算法时，请不要为 `group_replication_compression_threshold` 设置大于 2113929216 字节的值，因为启用消息压缩时，超过此大小的事务无法提交。

    更多信息，请参见 第 20.7.4 节，“消息压缩”。

+   `group_replication_consistency`

    | 命令行格式 | `--group-replication-consistency=value` |
    | --- | --- |
    | 引入版本 | 8.0.14 |
    | 系统变量 | `group_replication_consistency` |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 枚举 |
    | 默认值 | `EVENTUAL` |
    | 有效值 | `EVENTUAL``BEFORE_ON_PRIMARY_FAILOVER``BEFORE``AFTER``BEFORE_AND_AFTER` |

    在运行 Group Replication 时可以更改此系统变量的值。`group_replication_consistency` 是一个服务器系统变量，而不是 Group Replication 插件特定的变量，因此不需要重新启动 Group Replication 才能使更改生效。更改系统变量的会话值会立即生效，更改全局值会影响在更改后启动的新会话。需要 `GROUP_REPLICATION_ADMIN` 权限才能更改此系统变量的全局设置。

    `group_replication_consistency` 控制组提供的事务一致性保证。您可以全局配置一致性，也可以针对每个事务进行配置。`group_replication_consistency` 还配置了单主组中新选举的主节点使用的围栏机制。必须考虑该变量对只读（RO）和读写（RW）事务的影响。以下列表显示了该变量的可能值，按照事务一致性保证递增的顺序排列：

    +   `EVENTUAL`

        在执行之前，RO 和 RW 事务都不会等待前置事务被应用。这是在添加此变量之前 Group Replication 的行为。一个 RW 事务不会等待其他成员应用事务。这意味着一个事务在其他成员之前可能在一个成员上被外部化。这也意味着在主要故障转移发生时，新的主要成员可以在之前的主要事务全部应用之前接受新的 RO 和 RW 事务。RO 事务可能导致过时的值，RW 事务可能由于冲突而导致回滚。

    +   `BEFORE_ON_PRIMARY_FAILOVER`

        具有新选举的主要成员正在应用来自旧主要成员的积压时，新的 RO 或 RW 事务将被保留（不被应用），直到任何积压都被应用。这确保了主要故障转移发生时，无论是故意还是非故意，客户端始终看到主要成员上的最新值。这保证了一致性，但意味着客户端必须能够处理应用积压时的延迟。通常这种延迟应该是最小的，但取决于积压的大小。

    +   `BEFORE`

        一个 RW 事务在被应用之前等待所有前置事务完成。一个 RO 事务在执行之前等待所有前置事务完成。这确保了该事务通过仅影响事务的延迟来读取最新值。这减少了每个 RW 事务上同步的开销，通过确保同步仅在 RO 事务上使用。这种一致性级别还包括`BEFORE_ON_PRIMARY_FAILOVER`提供的一致性保证。

    +   `AFTER`

        一个 RW 事务等待直到其更改已应用到所有其他成员。这个值对 RO 事务没有影响。这种模式确保了当事务在本地成员上提交时，任何后续事务都会读取任何组成员上写入的值或更近期的值。在主要用于 RO 操作的组中使用此模式，以确保一旦提交，应用的 RW 事务就会在所有地方应用。您的应用程序可以使用此模式确保后续读取获取包含最新写入的最新数据。这减少了每个 RO 事务上同步的开销，通过确保同步仅在 RW 事务上使用。这种一致性级别还包括`BEFORE_ON_PRIMARY_FAILOVER`提供的一致性保证。

    +   `BEFORE_AND_AFTER`

        一个 RW 事务等待 1) 所有前置事务完成后才被应用，2) 直到其更改在其他成员上被应用。一个 RO 事务在执行之前等待所有前置事务完成。这种一致性级别还包括`BEFORE_ON_PRIMARY_FAILOVER`提供的一致性保证。

    有关更多信息，请参见 Section 20.5.3, “事务一致性保证”。

+   `group_replication_enforce_update_everywhere_checks`

    | 命令行格式 | `--group-replication-enforce-update-everywhere-checks[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `group_replication_enforce_update_everywhere_checks` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    注意

    这个系统变量是一个群组范围的配置设置，需要对复制组进行完全重启才能使更改生效。

    `group_replication_enforce_update_everywhere_checks` 启用或禁用多主更新到处的严格一致性检查。默认情况下，检查是禁用的。在单主模式下，所有群组成员必须禁用此选项。在多主模式下，当启用此选项时，语句将按以下方式进行检查，以确保它们与多主模式兼容：

    +   如果事务在 `SERIALIZABLE` 隔离级别下执行，则在与群组同步时其提交将失败。

    +   如果事务针对具有级联约束的外键的表执行，则在与群组同步时，事务将无法提交。

    这个系统变量是一个群组范围的配置设置。在所有群组成员上必须具有相同的值，在 Group Replication 运行时不能更改，并且需要通过一个具有 `group_replication_bootstrap_group=ON` 的服务器进行完全重启群组（引导）以使值更改生效。有关在已执行和认证事务的情况下安全引导群组的说明，请参见 Section 20.5.2, “重新启动群组”。

    如果群组为此系统变量设置了一个值，并且加入的成员为该系统变量设置了不同的值，则加入的成员在将值更改为匹配之前无法加入群组。如果群组成员为此系统变量设置了一个值，而加入的成员不支持该系统变量，则无法加入群组。

    从 MySQL 8.0.16 版本开始，您可以使用`group_replication_switch_to_single_primary_mode()`和`group_replication_switch_to_multi_primary_mode()`函数在群组仍在运行时更改此系统变量的值。有关更多信息，请参见第 20.5.1.2 节，“更改群组模式”。

+   `group_replication_exit_state_action`

    | 命令行格式 | `--group-replication-exit-state-action=value` |
    | --- | --- |
    | 引入版本 | 8.0.12 |
    | 系统变量 | `group_replication_exit_state_action` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 枚举 |
    | 默认值（≥ 8.0.16） | `READ_ONLY` |
    | 默认值（≥ 8.0.12, ≤ 8.0.15） | `ABORT_SERVER` |
    | 有效取值（≥ 8.0.18） | `ABORT_SERVER``OFFLINE_MODE``READ_ONLY` |
    | 有效取值（≥ 8.0.12, ≤ 8.0.17） | `ABORT_SERVER``READ_ONLY` |

    当 Group Replication 在运行时，可以更改此系统变量的值，并且更改会立即生效。当发生需要该行为的问题时，会读取系统变量的当前值。

    `group_replication_exit_state_action` 配置了当此服务器实例意外离开群组时 Group Replication 的行为，例如在遇到应用程序错误后，或在丢失多数情况下，或当群组的另一个成员因超时而将其驱逐时。在丢失多数情况下，成员离开群组的超时期由`group_replication_unreachable_majority_timeout`系统变量设置，而对于怀疑的超时期由`group_replication_member_expel_timeout`系统变量设置。请注意，被驱逐的群组成员在重新连接到群组之前不知道自己被驱逐，因此只有在成员成功重新连接或成员对自己提出怀疑并驱逐自己时才会执行指定的操作。

    当由于超时的怀疑或多数丢失而将组成员驱逐时，如果成员将`group_replication_autorejoin_tries`系统变量设置为指定自动重新加入尝试次数，则首先在超级只读模式下进行指定次数的尝试，然后按照`group_replication_exit_state_action`指定的操作进行。在出现应用程序错误时不会进行自动重新加入尝试，因为这些错误无法恢复。

    当`group_replication_exit_state_action`设置为`READ_ONLY`时，如果成员意外退出组或耗尽自动重新加入尝试次数，则实例将将 MySQL 切换到超级只读模式（通过将系统变量`super_read_only`设置为`ON`）。`READ_ONLY`退出操作是在引入系统变量之前的 MySQL 8.0 版本中的行为，并且从 MySQL 8.0.16 开始再次成为默认设置。

    当`group_replication_exit_state_action`设置为`OFFLINE_MODE`时，如果成员意外退出组或耗尽自动重新加入尝试次数，则实例将将 MySQL 切换到离线模式（通过将系统变量`offline_mode`设置为`ON`）。在此模式下，连接的客户端用户在下一次请求时将被断开连接，不再接受连接，但具有`CONNECTION_ADMIN`权限（或已弃用的`SUPER`权限）的客户端用户除外。Group Replication 还将系统变量`super_read_only`设置为`ON`，因此即使使用`CONNECTION_ADMIN`或`SUPER`权限连接，客户端也无法进行任何更新。`OFFLINE_MODE`退出操作从 MySQL 8.0.18 开始提供。

    当`group_replication_exit_state_action`设置为`ABORT_SERVER`时，如果成员意外退出组或耗尽自动重新加入尝试次数，则实例将关闭 MySQL。此设置从 MySQL 8.0.12（添加系统变量时）到 MySQL 8.0.15（含）期间为默认设置。

    重要提示

    如果成员成功加入组之前发生故障，则指定的退出操作*不会执行*。如果在本地配置检查期间发生故障，或者加入成员的配置与组的配置不匹配，则会出现这种情况。在这些情况下，`super_read_only`系统变量保持其原始值，继续接受连接，并且服务器不会关闭 MySQL。因此，为了确保在 Group Replication 未启动时服务器无法接受更新，我们建议在服务器启动时在配置文件中设置`super_read_only=ON`，Group Replication 在成功启动后会将其更改为`OFF`。当服务器配置为在服务器启动时启动 Group Replication（`group_replication_start_on_boot=ON`命令手动启动 Group Replication 时也很有用。

    有关使用此选项的更多信息以及采取退出操作的全部情况，请参见 Section 20.7.7.4, “退出操作”。

+   `group_replication_flow_control_applier_threshold`

    | 命令行格式 | `--group-replication-flow-control-applier-threshold=#` |
    | --- | --- |
    | 系统变量 | `group_replication_flow_control_applier_threshold` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `25000` |
    | 最小值 | `0` |
    | 最大值 | `2147483647` |
    | 单位 | 事务 |

    此系统变量的值可以在 Group Replication 运行时更改，并立即生效。

    `group_replication_flow_control_applier_threshold`指定了在应用程序队列中等待的事务数量，触发流量控制。

+   `group_replication_flow_control_certifier_threshold`

    | 命令行格式 | `--group-replication-flow-control-certifier-threshold=#` |
    | --- | --- |
    | 系统变量 | `group_replication_flow_control_certifier_threshold` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `25000` |
    | 最小值 | `0` |
    | 最大值 | `2147483647` |
    | 单位 | 事务 |

    此系统变量的值可以在 Group Replication 运行时更改，并立即生效。

    `group_replication_flow_control_certifier_threshold` 指定了在认证者队列中等待的事务数量触发流量控制。

+   `group_replication_flow_control_hold_percent`

    | 命令行格式 | `--group-replication-flow-control-hold-percent=#` |
    | --- | --- |
    | 系统变量 | `group_replication_flow_control_hold_percent` |
    | 作用域 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `10` |
    | 最小值 | `0` |
    | 最大值 | `100` |
    | 单位 | 百分比 |

    此系统变量的值可以在 Group Replication 运行时更改，并立即生效。

    `group_replication_flow_control_hold_percent` 定义了集群配额剩余未使用的百分比，以允许处于流量控制状态的集群赶上积压工作。 值为 0 意味着没有配额的任何部分用于赶上工作积压。

+   `group_replication_flow_control_max_quota`

    | 命令行格式 | `--group-replication-flow-control-max-quota=#` |
    | --- | --- |
    | 系统变量 | `group_replication_flow_control_max_quota` |
    | 作用域 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `0` |
    | 最小值 | `0` |
    | 最大值 | `2147483647` |

    此系统变量的值可以在 Group Replication 运行时更改，并立即生效。

    `group_replication_flow_control_max_quota` 定义了组的最大流量控制配额，或者在启用流量控制时任何时期的最大可用配额。值为 0 意味着没有设置最大配额。此系统变量的值不能小于`group_replication_flow_control_min_quota`和`group_replication_flow_control_min_recovery_quota`。

+   `group_replication_flow_control_member_quota_percent`

    | 命令行格式 | `--group-replication-flow-control-member-quota-percent=#` |
    | --- | --- |
    | 系统变量 | `group_replication_flow_control_member_quota_percent` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `0` |
    | 最小值 | `0` |
    | 最大值 | `100` |
    | 单位 | 百分比 |

    此系统变量的值可以在 Group Replication 运行时更改，并立即生效。

    `group_replication_flow_control_member_quota_percent` 定义了成员在计算配额时应该假定自己可用的配额百分比。值为 0 意味着配额应该在上一个时期是写入者的成员之间均匀分配。

+   `group_replication_flow_control_min_quota`

    | 命令行格式 | `--group-replication-flow-control-min-quota=#` |
    | --- | --- |
    | 系统变量 | `group_replication_flow_control_min_quota` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `0` |
    | 最小值 | `0` |
    | 最大值 | `2147483647` |

    此系统变量的值可以在 Group Replication 运行时更改，并立即生效。

    `group_replication_flow_control_min_quota` 控制着可以分配给成员的最低流量控制配额，独立于上一个周期执行的计算最小配额。数值为 0 意味着没有最低配额。此系统变量的值不能大于`group_replication_flow_control_max_quota`。

+   `group_replication_flow_control_min_recovery_quota`

    | 命令行格式 | `--group-replication-flow-control-min-recovery-quota=#` |
    | --- | --- |
    | 系统变量 | `group_replication_flow_control_min_recovery_quota` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `0` |
    | 最小值 | `0` |
    | 最大值 | `2147483647` |

    此系统变量的值可以在 Group Replication 运行时更改，并立即生效。

    `group_replication_flow_control_min_recovery_quota` 控制着因为组中另一个正在恢复的成员而可以分配给成员的最低配额，独立于上一个周期执行的计算最小配额。数值为 0 意味着没有最低配额。此系统变量的值不能大于`group_replication_flow_control_max_quota`。

+   `group_replication_flow_control_mode`

    | 命令行格式 | `--group-replication-flow-control-mode=value` |
    | --- | --- |
    | 系统变量 | `group_replication_flow_control_mode` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 枚举 |
    | 默认值 | `QUOTA` |
    | 有效值 | `DISABLED``QUOTA` |

    此系统变量的值可以在 Group Replication 运行时更改，并立即生效。

    `group_replication_flow_control_mode` 指定了流量控制的模式。

+   `group_replication_flow_control_period`

    | 命令行格式 | `--group-replication-flow-control-period=#` |
    | --- | --- |
    | 系统变量 | `group_replication_flow_control_period` |
    | 范围 | 全局 |
    | Dynamic | 是 |
    | `SET_VAR` Hint Applies | No |
    | 类型 | 整数 |
    | 默认值 | `1` |
    | 最小值 | `1` |
    | 最大值 | `60` |
    | 单位 | 秒 |

    此系统变量的值可以在 Group Replication 运行时更改，并立即生效。

    `group_replication_flow_control_period` 定义了在流量控制迭代之间等待多少秒，其中发送流量控制消息并运行流量控制管理任务。

+   `group_replication_flow_control_release_percent`

    | 命令行格式 | `--group-replication-flow-control-release-percent=#` |
    | --- | --- |
    | 系统变量 | `group_replication_flow_control_release_percent` |
    | 范围 | 全局 |
    | Dynamic | Yes |
    | `SET_VAR` Hint Applies | No |
    | 类型 | 整数 |
    | 默认值 | `50` |
    | 最小值 | `0` |
    | 最大值 | `1000` |
    | 单位 | 百分比 |

    此系统变量的值可以在 Group Replication 运行时更改，并立即生效。

    `group_replication_flow_control_release_percent` 定义了当流量控制不再需要限制写入成员时，如何释放组配额，其中百分比是每个流量控制周期的配额增加。 值为 0 意味着一旦流量控制阈值在限制范围内，配额就会在单个流量控制迭代中释放。 该范围允许配额释放高达当前配额的 10 倍，这样可以更好地适应，主要是当流量控制周期较长且配额非常小时。

+   `group_replication_force_members`

    | 命令行格式 | `--group-replication-force-members=value` |
    | --- | --- |
    | 系统变量 | `group_replication_force_members` |
    | 范围 | 全局 |
    | Dynamic | 是 |
    | `SET_VAR` Hint Applies | No |
    | 类型 | 字符串 |

    此系统变量用于强制应用新的组成员。在 Group Replication 运行时，可以更改此系统变量的值，并立即生效。您只需要在要保留在组中的一个组成员上设置系统变量的值。有关可能需要强制应用新的组成员的情况以及在使用此系统变量时要遵循的程序的详细信息，请参见第 20.7.8 节，“处理网络分区和失去法定人数”。

    `group_replication_force_members`指定一组对等地址，以逗号分隔的列表形式，例如`host1:port1`，`host2:port2`。未包含在列表中的任何现有成员将不会接收到组的新视图，并将被阻止。对于每个要继续作为成员的现有成员，您必须包括 IP 地址或主机名以及端口，就像在`group_replication_local_address`系统变量中为每个成员给出的那样。IPv6 地址必须用方括号指定。例如：

    ```sql
    "198.51.100.44:33061,[2001:db8:85a3:8d3:1319:8a2e:370:7348]:33061,example.org:33061"
    ```

    Group Replication 的组通信引擎（XCom）检查提供的 IP 地址是否具有有效格式，并检查您是否包含了当前无法访问的任何组成员。否则，新配置将不被验证，因此您必须小心地只包括在线可访问的组成员。列表中的任何不正确值或无效主机名都可能导致组被阻止，出现无效配置。

    在强制应用新的成员配置之前，确保要排除的服务器已关闭是很重要的。如果没有关闭，请在继续之前将它们关闭。仍然在线的组成员可以自动形成新的配置，如果已经发生了这种情况，强制进一步的新配置可能会为组创建人为的脑裂情况。

    在使用`group_replication_force_members`系统变量成功强制应用新的组成员并解除组阻塞后，请确保清除该系统变量。为了发出`START GROUP_REPLICATION`语句，`group_replication_force_members`必须为空。

+   `group_replication_group_name`

    | 命令行格式 | `--group-replication-group-name=value` |
    | --- | --- |
    | 系统变量 | `group_replication_group_name` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 字符串 |

    此系统变量的值在 Group Replication 运行时无法更改。

    `group_replication_group_name` 指定了此服务器实例所属的组的名称，必须是有效的 UUID。这个 UUID 是 GTID 的一部分，当组成员从客户端接收到事务，以及组成员内部生成的视图更改事件被写入二进制日志时使用。

    重要提示

    必须使用唯一的 UUID。

+   `group_replication_group_seeds`

    | 命令行格式 | `--group-replication-group-seeds=value` |
    | --- | --- |
    | 系统变量 | `group_replication_group_seeds` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 字符串 |

    此系统变量的值可以在 Group Replication 运行时更改，但更改只有在您停止并重新启动组成员的 Group Replication 后才会生效。

    `group_replication_group_seeds` 是一个组成员的列表，加入的成员可以连接到这些组成员以获取所有当前组成员的详细信息。加入的成员使用这些详细信息来选择并连接到一个组成员，以获取与组的同步所需的数据。该列表包含每个包含的种子成员的单个内部网络地址或主机名，如在种子成员的 `group_replication_local_address` 系统变量中配置的那样（而不是种子成员的 SQL 客户端连接，如 MySQL Server 的 `hostname` 和 `port` 系统变量所指定的）。种子成员的地址被指定为逗号分隔的列表，例如 `host1:port1`,`host2:port2`。IPv6 地址必须用方括号指定。例如：

    ```sql
    group_replication_group_seeds= "198.51.100.44:33061,[2001:db8:85a3:8d3:1319:8a2e:370:7348]:33061, example.org:33061"
    ```

    请注意，您为此变量指定的值在发出 `START GROUP_REPLICATION` 语句并且 Group Communication System (GCS) 可用之前不会被验证。

    通常，此列表包含组的所有成员，但您可以选择一组成员的子集作为种子。列表必须至少包含一个有效成员地址。在启动 Group Replication 时，每个地址都会被验证。如果列表不包含任何有效成员地址，则发出`START GROUP_REPLICATION`将失败。

    当服务器加入复制组时，它会尝试连接到其`group_replication_group_seeds`系统变量中列出的第一个种子成员。如果连接被拒绝，加入成员会尝试按顺序连接列表中的其他种子成员。如果加入成员连接到一个种子成员，但由于某种原因未被添加到复制组（例如，因为种子成员没有在其允许列表中包含加入成员的地址并关闭了连接），则加入成员会继续按顺序尝试剩余的种子成员。

    加入成员必须使用与种子成员在`group_replication_group_seeds`选项中广告的协议（IPv4 或 IPv6）与种子成员进行通信。对于 Group Replication 的 IP 地址权限，种子成员的允许列表必须包含加入成员的 IP 地址，以便与种子成员提供的协议匹配，或者解析为该协议的地址的主机名。如果加入成员的协议与种子成员广告的协议不匹配，则必须设置并允许此地址或主机名，除了加入成员的`group_replication_local_address`。如果加入成员没有适当协议的允许地址，则其连接尝试将被拒绝。有关更多信息，请参见第 20.6.4 节，“Group Replication IP 地址权限”。

+   `group_replication_gtid_assignment_block_size`

    | 命令行格式 | `--group-replication-gtid-assignment-block-size=#` |
    | --- | --- |
    | 系统变量 | `group_replication_gtid_assignment_block_size` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `1000000` |
    | 最小值 | `1` |
    | 最大值（64 位平台） | `9223372036854775807` |
    | 最大值（32 位平台） | `4294967295` |

    注意

    这个系统变量是一个群组范围的配置设置，需要对复制组进行完全重启才能使更改生效。

    `group_replication_gtid_assignment_block_size` 指定为每个组成员保留的连续 GTID 数。每个成员消耗自己的块，并在需要时保留更多。

    这个系统变量是一个群组范围的配置设置。在所有群组成员上必须具有相同的值，不能在 Group Replication 运行时更改，并且需要对群组进行完全重启（由具有 `group_replication_bootstrap_group=ON` 的服务器引导）才能使值更改生效。有关在已执行和认证事务的情况下安全引导群组的说明，请参见 第 20.5.2 节，“重新启动群组”。

    如果群组为此系统变量设置了一个值，并且加入成员为该系统变量设置了不同的值，则加入成员无法加入群组，直到将值更改为匹配。如果群组成员为此系统变量设置了一个值，而加入成员不支持该系统变量，则无法加入群组。

+   `group_replication_ip_allowlist`

    | 命令行格式 | `--group-replication-ip-allowlist=value` |
    | --- | --- |
    | 引入版本 | 8.0.22 |
    | 系统变量 | `group_replication_ip_allowlist` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 字符串 |
    | 默认值 | `AUTOMATIC` |

    `group_replication_ip_allowlist` 在 MySQL 8.0.22 中可用，用于替换 `group_replication_ip_whitelist`。从 MySQL 8.0.24 开始，可以在 Group Replication 运行时更改此系统变量的值，并且更改立即在成员上生效。

    `group_replication_ip_allowlist`指定哪些主机被允许连接到组。当 XCom 通信堆栈用于组时（`group_replication_communication_stack=XCOM`），白名单用于控制对组的访问。当 MySQL 通信堆栈用于组时（`group_replication_communication_stack=MYSQL`），用户认证用于控制对组的访问，白名单不会被使用，如果设置了则会被忽略。

    您在`group_replication_local_address`中为每个组成员指定的地址必须在复制组中的其他服务器上得到允许。请注意，直到发出`START GROUP_REPLICATION`语句并且组通信系统（GCS）可用之前，您为此变量指定的值不会被验证。

    默认情况下，此系统变量设置为`AUTOMATIC`，允许来自主机上活动的私有子网的连接。组复制的组通信引擎（XCom）会自动扫描主机上的活动接口，并识别具有私有子网地址的接口。这些地址以及 IPv4 和（从 MySQL 8.0.14 开始）IPv6 的`localhost` IP 地址用于创建组复制白名单。有关自动允许地址的范围列表，请参见 Section 20.6.4, “Group Replication IP Address Permissions”。

    自动私有地址白名单不能用于来自私有网络之外的服务器的连接。对于位于不同机器上的服务器实例之间的组复制连接，您必须提供公共 IP 地址并将其指定为显式白名单。如果为白名单指定了任何条目，则私有地址不会自动添加，因此如果使用其中任何一个，必须明确指定。`localhost` IP 地址会自动添加。

    作为`group_replication_ip_allowlist`选项的值，您可以指定以下任意组合：

    +   IPv4 地址（例如，`198.51.100.44`）

    +   具有 CIDR 表示法的 IPv4 地址（例如，`192.0.2.21/24`）

    +   IPv6 地址，从 MySQL 8.0.14 开始（例如，`2001:db8:85a3:8d3:1319:8a2e:370:7348`）

    +   具有 CIDR 表示法的 IPv6 地址，从 MySQL 8.0.14 开始（例如，`2001:db8:85a3:8d3::/64`）

    +   主机名（例如，`example.org`）

    +   具有 CIDR 表示法的主机名（例如，`www.example.com/24`)

    在 MySQL 8.0.14 之前，主机名只能解析为 IPv4 地址。从 MySQL 8.0.14 开始，主机名可以解析为 IPv4 地址、IPv6 地址或两者都有。如果主机名解析为 IPv4 和 IPv6 地址，始终使用 IPv4 地址进行组复制连接。您可以结合主机名或 IP 地址使用 CIDR 表示法来允许具有特定网络前缀的 IP 地址块，但确保指定子网中的所有 IP 地址都在您的控制之下。

    每个允许列表中的条目之间必须用逗号分隔。例如：

    ```sql
    "192.0.2.21/24,198.51.100.44,203.0.113.0/24,2001:db8:85a3:8d3:1319:8a2e:370:7348,example.org,www.example.com/24"
    ```

    如果组的任何种子成员在加入成员具有 IPv4 `group_replication_local_address`时列出了带有 IPv6 地址的`group_replication_group_seeds`选项，或反之亦然，则还必须设置和允许加入成员的另一地址，以供种子成员提供的协议使用（或解析为该协议的地址的主机名）。有关更多信息，请参见第 20.6.4 节，“组复制 IP 地址权限”。

    可以根据您的安全需求在不同的组成员上配置不同的允许列表，例如，为了保持不同的子网分开。然而，当重新配置组时可能会导致问题。如果没有特定的安全要求要求做出其他安排，请在组的所有成员上使用相同的允许列表。有关更多详细信息，请参见第 20.6.4 节，“组复制 IP 地址权限”。

    对于主机名，只有在另一个服务器发出连接请求时才会进行名称解析。无法解析的主机名不会被视为允许列表验证的一部分，并且会向错误日志中写入警告消息。对已解析的主机名执行前向确认反向 DNS（FCrDNS）验证。

    警告

    主机名在允许列表中比 IP 地址不安全。FCrDNS 验证提供了很好的保护级别，但可能会受到某些类型攻击的影响。仅在绝对必要时在允许列表中指定主机名，并确保所有用于名称解析的组件，如 DNS 服务器，都在您的控制之下。您还可以使用 hosts 文件在本地实现名称解析，以避免使用外部组件。

+   `group_replication_ip_whitelist`

    | 命令行格式 | `--group-replication-ip-whitelist=value` |
    | --- | --- |
    | 已弃用 | 8.0.22 |
    | 系统变量 | `group_replication_ip_whitelist` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | [`SET_VAR`](https://optimizer-hints.html#optimizer-hints-set-var "变量设置提示语法")提示适用 | 否 |
    | 类型 | 字符串 |
    | 默认值 | `AUTOMATIC` |

    从 MySQL 8.0.22 开始，[`group_replication_ip_whitelist`](https://group-replication-system-variables.html#sysvar_group_replication_ip_whitelist)已被弃用，而[`group_replication_ip_allowlist`](https://group-replication-system-variables.html#sysvar_group_replication_ip_allowlist)可用来替代它。对于这两个系统变量，默认值均为`AUTOMATIC`。

    在群组复制启动时，如果其中一个系统变量已设置为用户定义的值而另一个没有，则使用更改后的值。如果两个系统变量都已设置为用户定义的值，则使用[`group_replication_ip_allowlist`](https://group-replication-system-variables.html#sysvar_group_replication_ip_allowlist)的值。

    如果在群组复制运行时更改[`group_replication_ip_whitelist`](https://group-replication-system-variables.html#sysvar_group_replication_ip_whitelist)或[`group_replication_ip_allowlist`](https://group-replication-system-variables.html#sysvar_group_replication_ip_allowlist)的值，这在 MySQL 8.0.24 中是可能的，那么两个变量都不会优先于另一个。

    新的系统变量与旧的系统变量的工作方式相同，只是术语发生了变化。对于[`group_replication_ip_allowlist`](https://group-replication-system-variables.html#sysvar_group_replication_ip_allowlist)给出的行为描述适用于旧系统变量和新系统变量。

+   [`group_replication_local_address`](https://group-replication-system-variables.html#sysvar_group_replication_local_address)

    | 命令行格式 | `--group-replication-local-address=value` |
    | --- | --- |
    | 系统变量 | [`group_replication_local_address`](https://group-replication-system-variables.html#sysvar_group_replication_local_address) |
    | 范围 | 全局 |
    | 动态 | 是 |
    | [`SET_VAR`](https://optimizer-hints.html#optimizer-hints-set-var "变量设置提示语法")提示适用 | 否 |
    | 类型 | 字符串 |

    此系统变量的值可以在群组复制运行时更改，但更改只在停止并重新启动群组成员的情况下生效。

    `group_replication_local_address` 设置了成员为其他成员提供连接的网络地址，格式为`host:port`。这个地址必须被组内所有成员访问，因为它被组通信引擎用于 Group Replication（XCom，一种 Paxos 变体）的 TCP 通信。如果您正在使用 MySQL 通信堆栈在成员之间建立组通信连接（`group_replication_communication_stack` = MYSQL），那么该地址必须是 MySQL Server 监听的 IP 地址和端口之一，由服务器的 `bind_address` 系统变量指定。

    警告

    不要使用此地址查询或管理成员上的数据库。这不是 SQL 客户端连接的主机和端口。

    您在 `group_replication_local_address` 中指定的地址或主机名被 Group Replication 用作复制组内组成员的唯一标识符。只要主机名或 IP 地址都不同，就可以为复制组的所有成员使用相同的端口，只要端口都不同，就可以为所有成员使用相同的主机名或 IP 地址。`group_replication_local_address` 的推荐端口是 33061。请注意，直到发出 `START GROUP_REPLICATION` 语句并且 Group Communication System (GCS) 可用之前，不会验证此变量的值。

    由 `group_replication_local_address` 配置的网络地址必须被所有组成员解析。例如，如果每个服务器实例在不同的机器上具有固定的网络地址，您可以使用机器的 IP 地址，例如 10.0.0.1。如果使用主机名，必须使用完全限定的名称，并确保通过 DNS、正确配置的 `/etc/hosts` 文件或其他名称解析过程解析。从 MySQL 8.0.14 开始，IPv6 地址（或解析为它们的主机名）也可以使用，以及 IPv4 地址。必须在方括号中指定 IPv6 地址，以区分端口号，例如：

    ```sql
    group_replication_local_address= "[2001:db8:85a3:8d3:1319:8a2e:370:7348]:33061"
    ```

    如果指定为服务器实例的组复制本地地址的主机名同时解析为 IPv4 和 IPv6 地址，则始终使用 IPv4 地址进行组复制连接。有关组复制支持 IPv6 网络以及使用 IPv4 成员和使用 IPv6 成员混合的复制组的更多信息，请参见第 20.5.5 节，“IPv6 和混合 IPv6 和 IPv4 组的支持”。

    如果您正在使用 XCom 通信堆栈在成员之间建立组通信连接（`group_replication_communication_stack = XCOM`），则在复制组中的其他服务器上必须将您为每个组成员指定的地址添加到`group_replication_ip_allowlist`（从 MySQL 8.0.22 开始）或`group_replication_ip_whitelist`（对于 MySQL 8.0.21 及更早版本）系统变量的列表中。当 XCom 通信堆栈用于组时，允许列表用于控制对组的访问。当 MySQL 通信堆栈用于组时，用户身份验证用于控制对组的访问，允许列表不起作用，如果设置则会被忽略。请注意，如果组的任何种子成员在此成员具有 IPv4 `group_replication_local_address`时列出具有 IPv6 地址，或反之亦然，则还必须设置并允许该成员的所需协议的替代地址（或解析为该协议地址的主机名）。有关更多信息，请参见第 20.6.4 节，“组复制 IP 地址权限”。

+   `group_replication_member_expel_timeout`

    | 命令行格式 | `--group-replication-member-expel-timeout=#` |
    | --- | --- |
    | 引入版本 | 8.0.13 |
    | 系统变量 | `group_replication_member_expel_timeout` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 (≥ 8.0.21) | `5` |
    | 默认值 (≤ 8.0.20) | `0` |
    | 最小值 | `0` |
    | 最大值 (≥ 8.0.14) | `3600` |
    | 最大值（≤ 8.0.13） | `31536000` |
    | 单位 | 秒 |

    此系统变量的值可以在 Group Replication 运行时更改，并立即生效。系统变量的当前值在 Group Replication 检查超时时读取。并非所有组成员都必须具有相同的设置，但建议如此以避免意外驱逐。

    `group_replication_member_expel_timeout` 指定了在创建怀疑后，Group Replication 组成员在将被怀疑失败的成员从组中驱逐之前等待的时间段（以秒为单位）。在创建怀疑之前的初始 5 秒检测期不计入此时间。在 MySQL 8.0.20 及之前的版本中，`group_replication_member_expel_timeout` 的值默认为 0，意味着没有等待时间，怀疑的成员在 5 秒检测期结束后立即有可能被驱逐。从 MySQL 8.0.21 开始，默认值为 5，意味着怀疑的成员在 5 秒检测期结束后 5 秒内有可能被驱逐。

    在组成员上更改 `group_replication_member_expel_timeout` 的值会立即对该组成员上现有及未来的怀疑生效。因此，您可以将其用作强制怀疑超时并驱逐怀疑成员以允许更改组配置的方法。有关更多信息，请参见 第 20.7.7.1 节，“驱逐超时”。

    增加`group_replication_member_expel_timeout`的值可以帮助避免在较慢或不稳定网络上发生不必要的驱逐，或在预期的瞬时网络中断或机器减速情况下。如果可疑成员在疑虑超时之前再次活跃，它会应用所有被其余组成员缓冲的消息，并进入`ONLINE`状态，无需操作者干预。您可以指定最多 3600 秒（1 小时）的超时值。重要的是要确保 XCom 的消息缓存足够大，以容纳在指定时间段内预期的消息量，再加上初始的 5 秒检测期，否则成员无法重新连接。您可以使用`group_replication_message_cache_size`系统变量调整缓存大小限制。有关更多信息，请参见第 20.7.6 节，“XCom 缓存管理”。

    如果超时时间已过，可疑成员在疑虑超时后立即有可能被驱逐。如果成员能够恢复通信并接收到一个将其驱逐的视图，并且成员已将`group_replication_autorejoin_tries`系统变量设置为指定自动重新加入尝试次数，则在超级只读模式下，它会继续进行指定数量的重新加入尝试。如果成员没有指定任何自动重新加入尝试，或者已耗尽指定数量的尝试次数，则会执行由系统变量`group_replication_exit_state_action`指定的操作。

    有关使用`group_replication_member_expel_timeout`设置的更多信息，请参见第 20.7.7.1 节，“驱逐超时”。在没有此系统变量的情况下，为避免不必要的驱逐而采取替代缓解策略，请参见第 20.3.2 节，“组复制限制”。

+   `group_replication_member_weight`

    | 命令行格式 | `--group-replication-member-weight=#` |
    | --- | --- |
    | 系统变量 | `group_replication_member_weight` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `50` |
    | 最小值 | `0` |
    | 最大值 | `100` |
    | 单位 | 百分比 |

    可以在 Group Replication 运行时更改此系统变量的值，并且更改会立即生效。在发生故障转移情况时，系统变量的当前值会被读取。

    `group_replication_member_weight` 指定了可以分配给成员的百分比权重，以影响在故障转移时成员被选为主要成员的机会，例如当现有主要成员离开单一主要组时。为成员分配数字权重以确保特定成员被选中，例如在主要成员的计划维护期间或在故障转移时确保某些硬件优先考虑。

    对于配置为以下成员的组：

    +   `成员-1`: group_replication_member_weight=30, server_uuid=aaaa

    +   `成员-2`: group_replication_member_weight=40, server_uuid=bbbb

    +   `成员-3`: group_replication_member_weight=40, server_uuid=cccc

    +   `成员-4`: group_replication_member_weight=40, server_uuid=dddd

    在选举新主要成员时，上述成员将按`成员-2`、`成员-3`、`成员-4`和`成员-1`的顺序排序。这导致在故障转移时选择`成员-2`作为新的主要成员。有关更多信息，请参见 Section 20.1.3.1, “Single-Primary Mode”。

+   `group_replication_message_cache_size`

    | 命令行格式 | `--group-replication-message-cache-size=#` |
    | --- | --- |
    | 引入版本 | 8.0.16 |
    | 系统变量 | `group_replication_message_cache_size` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `1073741824 (1 GB)` |
    | 最小值（64 位平台，≥ 8.0.21） | `134217728 (128 MB)` |
    | 最小值（64 位平台，≤ 8.0.20） | `1073741824 (1 GB)` |
    | 最小值（32 位平台，≥ 8.0.21） | `134217728 (128 MB)` |
    | 最小值（32 位平台，≤ 8.0.20） | `1073741824 (1 GB)` |
    | 最大值（64 位平台） | `18446744073709551615 (16 EiB)` |
    | 最大值（32 位平台） | `315360004294967295 (4 GB)` |
    | 单位 | 字节 |

    此系统变量应在所有组成员上具有相同的值。在 Group Replication 运行时可以更改此系统变量的值。更改在您停止并重新启动成员上的 Group Replication 后生效。在此过程中，系统变量的值允许在组成员之间有所不同，但在断开连接的情况下，成员可能无法重新连接。

    `group_replication_message_cache_size` 设置了在 Group Replication（XCom）的组通信引擎中用于消息缓存的最大内存量。XCom 消息缓存保存了作为共识协议的一部分在组成员之间交换的消息（及其元数据）。消息缓存除了其他功能外，还用于在成员在一段时间内无法与其他组成员通信后重新连接到组时恢复丢失的消息。

    `group_replication_member_expel_timeout` 系统变量确定了等待期限（最长一小时），该期限是在初始 5 秒检测期限之外允许成员返回到组中而不被驱逐的。XCom 消息缓存的大小应该根据预期的消息量设置，以便在此时间段内包含所有成员成功返回所需的所有丢失消息。直到 MySQL 8.0.20 版本，仅有 5 秒的检测期限是默认值，但从 MySQL 8.0.21 版本开始，默认值是在 5 秒检测期限之后等待 5 秒，总共为 10 秒的时间段。

    确保系统上有足够的内存供您选择的缓存大小限制使用，考虑到 MySQL 服务器的其他缓存和对象池的大小。默认设置为 1073741824 字节（1 GB）。最小设置也是 1 GB，直到 MySQL 8.0.20 版本。从 MySQL 8.0.21 开始，最小设置为 134217728 字节（128 MB），这使得可以在内存可用量受限的主机上部署，并且具有良好的网络连接性，以最小化组成员之间瞬时连接丢失的频率和持续时间。请注意，使用 `group_replication_message_cache_size` 设置的限制仅适用于缓存中存储的数据，缓存结构需要额外的 50 MB 内存。

    缓存大小限制可以在运行时动态增加或减少。如果减少缓存大小限制，XCom 将删除已经决定并传递的最旧条目，直到当前大小低于限制。当从消息缓存中删除可能需要用于恢复的消息，但当前无法访问的成员时，Group Replication 的组通信系统（GCS）会通过警告消息向您发出警告。有关调整消息缓存大小的更多信息，请参见 Section 20.7.6, “XCom Cache Management”。

+   `group_replication_paxos_single_leader`

    | 命令行格式 | `--group-replication-paxos-single-leader[={OFF&#124;ON}]` |
    | --- | --- |
    | 引入版本 | 8.0.27 |
    | 系统变量 | `group_replication_paxos_single_leader` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    注意

    此系统变量是一个群组范围的配置设置，需要完全重新启动复制组才能使更改生效。

    `group_replication_paxos_single_leader`从 MySQL 8.0.27 开始可用。当群组处于单主模式时，它使群组通信引擎能够与单一共识领导者一起运行。使用默认设置`OFF`，此行为被禁用，并且在此系统变量可用之前的版本中使用每个群组成员作为领导者的行为。当系统变量设置为`ON`时，群组通信引擎可以使用单一领导者来推动共识。在单一共识领导者模式下操作可以提高性能和韧性，特别是当群组的某些次要成员当前无法访问时。有关更多信息，请参见 Section 20.7.3, “Single Consensus Leader”。

    为了使群组通信引擎使用单一共识领导者，群组的通信协议版本必须是 MySQL 8.0.27 或更高版本。使用`group_replication_get_communication_protocol()`函数查看群组的通信协议版本。如果使用较低版本，则群组无法使用此行为。如果所有群组成员都支持，您可以使用`group_replication_set_communication_protocol()`函数将群组的通信协议设置为更高版本。有关更多信息，请参见 Section 20.5.1.4, “Setting a Group's Communication Protocol Version”。

    此系统变量是一个群组范围的配置设置。它必须在所有群组成员上具有相同的值，在 Group Replication 运行时无法更改，并且需要对群组进行完全重新启动（由具有`group_replication_bootstrap_group=ON`的服务器引导）才能使值更改生效。有关在已执行和认证事务的情况下安全引导群组的说明，请参见 Section 20.5.2, “Restarting a Group”。

    如果组为此系统变量设置了一个值，并且加入成员为该系统变量设置了不同的值，则加入成员在值匹配之前无法加入该组。如果组成员为此系统变量设置了一个值，而加入成员不支持该系统变量，则无法加入该组。

    在性能模式表`replication_group_communication_information`中的字段`WRITE_CONSENSUS_SINGLE_LEADER_CAPABLE`显示组是否支持使用单个领导者，即使在查询的成员上当前设置为`OFF`的`group_replication_paxos_single_leader`。如果组是在设置为`ON`的`group_replication_paxos_single_leader`并且其通信协议版本为 MySQL 8.0.27 或更高版本的情况下启动的，则该字段设置为 1。

+   `group_replication_poll_spin_loops`

    | 命令行格式 | `--group-replication-poll-spin-loops=#` |
    | --- | --- |
    | 系统变量 | `group_replication_poll_spin_loops` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `0` |
    | 最小值 | `0` |
    | 最大值（64 位平台） | `18446744073709551615` |
    | 最大值（32 位平台） | `4294967295` |

    此系统变量的值可以在 Group Replication 运行时更改，但更改只有在您停止并重新启动组成员上的 Group Replication 后才会生效。

    `group_replication_poll_spin_loops` 指定组通信线程在等待通信引擎互斥锁被释放之前等待更多传入网络消息的次数。

+   `group_replication_recovery_complete_at`

    | 命令行格式 | `--group-replication-recovery-complete-at=value` |
    | --- | --- |
    | 已弃用 | 8.0.34 |
    | 系统变量 | `group_replication_recovery_complete_at` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` ��示适用 | 否 |
    | 类型 | 枚举 |
    | 默认值 | `TRANSACTIONS_APPLIED` |
    | 有效值 | `TRANSACTIONS_CERTIFIED``TRANSACTIONS_APPLIED` |

    当 Group Replication 运行时，可以更改此系统变量的值，但更改只有在停止并重新启动组成员上的 Group Replication 后才会生效。

    `group_replication_recovery_complete_at` 指定了在从现有成员接收状态传输后处理缓存事务时应用的策略。您可以选择在成员接收并认证了加入组前错过的所有事务后将其标记为在线（`TRANSACTIONS_CERTIFIED`），或者只有在接收、认证和应用了这些事务后才将其标记为在线（`TRANSACTIONS_APPLIED`）。

    此变量在 MySQL 8.0.34 中已弃用（`TRANSACTIONS_CERTIFIED` 也是如此）。预计在将来的 MySQL 版本中将其移除。

+   `group_replication_recovery_compression_algorithms`

    | 命令行格式 | `--group-replication-recovery-compression-algorithms=value` |
    | --- | --- |
    | 引入版本 | 8.0.18 |
    | 系统变量 | `group_replication_recovery_compression_algorithms` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 集合 |
    | 默认值 | `uncompressed` |
    | 有效数值 | `zlib``zstd``uncompressed` |

    当 Group Replication 运行时，可以更改此系统变量的值，但更改只有在停止并重新启动组成员上的 Group Replication 后才会生效。

    `group_replication_recovery_compression_algorithms` 指定了允许用于 Group Replication 分布式恢复连接的压缩算法，用于从捐赠者的二进制日志传输状态。可用的算法与 `protocol_compression_algorithms` 系统变量相同。有关更多信息，请参见第 6.2.8 节，“连接压缩控制”。

    如果服务器已设置为支持克隆（参见第 20.5.4.2 节，“用于分布式恢复的克隆”），并且在分布式恢复期间使用远程克隆操作，则此设置不适用。对于此状态传输方法，克隆插件的 `clone_enable_compression` 设置适用。

+   `group_replication_recovery_get_public_key`

    | 命令行格式 | `--group-replication-recovery-get-public-key[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `group_replication_recovery_get_public_key` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    当 Group Replication 运行时，可以更改此系统变量的值，但更改只在您停止并重新启动组复制时才会生效。

    `group_replication_recovery_get_public_key` 指定是否从源请求用于 RSA 密钥对密码交换所需的公钥。如果 `group_replication_recovery_public_key_path` 设置为有效的公钥文件，则优先于 `group_replication_recovery_get_public_key`。如果您未在 `group_replication_recovery` 通道上使用 SSL 进行分布式恢复，并且 Group Replication 的复制用户帐户使用 `caching_sha2_password` 插件进行身份验证（这是 MySQL 8.0 中的默认设置），则此变量适用。有关更多详细信息，请参见 第 20.6.3.1.1 节，“使用 Caching SHA-2 认证插件的复制用户”。

+   `group_replication_recovery_public_key_path`

    | 命令行格式 | `--group-replication-recovery-public-key-path=file_name` |
    | --- | --- |
    | 系统变量 | `group_replication_recovery_public_key_path` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 文件名 |
    | 默认值 | `空字符串` |

    当 Group Replication 运行时，可以更改此系统变量的值，但更改只在您停止并重新启动组复制时才会生效。

    `group_replication_recovery_public_key_path` 指定包含源端所需的用于 RSA 密钥对密码交换的公钥的副本的文件的路径名。该文件必须采用 PEM 格式。如果设置了 `group_replication_recovery_public_key_path` 为有效的公钥文件，则它优先于 `group_replication_recovery_get_public_key`。此变量适用于在分布式恢复过程中未使用 SSL 进行 `group_replication_recovery` 通道（因此 `group_replication_recovery_use_ssl` 设置为 `OFF`），并且用于 Group Replication 的复制用户帐户使用 `caching_sha2_password` 插件（这是 MySQL 8.0 中的默认设置）或 `sha256_password` 插件进行身份验证。（对于 `sha256_password`，只有在使用 OpenSSL 构建 MySQL 时，设置 `group_replication_recovery_public_key_path` 才适用。）有关更多详细信息，请参阅 第 20.6.3.1.1 节，“使用 Caching SHA-2 认证插件的复制用户”。

+   `group_replication_recovery_reconnect_interval`

    | 命令行格式 | `--group-replication-recovery-reconnect-interval=#` |
    | --- | --- |
    | 系统变量 | `group_replication_recovery_reconnect_interval` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `60` |
    | 最小值 | `0` |
    | 最大值 | `31536000` |
    | 单位 | 秒 |

    此系统变量的值可以在 Group Replication 运行时更改，但更改只有在停止并重新启动组成员上的 Group Replication 后才会生效。

    `group_replication_recovery_reconnect_interval` 指定在分布式恢复过程中当组中找不到合适的提供者时重新连接尝试之间的休眠时间，单位为秒。

+   `group_replication_recovery_retry_count`

    | 命令行格式 | `--group-replication-recovery-retry-count=#` |
    | --- | --- |
    | 系统变量 | `group_replication_recovery_retry_count` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `10` |
    | 最小值 | `0` |
    | 最大值 | `31536000` |

    此系统变量的值可以在 Group Replication 运行时更改，但更改仅在您停止并重新启动组成员上的 Group Replication 后生效。

    `group_replication_recovery_retry_count` 指定加入的成员在放弃之前尝试连接可用捐赠者的次数。

+   `group_replication_recovery_ssl_ca`

    | 命令行格式 | `--group-replication-recovery-ssl-ca=value` |
    | --- | --- |
    | 系统变量 | `group_replication_recovery_ssl_ca` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 字符串 |

    此系统变量的值可以在 Group Replication 运行时更改，但更改仅在您停止并重新启动组成员上的 Group Replication 后生效。

    `group_replication_recovery_ssl_ca` 指定包含用于分布式恢复连接的受信任 SSL 证书颁发机构列表的文件路径。有关配置分布式恢复的 SSL 信息，请参阅 Section 20.6.2, “Securing Group Communication Connections with Secure Socket Layer (SSL)”")。

    如果此服务器已设置为支持克隆（参见 Section 20.5.4.2, “Cloning for Distributed Recovery”），并且您已将 `group_replication_recovery_use_ssl` 设置为 `ON`，Group Replication 将自动配置克隆 SSL 选项 `clone_ssl_ca` 的设置，以匹配您对 `group_replication_recovery_ssl_ca` 的设置。

    当 MySQL 通信堆栈用于组时（`group_replication_communication_stack = MYSQL`，此设置用于组通信连接的 TLS/SSL 配置，以及分布式恢复连接。

+   `group_replication_recovery_ssl_capath`

    | 命令行格式 | `--group-replication-recovery-ssl-capath=value` |
    | --- | --- |
    | 系统变量 | `group_replication_recovery_ssl_capath` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 字符串 |

    此系统变量的值可以在 Group Replication 运行时更改，但更改只有在您停止并重新启动组成员上的 Group Replication 后才会生效。

    `group_replication_recovery_ssl_capath` 指定包含用于分布式恢复连接的受信任 SSL 证书颁发机构证书的目录路径。有关为分布式恢复配置 SSL 的信息，请参见 第 20.6.2 节，“使用安全套接字层 (SSL) 保护组通信连接” 保护组通信连接")。

    当 MySQL 通信堆栈用于组时（`group_replication_communication_stack = MYSQL`），此设置用于组通信连接的 TLS/SSL 配置，以及分布式恢复连接。

+   `group_replication_recovery_ssl_cert`

    | 命令行格式 | `--group-replication-recovery-ssl-cert=value` |
    | --- | --- |
    | 系统变量 | `group_replication_recovery_ssl_cert` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 字符串 |

    此系统变量的值可以在 Group Replication 运行时更改，但更改只有在您停止并重新启动组成员上的 Group Replication 后才会生效。

    `group_replication_recovery_ssl_cert` 指定用于建立分布式恢复安全连接的 SSL 证书文件的名称。有关为分布式恢复配置 SSL 的信息，请参见 第 20.6.2 节，“使用安全套接字层 (SSL) 保护组通信连接” 保护组通信连接")。

    如果此服务器已设置支持克隆（参见第 20.5.4.2 节，“用于分布式恢复的克隆”），并且您已将`group_replication_recovery_use_ssl`设置为`ON`，Group Replication 会自动配置克隆 SSL 选项`clone_ssl_cert`的设置，以匹配您对`group_replication_recovery_ssl_cert`的设置。

    当 MySQL 通信堆栈用于组（`group_replication_communication_stack = MYSQL`）时，此设置用于组通信连接的 TLS/SSL 配置，以及分布式恢复连接。

+   `group_replication_recovery_ssl_cipher`

    | 命令行格式 | `--group-replication-recovery-ssl-cipher=value` |
    | --- | --- |
    | 系统变量 | `group_replication_recovery_ssl_cipher` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 字符串 |

    此系统变量的值可以在 Group Replication 运行时更改，但更改只有在您停止并重新启动组复制组件后才会生效。

    `group_replication_recovery_ssl_cipher`指定 SSL 加密的允许密码列表。有关配置分布式恢复的 SSL 信息，请参阅第 20.6.2 节，“使用安全套接字层（SSL）保护组通信连接”。

    当 MySQL 通信堆栈用于组（`group_replication_communication_stack = MYSQL`）时，此设置用于组通信连接的 TLS/SSL 配置，以及分布式恢复连接。

+   `group_replication_recovery_ssl_crl`

    | 命令行格式 | `--group-replication-recovery-ssl-crl=value` |
    | --- | --- |
    | 系统变量 | `group_replication_recovery_ssl_crl` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 文件名 |

    此系统变量的值可以在 Group Replication 运行时更改，但更改只有在您停止并重新启动组成员上的 Group Replication 后才会生效。

    `group_replication_recovery_ssl_crl` 指定包含证书吊销列表文件的目录路径。参见第 20.6.2 节，“使用安全套接字层 (SSL) 保护组通信连接” 保护组通信连接")，了解有关为分布式恢复配置 SSL 的信息。

    当 MySQL 通信堆栈用于组时（`group_replication_communication_stack = MYSQL`），此设置用于组通信连接的 TLS/SSL 配置，以及分布式恢复连接。

+   `group_replication_recovery_ssl_crlpath`

    | 命令行格式 | `--group-replication-recovery-ssl-crlpath=value` |
    | --- | --- |
    | 系统变量 | `group_replication_recovery_ssl_crlpath` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 目录名称 |

    此系统变量的值可以在 Group Replication 运行时更改，但更改只有在您停止并重新启动组成员上的 Group Replication 后才会生效。

    `group_replication_recovery_ssl_crlpath` 指定包含证书吊销列表文件的目录路径。参见第 20.6.2 节，“使用安全套接字层 (SSL) 保护组通信连接” 保护组通信连接")，了解有关为分布式恢复配置 SSL 的信息。

    当 MySQL 通信堆栈用于组时（`group_replication_communication_stack = MYSQL`），此设置用于组通信连接的 TLS/SSL 配置，以及分布式恢复连接。

+   `group_replication_recovery_ssl_key`

    | 命令行格式 | `--group-replication-recovery-ssl-key=value` |
    | --- | --- |
    | 系统变量 | `group_replication_recovery_ssl_key` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 字符串 |

    此系统变量的值可以在 Group Replication 运行时更改，但更改只有在您停止并重新启动组成员上的 Group Replication 后才会生效。

    `group_replication_recovery_ssl_key` 指定用于建立安全连接的 SSL 密钥文件的名称。有关配置分布式恢复的 SSL 信息，请参阅 第 20.6.2 节，“使用安全套接字层 (SSL) 保护组通信连接”")。

    如果此服务器已设置为支持克隆（请参阅 第 20.5.4.2 节，“用于分布式恢复的克隆”），并且您已将 `group_replication_recovery_use_ssl` 设置为 `ON`，Group Replication 会自动配置克隆 SSL 选项 `clone_ssl_key` 的设置，以匹配您对 `group_replication_recovery_ssl_key` 的设置。

    当 MySQL 通信堆栈用于组时（`group_replication_communication_stack = MYSQL`），此设置用于组通信连接的 TLS/SSL 配置，以及分布式恢复连接。

+   `group_replication_recovery_ssl_verify_server_cert`

    | 命令行格式 | `--group-replication-recovery-ssl-verify-server-cert[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `group_replication_recovery_ssl_verify_server_cert` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    此系统变量的值可以在 Group Replication 运行时更改，但更改只有在您停止并重新启动组成员上的 Group Replication 后才会生效。

    `group_replication_recovery_ssl_verify_server_cert` 指定分布式恢复连接是否应检查捐赠者发送的证书中服务器的通用名称值。有关配置分布式恢复的 SSL 信息，请参阅 第 20.6.2 节，“使用安全套接字层 (SSL) 保护组通信连接”")。

    当 MySQL 通信堆栈用于组时（`group_replication_communication_stack = MYSQL`），此设置用于组通信连接的 TLS/SSL 配置，以及分布式恢复连接。

+   `group_replication_recovery_tls_ciphersuites`

    | 命令行格式 | `--group-replication-recovery-tls-ciphersuites=value` |
    | --- | --- |
    | 引入版本 | 8.0.19 |
    | 系统变量 | `group_replication_recovery_tls_ciphersuites` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 字符串 |
    | 默认值 | `NULL` |

    此系统变量的值可以在 Group Replication 运行时更改，但更改仅在您停止并重新启动组成员上的 Group Replication 后生效。

    `group_replication_recovery_tls_ciphersuites` 指定在使用 TLSv1.3 进行连接加密时，分布式恢复连接的允许密码套件的一个或多个以冒号分隔的列表，而且此服务器实例是分布式恢复连接中的客户端，即加入成员。如果在使用 TLSv1.3 时将此系统变量设置为 `NULL`（如果未设置系统变量，则为默认值），则允许默认启用的密码套件，如 第 8.3.2 节，“加密连接 TLS 协议和密码” 中所列。如果将此系统变量设置为空字符串，则不允许任何密码套件，因此不使用 TLSv1.3。此系统变量从 MySQL 8.0.19 版本开始提供。有关配置分布式恢复的 SSL 信息，请参阅 第 20.6.2 节，“使用安全套接字层 (SSL) 保护组通信连接”")。

    当 MySQL 通信堆栈用于组（`group_replication_communication_stack = MYSQL`）时，此设置用于组通信连接的 TLS/SSL 配置，以及分布式恢复连接。

+   `group_replication_recovery_tls_version`

    | 命令行格式 | `--group-replication-recovery-tls-version=value` |
    | --- | --- |
    | 引入版本 | 8.0.19 |
    | 系统变量 | `group_replication_recovery_tls_version` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 字符串 |
    | 默认值（≥ 8.0.28） | `TLSv1.2,TLSv1.3` |
    | 默认值（≥ 8.0.19，≤ 8.0.27） | `TLSv1,TLSv1.1,TLSv1.2,TLSv1.3` |

    此系统变量的值可以在 Group Replication 运行时更改，但更改只在您停止并重新启动组成员上的 Group Replication 后生效。

    `group_replication_recovery_tls_version` 指定了一个逗号分隔的允许的一个或多个 TLS 协议列表，用于连接加密，当此服务器实例是分布式恢复连接中的客户端（即加入成员）时。每个分布式恢复连接中涉及的组成员作为客户端（加入成员）和服务器（捐赠者）协商它们都设置支持的最高协议版本。此系统变量从 MySQL 8.0.19 开始可用。

    当 MySQL 通信堆栈用于组（`group_replication_communication_stack = MYSQL`）时，此设置用于组通信连接的 TLS/SSL 配置，以及分布式恢复连接。

    如果未设置此系统变量，则默认使用“`TLSv1,TLSv1.1,TLSv1.2,TLSv1.3`”直到 MySQL 8.0.27，从 MySQL 8.0.28 开始，默认使用“`TLSv1.2,TLSv1.3`”。确保指定的协议版本是连续的，中间没有跳过版本号。

    重要

    +   从 MySQL 8.0.28 开始，MySQL Server 中移除了对 TLSv1 和 TLSv1.1 连接协议的支持。这些协议从 MySQL 8.0.26 开始被弃用，尽管 MySQL Server 客户端，包括充当客户端的 Group Replication 服务器实例，如果使用了弃用的 TLS 协议版本，不会向用户返回警告。有关更多信息，请参阅移除对 TLSv1 和 TLSv1.1 协议的支持。

    +   从 MySQL 8.0.16 开始，MySQL Server 支持 TLSv1.3 协议，前提是 MySQL Server 使用 OpenSSL 1.1.1 进行编译。服务器在启动时检查 OpenSSL 的版本，如果低于 1.1.1，则将从系统变量的默认值中移除 TLSv1.3。在这种情况下，直到 MySQL 8.0.27 为止，默认值为“`TLSv1,TLSv1.1,TLSv1.2`”，从 MySQL 8.0.28 开始为“`TLSv1.2`”。

    +   从 MySQL 8.0.18 开始，Group Replication 支持 TLSv1.3，并从 MySQL 8.0.19 开始支持密码套件选择。有关更多信息，请参见第 20.6.2 节，“使用安全套接字层（SSL）保护组通信连接”")。

    有关配置分布式恢复的 SSL 信息，请参见第 20.6.2 节，“使用安全套接字层（SSL）保护组通信连接”")��

+   `group_replication_recovery_use_ssl`

    | 命令行格式 | `--group-replication-recovery-use-ssl[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `group_replication_recovery_use_ssl` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    此系统变量的值可以在 Group Replication 运行时更改，但更改只有在您停止并重新启动组成员上的 Group Replication 后才会生效。

    `group_replication_recovery_use_ssl`指定 Group Replication 组成员之间的分布式恢复连接是否应该使用 SSL。有关配置分布式恢复的 SSL 信息，请参见第 20.6.2 节，“使用安全套接字层（SSL）保护组通信连接”")。

    如果此服务器已设置为支持克隆（请参阅第 20.5.4.2 节，“用于分布式恢复的克隆”)，并且您将此选项设置为`ON`，则 Group Replication 将使用 SSL 进行远程克隆操作以及从捐赠者的二进制日志传输状态。如果将此选项设置为`OFF`，则 Group Replication 不会使用 SSL 进行远程克隆操作。

+   `group_replication_recovery_zstd_compression_level`

    | 命令行格式 | `--group-replication-recovery-zstd-compression-level=#` |
    | --- | --- |
    | 引入版本 | 8.0.18 |
    | 系统变量 | `group_replication_recovery_zstd_compression_level` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `3` |
    | 最小值 | `1` |
    | 最大值 | `22` |

    在 Group Replication 运行时可以更改此系统变量的值，但更改只有在停止并重新启动组复制时才会生效。

    `group_replication_recovery_zstd_compression_level` 指定用于使用`zstd`压缩算法的 Group Replication 分布式恢复连接的压缩级别。允许的级别从 1 到 22，较大的值表示较高级别的压缩。默认的`zstd`压缩级别为 3。对于不使用`zstd`压缩的分布式恢复连接，此变量不起作用。

    更多信息，请参阅 第 6.2.8 节，“连接压缩控制”。

+   `group_replication_single_primary_mode`

    | 命令行格式 | `--group-replication-single-primary-mode[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `group_replication_single_primary_mode` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `ON` |

    注意

    此系统变量是一个组范围的配置设置，需要完全重新启动复制组才能使更改生效。

    `group_replication_single_primary_mode` 指示组自动选择一个服务器作为处理读/写工作负载的主服务器。这个服务器是主服务器，其他所有服务器都是从服务器。

    此系统变量是一个组范围的配置设置。它必须在所有组成员上具有相同的值，不能在组复制运行时更改，并且需要对组进行完全重新启动（由具有`group_replication_bootstrap_group=ON`的服务器引导）以使值更改生效。有关在已执行和认证事务的组中安全引导组的说明，请参见第 20.5.2 节，“重新启动组”。

    如果组为此系统变量设置了一个值，并且加入的成员为该系统变量设置了不同的值，则加入的成员在值匹配之前无法加入该组。如果组成员为此系统变量设置了一个值，而加入的成员不支持该系统变量，则无法加入该组。

    将此变量设置为`ON`会导致`group_replication_auto_increment_increment`的任何设置被忽略。

    在 MySQL 8.0.16 及更高版本中，您可以使用`group_replication_switch_to_single_primary_mode()`和`group_replication_switch_to_multi_primary_mode()`函数在组仍在运行时更改此系统变量的值。有关更多信息，请参见第 20.5.1.2 节，“更改组模式”。

+   `group_replication_ssl_mode`

    | 命令行格式 | `--group-replication-ssl-mode=value` |
    | --- | --- |
    | 系统变量 | `group_replication_ssl_mode` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 枚举 |
    | 默认值 | `DISABLED` |
    | 有效值 | `DISABLED``REQUIRED``VERIFY_CA``VERIFY_IDENTITY` |

    可以在组复制运行时更改此系统变量的值，但更改只有在停止并重新启动组复制后才会生效。

    `group_replication_ssl_mode`设置了组复制成员之间组通信连接的安全状态。可能的值如下：

    DISABLED

    建立一个未加密的连接（默认值）。

    REQUIRED

    如果服务器支持安全连接，则建立安全连接。

    VERIFY_CA

    类似于 `REQUIRED`，但还根据配置的证书颁发机构（CA）证书验证服务器 TLS 证书。

    VERIFY_IDENTITY

    类似于 `VERIFY_CA`，但还验证服务器证书是否与尝试连接的主机匹配。

    请参阅 第 20.6.2 节，“使用安全套接字层（SSL）保护组通信连接” 以获取有关为组通信配置 SSL 的信息。

+   `group_replication_start_on_boot`

    | 命令行格式 | `--group-replication-start-on-boot[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `group_replication_start_on_boot` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `ON` |

    此系统变量的值可以在 Group Replication 运行���更改，但更改仅在您停止并重新启动组成员上的 Group Replication 后才会生效。

    `group_replication_start_on_boot` 指定服务器在启动时是否应自动启动 Group Replication（`ON`）或不启动（`OFF`）。当您将此选项设置为 `ON` 时，在使用远程克隆操作进行分布式恢复后，Group Replication 将自动重新启动。

    要在服务器启动时自动启动 Group Replication，必须使用 `CHANGE MASTER TO` 语句将分布式恢复的用户凭据存储在服务器上的复制元数据存储库中。如果您希望在 `START GROUP_REPLICATION` 语句中指定用户凭据，该语句仅将用户凭据存储在内存中，请确保 `group_replication_start_on_boot` 设置为 `OFF`。

+   `group_replication_tls_source`

    | 命令行格式 | `--group-replication-tls-source=value` |
    | --- | --- |
    | 引入版本 | 8.0.21 |
    | 系统变量 | `group_replication_tls_source` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 枚举 |
    | 默认值 | `mysql_main` |
    | 有效值 | `mysql_main``mysql_admin` |

    在 Group Replication 运行时可以更改此系统变量的值，但更改只有在您停止并重新启动组成员上的 Group Replication 后才会生效。

    `group_replication_tls_source` 指定了 Group Replication 的 TLS 材料来源。

+   `group_replication_transaction_size_limit`

    | 命令行格式 | `--group-replication-transaction-size-limit=#` |
    | --- | --- |
    | 系统变量 | `group_replication_transaction_size_limit` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `150000000` |
    | 最小值 | `0` |
    | 最大值 | `2147483647` |
    | 单位 | 字节 |

    此系统变量在所有组成员上应具有相同的值。在 Group Replication 运行时可以更改此系统变量的值。更改立即在组成员上生效，并且从该成员上启动的下一个事务开始应用。在此过程中，系统变量的值允许在组成员之间有所不同，但某些事务可能会被拒绝。

    `group_replication_transaction_size_limit` 配置了复制组接受的最大事务大小（以字节为单位）。大于此大小的事务将被接收成员回滚，并且不会广播到组中。大事务可能会导致复制组在内存分配方面出现问题，这可能导致系统变慢，或者在网络带宽消耗方面出现问题，这可能导致成员被怀疑已经失败，因为它正忙于处理大事务。

    当此系统变量设置为 0 时，组接受的事务大小没有限制。从 MySQL 8.0 开始，此系统变量的默认设置为 150000000 字节（约为 143 MB）。根据组需要容忍的最大消息大小调整此系统变量的值，要记住，处理事务所需的时间与其大小成正比。`group_replication_transaction_size_limit` 的值应在所有组成员上相同。有关大事务的进一步缓解策略，请参见 第 20.3.2 节，“Group Replication 限制”。

+   `group_replication_unreachable_majority_timeout`

    | 命令行格式 | `--group-replication-unreachable-majority-timeout=#` |
    | --- | --- |
    | 系统变量 | `group_replication_unreachable_majority_timeout` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `0` |
    | 最小值 | `0` |
    | 最大值 | `31536000` |
    | 单元 | 秒 |

    此系统变量的值可以在 Group Replication 运行时更改，并立即生效。当发生需要该行为的问题时，会读取系统变量的当前值。

    `group_replication_unreachable_majority_timeout` 指定了成员在遭受网络分区且无法连接到多数派时等待离开组的秒数。在一个由 5 台服务器（S1，S2，S3，S4，S5）组成的组中，如果在(S1，S2)和(S3，S4，S5)之间存在断开连接，则存在网络分区。第一组(S1，S2)现在处于少数派，因为它无法联系到超过一半的组。而多数派组(S3，S4，S5)仍在运行，少数派组等待指定的时间进行网络重新连接。有关此场景的详细描述，请参见第 20.7.8 节，“处理网络分区和丢失法定人数”。

    默认情况下，`group_replication_unreachable_majority_timeout` 设置为 0，这意味着由于网络分区而发现自己处于少数派的成员将永远等待离开组。如果设置了超时时间，当指定时间到达时，少数派处理的所有待处理事务都将被回滚，并且少数派分区中的服务器将移至`ERROR`状态。如果成员的 `group_replication_autorejoin_tries` 系统变量设置了指定的自动重新加入尝试次数，它将在超级只读模式下进行指定次数的重新加入尝试。如果成员没有指定任何自动重新加入尝试，或者已经耗尽了指定次数的尝试次数，则会按照系统变量 `group_replication_exit_state_action` 指定的操作进行。

    警告

    当你有一个对称的组，例如只有两个成员（S0，S2），如果存在网络分区且没有多数派，在配置的超时时间后，所有成员都会进入`ERROR`状态。

    欲了解更多关于此选项的信息，请参阅 第 20.7.7.2 节，“无法达到多数超时”。

+   `group_replication_view_change_uuid`

    | 命令行格式 | `--group-replication-view-change-uuid=value` |
    | --- | --- |
    | 引入版本 | 8.0.26 |
    | 系统变量 | `group_replication_view_change_uuid` |
    | 作用域 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 字符串 |
    | 默认值 | `AUTOMATIC` |

    注意

    这个系统变量是一个整个组的配置设置，需要对复制组进行完全重启才能使更改生效。

    `group_replication_view_change_uuid` 指定一个替代 UUID，用作组生成的视图更改事件中 GTID 标识符的 UUID 部分。替代 UUID 使这些内部生成的事务易于与从客户端接收的事务区分开。如果您的设置允许在组之间进行故障切换，并且您需要识别和丢弃特定于备份组的事务，则这可能很有用。此系统变量的默认值为 `AUTOMATIC`，意味着视图更改事件的 GTID 使用由 `group_replication_group_name` 系统变量指定的组名，就像来自客户端的事务一样。在没有此系统变量的版本中，组成员被视为具有值 `AUTOMATIC`。

    替代 UUID 必须与由 `group_replication_group_name` 系统变量指定的组名不同，并且必须与任何组成员的服务器 UUID 不同。它还必须与应用于拓扑中任何复制通道上的匿名事务的 GTID 中使用的任何 UUID 不同，使用 `CHANGE REPLICATION SOURCE TO` 语句的 `ASSIGN_GTIDS_TO_ANONYMOUS_TRANSACTIONS` 选项。

    这个系统变量是一个群组范围的配置设置。它必须在所有群组成员上具有相同的数值，不能在 Group Replication 运行时更改，并且需要对群组进行完全重启（由具有 `group_replication_bootstrap_group=ON` 的服务器引导）以使数值更改生效。有关在已执行和认证事务的群组中安全引导群组的说明，请参见 Section 20.5.2, “Restarting a Group”。

    如果群组对这个系统变量有一个数值设置，而加入的成员对这个系统变量有一个不同的数值设置，那么加入的成员在数值匹配之前无法加入群组。如果群组成员对这个系统变量有一个数值设置，而加入的成员不支持这个系统变量，那么它无法加入群组。
