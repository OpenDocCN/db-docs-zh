# 20.5.5 支持 IPv6 和混合 IPv6 和 IPv4 组

> 原文：[`dev.mysql.com/doc/refman/8.0/en/group-replication-ipv6.html`](https://dev.mysql.com/doc/refman/8.0/en/group-replication-ipv6.html)

从 MySQL 8.0.14 开始，Group Replication 组成员可以使用 IPv6 地址作为组内通信的替代方案，而不是 IPv4 地址。要使用 IPv6 地址，服务器主机上的操作系统和 MySQL Server 实例都必须配置为支持 IPv6。有关为服务器实例设置 IPv6 支持的说明，请参见 Section 7.1.13, “IPv6 Support”。

IPv6 地址，或解析为其的主机名，可以被指定为成员在 `group_replication_local_address` 选项中提供给其他成员连接的网络地址。当与端口号一起指定时，IPv6 地址必须用方括号指定，例如：

```sql
group_replication_local_address= "[2001:db8:85a3:8d3:1319:8a2e:370:7348]:33061"
```

在 `group_replication_local_address` 中指定的网络地址或主机名被 Group Replication 用作复制组内组成员的唯一标识符。如果为服务器实例指定的 Group Replication 本地地址的主机名解析为 IPv4 和 IPv6 地址，那么始终使用 IPv4 地址进行 Group Replication 连接。指定为 Group Replication 本地地址的地址或主机名与 MySQL 服务器 SQL 协议主机和端口不同，并且不在服务器实例的 `bind_address` 系统变量中指定。为了 Group Replication 的 IP 地址权限目的（参见 Section 20.6.4, “Group Replication IP Address Permissions”），您在 `group_replication_local_address` 中为每个组成员指定的地址必须添加到复制组中其他服务器的 `group_replication_ip_allowlist`（从 MySQL 8.0.22 开始）或 `group_replication_ip_whitelist` 系统变量的列表中。

复制组可以包含将 IPv6 地址作为其 Group Replication 本地地址的成员组合，以及呈现 IPv4 地址的成员。当服务器加入这样的混合组时，必须使用种子成员在 `group_replication_group_seeds` 选项中广告的协议进行初始联系，无论是 IPv4 还是 IPv6。如果组的任何种子成员在 `group_replication_group_seeds` 选项中列出了具有 IPv6 地址的情况，而加入成员具有 IPv4 Group Replication 本地地址，或者反之，则还必须为加入成员为所需协议设置和允许替代地址（或解析为该协议地址的主机名）。如果加入成员没有适当协议的允许地址，则其连接尝试将被拒绝。替代地址或主机名只需要添加到复制组中其他服务器的 `group_replication_ip_allowlist`（从 MySQL 8.0.22 开始）或 `group_replication_ip_whitelist` 系统变量中，而不需要添加到加入成员的 `group_replication_local_address` 值（该值只能包含单个地址）。

例如，服务器 A 是一个组的种子成员，并具有以下 Group Replication 的配置设置，以便在 `group_replication_group_seeds` 选项中广告 IPv6 地址：

```sql
group_replication_bootstrap_group=on
group_replication_local_address= "[2001:db8:85a3:8d3:1319:8a2e:370:7348]:33061"
group_replication_group_seeds= "[2001:db8:85a3:8d3:1319:8a2e:370:7348]:33061"
```

服务器 B 是组的加入成员，并具有以下 Group Replication 的配置设置，以便具有 IPv4 Group Replication 本地地址：

```sql
group_replication_bootstrap_group=off
group_replication_local_address= "203.0.113.21:33061"
group_replication_group_seeds= "[2001:db8:85a3:8d3:1319:8a2e:370:7348]:33061"
```

服务器 B 还具有替代 IPv6 地址 `2001:db8:8b0:40:3d9c:cc43:e006:19e8`。为了成功加入组，服务器 B 的 IPv4 Group Replication 本地地址和其替代 IPv6 地址都必须列在服务器 A 的允许列表中，如下例所示：

```sql
group_replication_ip_allowlist=
"203.0.113.0/24,2001:db8:85a3:8d3:1319:8a2e:370:7348,
2001:db8:8b0:40:3d9c:cc43:e006:19e8"
```

作为 Group Replication IP 地址权限的最佳实践，服务器 B（以及所有其他组成员）应具有与服务器 A 相同的允许列表，除非安全要求另有规定。

如果复制组的任何成员或所有成员使用不支持在组复制中使用 IPv6 地址的较旧 MySQL Server 版本，则成员无法使用 IPv6 地址（或解析为 IPv6 地址的主机名）作为其组复制本地地址参与组。这适用于至少一个现有成员使用 IPv6 地址且不支持此功能的新成员尝试加入的情况，以及新成员尝试使用 IPv6 地址加入但组中至少有一个成员不支持此功能的情况。在每种情况下，新成员都无法加入。要使加入成员提供 IPv4 地址进行组通信，您可以将 `group_replication_local_address` 的值更改为 IPv4 地址，或配置您的 DNS 将加入成员的现有主机名解析为 IPv4 地址。在将每个组成员升级到支持 IPv6 用于组复制的 MySQL Server 版本后，您可以将每个成员的 `group_replication_local_address` 值更改为 IPv6 地址，或配置您的 DNS 提供 IPv6 地址。更改 `group_replication_local_address` 的值仅在停止并重新启动组复制时生效。

IPv6 地址还可以用作分布式恢复端点，从 MySQL 8.0.21 开始，可以使用 `group_replication_advertise_recovery_endpoints` 系统变量指定。在此列表中使用的地址适用相同的规则。请参阅 Section 20.5.4.1, “Connections for Distributed Recovery”。
