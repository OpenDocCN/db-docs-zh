# 20.6.4 组复制 IP 地址权限

> 原文：[`dev.mysql.com/doc/refman/8.0/en/group-replication-ip-address-permissions.html`](https://dev.mysql.com/doc/refman/8.0/en/group-replication-ip-address-permissions.html)

仅当使用 XCom 通信堆栈建立组通信时（`group_replication_communication_stack=XCOM`），组复制插件允许您指定一个主机允许列表，从中可以接受传入的组通信系统连接。如果在服务器 s1 上指定了一个允许列表，那么当服务器 s2 正在与 s1 建立连接以进行组通信时，s1 在接受来自 s2 的连接之前首先检查允许列表。如果 s2 在允许列表中，则 s1 接受连接，否则 s1 拒绝 s2 的连接尝试。从 MySQL 8.0.22 开始，系统变量`group_replication_ip_allowlist`用于指定允许列表，而对于 MySQL 8.0.22 之前的版本，使用系统变量`group_replication_ip_whitelist`。新的系统变量的工作方式与旧的系统变量相同，只是术语已更改。

注意

当使用 MySQL 通信堆栈建立组通信时（`group_replication_communication_stack=MYSQL`），`group_replication_ip_allowlist`和`group_replication_ip_whitelist`的设置将被忽略。请参阅第 20.6.1 节，“连接安全管理的通信堆栈”。

如果您没有明确指定允许列表，则组通信引擎（XCom）会自动扫描主机上的活动接口，并识别具有私有子网地址的接口，以及为每个接口配置的子网掩码。这些地址以及 IPv4 的`localhost` IP 地址和（从 MySQL 8.0.14 开始）IPv6 用于创建自动组复制允许列表。因此，自动允许列表包括在适当的子网掩码应用后在主机中找到的以下范围的任何 IP 地址：

```sql
IPv4 (as defined in RFC 1918)
10/8 prefix       (10.0.0.0 - 10.255.255.255) - Class A
172.16/12 prefix  (172.16.0.0 - 172.31.255.255) - Class B
192.168/16 prefix (192.168.0.0 - 192.168.255.255) - Class C

IPv6 (as defined in RFC 4193 and RFC 5156)
fc00:/7 prefix    - unique-local addresses
fe80::/10 prefix  - link-local unicast addresses

127.0.0.1 - localhost for IPv4
::1       - localhost for IPv6
```

在错误日志中添加了一个条目，指出已自动允许主机的地址。

自动允许私有地址的白名单不能用于来自私有网络之外的服务器的连接，因此，即使服务器具有公共 IP 接口，也不会默认允许外部主机从外部连接到 Group Replication。对于位于不同机器上的服务器实例之间的 Group Replication 连接，您必须提供公共 IP 地址并将其指定为显式白名单。如果您为白名单指定任何条目，则私有和`localhost`地址不会自动添加，因此，如果您使用其中任何一个，必须明确指定。

要手动指定白名单，请使用`group_replication_ip_allowlist`（MySQL 8.0.22 及更高版本）或`group_replication_ip_whitelist`系统变量。在 MySQL 8.0.24 之前，您不能在服务器作为复制组的活动成员时更改白名单。如果成员处于活动状态，则必须在更改白名单之前执行`STOP GROUP_REPLICATION`，然后执行更改白名单，并在之后执行`START GROUP_REPLICATION`。从 MySQL 8.0.24 开始，您可以在 Group Replication 运行时更改白名单。

白名单必须包含在每个成员的`group_replication_local_address`系统变量中指定的 IP 地址或主机名。此地址与 MySQL 服务器 SQL 协议主机和端口不同，并且不在服务器实例的`bind_address`系统变量中指定。如果用作服务器实例的 Group Replication 本地地址的主机名解析为 IPv4 和 IPv6 地址，则 IPv4 地址优先用于 Group Replication 连接。

指定为分布式恢复端点的 IP 地址以及如果用于分布式恢复（这是默认设置）的成员标准 SQL 客户端连接的 IP 地址不需要添加到白名单中。白名单仅用于每个成员的`group_replication_local_address`指定的地址。加入成员必须通过白名单允许其对组的初始连接，以便检索用于分布式恢复的地址或地址。

在白名单中，您可以指定以下任意组合：

+   IPv4 地址（例如，`198.51.100.44`）

+   具有 CIDR 表示法的 IPv4 地址（例如，`192.0.2.21/24`）

+   IPv6 地址，从 MySQL 8.0.14 开始（例如，`2001:db8:85a3:8d3:1319:8a2e:370:7348`）

+   具有 CIDR 表示法的 IPv6 地址，从 MySQL 8.0.14 开始（例如，`2001:db8:85a3:8d3::/64`）

+   主机名（例如，`example.org`）

+   使用 CIDR 表示法的主机名（例如，`www.example.com/24`）

在 MySQL 8.0.14 之前，主机名只能解析为 IPv4 地址。从 MySQL 8.0.14 开始，主机名可以解析为 IPv4 地址、IPv6 地址或两者都可以。如果一个主机名同时解析为 IPv4 和 IPv6 地址，则始终使用 IPv4 地址进行组复制连接。您可以结合主机名或 IP 地址使用 CIDR 表示法来允许具有特定网络前缀的 IP 地址块，但请确保指定子网中的所有 IP 地址都在您的控制范围内。

注意

当因为 IP 地址不在允许列表中而拒绝连接尝试时，拒绝消息总是以 IPv6 格式打印 IP 地址。在此格式中，IPv4 地址以`::ffff:`开头（一个 IPv4 映射的 IPv6 地址）。您不需要使用此格式来指定允许列表中的 IPv4 地址；对于 IPv4 地址，请使用标准 IPv4 格式。

在允许列表中，每个条目之间必须用逗号分隔。例如：

```sql
mysql> SET GLOBAL group_replication_ip_allowlist="192.0.2.21/24,198.51.100.44,203.0.113.0/24,2001:db8:85a3:8d3:1319:8a2e:370:7348,example.org,www.example.com/24";
```

要加入复制组，服务器需要在其请求加入组的种子成员上获得许可。通常，这将是复制组的引导成员，但可以是任何在服务器配置中由`group_replication_group_seeds`选项列出的服务器之一。如果组的任何种子成员在加入成员具有 IPv4 `group_replication_local_address`时使用 IPv6 地址列出，或反之亦然，则还必须为加入成员为种子成员提供的协议设置和允许备用地址（或解析为该协议地址的主机名）。这是因为当服务器加入复制组时，必须使用种子成员在`group_replication_group_seeds`选项中广告的协议进行初始联系，无论是 IPv4 还是 IPv6。如果加入成员没有适当协议的允许地址，其连接尝试将被拒绝。有关管理混合 IPv4 和 IPv6 复制组的更多信息，请参见第 20.5.5 节，“IPv6 和混合 IPv6 和 IPv4 组的支持”。

当重新配置复制组（例如，选举新的主服务器或成员加入或离开时），组成员之间重新建立连接。如果一个组成员只被不再是复制组一部分的服务器允许访问，那么在重新配置后，它将无法重新连接到不允许它的剩余服务器。为了完全避免这种情况，请为所有作为复制组成员的服务器指定相同的白名单。

注意

根据您的安全要求，可以为不同的组成员配置不同的白名单，例如，为了保持不同的子网分开。如果您需要配置不同的白名单以满足安全要求，请确保在复制组中的白名单之间有足够的重叠，以最大限度地增加服务器在没有原始种子成员的情况下重新连接的可能性。

对于主机名，名称解析仅在另一个服务器发出连接请求时进行。无法解析的主机名不会被考虑用于白名单验证，并且会将警告消息写入错误日志。对已解析的主机名执行前向确认反向 DNS（FCrDNS）验证。

警告

主机名在白名单中比 IP 地址不安全。FCrDNS 验证提供了很好的保护水平，但可能会受到某些类型攻击的影响。仅在绝对必要时在白名单中指定主机名，并确保所有用于名称解析的组件，如 DNS 服务器，都在您的控制之下。您还可以使用 hosts 文件在本地实现名称解析，以避免使用外部组件。
