# 6.2.5 使用类似 URI 字符串或键值对连接到服务器

> 原文：[`dev.mysql.com/doc/refman/8.0/en/connecting-using-uri-or-key-value-pairs.html`](https://dev.mysql.com/doc/refman/8.0/en/connecting-using-uri-or-key-value-pairs.html)

本节描述了如何使用类似 URI 的连接字符串或键值对来指定如何建立与 MySQL 服务器的连接，适用于诸如 MySQL Shell 的客户端。有关使用命令行选项建立连接的信息，适用于诸如**mysql**或**mysqldump**等客户端，请参阅第 6.2.4 节，“使用命令选项连接到 MySQL 服务器”。如果无法连接，请参阅第 8.2.22 节，“连接到 MySQL 时出现问题的故障排除”。

注意

“类似 URI”一词表示连接字符串语法类似但不完全相同于[RFC 3986](https://tools.ietf.org/html/rfc3986)定义的 URI（统一资源标识符）语法。

以下 MySQL 客户端支持使用类似 URI 连接字符串或键值对连接到 MySQL 服务器：

+   MySQL Shell

+   实现 X DevAPI 的 MySQL 连接器

本节记录了所有有效的类似 URI 字符串和键值对连接参数，其中许多与命令行选项指定的参数类似：

+   使用类似 URI 字符串指定的参数使用如`myuser@example.com:3306/main-schema`的语法。有关完整语法，请参阅使用类似 URI 连接字符串连接。

+   使用键值对指定的参数使用如`{user:'myuser', host:'example.com', port:3306, schema:'main-schema'}`的语法。有关完整语法，请参阅使用键值对连接。

连接参数不区分大小写。每个参数只能��定一次。如果一个参数指定多次，将会出现错误。

本节涵盖以下主题：

+   基本连接参数

+   附加连接参数

+   使用类似 URI 连接字符串连接

+   使用键值对连接

#### 基本连接参数

以下讨论描述了指定连接到 MySQL 时可用的参数。这些参数可以使用符合基本 URI 类似语法的字符串（请参阅使用类似 URI 的连接字符串进行连接）或作为键值对（请参阅使用键值对进行连接）来提供。

+   *`scheme`*: 要使用的传输协议。对于 X 协议连接，请使用 `mysqlx`，对于经典 MySQL 协议连接，请使用 `mysql`。如果未指定协议，服务器将尝试猜测协议。支持 DNS SRV 的连接器可以使用 `mysqlx+srv` 方案（请参阅使用 DNS SRV 记录进行连接）。

+   *`user`*: 提供身份验证过程中使用的 MySQL 用户账户。

+   *`password`*: 用于身份验证过程的密码。

    警告

    在连接规范中明确指定密码是不安全且不推荐的。稍后的讨论将展示如何导致出现密码的交互提示。

+   *`host`*: 运行服务器实例的主机。值可以是主机名、IPv4 地址或 IPv6 地址。如果未指定主机，则默认为 `localhost`。

+   *`port`*: 目标 MySQL 服务器侦听连接的 TCP/IP 网络端口。如果未指定端口，则 X 协议连接的默认端口为 33060，经典 MySQL 协议连接的默认端口为 3306。

+   *`socket`*: Unix 套接字文件路径或 Windows 命名管道的名称。值为本地文件路径。在类似 URI 的字符串中，它们必须进行编码，可以使用百分号编码或用括号括起路径。括号消除了需要对字符进行百分号编码，例如 `/` 目录分隔符字符。例如，要使用 Unix 套接字 `/tmp/mysql.sock` 连接为 `root@localhost`，请使用百分号编码指定路径为 `root@localhost?socket=%2Ftmp%2Fmysql.sock`，或使用括号指定路径为 `root@localhost?socket=(/tmp/mysql.sock)`。

+   *`schema`*: 连接的默认数据库。如果未指定数据库，则连接没有默认数据库。

在 Unix 上处理 `localhost` 取决于传输协议的类型。使用经典 MySQL 协议的连接将 `localhost` 与其他 MySQL 客户端一样处理，这意味着假定 `localhost` 用于基于套接字的连接。对于使用 X 协议的连接，`localhost` 的行为不同，它被假定代表回环地址，例如，IPv4 地址 127.0.0.1。

#### 附加连接参数

您可以指定连接的选项，可以作为 URI 类似字符串中的属性通过附加 `?*`attribute=value`*`，或作为键值对。以下选项可用：

+   `ssl-mode`: 连接的期望安全状态。以下模式是允许的：

    +   `DISABLED`

    +   `PREFERRED`

    +   `REQUIRED`

    +   `VERIFY_CA`

    +   `VERIFY_IDENTITY`

    重要

    `VERIFY_CA` 和 `VERIFY_IDENTITY` 比默认的`PREFERRED`更好，因为它们有助于防止中间人攻击。

    有关这些模式的信息，请参见加密连接的命令选项中的`--ssl-mode`选项描述。

+   `ssl-ca`: PEM 格式的 X.509 证书颁发机构文件路径。

+   `ssl-capath`: 包含 PEM 格式的 X.509 证书颁发机构文件的目录路径。

+   `ssl-cert`: PEM 格式的 X.509 证书文件路径。

+   `ssl-cipher`: 用于使用 TLS 协议进行连接的加密密码，直到 TLSv1.2。

+   `ssl-crl`: PEM 格式的包含证书吊销列表的文件路径。

+   `ssl-crlpath`: PEM 格式的包含证书吊销列表文件的目录路径。

+   `ssl-key`: PEM 格式的 X.509 密钥文件路径。

+   `tls-version`: 经典 MySQL 协议加密连接允许的 TLS 协议。此选项仅由 MySQL Shell 支持。`tls-version`（单数）的值是逗号分隔的列表，例如`TLSv1.2,TLSv1.3`。详情请参见第 8.3.2 节，“加密连接 TLS 协议和密码”。此选项取决于未将`ssl-mode`选项设置为`DISABLED`。

+   `tls-versions`: 加密 X 协议连接的允许 TLS 协议。`tls-versions`（复数）的值是一个数组，例如`[TLSv1.2,TLSv1.3]`。详情请参见第 8.3.2 节，“加密连接 TLS 协议和密码”。此选项取决于未将`ssl-mode`选项设置为`DISABLED`。

+   `tls-ciphersuites`: 允许的 TLS 密码套件。`tls-ciphersuites`的值是列出在[TLS 密码套件](https://www.iana.org/assignments/tls-parameters/tls-parameters.xhtml#tls-parameters-4)中的 IANA 密码套件名称的列表。详情请参见第 8.3.2 节，“加密连接 TLS 协议和密码”。此选项取决于未将`ssl-mode`选项设置为`DISABLED`。

+   `auth-method`: 用于连接的身份验证方法。默认值为`AUTO`，表示服务器会尝试猜测。以下方法是允许的：

    +   `AUTO`

    +   `MYSQL41`

    +   `SHA256_MEMORY`

    +   `FROM_CAPABILITIES`

    +   `FALLBACK`

    +   `PLAIN`

    对于 X 协议连接，任何配置的`auth-method`都会被覆盖为以下身份验证方法序列：`MYSQL41`、`SHA256_MEMORY`、`PLAIN`。

+   `get-server-public-key`：从服务器请求所需的用于 RSA 密钥对密码交换的公钥。在使用 SSL 模式`DISABLED`连接到 MySQL 8.0 服务器时使用经典 MySQL 协议。在这种情况下，您必须指定协议。例如：

    ```sql
    mysql://user@localhost:3306?get-server-public-key=true
    ```

    此选项适用于使用`caching_sha2_password`认证插件进行身份验证的客户端。对于该插件，除非请求，否则服务器不会发送公钥。对于不使用该插件进行身份验证的帐户，此选项将被忽略。如果不使用基于 RSA 的密码交换，例如客户端使用安全连接连接到服务器时，此选项也将被忽略。

    如果`server-public-key-path=*`file_name`*`给出并指定有效的公钥文件，则优先于`get-server-public-key`。

    关于`caching_sha2_password`插件的信息，请参见 Section 8.4.1.2，“缓存 SHA-2 可插拔认证”。

+   `server-public-key-path`：以 PEM 格式存储的文件路径名，其中包含服务器所需的用于 RSA 密钥对密码交换的客户端端公钥的副本。在使用 SSL 模式`DISABLED`连接到 MySQL 8.0 服务器时使用经典 MySQL 协议。

    此选项适用于使用`sha256_password`或`caching_sha2_password`认证插件进行身份验证的客户端。对于不使用这些插件进行身份验证的帐户，此选项将被忽略。如果不使用基于 RSA 的密码交换，例如客户端使用安全连接连接到服务器时，此选项也将被忽略。

    如果`server-public-key-path=*`file_name`*`给出并指定有效的公钥文件，则优先于`get-server-public-key`。

    关于`sha256_password`和`caching_sha2_password`插件的信息，请参见 Section 8.4.1.3，“SHA-256 可插拔认证”，以及 Section 8.4.1.2，“缓存 SHA-2 可插拔认证”。

+   `ssh`：连接到 SSH 服务器以使用 SSH 隧道访问 MySQL 服务器实例的 URI。URI 格式为`[user@]host[:port]`。使用`uri`选项指定目标 MySQL 服务器实例的 URI。有关从 MySQL Shell 进行 SSH 隧道连接的信息，请参见使用 SSH 隧道。

+   `uri`：通过 SSH 隧道从由`ssh`选项指定的服务器访问的 MySQL 服务器实例的 URI。URI 格式为`[scheme://][user@]host[:port]`。不要使用基本连接参数（`scheme`，`user`，`host`，`port`）来指定用于 SSH 隧道连接的 MySQL 服务器连接，只需使用`uri`选项。

+   `ssh-password`：连接到 SSH 服务器的密码。

    警告

    在连接规范中指定明确的密码是不安全且不推荐的做法。当需要密码时，MySQL Shell 会交互式地提示输入密码。

+   `ssh-config-file`：用于连接到 SSH 服务器的 SSH 配置文件。您可以使用 MySQL Shell 配置选项`ssh.configFile`来设置自定义文件作为默认文件，如果未指定此选项。如果未设置`ssh.configFile`，则默认为标准 SSH 配置文件`~/.ssh/config`。

+   `ssh-identity-file`：用于连接到 SSH 服务器的身份文件。如果未指定此选项，则默认使用 SSH 代理中配置的任何身份文件（如果使用），或者在 SSH 配置文件中配置的身份文件，或者 SSH 配置文件夹中的标准私钥文件（`~/.ssh/id_rsa`）。

+   `ssh-identity-pass`：由`ssh-identity-file`选项指定的身份文件的密码。

    警告

    在连接规范中指定明确的密码是不安全且不推荐的做法。当需要密码时，MySQL Shell 会交互式地提示输入密码。

+   `connect-timeout`：用于配置客户端（如 MySQL Shell）等待连接到无响应的 MySQL 服务器的秒数。

+   `compression`：此选项请求或禁用连接的压缩。在 MySQL 8.0.19 之前，它仅适用于经典的 MySQL 协议连接，而从 MySQL 8.0.20 开始，它也适用于 X 协议连接。

    +   在 MySQL 8.0.19 之前，此选项的值为`true`（或 1），表示启用压缩，而默认值`false`（或 0）表示禁用压缩。

    +   从 MySQL 8.0.20 开始，此选项的值为`required`，表示请求压缩并在服务器不支持时失败；`preferred`表示请求压缩并在无法时回退到未压缩连接；`disabled`表示请求未压缩连接并在服务器不允许时失败。`preferred`是 X 协议连接的默认值，而`disabled`是经典的 MySQL 协议连接的默认值。有关 X 插件连接压缩控制的信息，请参阅第 22.5.5 节，“X 插件连接压缩”。

    请注意，不同的 MySQL 客户端以不同的方式实现对连接压缩的支持。请查阅您的客户端文档以获取详细信息。

+   `compression-algorithms` 和 `compression-level`：这些选项在 MySQL Shell 8.0.20 及更高版本中可用，用于更好地控制连接压缩。您可以指定它们以选择连接所使用的压缩算法，以及与该算法一起使用的数字压缩级别。您还可以使用 `compression-algorithms` 代替 `compression` 来请求连接的压缩。有关 MySQL Shell 连接压缩控制的信息，请参阅 使用压缩连接。

+   `connection-attributes`：控制应用程序在连接时传递给服务器的键值对。有关连接属性的一般信息，请参阅 第 29.12.9 节，“性能模式连接属性表”。客户端通常定义一组默认属性，可以禁用或启用。例如：

    ```sql
    mysqlx://user@host?connection-attributes
    mysqlx://user@host?connection-attributes=true
    mysqlx://user@host?connection-attributes=false
    ```

    默认行为是发送默认属性集。应用程序可以指定要传递的除默认属性之外的属性。您可以将额外的连接属性指定为连接字符串中的 `connection-attributes` 参数。`connection-attributes` 参数值必须为空（与指定 `true` 相同），是一个布尔值（`true` 或 `false` 用于启用或禁用默认属性集），或者是一个由逗号分隔的零个或多个 `key=value` 指定器的列表（用于发送除默认属性集之外的属性）。在列表中，缺少的键值将被视为空字符串。更多示例：

    ```sql
    mysqlx://user@host?connection-attributes=[attr1=val1,attr2,attr3=]
    mysqlx://user@host?connection-attributes=[]
    ```

    应用程序定义的属性名称不能以 `_` 开头，因为这些名称保留用于内部属性。

#### 使用类似 URI 的连接字符串进行连接

您可以使用类似 URI 的字符串指定与 MySQL Server 的连接。这种字符串可以与 MySQL Shell 的 `--uri` 命令选项、MySQL Shell 的 `\connect` 命令以及实现 X DevAPI 的 MySQL 连接器一起使用。

注意

“类似 URI” 一词表示连接字符串语法类似于但不完全相同于由 [RFC 3986](https://tools.ietf.org/html/rfc3986) 定义的 URI（统一资源标识符）语法。

类似 URI 的连接字符串具有以下语法：

```sql
[*scheme*://][*user*[:[*password*]]@]*host*[:*port*][/*schema*]?*attribute1=value1&attribute2=value2...*
```

重要

URI-like 字符串的元素中必须使用百分号编码来处理保留字符。例如，如果您指定包含 `@` 字符的字符串，则该字符必须替换为 `%40`。如果在 IPv6 地址中包含区域 ID，则用作分隔符的 `%` 字符必须替换为 `%25`。

您可以在 [基本连接参数 中描述的 URI-like 连接字符串中使用的参数。

MySQL Shell 的`shell.parseUri()`和`shell.unparseUri()`方法可用于拆解和组装类似 URI 的连接字符串。给定类似 URI 的连接字符串，`shell.parseUri()`返回一个包含字符串中每个元素的字典。`shell.unparseUri()`将 URI 组件和连接选项的字典转换为用于连接到 MySQL 的有效类似 URI 连接字符串，可在 MySQL Shell 中使用，也可由实现 X DevAPI 的 MySQL 连接器使用。

如果在类似 URI 的字符串中未指定密码（建议不指定），交互式客户端会提示输入密码。以下示例展示了如何指定类似 URI 的字符串，其中包含用户名*`user_name`*。在每种情况下，都会提示输入密码。

+   连接到监听端口 33065 的本地服务器实例的 X 协议连接。

    ```sql
    mysqlx://*user_name*@localhost:33065
    ```

+   到监听端口 3333 的本地服务器实例的经典 MySQL 协议连接。

    ```sql
    mysql://*user_name*@localhost:3333
    ```

+   通过主机名、IPv4 地址和 IPv6 地址连接到远程服务器实例的 X 协议连接。

    ```sql
    mysqlx://*user_name*@server.example.com/
    mysqlx://*user_name*@198.51.100.14:123
    mysqlx://*user_name*@[2001:db8:85a3:8d3:1319:8a2e:370:7348]
    ```

+   使用套接字进行 X 协议连接，路径可以使用百分号编码或括号提供。

    ```sql
    mysqlx://*user_name*@/*path*%2F*to*%2F*socket.sock*
    mysqlx://*user_name*@(*/path/to/socket.sock*)
    ```

+   可以指定可选路径，表示数据库。

    ```sql
    # use 'world' as the default database
    mysqlx://*user_name*@198.51.100.1/world

    # use 'world_x' as the default database, encoding _ as %5F
    mysqlx://*user_name*@198.51.100.2:33060/world%5Fx
    ```

+   可以指定可选查询，由每个以`*`key`*=*`value`*`对或单个*`key`*给出的值组成。要指定多个值，请用`,`字符分隔它们。可以混合使用`*`key`*=*`value`*`和*`key`*值。值可以是列表类型，列表值按出现顺序排序。字符串必须是百分号编码或用括号括起来。以下是等效的。

    ```sql
    ssluser@127.0.0.1?ssl-ca=%2Froot%2Fclientcert%2Fca-cert.pem\
    &ssl-cert=%2Froot%2Fclientcert%2Fclient-cert.pem\
    &ssl-key=%2Froot%2Fclientcert%2Fclient-key

    ssluser@127.0.0.1?ssl-ca=(/root/clientcert/ca-cert.pem)\
    &ssl-cert=(/root/clientcert/client-cert.pem)\
    &ssl-key=(/root/clientcert/client-key)
    ```

+   指定用于加密连接的 TLS 版本和密码套件：

    ```sql
    mysql://*user_name*@198.51.100.2:3306/world%5Fx?\
    tls-versions=[TLSv1.2,TLSv1.3]&tls-ciphersuites=[TLS_DHE_PSK_WITH_AES_128_\
    GCM_SHA256, TLS_CHACHA20_POLY1305_SHA256]
    ```

前面的示例假定连接需要密码。对于交互式客户端，在登录提示符处会要求输入指定用户的密码。如果用户帐户没有密码（这是不安全且不建议的），或者正在使用套接字对等凭据认证（例如，使用 Unix 套接字连接），则必须在连接字符串中明确指定不提供密码且不需要密码提示。为此，在字符串中在*`user_name`*后面放置一个`:`，但不要在其后指定密码。例如：

```sql
mysqlx://*user_name*:@localhost
```

#### 使用键值对进行连接

在 MySQL Shell 和一些实现 X DevAPI 的 MySQL 连接器中，您可以使用键值对指定与 MySQL 服务器的连接，这些键值对以自然语言构造的形式提供给实现。例如，您可以在 JavaScript 中以 JSON 对象的形式或在 Python 中以字典的形式提供键值对作为连接参数。无论以何种方式提供键值对，概念都是相同的：本节中描述的键可以分配用于指定连接的值。您可以在 MySQL Shell 的`shell.connect()`方法或 InnoDB Cluster 的`dba.createCluster()`方法中使用键值对指定连接，并且在一些实现 X DevAPI 的 MySQL 连接器中也可以这样做。

通常，键值对用`{`和`}`字符括起来，`,`字符用作键值对之间的分隔符。`:`字符用于键和值之间的分隔，字符串必须被定界（例如，使用`'`字符）。与类似 URI 的连接字符串不同，不需要对字符串进行百分比编码。

以键值对形式指定的连接具有以下格式：

```sql
{ *key*: *value*, *key*: *value*, ...}
```

您可以用作连接键的参数在基本连接参数中进行了描述。

如果在键值对中未指定密码（建议不要指定），交互式客户端会提示输入密码。以下示例展示了如何使用用户名`'*`user_name`*'`指定连接。在每种情况下，都会提示输入密码。

+   在本地服务器实例上监听端口 33065 的 X 协议连接。

    ```sql
    {user:'*user_name*', host:'localhost', port:33065}
    ```

+   在本地服务器实例上监听端口 3333 的经典 MySQL 协议连接。

    ```sql
    {user:'*user_name*', host:'localhost', port:3333}
    ```

+   使用主机名、IPv4 地址和 IPv6 地址的远程服务器实例的 X 协议连接。

    ```sql
    {user:'*user_name*', host:'server.example.com'}
    {user:'*user_name*', host:198.51.100.14:123}
    {user:'*user_name*', host:[2001:db8:85a3:8d3:1319:8a2e:370:7348]}
    ```

+   使用套接字的 X 协议连接。

    ```sql
    {user:'*user_name*', socket:'*/path/to/socket/file*'}
    ```

+   可以指定一个可选的模式，代表一个数据库。

    ```sql
    {user:'*user_name*', host:'localhost', schema:'world'}
    ```

前面的示例假定连接需要密码。对于交互式客户端，在登录提示符处会请求指定用户的密码。如果用户帐户没有密码（这是不安全且不推荐的），或者正在使用套接字对等凭据验证（例如，使用 Unix 套接字连接），则必须明确指定不提供密码且不需要密码提示。为此，在`password`键后面使用`''`提供一个空字符串。例如：

```sql
{user:'*user_name*', password:'', host:'localhost'}
```
