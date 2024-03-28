# 6.2.6 使用 DNS SRV 记录连接服务器

> 原文：[`dev.mysql.com/doc/refman/8.0/en/connecting-using-dns-srv.html`](https://dev.mysql.com/doc/refman/8.0/en/connecting-using-dns-srv.html)

在域名系统（DNS）中，SRV 记录（服务位置记录）是一种资源记录类型，使客户端能够指定指示服务、协议和域的名称。对名称进行 DNS 查找会返回一个包含提供所需服务的域中多个可用服务器名称的回复。有关 DNS SRV 的信息，包括记录如何定义列出服务器的优先顺序，请参阅 [RFC 2782](https://tools.ietf.org/html/rfc2782)。

MySQL 支持使用 DNS SRV 记录连接服务器。接收到 DNS SRV 查找结果的客户端会尝试按照 DNS 管理员为每个主机分配的优先级和权重的顺序连接到 MySQL 服务器。仅当客户端无法连接到任何服务器时才会发生连接失败。

当多个 MySQL 实例（例如服务器集群）为您的应用程序提供相同的服务时，DNS SRV 记录可用于帮助故障转移、负载平衡和复制服务。对于应用程序直接管理连接尝试的候选服务器集合来说是繁琐的，DNS SRV 记录提供了一种替代方案：

+   DNS SRV 记录使 DNS 管理员能够将单个 DNS 域映射到多个服务器。当服务器被添加或从配置中移除，或者它们的主机名发生更改时，DNS SRV 记录也可以由管理员在中心位置进行更新。

+   集中管理 DNS SRV 记录消除了个别客户端在连接请求中识别每个可能主机的需求，或者连接由额外的软件组件处理的需求。应用程序可以使用 DNS SRV 记录获取关于候选 MySQL 服务器的信息，而不是自行管理服务器信息。

+   DNS SRV 记录可以与连接池结合使用，这种情况下，当不再在当前 DNS SRV 记录列表中的主机变得空闲时，连接将从池中移除。

MySQL 支持在以下情境中使用 DNS SRV 记录连接服务器：

+   几个 MySQL 连接器实现了 DNS SRV 支持；连接器特定选项使得可以请求 DNS SRV 记录查找，无论是用于 X 协议连接还是经典的 MySQL 协议连接。有关一般信息，请参阅 使用 DNS SRV 记录连接。有关详细信息，请参阅各个 MySQL 连接器的文档。

+   C API 提供了一个`mysql_real_connect_dns_srv()`函数，类似于`mysql_real_connect()`，不同之处在于参数列表不指定要连接到的 MySQL 服务器的特定主机。而是指定一个指定一组服务器的 DNS SRV 记录。参见 mysql_real_connect_dns_srv()。

+   **mysql**客户端具有一个`--dns-srv-name`选项，用于指示指定一组服务器的 DNS SRV 记录。参见 Section 6.5.1, “mysql — The MySQL Command-Line Client”。

DNS SRV 名称由服务、协议和域组成，其中服务和协议分别以下划线为前缀：

```sql
_*service*._*protocol*.*domain*
```

以下 DNS SRV 记录标识了多个候选服务器，例如客户端用于建立 X 协议连接：

```sql
Name                      TTL   Class  Priority Weight Port  Target
_mysqlx._tcp.example.com. 86400 IN SRV 0        5      33060 server1.example.com.
_mysqlx._tcp.example.com. 86400 IN SRV 0        10     33060 server2.example.com.
_mysqlx._tcp.example.com. 86400 IN SRV 10       5      33060 server3.example.com.
_mysqlx._tcp.example.com. 86400 IN SRV 20       5      33060 server4.example.com.
```

在这里，`mysqlx`表示 X 协议服务，`tcp`表示 TCP 协议。客户端可以使用名称`_mysqlx._tcp.example.com`请求此 DNS SRV 记录。指定连接请求中的名称的特定语法取决于客户端类型。例如，客户端可能支持在类似 URI 的连接字符串中指定名称或作为键值对。

经典协议连接的 DNS SRV 记录可能如下所示：

```sql
Name                     TTL   Class  Priority Weight  Port Target
_mysql._tcp.example.com. 86400 IN SRV 0        5       3306 server1.example.com.
_mysql._tcp.example.com. 86400 IN SRV 0        10      3306 server2.example.com.
_mysql._tcp.example.com. 86400 IN SRV 10       5       3306 server3.example.com.
_mysql._tcp.example.com. 86400 IN SRV 20       5       3306 server4.example.com.
```

在这里，名称`mysql`指定了经典 MySQL 协议服务，端口为 3306（默认经典 MySQL 协议端口），而不是 33060（默认 X 协议端口）。

使用 DNS SRV 记录查找时，客户端通常必须遵循这些连接请求规则（可能存在客户端或连接器特定的例外情况）：

+   请求必须指定完整的 DNS SRV 记录名称，服务和协议名称前缀为下划线。

+   请求不能指定多个主机名。

+   请求不能指定端口号。

+   仅支持 TCP 连接。不能使用 Unix 套接字文件、Windows 命名管道和共享内存。

有关在 X DevAPI 中使用基于 DNS SRV 的连接的更多信息，请参见使用 DNS SRV 记录进行连接。
