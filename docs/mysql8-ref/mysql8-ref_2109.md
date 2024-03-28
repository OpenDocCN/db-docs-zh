> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-host-cache-table.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-host-cache-table.html)

#### 29.12.21.3 主机缓存表

MySQL 服务器维护一个内存中的主机缓存，其中包含客户端主机名和 IP 地址信息，并用于避免域名系统（DNS）查找。[`host_cache`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-host-cache-table.html)表公开了此缓存的内容。[`host_cache_size`](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_host_cache_size)系统变量控制主机缓存的大小，以及[`host_cache`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-host-cache-table.html)表的大小。有关主机缓存的操作和配置信息，请参见第 7.1.12.3 节，“DNS 查找和主机缓存”。

因为[`host_cache`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-host-cache-table.html)表公开了主机缓存的内容，可以使用`SELECT`语句进行检查。这可能有助于诊断连接问题的原因。

[`host_cache`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-host-cache-table.html)表具有以下列：

+   `IP`

    以字符串形式表示连接到服务器的客户端的 IP 地址。

+   `HOST`

    客户端 IP 的解析 DNS 主机名，如果名称未知则为`NULL`。

+   `HOST_VALIDATED`

    客户端 IP 的 IP 到主机名到 IP DNS 解析是否成功。如果`HOST_VALIDATED`为`YES`，则`HOST`列用作对应于 IP 的主机名，以避免额外调用 DNS。当`HOST_VALIDATED`为`NO`时，将尝试为每个连接尝试进行 DNS 解析，直到最终以有效结果或永久错误完成。此信息使服务器能够在临时 DNS 故障期间避免缓存错误或缺失的主机名，这将永久地对客户端产生负面影响。

+   `SUM_CONNECT_ERRORS`

    被视为“阻塞”的连接错误数量（根据[`max_connect_errors`](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_max_connect_errors)系统变量进行评估）。仅计算协议握手错误，并且仅适用于通过验证的主机（`HOST_VALIDATED = YES`）。

    一旦给定主机的`SUM_CONNECT_ERRORS`达到`max_connect_errors`的值，来自该主机的新连接将被阻止。`SUM_CONNECT_ERRORS`值可以超过`max_connect_errors`的值，因为在主机未被阻止的情况下，可以同时发生来自主机的多个连接尝试。任何一个或全部都可能失败，独立地增加`SUM_CONNECT_ERRORS`，可能超过`max_connect_errors`的值。

    假设`max_connect_errors`为 200，对于给定主机的`SUM_CONNECT_ERRORS`为 199。如果有 10 个客户端同时尝试从该主机连接，由于`SUM_CONNECT_ERRORS`尚未达到 200，它们中的任何一个都不会被阻止。如果其中五个客户端发生阻止错误，对于每个客户端，`SUM_CONNECT_ERRORS`会增加一个，导致`SUM_CONNECT_ERRORS`值为 204。另外五个客户端成功连接且不被阻止，因为当它们的连接尝试开始时，`SUM_CONNECT_ERRORS`的值尚未达到 200。当`SUM_CONNECT_ERRORS`达到 200 后，从该主机发起的新连接将被阻止。

+   `COUNT_HOST_BLOCKED_ERRORS`

    因为`SUM_CONNECT_ERRORS`超过`max_connect_errors`系统变量值而被阻止的连接数量。

+   `COUNT_NAMEINFO_TRANSIENT_ERRORS`

    IP 到主机名 DNS 解析过程中的瞬时错误次数。

+   `COUNT_NAMEINFO_PERMANENT_ERRORS`

    IP 到主机名 DNS 解析过程中的永久错误次数。

+   `COUNT_FORMAT_ERRORS`

    主机名格式错误的数量。MySQL 不会对`mysql.user`系统表中`Host`列值与完全由数字组成的主机名进行匹配，例如`1.2.example.com`。而是使用客户端 IP 地址。关于为什么不进行这种匹配的原因，请参见 Section 8.2.4, “Specifying Account Names”。

+   `COUNT_ADDRINFO_TRANSIENT_ERRORS`

    主机名到 IP 反向 DNS 解析过程中出现的瞬时错误次数。

+   `COUNT_ADDRINFO_PERMANENT_ERRORS`

    主机名到 IP 反向 DNS 解析过程中的永久错误次数。

+   `COUNT_FCRDNS_ERRORS`

    前向确认反向 DNS 错误的数量。当 IP 到主机名到 IP DNS 解析产生一个与客户端原始 IP 地址不匹配的 IP 地址时，就会发生这些错误。

+   `COUNT_HOST_ACL_ERRORS`

    出现的错误次数，因为没有用户被允许从客户端主机连接。在这种情况下，服务器返回`ER_HOST_NOT_PRIVILEGED`，甚至不要求用户名或密码。

+   `COUNT_NO_AUTH_PLUGIN_ERRORS`

    由于请求不可用的身份验证插件而导致的错误数量。例如，如果插件从未加载或加载尝试失败，则插件可能不可用。

+   `COUNT_AUTH_PLUGIN_ERRORS`

    身份验证插件报告的错误数量。

    身份验证插件可以报告不同的错误代码以指示失败的根本原因。根据错误类型，将递增以下列之一：`COUNT_AUTHENTICATION_ERRORS`、`COUNT_AUTH_PLUGIN_ERRORS`、`COUNT_HANDSHAKE_ERRORS`。新的返回代码是现有插件 API 的可选扩展。未知或意外的插件错误计入`COUNT_AUTH_PLUGIN_ERRORS`列。

+   `COUNT_HANDSHAKE_ERRORS`

    在协议层面检测到的错误数量。

+   `COUNT_PROXY_USER_ERRORS`

    当代理用户 A 被代理到另一个不存在的用户 B 时检测到的错误数量。

+   `COUNT_PROXY_USER_ACL_ERRORS`

    当代理用户 A 被代理到另一个存在但 A 没有`PROXY`权限的用户 B 时检测到的错误数量。

+   `COUNT_AUTHENTICATION_ERRORS`

    由于身份验证失败导致的错误数量。

+   `COUNT_SSL_ERRORS`

    由于 SSL 问题导致的错误数量。

+   `COUNT_MAX_USER_CONNECTIONS_ERRORS`

    由于超出每用户连接配额而导致的错误数量。请参阅 Section 8.2.21, “Setting Account Resource Limits”。

+   `COUNT_MAX_USER_CONNECTIONS_PER_HOUR_ERRORS`

    由于超出每小时每用户连接配额而导致的错误数量。请参阅 Section 8.2.21, “Setting Account Resource Limits”。

+   `COUNT_DEFAULT_DATABASE_ERRORS`

    与默认数据库相关的错误数量。例如，数据库不存在或用户没有访问权限。

+   `COUNT_INIT_CONNECT_ERRORS`

    由于`init_connect`系统变量值中语句执行失败而导致的错误数量。

+   `COUNT_LOCAL_ERRORS`

    与服务器实现本地的与网络、身份验证或授权无关的错误数量。例如，内存不足情况属于此类别。

+   `COUNT_UNKNOWN_ERRORS`

    在此表中其他未被其他列考虑的未知错误数量。该列保留用于将来使用，以防必须报告新的错误条件，并且如果需要保留`host_cache`表的向后兼容性和结构。

+   `FIRST_SEEN`

    在`IP`列中看到的客户端首次连接尝试的时间戳。

+   `LAST_SEEN`

    在`IP`列中看到的客户端最近连接尝试的时间戳。

+   `FIRST_ERROR_SEEN`

    客户端在`IP`列中首次出现错误的时间戳。

+   `LAST_ERROR_SEEN`

    在`IP`列中记录客户端最近出现错误的时间戳。

`host_cache`表具有以下索引：

+   (`IP`)上的主键

+   (`HOST`)上的索引

对于`host_cache`表，允许使用`TRUNCATE TABLE`语句。需要对该表具有`DROP`权限。清空表会刷新主机缓存，具有刷新主机缓存中描述的效果。
