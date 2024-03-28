# 6.2.3 连接到服务器的命令选项

> 原文：[`dev.mysql.com/doc/refman/8.0/en/connection-options.html`](https://dev.mysql.com/doc/refman/8.0/en/connection-options.html)

本节描述了大多数 MySQL 客户端程序支持的选项，用于控制客户端程序如何与服务器建立连接，连接是否加密以及连接是否压缩。这些选项可以在命令行或选项文件中指定。

+   连接建立的命令选项

+   加密连接的命令选项

+   连接压缩的命令选项

#### 连接建立的命令选项

本节描述了控制客户端程序如何与服务器建立连接的选项。有关更多信息和示例，请参见第 6.2.4 节，“使用命令选项连接到 MySQL 服务器”。

**表 6.4 连接建立选项摘要**

| 选项名称 | 描述 | 引入版本 |
| --- | --- | --- |
| --default-auth | 要使用的身份验证插件 |  |
| --host | MySQL 服务器所在的主机 |  |
| --password | 连接到服务器时要使用的密码 |  |
| --password1 | 连接到服务器时要使用的第一个多因素身份验证密码 | 8.0.27 |
| --password2 | 连接到服务器时要使用的第二个多因素身份验证密码 | 8.0.27 |
| --password3 | 连接到服务器时要使用的第三个多因素身份验证密码 | 8.0.27 |
| --pipe | 使用命名管道连接到服务器（仅限 Windows） |  |
| --plugin-dir | 安装插件的目录 |  |
| --port | 连接的 TCP/IP 端口号 |  |
| --protocol | 要使用的传输协议 |  |
| --shared-memory-base-name | 共享内存连接的共享内存名称（仅限 Windows） |  |
| --socket | 要使用的 Unix 套接字文件或 Windows 命名管道 |  |
| --user | 连接到服务器时要使用的 MySQL 用户名 |  |
| 选项名称 | 描述 | 引入版本 |

+   `--default-auth=*`plugin`*`

    | 命令行格式 | `--default-auth=plugin` |
    | --- | --- |
    | 类型 | 字符串 |

    提示使用哪个客户端端身份验证插件。参见第 8.2.17 节，“可插拔身份验证”。

+   `--host=*`host_name`*`, `-h *`host_name`*`

    | 命令行格式 | `--host=host_name` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `localhost` |

    MySQL 服务器运行的主机。该值可以是主机名、IPv4 地址或 IPv6 地址。默认值为`localhost`。

+   [`--password[=*`pass_val`*]`](connection-options.html#option_general_password), `-p[*`pass_val`*]`

    | 命令行格式 | `--password[=password]` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `[none]` |

    用于连接到服务器的 MySQL 帐户的密码。密码值是可选的。如果未提供，则客户端程序会提示输入密码。如果提供了密码，则`--password=`或`-p`后面必须*没有空格*。如果未指定密码选项，则默认情况是不发送密码。

    在命令行上指定密码应被视为不安全。为了避免在命令行上提供密码，请使用选项文件。参见第 8.1.2.1 节，“密码安全的最终用户指南”。

    要明确指定没有密码，且客户端程序不应提示输入密码，请使用`--skip-password`选项。

+   [`--password1[=*`pass_val`*]`](connection-options.html#option_general_password1)

    | 命令行格式 | `--password1[=password]` |
    | --- | --- |
    | 引入版本 | 8.0.27 |
    | 类型 | 字符串 |

    用于连接到服务器的 MySQL 帐户的多因素身份验证因子 1 的密码。密码值是可选的。如果未提供，则客户端程序会提示输入密码。如果提供了密码，则`--password1=`后面必须*没有空格*。如果未指定密码选项，则默认情况是不发送密码。

    在命令行上指定密码应被视为不安全。为了避免在命令行上提供密码，请使用选项文件。参见第 8.1.2.1 节，“密码安全的最终用户指南”。

    要明确指定没有密码，且客户端程序不应提示输入密码，请使用`--skip-password1`选项。

    `--password1`和`--password`是同义词，`--skip-password1`和`--skip-password`也是同义词。

+   [`--password2[=*`pass_val`*]`](connection-options.html#option_general_password2)

    | 命令行格式 | `--password2[=password]` |
    | --- | --- |
    | 引入版本 | 8.0.27 |
    | 类型 | 字符串 |

    用于连接到服务器的 MySQL 帐户的多因素认证因子 2 的密码。此选项的语义类似于`--password1`的语义；有关详细信息，请参阅该选项的描述。

+   [`--password3[=*`pass_val`*]`](connection-options.html#option_general_password3)

    | 命令行格式 | `--password3[=password]` |
    | --- | --- |
    | 引入版本 | 8.0.27 |
    | 类型 | 字符串 |

    用于连接到服务器的 MySQL 帐户的多因素认证因子 3 的密码。此选项的语义类似于`--password1`的语义；有关详细信息，请参阅该选项的描述。

+   `--pipe`, `-W`

    | 命令行格式 | `--pipe` |
    | --- | --- |
    | 类型 | 字符串 |

    在 Windows 上，使用命名管道连接到服务器。此选项仅在服务器启动时启用了`named_pipe`系统变量以支持命名管道连接时适用。此外，进行连接的用户必须是由`named_pipe_full_access_group`系统变量指定的 Windows 组的成员。

+   `--plugin-dir=*`dir_name`*`

    | 命令行格式 | `--plugin-dir=dir_name` |
    | --- | --- |
    | 类型 | 目录名称 |

    查找插件的目录。如果使用`--default-auth`选项指定了身份验证插件但客户端程序找不到它，请指定此选项。请参阅第 8.2.17 节，“可插拔认证”。

+   `--port=*`port_num`*`, `-P *`port_num`*`

    | 命令行格式 | `--port=port_num` |
    | --- | --- |
    | 类型 | 数字 |
    | 默认值 | `3306` |

    对于 TCP/IP 连接，要使用的端口号。默认端口号为 3306。

+   `--protocol={TCP|SOCKET|PIPE|MEMORY}`

    | 命令行格式 | `--protocol=type` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `[见文本]` |
    | 有效值 | `TCP``SOCKET``PIPE``MEMORY` |

    此选项明确指定用于连接到服务器的传输协议。当其他连接参数通常导致使用不希望使用的协议时，这很有用。例如，默认情况下，在 Unix 上连接到`localhost`时会使用 Unix 套接字文件：

    ```sql
    mysql --host=localhost
    ```

    要强制使用 TCP/IP 传输，请指定`--protocol`选项：

    ```sql
    mysql --host=localhost --protocol=TCP
    ```

    以下表格显示了允许的`--protocol`选项值，并指示每个值适用的平台。这些值不区分大小写。

    | `--protocol` 值 | 使用的传输协议 | 适用的平台 |
    | --- | --- | --- |
    | `TCP` | TCP/IP 传输到本地或远程服务器 | 所有 |
    | `SOCKET` | Unix 套接字文件传输到本地服务器 | Unix 和类 Unix 系统 |
    | `PIPE` | 命名管道传输到本地服务器 | Windows |
    | `MEMORY` | 共享内存传输到本地服务器 | Windows |

    另请参阅第 6.2.7 节，“连接传输协议”

+   `--shared-memory-base-name=*`name`*`

    | 命令行格式 | `--shared-memory-base-name=name` |
    | --- | --- |
    | 平台特定 | Windows |

    在 Windows 上，用于使用共享内存连接到本地服务器的共享内存名称。默认值为`MYSQL`。共享内存名称区分大小写。

    此选项仅在服务器启动时启用了`shared_memory`系统变量以支持共享内存连接时才适用。

+   `--socket=*`path`*`, `-S *`path`*`

    | 命令行格式 | `--socket={file_name&#124;pipe_name}` |
    | --- | --- |
    | 类型 | 字符串 |

    在 Unix 上，用于使用命名管道连接到本地服务器的 Unix 套接字文件的名称。默认的 Unix 套接字文件名为`/tmp/mysql.sock`。

    在 Windows 上，用于连接到本地服务器的命名管道的名称。默认的 Windows 管道名称为`MySQL`。管道名称不区分大小写。

    在 Windows 上，只有在服务器启动时启用了`named_pipe`系统变量以支持命名管道连接时，此选项才适用。此外，进行连接的用户必须是由`named_pipe_full_access_group`系统变量指定的 Windows 组的成员。

+   `--user=*`user_name`*`, `-u *`user_name`*`

    | 命令行格式 | `--user=user_name` |
    | --- | --- |
    | 类型 | 字符串 |

    用于连接到服务器的 MySQL 帐户的用户名。在 Windows 上，默认用户名为`ODBC`，在 Unix 上为您的 Unix 登录名。

#### 加密连接的命令选项

本节描述了客户端程序的选项，指定是否使用加密连接到服务器，证书和密钥文件的名称，以及与加密连接支持相关的其他参数。有关建议用法示例以及如何检查连接是否加密，请参阅 8.3.1 节，“配置 MySQL 使用加密连接”。

注意

这些选项仅对使用加密传输协议的连接有效；即 TCP/IP 和 Unix 套接字文件连接。请参阅 6.2.7 节，“连接传输协议”

有关从 MySQL C API 使用加密连接的信息，请参阅 支持加密连接。

**表 6.5 连接加密选项摘要**

| 选项名称 | 描述 | 引入版本 | 废弃版本 |
| --- | --- | --- | --- |
| --get-server-public-key | 从服务器请求 RSA 公钥 |  |  |
| --server-public-key-path | 包含 RSA 公钥的文件路径名 |  |  |
| --ssl-ca | 包含受信任的 SSL 证书颁发机构列表的文件 |  |  |
| --ssl-capath | 包含受信任的 SSL 证书颁发机构证书文件的目录 |  |  |
| --ssl-cert | 包含 X.509 证书的文件 |  |  |
| --ssl-cipher | 连接加密的允许密码 |  |  |
| --ssl-crl | 包含证书吊销列表的文件 |  |  |
| --ssl-crlpath | 包含证书吊销列表文件的目录 |  |  |
| --ssl-fips-mode | 是否在客户端启用 FIPS 模式 |  | 8.0.34 |
| --ssl-key | 包含 X.509 密钥的文件 |  |  |
| --ssl-mode | 与服务器连接的期望安全状态 |  |  |
| --ssl-session-data | 包含 SSL 会话数据的文件 | 8.0.29 |  |
| --ssl-session-data-continue-on-failed-reuse | 如果会话重用失败是否建立连接 | 8.0.29 |  |
| --tls-ciphersuites | 用于加密连接的允许的 TLSv1.3 密码套件 | 8.0.16 |  |
| --tls-version | 加密连接的允许 TLS 协议 |  |  |
| 选项名称 | 描述 | 引入 | 废弃 |

+   `--get-server-public-key`

    | 命令行格式 | `--get-server-public-key` |
    | --- | --- |
    | 类型 | 布尔值 |

    从服务器请求用于 RSA 密钥对密码交换所需的公钥。此选项适用于使用`caching_sha2_password`认证插件进行身份验证的客户端。对于该插件，除非请求，否则服务器不会发送公钥。对于不使用该插件进行身份验证的帐户，此选项将被忽略。如果不使用基于 RSA 的密码交换（例如客户端使用安全连接连接到服务器时），则也会被忽略。

    如果`--server-public-key-path=*`file_name`*`被指定并指定了有效的公钥文件，则优先于`--get-server-public-key`。

    有关`caching_sha2_password`插件的信息，请参见第 8.4.1.2 节，“缓存 SHA-2 可插拔认证”。

+   `--server-public-key-path=*`file_name`*`

    | 命令行格式 | `--server-public-key-path=file_name` |
    | --- | --- |
    | 类型 | 文件名 |

    包含客户端端的用于 RSA 密钥对密码交换所需的服务器端公钥的 PEM 格式文件的路径名。此选项适用于使用`sha256_password`或`caching_sha2_password`认证插件进行身份验证的客户端。对于不使用这些插件进行身份验证的帐户，此选项将被忽略。如果不使用基于 RSA 的密码交换（例如客户端使用安全连接连接到服务器时），则也会被忽略。

    如果`--server-public-key-path=*`file_name`*`被指定并指定了有效的公钥文件，则优先于`--get-server-public-key`。

    此选项仅在使用 OpenSSL 构建 MySQL 时可用。

    有关`sha256_password`和`caching_sha2_password`插件的信息，请参见第 8.4.1.3 节，“SHA-256 可插拔认证”，以及第 8.4.1.2 节，“缓存 SHA-2 可插拔认证”。

+   `--ssl-ca=*`file_name`*`

    | 命令行格式 | `--ssl-ca=file_name` |
    | --- | --- |
    | 类型 | 文件名 |

    包含受信任 SSL 证书颁发机构（CA）证书文件的路径名。该文件包含一组受信任的 SSL 证书颁发机构。

    要告诉客户端在与服务器建立加密连接时不验证服务器证书，请同时指定`--ssl-ca`和`--ssl-capath`。服务器仍然根据客户端账户的任何适用要求验证客户端，并且仍然使用服务器端指定的任何`ssl_ca`或`ssl_capath`系统变量值。

    要为服务器指定 CA 文件，请设置`ssl_ca`系统变量。

+   `--ssl-capath=*`dir_name`*`

    | 命令行格式 | `--ssl-capath=dir_name` |
    | --- | --- |
    | 类型 | 目录名 |

    包含受信任 SSL 证书颁发机构（CA）证书文件的目录路径名，格式为 PEM。

    要告诉客户端在与服务器建立加密连接时不验证服务器证书，请同时指定`--ssl-ca`和`--ssl-capath`。服务器仍然根据客户端账户的任何适用要求验证客户端，并且仍然使用服务器端指定的任何`ssl_ca`或`ssl_capath`系统变量值。

    要为服务器指定 CA 目录，请设置`ssl_capath`系统变量。

+   `--ssl-cert=*`file_name`*`

    | 命令行格式 | `--ssl-cert=file_name` |
    | --- | --- |
    | 类型 | 文件名 |

    包含客户端 SSL 公钥证书文件的路径名，格式为 PEM。

    要指定服务器 SSL 公钥证书文件，请设置`ssl_cert`系统变量。

    注意

    链式 SSL 证书支持在 v8.0.30 中添加；之前只读取第一个证书。

+   `--ssl-cipher=*`cipher_list`*`

    | 命令行格式 | `--ssl-cipher=name` |
    | --- | --- |
    | 类型 | 字符串 |

    允许加密连接使用 TLS 协议直至 TLSv1.2 的加密密码列表。如果列表中没有支持的密码，使用这些 TLS 协议的加密连接将无法工作。

    为了最大的可移植性，*`cipher_list`*应该是一个或多个密码名称的列表，用冒号分隔。例如：

    ```sql
    --ssl-cipher=AES128-SHA
    --ssl-cipher=DHE-RSA-AES128-GCM-SHA256:AES128-SHA
    ```

    OpenSSL 支持在[`www.openssl.org/docs/manmaster/man1/ciphers.html`](https://www.openssl.org/docs/manmaster/man1/ciphers.html)中描述的指定密码的语法。

    有关 MySQL 支持的加密密码，请参阅第 8.3.2 节，“加密连接 TLS 协议和密码”。

    要为服务器指定加密密码，设置`ssl_cipher`系统变量。

+   `--ssl-crl=*`file_name`*`

    | 命令行格式 | `--ssl-crl=file_name` |
    | --- | --- |
    | 类型 | 文件名 |

    包含以 PEM 格式的证书吊销列表的文件路径名。

    如果既没有提供`--ssl-crl`也没有提供`--ssl-crlpath`，则不执行 CRL 检查，即使 CA 路径包含证书吊销列表。

    要为服务器指定吊销列表文件，设置`ssl_crl`系统变量。

+   `--ssl-crlpath=*`dir_name`*`

    | 命令行格式 | `--ssl-crlpath=dir_name` |
    | --- | --- |
    | 类型 | 目录名 |

    包含以 PEM 格式的证书吊销列表文件的目录路径名。

    如果既没有提供`--ssl-crl`也没有提供`--ssl-crlpath`，则不执行 CRL 检查，即使 CA 路径包含证书吊销列表。

    要为服务器指定吊销列表目录，设置`ssl_crlpath`系统变量。

+   `--ssl-fips-mode={OFF|ON|STRICT}`

    | 命令行格式 | `--ssl-fips-mode={OFF&#124;ON&#124;STRICT}` |
    | --- | --- |
    | 已弃用 | 8.0.34 |
    | 类型 | 枚举 |
    | 默认值 | `OFF` |
    | 有效值 | `OFF``ON``STRICT` |

    控制是否在客户端端启用 FIPS 模式。`--ssl-fips-mode`选项与其他`--ssl-*`xxx`*`选项不同，它不用于建立加密连接，而是用于影响允许的加密操作。请参阅第 8.8 节，“FIPS 支持”。

    这些`--ssl-fips-mode`值是允许的：

    +   `OFF`: 禁用 FIPS 模式。

    +   `ON`: 启用 FIPS 模式。

    +   `STRICT`: 启用“严格”FIPS 模式。

    注意

    如果 OpenSSL FIPS 对象模块不可用，则`--ssl-fips-mode`的唯一允许值为`OFF`。在这种情况下，将`--ssl-fips-mode`设置为`ON`或`STRICT`会导致客户端在启动时产生警告并在非 FIPS 模式下运行。

    要为服务器指定 FIPS 模式，设置`ssl_fips_mode`系统变量。

+   `--ssl-key=*`file_name`*`

    | 命令行格式 | `--ssl-key=file_name` |
    | --- | --- |
    | 类型 | 文件名 |

    客户端 SSL 私钥文件的 PEM 格式路径名。为了更好的安全性，请使用至少 2048 位的 RSA 密钥大小的证书。

    如果密钥文件受密码保护，则客户端程序会提示用户输入密码。密码必须以交互方式给出；不能存储在文件中。如果密码不正确，则程序会继续，就好像无法读取密钥一样。

    要指定服务器 SSL 私钥文件，请设置 `ssl_key` 系统变量。

+   `--ssl-mode=*`mode`*`

    | 命令行格式 | `--ssl-mode=mode` |
    | --- | --- |
    | 类型 | 枚举 |
    | 默认值 | `PREFERRED` |
    | 有效值 | `DISABLED``PREFERRED``REQUIRED``VERIFY_CA``VERIFY_IDENTITY` |

    此选项指定与服务器的连接所需的安全状态。这些模式值是允许的，按照严格程度递增的顺序：

    +   `DISABLED`: 建立一个未加密的连接。

    +   `PREFERRED`: 如果服务器支持加密连接，则建立加密连接，如果无法建立加密连接，则回退到未加密连接。如果未指定 `--ssl-mode`，则默认为此选项。

        通过 Unix 套接字文件进行的连接不会使用 `PREFERRED` 模式进行加密。要强制对 Unix 套接字文件连接进行加密，请使用 `REQUIRED` 或更严格的模式。（但是，默认情况下，套接字文件传输是安全的，因此加密套接字文件连接不会增加安全性，反而会增加 CPU 负载。）

    +   `REQUIRED`: 如果服务器支持加密连接，则建立加密连接。如果无法建立加密连接，则连接尝试失败。

    +   `VERIFY_CA`: 类似于 `REQUIRED`，但还会根据配置的 CA 证书验证服务器证书颁发机构（CA）证书。如果找不到有效的匹配 CA 证书，则连接尝试失败。

    +   `VERIFY_IDENTITY`: 类似于 `VERIFY_CA`，但还通过检查客户端用于连接到服务器的主机名与服务器发送给客户端的证书中的标识进行主机名身份验证：

        +   从 MySQL 8.0.12 开始，如果客户端使用 OpenSSL 1.0.2 或更高版本，则客户端会检查用于连接的主机名是否与服务器证书中的主题备用名称值或通用名称值匹配。主机名身份验证也适用于使用通配符指定通用名称的证书。

        +   否则，客户端会检查用于连接的主机名是否与服务器证书中的通用名称值匹配。

        如果存在不匹配，连接将失败。对于加密连接，此选项有助于防止中间人攻击。

        注意

        使用`VERIFY_IDENTITY`进行主机名身份验证无法与由服务器自动创建或使用**mysql_ssl_rsa_setup**（请参见 8.3.3.1 节，“使用 MySQL 创建 SSL 和 RSA 证书和密钥”会在其他默认设置不变的情况下产生加密连接。然而，为了防止复杂的中间人攻击，客户端验证服务器的身份非常重要。设置`--ssl-mode=VERIFY_CA`和`--ssl-mode=VERIFY_IDENTITY`比默认设置更好，以帮助防止这种类型的攻击。要实施这些设置之一，必须首先确保服务器的 CA 证书可靠地提供给所有在您的环境中使用它的客户端，否则将导致可用性问题。因此，它们不是默认设置。

    `--ssl-mode`选项与 CA 证书选项的交互如下：

    +   如果未显式设置`--ssl-mode`，则使用`--ssl-ca`或`--ssl-capath`意味着`--ssl-mode=VERIFY_CA`。

    +   对于`VERIFY_CA`或`VERIFY_IDENTITY`的`--ssl-mode`值，还需要`--ssl-ca`或`--ssl-capath`，以提供与服务器使用的 CA 证书匹配的 CA 证书。

    +   具有值不是`VERIFY_CA`或`VERIFY_IDENTITY`的显式`--ssl-mode`选项，以及显式`--ssl-ca`或`--ssl-capath`选项，会产生警告，表明尽管指定了 CA 证书选项，但不会执行对服务器证书的验证。

    要求 MySQL 帐户使用加密连接，请使用 `CREATE USER` 创建带有 `REQUIRE SSL` 子句的帐户，或者对现有帐户使用 `ALTER USER` 添加 `REQUIRE SSL` 子句。这将导致使用该帐户的客户端的连接尝试被拒绝，除非 MySQL 支持加密连接并且可以建立加密连接。

    `REQUIRE` 子句允许其他与加密相关的选项，这些选项可用于强制执行比 `REQUIRE SSL` 更严格的安全要求。有关客户端连接时必须或可以指定的命令选项的详细信息，请参阅 CREATE USER SSL/TLS 选项。

+   `--ssl-session-data=*`file_name`*`

    | 命令行格式 | `--ssl-session-data=file_name` |
    | --- | --- |
    | 引入版本 | 8.0.29 |
    | 类型 | 文件名 |

    客户端 SSL 会话数据文件的 PEM 格式路径名，用于会话重用。

    当您使用 `--ssl-session-data` 选项调用 MySQL 客户端程序时，如果提供了文件，客户端会尝试从文件中反序列化会话数据，然后使用它来建立新连接。如果您提供了一个文件，但会话未被重用，则连接将失败，除非您在调用客户端程序时在命令行上还指定了 `--ssl-session-data-continue-on-failed-reuse` 选项。

    **mysql** 命令 `ssl_session_data_print` 生成会话数据文件（参见 Section 6.5.1.2, “mysql Client Commands”）。

+   `ssl-session-data-continue-on-failed-reuse`

    | 命令行格式 | `--ssl-session-data-continue-on-failed-reuse` |
    | --- | --- |
    | 引入版本 | 8.0.29 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    控制是否启动新连接以替换尝试但未能重用使用 `--ssl-session-data` 命令行选项指定的会话数据的连接。默认情况下，`--ssl-session-data-continue-on-failed-reuse` 命令行选项关闭，这会导致当提供会话数据但未重用时，客户端程序返回连接失败。

    为了确保在会话重用失败后静默打开新的不相关连接，请使用`--ssl-session-data`和`--ssl-session-data-continue-on-failed-reuse`命令行选项调用 MySQL 客户端程序。

+   `--tls-ciphersuites=*`ciphersuite_list`*`

    | 命令行格式 | `--tls-ciphersuites=ciphersuite_list` |
    | --- | --- |
    | 引入版本 | 8.0.16 |
    | 类型 | 字符串 |
    | 默认值 | `空字符串` |

    此选项指定客户端允许使用 TLSv1.3 的加密连接的密码套件。该值是一个或多个以冒号分隔的密码套件名称列表。例如：

    ```sql
    mysql --tls-ciphersuites="*suite1*:*suite2*:*suite3*"
    ```

    可以为此选项命名的密码套件取决于用于编译 MySQL 的 SSL 库。如果未设置此选项，则客户端允许默认密码套件。如果将选项设置为空字符串，则不启用任何密码套件，无法建立加密连接。有关更多信息，请参见第 8.3.2 节，“加密连接 TLS 协议和密码套件”。

    此选项在 MySQL 8.0.16 中添加。

    要指定服务器允许的密码套件，设置`tls_ciphersuites`系统变量。

+   `--tls-version=*`protocol_list`*`

    | 命令行格式 | `--tls-version=protocol_list` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值（≥ 8.0.16） | `TLSv1,TLSv1.1,TLSv1.2,TLSv1.3`（OpenSSL 1.1.1 或更高版本）`TLSv1,TLSv1.1,TLSv1.2`（否则） |
    | 默认值（≤ 8.0.15） | `TLSv1,TLSv1.1,TLSv1.2` |

    此选项指定客户端允许用于加密连接的 TLS 协议。该值是一个或多个逗号分隔的协议版本列表。例如：

    ```sql
    mysql --tls-version="TLSv1.2,TLSv1.3"
    ```

    可以为此选项命名的协议取决于用于编译 MySQL 的 SSL 库以及 MySQL Server 的发布版本。

    重要

    +   从 MySQL 8.0.28 开始，MySQL 服务器移除了对 TLSv1 和 TLSv1.1 连接协议的支持。这些协议从 MySQL 8.0.26 开始被弃用，尽管 MySQL 服务器客户端在使用弃用的 TLS 协议版本时不会向用户返回警告。从 MySQL 8.0.28 开始，支持`--tls-version`选项的客户端，包括 MySQL Shell，不能使用将协议设置为 TLSv1 或 TLSv1.1 的 TLS/SSL 连接。如果客户端尝试使用这些协议进行连接，对于 TCP 连接，连接将失败，并向客户端返回错误。对于套接字连接，如果`--ssl-mode`设置为`REQUIRED`，连接将失败，否则连接将建立但 TLS/SSL 被禁用。有关更多信息，请参阅移除对 TLSv1 和 TLSv1.1 协议的支持。

    +   从 MySQL 8.0.16 开始，MySQL 服务器支持 TLSv1.3 协议，前提是 MySQL 服务器使用了 OpenSSL 1.1.1 或更高版本进行编译。服务器在启动时检查 OpenSSL 的版本，如果低于 1.1.1，则将从与 TLS 版本相关的服务器系统变量的默认值中移除 TLSv1.3（例如`tls_version`系统变量）。

    允许的协议应该被选择，以避免在列表中留下“漏洞”。例如，这些值没有漏洞：

    ```sql
    --tls-version="TLSv1,TLSv1.1,TLSv1.2,TLSv1.3"
    --tls-version="TLSv1.1,TLSv1.2,TLSv1.3"
    --tls-version="TLSv1.2,TLSv1.3"
    --tls-version="TLSv1.3"

    From MySQL 8.0.28, only the last two values are suitable.
    ```

    这些值有漏洞，不应该使用：

    ```sql
    --tls-version="TLSv1,TLSv1.2"
    --tls-version="TLSv1.1,TLSv1.3"
    ```

    有关详细信息，请参阅第 8.3.2 节，“加密连接 TLS 协议和密码”。

    要指定服务器允许的 TLS 协议，设置`tls_version`系统变量。

#### 连接压缩的命令选项

本节描述了使客户端程序能够控制与服务器连接中压缩使用的选项。有关更多信息和示例，请参阅第 6.2.8 节，“连接压缩控制”。

**表 6.6 连接压缩选项摘要**

| 选项名称 | 描述 | 引入版本 | 弃用版本 |
| --- | --- | --- | --- |
| --compress | 压缩客户端和服务器之间发送的所有信息 |  | 8.0.18 |
| --compression-algorithms | 连接到服务器的允许的压缩算法 | 8.0.18 |  |
| --zstd-compression-level | 使用 zstd 压缩连接到服务器的压缩级别 | 8.0.18 |  |

+   `--compress`, `-C`

    | 命令行格式 | `--compress[={OFF&#124;ON}]` |
    | --- | --- |
    | 废弃 | 8.0.18 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    如果可能的话，压缩客户端和服务器之间发送的所有信息。

    从 MySQL 8.0.18 开始，此选项已被废弃。预计在未来的 MySQL 版本中将会移除。请参阅配置传统连接压缩。

+   `--compression-algorithms=*`value`*`

    | 命令行格式 | `--compression-algorithms=value` |
    | --- | --- |
    | 引入版本 | 8.0.18 |
    | 类型 | 集合 |
    | 默认值 | `uncompressed` |
    | 有效值 | `zlib``zstd``uncompressed` |

    用于连接到服务器的允许的压缩算法。可用的算法与`protocol_compression_algorithms`系统变量相同。默认值为`uncompressed`。

    此选项在 MySQL 8.0.18 中添加。

+   `--zstd-compression-level=*`level`*`

    | 命令行格式 | `--zstd-compression-level=#` |
    | --- | --- |
    | 引入版本 | 8.0.18 |
    | 类型 | 整数 |

    用于使用`zstd`压缩算法连接到服务器的连接的压缩级别。允许的级别从 1 到 22，较大的值表示较高级别的压缩。默认的`zstd`压缩级别为 3。压缩级别设置对不使用`zstd`压缩的连接没有影响。

    此选项在 MySQL 8.0.18 中添加。
