# 8.3.2 加密连接 TLS 协议和密码

> 原文：[`dev.mysql.com/doc/refman/8.0/en/encrypted-connection-protocols-ciphers.html`](https://dev.mysql.com/doc/refman/8.0/en/encrypted-connection-protocols-ciphers.html)

MySQL 支持多个 TLS 协议和密码，并允许配置哪些协议和密码用于加密连接。还可以确定当前会话使用的协议和密码。

+   支持的 TLS 协议

+   移除对 TLSv1 和 TLSv1.1 协议的支持

+   连接 TLS 协议配置

+   连接密码配置

+   连接 TLS 协议协商

+   监视当前客户端会话的 TLS 协议和密码

#### 支持的 TLS 协议

允许连接到给定 MySQL 服务器实例的协议集受多个因素影响，如下所示：

MySQL 服务器版本

+   直到 MySQL 8.0.15，MySQL 支持 TLSv1、TLSv1.1 和 TLSv1.2 协议。

+   从 MySQL 8.0.16 开始，MySQL 还支持 TLSv1.3 协议。要使用 TLSv1.3，MySQL 服务器和客户端应用程序必须使用 OpenSSL 1.1.1 或更高版本进行编译。从 MySQL 8.0.18 开始，Group Replication 组件支持 TLSv1.3（有关详细信息，请参见第 20.6.2 节，“使用安全套接字层（SSL）保护组通信连接”）。

+   截至 MySQL 8.0.26 版本，TLSv1 和 TLSv1.1 协议已被弃用。这些协议版本较旧，分别于 1996 年和 2006 年发布，使用的算法较弱且过时。有关背景，请参考 IETF 备忘录[Deprecating TLSv1.0 and TLSv1.1](https://tools.ietf.org/id/draft-ietf-tls-oldversions-deprecate-02.html)。

+   截至 MySQL 8.0.28 版本，MySQL 不再支持 TLSv1 和 TLSv1.1 协议。从此版本开始，客户端不能使用协议设置为 TLSv1 或 TLSv1.1 进行 TLS/SSL 连接。有关更多详细信息，请参见移除对 TLSv1 和 TLSv1.1 协议的支持。

**表 8.13 MySQL 服务器 TLS 协议支持**

| MySQL 服务器版本 | 支持的 TLS 协议 |
| --- | --- |
| MySQL 8.0.15 及以下版本 | TLSv1、TLSv1.1、TLSv1.2 |
| MySQL 8.0.16 和 MySQL 8.0.17 | TLSv1、TLSv1.1、TLSv1.2、TLSv1.3（不包括 Group Replication） |
| MySQL 8.0.18 到 MySQL 8.0.25 | TLSv1、TLSv1.1、TLSv1.2、TLSv1.3（包括 Group Replication） |
| MySQL 8.0.26 和 MySQL 8.0.27 | TLSv1（已弃用）、TLSv1.1（已弃用）、TLSv1.2、TLSv1.3 |
| MySQL 8.0.28 及更高版本 | TLSv1.2、TLSv1.3 |

SSL 库

如果 SSL 库不支持特定协议，MySQL 也不支持，以下讨论中指定该协议的任何部分都不适用。特别要注意的是，要使用 TLSv1.3，MySQL 服务器和客户端应用程序都必须使用 OpenSSL 1.1.1 或更高版本进行编译。MySQL 服务器在启动时检查 OpenSSL 的版本，如果低于 1.1.1，则会从与 TLS 版本相关的服务器系统变量的默认值中删除 TLSv1.3（`tls_version`、`admin_tls_version` 和 `group_replication_recovery_tls_version`）。

MySQL 实例配置

可以在服务器端和客户端上配置允许的 TLS 协议，只包括受支持的 TLS 协议的子集。双方的配置必须至少包括一个共同的协议，否则连接尝试无法协商要使用的协议。有关详细信息，请参阅 连接 TLS 协议协商。

系统范围的主机配置

主机系统可能只允许某些 TLS 协议，这意味着即使 MySQL 本身允许它们，MySQL 连接也不能使用非允许的协议：

+   假设 MySQL 配置允许 TLSv1、TLSv1.1 和 TLSv1.2，但您的主机系统配置只允许使用 TLSv1.2 或更高版本的连接。在这种情况下，即使 MySQL 配置允许它们，您也无法建立使用 TLSv1 或 TLSv1.1 的 MySQL 连接，因为主机系统不允许���们。

+   如果 MySQL 配置允许 TLSv1、TLSv1.1 和 TLSv1.2，但您的主机系统配置只允许使用 TLSv1.3 或更高版本的连接，则根本无法建立 MySQL 连接，因为 MySQL 允许的任何协议都不被主机系统允许。

解决此问题的方法包括：

+   更改系统范围的主机配置以允许额外的 TLS 协议。请查阅操作系统文档以获取说明。例如，您的系统可能有一个包含以下行的 `/etc/ssl/openssl.cnf` 文件，以将 TLS 协议限制为 TLSv1.2 或更高版本：

    ```sql
    [system_default_sect]
    MinProtocol = TLSv1.2
    ```

    将值更改为较低的协议版本或`None`会使系统更加宽松。这种解决方法的缺点是允许较低（不太安全）的协议可能会产生不利的安全后果。

+   如果您无法或不愿更改主机系统的 TLS 配置，请将 MySQL 应用程序更改为使用主机系统允许的更高（更安全）的 TLS 协议。对于仅支持较低协议版本的旧版本 MySQL 可能无法实现此目标。例如，在 MySQL 5.6.46 之前，只支持 TLSv1 协议，因此即使客户端来自支持更高协议版本的新 MySQL 版本，尝试连接到早于 5.6.46 的服务器也会失败。在这种情况下，可能需要升级到支持额外 TLS 版本的 MySQL 版本。

#### 不再支持 TLSv1 和 TLSv1.1 协议。

从 MySQL 8.0.28 开始，不再支持 TLSv1 和 TLSv1.1 连接协议。这些协议从 MySQL 8.0.26 开始被弃用。有关背景，请参考 IETF 备忘录[Deprecating TLSv1.0 and TLSv1.1](https://tools.ietf.org/id/draft-ietf-tls-oldversions-deprecate-02.html)。建议使用更安全的 TLSv1.2 和 TLSv1.3 协议进行连接。TLSv1.3 要求 MySQL 服务器和客户端应用程序都使用 OpenSSL 1.1.1 进行编译。

不再支持 TLSv1 和 TLSv1.1 是因为这些协议版本过时，分别于 1996 年和 2006 年发布。使用的算法过于薄弱和过时。除非您使用非常旧的 MySQL Server 或连接器版本，否则不太可能使用 TLSv1.0 或 TLSv1.1 进行连接。MySQL 连接器和客户端默认选择可用的最高 TLS 版本。

在不支持 TLSv1 和 TLSv1.1 连接协议的版本中（从 MySQL 8.0.28 开始），包括 MySQL Shell 在内支持用于指定连接到 MySQL 服务器的 TLS 协议的`--tls-version`选项的客户端无法使用设置为 TLSv1 或 TLSv1.1 的协议进行 TLS/SSL 连接。如果客户端尝试使用这些协议进行连接，则对于 TCP 连接，连接将失败，并向客户端返回错误。对于套接字连接，如果`--ssl-mode`设置为`REQUIRED`，则连接将失败，否则将进行连接，但禁用 TLS/SSL。

从 MySQL 8.0.28 开始，服务器端的以下设置已更改：

+   服务器的`tls_version`和`admin_tls_version`系统变量的默认值不再包括 TLSv1 和 TLSv1.1。

+   Group Replication 系统变量`group_replication_recovery_tls_version`的默认值不再包括 TLSv1 和 TLSv1.1。

+   对于异步复制，副本不能将与源服务器的连接协议设置为 TLSv1 或 TLSv1.1（`CHANGE REPLICATION SOURCE TO` 语句的 `SOURCE_TLS_VERSION` 选项）。

在 TLSv1 和 TLSv1.1 连接协议被弃用的版本中（MySQL 8.0.26 和 MySQL 8.0.27），如果它们包含在 `tls_version` 或 `admin_tls_version` 系统变量的值中，并且客户端成功使用它们连接，则服务器会在错误日志中写入警告。如果在运行时设置弃用的协议并使用 `ALTER INSTANCE RELOAD TLS` 语句实现它们，也会返回警告。如果配置允许弃用的 TLS 协议，包括为与源服务器的连接指定 TLS 协议的副本和为分布式恢复连接指定 TLS 协议的 Group Replication 组成员，那么客户端不会发出警告。

更多信息，请参见 MySQL 8.0 是否支持 TLS 1.0 和 1.1？

#### 连接 TLS 协议配置

在服务器端，`tls_version` 系统变量的值确定 MySQL 服务器允许加密连接使用的 TLS 协议。`tls_version` 值适用于来自客户端的连接、常规源/副本复制连接（其中此服务器实例是源）、Group Replication 组通信连接以及 Group Replication 分布式恢复连接（其中此服务器实例是捐赠者）。管理连接接口配置类似，但使用 `admin_tls_version` 系统变量（参见第 7.1.12.2 节，“管理连接管理”）。此讨论也适用于 `admin_tls_version`。

`tls_version` 值是一个或多个逗号分隔的 TLS 协议版本列表，不区分大小写。默认情况下，此变量列出了编译 MySQL 和 MySQL 服务器发布所使用的 SSL 库支持的所有协议。因此，默认设置如表 8.14，“MySQL 服务器 TLS 协议默认设置”所示。

**表 8.14 MySQL 服务器 TLS 协议默认设置**

| MySQL 服务器发布版本 | `tls_version` 默认设置 |
| --- | --- |
| MySQL 8.0.15 及以下版本 | `TLSv1,TLSv1.1,TLSv1.2` |
| MySQL 8.0.16 和 MySQL 8.0.17 | `TLSv1,TLSv1.1,TLSv1.2,TLSv1.3 (使用 OpenSSL 1.1.1)` `TLSv1,TLSv1.1,TLSv1.2 (其他情况)` Group Replication 不支持 TLSv1.3 |
| MySQL 8.0.18 到 MySQL 8.0.25 | `TLSv1,TLSv1.1,TLSv1.2,TLSv1.3 (使用 OpenSSL 1.1.1)` `TLSv1,TLSv1.1,TLSv1.2 (其他情况)` Group Replication 支持 TLSv1.3 |
| MySQL 8.0.26 和 MySQL 8.0.27 | `TLSv1,TLSv1.1,TLSv1.2,TLSv1.3 (使用 OpenSSL 1.1.1)` `TLSv1,TLSv1.1,TLSv1.2 (其他情况)` `TLSv1 和 TLSv1.1 已被弃用` |
| MySQL 8.0.28 及以上版本 | `TLSv1.2,TLSv1.3` |

要在运行时确定 `tls_version` 的值，请使用以下语句：

```sql
mysql> SHOW GLOBAL VARIABLES LIKE 'tls_version';
+---------------+-----------------------+
| Variable_name | Value                 |
+---------------+-----------------------+
| tls_version   | TLSv1.2,TLSv1.3       |
+---------------+-----------------------+
```

要更改 `tls_version` 的值，请在服务器启动时设置。例如，要允许使用 TLSv1.2 或 TLSv1.3 协议的连接，但禁止使用较不安全的 TLSv1 和 TLSv1.1 协议的连接，请在服务器的 `my.cnf` 文件中使用以下行：

```sql
[mysqld]
tls_version=TLSv1.2,TLSv1.3
```

要更加严格并仅允许 TLSv1.3 连接，设置 `tls_version` 如下：

```sql
[mysqld]
tls_version=TLSv1.3
```

从 MySQL 8.0.16 开始，`tls_version` 可以在运行时更改。请参阅 Server-Side Runtime Configuration and Monitoring for Encrypted Connections。

在客户端，`--tls-version` 选项指定客户端程序允许连接到服务器的 TLS 协议版本。选项值的格式与之前描述的 `tls_version` 系统变量相同（一个或多个逗号分隔的协议版本列表）。

对于源/复制复制连接，其中此服务器实例是复制品，`CHANGE REPLICATION SOURCE TO` 语句（从 MySQL 8.0.23 开始）或 `CHANGE MASTER TO` 语句（MySQL 8.0.23 之前）的 `SOURCE_TLS_VERSION` | `MASTER_TLS_VERSION` 选项指定复制品允许连接到源的 TLS 协议版本。选项值的格式与之前描述的 `tls_version` 系统变量相同。参见 Section 19.3.1, “Setting Up Replication to Use Encrypted Connections”。

可以为`SOURCE_TLS_VERSION` | `MASTER_TLS_VERSION`指定的协议取决于 SSL 库。此选项独立于服务器`tls_version`值，并不受其影响。例如，作为副本的服务器可以配置`tls_version`设置为 TLSv1.3，以仅允许使用 TLSv1.3 的传入连接，但也可以配置`SOURCE_TLS_VERSION` | `MASTER_TLS_VERSION`设置为 TLSv1.2，以仅允许对源的传出副本连接使用 TLSv1.2。

对于 Group Replication 分布式恢复连接，其中此服务器实例是发起分布式恢复的加入成员（即客户端），`group_replication_recovery_tls_version`系统变量指定客户端允许的协议。同样，此选项独立于服务器`tls_version`值，并不受其影响，当此服务器实例是捐赠者时应用。Group Replication 服务器通常在其组成员身份的过程中既作为捐赠者又作为加入成员参与分布式恢复，因此应设置这两个系统变量。参见第 20.6.2 节，“使用安全套接字层（SSL）保护组通信连接”。

TLS 协议配置影响给定连接使用的协议，如连接 TLS 协议协商中所述。

允许的协议应该被选择，以免在列表中留下“漏洞”。例如，这些服务器配置数值没有漏洞：

```sql
tls_version=TLSv1,TLSv1.1,TLSv1.2,TLSv1.3
tls_version=TLSv1.1,TLSv1.2,TLSv1.3
tls_version=TLSv1.2,TLSv1.3
tls_version=TLSv1.3
```

这些数值有漏洞，不应使用：

```sql
tls_version=TLSv1,TLSv1.2       *(TLSv1.1 is missing)* tls_version=TLSv1.1,TLSv1.3     *(TLSv1.2 is missing)*
```

对于其他配置上下文，如客户端或副本，也适用于禁止漏洞。

除非你打算禁用加密连接，否则允许的协议列表不应为空。如果将 TLS 版本参数设置为空字符串，加密连接将无法建立：

+   `tls_version`：服务器不允许加密传入连接。

+   `--tls-version`：客户端不允许向服务器建立加密传出连接。

+   `SOURCE_TLS_VERSION` | `MASTER_TLS_VERSION`：副本不允许向源建立加密传出连接。

+   `group_replication_recovery_tls_version`：加入成员不允许与分布式恢复连接建立加密连接。

#### 连接密码配置

一组默认的密码适用于加密连接，可以通过显式配置允许的密码来覆盖。在连接建立过程中，连接的双方必须允许一些共同的密码，否则连接将失败。在双��都允许的密码中，SSL 库选择由提供的具有最高优先级的证书支持的密码。

要指定适用于使用 TLS 协议直至 TLSv1.2 的加密连接的密码或密码：

+   在服务器端设置`ssl_cipher`系统变量，并为客户端程序使用`--ssl-cipher`选项。

+   对于常规的源/副本复制连接，在这个服务器实例作为源时，设置`ssl_cipher`系统变量。当这个服务器实例作为副本时，使用`CHANGE REPLICATION SOURCE TO`语句（从 MySQL 8.0.23 开始）或`CHANGE MASTER TO`语句（MySQL 8.0.23 之前）的`SOURCE_SSL_CIPHER` | `MASTER_SSL_CIPHER`选项。参见 Section 19.3.1, “Setting Up Replication to Use Encrypted Connections”。

+   对于 Group Replication 组成员，对于 Group Replication 组通信连接以及 Group Replication 分布式恢复连接（其中这个服务器实例是捐赠者），设置`ssl_cipher`系统变量。对于 Group Replication 分布式恢复连接（其中这个服务器实例是加入成员），使用`group_replication_recovery_ssl_cipher`系统变量。参见 Section 20.6.2, “Securing Group Communication Connections with Secure Socket Layer (SSL)”")。

对于使用 TLSv1.3 的加密连接，OpenSSL 1.1.1 及更高版本支持以下密码套件，默认情况下启用前三个：

```sql
TLS_AES_128_GCM_SHA256
TLS_AES_256_GCM_SHA384
TLS_CHACHA20_POLY1305_SHA256
TLS_AES_128_CCM_SHA256
```

注意

在 MySQL 8.0.35 之前，`TLS_AES_128_CCM_8_SHA256`支持与服务器系统变量`--tls-ciphersuites`或`--admin-tls-ciphersuites`一起使用。如果为 MySQL 8.0.35 及更高版本配置`TLS_AES_128_CCM_8_SHA256`，将生成一个弃用警告。

要显式配置允许的 TLSv1.3 密码套件，请设置以下参数。在每种情况下，配置值是一个零个或多个以冒号分隔的密码套件名称列表。

+   在服务器端，使用`tls_ciphersuites`系统变量。如果未设置此变量，则其默认值为`NULL`，这意味着服务器允许默认一组密码套件。如果将变量设置为空字符串，则不启用任何密码套件，无法建立加密连接。

+   在客户端端，使用`--tls-ciphersuites`选项。如果未设置此选项，则客户端允许默认一组密码套件。如果将选项设置为空字符串，则不启用任何密码套件，无法建立加密连接。

+   对于常规的源/副本复制连接，在此服务器实例作为源时，请使用`tls_ciphersuites`系统变量。在此服务器实例作为副本时，请使用`SOURCE_TLS_CIPHERSUITES` | `MASTER_TLS_CIPHERSUITES`选项，用于`CHANGE REPLICATION SOURCE TO`语句（从 MySQL 8.0.23 开始）或`CHANGE MASTER TO`语句（在 MySQL 8.0.23 之前）。参见 Section 19.3.1, “Setting Up Replication to Use Encrypted Connections”。

+   对于 Group Replication 组成员，在 Group Replication 组通信连接以及在此服务器实例作为捐赠者时的 Group Replication 分布式恢复连接中，请使用`tls_ciphersuites`系统变量。对于此服务器实例作为加入成员时的 Group Replication 分布式恢复连接，请使用`group_replication_recovery_tls_ciphersuites`系统变量。参见 Section 20.6.2, “Securing Group Communication Connections with Secure Socket Layer (SSL)”")。

注意

MySQL 8.0.16 版本开始支持密码套件，但要求 MySQL 服务器和客户端应用程序都使用 OpenSSL 1.1.1 或更高版本进行编译。

在 MySQL 8.0.16 至 8.0.18 中，`group_replication_recovery_tls_ciphersuites`系统变量和`CHANGE REPLICATION SOURCE TO`语句（从 MySQL 8.0.23 开始）或`CHANGE MASTER TO`语句（在 MySQL 8.0.23 之前）中的`SOURCE_TLS_CIPHERSUITES` | `MASTER_TLS_CIPHERSUITES`选项不可用。在这些版本中，如果用于源/副本复制连接的 TLSv1.3，或在分布式恢复的 Group Replication 中使用 TLSv1.3（从 MySQL 8.0.18 开始支持），则复制源或 Group Replication 捐赠服务器必须允许至少启用默认情况下启用的一个 TLSv1.3 密码套件。从 MySQL 8.0.19 开始，您可以使用选项配置客户端支持任何选择的密码套件，包括仅使用非默认密码套件（如果需要）。

给定密码可能仅适用于特定的 TLS 协议，这会影响 TLS 协议协商过程。请参阅连接 TLS 协议协商。

要确定给定服务器支持哪些密码，检查`Ssl_cipher_list`状态变量的会话值：

```sql
SHOW SESSION STATUS LIKE 'Ssl_cipher_list';
```

`Ssl_cipher_list`状态变量列出了可能的 SSL 密码（非 SSL 连接为空）。如果 MySQL 支持 TLSv1.3，则该值包括可能的 TLSv1.3 密码套件。

注意

ECDSA 密码只能与使用 ECDSA 进行数字签名的 SSL 证书结合使用，并且不能与使用 RSA 的证书一起使用。MySQL 服务器的 SSL 证书自动生成过程不生成 ECDSA 签名证书，它只生成 RSA 签名证书。除非您有可用的 ECDSA 证书，请勿选择 ECDSA 密码。

对于使用 TLS.v1.3 的加密连接，MySQL 使用 SSL 库的默认密码套件列表。

对于使用 TLS 协议直至 TLSv1.2 的加密连接，MySQL 将以下默认密码列表传递给 SSL 库。

```sql
ECDHE-ECDSA-AES128-GCM-SHA256
ECDHE-ECDSA-AES256-GCM-SHA384
ECDHE-RSA-AES128-GCM-SHA256
ECDHE-RSA-AES256-GCM-SHA384
ECDHE-ECDSA-CHACHA20-POLY1305
ECDHE-RSA-CHACHA20-POLY1305
ECDHE-ECDSA-AES256-CCM
ECDHE-ECDSA-AES128-CCM
DHE-RSA-AES128-GCM-SHA256
DHE-RSA-AES256-GCM-SHA384
DHE-RSA-AES256-CCM
DHE-RSA-AES128-CCM
DHE-RSA-CHACHA20-POLY1305
```

这些密码限制已经生效：

+   从 MySQL 8.0.35 开始，以下密码已被弃用，并在与服务器系统变量`--ssl-cipher`和`--admin-ssl-cipher`一起使用时产生警告：

    ```sql
    ECDHE-ECDSA-AES128-SHA256
    ECDHE-RSA-AES128-SHA256
    ECDHE-ECDSA-AES256-SHA384
    ECDHE-RSA-AES256-SHA384
    DHE-DSS-AES128-GCM-SHA256
    DHE-RSA-AES128-SHA256
    DHE-DSS-AES128-SHA256
    DHE-DSS-AES256-GCM-SHA384
    DHE-RSA-AES256-SHA256
    DHE-DSS-AES256-SHA256
    ECDHE-RSA-AES128-SHA
    ECDHE-ECDSA-AES128-SHA
    ECDHE-RSA-AES256-SHA
    ECDHE-ECDSA-AES256-SHA
    DHE-DSS-AES128-SHA
    DHE-RSA-AES128-SHA
    TLS_DHE_DSS_WITH_AES_256_CBC_SHA
    DHE-RSA-AES256-SHA
    AES128-GCM-SHA256
    DH-DSS-AES128-GCM-SHA256
    ECDH-ECDSA-AES128-GCM-SHA256
    AES256-GCM-SHA384
    DH-DSS-AES256-GCM-SHA384
    ECDH-ECDSA-AES256-GCM-SHA384
    AES128-SHA256
    DH-DSS-AES128-SHA256
    ECDH-ECDSA-AES128-SHA256
    AES256-SHA256
    DH-DSS-AES256-SHA256
    ECDH-ECDSA-AES256-SHA384
    AES128-SHA
    DH-DSS-AES128-SHA
    ECDH-ECDSA-AES128-SHA
    AES256-SHA
    DH-DSS-AES256-SHA
    ECDH-ECDSA-AES256-SHA
    DH-RSA-AES128-GCM-SHA256
    ECDH-RSA-AES128-GCM-SHA256
    DH-RSA-AES256-GCM-SHA384
    ECDH-RSA-AES256-GCM-SHA384
    DH-RSA-AES128-SHA256
    ECDH-RSA-AES128-SHA256
    DH-RSA-AES256-SHA256
    ECDH-RSA-AES256-SHA384
    ECDHE-RSA-AES128-SHA
    ECDHE-ECDSA-AES128-SHA
    ECDHE-RSA-AES256-SHA
    ECDHE-ECDSA-AES256-SHA
    DHE-DSS-AES128-SHA
    DHE-RSA-AES128-SHA
    TLS_DHE_DSS_WITH_AES_256_CBC_SHA
    DHE-RSA-AES256-SHA
    AES128-SHA
    DH-DSS-AES128-SHA
    ECDH-ECDSA-AES128-SHA
    AES256-SHA
    DH-DSS-AES256-SHA
    ECDH-ECDSA-AES256-SHA
    DH-RSA-AES128-SHA
    ECDH-RSA-AES128-SHA
    DH-RSA-AES256-SHA
    ECDH-RSA-AES256-SHA
    DES-CBC3-SHA
    ```

+   以下密码已被永久限制：

    ```sql
    !DHE-DSS-DES-CBC3-SHA
    !DHE-RSA-DES-CBC3-SHA
    !ECDH-RSA-DES-CBC3-SHA
    !ECDH-ECDSA-DES-CBC3-SHA
    !ECDHE-RSA-DES-CBC3-SHA
    !ECDHE-ECDSA-DES-CBC3-SHA
    ```

+   以下密码类别已被永久限制：

    ```sql
    !aNULL
    !eNULL
    !EXPORT
    !LOW
    !MD5
    !DES
    !RC2
    !RC4
    !PSK
    !SSLv3
    ```

如果服务器启动时使用`ssl_cert`系统变量设置为使用任何前述受限密码或密码类别的证书，则服务器将以禁用加密连接的支持启动。

#### 连接 TLS 协议协商

在 MySQL 中，连接尝试会协商双方都支持的最高 TLS 协议版本，该版本在双方都支持的协议兼容加密密码上可用。协商过程取决于诸如用于编译服务器和客户端的 SSL 库、TLS 协议和加密密码配置以及使用的密钥大小等因素：

+   要使连接尝试成功，服务器和客户端 TLS 协议配置必须允许一些共同的协议。

+   类似地，服务器和客户端加密密码配置必须允许一些共同的密码。给定的密码可能仅适用于特定的 TLS 协议，因此除非还有兼容的密码，否则在协商过程中不会选择协议。

+   如果 TLSv1.3 可用，则尽可能使用。 （这意味着服务器和客户端配置都必须允许 TLSv1.3，并且两者还必须允许一些 TLSv1.3 兼容的加密密码。）否则，MySQL 将继续使用可用协议列表，如果可能的话使用 TLSv1.2，依此类推。协商从更安全的协议到不太安全的协议进行。协商顺序与配置协议的顺序无关。例如，无论`tls_version`的值是`TLSv1,TLSv1.1,TLSv1.2,TLSv1.3`还是`TLSv1.3,TLSv1.2,TLSv1.1,TLSv1`，协商顺序都是相同的。

+   TLSv1.2 与所有密钥大小为 512 位或更小的密码不兼容。要使用此协议与此类密钥，请在服务器端设置`ssl_cipher`系统变量，或使用`--ssl-cipher`客户端选项显式指定密码名称：

    ```sql
    AES128-SHA
    AES128-SHA256
    AES256-SHA
    AES256-SHA256
    CAMELLIA128-SHA
    CAMELLIA256-SHA
    DES-CBC3-SHA
    DHE-RSA-AES256-SHA
    RC4-MD5
    RC4-SHA
    SEED-SHA
    ```

+   为了更好的安全性，请使用至少 2048 位的 RSA 密钥大小的证书。

如果服务器和客户端没有共同的允许协议，以及共同的协议兼容密码，服务器将终止连接请求。例如：

+   如果服务器配置为`tls_version=TLSv1.1,TLSv1.2`：

    +   对于使用`--tls-version=TLSv1`调用的客户端以及仅支持 TLSv1 的旧客户端，连接尝试将失败。

    +   类似地，为配置为`MASTER_TLS_VERSION = 'TLSv1'`的复制品以及仅支持 TLSv1 的旧复制品，连接尝试将失败。

+   如果服务器配置为`tls_version=TLSv1`或是仅支持 TLSv1 的旧服务器：

    +   对于使用`--tls-version=TLSv1.1,TLSv1.2`调用的客户端，连接尝试将失败。

    +   同样，对于配置为`MASTER_TLS_VERSION = 'TLSv1.1,TLSv1.2'`的副本，连接尝试将失败。

MySQL 允许指定要支持的协议列表。此列表直接传递给底层 SSL 库，最终由该库决定实际启用来自提供的列表的哪些协议。有关 SSL 库处理此操作的信息，请参考 MySQL 源代码和 OpenSSL [`SSL_CTX_new()`](https://www.openssl.org/docs/man1.1.0/ssl/SSL_CTX_new.html) 文档。

#### 监控当前客户端会话的 TLS 协议和密码

要确定当前客户端会话使用的加密 TLS 协议和密码，检查`Ssl_version`和`Ssl_cipher`状态变量的会话值：

```sql
mysql> SELECT * FROM performance_schema.session_status
       WHERE VARIABLE_NAME IN ('Ssl_version','Ssl_cipher');
+---------------+---------------------------+
| VARIABLE_NAME | VARIABLE_VALUE            |
+---------------+---------------------------+
| Ssl_cipher    | DHE-RSA-AES128-GCM-SHA256 |
| Ssl_version   | TLSv1.2                   |
+---------------+---------------------------+
```

如果连接未加密，则两个变量的值为空。
