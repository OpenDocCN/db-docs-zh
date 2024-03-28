# 22.5.3 使用 X 插件进行加密连接

> 原文：[`dev.mysql.com/doc/refman/8.0/en/x-plugin-encrypted-connections.html`](https://dev.mysql.com/doc/refman/8.0/en/x-plugin-encrypted-connections.html)

本节解释了如何配置 X 插件以使用加密连接。有关更多背景信息，请参阅 Section 8.3, “Using Encrypted Connections”。

要启用对加密连接的支持进行配置，X 插件具有`mysqlx_ssl_*`xxx`*`系统变量，这些变量的值可以与用于 MySQL 服务器的`ssl_*`xxx`*`系统变量不同。例如，X 插件可以具有与 MySQL 服务器不同的 SSL 密钥、证书和证书颁发机构文件。这些变量在 Section 22.5.6.2, “X Plugin Options and System Variables”中有描述。同样，X 插件具有其自己的`Mysqlx_ssl_*`xxx`*`状态变量，对应于 MySQL 服务器加密连接的`Ssl_*`xxx`*`状态变量。请参阅 Section 22.5.6.3, “X Plugin Status Variables”。

在初始化时，X 插件确定其用于加密连接的 TLS 上下文如下：

+   如果所有`mysqlx_ssl_*`xxx`*`系统变量都具有其默认值，则 X 插件使用与 MySQL 服务器主连接接口相同的 TLS 上下文，该接口由`ssl_*`xxx`*`系统变量的值确定。

+   如果任何`mysqlx_ssl_*`xxx`*`变量具有非默认值，则 X 插件使用由其自身系统变量的值定义的 TLS 上下文。（如果任何`mysqlx_ssl_*`xxx`*`系统变量设置为与其默认值不同的值，则是这种情况。）

这意味着，在启用 X 插件的服务器上，您可以选择通过仅设置`ssl_*`xxx`*`变量来共享 MySQL 协议和 X 协议连接的相同加密配置，或者通过分别配置`ssl_*`xxx`*`和`mysqlx_ssl_*`xxx`*`变量来为 MySQL 协议和 X 协议连接设置不同的加密配置。

要使 MySQL 协议和 X 协议连接使用相同的加密配置，请在`my.cnf`中仅设置`ssl_*`xxx`*`系统变量：

```sql
[mysqld]
ssl_ca=ca.pem
ssl_cert=server-cert.pem
ssl_key=server-key.pem
```

要为 MySQL 协议和 X 协议连接分别配置加密，请在`my.cnf`中设置`ssl_*`xxx`*`和`mysqlx_ssl_*`xxx`*`系统变量：

```sql
[mysqld]
ssl_ca=ca1.pem
ssl_cert=server-cert1.pem
ssl_key=server-key1.pem

mysqlx_ssl_ca=ca2.pem
mysqlx_ssl_cert=server-cert2.pem
mysqlx_ssl_key=server-key2.pem
```

有关配置连接加密支持的一般信息，请参阅 Section 8.3.1, “Configuring MySQL to Use Encrypted Connections”。该讨论是针对 MySQL 服务器编写的，但参数名称对于 X 插件是类似的。（X 插件的`mysqlx_ssl_*`xxx`*`系统变量名称对应于 MySQL 服务器的`ssl_*`xxx`*`系统变量名称。）

`tls_version` 系统变量确定了 MySQL 协议连接所允许的 TLS 版本，也适用于 X 协议连接。因此，这两种连接类型所允许的 TLS 版本是相同的。

每个连接的加密是可选的，但可以通过在创建用户的 `CREATE USER` 语句中包含适当的 `REQUIRE` 子句来要求特定用户在 X 协议和 MySQL 协议连接中使用加密。有关详细信息，请参见 Section 15.7.1.3, “CREATE USER Statement”。或者，要求所有用户在 X 协议和 MySQL 协议连接中使用加密，可以启用 `require_secure_transport` 系统变量。有关更多信息，请参见 将加密连接配置为强制性。
