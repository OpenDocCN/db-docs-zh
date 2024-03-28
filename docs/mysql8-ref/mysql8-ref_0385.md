# 8.3.1 配置 MySQL 使用加密连接

> 原文：[`dev.mysql.com/doc/refman/8.0/en/using-encrypted-connections.html`](https://dev.mysql.com/doc/refman/8.0/en/using-encrypted-connections.html)

有几个配置参数可用于指示是否使用加密连接，并指定适当的证书和密钥文件。本节提供了有关为加密连接配置服务器和客户端的一般指导：

+   用于加密连接的服务器端启动配置

+   加密连接的服务器端运行时配置和监控

+   用于加密连接的客户端端配置

+   将加密连接配置为强制性

还可以在其他情境中使用加密连接，如这些附加部分中所讨论的：

+   源服务器和副本复制服务器之间。参见 第 19.3.1 节，“设置复制以使用加密连接”。

+   在组复制服务器之间。参见 第 20.6.2 节，“使用安全套接字层（SSL）保护组通信连接”。

+   基于 MySQL C API 的客户端程序。参见 支持加密连接。

有关创建所需证书和密钥文件的说明，请参阅 第 8.3.3 节，“创建 SSL 和 RSA 证书和密钥”。

#### 用于加密连接的服务器端启动配置

在服务器端，`--ssl` 选项指定服务器允许但不要求加密连接。此选项默认启用，因此不需要显式指定。

要求客户端使用加密连接连接，请启用 `require_secure_transport` 系统变量。参见 将加密连接配置为强制性。

服务器端的这些系统变量指定了服务器在允许客户端建立加密连接时使用的证书和密钥文件：

+   `ssl_ca`：证书颁发机构（CA）证书文件的路径名。(`ssl_capath`类似，但指定了 CA 证书文件的目录路径名。)

+   `ssl_cert`：服务器公钥证书文件的路径名。此证书可以发送给客户端，并根据其拥有的 CA 证书进行验证。

+   `ssl_key`：服务器私钥文件的路径名。

例如，要启用服务器进行加密连接，请在`my.cnf`文件中使用以下行启动它，根据需要更改文件名：

```sql
[mysqld]
ssl_ca=ca.pem
ssl_cert=server-cert.pem
ssl_key=server-key.pem
```

要额外指定客户端必须使用加密连接，请启用`require_secure_transport`系统变量：

```sql
[mysqld]
ssl_ca=ca.pem
ssl_cert=server-cert.pem
ssl_key=server-key.pem
require_secure_transport=ON
```

每个证书和密钥系统变量都指定了一个 PEM 格式的文件。如果需要创建所需的证书和密钥文件，请参阅第 8.3.3 节，“创建 SSL 和 RSA 证书和密钥”。使用 OpenSSL 编译的 MySQL 服务器可以在启动时自动生成缺失的证书和密钥文件。请参阅第 8.3.3.1 节，“使用 MySQL 创建 SSL 和 RSA 证书和密钥”。或者，如果您有 MySQL 源代码分发版，您可以使用其`mysql-test/std_data`目录中的演示证书和密钥文件测试您的设置。

服务器执行证书和密钥文件的自动发现。如果除了`--ssl`（可能还包括`ssl_cipher`）之外没有给出明确的加密连接选项来配置加密连接，服务器会尝试在启动时自动启用加密连接支持：

+   如果服务器在数据目录中发现名为`ca.pem`、`server-cert.pem`和`server-key.pem`的有效证书和密钥文件，它会启用客户端的加密连接支持。（文件不一定是自动生成的；重要的是它们具有这些名称并且是有效的。）

+   如果服务器在数据目录中找不到有效的证书和密钥文件，它会继续执行，但不支持加密连接。

如果服务器自动启用了加密连接支持，它会在错误日志中写下一条说明。如果服务器发现 CA 证书是自签名的，它会在错误日志中写下一个警告。（如果证书是由服务器自动创建或使用 **mysql_ssl_rsa_setup** 手动创建，则证书是自签名的。）

MySQL 还为服务器端提供了这些系统变量来控制加密连接：

+   `ssl_cipher`：连接加密的可允许密码列表。

+   `ssl_crl`：包含证书吊销列表的文件的路径名。（`ssl_crlpath` 类似，但指定了证书吊销列表文件的目录路径名。）

+   `tls_version`、`tls_ciphersuites`：服务器允许用于加密连接的加密协议和密码套件；参见 Section 8.3.2, “Encrypted Connection TLS Protocols and Ciphers”。例如，您可以配置 `tls_version` 来阻止客户端使用不太安全的协议。

如果服务器无法从服务器端加密连接控制的系统变量创建有效的 TLS 上下文，则服务器将在没有加密连接支持的情况下执行。

#### 用于加密连接的服务器端运行时配置和监控

在 MySQL 8.0.16 之前，配置加密连接支持的 `tls_*`xxx`*` 和 `ssl_*`xxx`*` 系统变量只能在服务器启动时设置。因此，这些系统变量确定服务器为所有新连接使用的 TLS 上下文。

从 MySQL 8.0.16 开始，`tls_*`xxx`*` 和 `ssl_*`xxx`*` 系统变量是动态的，可以在运行时设置，而不仅仅是在启动时。如果使用 `SET GLOBAL` 进行更改，新值仅在服务器重新启动之前有效。如果使用 `SET PERSIST` 进行更改，新值也会延续到后续的服务器重新启动。参见 Section 15.7.6.1, “SET Syntax for Variable Assignment”。然而，对这些变量的运行时更改不会立即影响新连接的 TLS 上下文，后文将对此进行解释。

随着 MySQL 8.0.16 中的更改，使得可以在运行时更改与 TLS 上下文相关的系统变量，服务器还启用了对用于新连接的实际 TLS 上下文的运行时更新。例如，这种功能可能很有用，以避免重新启动运行时间过长以至于 SSL 证书已过期的 MySQL 服务器。

要创建初始 TLS 上下文，服务器使用启动时上下文相关系统变量的值。为了公开上下文值，服务器还初始化了一组相应的状态变量。以下表格显示了定义 TLS 上下文的系统变量以及公开当前活动上下文值的相应状态变量。

**表 8.12 服务器主连接接口 TLS 上下文的系统和状态变量**

| 系统变量名称 | 对应的状态变量名称 |
| --- | --- |
| `ssl_ca` | `Current_tls_ca` |
| `ssl_capath` | `Current_tls_capath` |
| `ssl_cert` | `Current_tls_cert` |
| `ssl_cipher` | `Current_tls_cipher` |
| `ssl_crl` | `Current_tls_crl` |
| `ssl_crlpath` | `Current_tls_crlpath` |
| `ssl_key` | `Current_tls_key` |
| `tls_ciphersuites` | `Current_tls_ciphersuites` |
| `tls_version` | `Current_tls_version` |

截至 MySQL 8.0.21，这些活动的 TLS 上下文值也作为性能模式 `tls_channel_status` 表中的属性公开，以及任何其他活动 TLS 上下文的属性。

要在运行时重新配置 TLS 上下文，请使用以下过程：

1.  设置每个应更改为新值的与 TLS 上下文相关的系统变量。

1.  执行`ALTER INSTANCE RELOAD TLS`。该语句会从 TLS 上下文相关系统变量的当前值重新配置活动的 TLS 上下文。它还会设置相关状态变量以反映新的活动上下文值。该语句需要`CONNECTION_ADMIN`权限。

1.  在执行`ALTER INSTANCE RELOAD TLS`后建立的新连接将使用新的 TLS 上下文。现有连接不受影响。如果需要终止现有连接，请使用`KILL`语句。

每对系统和状态变量的成员可能由于重新配置过程的方式而暂时具有不同的值：

+   在`ALTER INSTANCE RELOAD TLS`之前对系统变量的更改不会改变 TLS 上下文。在这一点上，这些更改对新连接没有影响，相应的上下文相关系统和状态变量可能具有不同的值。这使您可以对个别系统变量进行任何所需的更改，然后在所有系统变量更改完成后使用`ALTER INSTANCE RELOAD TLS`原子地更新活动的 TLS 上下文。

+   在`ALTER INSTANCE RELOAD TLS`之后，相应的系统和状态变量具有相同的值。这种情况会一直持续，直到对系统变量进行下一次更改。

在某些情况下，仅通过`ALTER INSTANCE RELOAD TLS`可能足以重新配置 TLS 上下文，而无需更改任何系统变量。假设由`ssl_cert`指定的文件中的证书已过期。只需用非过期证书替换现有文件内容，并执行`ALTER INSTANCE RELOAD TLS`即可使新文件内容被读取并用于新连接。

截至 MySQL 8.0.21，服务器为管理连接接口实现了独立的连接加密配置。请参阅 Administrative Interface Support for Encrypted Connections。此外，`ALTER INSTANCE RELOAD TLS` 还通过 `FOR CHANNEL` 子句扩展，允许指定要重新加载 TLS 上下文的通道（接口）。请参阅 Section 15.1.5, “ALTER INSTANCE Statement”。没有状态变量来公开管理接口的 TLS 上下文，但是性能模式 `tls_channel_status` 表公开了主接口和管理接口的 TLS 属性。请参阅 Section 29.12.21.9, “The tls_channel_status Table”。

更新主接口的 TLS 上下文会产生以下影响：

+   此更新更改了主连接接口上用于新连接的 TLS 上下文。

+   此更新还会更改管理接口上用于新连接的 TLS 上下文，除非为该接口配置了某些非默认的 TLS 参数值。

+   此更新不会影响其他已启用的服务器插件或组件（如 Group Replication 或 X Plugin）使用的 TLS 上下文：

    +   要将主接口重新配置应用于 Group Replication 的组通信连接，这些连接从服务器的与 TLS 上下文相关的系统变量中获取其设置，必须执行 `STOP GROUP_REPLICATION`，然后执行 `START GROUP_REPLICATION` 来停止并重新启动 Group Replication。

    +   X Plugin 在插件初始化时初始化其 TLS 上下文，如 Section 22.5.3, “Using Encrypted Connections with X Plugin” 中所述。此上下文之后不会更改。

默认情况下，如果配置值不允许创建新的 TLS 上下文，则 `RELOAD TLS` 操作将出现错误并且不会生效。先前的上下文值将继续用于新连接。如果给定了可选的 `NO ROLLBACK ON ERROR` 子句并且无法创建新上下文，则不会发生回滚。相反，会生成警告，并且在适用语句的接口上为新连接禁用加密。

在连接接口上启用或禁用加密连接的选项仅在启动时生效。例如，`--ssl`和`--admin-ssl`选项仅在启动时影响主要和管理接口是否支持加密连接。这些选项在运行时被忽略，对`ALTER INSTANCE RELOAD TLS`的操作没有影响。例如，您可以使用`--ssl=OFF`在主接口上禁用加密连接启动服务器，然后重新配置 TLS 并执行`ALTER INSTANCE RELOAD TLS`在运行时启用加密连接。

#### 客户端端加密连接的配置

有关与建立加密连接相关的客户端选项的完整列表，请参阅加密连接的命令选项。

默认情况下，如果服务器支持加密连接，MySQL 客户端程序会尝试建立加密连接，进一步的控制可以通过`--ssl-mode`选项进行设置：

+   在没有`--ssl-mode`选项的情况下，客户端尝试使用加密连接进行连接，如果无法建立加密连接，则退回到未加密连接。这也是在明确指定`--ssl-mode=PREFERRED`选项时的行为。

+   使用`--ssl-mode=REQUIRED`，客户端需要一个加密连接，如果无法建立加密连接则失败。

+   使用`--ssl-mode=DISABLED`，客户端使用一个未加密的连接。

+   使用`--ssl-mode=VERIFY_CA`或`--ssl-mode=VERIFY_IDENTITY`，客户端需要一个加密连接，并且还会对服务器 CA 证书进行验证，并且（使用`VERIFY_IDENTITY`）对其证书中的服务器主机名进行验证。

重要提示

默认设置`--ssl-mode=PREFERRED`如果其他默认设置未更改，则会产生加密连接。然而，为了防止复杂的中间人攻击，客户端验证服务器的身份非常重要。设置`--ssl-mode=VERIFY_CA`和`--ssl-mode=VERIFY_IDENTITY`比默认设置更好，以帮助防止这种类型的攻击。`VERIFY_CA`使客户端检查服务器的证书是否有效。`VERIFY_IDENTITY`使客户端检查服务器的证书是否有效，并且使客户端检查客户端使用的主机名是否与服务器证书中的身份匹配。要实施这些设置之一，您必须首先确保服务器的 CA 证书可靠地对所有在您的环境中使用它的客户端可用，否则将导致可用性问题。因此，它们不是默认设置。

如果在服务器端启用了`require_secure_transport`系统变量以导致服务器要求加密连接，则尝试建立未加密连接将失败。请参阅将加密连接配置为强制性。

客户端端的以下选项标识客户端在与服务器建立加密连接时使用的证书和密钥文件。它们类似于服务器端使用的`ssl_ca`、`ssl_cert`和`ssl_key`系统变量，但`--ssl-cert`和`--ssl-key`标识客户端的公钥和私钥：

+   `--ssl-ca`：证书颁发机构（CA）证书文件的路径名。如果使用此选项，必须指定与服务器使用的相同证书。(`--ssl-capath`类似，但指定 CA 证书文件目录的路径名。)

+   `--ssl-cert`：客户端公钥证书文件的路径名。

+   `--ssl-key`：客户端私钥文件的路径名。

为了相对于默认加密提供额外的安全性，客户端可以提供与服务器使用的 CA 证书匹配的证书，并启用主机名身份验证。这样，服务器和客户端都信任同一 CA 证书，并且客户端验证连接的主机是预期的主机：

+   要指定 CA 证书，请使用`--ssl-ca`（或`--ssl-capath`），并指定`--ssl-mode=VERIFY_CA`。

+   若要启用主机名身份验证，请使用`--ssl-mode=VERIFY_IDENTITY`而不是`--ssl-mode=VERIFY_CA`。

注意

使用`VERIFY_IDENTITY`进行主机名身份验证无法与服务器自动创建的自签名证书或使用**mysql_ssl_rsa_setup**手动创建的证书一起使用（参见第 8.3.3.1 节，“使用 MySQL 创建 SSL 和 RSA 证书和密钥”）。这些自签名证书不包含服务器名称作为通用名称值。

在 MySQL 8.0.12 之前，主机名身份验证也无法与使用通配符指定通用名称的证书一起使用，因为该名称与服务器名称逐字比较。

MySQL 还为客户端提供了这些选项来控制客户端端的加密连接：

+   `--ssl-cipher`: 连接加密的允许密码列表。

+   `--ssl-crl`: 包含证书吊销列表的文件路径名。（`--ssl-crlpath`类似，但指定证书吊销列表文件目录的路径名。）

+   `--tls-version`, `--tls-ciphersuites`: 允许的加密协议和密码套件；参见第 8.3.2 节，“加密连接 TLS 协议和密码”。

根据客户端使用的 MySQL 帐户的加密要求，客户端可能需要指定某些选项以使用加密连接到 MySQL 服务器。

假设您想使用没有特殊加密要求的帐户连接，或者使用包含`REQUIRE SSL`子句的`CREATE USER`语句创建的帐户。假设服务器支持加密连接，客户端可以使用没有`--ssl-mode`选项或明确的`--ssl-mode=PREFERRED`选项连接使用加密：

```sql
mysql
```

或者：

```sql
mysql --ssl-mode=PREFERRED
```

对于使用`REQUIRE SSL`子句创建的帐户，如果无法建立加密连接，则连接尝试将失败。对于没有特殊加密要求的帐户，如果无法建立加密连接，则尝试回退到未加密连接。为了防止回退并在无法获取加密连接时失败，请这样连接：

```sql
mysql --ssl-mode=REQUIRED
```

如果帐户具有更严格的安全要求，则必须指定其他选项以建立加密连接：

+   对于使用`REQUIRE X509`子句创建的帐户，客户端必须至少指定`--ssl-cert`和`--ssl-key`。此外，建议使用`--ssl-ca`（或`--ssl-capath`）以便验证服务器提供的公共证书。例如（在一行上输入命令）：

    ```sql
    mysql --ssl-ca=ca.pem
          --ssl-cert=client-cert.pem
          --ssl-key=client-key.pem
    ```

+   对于使用`REQUIRE ISSUER`或`REQUIRE SUBJECT`子句创建的帐户，加密要求与`REQUIRE X509`相同，但证书必须分别与帐户定义中指定的发行者或主题匹配。

有关`REQUIRE`子句的更多信息，请参见第 15.7.1.3 节，“CREATE USER Statement”。

MySQL 服务器可以生成客户端证书和密钥文件，客户端可以使用这些文件连接到 MySQL 服务器实例。请参见第 8.3.3 节，“创建 SSL 和 RSA 证书和密钥”。

重要提示

如果连接到 MySQL 服务器实例的客户端使用具有`extendedKeyUsage`扩展（X.509 v3 扩展）的 SSL 证书，则扩展密钥用途必须包括客户端身份验证（`clientAuth`）。如果 SSL 证书仅用于服务器身份验证（`serverAuth`）和其他非客户端证书目的，证书验证将失败，客户端连接到 MySQL 服务器实例将失败。MySQL Server 生成的 SSL 证书中没有`extendedKeyUsage`扩展（如第 8.3.3.1 节，“使用 MySQL 创建 SSL 和 RSA 证书和密钥”中所述），并且使用**openssl**命令创建的 SSL 证书遵循第 8.3.3.2 节，“使用 openssl 创建 SSL 证书和密钥”中的说明。如果使用其他方式创建的客户端证书，请确保任何`extendedKeyUsage`扩展包括客户端身份验证。

为了防止使用加密并覆盖其他`--ssl-*`xxx`*`选项，请使用`--ssl-mode=DISABLED`调用客户端程序：

```sql
mysql --ssl-mode=DISABLED
```

要确定当前与服务器的连接是否使用加密，请检查 `Ssl_cipher` 状态变量的会话值。如果值为空，则连接未加密。否则，连接已加密，并且该值指示加密密码。例如：

```sql
mysql> SHOW SESSION STATUS LIKE 'Ssl_cipher';
+---------------+---------------------------+
| Variable_name | Value                     |
+---------------+---------------------------+
| Ssl_cipher    | DHE-RSA-AES128-GCM-SHA256 |
+---------------+---------------------------+
```

对于 **mysql** 客户端，另一种方法是使用 `STATUS` 或 `\s` 命令并检查 `SSL` 行：

```sql
mysql> \s
...
SSL: Not in use
...
```

或者：

```sql
mysql> \s
...
SSL: Cipher in use is DHE-RSA-AES128-GCM-SHA256
...
```

#### 配置加密连接为强制性

对于一些 MySQL 部署，使用加密连接可能不仅仅是可取的，而是强制性的（例如，为了满足监管要求）。本节讨论了使您能够执行此操作的配置设置。这些控制级别可用：

+   您可以配置服务器要求客户端使用加密连接进行连接。

+   您可以调用单独的客户端程序要求加密连接，即使服务器允许但不要求加密。

+   您可以配置单独的 MySQL 帐户仅在加密连接上可用。

要求客户端使用加密连接连接，请启用 `require_secure_transport` 系统变量。例如，在服务器的 `my.cnf` 文件中添加以下行：

```sql
[mysqld]
require_secure_transport=ON
```

或者，要在运行时设置并持久化该值，请使用以下语句：

```sql
SET PERSIST require_secure_transport=ON;
```

`SET PERSIST` 为运行中的 MySQL 实例设置一个值。它还保存该值，导致其在后续服务器重启时使用。参见 Section 15.7.6.1, “SET Syntax for Variable Assignment”。

启用 `require_secure_transport` 后，客户端连接到服务器需要使用某种形式的安全传输，服务器仅允许使用 SSL 的 TCP/IP 连接，或者使用套接字文件（在 Unix 上）或共享内存（在 Windows 上）的连接。服务器拒绝非安全连接尝试，这些尝试将因 `ER_SECURE_TRANSPORT_REQUIRED` 错误而失败。

要调用客户端程序，使其要求加密连接，无论服务器是否要求加密，请使用 `--ssl-mode` 选项值 `REQUIRED`、`VERIFY_CA` 或 `VERIFY_IDENTITY`。例如：

```sql
mysql --ssl-mode=REQUIRED
mysqldump --ssl-mode=VERIFY_CA
mysqladmin --ssl-mode=VERIFY_IDENTITY
```

要配置一个 MySQL 帐户仅在加密连接上可用，请在创建帐户的 `CREATE USER` 语句中包含一个 `REQUIRE` 子句，指定您需要的加密特性。例如，要求加密连接和使用有效的 X.509 证书，使用 `REQUIRE X509`：

```sql
CREATE USER 'jeffrey'@'localhost' REQUIRE X509;
```

有关`REQUIRE`子句的更多信息，请参见第 15.7.1.3 节，“CREATE USER Statement”。

要修改没有加密要求的现有帐户，请使用`ALTER USER`语句。
