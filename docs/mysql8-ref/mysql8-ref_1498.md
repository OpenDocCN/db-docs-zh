# 20.3.1 Group Replication Requirements

> 原文：[`dev.mysql.com/doc/refman/8.0/en/group-replication-requirements.html`](https://dev.mysql.com/doc/refman/8.0/en/group-replication-requirements.html)

+   基础设施

+   服务器实例配置

您希望用于 Group Replication 的服务器实例必须满足以下要求。

#### 基础设施

+   **InnoDB 存储引擎。** 数据必须存储在`InnoDB`事务性存储引擎中。事务乐观执行，然后在提交时检查冲突。如果存在冲突，为了在组内保持一致性，一些事务将被回滚。这意味着需要一个事务性存储引擎。此外，`InnoDB`提供了一些额外功能，使其在与 Group Replication 一起操作时能更好地管理和处理冲突。使用其他存储引擎，包括临时`MEMORY`存储引擎，可能会导致 Group Replication 中的错误。在将实例与 Group Replication 一起使用之前，请将任何其他存储引擎中的表转换为使用`InnoDB`。您可以通过在组成员上设置`disabled_storage_engines`系统变量来阻止使用其他存储引擎，例如：

    ```sql
    disabled_storage_engines="MyISAM,BLACKHOLE,FEDERATED,ARCHIVE,MEMORY"
    ```

+   **主键。** 每个要被组复制的表必须有一个定义好的主键，或者等效的主键，其中等效主键是一个非空唯一键。这些键被要求作为表中每一行的唯一标识符，使系统能够通过确定每个事务修改了哪些行来确定哪些事务发生了冲突。组复制有自己内置的主键或主键等效的检查，不使用`sql_require_primary_key`系统变量执行的检查。您可以在运行组复制的服务器实例上设置`sql_require_primary_key=ON`，并且您可以为组复制通道设置`CHANGE REPLICATION SOURCE TO` | `CHANGE MASTER TO`语句的`REQUIRE_TABLE_PRIMARY_KEY_CHECK`选项为`ON`。但是，请注意，您可能会发现一些在组复制内置检查下允许的事务，在设置`sql_require_primary_key=ON`或`REQUIRE_TABLE_PRIMARY_KEY_CHECK=ON`时不被允许。

+   **网络性能。** MySQL 组复制被设计用于部署在服务器实例非常接近的集群环境中。组的性能和稳定性可能会受到网络延迟和网络带宽的影响。所有组成员之间必须始终保持双向通信。如果某个服务器实例的入站或出站通信被阻止（例如，由防火墙或连接问题），该成员无法在组中运行，并且组成员（包括存在问题的成员）可能无法报告受影响服务器实例的正确成员状态。

    从 MySQL 8.0.14 开始，您可以在远程组复制服务器之间使用 IPv4 或 IPv6 网络基础设施，或两者混合，进行 TCP 通信。组复制也可以在虚拟专用网络（VPN）上运行，没有任何限制。

    同样从 MySQL 8.0.14 开始，当组复制服务器实例位于同一位置并共享本地组通信引擎（XCom）实例时，会尽可能使用专用输入通道进行通信，而不是 TCP 套接字，以降低开销。对于某些需要远程 XCom 实例之间通信的组复制任务，比如加入一个组，仍然使用 TCP 网络，因此网络性能会影响组的性能。

#### 服务器实例配置

必须按照以下所示配置的选项在组成员的服务器实例上进行配置。

+   **唯一服务器标识符。** 使用`server_id`系统变量为服务器配置一个唯一的服务器 ID，这对于复制拓扑中的所有服务器都是必需的。服务器 ID 必须是介于 1 和(2³²)−1 之间的正整数，并且必须与复制拓扑中任何其他服务器使用的任何其他服务器 ID 不同。

+   **二进制日志激活。** 设置[`--log-bin[=log_file_name]`](replication-options-binary-log.html#sysvar_log_bin)。从 MySQL 8.0 开始，默认情况下启用了二进制日志记录，除非您想更改二进制日志文件的名称，否则不需要指定此选项。组复制会复制二进制日志的内容，因此二进制日志需要处于活动状态才能运行。请参阅第 7.4.4 节，“二进制日志”。

+   **复制更新记录。** 设置`log_replica_updates=ON`（从 MySQL 8.0.26 开始）或`log_slave_updates=ON`（在 MySQL 8.0.26 之前）。从 MySQL 8.0 开始，此设置是默认的，因此您不需要指定它。组成员需要记录从其提供者在加入时接收并通过复制应用程序应用的事务，并记录他们接收并应用于组中的所有事务。这使得组复制能够通过从现有组成员的二进制日志进行状态传输来进行分布式恢复。

+   **二进制日志行格式。** 设置`binlog_format=row`。这是默认设置，因此您不需要指定它。组复制依赖于基于行的复制格式来在组中的服务器之间一致地传播更改，并提取必要的信息以检测在组中不同服务器上同时执行的事务之间的冲突。从 MySQL 8.0.19 开始，`REQUIRE_ROW_FORMAT`设置会自动添加到组复制的通道中，以强制在应用事务时使用基于行的复制。请参阅第 19.2.1 节，“复制格式”和第 19.3.3 节，“复制权限检查”。

+   **关闭二进制日志校验和（至 MySQL 8.0.20）。** 在 MySQL 8.0.20 及之前的版本中，设置`binlog_checksum=NONE`。在这些版本中，组复制无法使用校验和，并且不支持其存在于二进制日志中。从 MySQL 8.0.21 开始，组复制支持校验和，因此组成员可以使用默认设置`binlog_checksum=CRC32`，您不需要指定它。

+   **全局事务标识符开启。** 将`gtid_mode=ON`和`enforce_gtid_consistency=ON`设置为`ON`。这些设置不是默认值。基于 GTID 的复制对于 Group Replication 是必需的，它使用全局事务标识符来跟踪已在组中的每个服务器实例上提交的事务。参见第 19.1.3 节，“使用全局事务标识符进行复制”。

+   **复制信息存储库。** 将`master_info_repository=TABLE`和`relay_log_info_repository=TABLE`设置为`TABLE`。在 MySQL 8.0 中，这些设置是默认的，`FILE`设置已被弃用。从 MySQL 8.0.23 开始，这些系统变量的使用已被弃用，因此省略这些系统变量，只允许默认设置。复制应用程序需要将复制元数据写入`mysql.slave_master_info`和`mysql.slave_relay_log_info`系统表，以确保 Group Replication 插件具有一致的可恢复性和事务管理的复制元数据。参见第 19.2.4.2 节，“复制元数据存储库”。

+   **事务写集提取。** 将`transaction_write_set_extraction=XXHASH64`设置为`XXHASH64`，以便在收集要记录到二进制日志的行时，服务器也收集写入集。在 MySQL 8.0 中，这是默认设置，从 MySQL 8.0.26 开始，该系统变量的使用已被弃用。写入集基于每行的主键，并且是一个简化且紧凑的视图，用于唯一标识已更改的行。Group Replication 在所有组成员上使用此信息进行冲突检测和认证。

+   **默认表加密。** 将`default_table_encryption`设置为所有组成员上的相同值。默认模式和表空间加密可以启用（`ON`）或禁用（`OFF`，默认值），只要所有成员的设置相同即可。

+   **表名小写。** 将`lower_case_table_names`设置为所有组成员上的相同值。对于使用`InnoDB`存储引擎的情况，设置为 1 是正确的，这是 Group Replication 所需的。请注意，这个设置在所有平台上都不是默认值。

+   **二进制日志依赖跟踪。** 将`binlog_transaction_dependency_tracking`设置为`WRITESET`可以根据组的工作负载提高组成员的性能。虽然在应用中继日志中的事务时，组复制在认证后进行自己的并行化，独立于为`binlog_transaction_dependency_tracking`设置的任何值，但是这个值确实影响了事务在组复制成员的二进制日志中的写入方式。这些日志中的依赖信息用于协助从捐赠者的二进制日志进行分布式恢复的状态传输过程，每当成员加入或重新加入组时都会发生。

    注意

    当`replica_preserve_commit_order`设置为`ON`时，将`binlog_transaction_dependency_tracking`设置为`WRITESET`与将其设置为`WRITESET_SESSION`具有相同的效果。

+   **多线程应用程序。** 组复制成员可以配置为多线程副本，使事务可以并行应用。从 MySQL 8.0.27 开始，默认情况下所有副本都配置为多线程。系统变量`replica_parallel_workers`（从 MySQL 8.0.26 开始）或`slave_parallel_workers`（MySQL 8.0.26 之前）的非零值启用了成员上的多线程应用程序。从 MySQL 8.0.27 开始，默认值为 4 个并行应用程序线程，最多可以指定 1024 个并行应用程序线程。对于多线程副本，还需要以下设置，这些设置是从 MySQL 8.0.27 开始的默认设置：

    `replica_preserve_commit_order=ON`（从 MySQL 8.0.26 开始）或`slave_preserve_commit_order=ON`（MySQL 8.0.26 之前）

    此设置是为了确保并行事务的最终提交与原始事务的顺序相同。组复制依赖于围绕所有参与成员以相同顺序接收和应用已提交事务的一致性机制构建。

    `replica_parallel_type=LOGICAL_CLOCK`（从 MySQL 8.0.26 开始）或`slave_parallel_type=LOGICAL_CLOCK`（MySQL 8.0.26 之前）

    此设置需要与`replica_preserve_commit_order=ON`或`slave_preserve_commit_order=ON`一起使用。它指定了用于决定在副本上允许哪些事务并行执行的策略。

    设置`replica_parallel_workers=0`或`slave_parallel_workers=0`会禁用并行执行，使得复制品只有一个应用程序线程和没有协调器线程。在这种设置下，`replica_parallel_type`或`slave_parallel_type`以及`replica_preserve_commit_order`或`slave_preserve_commit_order`选项不起作用，会被忽略。从 MySQL 8.0.27 开始，如果在复制品上使用 GTIDs 时禁用了并行执行，复制品实际上会使用一个并行工作者，以利用在不访问文件位置的情况下重试事务的方法。然而，这种行为对用户不会有任何改变。

+   **分离的 XA 事务。** MySQL 8.0.29 及更高版本支持分离的 XA 事务。分离的事务是指一旦准备好，就不再与当前会话连接的事务。这会自动发生作为执行`XA PREPARE`的一部分。准备好的 XA 事务可以由另一个连接提交或回滚，然后当前会话可以启动另一个 XA 事务或本地事务，而无需等待刚刚准备的事务完成。

    当启用分离的 XA 事务支持（`xa_detach_on_prepare = ON`）时，任何连接到此服务器的连接都可以列出（使用`XA RECOVER`）、回滚或提交任何准备好的 XA 事务。此外，在分离的 XA 事务中不能使用临时表。

    通过将`xa_detach_on_prepare`设置为`OFF`可以禁用对分离的 XA 事务的支持，但这并不推荐。特别是，如果此服务器正在设置为 MySQL 组复制中的一个实例，您应该将此变量保持为其默认值（`ON`）。

    更多信息请参见 Section 15.3.8.2, “XA Transaction States”。
