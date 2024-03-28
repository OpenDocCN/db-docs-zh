# 20.6.2 用安全套接字层（SSL）保护组通信连接

> 原文：[`dev.mysql.com/doc/refman/8.0/en/group-replication-secure-socket-layer-support-ssl.html`](https://dev.mysql.com/doc/refman/8.0/en/group-replication-secure-socket-layer-support-ssl.html)

安全套接字可以用于组内成员之间的组通信连接。

Group Replication 系统变量`group_replication_ssl_mode`用于激活 SSL 用于组通信连接并指定连接的安全模式。默认设置意味着不使用 SSL。该选项有以下可能的值：

**表 20.1 group_replication_ssl_mode 配置值**

| 值 | 描述 |
| --- | --- |
| `DISABLED` | 建立一个未加密的连接（默认设置）。 |
| `REQUIRED` | 如果服务器支持安全连接，请建立安全连接。 |
| `VERIFY_CA` | 类似于`REQUIRED`，但另外根据配置的证书颁发机构（CA）证书验证服务器 TLS 证书。 |
| `VERIFY_IDENTITY` | 类似于`VERIFY_CA`，但另外验证服务器证书是否与尝试连接的主机匹配。 |

如果使用 SSL，则配置安全连接的方式取决于是使用 XCom 还是 MySQL 通信堆栈进行组通信（自 MySQL 8.0.27 起可选择两者之间的一个）。

**当使用 XCom 通信堆栈（`group_replication_communication_stack=XCOM`）时：** Group Replication 的组通信连接的其余配置取自服务器的 SSL 配置。有关配置服务器 SSL 的选项的更多信息，请参阅加密连接的命令选项。应用于 Group Replication 组通信连接的服务器 SSL 选项如下：

**表 20.2 SSL 选项**

| 服务器配置 | 描述 |
| --- | --- |
| `ssl_key` | SSL 私钥文件的路径名，格式为 PEM。在客户端端，这是客户端私钥。在服务器端，这是服务器私钥。 |
| `ssl_cert` | SSL 公钥证书文件的路径名，格式为 PEM。在客户端端，这是客户端公钥证书。在服务器端，这是服务器公钥证书。 |
| `ssl_ca` | 证书颁发机构（CA）证书文件的路径名，格式为 PEM。 |
| `ssl_capath` | 包含受信任的 SSL 证书颁发机构（CA）证书文件的目录路径名，格式为 PEM。 |
| `ssl_crl` | 包含证书吊销列表的 PEM 格式文件的路径名。 |
| `ssl_crlpath` | 包含 PEM 格式证书吊销列表文件的目录路径名。 |
| `ssl_cipher` | 用于加密连接的可接受密码列表。 |
| `tls_version` | 服务器允许用于加密连接的 TLS 协议列表。 |
| `tls_ciphersuites` | 服务器允许用于加密连接的 TLSv1.3 密码套件。 |

重要提示

+   从 MySQL 8.0.28 版本开始，MySQL Server 移除了对 TLSv1 和 TLSv1.1 连接协议的支持。这些协议从 MySQL 8.0.26 开始被弃用，尽管 MySQL Server 客户端，包括作为客户端的 Group Replication 服务器实例，如果使用了弃用的 TLS 协议版本，不会向用户返回警告。有关更多信息，请参阅 移除对 TLSv1 和 TLSv1.1 协议的支持。

+   从 MySQL 8.0.16 版本开始，MySQL Server 支持 TLSv1.3 协议，前提是 MySQL Server 使用 OpenSSL 1.1.1 进行编译。服务器在启动时检查 OpenSSL 的版本，如果低于 1.1.1，则将从与 TLS 版本相关的服务器系统变量的默认值中移除 TLSv1.3（包括 `group_replication_recovery_tls_version` 系统变量）。

+   MySQL 8.0.18 版本开始，Group Replication 支持 TLSv1.3。在 MySQL 8.0.16 和 MySQL 8.0.17 中，如果服务器支持 TLSv1.3，则协议不受组通信引擎支持，无法被 Group Replication 使用。

+   在 MySQL 8.0.18 中，TLSv1.3 可用于 Group Replication 的分布式恢复连接，但 `group_replication_recovery_tls_version` 和 `group_replication_recovery_tls_ciphersuites` 系统变量不可用。因此，捐赠服务器必须允许至少一个默认启用的 TLSv1.3 密码套件的使用，如 第 8.3.2 节，“加密连接 TLS 协议和密码” 中所列。从 MySQL 8.0.19 开始，您可以使用选项配置客户端支持任何选择的密码套件，包括仅使用非默认密码套件。

+   在`tls_version`系统变量中指定的 TLS 协议列表中，确保指定的版本是连续的（例如，`TLSv1.2,TLSv1.3`）。如果协议列表中存在任何间隙（例如，如果您指定了`TLSv1,TLSv1.2`，省略了 TLS 1.1），Group Replication 可能无法建立组通信连接。

在一个复制组中，OpenSSL 协商使用所有成员支持的最高 TLS 协议。一个配置为仅使用 TLSv1.3（`tls_version=TLSv1.3`）的加入成员无法加入一个不支持 TLSv1.3 的复制组，因为在这种情况下，组成员使用较低的 TLS 协议版本。为了将成员加入到组中，您必须配置加入成员也允许使用现有组成员支持的较低的 TLS 协议版本。相反，如果一个加入成员不支持 TLSv1.3，但现有组成员都支持并且在彼此之间的连接中使用该版本，那么如果现有组成员已经允许使用合适的较低 TLS 协议版本，或者您对其进行配置，该成员可以加入。在这种情况下，OpenSSL 使用较低的 TLS 协议版本进行每个成员到加入成员的连接。每个成员与其他现有成员的连接继续使用两个成员都支持的最高可用协议。

从 MySQL 8.0.16 开始，您可以在运行时更改 `tls_version` 系统变量以更改服务器的允许 TLS 协议版本列表。请注意，对于 Group Replication，`ALTER INSTANCE RELOAD TLS` 语句重新配置服务器的 TLS 上下文，从定义上下文的系统变量的当前值中获取，不会在运行 Group Replication 时更改 Group Replication 的组通信连接的 TLS 上下文。要将重新配置应用于这些连接，必须执行 `STOP GROUP_REPLICATION`，然后执行 `START GROUP_REPLICATION` 以重新启动已更改 `tls_version` 系统变量的成员或成员的 Group Replication。同样，如果要使组的所有成员更改为使用更高或更低的 TLS 协议版本，必须在更改允许的 TLS 协议版本列表后，在成员上执行滚动重启 Group Replication，以便 OpenSSL 在完成滚动重启时协商使用更高的 TLS 协议版本。有关在运行时更改允许的 TLS 协议版本列表的说明，请参见 第 8.3.2 节，“加密连接 TLS 协议和密码” 和 服务器端运行时配置和监视加密连接。

以下示例显示了配置服务器上 SSL 并激活 SSL 用于组复制组通信连接的 `my.cnf` 文件中的部分内容：

```sql
[mysqld]
ssl_ca = "cacert.pem"
ssl_capath = "/.../ca_directory"
ssl_cert = "server-cert.pem"
ssl_cipher = "DHE-RSA-AEs256-SHA"
ssl_crl = "crl-server-revoked.crl"
ssl_crlpath = "/.../crl_directory"
ssl_key = "server-key.pem"
group_replication_ssl_mode= REQUIRED
```

重要提示

`ALTER INSTANCE RELOAD TLS` 语句重新配置服务器的 TLS 上下文，从定义上下文的系统变量的当前值中获取，不会在运行 Group Replication 时更改 Group Replication 的组通信连接的 TLS 上下文。要将重新配置应用于这些连接，必须执行 `STOP GROUP_REPLICATION`，然后执行 `START GROUP_REPLICATION` 以重新启动 Group Replication。

加入成员和现有成员之间进行的分布式恢复连接不受上述选项的覆盖。这些连接使用 Group Replication 的专用分布式恢复 SSL 选项，详细描述在第 20.6.3.2 节，“用于分布式恢复的安全套接字层（SSL）连接”。

**当使用 MySQL 通信堆栈（group_replication_communication_stack=MYSQL）时：** 分布式恢复组的安全设置应用于组成员之间的正常通信。请参阅第 20.6.3 节，“保护分布式恢复连接”以了解如何配置安全设置。
