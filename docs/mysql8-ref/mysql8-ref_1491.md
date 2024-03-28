> 原文：[`dev.mysql.com/doc/refman/8.0/en/group-replication-configuring-instances.html`](https://dev.mysql.com/doc/refman/8.0/en/group-replication-configuring-instances.html)

#### 20.2.1.2 为 Group Replication 配置实例

本节解释了要为用于 Group Replication 的 MySQL Server 实例配置的配置设置。有关背景信息，请参阅 Section 20.3, “Requirements and Limitations”。

+   存储引擎

+   复制框架

+   Group Replication 设置

##### 存储引擎

对于 Group Replication，数据必须存储在 InnoDB 事务性存储引擎中（详情请参阅 Section 20.3.1, “Group Replication Requirements”）。使用其他存储引擎，包括临时的 `MEMORY` 存储引擎，可能会导致 Group Replication 中的错误。请设置 `disabled_storage_engines` 系统变量如下以防止其使用：

```sql
disabled_storage_engines="MyISAM,BLACKHOLE,FEDERATED,ARCHIVE,MEMORY"
```

请注意，当禁用 `MyISAM` 存储引擎时，当您将 MySQL 实例升级到仍在使用 **mysql_upgrade** 的版本（在 MySQL 8.0.16 之前），**mysql_upgrade** 可能会失败并显示错误。为了处理这个问题，您可以在运行 **mysql_upgrade** 时重新启用该存储引擎，然后在重新启动服务器时再次禁用它。更多信息，请参阅 Section 6.4.5, “mysql_upgrade — Check and Upgrade MySQL Tables”。

##### 复制框架

以下设置根据 MySQL Group Replication 的要求配置复制。

```sql
server_id=1
gtid_mode=ON
enforce_gtid_consistency=ON
```

这些设置配置服务器使用唯一标识号 1，启用 Section 19.1.3, “Replication with Global Transaction Identifiers”，并且只允许执行可以安全记录使用 GTID 的语句。

在 MySQL 8.0.20 及更早版本中，还需要以下设置：

```sql
binlog_checksum=NONE
```

此设置禁用写入二进制日志的事件的校验和，默认情况下为启用状态。从 MySQL 8.0.21 开始，组复制支持二进制日志中的校验和，并可以使用它们来验证某些通道上事件的完整性，因此您可以使用默认设置。有关更多详细信息，请参见第 20.3.2 节，“组复制限制”。

如果您使用的是早于 8.0.3 的 MySQL 版本，在该版本中改进了复制的默认值，您还需要将这些行添加到成员的选项文件中。如果在后续版本的选项文件中有这些系统变量之一，请确保它们设置如下所示。有关更多详细信息，请参见第 20.3.1 节，“组复制要求”。

```sql
log_bin=binlog
log_slave_updates=ON
binlog_format=ROW
master_info_repository=TABLE
relay_log_info_repository=TABLE
transaction_write_set_extraction=XXHASH64
```

##### 组复制设置

此时，选项文件确保服务器已配置并被指示在给定配置下实例化复制基础设施。以下部分为服务器配置组复制设置。

```sql
plugin_load_add='group_replication.so'
group_replication_group_name="aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa"
group_replication_start_on_boot=off
group_replication_local_address= "s1:33061"
group_replication_group_seeds= "s1:33061,s2:33061,s3:33061"
group_replication_bootstrap_group=off
```

+   `plugin-load-add`将组复制插件添加到服务器在启动时加载的插件列表中。在生产部署中，这比手动安装插件更可取。

+   配置`group_replication_group_name`告诉插件，它正在加入或创建的组名为"aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa"。

    `group_replication_group_name`的值必须是有效的 UUID。您可以使用`SELECT UUID()`来生成一个。此 UUID 是 GTID 的一部分，当来自客户端的事务被组成员接收，并且由组成员内部生成的视图更改事件被写入二进制日志时使用。

+   将`group_replication_start_on_boot`变量配置为`off`会指示插件在服务器启动时不自动启动操作。在设置组复制时很重要，因为这样可以确保您在手动启动插件之前可以配置服务器。一旦成员配置完成，您可以将`group_replication_start_on_boot`设置为`on`，这样组复制将在服务器启动时自动启动。

+   配置`group_replication_local_address`设置成员用于与组内其他成员进行内部通信的网络地址和端口。组复制使用此地址进行涉及组通信引擎（XCom，一种 Paxos 变体）的远程实例的内部成员之间连接。

    重要

    组复制本地地址必须与用于 SQL 客户端连接的主机名和端口不同，这些由 MySQL 服务器的`hostname`和`port`系统变量定义。它不能用于客户端应用程序。它只能用于运行组复制时组成员之间的内部通信。

    `group_replication_local_address`配置的网络地址必须由所有组成员解析。例如，如果每个服务器实例位于具有固定网络地址的不同机器上，你可以使用机器的 IP 地址，例如 10.0.0.1。如果使用主机名，必须使用完全限定名称，并确保通过 DNS、正确配置的`/etc/hosts`文件或其他名称解析过程解析。从 MySQL 8.0.14 开始，IPv6 地址（或解析为其的主机名）也可以使用，以及 IPv4 地址。一个组可以包含使用 IPv6 和使用 IPv4 的成员的混合。有关组复制对 IPv6 网络的支持以及混合 IPv4 和 IPv6 组的更多信息，请参阅第 20.5.5 节，“IPv6 和混合 IPv6 和 IPv4 组的支持”。

    推荐的`group_replication_local_address`端口是 33061。`group_replication_local_address`被组复制用作复制组内组成员的唯一标识符。只要主机名或 IP 地址都不同，你可以为所有复制组成员使用相同的端口，就像在本教程中演示的那样。或者，只要端口都不同，你可以为所有成员使用相同的主机名或 IP 地址，例如在第 20.2.2 节，“本地部署组复制”中所示。

    现有成员为组复制的分布式恢复过程提供给加入成员的连接，并非由`group_replication_local_address`配置的网络地址。在 MySQL 8.0.20 之前，组成员为分布式恢复提供其标准 SQL 客户端连接给加入成员，由 MySQL 服务器的`hostname`和`port`系统变量指定。从 MySQL 8.0.21 开始，组成员可以为加入成员提供一组专用客户端连接作为分布式恢复的备用端点。更多详情，请参阅第 20.5.4.1 节，“分布式恢复的连接”。

    重要

    如果加入成员无法正确使用由 MySQL 服务器的`hostname`系统变量定义的主机名来识别其他成员，分布式恢复可能会失败。建议运行 MySQL 的操作系统具有经过正确配置的唯一主机名，可以使用 DNS 或本地设置。服务器用于 SQL 客户端连接的主机名可以在性能模式表`replication_group_members`的`Member_host`列中验证。如果多个组成员外部化了操作系统设置的默认主机名，加入成员可能无法将其解析为正确的成员地址，无法连接进行分布式恢复。在这种情况下，您可以使用 MySQL 服务器的`report_host`系统变量为每个服务器配置一个唯一的主机名来外部化。

+   配置`group_replication_group_seeds`设置了组成员的主机名和端口，新成员使用这些信息来建立与组的连接。这些成员被称为种子成员。一旦连接建立，组成员信息将列在性能模式表`replication_group_members`中。通常`group_replication_group_seeds`列表包含每个组成员的`group_replication_local_address`的`hostname:port`，但这并非强制，可以选择一部分组成员作为种子。

    重要

    在`group_replication_group_seeds`中列出的`hostname:port`是种子成员的内部网络地址，由`group_replication_local_address`配置，而不是用于 SQL 客户端连接的`hostname:port`，例如在性能模式表`replication_group_members`中显示。

    启动组的服务器不使用此选项，因为它是初始服务器，负责引导组。换句话说，启动组的服务器上存在的任何数据都将用作下一个加入成员的数据。第二个加入的服务器请求组内唯一的成员加入，第二个服务器上缺失的数据将从引导成员的捐赠数据中复制，然后组扩展。第三个加入的服务器可以请求其中任何两个加入，数据将同步到新成员，然后组再次扩展。后续服务器加入时重复此过程。

    警告

    当同时加入多个服务器时，请确保它们指向已经在组内的种子成员。不要使用正在加入组的成员作为种子，因为当联系时它们可能尚未加入组。

    最好的做法是首先启动引导成员，并让它创建组。然后将其设置为其余加入成员的种子成员。这样确保在加入其余成员时已经形成了一个组。

    不支持同时创建组并加入多个成员。这样做可能会成功，但有可能操作竞争，然后加入组的操作最终会出现错误或超时。

    加入成员必须使用种子成员在`group_replication_group_seeds`选项中广告的相同协议（IPv4 或 IPv6）与种子成员通信。为了 Group Replication 的 IP 地址权限，种子成员的允许列表必须包含加入成员的 IP 地址，以供种子成员提供的协议使用，或者解析为该协议的地址的主机名。如果加入成员的协议与种子成员广告的协议不匹配，则必须设置并允许此地址或主机名，以及加入成员的`group_replication_local_address`。如果加入成员没有适当协议的允许地址，则其连接尝试将被拒绝。有关更多信息，请参见 Section 20.6.4, “Group Replication IP Address Permissions”。

+   配置`group_replication_bootstrap_group`指示插件是否引导组。在这种情况下，即使`s1`是组的第一个成员，我们在选项文件中将此变量设置为关闭。相反，我们在实例运行时配置`group_replication_bootstrap_group`，以确保只有一个成员实际引导组。

    重要

    `group_replication_bootstrap_group`变量必须在任何时候仅在属于组的一个服务器实例上启用，通常是您引导组的第一次（或者整个组被关闭并重新启动的情况下）。如果多次引导组，例如当多个服务器实例设置了此选项时，则可能会创建人为的脑裂情况，即存在具有相同名称的两个不同组。始终在第一个服务器实例上线后设置`group_replication_bootstrap_group=off`。

本教程中描述的系统变量是启动新成员所需的配置设置，但还有其他系统变量可用于配置组成员。这些列在 Section 20.9, “Group Replication Variables”中。

重要

一些系统变量，有些是特定于组复制，有些不是，都是组范围的配置设置，必须在所有组成员上具有相同的值。如果组成员为其中一个这些系统变量设置了值，并且加入的成员为其设置了不同的值，则加入的成员无法加入组，并返回错误消息。如果组成员为此系统变量设置了值，而加入的成员不支持该系统变量，则无法加入组。所有这些系统变量都在第 20.9 节，“组复制变量”中标识。
