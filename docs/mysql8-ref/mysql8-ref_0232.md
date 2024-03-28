# 7.1.8 服务器系统变量

> 原文：[`dev.mysql.com/doc/refman/8.0/en/server-system-variables.html`](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html)

MySQL 服务器维护许多影响其操作的系统变量。大多数系统变量可以在服务器启动时使用命令行选项或选项文件设置。大多数可以使用`SET`语句在运行时动态更改，这使您可以修改服务器的操作而无需停止和重新启动它。一些变量是只读的，它们的值由系统环境、MySQL 在系统上的安装方式或可能由用于编译 MySQL 的选项确定。大多数系统变量都有默认值，但也有例外，包括只读变量。您还可以在表达式中使用系统变量值。

设置全局系统变量的运行时值通常需要`SYSTEM_VARIABLES_ADMIN`权限（或已废弃的`SUPER`权限）。设置会话系统运行时变量值通常不需要特殊权限，任何用户都可以执行，尽管也有例外情况。更多信息，请参阅 Section 7.1.9.1, “System Variable Privileges”

有几种方法可以查看系统变量的名称和值：

+   要查看服务器使用的值（基于其编译默认值和读取的任何选项文件），请使用此命令：

    ```sql
    mysqld --verbose --help
    ```

+   要查看服务器仅基于其编译默认值使用的值，忽略任何选项文件中的设置，请使用此命令：

    ```sql
    mysqld --no-defaults --verbose --help
    ```

+   要查看运行中服务器使用的当前值，请使用`SHOW VARIABLES`语句或性能模式系统变量表。请参阅 Section 29.12.14, “Performance Schema System Variable Tables”.

本节提供了每个系统变量的描述。有关系统变量摘要表，请参阅 Section 7.1.5, “Server System Variable Reference”。有关系统变量操作的更多信息，请参阅 Section 7.1.9, “Using System Variables”。

有关其他系统变量信息，请参阅以下章节：

+   Section 7.1.9, “Using System Variables”，讨论了设置和显示系统变量值的语法。

+   Section 7.1.9.2, “Dynamic System Variables”，列出了可以在运行时设置的变量。

+   有关调整系统变量的信息，请参阅第 7.1.1 节，“配置服务器”。

+   第 17.14 节，“InnoDB 启动选项和系统变量”列出了`InnoDB`系统变量。

+   第 25.4.3.9.2 节，“NDB 集群系统变量”列出了特定于 NDB 集群的系统变量。

+   有关特定于复制的服务器系统变量的信息，请参见第 19.1.6 节，“复制和二进制日志选项和变量”。

注意

以下一些变量描述涉及“启用”或“禁用”变量。这些变量可以通过`SET`语句将它们设置为`ON`或`1`来启用，或者通过将它们设置为`OFF`或`0`来禁用。布尔变量可以在启动时设置为`ON`、`TRUE`、`OFF`和`FALSE`（不区分大小写），以及`1`和`0`。参见第 6.2.2.4 节，“程序选项修饰符”。

一些系统变量控制缓冲区或缓存的大小。对于给定的缓冲区，服务器可能需要分配内部数据结构。这些结构通常是从分配给缓冲区的总内存中分配的，所需空间的量可能取决于平台。这意味着当您为控制缓冲区大小的系统变量分配一个值时，实际可用的空间量可能与分配的值不同。在某些情况下，该量可能小于分配的值。服务器也可能将值调整为更高的值。例如，如果您为最小值为 1024 的变量分配了值 0，则服务器会将该值设置为 1024。

缓冲区大小、长度和堆栈大小的值均以字节为单位，除非另有规定。

注意

一些系统变量描述包括块大小，在这种情况下，不是已声明块大小的整数倍的值在被服务器存储之前会被舍入到下一个较低的块大小的倍数，即`FLOOR(*`value`*)` `* *`block_size`*`。

*示例*：假设给定变量的块大小为 4096，并且您将变量的值设置为 100000（我们假设变量的最大值大于此数字）。由于 100000 / 4096 = 24.4140625，服务器会在存储之前自动将该值降低到 98304（24 * 4096）。

在某些情况下，变量的规定最大值是 MySQL 解析器允许的最大值，但不是块大小的精确倍数。在这种情况下，有效最大值是块大小的下一个较低倍数。

*示例*: 系统变量的最大值显示为 4294967295（2³²-1），其块大小为 1024。4294967295 / 1024 = 4194303.9990234375，因此如果将此变量设置为其规定的最大值，则实际存储的值为 4194303 * 1024 = 4294966272。

一些系统变量接受文件名值。除非另有说明，如果值是相对路径名，则默认文件位置是数据目录。要明确指定位置，请使用绝对路径名。假设数据目录是`/var/mysql/data`。如果文件值变量以相对路径名给出，则其位置在`/var/mysql/data`下。如果值是绝对路径名，则其位置如路径名所示。

+   `activate_all_roles_on_login`

    | 命令行格式 | `--activate-all-roles-on-login[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `activate_all_roles_on_login` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    是否启用用户登录时自动激活所有授予的角色：

    +   如果`activate_all_roles_on_login`被启用，服务器在登录时会激活每个账户被授予的所有角色。这优先于通过`SET DEFAULT ROLE`指定的默认角色。

    +   如果`activate_all_roles_on_login`被禁用，服务器在登录时会激活通过`SET DEFAULT ROLE`指定的默认角色（如果有）。

    授予的角色包括明确授予用户的角色以及在`mandatory_roles`系统变量值中命名的角色。

    `activate_all_roles_on_login`仅在登录时应用，并且对于以定义者上下文执行的存储程序和视图的执行开始。要在会话中更改活动角色，请使用`SET ROLE`。要为存储程序更改活动角色，程序体应执行`SET ROLE`。

+   `admin_address`

    | 命令行格式 | `--admin-address=addr` |
    | --- | --- |
    | 引入版本 | 8.0.14 |
    | 系统变量 | `admin_address` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 字符串 |

    用于在管理网络接口上监听 TCP/IP 连接的 IP 地址（请参见第 7.1.12.1 节，“连接接口”）。没有默认的`admin_address`值。如果在启动时未指定此变量，则服务器不维护任何管理接口。服务器还有一个`bind_address`系统变量用于配置常规（非管理）客户端 TCP/IP 连接。请参见第 7.1.12.1 节，“连接接口”。

    如果指定了`admin_address`，其值必须满足以下要求：

    +   值必须是单个 IPv4 地址、IPv6 地址或主机名。

    +   值不能指定通配符地址格式（`*`，`0.0.0.0`或`::`）。

    +   从 MySQL 8.0.22 开始，值可以包括网络命名空间指定符。

    IP 地址可以指定为 IPv4 或 IPv6 地址。如果值是主机名，服务器将解析该名称为 IP 地址并绑定到该地址。如果主机名解析为多个 IP 地址，则服务器使用第一个 IPv4 地址（如果有）或否则使用第一个 IPv6 地址。

    服务器对不同类型的地址处理如下：

    +   如果地址是 IPv4 映射地址，则服务器接受该地址的 TCP/IP 连接，无论是 IPv4 还是 IPv6 格式。例如，如果服务器绑定到`::ffff:127.0.0.1`，客户端可以使用`--host=127.0.0.1`或`--host=::ffff:127.0.0.1`进行连接。

    +   如果地址是“常规”IPv4 或 IPv6 地址（例如`127.0.0.1`或`::1`），服务器仅接受该 IPv4 或 IPv6 地址的 TCP/IP 连接。

    指定地址的网络命名空间的规则如下：

    +   可以为 IP 地址或主机名指定网络命名空间。

    +   无法为通配符 IP 地址指定网络命名空间。

    +   对于给定的地址，网络命名空间是可选的。如果给定，必须在地址后紧跟`/*`ns`*`后缀指定。

    +   没有`/*`ns`*`后缀的地址使用主机系统的全局命名空间。因此，全局命名空间是默认值。

    +   带有`/*`ns`*`后缀的地址使用名为*`ns`*的命名空间。

    +   主机系统必须支持网络命名空间，并且每个命名空间必须事先设置好。命名不存在的命名空间会产生错误。

    有关网络命名空间的更多信息，请参见第 7.1.14 节，“网络命名空间支持”。

    如果绑定到地址失败，服务器将产生错误并且不会启动。

    `admin_address`系统变量类似于将服务器绑定到普通客户端连接地址的`bind_address`系统变量，但有以下区别：

    +   `bind_address`允许多个地址。`admin_address`允许单个地址。

    +   `bind_address`允许通配符地址。`admin_address`不允许。

+   `admin_port`

    | 命令行格式 | `--admin-port=port_num` |
    | --- | --- |
    | 引入版本 | 8.0.14 |
    | 系统变量 | `admin_port` |
    | 作用范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `33062` |
    | 最小值 | `0` |
    | 最大值 | `65535` |

    用于管理网络接口连接的 TCP/IP 端口号（参见第 7.1.12.1 节，“连接接口”）。将此变量设置为 0 会使用默认值。

    如果未指定`admin_address`，则设置`admin_port`不起作用，因为在这种情况下，服务器不维护管理网络接口。

+   `admin_ssl_ca`

    | 命令行格式 | `--admin-ssl-ca=file_name` |
    | --- | --- |
    | 引入版本 | 8.0.21 |
    | 系统变量 | `admin_ssl_ca` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 文件名 |
    | 默认值 | `NULL` |

    `admin_ssl_ca`系统变量类似于`ssl_ca`，不同之处在于它适用于管理连接接口而不是主连接接口。有关为管理接口配置加密支持的信息，请参见加密连接的管理接口支持。

+   `admin_ssl_capath`

    | 命令行格式 | `--admin-ssl-capath=dir_name` |
    | --- | --- |
    | 引入版本 | 8.0.21 |
    | 系统变量 | `admin_ssl_capath` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 目录名 |
    | 默认值 | `NULL` |

    `admin_ssl_capath` 系统变量类似于 `ssl_capath`，不同之处在于它适用于管理连接接口而不是主连接接口。有关为管理接口配置加密支持的信息，请参阅加密连接的管理接口支持。

+   `admin_ssl_cert`

    | 命令行格式 | `--admin-ssl-cert=file_name` |
    | --- | --- |
    | 引入版本 | 8.0.21 |
    | 系统变量 | `admin_ssl_cert` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 文件名 |
    | 默认值 | `NULL` |

    `admin_ssl_cert` 系统变量类似于 `ssl_cert`，不同之处在于它适用于管理连接接口而不是主连接接口。有关为管理接口配置加密支持的信息，请参阅加密连接的管理接口支持。

+   `admin_ssl_cipher`

    | 命令行格式 | `--admin-ssl-cipher=name` |
    | --- | --- |
    | 引入版本 | 8.0.21 |
    | 系统变量 | `admin_ssl_cipher` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 字符串 |
    | 默认值 | `NULL` |

    `admin_ssl_cipher` 系统变量类似于 `ssl_cipher`，不同之处在于它适用于管理连接接口而不是主连接接口。有关为管理接口配置加密支持的信息，请参阅加密连接的管理接口支持。

+   `admin_ssl_crl`

    | 命令行格式 | `--admin-ssl-crl=file_name` |
    | --- | --- |
    | 引入版本 | 8.0.21 |
    | 系统变量 | `admin_ssl_crl` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 文件名 |
    | 默认值 | `NULL` |

    `admin_ssl_crl` 系统变量类似于 `ssl_crl`，不同之处在于它适用于管理连接接口而不是主连接接口。有关为管理接口配置加密支持的信息，请参阅 加密连接的管理接口支持。

+   `admin_ssl_crlpath`

    | 命令行格式 | `--admin-ssl-crlpath=dir_name` |
    | --- | --- |
    | 引入版本 | 8.0.21 |
    | 系统变量 | `admin_ssl_crlpath` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 目录名 |
    | 默认值 | `NULL` |

    `admin_ssl_crlpath` 系统变量类似于 `ssl_crlpath`，不同之处在于它适用于管理连接接口而不是主连接接口。有关为管理接口配置加密支持的信息，请参阅 加密连接的管理接口支持。

+   `admin_ssl_key`

    | 命令行格式 | `--admin-ssl-key=file_name` |
    | --- | --- |
    | 引入版本 | 8.0.21 |
    | 系统变量 | `admin_ssl_key` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 文件名 |
    | 默认值 | `NULL` |

    `admin_ssl_key` 系统变量类似于 `ssl_key`，不同之处在于它适用于管理连接接口而不是主连接接口。有关为管理接口配置加密支持的信息，请参阅 加密连接的管理接口支持。

+   `admin_tls_ciphersuites`

    | 命令行格式 | `--admin-tls-ciphersuites=ciphersuite_list` |
    | --- | --- |
    | 引入版本 | 8.0.21 |
    | 系统变量 | `admin_tls_ciphersuites` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 字符串 |
    | 默认值 | `NULL` |

    `admin_tls_ciphersuites` 系统变量类似于 `tls_ciphersuites`，不同之处在于它适用于管理连接接口而不是主连接接口。有关配置管理接口加密支持的信息，请参阅管理接口支持加密连接。

+   `admin_tls_version`

    | 命令行格式 | `--admin-tls-version=protocol_list` |
    | --- | --- |
    | 引入版本 | 8.0.21 |
    | 系统变量 | `admin_tls_version` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 字符串 |
    | 默认值（≥ 8.0.28） | `TLSv1.2,TLSv1.3` |
    | 默认值（≥ 8.0.21，≤ 8.0.27） | `TLSv1,TLSv1.1,TLSv1.2,TLSv1.3` |

    `admin_tls_version` 系统变量类似于 `tls_version`，不同之处在于它适用于管理连接接口而不是主连接接口。有关配置管理接口加密支持的信息，请参阅管理接口支持加密连接。

    重要

    +   从 MySQL 8.0.28 版本开始，MySQL 服务器移除了对 TLSv1 和 TLSv1.1 连接协议的支持。这些协议从 MySQL 8.0.26 版本开始被弃用。有关更多信息，请参阅移除对 TLSv1 和 TLSv1.1 协议的支持。

    +   从 MySQL 8.0.16 版本开始，MySQL 服务器支持 TLSv1.3 协议，前提是 MySQL 服务器使用 OpenSSL 1.1.1 或更高版本进行编译。服务器在启动时检查 OpenSSL 的版本，如果低于 1.1.1，则将从系统变量的默认值中移除 TLSv1.3。在这种情况下，从 MySQL 8.0.27 版本开始默认值为“`TLSv1,TLSv1.1,TLSv1.2`”，从 MySQL 8.0.28 版本开始为“`TLSv1.2`”。

+   `authentication_policy`

    | 命令行格式 | `--authentication-policy=value` |
    | --- | --- |
    | 引入版本 | 8.0.27 |
    | 系统变量 | `authentication_policy` |
    | 作用域 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 字符串 |
    | 默认值 | `*,,` |

    此变量用于管理多因素认证（MFA）功能。它适用于用于管理 MySQL 账户定义的`CREATE USER`和`ALTER USER`语句中的认证因素相关子句，其中“因素”对应于与账户关联的认证方法或插件：

    +   `authentication_policy` 控制账户可以拥有的认证因素数量。也就是说，它控制哪些因素是必需的或允许的。

    +   `authentication_policy` 还控制了每个因素允许使用的插件（或方法）。

    +   `authentication_policy`，与`default_authentication_plugin` 一起，确定了对于未明确命名插件的认证规范的默认认证插件。

    因为`authentication_policy` 仅在创建或更改账户时应用，所以对其值的更改不会影响现有用户账户。

    注意

    尽管`authentication_policy` 系统变量对`CREATE USER`和`ALTER USER`语句中的认证相关子句施加了一定的约束，但拥有`AUTHENTICATION_POLICY_ADMIN` 权限的用户不受这些约束的限制。（对于否则不允许的语句会发出警告。）

    `authentication_policy` 的值是一个由 1、2 或 3 个逗号分隔的元素组成的列表。每个存在的元素可以是一个认证插件名称、一个星号（`*`）、空值或缺失。（例外：元素 1 不能是空值或缺失。）在所有情况下，一个元素可以被空格字符包围，整个列表被单引号括起来。

    在列表中指定元素 *`N`* 的值类型对于因素 *`N`* 是否必须存在于账户定义中，以及可以使用哪些认证插件具有影响：

    +   如果元素*`N`*是身份验证插件名称，则需要因子*`N`*的身份验证方法，并且必须使用命名的插件。

        此外，该插件成为不明确命名插件的因子*`N`*身份验证方法的默认插件。有关详细信息，请参阅默认身份验证插件。

        使用内部凭证存储的身份验证插件只能指定给第一个元素，不能重复。例如，不允许以下设置：

        +   `authenication_policy = 'caching_sha2_password, sha256_password'`

        +   `authentication_policy = 'caching_sha2_password, authentication_fido, sha256_password'`

    +   如果元素*`N`*是星号（`*`），则需要因子*`N`*的身份验证方法。它可以使用任何对元素*`N`*有效的身份验证插件（稍后描述）。

    +   如果元素*`N`*为空，则因子*`N`*的身份验证方法是可选的。如果给定，它可以使用任何对元素*`N`*有效的身份验证插件（稍后描述）。

    +   如果列表中缺少元素*`N`*（即值中少于*`N`*−1 个逗号），则禁止因子*`N`*的身份验证方法。例如，值为`'*'`只允许单个因子，因此对使用`CREATE USER`创建的新帐户或使用`ALTER USER`更改现有帐户的情况，强制执行单因子身份验证（1FA）。在这种情况下，这些语句不能指定因子 2 或 3 的身份验证。

    当`authentication_policy`元素命名一个身份验证插件时，元素的允许插件名称受到以下条件的约束：

    +   元素 1 必须命名一个不需要注册步骤的插件。例如，不能命名`authentication_fido`。

    +   元素 2 和 3 必须命名一个不使用内部凭证存储的插件。

        有关哪些身份验证插件使用内部凭证存储的信息，请参阅第 8.2.15 节“密码管理”。

    当`authentication_policy`元素*`N`*为`*`时，帐户定义中因子*`N`*的允许插件名称受到以下条件的约束：

    +   对于因子 1，帐户定义可以使用任何插件。对于未命名插件的身份验证规则适用于身份验证规范。请参阅默认身份验证插件。

    +   对于因素 2 和 3，帐户定义不能命名使用内部凭据存储的插件。例如，对于 '`*,*`'，'`*,*,*`'，'`*,`'，'`*,,`' `authentication_policy`设置，只允许使用内部凭据存储的插件作为第一个因素，并且不能重复。

    当`authentication_policy`元素 *`N`*为空时，帐户定义中因素 *`N`* 的允许插件名称受以下条件约束：

    +   对于因素 1，这不适用，因为元素 1 不能是空的。

    +   对于因素 2 和 3，帐户定义不能命名使用内部凭据存储的插件。

    空元素必须出现在列表的末尾，跟在非空元素之后。换句话说，第一个元素不能是空的，要么没有元素是空的，要么最后一个元素是空的，要么最后两个元素是空的。例如，值为 `',,'` 是不允许的，因为这将意味着所有因素都是可选的。这是不可能的；帐户必须至少有一个身份验证因素。

    `authentication_policy`的默认值是 `'*,,'`。这意味着帐户定义中需要因素 1，并且可以使用任何身份验证插件，而因素 2 和 3 是可选的，每个因素可以使用不使用内部凭据存储的任何身份验证插件。

    以下表格显示了一些`authentication_policy`值及每个值为创建或更改帐户所建立的策略。

    **表 7.4 示例 authentication_policy 值**

    | authentication_policy 值 | 有效策略 |
    | --- | --- |
    | `'*'` | 仅允许创建或更改具有一个因素的帐户。 |
    | `'*,*'` | 仅允许创建或更改具有两个因素的帐户。 |
    | `'*,*,*'` | 仅允许创建或更改具有三个因素的帐户。 |
    | `'*,'` | 允许创建或更改具有一或两个因素的帐户。 |
    | `'*,,'` | 允许创建或更改具有一、两或三个因素的帐户。 |
    | `'*,*,'` | 允许创建或更改具有两或三个因素的帐户。 |
    | `'*,*`auth_plugin`*'` | 允许创建或更改具有两个因素的帐户，其中第一个因素可以是任何身份验证方法，第二个因素必须是指定的插件。 |
    | `'*`auth_plugin`*,*,'` | 允许创建或更改具有两个或三个因素的帐户，其中第一个因素必须是指定的插件。 |
    | `'*`auth_plugin`*,'` | 允许创建或更改具有一或两个因素的帐户，其中第一个因素必须是指定的插件。 |
    | `'*`auth_plugin`*,*`auth_plugin`*,*`auth_plugin`*'` | 允许创建或更改具有三个因素的帐户，其中因素必须使用指定的插件。 |
    | authentication_policy 值 | 有效策略 |

+   `authentication_windows_log_level`

    | 命令行格式 | `--authentication-windows-log-level=#` |
    | --- | --- |
    | 系统变量 | `authentication_windows_log_level` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `2` |
    | 最小值 | `0` |
    | 最大值 | `4` |

    此变量仅在启用`authentication_windows` Windows 身份验证插件并启用调试代码时可用。请参阅第 8.4.1.6 节，“Windows 可插拔身份验证”。

    此变量设置 Windows 身份验证插件的日志级别。下表显示了允许的值。

    | 值 | 描述 |
    | --- | --- |
    | 0 | 不记录 |
    | 1 | 仅记录错误消息 |
    | 2 | 记录级别 1 的消息和警告消息 |
    | 3 | 记录级别 2 的消息和信息注释 |
    | 4 | 记录级别 3 的消息和调试消息 |

+   `authentication_windows_use_principal_name`

    | 命令行格式 | `--authentication-windows-use-principal-name[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `authentication_windows_use_principal_name` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `ON` |

    此变量仅在启用`authentication_windows` Windows 身份验证插件时可用。请参阅第 8.4.1.6 节，“Windows 可插拔身份验证”。

    使用`InitSecurityContext()`函数进行身份验证的客户端应提供一个标识其连接的服务的字符串（*`targetName`*）。MySQL 使用服务器正在运行的帐户的主体名称（UPN）。UPN 的形式为`*`user_id`*@*`computer_name`*`，无需在任何地方注册即可使用。此 UPN 在身份验证握手开始时由服务器发送。

    此变量控制服务器是否在初始挑战中发送 UPN。默认情况下，该变量已启用。出于安全原因，可以禁用该变量以避免将服务器的帐户名称以明文形式发送给客户端。如果禁用该变量，服务器始终在第一个挑战中发送一个`0x00`字节，客户端不��定*`targetName`*，结果使用 NTLM 身份验证。

    如果服务器无法获取其 UPN（主要发生在不支持 Kerberos 身份验证的环境中），服务器不会发送 UPN，并使用 NTLM 身份验证。

+   `autocommit`

    | 命令行格式 | `--autocommit[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `autocommit` |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `ON` |

    自动提交模式。如果设置为 1，则对表的所有更改立即生效。如果设置为 0，则必须使用`COMMIT`接受事务或使用`ROLLBACK`取消事务。如果`autocommit`为 0，并将其更改为 1，则 MySQL 会自动执行任何未完成的事务的`COMMIT`。开始事务的另一种方法是使用`START TRANSACTION`或`BEGIN`语句。参见第 15.3.1 节，“START TRANSACTION, COMMIT, and ROLLBACK Statements”。

    默认情况下，客户端连接时`autocommit`设置为 1。要使客户端以默认值 0 开始，通过使用`--autocommit=0`选项启动服务器来设置全局`autocommit`值。要使用选项文件设置变量，请包含以下行：

    ```sql
    [mysqld]
    autocommit=0
    ```

+   `automatic_sp_privileges`

    | 命令行格式 | `--automatic-sp-privileges[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `automatic_sp_privileges` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `ON` |

    当此变量的值为 1（默认值）时，如果用户无法执行、修改或删除存储过程，则服务器会自动授予存储过程的创建者`EXECUTE`和`ALTER ROUTINE`权限。（删除存储过程需要`ALTER ROUTINE`权限。）服务器在删除存储过程时也会自动从创建者那里撤销这些权限。如果`automatic_sp_privileges`为 0，则服务器不会自动添加或删除这些权限。

    例程的创建者是用于执行其`CREATE`语句的帐户。这可能与例程定义中命名为`DEFINER`的帐户不同。

    如果您使用`--skip-new`启动**mysqld**，则`automatic_sp_privileges`将设置为`OFF`。

    另请参阅第 27.2.2 节，“存储过程和 MySQL 权限”。

+   `auto_generate_certs`

    | 命令行格式 | `--auto-generate-certs[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `auto_generate_certs` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `ON` |

    此变量控制服务器是否在数据目录中自动生成 SSL 密钥和证书文件，如果它们尚不存在。

    在启动时，如果启用了`auto_generate_certs`系统变量，且数据目录中缺少服务器端和客户端 SSL 证书和密钥文件，并且除了`--ssl`之外没有指定其他 SSL 选项，则服务器会自动生成这些文件。这些文件使得使用 SSL 进行安全客户端连接；请参见第 8.3.1 节，“配置 MySQL 使用加密连接”。

    有关 SSL 文件自动生成的更多信息，包括文件名和特性，请参见第 8.3.3.1 节，“使用 MySQL 创建 SSL 和 RSA 证书和密钥”

    `sha256_password_auto_generate_rsa_keys`和`caching_sha2_password_auto_generate_rsa_keys`系统变量相关，但控制着在未加密连接上使用 RSA 进行安全密码交换所需的 RSA 密钥对文件的自动生成。

+   `avoid_temporal_upgrade`

    | 命令行格式 | `--avoid-temporal-upgrade[={OFF&#124;ON}]` |
    | --- | --- |
    | 已弃用 | 是 |
    | 系统变量 | `avoid_temporal_upgrade` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    此变量控制是否将`ALTER TABLE`隐式升级为预先 5.6.4 格式的临时列（`TIME`、`DATETIME`和`TIMESTAMP`列，不支持分数秒精度）。升级这些列需要重建表，这会阻止任何可能适用于要执行的操作的快速修改。

    此变量默认情况下是禁用的。启用它会导致`ALTER TABLE`不重建临时列，从而能够利用可能的快速修改。

    此变量已被弃用；预计在未来的 MySQL 版本中将被移除。

+   `back_log`

    | 命令行格式 | `--back-log=#` |
    | --- | --- |
    | 系统变量 | `back_log` |
    | 作用范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `-1`（表示自动调整大小；不要分配此文字值） |
    | 最小值 | `1` |
    | 最大值 | `65535` |

    MySQL 可以拥有的未完成连接请求的数量。当主 MySQL 线程在很短的时间内收到很多连接请求时，这就会发挥作用。然后，主线程需要一些时间（尽管很少）来检查连接并启动一个新线程。`back_log` 值表示在 MySQL 暂时停止回答新请求之前可以堆叠多少请求。只有在预期在短时间内有大量连接时才需要增加此值。

    换句话说，这个值是用于传入 TCP/IP 连接的监听队列的大小。您的操作系统对此队列的大小有自己的限制。Unix `listen()` 系统调用的手册页面应该有更多细节。查阅您的操作系统文档以获取此变量的最大值。`back_log` 不能设置得比您的操作系统限制更高。

    默认值为`max_connections`的值，这使得允许的积压队列可以调整到最大允许的连接数。

+   `basedir`

    | 命令行格式 | `--basedir=dir_name` |
    | --- | --- |
    | 系统变量 | `basedir` |
    | 作用范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 目录名称 |
    | 默认值 | `mysqld 安装目录的父目录` |

    MySQL 安装基础目录的路径。

+   `big_tables`

    | 命令行格式 | `--big-tables[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `big_tables` |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    如果启用，服务器会将所有临时表存储在磁盘上而不是内存中。这可以防止大多数需要大型临时表的`SELECT`操作出现`表 *tbl_name* 已满`错误，但也会减慢对于只需要内存表的查询速度。

    新连接的默认值为`OFF`（使用内存临时表）。通常情况下，不应该有必要启用此变量。当由`TempTable`存储引擎（默认）管理内存中的*内部*临时表，并且`TempTable`存储引擎可以占用的最大内存量超过时，`TempTable`存储引擎开始将数据存储到磁盘上的临时文件中。当由`MEMORY`存储引擎管理内存中的临时表时，内存表会根据需要自动转换为基于磁盘的表。更多信息，请参见第 10.4.4 节，“MySQL 中的内部临时表使用”。

+   `bind_address`

    | 命令行格式 | `--bind-address=addr` |
    | --- | --- |
    | 系统变量 | `bind_address` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 字符串 |
    | 默认值 | `*` |

    MySQL 服务器监听一个或多个网络套接字以进行 TCP/IP 连接。每个套接字绑定到一个地址，但一个地址可能映射到多个网络接口。要指定服务器如何监听 TCP/IP 连接，请在服务器启动时设置`bind_address`系统变量。服务器还有一个`admin_address`系统变量，用于在专用接口上启用管理连接。参见第 7.1.12.1 节，“连接接口”。

    如果指定了`bind_address`，其值必须满足以下要求：

    +   在 MySQL 8.0.13 之前，`bind_address`接受单个地址值，该值可以指定单个非通配符 IP 地址或主机名，或允许监听多个网络接口的通配符地址格式之一（`*`，`0.0.0.0`或`::`）。

    +   截至 MySQL 8.0.13，`bind_address`接受如上所述的单个值，或逗号分隔值的列表。当变量命名多个值的列表时，每个值必须指定单个非通配符 IP 地址（IPv4 或 IPv6）或主���名。在值列表中不允许使用通配符地址格式（`*`，`0.0.0.0`或`::`）。

    +   截至 MySQL 8.0.22，地址可以包括网络命名空间指定符。

    IP 地址可以指定为 IPv4 或 IPv6 地址。对于任何主机名的值，服务器会将名称解析为 IP 地址并绑定到该地址。如果主机名解析为多个 IP 地址，则服务器将使用第一个 IPv4 地址（如果有）或否则使用第一个 IPv6 地址。

    服务器对不同类型的地址处理如下：

    +   如果地址是`*`，服务器接受所有服务器主机 IPv4 接口的 TCP/IP 连接，并且如果服务器主机支持 IPv6，则在所有 IPv6 接口上也接受连接。使用此地址允许在所有服务器接口上同时进行 IPv4 和 IPv6 连接。这是默认值。如果变量指定了多个值的列表，则不允许此值。

    +   如果地址是`0.0.0.0`，服务器接受所有服务器主机 IPv4 接口的 TCP/IP 连接。如果变量指定了多个值的列表，则不允许此值。

    +   如果地址是`::`，服务器接受所有服务器主机 IPv4 和 IPv6 接口的 TCP/IP 连接。如果变量指定了多个值的列表，则不允许此值。

    +   如果地址是 IPv4 映射地址，则服务器接受该地址的 TCP/IP 连接，无论是 IPv4 还是 IPv6 格式。例如，如果服务器绑定到`::ffff:127.0.0.1`，客户端可以使用`--host=127.0.0.1`或`--host=::ffff:127.0.0.1`进行连接。

    +   如果地址是“常规”IPv4 或 IPv6 地址（例如`127.0.0.1`或`::1`），服务器只接受该 IPv4 或 IPv6 地址的 TCP/IP 连接。

    这些规则适用于指定地址的网络命名空间：

    +   可以为 IP 地址或主机名指定网络命名空间。

    +   不能为通配符 IP 地址指定网络命名空间。

    +   对于给定的地址，网络命名空间是可选的。如果提供，必须在地址后面紧跟`/*`ns`*`后缀指定。

    +   没有`/*`ns`*`后缀的地址使用主机系统的全局命名空间。因此，默认情况下是全局命名空间。

    +   带有`/*`ns`*`后缀的地址使用命名为*`ns`*`的命名空间。

    +   主机系统必须支持网络命名空间，并且每个命名空间必须事先设置好。命名不存在的命名空间会产生错误。

    +   如果变量值指定了多个地址，则可以包括全局命名空间中的地址，命名空间中的地址，或两者混合。

    有关网络命名空间的其他信息，请参见第 7.1.14 节，“网络命名空间支持”。

    如果绑定到任何地址失败，服务器将产生错误并且不会启动。

    示例:

    +   `bind_address=*`

        服务器监听所有 IPv4 或 IPv6 地址，如`*`通配符所指定。

    +   `bind_address=198.51.100.20`

        服务器仅监听`198.51.100.20` IPv4 地址。

    +   `bind_address=198.51.100.20,2001:db8:0:f101::1`

        服务器监听`198.51.100.20` IPv4 地址和`2001:db8:0:f101::1` IPv6 地址。

    +   `bind_address=198.51.100.20,*`

        这会产生错误，因为当`bind_address`指定多个值的列表时，不允许使用通配符地址。

    +   `bind_address=198.51.100.20/red,2001:db8:0:f101::1/blue,192.0.2.50`

        服务器在`red`命名空间中的`198.51.100.20` IPv4 地址上监听，在`blue`命名空间中的`2001:db8:0:f101::1` IPv6 地址上监听，在全局命名空间中的`192.0.2.50` IPv4 地址上监听。

    当`bind_address`指定单个值（通配符或非通配符）时，服务器监听单个套接字，对于通配符地址可能绑定到多个网络接口。当`bind_address`指定多个值的列表时，服务器对每个值监听一个套接字，每个套接字绑定到单个网络接口。套接字数量与指定的值数量成正比。根据操作系统的连接接受效率，长值列表可能会因接受 TCP/IP 连接而产生性能损失。

    因为为监听套接字和网络命名空间文件分配文件描述符，可能需要增加`open_files_limit`系统变量。

    如果您打算将服务器绑定到特定地址，请确保`mysql.user`系统表中包含一个具有管理权限的帐户，以便您可以用来连接到该地址。否则，您将无法关闭服务器。例如，如果您将服务器绑定到`*`，您可以使用所有现有帐户连接到它。但是，如果您将服务器绑定到`::1`，它只接受该地址上的连接。在这种情况下，请首先确保`mysql.user`表中存在`'root'@'::1'`帐户，以便您仍然可以连接到服务器关闭它。

+   `block_encryption_mode`

    | 命令行格式 | `--block-encryption-mode=#` |
    | --- | --- |
    | 系统变量 | `block_encryption_mode` |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 字符串 |
    | 默认值 | `aes-128-ecb` |

    此变量控制块加密模式，适用于基于块的算法，如 AES。它影响`AES_ENCRYPT()`和`AES_DECRYPT()`的加密。

    `block_encryption_mode`采用`aes-*`keylen`*-*`mode`*`格式的值，其中*`keylen`*是以位为单位的密钥长度，*`mode`*是加密模式。该值不区分大小写。允许的*`keylen`*值为 128、192 和 256。允许的*`mode`*值为`ECB`、`CBC`、`CFB1`、`CFB8`、`CFB128`和`OFB`。

    例如，此语句使 AES 加密函数使用 256 位的密钥长度和 CBC 模式：

    ```sql
    SET block_encryption_mode = 'aes-256-cbc';
    ```

    尝试将`block_encryption_mode`设置为包含不受支持的密钥长度或 SSL 库不支持的模式的值时会出现错误。

+   `build_id`

    | 引入版本 | 8.0.31 |
    | --- | --- |
    | 系统变量 | `build_id` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 特定平台 | Linux |

    这是一个 160 位的`SHA1`签名，由链接器在 Linux 系统上编译服务器时生成（默认启用`-DWITH_BUILD_ID=ON`），并转换为十六进制字符串。这个只读值作为唯一的构建 ID，并在启动时写入服务器日志。

    `build_id`在 Linux 以外的平台上不受支持。

+   `bulk_insert_buffer_size`

    | 命令行格式 | `--bulk-insert-buffer-size=#` |
    | --- | --- |
    | 系统变量 | `bulk_insert_buffer_size` |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 是 |
    | 类型 | 整数 |
    | 默认值 | `8388608` |
    | 最小值 | `0` |
    | 最大值（64 位平台） | `18446744073709551615` |
    | 最大值（32 位平台） | `4294967295` |
    | 单位 | 字节/线程 |

    `MyISAM`使用特殊的类似树状的缓存，使得在向非空表添加数据时，对于`INSERT ... SELECT`、`INSERT ... VALUES (...), (...), ...`和`LOAD DATA`进行批量插入更快。此变量限制每个线程的缓存树的大小（以字节为单位）。将其设置为 0 将禁用此优化。默认值为 8MB。

    截至 MySQL 8.0.14 版本，设置此系统变量的会话值是一项受限操作。会话用户必须具有足够的特权来设置受限会话变量。请参阅第 7.1.9.1 节，“系统变量特权”。

+   `caching_sha2_password_digest_rounds`

    | 命令行格式 | `--caching-sha2-password-digest-rounds=#` |
    | --- | --- |
    | 引入版本 | 8.0.24 |
    | 系统变量 | `caching_sha2_password_digest_rounds` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `5000` |
    | 最小值 | `5000` |
    | 最大值 | `4095000` |

    `caching_sha2_password`身份验证插件用于密码存储的哈希轮次数量。

    将哈希轮次的数量增加到默认值以上会导致性能损失，与增加量相关：

    +   创建使用`caching_sha2_password`插件的账户对创建账户所在的客户端会话没有影响，但服务器必须执行哈希轮次以完成操作。

    +   对于使用该账户的客户端连接，服务器必须执行哈希轮次并将结果保存在缓存中。第一个客户端连接的登录时间会更长，但对于后续连接则不会。此行为发生在每次服务器重新启动之后。

+   `caching_sha2_password_auto_generate_rsa_keys`

    | 命令行格式 | `--caching-sha2-password-auto-generate-rsa-keys[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `caching_sha2_password_auto_generate_rsa_keys` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `ON` |

    服务器使用此变量来确定是否在数据目录中自动生成 RSA 私钥/公钥对文件（如果尚不存在）。

    在启动时，如果满足以下所有条件，服务器会自动生成数据目录中的 RSA 私钥/公钥文件：启用了`sha256_password_auto_generate_rsa_keys`或`caching_sha2_password_auto_generate_rsa_keys`系统变量；未指定 RSA 选项；数据目录中缺少 RSA 文件。这些密钥对文件使得通过 RSA 在未加密连接上进行安全密码交换成为可能，适用于由`sha256_password`或`caching_sha2_password`插件认证的帐户；请参见第 8.4.1.3 节，“SHA-256 可插拔认证”，以及第 8.4.1.2 节，“缓存 SHA-2 可插拔认证”。

    有关 RSA 文件自动生成的更多信息，包括文件名和特性，请参见第 8.3.3.1 节，“使用 MySQL 创建 SSL 和 RSA 证书和密钥”

    `auto_generate_certs`系统变量与此相关，但控制着用于使用 SSL 进行安全连接所需的 SSL 证书和密钥文件的自动生成。

+   `caching_sha2_password_private_key_path`

    | 命令行格式 | `--caching-sha2-password-private-key-path=file_name` |
    | --- | --- |
    | 系统变量 | `caching_sha2_password_private_key_path` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 文件名 |
    | 默认值 | `private_key.pem` |

    此变量指定`caching_sha2_password`认证插件的 RSA 私钥文件的路径名。如果文件被命名为相对路径，则会相对于服务器数据目录进行解释。文件必须采用 PEM 格式。

    重要提示

    因为这个文件存储了私钥，其访问模式应该受限，只有 MySQL 服务器才能读取它。

    有关`caching_sha2_password`的信息，请参见第 8.4.1.2 节，“缓存 SHA-2 可插拔认证”。

+   `caching_sha2_password_public_key_path`

    | 命令行格式 | `--caching-sha2-password-public-key-path=file_name` |
    | --- | --- |
    | 系统变量 | `caching_sha2_password_public_key_path` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 文件名 |
    | 默认值 | `public_key.pem` |

    该变量指定了`caching_sha2_password`认证插件的 RSA 公钥文件的路径名。如果文件以相对路径命名，则解释为相对于服务器数据目录的路径。该文件必须采用 PEM 格式。

    有关`caching_sha2_password`的信息，包括客户端如何请求 RSA 公钥的信息，请参阅第 8.4.1.2 节，“缓存 SHA-2 可插拔认证”。

+   `character_set_client`

    | 系统变量 | `character_set_client` |
    | --- | --- |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 字符串 |
    | 默认值 | `utf8mb4` |

    该变量用于表示客户端发送的语句的字符集。当客户端连接到服务器时，该变量的会话值是根据客户端请求的字符集设置的。（许多客户端支持`--default-character-set`选项，以明确指定字符集。另请参阅第 12.4 节，“连接字符集和校对规则”。）当客户端请求的值未知或不可用，或者服务器配置为忽略客户端请求时，全局变量的值用于设置会话值：

    +   客户端请求了服务器不认识的字符集。例如，启用日语的客户端在连接到未配置`sjis`支持的服务器时请求`sjis`。

    +   客户端来自 MySQL 4.1 之前的版本，因此不会请求字符集。

    +   **mysqld**是使用`--skip-character-set-client-handshake`选项启动的，这会导致它忽略客户端字符集配置。

    有些字符集不能用作客户端字符集。尝试将它们用作`character_set_client`值会产生错误。请参阅不允许的客户端字符集。

+   `character_set_connection`

    | 系统变量 | `character_set_connection` |
    | --- | --- |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 字符串 |
    | 默认值 | `utf8mb4` |

    用于没有字符集引导符指定的文字和数字转换为字符串时使用的字符集。有关引导符的信息，请参见第 12.3.8 节，“字符集引导符”。

+   `character_set_database`

    | 系统变量 | `character_set_database` |
    | --- | --- |
    | 作用范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 字符串 |
    | 默认值 | `utf8mb4` |
    | 脚注 | 此选项是动态的，但应仅由服务器设置。不应手动设置此变量。 |

    默认数据库使用的字符集。服务器在默认数据库更改时设置此变量。如果没有默认数据库，则该变量的值与`character_set_server`相同。

    从 MySQL 8.0.14 开始，设置此系统变量的会话值是受限制的操作。会话用户必须具有足够权限来设置受限制的会话变量。请参阅第 7.1.9.1 节，“系统变量权限”。

    全局`character_set_database`和`collation_database`系统变量已被弃用；预计它们将在 MySQL 的未来版本中被移除。

    弃用对会话`character_set_database`和`collation_database`系统变量进行赋值，并且赋值会产生警告。预计在未来的 MySQL 版本中，会话变量将变为只读（对其进行赋值将产生错误），但仍然可以访问会话变量以确定默认数据库的字符集和校对规则。

+   `character_set_filesystem`

    | 命令行格式 | `--character-set-filesystem=name` |
    | --- | --- |
    | 系统变量 | `character_set_filesystem` |
    | 作用范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 字符串 |
    | 默认值 | `binary` |

    文件系统字符集。该变量用于解释引用文件名的字符串文字，例如在`LOAD DATA`和`SELECT ... INTO OUTFILE`语句以及`LOAD_FILE()`函数中。在尝试打开文件之前，这些文件名会从`character_set_client`转换为`character_set_filesystem`。默认值为`binary`，表示不进行转换。对于允许使用多字节文件名的系统，可能需要不同的值。例如，如果系统使用 UTF-8 表示文件名，则将`character_set_filesystem`设置为`'utf8mb4'`。

    从 MySQL 8.0.14 开始，设置此系统变量的会话值是受限制的操作。会话用户必须具有足够权限来设置受限制的会话变量。参见第 7.1.9.1 节，“系统变量权限”。

+   `character_set_results`

    | 系统变量 | `character_set_results` |
    | --- | --- |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 字符串 |
    | 默认值 | `utf8mb4` |

    用于将查询结果返回给客户端的字符集。这包括结果数据，如列值，结果元数据，如列名，以及错误消息。

+   `character_set_server`

    | 命令行格式 | `--character-set-server=name` |
    | --- | --- |
    | 系统变量 | `character_set_server` |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 字符串 |
    | 默��值 | `utf8mb4` |

    服务器的默认字符集。参见第 12.15 节，“字符集配置”。如果设置了这个变量，还应该设置`collation_server`来指定字符集的排序规则。

+   `character_set_system`

    | 系统变量 | `character_set_system` |
    | --- | --- |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 字符串 |
    | 默认值 | `utf8mb3` |

    服务器用于存储标识符的字符集。该值始终为`utf8mb3`。

+   `character_sets_dir`

    | 命令行格式 | `--character-sets-dir=dir_name` |
    | --- | --- |
    | 系统变量 | `character_sets_dir` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 目录名称 |

    安装字符集的目录。参见第 12.15 节，“字符集配置”。

+   `check_proxy_users`

    | 命令行格式 | `--check-proxy-users[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `check_proxy_users` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    一些认证插件为自身实现了代理用户映射（例如，PAM 和 Windows 认证插件）。其他认证插件默认不支持代理用户。其中一些可以请求 MySQL 服务器自身根据授予的代理权限映射代理用户：`mysql_native_password`、`sha256_password`。

    如果启用了`check_proxy_users`系统变量，则服务器会为任何请求进行代理用户映射的认证插件执行代理用户映射。但是，可能还需要启用特定于插件的系统变量以利用服务器代理用户映射支持：

    +   对于`mysql_native_password`插件，请启用`mysql_native_password_proxy_users`。

    +   对于`sha256_password`插件，请启用`sha256_password_proxy_users`。

    有关用户代理的信息，请参见第 8.2.19 节，“代理用户”。

+   `collation_connection`

    | 系统变量 | `collation_connection` |
    | --- | --- |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 字符串 |

    连接字符集的排序规则。对于字面字符串的比较，`collation_connection`很重要。对于与列值进行比较的字符串，`collation_connection`并不重要，因为列有自己的排序规则，其排序规则优先级更高（参见第 12.8.4 节，“表达式中的排序强制性”）。

    在 MySQL 8.0.33 及更高版本中，使用用户定义的排序规则名称作为此变量会引发警告。

+   `collation_database`

    | 系统变量 | `collation_database` |
    | --- | --- |
    | 作用范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 字符串 |
    | 默认值 | `utf8mb4_0900_ai_ci` |
    | 注脚 | 此选项是动态的，但应仅由服务器设置。您不应手动设置此变量。 |

    默认数据库使用的排序规则。服务器在默认数据库更改时设置此变量。如果没有默认数据库，则该变量与`collation_server`具有相同的值。

    截至 MySQL 8.0.18，设置此系统变量的会话值不再是受限操作。

    全局`character_set_database`和`collation_database`系统变量已被弃用；预计它们将在 MySQL 的未来版本中被移除。

    对会话`character_set_database`和`collation_database`系统变量进行赋值已被弃用，并且赋值会产生警告。预计在 MySQL 的未来版本中，会话变量将变为只读（并且赋值将产生错误），在这个版本中仍然可以访问会话变量以确定默认数据库的字符集和排序规则。

    在 MySQL 8.0.33 及更高版本中，使用用户定义的排序规则名称作为`collation_database`会引发警告。

+   `collation_server`

    | 命令行格式 | `--collation-server=name` |
    | --- | --- |
    | 系统变量 | `collation_server` |
    | 作用范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 字符串 |
    | 默认值 | `utf8mb4_0900_ai_ci` |

    服务器的默认排序规则。请参阅 第 12.15 节，“字符集配置”。

    从 MySQL 8.0.33 开始，将其设置为用户定义排序的名称��引发警告。

+   `completion_type`

    | 命令行格式 | `--completion-type=#` |
    | --- | --- |
    | 系统变量 | `completion_type` |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 枚举 |
    | 默认值 | `NO_CHAIN` |
    | 有效值 | `NO_CHAIN``CHAIN``RELEASE``0``1``2` |

    事务完成类型。此变量可以采用以下表中显示的值。该变量可以使用名称值或相应的整数值进行赋值。

    | 值 | 描述 |
    | --- | --- |
    | `NO_CHAIN`（或 0） | `COMMIT` 和 `ROLLBACK` 不受影响。这是默认值。 |
    | `CHAIN`（或 1） | `COMMIT` 和 `ROLLBACK` 等效于 `COMMIT AND CHAIN` 和 `ROLLBACK AND CHAIN`，分别。（新事务立即以刚终止事务的隔离级别开始。） |
    | `RELEASE`（或 2） | `COMMIT` 和 `ROLLBACK` 等效于 `COMMIT RELEASE` 和 `ROLLBACK RELEASE`，分别。（服务器在终止事务后断开连接。） |

    `completion_type` 影响以 `START TRANSACTION` 或 `BEGIN` 开始并以 `COMMIT` 或 `ROLLBACK` 结束的事务。它不适用于由执行 第 15.3.3 节，“导致隐式提交的语句” 中列出的语句导致的隐式提交。它也不适用于 `XA COMMIT`、`XA ROLLBACK` 或当 `autocommit=1` 时。

+   `component_scheduler.enabled`

    | 命令行格式 | `--component-scheduler.enabled[=value]` |
    | --- | --- |
    | 引入 | 8.0.34 |
    | 系统变量 | `component_scheduler.enabled` |
    | 作用域 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔 |
    | 默认值 | `ON` |

    在启动时设置为`OFF`，后台线程不会启动。任务仍然可以被调度，但直到启用`component_scheduler`后才会运行。在启动时设置为`ON`，组件将完全运行。

    还可以动态设置值以获得以下效果：

    +   `ON`启动后台线程，立即开始为队列提供服务。

    +   `OFF`表示后台线程的终止，等待其结束。后台线程在访问队列以检查要执行的任务之前会检查终止标志。

+   `concurrent_insert`

    | 命令行格式 | `--concurrent-insert[=value]` |
    | --- | --- |
    | 系统变量 | `concurrent_insert` |
    | 作用域 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 枚举 |
    | 默认值 | `AUTO` |
    | 有效值 | `NEVER``AUTO``ALWAYS``0``1``2` |

    如果`AUTO`（默认值），MySQL 允许`INSERT`和`SELECT`语句同时运行，用于没有数据文件中间空闲块的`MyISAM`表。

    此变量可以采用以下表中显示的值。该变量可以使用名称值或相应的整数值进行分配。

    | 值 | 描述 |
    | --- | --- |
    | `NEVER`（或 0） | 禁用并发插入 |
    | `AUTO`（或 1） | （默认）为没有空洞的`MyISAM`表启用并发插入 |
    | `ALWAYS`（或 2） | 为所有`MyISAM`表启用并发插入，即使这些表有空洞。对于有空洞的表，如果另一个线程正在使用，则新行将插入到表的末尾。否则，MySQL 会获取正常的写锁，并将行插入到空洞中。 |

    如果使用`--skip-new`启动**mysqld**，则`concurrent_insert`设置为`NEVER`。

    另请参阅 第 10.11.3 节，“并发插入”。

+   `connect_timeout`

    | 命令行格式 | `--connect-timeout=#` |
    | --- | --- |
    | 系统变量 | `connect_timeout` |
    | 作用域 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `10` |
    | 最小值 | `2` |
    | 最大值 | `31536000` |
    | 单位 | 秒 |

    **mysqld** 服务器在响应 `Bad handshake` 之前等待连接数据包的秒数。默认值为 10 秒。

    增加 `connect_timeout` 的值可能有助于解决客户端经常遇到的错误形式为 `Lost connection to MySQL server at '*`XXX`*', system error: *`errno`*` 的问题。

+   `connection_memory_chunk_size`

    | 命令行格式 | `--connection-memory-chunk-size=#` |
    | --- | --- |
    | 引入版本 | 8.0.28 |
    | 系统变量 | `connection_memory_chunk_size` |
    | 作用范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值（≥ 8.0.34） | `8192` |
    | 默认值（≥ 8.0.28，≤ 8.0.33） | `8912` |
    | 最小值 | `0` |
    | 最大值 | `536870912` |
    | 单位 | 字节 |

    设置更新全局内存使用计数器 `Global_connection_memory` 的分块���小。仅当所有用户连接的总内存消耗变化超过此数量时，才会更新状态变量。通过设置 `connection_memory_chunk_size = 0` 来禁用更新。

    内存计算不包括任何系统用户（如 MySQL 根用户）使用的内存。`InnoDB` 缓冲池使用的内存也不包括在内。

    设置此变量需要具有 `SYSTEM_VARIABLES_ADMIN` 或 `SUPER` 权限。

+   `connection_memory_limit`

    | 命令行格式 | `--connection-memory-limit=#` |
    | --- | --- |
    | 引入版本 | 8.0.28 |
    | 系统变量 | `connection_memory_limit` |
    | 作用范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `18446744073709551615` |
    | 最小值 | `2097152` |
    | 最大值 | `18446744073709551615` |
    | 单位 | 字节 |

    设置单个用户连接可以使用的最大内存量。如果任何用户连接使用超过此数量，来自此连接的所有查询都将被拒绝，并显示 `ER_CONN_LIMIT`，包括当前正在运行的任何查询。

    此变量设置的限制不适用于系统用户或 MySQL 根帐户。`InnoDB`缓冲池使用的内存也不包括在内。

    您必须具有`SYSTEM_VARIABLES_ADMIN`或`SUPER`权限才能设置此变量。

+   `core_file`

    | 系统变量 | `core_file` |
    | --- | --- |
    | 作用范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    服务器意外退出时是否写入核心文件。此变量由`--core-file`选项设置。

+   `create_admin_listener_thread`

    | 命令行格式 | `--create-admin-listener-thread[={OFF&#124;ON}]` |
    | --- | --- |
    | 引入版本 | 8.0.14 |
    | 系统变量 | `create_admin_listener_thread` |
    | 作用范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    是否在管理网络接口上为客户端连接使用专用监听线程（请参阅第 7.1.12.1 节，“连接接口”）。默认值为`OFF`；也就是说，用于主接口上的普通连接的管理线程也处理管理接口的连接。

    根据平台类型和工作负载等因素，您可能会发现此变量的一个设置比另一个设置能够获得更好的性能。

    如果未指定`admin_address`，则设置`create_admin_listener_thread`不起作用，因为在这种情况下，服务器不维护任何管理网络接口。

+   `cte_max_recursion_depth`

    | 命令行格式 | `--cte-max-recursion-depth=#` |
    | --- | --- |
    | 系统变量 | `cte_max_recursion_depth` |
    | 作用范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `1000` |
    | 最小值 | `0` |
    | 最大值 | `4294967295` |

    公共表达式（CTE）的最大递归深度。服务器终止任何递归级别超过此变量值的 CTE 的执行。更多信息，请参见 限制公共表达式递归。

+   `datadir`

    | 命令行格式 | `--datadir=dir_name` |
    | --- | --- |
    | 系统变量 | `datadir` |
    | 作用范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 目录名称 |

    MySQL 服务器数据目录的路径。相对路径将相对于当前目录解析。如果您期望服务器自动启动（即，在您无法提前知道当前目录的情况下），最好将 `datadir` 值指定为绝对路径。

+   `debug`

    | 命令行格式 | `--debug[=debug_options]` |
    | --- | --- |
    | 系统变量 | `debug` |
    | 作用范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 字符串 |
    | 默认值（Unix） | `d:t:i:o,/tmp/mysqld.trace` |
    | 默认值（Windows） | `d:t:i:O,\mysqld.trace` |

    此变量指示当前的调试设置。仅适用于使用调试支持构建的服务器。初始值来自服务器启动时给定的 `--debug` 选项的实例值。全局和会话值可以在运行时设置。

    设置此系统变量的会话值是受限制的操作。会话用户必须具有足够权限来设置受限制的会话变量。请参见 第 7.1.9.1 节，“系统变量权限”。

    分配以 `+` 或 `-` 开头的值会将该值添加到或从当前值中减去：

    ```sql
    mysql> SET debug = 'T';
    mysql> SELECT @@debug;
    +---------+
    | @@debug |
    +---------+
    | T       |
    +---------+

    mysql> SET debug = '+P';
    mysql> SELECT @@debug;
    +---------+
    | @@debug |
    +---------+
    | P:T     |
    +---------+

    mysql> SET debug = '-P';
    mysql> SELECT @@debug;
    +---------+
    | @@debug |
    +---------+
    | T       |
    +---------+
    ```

    更多信息，请参见 第 7.9.4 节，“DBUG 包”。

+   `debug_sync`

    | 系统变量 | `debug_sync` |
    | --- | --- |
    | 作用范围 | 会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 字符串 |

    此变量是 Debug Sync 功能的用户界面。使用 Debug Sync 需要将 MySQL 配置为具有`-DWITH_DEBUG=ON` **CMake**选项（请参见第 2.8.7 节，“MySQL 源配置选项”）；否则，此系统变量将不可用。

    全局变量值是只读的，并指示该功能是否已启用。默认情况下，Debug Sync 已禁用，`debug_sync`的值为`OFF`。如果服务器使用`--debug-sync-timeout=*`N`*`启动，其中*`N`*是大于 0 的超时值，则 Debug Sync 已启用，并且`debug_sync`的值为`ON - current signal`，后跟信号名称。此外，*`N`*成为各个同步点的默认超时时间。

    任何用户都可以读取会话值，并且具有与全局变量相同的值。会话值可以设置以控制同步点。

    设置此系统变量的会话值是受限操作。会话用户必须具有足够权限来设置受限会话变量。请参见第 7.1.9.1 节，“系统变量权限”。

    有关 Debug Sync 功能的描述以及如何使用同步点，请参见 MySQL 内部：测试同步。

+   `default_authentication_plugin`

    | 命令行格式 | `--default-authentication-plugin=plugin_name` |
    | --- | --- |
    | 已弃用 | 8.0.27 |
    | 系统变量 | `default_authentication_plugin` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 枚举 |
    | 默认值 | `caching_sha2_password` |
    | 有效值 | `mysql_native_password``sha256_password``caching_sha2_password` |

    默认认证插件。必须是使用内部凭据存储的插件，因此允许这些值：

    +   `mysql_native_password`：使用 MySQL 本机密码；请参见第 8.4.1.1 节，“本机可插拔认证”。

    +   `sha256_password`：使用 SHA-256 密码；请参见第 8.4.1.3 节，“SHA-256 可插拔认证”。

    +   `caching_sha2_password`：使用 SHA-256 密码；请参见第 8.4.1.2 节，“缓存 SHA-2 可插拔认证”。

    有关哪些认证插件使用内部凭据存储的信息，请参阅第 8.2.15 节，“密码管理”。

    注意

    在 MySQL 8.0 中，`caching_sha2_password` 是默认的认证插件，而不是 `mysql_native_password`。有关此更改对服务器操作以及服务器与客户端和连接器兼容性的影响的信息，请参阅将 caching_sha2_password 作为首选认证插件。

    在 MySQL 8.0.27 之前，`default_authentication_plugin` 的值会影响服务器操作的这些方面：

    +   它确定服务器为由 `CREATE USER` 创建的未明确指定认证插件的新帐户分配的认证插件。

    +   对于使用以下形式语句创建的帐户，服务器将该帐户与默认认证插件关联，并根据该插件的要求对密码进行哈希处理：

        ```sql
        CREATE USER ... IDENTIFIED BY '*cleartext password*';
        ```

    从 MySQL 8.0.27 开始，引入了多因素认证，`default_authentication_plugin` 仍然被使用，但与 `authentication_policy` 系统变量结合使用，并且优先级低于它。有关详细信息，请参阅默认认证插件。由于这种降低的作用，`default_authentication_plugin` 在 MySQL 8.0.27 中已被弃用，并可能在未来的 MySQL 版本中移除。

+   `default_collation_for_utf8mb4`

    | 系统变量 | `default_collation_for_utf8mb4` |
    | --- | --- |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 枚举 |
    | 有效值 | `utf8mb4_0900_ai_ci``utf8mb4_general_ci` |

    重要

    `default_collation_for_utf8mb4` 系统变量仅供 MySQL 复制内部使用。

    该变量由服务器设置为`utf8mb4`字符集的默认排序规则。该变量的值从源复制到副本，以便副本可以正确处理源自具有不同默认排序规则的`utf8mb4`的源数据。该变量主要用于支持从 MySQL 5.7 或更早版本的复制源服务器到 MySQL 8.0 副本服务器的复制，或者从具有 MySQL 5.7 主节点和一个或多个 MySQL 8.0 次要节点的组复制。MySQL 5.7 中`utf8mb4`的默认排序规则是`utf8mb4_general_ci`，但在 MySQL 8.0 中是`utf8mb4_0900_ai_ci`。该变量在 MySQL 8.0 之前的版本中不存在，因此如果副本未收到该变量的值，则假定源自较早版本，并将值设置为先前的默认排序规则`utf8mb4_general_ci`。

    从 MySQL 8.0.18 开始，设置此系统变量的会话值不再是受限制的操作。

    默认`utf8mb4`排序规则在以下语句中使用：

    +   `SHOW COLLATION`和`SHOW CHARACTER SET`。

    +   `CREATE TABLE`和`ALTER TABLE`具有`CHARACTER SET utf8mb4`子句而没有`COLLATION`子句，无论是用于表字符集还是列字符集。

    +   `CREATE DATABASE`和`ALTER DATABASE`具有`CHARACTER SET utf8mb4`子句而没有`COLLATION`子句。

    +   任何包含形式为`_utf8mb4'*`some text`*'`的字符串文字的语句，而没有`COLLATE`子句。

    另请参阅第 12.9 节“Unicode 支持”。

+   `default_password_lifetime`

    | 命令行格式 | `--default-password-lifetime=#` |
    | --- | --- |
    | 系统变量 | `default_password_lifetime` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `0` |
    | 最小值 | `0` |
    | 最大值 | `65535` |
    | 单位 | 天 |

    该变量定义了全局自动密码过期策略。默认`default_password_lifetime`值为 0，表示禁用自动密码过期。如果`default_password_lifetime`的值是正整数*`N`*，则表示允许的密码寿命；密码必须每*`N`*天更改一次。

    全局密码过期策略可以根据需要为个别帐户使用`CREATE USER`和`ALTER USER`语句的密码过期选项进行覆盖。请参阅第 8.2.15 节，“密码管理”。

+   `default_storage_engine`

    | 命令行格式 | `--default-storage-engine=name` |
    | --- | --- |
    | 系统变量 | `default_storage_engine` |
    | 作用范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 枚举 |
    | 默认值 | `InnoDB` |

    表的默认存储引擎。请参阅第十八章，*替代存储引擎*。此变量仅设置永久表的存储引擎。要设置`TEMPORARY`表的存储引擎，请设置`default_tmp_storage_engine`系统变量。

    要查看可用和已启用的存储引擎，请使用`SHOW ENGINES`语句或查询`INFORMATION_SCHEMA` `ENGINES`表。

    如果在服务器启动时禁用默认存储引擎，则必须为永久表和`TEMPORARY`表设置不同的默认引擎，否则服务器将无法启动。

+   `default_table_encryption`

    | 命令行格式 | `--default-table-encryption[={OFF&#124;ON}]` |
    | --- | --- |
    | 引入版本 | 8.0.16 |
    | 系统变量 | `default_table_encryption` |
    | 作用范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 是 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    定义在创建模式和通用表空间时应用的默认加密设置，当未指定`ENCRYPTION`子句时。

    `default_table_encryption`变量仅适用于用户创建的模式和通用表空间。它不管理`mysql`系统表空间的加密。

    设置运行时值`default_table_encryption`需要`SYSTEM_VARIABLES_ADMIN`和`TABLE_ENCRYPTION_ADMIN`权限，或已弃用的`SUPER`权限。

    `default_table_encryption`支持`SET PERSIST`和`SET PERSIST_ONLY`语法。请参阅第 7.1.9.3 节，“持久化系统��量”。

    有关更多信息，请参阅为模式和通用表空间定义加密默认值。

+   `default_tmp_storage_engine`

    | 命令行格式 | `--default-tmp-storage-engine=name` |
    | --- | --- |
    | 系统变量 | `default_tmp_storage_engine` |
    | 作用范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 是 |
    | 类型 | 枚举 |
    | 默认值 | `InnoDB` |

    `TEMPORARY`表的默认存储引擎（使用`CREATE TEMPORARY TABLE`创建）。要设置永久表的存储引擎，请设置`default_storage_engine`系统变量。还请参阅有关该变量可能值的讨论。

    如果在服务器启动时禁用默认存储引擎，则必须将永久表和`TEMPORARY`表的默认引擎设置为不同的引擎，否则服务器将无法启动。

+   `default_week_format`

    | 命令行格式 | `--default-week-format=#` |
    | --- | --- |
    | 系统变量 | `default_week_format` |
    | 作用范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `0` |
    | 最小值 | `0` |
    | 最大值 | `7` |

    `WEEK()`函数的默认模式值。请参阅[第 14.7 节，“日期和时间函数”。

+   `delay_key_write`

    | 命令行格式 | `--delay-key-write[={OFF&#124;ON&#124;ALL}]` |
    | --- | --- |
    | 系统变量 | `delay_key_write` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 枚举 |
    | 默认值 | `ON` |
    | 有效值 | `OFF``ON``ALL` |

    此变量指定如何使用延迟键写入。仅适用于 `MyISAM` 表。延迟键写入导致在写入之间不刷新键缓冲区。另请参阅 Section 18.2.1, “MyISAM 启动选项”。

    此变量可以具有以下值之一，以影响可以在 `CREATE TABLE` 语句中使用的 `DELAY_KEY_WRITE` 表选项的处理。

    | 选项 | 描述 |
    | --- | --- |
    | `OFF` | 忽略 `DELAY_KEY_WRITE`。 |
    | `ON` | MySQL 尊重在 `CREATE TABLE` 中指定的任何 `DELAY_KEY_WRITE` 选项。这是默认值。 |
    | `ALL` | 所有新打开的表都被视为已启用 `DELAY_KEY_WRITE` 选项创建的。 |

    注意

    如果将此变量设置为 `ALL`，则在表正在使用时，不应该从另一个程序（例如另一个 MySQL 服务器或 **myisamchk**）中使用 `MyISAM` 表。这样做会导致索引损坏。

    如果为表启用了 `DELAY_KEY_WRITE`，则仅在关闭表时才会刷新表的键缓冲区，而不是在每次索引更新时。这大大加快了键的写入速度，但如果您使用此功能，应该通过使用设置 `myisam_recover_options` 系统变量（例如，`myisam_recover_options='BACKUP,FORCE'`）来启动服务器以自动检查所有 `MyISAM` 表。请参阅 Section 7.1.8, “服务器系统变量” 和 Section 18.2.1, “MyISAM 启动选项”。

    如果您使用 `--skip-new` 启动 **mysqld**，`delay_key_write` 将设置为 `OFF`。

    警告

    如果您使用 `--external-locking` 启用外部锁定，对于使用延迟键写入的表，没有保护防止索引损坏。

+   `delayed_insert_limit`

    | 命令行格式 | `--delayed-insert-limit=#` |
    | --- | --- |
    | 弃用 | 是 |
    | 系统变量 | `delayed_insert_limit` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `100` |
    | 最小值 | `1` |
    | 最大值（64 位平台） | `18446744073709551615` |
    | 最大值（32 位平台） | `4294967295` |

    此系统变量已弃用（因为不支持 `DELAYED` 插入），您应该期望在将来的版本中将其移除。

+   `delayed_insert_timeout`

    | 命令行格式 | `--delayed-insert-timeout=#` |
    | --- | --- |
    | 已弃用 | 是 |
    | 系统变量 | `delayed_insert_timeout` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `300` |
    | 最小值 | `1` |
    | 最大值 | `31536000` |
    | 单位 | 秒 |

    此系统变量已被弃用（因为不支持`DELAYED`插入），您应该期待在将来的版本中将其移除。

+   `delayed_queue_size`

    | 命令行格式 | `--delayed-queue-size=#` |
    | --- | --- |
    | 已弃用 | 是 |
    | 系统变量 | `delayed_queue_size` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `1000` |
    | 最小值 | `1` |
    | 最大值（64 位平台） | `18446744073709551615` |
    | 最大值（32 位平台） | `4294967295` |

    此系统变量已被弃用（因为不支持`DELAYED`插入），您应该期待在将来的版本中将其移除。

+   `disabled_storage_engines`

    | 命令行格式 | `--disabled-storage-engines=engine[,engine]...` |
    | --- | --- |
    | 系统变量 | `disabled_storage_engines` |
    | 作用范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 字符串 |
    | 默认值 | `空字符串` |

    此变量指示哪些存储引擎不能用于创建表或表空间。例如，要防止创建新的`MyISAM`或`FEDERATED`表，请在服务器选项文件中添加以下行：

    ```sql
    [mysqld]
    disabled_storage_engines="MyISAM,FEDERATED"
    ```

    默认情况下，`disabled_storage_engines`为空（未禁用任何引擎），但可以设置为一个或多个引擎的逗号分隔列表（不区分大小写）。值中命名的任何引擎都不能用于使用`CREATE TABLE`或`CREATE TABLESPACE`创建表或表空间，并且不能与`ALTER TABLE ... ENGINE`或`ALTER TABLESPACE ... ENGINE`一起用于更改现有表或表空间的存储引擎。尝试这样做会导致`ER_DISABLED_STORAGE_ENGINE`错误。

    `disabled_storage_engines`不限制现有表的其他 DDL 语句，例如`CREATE INDEX`、`TRUNCATE TABLE`、`ANALYZE TABLE`、`DROP TABLE`或`DROP TABLESPACE`。这允许平稳过渡，以便使用已禁用引擎的现有表或表空间可以通过诸如`ALTER TABLE ... ENGINE *`permitted_engine`*`之类的手段迁移到允许的引擎。

    可以将`default_storage_engine`或`default_tmp_storage_engine`系统变量设置为已禁用的存储引擎。这可能会导致应用程序表现不稳定或失败，尽管在开发环境中识别使用已禁用引擎的应用程序可能是一种有用的技术，以便对其进行修改。

    如果服务器使用以下选项启动，则`disabled_storage_engines`被禁用且无效：`--initialize`、`--initialize-insecure`、`--skip-grant-tables`。

    注意

    设置`disabled_storage_engines`可能会导致与**mysql_upgrade**的问题。有关详细信息，请参见第 6.4.5 节，“mysql_upgrade — Check and Upgrade MySQL Tables”。

+   `disconnect_on_expired_password`

    | 命令行格式 | `--disconnect-on-expired-password[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `disconnect_on_expired_password` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `ON` |

    这个变量控制服务器如何处理具有过期密码的客户端：

    +   如果客户端指示它可以处理过期密码，那么`disconnect_on_expired_password`的值就无关紧要。服务器允许客户端连接，但将其置于沙盒模式。

    +   如果客户端未指示其能够处理过期密码，服务器将根据`disconnect_on_expired_password`的值处理客户端：

        +   如果`disconnect_on_expired_password`:被启用，服务器会断开与客户端的连接。

        +   如果`disconnect_on_expired_password`:被禁用，服务器允许客户端连接，但将其置于沙盒模式。

    有关与过期密码处理相关的客户端和服务器设置交互的更多信息，请参阅第 8.2.16 节，“服务器处理过期密码”。

+   `div_precision_increment`

    | 命令行格式 | `--div-precision-increment=#` |
    | --- | --- |
    | 系统变量 | `div_precision_increment` |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 是 |
    | 类型 | 整数 |
    | 默认值 | `4` |
    | 最小值 | `0` |
    | 最大值 | `30` |

    此变量指示在使用`/`运算符执行的除法操作的结果的小数位数增加的数量。默认值为 4。最小值和最大值分别为 0 和 30。以下示例说明了增加默认值的效果。

    ```sql
    mysql> SELECT 1/7;
    +--------+
    | 1/7    |
    +--------+
    | 0.1429 |
    +--------+
    mysql> SET div_precision_increment = 12;
    mysql> SELECT 1/7;
    +----------------+
    | 1/7            |
    +----------------+
    | 0.142857142857 |
    +----------------+
    ```

+   `dragnet.log_error_filter_rules`

    | 命令行格式 | `--dragnet.log-error-filter-rules=value` |
    | --- | --- |
    | 系统变量 | `dragnet.log_error_filter_rules` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 字符串 |
    | 默认值 | `IF prio>=INFORMATION THEN drop. IF EXISTS source_line THEN unset source_line.` |

    控制`log_filter_dragnet`错误日志过滤组件操作的过滤规则。如果未安装`log_filter_dragnet`，则`dragnet.log_error_filter_rules`不可用。如果安装了但未启用`log_filter_dragnet`，对`dragnet.log_error_filter_rules`的更改不会生效。

    默认值的效果类似于使用设置为`log_error_verbosity=2`的`log_sink_internal`过滤器执行的过滤。

    从 MySQL 8.0.12 开始，可以查询 `dragnet.Status` 状态变量，以确定最近一次对 `dragnet.log_error_filter_rules` 的赋值结果。

    在 MySQL 8.0.12 之前，运行时对 `dragnet.log_error_filter_rules` 的成功赋值会产生一个确认新值的注释：

    ```sql
    mysql> SET GLOBAL dragnet.log_error_filter_rules = 'IF prio <> 0 THEN unset prio.';
    Query OK, 0 rows affected, 1 warning (0.00 sec)

    mysql> SHOW WARNINGS\G
    *************************** 1\. row ***************************
      Level: Note
       Code: 4569
    Message: filter configuration accepted:
             SET @@GLOBAL.dragnet.log_error_filter_rules=
             'IF prio!=ERROR THEN unset prio.';
    ```

    `SHOW WARNINGS` 显示的值表示在规则集成功解析并编译成内部形式后的“反编译”规范表示。从语义上讲，这个规范形式与赋给 `dragnet.log_error_filter_rules` 的值相同，但赋值和规范值之间可能存在一些差异，如前面的例子所示：

    +   `<>` 运算符被更改为 `!=`。

    +   数字优先级为 0 的值被更改为相应的优先级符号 `ERROR`。

    +   可选空格被移除。

    更多信息，请参见 7.4.2.4 节，“错误日志过滤类型”，以及 7.5.3 节，“错误日志组件”。

+   `enterprise_encryption.maximum_rsa_key_size`

    | 命令行格式 | `--enterprise-encryption.maximum-rsa-key-size=#` |
    | --- | --- |
    | 引入版本 | 8.0.30 |
    | 系统变量 | `enterprise_encryption.maximum_rsa_key_size` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `4096` |
    | 最小值 | `2048` |
    | 最大值 | `16384` |

    此变量限制了 MySQL Enterprise Encryption 生成的 RSA 密钥的最大大小。只有安装了 MySQL Enterprise Encryption 组件 `component_enterprise_encryption`（从 MySQL 8.0.30 开始提供）才可用此变量。如果使用 `openssl_udf` 共享库提供 MySQL Enterprise Encryption 函数，则此变量不可用。

    最低设置为 2048 位，这是当前最佳实践所接受的最小 RSA 密钥长度。默认设置为 4096 位。最高设置为 16384 位。生成更长的密钥可能会消耗大量 CPU 资源，因此您可以使用此设置来限制密钥的长度，以提供符合您需求的足够安全性，并在资源使用方面取得平衡。请注意，`openssl_udf` 共享库提供的函数允许从 1024 位开始的密钥长度，并在升级到组件后，最小密钥长度大于此值。更多信息请参见 Section 8.6.2, “Configuring MySQL Enterprise Encryption”。 

+   `enterprise_encryption.rsa_support_legacy_padding`

    | 命令行格式 | `--enterprise-encryption.rsa_support_legacy_padding[={OFF&#124;ON}]` |
    | --- | --- |
    | 引入版本 | 8.0.30 |
    | 系统变量 | `enterprise_encryption.rsa_support_legacy_padding` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    此变量控制着 MySQL Enterprise Encryption 在 MySQL 8.0.30 之前使用 `openssl_udf` 共享库函数生成的加密数据和签名是否可以被 MySQL Enterprise Encryption 组件 `component_enterprise_encryption` 的函数解密或验证。该变量仅在安装了 MySQL Enterprise Encryption 组件时可用，如果使用 `openssl_udf` 共享库提供 MySQL Enterprise Encryption 函数，则不可用。

    为了使组件函数支持由传统的 `openssl_udf` 共享库函数生成的内容的解密和验证，您必须将系统变量 padding 设置为 `ON`。当设置为 `ON` 时，如果组件函数无法解密或验证内容（假设它采用了组件使用的 RSAES-OAEP 或 RSASSA-PSS 方案），它们将尝试另一种方案（假设它采用了 `openssl_udf` 共享库函数使用的 RSAES-PKCS1-v1_5 或 RSASSA-PKCS1-v1_5 方案）。当设置为 `OFF` 时，如果组件函数无法使用其正常方案解密或验证内容，则返回空输出。更多信息请参见 Section 8.6.2, “Configuring MySQL Enterprise Encryption”。

+   `end_markers_in_json`

    | 命令行格式 | `--end-markers-in-json[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `end_markers_in_json` |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 是 |
    | 类型 | 布尔 |
    | 默认值 | `OFF` |

    优化器 JSON 输出是否应添加结束标记。请参阅 MySQL 内部：end_markers_in_json 系统变量。

+   `eq_range_index_dive_limit`

    | 命令行格式 | `--eq-range-index-dive-limit=#` |
    | --- | --- |
    | 系统变量 | `eq_range_index_dive_limit` |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 是 |
    | 类型 | 整数 |
    | 默认值 | `200` |
    | 最小值 | `0` |
    | 最大值 | `4294967295` |

    此变量指示优化器在估计符合条件的行数时，当相等比较条件中存在*`col_name`*值时，应从使用索引深入切换到使用索引统计数据的相等范围数。它适用于评估具有以下等效形式之一的表达式，其中优化器使用非唯一索引查找*`col_name`*值：

    ```sql
    *col_name* IN(*val1*, ..., *valN*)
    *col_name* = *val1* OR ... OR *col_name* = *valN*
    ```

    在这两种情况下，表达式包含*`N`*个相等范围。优化器可以使用索引深入或索引统计数据进行行估计。如果`eq_range_index_dive_limit`大于 0，则优化器在存在`eq_range_index_dive_limit`个或更多相等范围时使用现有的索引统计数据而不是索引深入。因此，为了允许索引深入用于多达*`N`*个相等范围，请将`eq_range_index_dive_limit`设置为*`N`* + 1\. 要禁用使用索引统计数据并始终使用索引深入，而不考虑*`N`*，请将`eq_range_index_dive_limit`设置为 0。

    有关更多信息，请参阅多值比较的相等范围优化。

    要更新表索引统计数据以获得最佳估计，请使用`ANALYZE TABLE`。

+   `error_count`

    上一条生成消息的语句导致的错误数。此变量只读。请参阅第 15.7.7.17 节，“SHOW ERRORS Statement”。

+   `event_scheduler`

    | 命令行格式 | `--event-scheduler[=value]` |
    | --- | --- |
    | 系统变量 | `event_scheduler` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 枚举 |
    | 默认值 | `ON` |
    | 有效值 | `ON``OFF``DISABLED` |

    此变量启用或禁用，并启动或停止事件调度程序。可能的状态值为`ON`、`OFF`和`DISABLED`。关闭事件调度程序并不等同于禁用事件调度程序，后者需要将状态设置为`DISABLED`。有关此变量及其对事件调度程序操作的影响的详细讨论，请参见第 27.4.2 节，“事件调度程序配置”

+   `explain_format`

    | 命令行格式 | `--explain-format=format` |
    | --- | --- |
    | 引入版本 | 8.0.32 |
    | 系统变量 | `explain_format` |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 枚举 |
    | 默认值 | `TRADITIONAL` |
    | 有效值 | `TRADITIONAL (DEFAULT)``JSON``TREE` |

    此变量确定在显示查询执行计划时，`EXPLAIN`在没有`FORMAT`选项的情况下使用的默认输出格式。可能的值及其效果如下所示：

    +   `TRADITIONAL`：使用 MySQL 的传统基于表的输出，就好像在`EXPLAIN`语句中指定了`FORMAT=TRADITIONAL`一样。这是变量的默认值。`DEFAULT`也被支持作为`TRADITIONAL`的同义词，并且具有完全相同的效果。

        注意

        不能在`EXPLAIN`语句的`FORMAT`选项中使用`DEFAULT`。

    +   `JSON`：使用 JSON 输出格式，就好像指定了`FORMAT=JSON`一样。

    +   `TREE`：使用基于树的输出格式，就好像指定了`FORMAT=TREE`一样。

    此变量的设置也会影响`EXPLAIN ANALYZE`。为此，`DEFAULT`和`TRADITIONAL`被解释为`TREE`。如果`explain_format`的值为`JSON`，并且发出了一个没有`FORMAT`选项的`EXPLAIN ANALYZE`语句，则该语句会引发一个错误（`ER_NOT_SUPPORTED_YET`）。

    使用`EXPLAIN`或`EXPLAIN ANALYZE`的格式说明符会覆盖`explain_format`的任何设置。

    当使用此语句显示关于表列的信息时，`explain_format`系统变量对`EXPLAIN`输出没有影响。

    设置`explain_format`的会话值不需要特殊权限；在全局级别设置它需要`SYSTEM_VARIABLES_ADMIN`（或已废弃的`SUPER`权限）。请参见第 7.1.9.1 节，“系统变量权限”。

    有关更多信息和示例，请参见获取执行计划信息。

+   `explicit_defaults_for_timestamp`

    | 命令行格式 | `--explicit-defaults-for-timestamp[={OFF&#124;ON}]` |
    | --- | --- |
    | 废弃 | 是 |
    | 系统变量 | `explicit_defaults_for_timestamp` |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `开启` |

    此系统变量确定服务器是否启用默认值和`TIMESTAMP`列中`NULL`值处理的某些非标准行为。默认情况下，`explicit_defaults_for_timestamp`已启用，禁用非标准行为。禁用`explicit_defaults_for_timestamp`将导致警告。

    从 MySQL 8.0.18 开始，设置此系统变量的会话值不再是受限操作。

    如果禁用了`explicit_defaults_for_timestamp`，服务器将启用非标准行为，并处理`TIMESTAMP`列如下：

    +   未显式声明具有`NULL`属性的`TIMESTAMP`列将自动声明具有`NOT NULL`属性。允许将此类列赋值为`NULL`并将列设置为当前时间戳。*例外*：从 MySQL 8.0.22 开始，尝试将`NULL`插入声明为`TIMESTAMP NOT NULL`的生成列将被拒绝并显示错误。

    +   表中的第一个`TIMESTAMP`列，如果未显式声明具有`NULL`属性或显式的`DEFAULT`或`ON UPDATE`属性，则将自动声明具有`DEFAULT CURRENT_TIMESTAMP`和`ON UPDATE CURRENT_TIMESTAMP`属性。

    +   如果未明确声明具有`NULL`属性或明确`DEFAULT`属性的`TIMESTAMP`列不是第一个列，则将自动声明为`DEFAULT '0000-00-00 00:00:00'`（“零”时间戳）。对于未为此列指定显式值的插入行，该列将被分配为`'0000-00-00 00:00:00'`，并且不会发出警告。

        根据是否启用了严格 SQL 模式或`NO_ZERO_DATE` SQL 模式，`'0000-00-00 00:00:00'`的默认值可能无效。请注意，`TRADITIONAL` SQL 模式包括严格模式和`NO_ZERO_DATE`。请参阅第 7.1.11 节，“服务器 SQL 模式”。

    刚刚描述的非标准行为已被弃用；预计它们将在未来的 MySQL 版本中被移除。

    如果启用了`explicit_defaults_for_timestamp`，服务器将禁用非标准行为，并按以下方式处理`TIMESTAMP`列：

    +   不可能将`TIMESTAMP`列分配为`NULL`值以将其设置为当前时间戳。要分配当前时间戳，请将列设置为`CURRENT_TIMESTAMP`或类似的同义词，如`NOW()`。

    +   未明确声明具有`NOT NULL`属性的`TIMESTAMP`列将自动声明具有`NULL`属性并允许`NULL`值。为此列分配`NULL`值将其设置为`NULL`，而不是当前时间戳。

    +   具有`NOT NULL`属性声明的`TIMESTAMP`列不允许`NULL`值。对于为此列指定`NULL`的插入操作，如果启用了严格的 SQL 模式，则对于单行插入将导致错误，对于禁用严格 SQL 模式的多行插入将插入`'0000-00-00 00:00:00'`。在任何情况下，将列分配为`NULL`值不会将其设置为当前时间戳。

    +   明确声明为`NOT NULL`属性且没有显式`DEFAULT`属性的`TIMESTAMP`列将被视为没有默认值。对于未为此类列指定显式值的插入行，结果取决于 SQL 模式。如果启用了严格的 SQL 模式，则会发生错误。如果未启用严格的 SQL 模式，则该列将声明为隐式默认值`'0000-00-00 00:00:00'`，并出现警告。这类似于 MySQL 处理其他时间类型（如`DATETIME`）的方式。

    +   不会自动使用`DEFAULT CURRENT_TIMESTAMP`或`ON UPDATE CURRENT_TIMESTAMP`属性声明`TIMESTAMP`列。这些属性必须明确指定。

    +   表中的第一个`TIMESTAMP`列与第一个之后的`TIMESTAMP`列处理方式相同。

    如果在服务器启动时禁用了`explicit_defaults_for_timestamp`，则此警告将出现在错误日志中：

    ```sql
    [Warning] TIMESTAMP with implicit DEFAULT value is deprecated.
    Please use --explicit_defaults_for_timestamp server option (see
    documentation for more details).
    ```

    如警告所示，要禁用已弃用的非标准行为，请在服务器启动时启用`explicit_defaults_for_timestamp`系统变量。

    注意

    `explicit_defaults_for_timestamp`本身已被弃用，因为它的唯一目的是允许控制将在未来的 MySQL 版本中删除的已弃用的`TIMESTAMP`行为。当删除这些行为时，预计`explicit_defaults_for_timestamp`也将被删除。

    更多信息，请参阅第 13.2.5 节“TIMESTAMP 和 DATETIME 的自动初始化和更新”。

+   `external_user`

    | 系统变量 | `external_user`` |
    | --- | --- |
    | 范围 | 会话 |
    | 动态 | 否 |
    | 适用的`SET_VAR`提示 | 否 |
    | 类型 | 字符串 |

    在认证过程中使用的外部用户名称，由用于认证客户端的插件设置。对于本地（内置）MySQL 认证，或者如果插件未设置该值，则此变量为`NULL`。请参阅第 8.2.19 节“代理用户”。

+   `flush`

    | 命令行格式 | `--flush[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `flush` |
    | 作用域 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    如果设置为`ON`，服务器会在每个 SQL 语句后将所有更改同步到磁盘。通常，MySQL 仅在每个 SQL 语句后将所有更改写入磁盘，并让操作系统处理同步到磁盘。请参阅 Section B.3.3.3，“如果 MySQL 一直崩溃该怎么办”。如果使用`--flush`选项启动**mysqld**，则此变量设置为`ON`。

    注意

    如果启用了`flush`，则`flush_time`的值无关紧要，对`flush_time`的更改不会影响刷新行为。

+   `flush_time`

    | 命令行格式 | `--flush-time=#` |
    | --- | --- |
    | 系统变量 | `flush_time` |
    | 作用域 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `0` |
    | 最小值 | `0` |
    | 最大值 | `31536000` |
    | 单位 | 秒 |

    如果将此值设置为非零值，则每隔`flush_time`秒关闭所有表以释放资源并将未刷新的数据同步到磁盘。此选项最好仅在资源有限的系统上使用。

    注意

    如果启用了`flush`，则`flush_time`的值无关紧要，对`flush_time`的更改不会影响刷新行为。

+   `foreign_key_checks`

    | 系统变量 | `foreign_key_checks` |
    | --- | --- |
    | 作用域 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 是 |
    | 类型 | 布尔值 |
    | 默认值 | `ON` |

    如果设置为 1（默认值），则会检查外键约束。如果设置为 0，则会忽略外键约束，但有几个例外。重新创建已删除的表时，如果表定义不符合引用该表的外键约束，则会返回错误。同样，如果外键定义形式不正确，则`ALTER TABLE`操作会返回错误。有关更多信息，请参阅 Section 15.1.20.5, “FOREIGN KEY Constraints”。

    设置此变量对`NDB`表的影响与对`InnoDB`表的影响相同。通常在正常操作期间保留此设置以强制执行引用完整性。禁用外键检查可用于以与其父/子关系所需的顺序不同的顺序重新加载`InnoDB`表。请参阅 Section 15.1.20.5, “FOREIGN KEY Constraints”。

    将`foreign_key_checks`设置为 0 也会影响数据定义语句：`DROP SCHEMA`即使包含有外键被其他模式外的表引用的表，也会删除模式，`DROP TABLE`会删除被其他表引用的具有外键的表。

    注意

    将`foreign_key_checks`设置为 1 不会触发对现有表数据的扫描。因此，在`foreign_key_checks = 0`时添加到表中的行不会验证一致性。

    不允许删除外键约束所需的索引，即使使用`foreign_key_checks=0`。在删除索引之前必须先移除外键约束。

+   `ft_boolean_syntax`

    | 命��行格式 | `--ft-boolean-syntax=name` |
    | --- | --- |
    | 系统变量 | `ft_boolean_syntax` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 字符串 |
    | 默认值 | `+ -><()~*:""&&#124;` |

    使用`IN BOOLEAN MODE`执行的布尔全文搜索支持的运算符列表。请参阅 Section 14.9.2, “Boolean Full-Text Searches”。

    默认变量值为`'+ -><()~*:""&|'`。更改值的规则如下：

    +   操作符函数由字符串中的位置确定。

    +   替换值必须是 14 个字符。

    +   每个字符必须是 ASCII 非字母数字字符。

    +   第一个或第二个字符必须是空格。

    +   不允许重复，除了第 11 和 12 位置的引用操作符。这两个字符不需要相同，但是它们是唯一可以的。

    +   位置 10、13 和 14（默认设置为`:`、`&`和`|`）保留用于未来扩展。

+   `ft_max_word_len`

    | 命令行格式 | `--ft-max-word-len=#` |
    | --- | --- |
    | 系统变量 | `ft_max_word_len` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `84` |
    | 最小值 | `10` |
    | 最大值 | `84` |

    要包含在`MyISAM` `FULLTEXT`索引中的单词的最大长度。

    注意

    在更改此变量后，必须重新构建`MyISAM`表上的`FULLTEXT`索引。使用`REPAIR TABLE *`tbl_name`* QUICK`。

+   `ft_min_word_len`

    | 命令行格式 | `--ft-min-word-len=#` |
    | --- | --- |
    | 系统变量 | `ft_min_word_len` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `4` |
    | 最小值 | `1` |
    | 最大值 | `82` |

    要包含在`MyISAM` `FULLTEXT`索引中的单词的最小长度。

    注意

    在更改此变量后，必须重新构建`MyISAM`表上的`FULLTEXT`索引。使用`REPAIR TABLE *`tbl_name`* QUICK`。

+   `ft_query_expansion_limit`

    | 命令行格式 | `--ft-query-expansion-limit=#` |
    | --- | --- |
    | 系统变量 | `ft_query_expansion_limit` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `20` |
    | 最小值 | `0` |
    | 最大值 | `1000` |

    使用`WITH QUERY EXPANSION`执行的全文搜索中使用的前几个匹配项的数量。

+   `ft_stopword_file`

    | 命令行格式 | `--ft-stopword-file=file_name` |
    | --- | --- |
    | 系统变量 | `ft_stopword_file` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 文件名 |

    用于全文搜索 `MyISAM` 表时读取停用词列表的文件。服务器会在数据目录中查找该文件，除非给定绝对路径名以指定不同目录。文件中的所有单词都会被使用；不会考虑注释。默认情况下，使用内置的停用词列表（在 `storage/myisam/ft_static.c` 文件中定义）。将此变量设置为空字符串（`''`）会禁用停用词过滤。另请参阅 Section 14.9.4, “全文停用词”。

    注意

    在更改此变量或停用词文件内容后，必须重新构建 `MyISAM` 表上的 `FULLTEXT` 索引。使用 `REPAIR TABLE *`tbl_name`* QUICK`。

+   `general_log`

    | 命令行格式 | `--general-log[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `general_log` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    是否启用一般查询日志。该值可以是 0（或 `OFF`）以禁用日志，或者是 1（或 `ON`）以启用日志。日志输出的目的地由 `log_output` 系统变量控制；如果该值为 `NONE`，即使启用了日志，也不会写入任何日志条目。

+   `general_log_file`

    | 命令行格式 | `--general-log-file=file_name` |
    | --- | --- |
    | 系统变量 | `general_log_file` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 文件名 |
    | 默认值 | `host_name.log` |

    一般查询日志文件的名称。默认值为 `*`host_name`*.log`，但初始值可以通过 `--general_log_file` 选项更改。

+   `generated_random_password_length`

    | 命令行格式 | `--generated-random-password-length=#` |
    | --- | --- |
    | 引入版本 | 8.0.18 |
    | 系统变量 | `generated_random_password_length` |
    | 作用范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `20` |
    | 最小值 | `5` |
    | 最大值 | `255` |

    为`CREATE USER`、`ALTER USER`和`SET PASSWORD`语句生成的随机密码中允许的最大字符数。更多信息，请参见随机密码生成。

+   `global_connection_memory_limit`

    | 命令行格式 | `--global-connection-memory-limit=#` |
    | --- | --- |
    | 引入版本 | 8.0.28 |
    | 系统变量 | `global_connection_memory_limit` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `18446744073709551615` |
    | 最小值 | `16777216` |
    | 最大值 | `18446744073709551615` |
    | 单位 | 字节 |

    设置所有用户连接可使用的内存总量；即`Global_connection_memory`不应超过此数量。每当超过时，所有常规用户的查询（包括当前正在运行的任何查询）都将被拒绝，并显示`ER_GLOBAL_CONN_LIMIT`。

    系统用户（如 MySQL 根用户）使用的内存包括在此总数中，但不计入断开连接限制；这些用户永远不会因内存使用而断开连接。

    `InnoDB`缓冲池使用的内存不计入总数。

    您必须具有`SYSTEM_VARIABLES_ADMIN`或`SUPER`权限才能设置此变量。

+   `global_connection_memory_tracking`

    | 命令行格式 | `--global-connection-memory-tracking={TRUE&#124;FALSE}` |
    | --- | --- |
    | 引入版本 | 8.0.28 |
    | 系统变量 | `global_connection_memory_tracking` |
    | 作用范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `FALSE` |

    确定服务器是否计算`Global_connection_memory`。必须显式启用此变量；否则，不会执行内存计算，也不会设置`Global_connection_memory`。

    您必须具有`SYSTEM_VARIABLES_ADMIN`或`SUPER`权限才能设置此变量。

+   `group_concat_max_len`

    | 命令行格式 | `--group-concat-max-len=#` |
    | --- | --- |
    | 系统变量 | `group_concat_max_len` |
    | 作用范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 是 |
    | 类型 | 整数 |
    | 默认值 | `1024` |
    | 最小值 | `4` |
    | 最大值（64 位平台） | `18446744073709551615` |
    | 最大值（32 位平台） | `4294967295` |

    `GROUP_CONCAT()`函数的最大允许结果长度（以字节为单位）。默认值为 1024。

+   `have_compress`

    如果`zlib`压缩库可用于服务器，则为`YES`，否则为`NO`。如果不可用，则无法使用`COMPRESS()`和`UNCOMPRESS()`函数。

+   `have_dynamic_loading`

    如果**mysqld**支持插件的动态加载，则为`YES`，否则为`NO`。如果值为`NO`，则无法在服务器启动时使用`--plugin-load`等选项加载插件，也无法在运行时使用`INSTALL PLUGIN`语句加载插件。

+   `have_geometry`

    如果服务器支持空间数据类型，则为`YES`，否则为`NO`。

+   `have_openssl`

    此变量是`have_ssl`的同义词。

    自 MySQL 8.0.26 起，`have_openssl`已被弃用，并可能在���来的 MySQL 版本中被移除。有关 MySQL 连接接口的 TLS 属性信息，请使用`tls_channel_status`表。

+   `have_profiling`

    如果存在语句分析功能，则为`YES`，否则为`NO`。如果存在，则`profiling`系统变量控制此功能是启用还是禁用。参见第 15.7.7.31 节，“SHOW PROFILES Statement”。

    此变量已被弃用，您应该期待它在未来的 MySQL 版本中被移除。

+   `have_query_cache`

    查询缓存在 MySQL 8.0.3 中被移除。`have_query_cache`已被弃用，始终具有值`NO`，您应该期待它在未来的 MySQL 版本中被移除。

+   `have_rtree_keys`

    如果 `RTREE` 索引可用，则为 `YES`，否则为 `NO`。（这些用于 `MyISAM` 表中的空间索引。）

+   `have_ssl`

    | 已弃用 | 8.0.26 |
    | --- | --- |
    | 系统变量 | `have_ssl` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 字符串 |
    | 有效值 | `YES`（支持 SSL）`DISABLED`（服务器编译时包含 SSL 支持，但未使用必要选项启动） |

    如果 **mysqld** 支持 SSL 连接，则为 `YES`，如果服务器编译时具有 SSL 支持，但未使用适当的连接加密选项启动，则为 `DISABLED`。有关更多信息，请参阅 Section 2.8.6, “配置 SSL 库支持”。

    截至 MySQL 8.0.26 版本，`have_ssl` 已弃用，并可能在未来的 MySQL 版本中移除。有关 MySQL 连接接口的 TLS 属性信息，请使用 `tls_channel_status` 表。

+   `have_statement_timeout`

    | 系统变量 | `have_statement_timeout` |
    | --- | --- |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |

    是否可用语句执行超时功能（请参阅语句执行时间优化提示）。如果此功能使用的后台线程无法初始化，则值可以为 `NO`。

+   `have_symlink`

    如果符号链接支持已启用，则为 `YES`，否则为 `NO`。这在 Unix 上是必需的，以支持 `DATA DIRECTORY` 和 `INDEX DIRECTORY` 表选项。如果服务器使用 `--skip-symbolic-links` 选项启动，则值为 `DISABLED`。

    此变量在 Windows 上无意义。

    注意

    符号链接支持以及控制它的 `--symbolic-links` 选项已弃用；预计将在未来的 MySQL 版本中移除。此外，默认情况下已禁用该选项。相关的 `have_symlink` 系统变量也已弃用，预计将在未来的 MySQL 版本中移除。

+   `histogram_generation_max_mem_size`

    | 命令行格式 | `--histogram-generation-max-mem-size=#` |
    | --- | --- |
    | 系统变量 | `histogram_generation_max_mem_size` |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `20000000` |
    | 最小值 | `1000000` |
    | 最大值（64 位平台） | `18446744073709551615` |
    | 最大值（32 位平台） | `4294967295` |
    | 单位 | 字节 |

    用于生成直方图统计信息的最大内存量。参见第 10.9.6 节，“优化器统计信息”，以及第 15.7.3.1 节，“ANALYZE TABLE 语句”。

    设置此系统变量的会话值是受限制的操作。会话用户必须具有足够权限来设置受限制的会话变量。参见第 7.1.9.1 节，“系统变量权限”。

+   `host_cache_size`

    | 命令行格式 | `--host-cache-size=#` |
    | --- | --- |
    | 系统变量 | `host_cache_size` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `-1`（表示自动调整大小；不要分配此文字值） |
    | 最小值 | `0` |
    | 最大值 | `65536` |

    MySQL 服务器维护一个内存中的主机缓存，其中包含客户端主机名和 IP 地址信息，用于避免域名系统（DNS）查找；参见第 7.1.12.3 节，“DNS 查找和主机缓存”。

    `host_cache_size` 变量控制主机缓存的大小，以及暴露缓存内容的性能模式 `host_cache` 表的大小。设置 `host_cache_size` 会产生以下效果：

    +   将大小设置为 0 会禁用主机缓存。在禁用缓存的情况下，服务器每次客户端连接时执行 DNS 查找。

    +   在运行时更改大小会导致隐式主机缓存刷新操作，清除主机缓存，截断 `host_cache` 表，并解除任何被阻止的主机。

    默认值自动调整为 128，再加上一个值为`max_connections`的值，最多为 500，再加上 500 中超过 20 的每个增量的 1，上限为 2000。

    使用`--skip-host-cache`选项类似于将`host_cache_size`系统变量设置为 0，但`host_cache_size`更灵活，因为它还可以用于在运行时调整、启用和禁用主机缓存，而不仅仅是在服务器启动时。 

    使用`--skip-host-cache`启动服务器不会阻止对`host_cache_size`值的运行时更改，但这些更改没有效果，即使`host_cache_size`设置大于 0，缓存也不会重新启用。

    更喜欢设置`host_cache_size`系统变量而不是`--skip-host-cache`选项，原因在前一段中给出。此外，`--skip-host-cache`选项已被弃用，预计在 MySQL 的未来版本中将被移除；在 MySQL 8.0.29 及更高版本中，使用该选项会引发警告。

+   `hostname`

    | 系统变量 | `hostname` |
    | --- | --- |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 字符串 |

    服务器在启动时将此变量设置为服务器主机名。截至 MySQL 8.0.17，最大长度为 255 个字符，根据 RFC 1034，之前为 60 个字符。

+   `identity`

    此变量是`last_insert_id`变量的同义词。它存在是为了与其他数据库系统兼容。您可以使用`SELECT @@identity`读取其值，并使用`SET identity`设置它。

+   `init_connect`

    | 命令行格式 | `--init-connect=name` |
    | --- | --- |
    | 系统变量 | `init_connect` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 字符串 |

    一个要为每个连接的客户端执行的字符串。该字符串由一个或多个 SQL 语句组成，用分号字符分隔。

    对于拥有`CONNECTION_ADMIN`权限（或已废弃的`SUPER`权限）的用户，不会执行`init_connect`的内容。这样做是为了防止`init_connect`的错误值阻止所有客户端连接。例如，该值可能包含一个语法错误的语句，从而导致客户端连接失败。不执行拥有`CONNECTION_ADMIN`或`SUPER`权限的用户的`init_connect`使他们能够打开连接并修复`init_connect`的值。

    对于密码过期的任何客户端用户，将跳过`init_connect`的执行。这是因为这样的用户无法执行任意语句，因此`init_connect`的执行失败，导致客户端无法连接。跳过`init_connect`的执行使用户能够连接并更改密码。

    服务器丢弃`init_connect`值中语句产生的任何结果集。

+   `information_schema_stats_expiry`

    | 命令行格式 | `--information-schema-stats-expiry=#` |
    | --- | --- |
    | 系统变量 | `information_schema_stats_expiry` |
    | 作用范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `86400` |
    | 最小值 | `0` |
    | 最大值 | `31536000` |
    | 单位 | 秒 |

    一些`INFORMATION_SCHEMA`表包含提供表统计信息的列：

    ```sql
    STATISTICS.CARDINALITY
    TABLES.AUTO_INCREMENT
    TABLES.AVG_ROW_LENGTH
    TABLES.CHECKSUM
    TABLES.CHECK_TIME
    TABLES.CREATE_TIME
    TABLES.DATA_FREE
    TABLES.DATA_LENGTH
    TABLES.INDEX_LENGTH
    TABLES.MAX_DATA_LENGTH
    TABLES.TABLE_ROWS
    TABLES.UPDATE_TIME
    ```

    这些列代表动态表元数据；即随着表内容变化而变化的信息。

    默认情况下，MySQL 在查询这些列时从`mysql.index_stats`和`mysql.table_stats`字典表中检索缓存的值，这比直接从存储引擎检索统计信息更有效。如果缓存的统计信息不可用或已过期，MySQL 将从存储引擎检索最新的统计信息，并将其缓存在`mysql.index_stats`和`mysql.table_stats`字典表中。随后的查询将检索缓存的统计信息，直到缓存的统计信息过期。服务器重新启动或首次打开`mysql.index_stats`和`mysql.table_stats`表不会自动更新缓存的统计信息。

    `information_schema_stats_expiry` 会话变量定义了缓存统计信息过期之前的时间段。默认值为 86400 秒（24 小时），但时间段可延长至一年。

    要随时更新给定表的缓存值，请使用 `ANALYZE TABLE`。

    要始终直接从存储引擎检索最新统计信息并绕过缓存值，请将 `information_schema_stats_expiry` 设置为 `0`。

    在以下情况下，查询统计列不会在 `mysql.index_stats` 和 `mysql.table_stats` 字典表中存储或更新统计信息：

    +   当缓存的统计信息尚未过期时。

    +   当 `information_schema_stats_expiry` 设置为 0 时。

    +   当服务器处于 `read_only`、`super_read_only`、`transaction_read_only` 或 `innodb_read_only` 模式时。

    +   当查询还获取性能模式数据时。

    在多语句事务中更新统计缓存可能在事务提交之前发生。因此，缓存可能包含与已知提交状态不符的信息。这可能发生在 `autocommit=0` 或 `START TRANSACTION` 之后。

    `information_schema_stats_expiry` 是一个会话变量，每个客户端会话都可以定义自己的过期值。从存储引擎检索并由一个会话缓存的统计信息可供其他会话使用。

    有关相关信息，请参阅 Section 10.2.3, “Optimizing INFORMATION_SCHEMA Queries”。

+   `init_file`

    | 命令行格式 | `--init-file=file_name` |
    | --- | --- |
    | 系统变量 | `init_file` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 文件名 |

    如果指定，此变量命名一个文件，其中包含在启动过程中读取和执行的 SQL 语句。在 MySQL 8.0.18 之前，每个语句必须在单独一行上，并且不应包含注释。从 MySQL 8.0.18 开始，文件中语句的可接受格式扩展以支持这些结构：

    +   `delimiter ;`，将语句分隔符设置为 `;` 字符。

    +   设置语句分隔符为`$$`字符序列的`delimiter $$`。

    +   同一行上的多个语句，由当前分隔符分隔。

    +   多行语句。

    +   从`#`字符开始到行尾的注释。

    +   从`-- `序列开始到行尾的注释。

    +   从`/*`序列开始到接下来的`*/`序列结束的 C 风格注释，可跨多行。

    +   用单引号(`'`)或双引号(`"`)字符括起来的多行字符串字面值。

    如果服务器使用`--initialize`或`--initialize-insecure`选项启动，则它将以引导模式运行，并且某些功能不可用，限制了文件中允许的语句。这些包括与帐户管理相关的语句（如`CREATE USER`或`GRANT`)、复制和全局事务标识符。参见第 19.1.3 节，“具有全局事务标识符的复制”。

    截至 MySQL 8.0.17 版本，服务器启动期间创建的线程用于诸如创建数据字典、运行升级程序和创建系统表等任务。为确保稳定和可预测的环境，这些线程使用服务器内置默认值执行某些系统变量，如`sql_mode`、`character_set_server`、`collation_server`、`completion_type`、`explicit_defaults_for_timestamp`和`default_table_encryption`。

    这些线程还用于执行在启动服务器时指定的任何文件中的语句，因此这些语句将使用服务器内置默认值执行这些系统变量。

+   `innodb_*`xxx`*`

    `InnoDB`系统变量列在第 17.14 节，“InnoDB 启动选项和系统变量”中。这些变量控制`InnoDB`表的存储、内存使用和 I/O 模式的许多方面，现在`InnoDB`是默认存储引擎，因此尤为重要。

+   `insert_id`

    在插入 `AUTO_INCREMENT` 值时，以下 `INSERT` 或 `ALTER TABLE` 语句要使用的值。这主要与二进制日志一起使用。

+   `interactive_timeout`

    | 命令行格式 | `--interactive-timeout=#` |
    | --- | --- |
    | 系统变量 | `interactive_timeout` |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `28800` |
    | 最小值 | `1` |
    | 最大值 | `31536000` |
    | 单位 | 秒 |

    服务器在关闭交互式连接之前等待交互式连接上的活动的秒数。交互式客户端被定义为使用 `CLIENT_INTERACTIVE` 选项来 `mysql_real_connect()` 的客户端。另请参见 `wait_timeout`。

+   `internal_tmp_disk_storage_engine`

    | 命令行格式 | `--internal-tmp-disk-storage-engine=#` |
    | --- | --- |
    | 已移除 | 8.0.16 |
    | 系统变量 | `internal_tmp_disk_storage_engine` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 枚举 |
    | 默认值 | `INNODB` |
    | 有效值 | `MYISAM``INNODB` |

    重要

    在 MySQL 8.0.16 及更高版本中，磁盘上的内部临时表始终使用 `InnoDB` 存储引擎；从 MySQL 8.0.16 开始，此变量已被移除，因此不再受支持。

    在 MySQL 8.0.16 之前，此变量确定用于磁盘上的内部临时表的存储引擎（请参阅磁盘上的内部临时表的存储引擎）。允许的值为 `MYISAM` 和 `INNODB`（默认值）。

+   `internal_tmp_mem_storage_engine`

    | 命令行格式 | `--internal-tmp-mem-storage-engine=#` |
    | --- | --- |
    | 系统变量 | `internal_tmp_mem_storage_engine` |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 是 |
    | 类型 | 枚举 |
    | 默认值 | `TempTable` |
    | 有效值 | `MEMORY``TempTable` |

    用于内存中的内部临时表的存储引擎（参见 Section 10.4.4, “MySQL 中的内部临时表使用”）。允许的值为 `TempTable`（默认）和 `MEMORY`。

    优化器（optimizer）使用由 `internal_tmp_mem_storage_engine` 定义的存储引擎用于内存中的内部临时表。

    从 MySQL 8.0.27 开始，为 `internal_tmp_mem_storage_engine` 配置会话设置需要 `SESSION_VARIABLES_ADMIN` 或 `SYSTEM_VARIABLES_ADMIN` 权限。

+   `join_buffer_size`

    | 命令行格式 | `--join-buffer-size=#` |
    | --- | --- |
    | 系统变量 | `join_buffer_size` |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 是 |
    | 类型 | 整数 |
    | 默认值 | `262144` |
    | 最小值 | `128` |
    | 最大值（Windows） | `4294967168` |
    | 最大值（其他，64 位平台） | `18446744073709551488` |
    | 最大值（其他，32 位平台） | `4294967168` |
    | 单位 | 字节 |
    | 块大小 | `128` |

    用于普通索引扫描、范围索引扫描和不使用索引执行全表扫描的缓冲区的最小大小。在 MySQL 8.0.18 及更高版本中，此变量还控制用于哈希连接的内存量。通常，获得快速连接的最佳方法是添加索引。增加 `join_buffer_size` 的值可以在无法添加索引时获得更快的完全连接。为每个两个表之间的完全连接分配一个连接缓冲区。对于多个表之间进行复杂连接且不使用索引的情况，��能需要多个连接缓冲区。

    默认值为 256KB。`join_buffer_size` 的最大允许设置为 4GB−1。对于 64 位平台，允许更大的值（除了 64 位 Windows，其中大值会被截断为 4GB−1 并显示警告）。块大小为 128，MySQL Server 在存储系统变量值之前会将不是块大小的整数倍的值向下舍入到下一个较低的块大小的整数倍。解析器允许值达到平台的最大无符号整数值（32 位系统为 4294967295 或 2³²−1，64 位系统为 18446744073709551615 或 2⁶⁴−1），但实际最大值是一个块大小较低的值。

    除非使用块嵌套循环或批量键访问算法，否则将缓冲区设置为大于每个匹配行所需的大小不会带来任何好处，所有连接至少分配最小大小，因此在全局设置此变量为大值时要谨慎。最好保持全局设置较小，并仅在执行大型连接的会话中将会话设置更改为较大值，或者通过使用`SET_VAR`优化提示在每个查询基础上更改设置（参见第 10.9.3 节，“优化提示”）。如果全局大小大于大多数使用它的查询所需的大小，内存分配时间可能会导致性能大幅下降。

    当使用块嵌套循环时，较大的连接缓冲区可以有益，直到第一个表中的所有行的所有必需列都存储在连接缓冲区中为止。这取决于查询；最佳大小可能小于保存第一个表的所有行。

    当使用批量键访问时，`join_buffer_size`的值定义了每个请求到存储引擎的键批的大小。缓冲区越大，对连接操作右表的顺序访问就越多，这可以显着提高性能。

    有关连接缓冲的更多信息，请参见第 10.2.1.7 节，“嵌套循环连接算法”。有关批量键访问的信息，请参见第 10.2.1.12 节，“块嵌套循环和批量键访问连接”。有关哈希连接的信息，请参见第 10.2.1.4 节，“哈希连接优化”。

+   `keep_files_on_create`

    | 命令行格式 | `--keep-files-on-create[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `keep_files_on_create` |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    如果使用没有`DATA DIRECTORY`选项创建`MyISAM`表，则`.MYD`文件将在数据库目录中创建。默认情况下，如果`MyISAM`在这种情况下找到现有的`.MYD`文件，它会覆盖它。对于使用没有`INDEX DIRECTORY`选项创建的表，`.MYI`文件也适用相同规则。要抑制此行为，请将`keep_files_on_create`变量设置为`ON`（1），在这种情况下，`MyISAM`不会覆盖现有文件，而是返回错误。默认值为`OFF`（0）。

    如果使用 `DATA DIRECTORY` 或 `INDEX DIRECTORY` 选项创建 `MyISAM` 表，并找到现有的 `.MYD` 或 `.MYI` 文件，MyISAM 总是返回错误。它不会覆盖指定目录中的文件。

+   `key_buffer_size`

    | 命令行格式 | `--key-buffer-size=#` |
    | --- | --- |
    | 系统变量 | `key_buffer_size` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `8388608` |
    | 最小值 | `0` |
    | 最大值（64 位平台） | `OS_PER_PROCESS_LIMIT` |
    | 最大值（32 位平台） | `4294967295` |
    | 单位 | 字节 |

    `MyISAM` 表的索引块被缓冲并由所有线程共享。`key_buffer_size` 是用于索引块的缓冲区的大小。关键缓冲区也称为关键缓存。

    最小允许设置为 0，但不能动态将 `key_buffer_size` 设置为 0。设置 `key_buffer_size` 为 0 会删除关键缓存，在运行时不允许。只有在启动时才允许将 `key_buffer_size` 设置为 0，此时关键缓存未初始化。在运行时将 `key_buffer_size` 设置从 0 更改为允许的非零值会初始化关键缓存。

    `key_buffer_size` 只能以 4096 字节的增量或倍数增加或减少。以非符合值增加或减少设置会产生警告，并将设置截断为符合值。

    `key_buffer_size` 在 32 位平台上的最大允许设置为 4GB−1。64 位平台允许更大的值。实际最大大小可能会更小，取决于您可用的物理 RAM 和操作系统或硬件平台强加的每进程 RAM 限制。此变量的值表示请求的内存量。在内部，服务器会分配尽可能多的内存，但实际分配量可能会更少。

    您可以增加该值以获得更好的索引处理效果，适用于主要功能是运行 MySQL 的系统，使用`MyISAM`存储引擎，该变量的合适值为机器总内存的 25%。但是，您应该意识到，如果将值设置得太大（例如，超过机器总内存的 50%），您的系统可能会开始分页并变得极其缓慢。这是因为 MySQL 依赖操作系统执行数据读取的文件系统缓存，因此您必须为文件系统缓存留出一些空间。您还应考虑除`MyISAM`之外可能使用的任何其他存储引擎的内存需求。

    要在同时写入多行时提高速度，可以使用`LOCK TABLES`。参见 Section 10.2.5.1, “Optimizing INSERT Statements”。

    您可以通过发出`SHOW STATUS`语句并检查`Key_read_requests`、`Key_reads`、`Key_write_requests`和`Key_writes`状态变量来检查关键缓冲区的性能。（参见 Section 15.7.7, “SHOW Statements”.）`Key_reads/Key_read_requests`比率通常应该小于 0.01。如果您主要使用更新和删除操作，`Key_writes/Key_write_requests`比率通常接近 1，但如果您倾向于同时更新影响多行或者使用`DELAY_KEY_WRITE`表选项，则可能会小得多。

    可以使用`key_buffer_size`与`Key_blocks_unused`状态变量以及缓冲块大小一起来确定关键缓冲区使用的比例，缓冲块大小可以从`key_cache_block_size`系统变量中获取：

    ```sql
    1 - ((Key_blocks_unused * key_cache_block_size) / key_buffer_size)
    ```

    这个值是一个近似值，因为关键缓冲区中的一些空间是为内部管理结构分配的。影响这些结构开销量的因素包括块大小和指针大小。随着块大小的增加，关键缓冲区被用于开销的百分比往往会减少。较大的块会导致读操作的数量减少（因为每次读取更多的键），但相反，会增加未被检查的键的读取次数（如果块中的所有键对查询都不相关）。

    可以创建多个`MyISAM`键缓存。4GB 的大小限制适用于每个缓存单独，而不是作为一组。参见第 10.10.2 节，“MyISAM 键缓存”。

+   `key_cache_age_threshold`

    | 命令行格式 | `--key-cache-age-threshold=#` |
    | --- | --- |
    | 系统变量 | `key_cache_age_threshold` |
    | 作用域 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `300` |
    | 最小值 | `100` |
    | 最大值（64 位平台） | `18446744073709551516` |
    | 最大值（32 位平台） | `4294967196` |
    | 块大小 | `100` |

    此值控制将缓冲区从键缓存的热子列表降级到温子列表的速度。较低的值会导致降级发生得更快。最小值为 100。默认值为 300。参见第 10.10.2 节，“MyISAM 键缓存”。

+   `key_cache_block_size`

    | 命令行格式 | `--key-cache-block-size=#` |
    | --- | --- |
    | 系统变量 | `key_cache_block_size` |
    | 作用域 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `1024` |
    | 最小值 | `512` |
    | 最大值 | `16384` |
    | 单位 | 字节 |
    | 块大小 | `512` |

    键缓存中块的大小（以字节为单位）。默认值为 1024。参见第 10.10.2 节，“MyISAM 键缓存”。

+   `key_cache_division_limit`

    | 命令行格式 | `--key-cache-division-limit=#` |
    | --- | --- |
    | 系统变量 | `key_cache_division_limit` |
    | 作用域 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `100` |
    | 最��值 | `1` |
    | 最大值 | `100` |

    键缓存缓冲区列表的热子列表和温子列表之间的分界点。该值是用于温子列表的缓冲区列表的百分比。允许的值范围从 1 到 100。默认值为 100。参见第 10.10.2 节，“MyISAM 键缓存”。

+   `large_files_support`

    | 系统变量 | `large_files_support` |
    | --- | --- |
    | 作用范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 适用 | 否 |
    | 类型 | 布尔值 |

    是否使用了大文件支持选项编译 **mysqld**。

+   `large_pages`

    | 命令行格式 | `--large-pages[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `large_pages` |
    | 作用范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 适用 | 否 |
    | 平台特定 | Linux |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    是否启用大页支持（通过 `--large-pages` 选项）。参见 Section 10.12.3.3, “Enabling Large Page Support”。

+   `large_page_size`

    | 系统变量 | `large_page_size` |
    | --- | --- |
    | 作用范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `0` |
    | 最小值 | `0` |
    | 最大值 | `65535` |
    | 单位 | 字节 |

    如果启用了大页支持，则显示内存页的大小。大内存页仅在 Linux 上受支持；在其他平台上，此变量的值始终为 0。参见 Section 10.12.3.3, “Enabling Large Page Support”。

+   `last_insert_id`

    `LAST_INSERT_ID()` 返回的值。当在更新表的语句中使用 `LAST_INSERT_ID()` 时，此值存储在二进制日志中。设置此变量不会更新 `mysql_insert_id()` C API 函数返回的值。

+   `lc_messages`

    | 命令行格式 | `--lc-messages=name` |
    | --- | --- |
    | 系统变量 | `lc_messages` |
    | 作用范围 | 全局, 会话 |
    | 动态 | 是 |
    | `SET_VAR` 适用 | 否 |
    | 类型 | 字符串 |
    | 默认值 | `en_US` |

    用于错误消息的区域设置。默认值为`en_US`。服务器将参数转换为语言名称，并与`lc_messages_dir`的值结合，生成错误消息文件的位置。参见第 12.12 节，“设置错误消息语言”。

+   `lc_messages_dir`

    | 命令行格式 | `--lc-messages-dir=dir_name` |
    | --- | --- |
    | 系统变量 | `lc_messages_dir` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 目录名称 |

    存放错误消息的目录。服务器将该值与`lc_messages`的值结合，生成错误消息文件的位置。参见第 12.12 节，“设置错误消息语言”。

+   `lc_time_names`

    | 命令行格式 | `--lc-time-names=value` |
    | --- | --- |
    | 系统变量 | `lc_time_names` |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 字符串 |

    此变量指定控制用于显示日期和月份名称和缩写的语言环境。此变量影响`DATE_FORMAT()`、`DAYNAME()`和`MONTHNAME()`函数的输出。语言环境名称是类似于`'ja_JP'`或`'pt_BR'`的 POSIX 风格值。默认值为`'en_US'`，不受系统语言环境设置的影响。有关更多信息，请参见第 12.16 节，“MySQL 服务器语言环境支持”。

+   `license`

    | 系统变量 | `license` |
    | --- | --- |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 字符串 |
    | 默认值 | `GPL` |

    服务器拥有的许可证类型。

+   `local_infile`

    | 命令行格式 | `--local-infile[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `local_infile` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    此变量控制服务器端对 `LOAD DATA` 语句的 `LOCAL` 能力。根据 `local_infile` 设置，服务器会拒绝或允许启用客户端端的 `LOCAL` 的客户端加载本地数据。

    要明确导致服务器拒绝或允许 `LOAD DATA LOCAL` 语句（无论客户端程序和库在构建时或运行时如何配置），请分别使用启用或禁用的 `local_infile` 启动 **mysqld**。`local_infile` 也可以在运行时设置。有关更多信息，请参见 第 8.1.6 节，“LOAD DATA LOCAL 的安全注意事项”。

+   `lock_wait_timeout`

    | 命令行格式 | `--lock-wait-timeout=#` |
    | --- | --- |
    | 系统变量 | `lock_wait_timeout` |
    | 作用范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 是 |
    | 类型 | 整数 |
    | 默认值 | `31536000` |
    | 最小值 | `1` |
    | 最大值 | `31536000` |
    | 单位 | 秒 |

    此变量指定尝试获取元数据锁的超时时间（以秒为单位）。允许的值范围从 1 到 31536000（1 年）。默认值为 31536000。

    此超时适用于所有使用元数据锁的语句。这些包括对表、视图、存储过程和存储函数的 DML 和 DDL 操作，以及 `LOCK TABLES`、`FLUSH TABLES WITH READ LOCK` 和 `HANDLER` 语句。

    此超时不适用于对 `mysql` 数据库中的系统表的隐式访问，例如由 `GRANT` 或 `REVOKE` 语句修改的授权表或表日志语句。超时适用于直接访问系统表的系统表，例如使用 `SELECT` 或 `UPDATE`。

    超时值分别适用于每个元数据锁尝试。给定语句可能需要多个锁，因此在报告超时错误之前，语句可能会阻塞的时间超过 `lock_wait_timeout` 值。当发生锁定超时时，会报告 `ER_LOCK_WAIT_TIMEOUT`。

    `lock_wait_timeout` 也定义了`LOCK INSTANCE FOR BACKUP` 语句在放弃锁之前等待锁的时间。

+   `locked_in_memory`

    | 系统变量 | `locked_in_memory` |
    | --- | --- |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔 |
    | 默认值 | `OFF` |

    是否使用 `--memlock` 将 **mysqld** 锁定在内存中。

+   `log_error`

    | 命令行格式 | `--log-error[=file_name]` |
    | --- | --- |
    | 系统变量 | `log_error` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 文件名 |

    默认的错误日志目的地。如果目的地是控制台，则值为 `stderr`。否则，目的地是一个文件，`log_error` 的值是文件名。请参见 7.4.2 节“错误日志”。

+   `log_error_services`

    | 命令行格式 | `--log-error-services=value` |
    | --- | --- |
    | 系统变量 | `log_error_services` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 字符串 |
    | 默认值 | `log_filter_internal; log_sink_internal` |

    启用错误日志记录的组件。该变量可以包含 0、1 或多个元素的列表。在后一种情况下，元素可以用分号或（从 MySQL 8.0.12 开始��逗号分隔，可选地跟随空格。给定设置不能同时使用分号和逗号分隔符。组件顺序很重要，因为服务器按照列出的顺序执行组件。

    从 MySQL 8.0.30 开始，如果尚未加载任何可加载（非内置）组件，则会隐式加载`log_error_services` 中命名的组件。在 MySQL 8.0.30 之前，必须首先使用 `INSTALL COMPONENT` 安装`log_error_services` 中命名的任何可加载（非内置）组件。有关更多信息，请参见 7.4.2.1 节“错误日志配置”。

+   `log_error_suppression_list`

    | 命令行格式 | `--log-error-suppression-list=value` |
    | --- | --- |
    | 引入版本 | 8.0.13 |
    | 系统变量 | `log_error_suppression_list` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` Hint Applies | 否 |
    | 类型 | 字符串 |
    | 默认值 | `空字符串` |

    `log_error_suppression_list`系统变量适用于用于错误日志的事件，并指定在出现`WARNING`或`INFORMATION`优先级时要抑制哪些事件。例如，如果某种特定类型的警告被认为是错误日志中频繁发生但不感兴趣的“噪音”，则可以将其抑制。此变量影响由默认启用的`log_filter_internal`错误日志过滤组件执行的过滤（请参阅第 7.5.3 节，“错误日志组件”）。如果禁用`log_filter_internal`，`log_error_suppression_list`将不起作用。

    `log_error_suppression_list`的值可以是空字符串表示不进行抑制，也可以是一个或多个逗号分隔的值列表，表示要抑制的错误代码。错误代码可以用符号形式或数字形式指定。数字代码可以带有或不带有`MY-`前缀。数字部分的前导零不重要。允许的代码格式示例：

    ```sql
    ER_SERVER_SHUTDOWN_COMPLETE
    MY-000031
    000031
    MY-31
    31
    ```

    符号值比数字值更易读和可移植。有关允许的错误符号和数字的信息，请参阅 MySQL 8.0 错误消息参考。

    `log_error_suppression_list`的效果与`log_error_verbosity`相结合。有关更多信息，请参阅第 7.4.2.5 节，“基于优先级的错误日志过滤（log_filter_internal）”。

+   `log_error_verbosity`

    | 命令行格式 | `--log-error-verbosity=#` |
    | --- | --- |
    | 系统变量 | `log_error_verbosity` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` Hint Applies | 否 |
    | 类型 | 整数 |
    | 默认值 | `2` |
    | 最小值 | `1` |
    | 最大值 | `3` |

    `log_error_verbosity` 系统变量指定处理意图写入错误日志的事件的详细程度。此变量影响由默认启用的 `log_filter_internal` 错误日志过滤组件执行的过滤（请参阅 Section 7.5.3, “Error Log Components”）。如果禁用了 `log_filter_internal`，`log_error_verbosity` 将不起作用。

    写入错误日志的事件具有 `ERROR`、`WARNING` 或 `INFORMATION` 的优先级。`log_error_verbosity` 控制基于哪些优先级允许写入日志的消息的详细程度，如下表所示。

    | log_error_verbosity 值 | 允许的消息优先级 |
    | --- | --- |
    | 1 | `ERROR` |
    | 2 | `ERROR`, `WARNING` |
    | 3 | `ERROR`, `WARNING`, `INFORMATION` |

    还有一个 `SYSTEM` 的优先级。关于非错误情况的系统消息将打印到错误日志中，而不管 `log_error_verbosity` 的值如何。这些消息包括启动和关闭消息，以及一些重要的设置更改。

    `log_error_verbosity` 的效果与 `log_error_suppression_list` 的效果相结合。有关更多信息，请参阅 Section 7.4.2.5, “Priority-Based Error Log Filtering (log_filter_internal)”")。

+   `log_output`

    | 命令行格式 | `--log-output=name` |
    | --- | --- |
    | 系统变量 | `log_output` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` Hint Applies | 否 |
    | 类型 | 设置 |
    | 默认值 | `FILE` |
    | 有效值 | `TABLE``FILE``NONE` |

    一般查询日志和慢查询日志输出的目标或目标。该值是一个列表，由 `TABLE`、`FILE` 和 `NONE` 中选择一个或多个逗号分隔的单词组成。`TABLE` 选择将日志记录到 `mysql` 系统模式中的 `general_log` 和 `slow_log` 表中。`FILE` 选择将日志记录到日志文件中。`NONE` 禁用日志记录。如果值中包含 `NONE`，则它优先于其他任何单词。可以同时给出 `TABLE` 和 `FILE` 以选择两个日志输出目标。

    此变量选择日志输出目的地，但不会启用日志输出。要实现这一点，请启用 `general_log` 和 `slow_query_log` 系统变量。对于 `FILE` 日志记录，`general_log_file` 和 `slow_query_log_file` 系统变量确定日志文件位置。有关更多信息，请参阅 第 7.4.1 节，“选择通用查询日志和慢查询日志输出目的地”。

+   `log_queries_not_using_indexes`

    | 命令行格式 | `--log-queries-not-using-indexes[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `log_queries_not_using_indexes` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    如果您启用了此变量并启用了慢查询日志，则会记录预计检索所有行的查询。请参阅 第 7.4.5 节，“慢查询日志”。此选项并不一定意味着不使用索引。例如，使用完整索引扫描的查询会使用索引，但会被记录，因为索引不会限制行数。

+   `log_raw`

    | 命令行格式 | `--log-raw[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量（≥ 8.0.19） | `log_raw` |
    | 作用范围（≥ 8.0.19） | 全局 |
    | 动态（≥ 8.0.19） | 是 |
    | `SET_VAR` 提示适用（≥ 8.0.19） | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    `log_raw` 系统变量最初设置为 `--log-raw` 选项的值。有关更多信息，请参阅该选项的描述。该系统变量也可以在运行时设置以更改密码屏蔽行为。

+   `log_slow_admin_statements`

    | 命令行格式 | `--log-slow-admin-statements[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `log_slow_admin_statements` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    将慢管理语句包括在写入慢查询日志的语句中。管理语句包括 `ALTER TABLE`, `ANALYZE TABLE`, `CHECK TABLE`, `CREATE INDEX`, `DROP INDEX`, `OPTIMIZE TABLE`, 和 `REPAIR TABLE`。

+   `log_slow_extra`

    | 命令行格式 | `--log-slow-extra[={OFF&#124;ON}]` |
    | --- | --- |
    | 引入 | 8.0.14 |
    | 系统变量 | `log_slow_extra` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    如果慢查询日志已启用且输出目标包括 `FILE`，服务器将向日志文件行写入附加字段，提供有关慢语句的信息。请参阅 第 7.4.5 节，“慢查询日志”。`TABLE` 输出不受影响。

+   `log_syslog`

    | 命令行格式 | `--log-syslog[={OFF&#124;ON}]` |
    | --- | --- |
    | 已弃用 | 是（在 8.0.13 中移除） |
    | 系统变量 | `log_syslog` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `ON`（当启用系统日志记录错误时） |

    在 MySQL 8.0 之前，此变量控制是否将错误日志记录到系统日志（Windows 上的事件日志和 Unix 及类 Unix 系统上的 `syslog`）。

    在 MySQL 8.0 中，`log_sink_syseventlog` 日志组件实现了将错误日志记录到系统日志（参见 第 7.4.2.8 节，“将错误记录到系统日志”），因此可以通过将该组件添加到 `log_error_services` 系统变量来启用此类日志记录。`log_syslog` 已被移除。（在 MySQL 8.0.13 之前，`log_syslog` 存在但已被弃用且无效果。）

+   `log_syslog_facility`

    | ��令行格式 | `--log-syslog-facility=value` |
    | --- | --- |
    | 已移除 | 8.0.13 |
    | 系统变量 | `log_syslog_facility` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 字符串 |
    | 默认值 | `daemon` |

    此变量在 MySQL 8.0.13 中已移除，并由 `syseventlog.facility` 替代。

+   `log_syslog_include_pid`

    | 命令行格式 | `--log-syslog-include-pid[={OFF&#124;ON}]` |
    | --- | --- |
    | 移除版本 | 8.0.13 |
    | 系统变量 | `log_syslog_include_pid` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `ON` |

    此变量在 MySQL 8.0.13 中已移除，并由 `syseventlog.include_pid` 替代。

+   `log_syslog_tag`

    | 命令行格式 | `--log-syslog-tag=tag` |
    | --- | --- |
    | 移除版本 | 8.0.13 |
    | 系统变量 | `log_syslog_tag` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 字符串 |
    | 默认值 | `空字符串` |

    此变量在 MySQL 8.0.13 中已移除，并由 `syseventlog.tag` 替代。

+   `log_timestamps`

    | 命令行格式 | `--log-timestamps=#` |
    | --- | --- |
    | 系统变量 | `log_timestamps` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 枚举 |
    | 默认值 | `UTC` |
    | 有效值 | `UTC``SYSTEM` |

    此变量控制写入错误日志、一般查询日志和慢查询日志文件中的时间戳的时区，并不影响写入表格（`mysql.general_log`、`mysql.slow_log`）的一般查询日志和慢查询日志消息的时区。从这些表中检索的行可以通过 `CONVERT_TZ()` 或设置会话 `time_zone` 系统变量将其从本地系统时区转换为任何所需的时区。

    允许的 `log_timestamps` 值为 `UTC`（默认）和 `SYSTEM`（本地系统时区）。

    时间戳使用 ISO 8601 / RFC 3339 格式编写：`*`YYYY-MM-DD`*T*`hh:mm:ss.uuuuuu`*` 加上 `Z` 表示 Zulu 时间（UTC）或 `±hh:mm`（与 UTC 的偏移）。

+   `log_throttle_queries_not_using_indexes`

    | 命令行格式 | `--log-throttle-queries-not-using-indexes=#` |
    | --- | --- |
    | 系统变量 | `log_throttle_queries_not_using_indexes` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `0` |
    | 最小值 | `0` |
    | 最大值 | `4294967295` |

    如果启用了`log_queries_not_using_indexes`，则`log_throttle_queries_not_using_indexes`变量限制每分钟可以写入慢查询日志的此类查询的数量。值为 0（默认）表示“无限制”。更多信息，请参见第 7.4.5 节，“慢查询日志”。

+   `long_query_time`

    | 命令行格式 | `--long-query-time=#` |
    | --- | --- |
    | 系统变量 | `long_query_time` |
    | 作用范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 数值 |
    | 默认值 | `10` |
    | 最小值 | `0` |
    | 最大值 | `31536000` |
    | 单位 | 秒 |

    如果一个查询花费的时间超过这么多秒，服务器会增加`Slow_queries`状态变量。如果慢查询日志已启用，则会将查询记录到慢查询日志文件中。这个值是实时测量的，而不是 CPU 时间，因此在轻负载系统上低于阈值的查询可能在重负载系统上高于阈值。`long_query_time`的最小和默认值分别为 0 和 10。最大值为 31536000，即 365 天的秒数。该值可以指定到微秒的分辨率。参见第 7.4.5 节，“慢查询日志”。

    较小的值会导致更多语句被视为长时间运行，结果是慢查询日志需要更多空间。对于非常小的值（小于一秒），日志可能在短时间内变得非常庞大。增加被视为长时间运行的语句数量也可能导致 MySQL 企业监控中“过多长时间运行进程”警报的误报，特别是如果启用了组复制。因此，非常小的值应仅在测试环境中使用，或者在生产环境中仅用于短时间。

    **mysqldump**执行完整表扫描，这意味着其查询通常会超过对于常规查询有用的`long_query_time`设置。从 MySQL 8.0.30 开始，如果要排除大部分或全部**mysqldump**的查询记录在慢查询日志中，可以设置**mysqldump**的`--mysqld-long-query-time`命令行选项以将系统变量的会话值更改为更高的值。

+   `low_priority_updates`

    | 命令行格式 | `--low-priority-updates[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `low_priority_updates` |
    | 作用域 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    如果设置为`1`，则所有`INSERT`、`UPDATE`、`DELETE`和`LOCK TABLE WRITE`语句将等待直到受影响表上没有挂起的`SELECT`或`LOCK TABLE READ`。可以使用`{INSERT | REPLACE | DELETE | UPDATE} LOW_PRIORITY ...`来降低仅一个查询的优先级来达到相同效果。此变量仅影响仅使用表级锁定的存储引擎（如`MyISAM`、`MEMORY`和`MERGE`）。参见 Section 10.11.2, “Table Locking Issues”。

    截至 MySQL 8.0.27，设置此系统变量的会话值是受限制的操作。会话用户必须具有足够权限来设置受限制的会话变量。参见 Section 7.1.9.1, “System Variable Privileges”。

+   `lower_case_file_system`

    | 系统变量 | `lower_case_file_system` |
    | --- | --- |
    | 作用域 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔值 |

    此变量描述数据目录所在文件系统中文件名的大小写敏感性。`OFF`表示文件名区分大小写，`ON`表示不区分大小写。此变量是只读的，因为它反映了文件系统属性，设置它不会对文件系统产生影响。

+   `lower_case_table_names`

    | 命令行格式 | `--lower-case-table-names[=#]` |
    | --- | --- |
    | 系统变量 | `lower_case_table_names` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值（macOS） | `2` |
    | 默认值（Unix） | `0` |
    | 默认值（Windows） | `1` |
    | 最小值 | `0` |
    | 最大值 | `2` |

    如果设置为 0，表名按指定方式存储，比较区分大小写。如果设置为 1，表名在磁盘上以小写形式存储，比较不区分大小写。如果设置为 2，表名按给定方式存储，但比较时以小写形式进行。此选项也适用于数据库名和表别名。更多细节，请参阅第 11.2.3 节，“标识符大小写敏感性”。

    此变量的默认值取决于平台（参见`lower_case_file_system`）。在 Linux 和其他类 Unix 系统上，默认值为`0`。在 Windows 上，默认值为`1`。在 macOS 上，默认值为`2`。在 Linux（和其他类 Unix 系统）上，将值设置为`2`是不支持的；服务器会强制将值改为`0`。

    如果在运行 MySQL 的系统上数据目录位于大小写不敏感的文件系统上（例如在 Windows 或 macOS 上），*不*应将`lower_case_table_names`设置为 0。这是一个不支持的组合，可能导致在使用错误的*`tbl_name`*大小写运行`INSERT INTO ... SELECT ... FROM *`tbl_name`*`操作时出现挂起条件。对于`MyISAM`，使用不同大小写访问表名可能导致索引损坏。

    如果在大小写不敏感的文件系统上尝试使用`--lower_case_table_names=0`启动服务器，将打印错误消息并退出服务器。

    此变量的设置会影响复制过滤选项的行为，涉及大小写敏感性。更多信息，请参阅第 19.2.5 节，“服务器如何评估复制过滤规则”。

    禁止使用与服务器初始化时使用的设置不同的`lower_case_table_names`设置启动服务器。这个限制是必要的，因为各种数据字典表字段使用的校对规则是在服务器初始化时定义的设置确定的，重新启动服务器使用不同设置会导致标识符排序和比较方面的不一致性。

    因此，在初始化服务器之前，有必要将`lower_case_table_names`配置为所需的设置。在大多数情况下，这需要在首次启动 MySQL 服务器之前在 MySQL 选项文件中配置`lower_case_table_names`。但是，在 Debian 和 Ubuntu 上的 APT 安装中，服务器会为您初始化，并且没有机会在之前的选项文件中配置设置。因此，在使用 APT 安装 MySQL 之前，您必须使用`debconf-set-selection`实用程序来启用`lower_case_table_names`。要这样做，请在使用 APT 安装 MySQL 之前运行以下命令：

    ```sql
    $> sudo debconf-set-selections <<< "mysql-server mysql-server/lowercase-table-names select Enabled"
    ```

    注意

    在 MySQL 8.0.17 中添加了使用`debconf-set-selections`启用`lower_case_table_names`的功能。启用`lower_case_table_names`会将值设置为 1。

+   `mandatory_roles`

    | 命令行格式 | `--mandatory-roles=value` |
    | --- | --- |
    | 系统变量 | `mandatory_roles` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 字符串 |
    | 默认值 | `空字符串` |

    服务器应将其视为强制性的角色。实际上，这些角色会自动授予每个用户，尽管设置`mandatory_roles`实际上不会更改任何用户帐户，并且授予的角色在`mysql.role_edges`系统表中不可见。

    变量值是一个逗号分隔的角色名称列表。例如：

    ```sql
    SET PERSIST mandatory_roles = '`role1`@`%`,`role2`,role3,role4@localhost';
    ```

    设置`mandatory_roles`的运行时值需要`ROLE_ADMIN`权限，以及通常需要设置全局系统变量运行时值所需的`SYSTEM_VARIABLES_ADMIN`权限（或已弃用的`SUPER`权限）。

    角色名称由`*`user_name`*@*`host_name`*`格式的用户部分和主机部分组成。如果省略主机部分，则默认为`%`。有关更多信息，请参见第 8.2.5 节，“指定角色名称”。

    `mandatory_roles`的值是一个字符串，因此如果引用了用户名和主机名，则必须以引号允许的方式编写。

    在`mandatory_roles`值中命名的角色不能通过`REVOKE`或`DROP ROLE`或`DROP USER`来撤销。

    为了防止会话默认成为系统会话，具有`SYSTEM_USER`权限的角色不能列在`mandatory_roles`系统变量的值中：

    +   如果在启动时为`mandatory_roles`分配了具有`SYSTEM_USER`权限的角色，则服务器会将消息写入错误日志并退出。

    +   如果在运行时为`mandatory_roles`分配了具有`SYSTEM_USER`权限的角色，则会发生错误，并且`mandatory_roles`的值保持不变。

    强制性角色，如显式授予的角色，直到激活（参见激活角色）才生效。在登录时，如果启用了`activate_all_roles_on_login`系统变量，则为所有授予的角色激活角色；否则，或者为其他设置为默认角色的角色。在运行时，`SET ROLE`激活角色。

    当分配给`mandatory_roles`的角色在创建时不存在，但稍后创建时可能需要特殊处理才能被视为强制性。有关详细信息，请参见定义强制性角色。

    `SHOW GRANTS`根据第 15.7.7.21 节，“SHOW GRANTS Statement”中描述的规则显示强制性角色。

+   `max_allowed_packet`

    | 命令行格式 | `--max-allowed-packet=#` |
    | --- | --- |
    | 系统变量 | `max_allowed_packet` |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `67108864` |
    | 最小值 | `1024` |
    | 最大值 | `1073741824` |
    | 单位 | 字节 |
    | 块大小 | `1024` |

    一个数据包或任何生成/中间字符串的最大大小，或者由`mysql_stmt_send_long_data()` C API 函数发送的任何参数。默认值为 64MB。

    数据包消息缓冲区初始化为`net_buffer_length`字节，但在需要时可以增长到`max_allowed_packet`字节。默认情况下，此值较小，以捕获大型（可能不正确的）数据包。

    如果您使用大型`BLOB`列或长字符串，您必须增加此值。它应该与您想要使用的最大`BLOB`一样大。`max_allowed_packet`的协议限制为 1GB。该值应为 1024 的倍数；非倍数将向下舍入到最接近的倍数。

    当您通过更改`max_allowed_packet`变量的值来更改消息缓冲区大小时，如果您的客户端程序允许，您还应该在客户端端更改缓冲区大小。客户端库内置的默认`max_allowed_packet`值为 1GB，但个别客户端程序可能会覆盖此值。例如，**mysql**和**mysqldump**分别具有 16MB 和 24MB 的默认值。它们还允许您通过在命令行或选项文件中设置`max_allowed_packet`来更改客户端端的值。

    此变量的会话值是只读的。客户端可以接收多达会话值的字节。但是，服务器不会向客户端发送比当前全局`max_allowed_packet`值更多的字节。（如果全局值在客户端连接后更改，则全局值可能小于会话值。）

+   `max_connect_errors`

    | 命令行格式 | `--max-connect-errors=#` |
    | --- | --- |
    | 系统变量 | `max_connect_errors` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `100` |
    | 最小值 | `1` |
    | 最大值（64 位平台） | `18446744073709551615` |
    | 最大值（32 位平台） | `4294967295` |

    当从主机发出的`max_connect_errors`连续连接请求未能成功连接时，服务器将阻止该主机进一步连接。 如果在前一个连接被中断后的`max_connect_errors`尝试中成功建立了来自主机的连接，则该主机的错误计数将被清零。 要解除被阻止的主机，请刷新主机缓存；请参阅 刷新主机缓存。

+   `max_connections`

    | 命令行格式 | `--max-connections=#` |
    | --- | --- |
    | 系统变量 | `max_connections` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `151` |
    | 最小值 | `1` |
    | 最大值 | `100000` |

    允许的最大同时客户端连接数。 最大有效值是`open_files_limit`的有效值 `- 810`和实际设置的`max_connections`的较小者。

    有关更多信息，请参阅 第 7.1.12.1 节，“连接接口”。

+   `max_delayed_threads`

    | 命令行格式 | `--max-delayed-threads=#` |
    | --- | --- |
    | 弃用 | 是 |
    | 系统变量 | `max_delayed_threads` |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `20` |
    | 最小值 | `0` |
    | 最大值 | `16384` |

    此系统变量已被弃用（因为不支持`DELAYED`插入）并可能在将来的 MySQL 版本中被移除。

    截至 MySQL 8.0.27，设置此系统变量的会话值是受限制的操作。 会话用户必须具有足够的权限来设置受限制的会话变量。 请参阅 第 7.1.9.1 节，“系统变量权限”。

+   `max_digest_length`

    | 命令行格式 | `--max-digest-length=#` |
    | --- | --- |
    | 系统变量 | `max_digest_length` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `1024` |
    | 最小值 | `0` |
    | 最大值 | `1048576` |
    | 单位 | 字节 |

    每个会话为计算规范化语句摘要保留的最大内存字节数。一旦在摘要计算过程中使用了该空间量，就会发生截断：来自解析语句的进一步标记不会被收集或计入其摘要值。仅在经过那么多字节的解析标记后才有所不同的语句会产生相同的规范化语句摘要，并且如果进行比较或用于摘要统计，则被视为相同。

    警告

    将`max_digest_length`设置为零会禁用摘要生成，这也会禁用需要摘要的服务器功能，例如 MySQL 企业防火墙。

    减小`max_digest_length`值会减少内存使用，但会导致更多语句的摘要值在仅在末尾有所不同的情况下变得无法区分。增加该值允许区分更长的语句，但会增加内存使用，特别是对于涉及大量同时会话的工作负载（服务器为每个会话分配`max_digest_length`字节）。

    解析器将此系统变量用作计算的规范化语句摘要的最大长度限制。如果性能模式跟踪语句摘要，则使用`performance_schema_max_digest_length`系统变量作为存储摘要的最大长度的限制。因此，如果`performance_schema_max_digest_length`小于`max_digest_length`，则在性能模式中存储的摘要值相对于原始摘要值进行了截断。

    有关语句摘要的更多信息，请参见第 29.10 节，“性能模式语句摘要和抽样”。

+   `max_error_count`

    | 命令行格式 | `--max-error-count=#` |
    | --- | --- |
    | 系统变量 | `max_error_count` |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 是 |
    | 类型 | 整数 |
    | 默认值 | `1024` |
    | 最小值 | `0` |
    | 最大值 | `65535` |

    用于显示由 `SHOW ERRORS` 和 `SHOW WARNINGS` 语句存储的错误、警告和信息消息的最大数量。这与诊断区域中的条件区域数量相同，因此可以通过 `GET DIAGNOSTICS` 检查的条件数量也相同。

    截至 MySQL 8.0.27 版本，设置此系统变量的会话值是一项受限操作。会话用户必须具有足够权限来设置受限会话变量。请参阅 第 7.1.9.1 节，“系统变量权限”。

+   `max_execution_time`

    | 命令行格式 | `--max-execution-time=#` |
    | --- | --- |
    | 系统变量 | `max_execution_time` |
    | 作用域 | 全局，会话 |
    | Dynamic | 是 |
    | `SET_VAR` 提示适用 | 是 |
    | 类型 | 整数 |
    | 默认值 | `0` |
    | 最小值 | `0` |
    | 最大值 | `4294967295` |
    | 单位 | 毫秒 |

    `SELECT` 语句的执行超时时间，单位为毫秒。如果值为 0，则未启用超时。

    `max_execution_time` 适用如下：

    +   全局 `max_execution_time` 值为新连接的会话值提供默认值。会话值适用于会话内执行的 `SELECT` 语句，其中不包括任何 `MAX_EXECUTION_TIME(*`N`*)` 优化提示，或者 *`N`* 为 0。

    +   `max_execution_time` 适用于只读 `SELECT` 语句。非只读语句是那些调用会导致数据修改的存储函数的语句。

    +   对于存储程序中的 `SELECT` 语句，将忽略 `max_execution_time`。

+   `max_heap_table_size`

    | 命令行格式 | `--max-heap-table-size=#` |
    | --- | --- |
    | 系统变量 | `max_heap_table_size` |
    | 作用域 | 全局，会话 |
    | Dynamic | 是 |
    | `SET_VAR` 提示适用 | 是 |
    | 类型 | 整数 |
    | 默认值 | `16777216` |
    | 最小值 | `16384` |
    | 最大值（64 位平台） | `18446744073709550592` |
    | 最大值（32 位平台） | `4294966272` |
    | 单位 | 字节 |
    | 块大小 | `1024` |

    此变量设置用户创建的`MEMORY`表允许增长的最大大小。该变量的值用于计算`MEMORY`表的`MAX_ROWS`值。

    设置此变量对任何现有的`MEMORY`表没有影响，除非该表使用类似`CREATE TABLE`或`ALTER TABLE`或`TRUNCATE TABLE`的语句重新创建。服务器重新启动还会将现有`MEMORY`表的最大大小设置为全局`max_heap_table_size`值。

    此变量还与`tmp_table_size`一起用于限制内部内存表的大小。请参阅第 10.4.4 节，“MySQL 中的内部临时表使用”。

    `max_heap_table_size`不会被复制。更多信息请参阅第 19.5.1.21 节，“复制和 MEMORY 表”，以及第 19.5.1.39 节，“复制和变量”。

+   `max_insert_delayed_threads`

    | 已弃用 | 是 |
    | --- | --- |
    | 系统变量 | `max_insert_delayed_threads` |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `0` |
    | 最小值 | `20` |
    | 最大值 | `16384` |

    此变量是`max_delayed_threads`的同义词。与`max_delayed_threads`一样，它已被弃用（因为不支持`DELAYED`插入）并可能在未来的 MySQL 版本中被移除。

    截至 MySQL 8.0.27 版本，设置此系统变量的会话值是受限操作。会话用户必须具有足够权限来设置受限会话变量。请参阅第 7.1.9.1 节，“系统变量权限”。

+   `max_join_size`

    | 命令行格式 | `--max-join-size=#` |
    | --- | --- |
    | 系统变量 | `max_join_size` |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 是 |
    | 类型 | 整数 |
    | 默认值 | `18446744073709551615` |
    | 最小值 | `1` |
    | 最大值 | `18446744073709551615` |

    自 MySQL 8.0.31 起，此变量表示由连接在基表中进行的最大行访问次数限制。如果服务器的估计表明必须从基表中读取的行数大于 `max_join_size`，则会拒绝该语句并显示错误。

    *MySQL 8.0.30 及更早版本*：不允许可能需要检查超过 `max_join_size` 行（对于单表语句）或行组合（对于多表语句）的语句，或者可能执行超过 `max_join_size` 次磁盘查找的语句。通过设置此值，您可以捕获未正确使用键并可能需要很长时间的语句。如果您的用户倾向于执行缺少 `WHERE` 子句的连接，执行时间长或返回数百万行的连接，请设置此值。有关更多信息，请参阅 Using Safe-Updates Mode (--safe-updates)")。

    无论 MySQL 版本如何，将此变量设置为非 `DEFAULT` 值会重置 `sql_big_selects` 的值为 `0`。如果再次设置 `sql_big_selects` 值，则 `max_join_size` 变量将被忽略。

+   `max_length_for_sort_data`

    | 命令行格式 | `--max-length-for-sort-data=#` |
    | --- | --- |
    | 已弃用 | 8.0.20 |
    | 系统变量 | `max_length_for_sort_data` |
    | 范围 | 全局, 会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 是 |
    | 类型 | 整数 |
    | 默认值 | `4096` |
    | 最小值 | `4` |
    | 最大值 | `8388608` |
    | 单位 | 字节 |

    由于优化器更改使其过时且无效，此变量自 MySQL 8.0.20 起已弃用。以前，它作为决定使用哪种 `filesort` 算法的索引值大小截止值。请参阅 Section 10.2.1.16, “ORDER BY Optimization”。

+   `max_points_in_geometry`

    | 命令行格式 | `--max-points-in-geometry=#` |
    | --- | --- |
    | 系统变量 | `max_points_in_geometry` |
    | 范围 | 全局, 会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 是 |
    | 类型 | 整数 |
    | 默认值 | `65536` |
    | 最小值 | `3` |
    | 最大值 | `1048576` |

    *`points_per_circle`* 参数的最大值，用于 `ST_Buffer_Strategy()` 函数。

+   `max_prepared_stmt_count`

    | 命令行格式 | `--max-prepared-stmt-count=#` |
    | --- | --- |
    | 系统变量 | `max_prepared_stmt_count` |
    | 作用域 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `16382` |
    | 最小值 | `0` |
    | 最大值 (≥ 8.0.18) | `4194304` |
    | 最大值 (≤ 8.0.17) | `1048576` |

    此变量限制服务器中准备语句的总数。它可用于可能存在拒绝服务攻击的环境，攻击基于准备大量语句使服务器耗尽内存。如果该值低于当前准备语句的数量，现有语句不受影响且可使用，但在当前数量降低到限制以下之前，将无法准备新语句。将该值设置为 0 将禁用准备语句。

+   `max_seeks_for_key`

    | 命令行格式 | `--max-seeks-for-key=#` |
    | --- | --- |
    | 系统变量 | `max_seeks_for_key` |
    | 作用域 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 是 |
    | 类型 | 整数 |
    | 默认值 (Windows) | `4294967295` |
    | 默认值 (其他，64 位平台) | `18446744073709551615` |
    | 默认值 (其他，32 位平台) | `4294967295` |
    | 最小值 | `1` |
    | 最大值 (Windows) | `4294967295` |
    | 最大值 (其他，64 位平台) | `18446744073709551615` |
    | 最大值 (其他，32 位平台) | `4294967295` |

    限制基于键查找行时假定的最大寻址次数。MySQL 优化器假定在通过扫描索引搜索匹配行时，不需要超过这个数量的键查找，无论索引的实际基数如何（参见 Section 15.7.7.22, “SHOW INDEX Statement”）。通过将其设置为一个较低的值（比如 100），您可以强制 MySQL 优先选择索引而不是表扫描。

+   `max_sort_length`

    | 命令行格式 | `--max-sort-length=#` |
    | --- | --- |
    | 系统变量 | `max_sort_length` |
    | 作用域 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 是 |
    | 类型 | 整数 |
    | 默认值 | `1024` |
    | 最小值 | `4` |
    | 最大值 | `8388608` |
    | 单位 | 字节 |

    用于对使用`PAD SPACE`排序规则的字符串值进行排序时要使用的字节数。服务器仅使用任何此类值的前`max_sort_length`字节，并忽略其余部分。因此，仅在第一个`max_sort_length`字节之后有差异的值在`GROUP BY`、`ORDER BY`和`DISTINCT`操作中被视为相等。（此行为与 MySQL 的先前版本不同，在先前版本中，此设置适用于所有用于比较的值。）

    增加`max_sort_length`的值可能需要同时增加`sort_buffer_size`的值。详情请参见 Section 10.2.1.16, “ORDER BY Optimization”

+   `max_sp_recursion_depth`

    | 命令行格式 | `--max-sp-recursion-depth[=#]` |
    | --- | --- |
    | 系统变量 | `max_sp_recursion_depth` |
    | 作用范围 | 全局, 会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `0` |
    | 最小值 | `0` |
    | 最大值 | `255` |

    任何给定存储过程可以递归调用的次数。此选项的默认值为 0，完全禁用存储过程中的递归。最大值为 255。

    存储过程递归会增加线程堆栈空间的需求。如果增加`max_sp_recursion_depth`的值，可能需要通过增加服务器启动时的`thread_stack`的值来增加线程堆栈大小。

+   `max_user_connections`

    | 命令行格式 | `--max-user-connections=#` |
    | --- | --- |
    | 系统变量 | `max_user_connections` |
    | 作用范围 | 全局, 会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `0` |
    | 最小值 | `0` |
    | 最大值 | `4294967295` |

    给定 MySQL 用户帐户允许的最大同时连接数。值为 0（默认）表示“无限制”。

    此变量具有全局值，可以在服务器启动或运行时设置。它还具有只读会话值，指示适用于当前会话关联帐户的有效同时连接限制。会话值的初始化如下：

    +   如果用户账户具有非零的`MAX_USER_CONNECTIONS`资源限制，则会话`max_user_connections`值设置为该限制。

    +   否则，会话`max_user_connections`值设置为全局值。

    账户资源限制使用`CREATE USER`或`ALTER USER`语句指定。请参见第 8.2.21 节，“设置账户资源限制”。

+   `max_write_lock_count`

    | 命令行格式 | `--max-write-lock-count=#` |
    | --- | --- |
    | 系统变量 | `max_write_lock_count` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值（Windows） | `4294967295` |
    | 默认值（其他，64 位平台） | `18446744073709551615` |
    | 默认值（其他，32 位平台） | `4294967295` |
    | 最小值 | `1` |
    | 最大值（Windows） | `4294967295` |
    | 最大值（其他，64 位平台） | `18446744073709551615` |
    | 最大值（其他，32 位平台） | `4294967295` |

    在这么多写锁之后，允许一些待处理的读锁请求在其中被处理。写锁请求的优先级高于读锁请求。但是，如果`max_write_lock_count`设置为较低值（比如，10），则读锁请求可能优先于待处理的写锁请求，如果读锁请求已经被放弃以优先处理 10 个写锁请求。通常情况下，这种行为不会发生，因为`max_write_lock_count`默认情况下具有非常大的值。

+   `mecab_rc_file`

    | 命令行格式 | `--mecab-rc-file=file_name` |
    | --- | --- |
    | 系统变量 | `mecab_rc_file` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 文件名 |

    `mecab_rc_file`选项用于设置 MeCab 全文解析器。

    `mecab_rc_file`选项定义了`mecabrc`配置文件的路径，该文件是 MeCab 的配置文件。该选项是只读的，只能在启动时设置。`mecabrc`配置文件是初始化 MeCab 所必需的。

    有关 MeCab 全文解析器的信息，请参见第 14.9.9 节，“MeCab 全文解析器插件”。

    有关可以在 MeCab `mecabrc`配置文件中指定的选项的信息，请参考[MeCab 文档](http://mecab.googlecode.com/svn/trunk/mecab/doc/index.html)，位于[Google Developers](https://code.google.com/)网站上。

+   `metadata_locks_cache_size`

    | 命令行格式 | `--metadata-locks-cache-size=#` |
    | --- | --- |
    | 已弃用 | 是（在 8.0.13 中移除） |
    | 系统变量 | `metadata_locks_cache_size` |
    | 作用范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `1024` |
    | 最小值 | `1` |
    | 最大值 | `1048576` |
    | 单位 | 字节 |

    MySQL 8.0.13 中删除了此系统变量。

+   `metadata_locks_hash_instances`

    | 命令行格式 | `--metadata-locks-hash-instances=#` |
    | --- | --- |
    | 已弃用 | 是（在 8.0.13 中移除） |
    | 系统变量 | `metadata_locks_hash_instances` |
    | 作用范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `8` |
    | 最小值 | `1` |
    | 最大值 | `1024` |

    MySQL 8.0.13 中删除了此系统变量。

+   `min_examined_row_limit`

    | 命令行格式 | `--min-examined-row-limit=#` |
    | --- | --- |
    | 系统变量 | `min_examined_row_limit` |
    | 作用范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `0` |
    | 最小值 | `0` |
    | 最大值（64 位平台） | `18446744073709551615` |
    | 最大值（32 位平台） | `4294967295` |

    查询行数少于此数字的查询��会记录到慢查询日志中。

    截至 MySQL 8.0.27，设置此系统变量的会话值是受限操作。会话用户必须具有足够权限来设置受限会话变量。请参阅第 7.1.9.1 节，“系统变量权限”。

+   `myisam_data_pointer_size`

    | 命令行格式 | `--myisam-data-pointer-size=#` |
    | --- | --- |
    | 系统变量 | `myisam_data_pointer_size` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `6` |
    | 最小值 | `2` |
    | 最大值 | `7` |
    | 单位 | 字节 |

    用于在未指定`MAX_ROWS`选项时由`CREATE TABLE`用于`MyISAM`表的默认指针大小（以字节为单位）。此变量不能小于 2 或大于 7。默认值为 6。参见第 B.3.2.10 节，“表已满”。

+   `myisam_max_sort_file_size`

    | 命令行格式 | `--myisam-max-sort-file-size=#` |
    | --- | --- |
    | 系统变量 | `myisam_max_sort_file_size` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值（Windows） | `2146435072` |
    | 默认值（其他，64 位平台） | `9223372036853727232` |
    | 默认值（其他，32 位平台） | `2147483648` |
    | 最小值 | `0` |
    | 最大值（Windows） | `2146435072` |
    | 最大值（其他，64 位平台） | `9223372036853727232` |
    | 最大值（其他，32 位平台） | `2147483648` |
    | 单位 | 字节 |

    MySQL 在重新创建`MyISAM`索引（在`REPAIR TABLE`、`ALTER TABLE`或`LOAD DATA`期间）时允许使用的临时文件的最大大小。如果文件大小超过此值，索引将使用键缓存创建，这会更慢。该值以字节为单位。

    如果`MyISAM`索引文件超过此大小且磁盘空间可用，则增加该值可能有助于性能。空间必须在包含原始索引文件的目录中的文件系统中可用。

+   `myisam_mmap_size`

    | 命令行格式 | `--myisam-mmap-size=#` |
    | --- | --- |
    | 系统变量 | `myisam_mmap_size` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值（64 位平台） | `18446744073709551615` |
    | 默认值（32 位平台） | `4294967295` |
    | 最小值 | `7` |
    | 最大值（64 位平台） | `18446744073709551615` |
    | 最大值（32 位平台） | `4294967295` |
    | 单位 | 字节 |

    用于内存映射压缩`MyISAM`文件的最大内存量。如果使用了许多压缩的`MyISAM`表，可以减少该值以减少内存交换问题的可能性。

+   `myisam_recover_options`

    | 命令行格式 | `--myisam-recover-options[=list]` |
    | --- | --- |
    | 系统变量 | `myisam_recover_options` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 枚举 |
    | 默认值 | `OFF` |
    | 有效值 | `OFF``DEFAULT``BACKUP``FORCE``QUICK` |

    设置`MyISAM`存储引擎的恢复模式。变量值是`OFF`、`DEFAULT`、`BACKUP`、`FORCE`或`QUICK`值的任意组合。如果指定多个值，请用逗号分隔。在服务器启动时未指定值的情况下，与指定`DEFAULT`相同，指定明确值为`""`会禁用恢复（与`OFF`值相同）。如果启用了恢复，每次**mysqld**打开`MyISAM`表时，它都会检查表是否标记为崩溃或未正确关闭。（最后一个选项仅在禁用外部锁定时运行。）如果是这种情况，**mysqld**会对表进行检查。如果表损坏，**mysqld**会尝试修复它。

    以下选项影响修复操作的方式。

    | 选项 | 描述 |
    | --- | --- |
    | `OFF` | 无恢复。 |
    | `DEFAULT` | 没有备份、强制或快速检查的恢复。 |
    | `BACKUP` | 如果在恢复过程中更改了数据文件，则将`*`tbl_name`*.MYD`文件备份为`*`tbl_name-datetime`*.BAK`。 |
    | `FORCE` | 即使从`.MYD`文件中丢失超过一行，也要运行恢复操作。 |
    | `QUICK` | 如果表中没有删除块，则不检查行。 |

    在服务器自动修复表之前，它会将有关修复的注释写入错误日志。如果您希望能够在没有用户干预的情况下从大多数问题中恢复，您应该使用`BACKUP,FORCE`选项。这将强制修复表，即使一些行将被删除，但它会将旧数据文件保留为备份，以便您以后检查发生了什么。

    参见 Section 18.2.1, “MyISAM Startup Options”。

+   `myisam_repair_threads`

    | 命令行格式 | `--myisam-repair-threads=#` |
    | --- | --- |
    | 已弃用 | 8.0.29 (在 8.0.30 中移除) |
    | 系统变量 | `myisam_repair_threads` |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `1` |
    | 最小值 | `1` |
    | 最大值 (64 位平台) | `18446744073709551615` |
    | 最大值 (32 位平台) | `4294967295` |

    注意

    此系统变量在 MySQL 8.0.29 中已弃用，并在 MySQL 8.0.30 中移除。

    从 MySQL 8.0.29 开始，除了 1 之外的值会产生警告。

    如果此值大于 1，则在`Repair by sorting`过程中`MyISAM`表索引会并行创建（每个索引在自己的线程中）。默认值为 1。

    注意

    多线程修复是*测试质量*代码。

+   `myisam_sort_buffer_size`

    | 命令行格式 | `--myisam-sort-buffer-size=#` |
    | --- | --- |
    | 系统变量 | `myisam_sort_buffer_size` |
    | 作用范围 | 全局, 会话 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `8388608` |
    | 最小值 | `4096` |
    | 最大值（64 位平台） | `18446744073709551615` |
    | 最大值（32 位平台） | `4294967295` |
    | 单位 | 字节 |

    在进行`REPAIR TABLE`修复`MyISAM`索引或使用`CREATE INDEX`或`ALTER TABLE`创建索引时分配的缓冲区大小。

+   `myisam_stats_method`

    | 命令行格式 | `--myisam-stats-method=name` |
    | --- | --- |
    | 系统变量 | `myisam_stats_method` |
    | 作用范围 | 全局, 会话 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 枚举 |
    | 默认值 | `nulls_unequal` |
    | 有效值 | `nulls_unequal``nulls_equal``nulls_ignored` |

    服务器在收集关于`MyISAM`表索引值分布的统计信息时如何处理`NULL`值。此变量有三个可能的值，`nulls_equal`、`nulls_unequal`和`nulls_ignored`。对于`nulls_equal`，所有`NULL`索引值被视为相等，并形成一个大小等于`NULL`值数量的单个值组。对于`nulls_unequal`，`NULL`值被视为不相等，每个`NULL`形成一个大小为 1 的独立值组。对于`nulls_ignored`，`NULL`值被忽略。

    用于生成表统计信息的方法会影响优化器选择用于查询执行的索引，详见 Section 10.3.8, “InnoDB and MyISAM Index Statistics Collection”。

+   `myisam_use_mmap`

    | 命令行格式 | `--myisam-use-mmap[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `myisam_use_mmap` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔 |
    | 默认值 | `OFF` |

    使用内存映射读写`MyISAM`表。

+   `mysql_native_password_proxy_users`

    | 命令行格式 | `--mysql-native-password-proxy-users[={OFF | ON}]` |
    | --- | --- | --- |
    | 系统变量 | `mysql_native_password_proxy_users` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` Hint Applies | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    此变量控制`mysql_native_password`内置认证插件是否支持代理用户。除非启用了`check_proxy_users`系统变量，否则不起作用。有关用户代理的信息，请参见第 8.2.19 节“代理用户”。

+   `named_pipe`

    | 命令行格式 | `--named-pipe[={OFF | ON}]` |
    | --- | --- | --- |
    | 系统变量 | `named_pipe` |
    | 作用范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` Hint Applies | 否 |
    | 特定平台 | Windows |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    （仅限 Windows。）指示服务器是否支持通过命名管道进行连接。

+   `named_pipe_full_access_group`

    | 命令行格式 | `--named-pipe-full-access-group=value` |
    | --- | --- |
    | 引入版本 | 8.0.14 |
    | 系统变量 | `named_pipe_full_access_group` |
    | 作用范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` Hint Applies | 否 |
    | 特定平台 | Windows |
    | 类型 | 字符串 |
    | 默认值 | `空字符串` |
    | 有效值 | `空字符串``有效的 Windows 本地组名称``*everyone*` |

    （仅限 Windows。）当启用`named_pipe`系统变量以支持命名管道连接时，MySQL 服务器创建的命名管道的客户端访问控制被设置为成功通信所需的最低权限。一些 MySQL 客户端软件可以在没有任何额外配置的情况下打开命名管道连接；然而，其他客户端软件可能仍然需要完全访问权限才能打开命名管道连接。

    此变量设置了一个 Windows 本地组的名称，该组的成员被 MySQL 服务器授予足够的访问权限以使用命名管道客户端。截至 MySQL 8.0.24，默认值设置为空字符串，这意味着没有 Windows 用户被授予对命名管道的完全访问权限。

    可以在 Windows 中创建一个新的本地组名（例如，`mysql_access_client_users`），然后在绝对必要时用它来替换默认值。在这种情况下，将组的成员限制为尽可能少的用户，当其客户端软件升级时，从组中移除用户。尝试使用受影响的命名管道客户端打开与 MySQL 的连接的非组成员将被拒绝访问，直到 Windows 管理员将用户添加到组中。新添加的用户必须注销并重新登录以加入组（Windows 要求）。

    将值设置为`'*everyone*'`提供了一个与语言无关的方式来引用 Windows 上的 Everyone 组。默认情况下，Everyone 组不安全。

+   `net_buffer_length`

    | 命令行格式 | `--net-buffer-length=#` |
    | --- | --- |
    | 系统变量 | `net_buffer_length` |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `16384` |
    | 最小值 | `1024` |
    | 最大值 | `1048576` |
    | 单位 | 字节 |
    | 块大小 | `1024` |

    每个客户端线程都与连接缓冲区和结果缓冲区相关联。两者都以`net_buffer_length`指定的大小开始，但根据需要动态扩大至`max_allowed_packet`字节。每个 SQL 语句后，结果缓冲区会缩小至`net_buffer_length`。

    通常不应更改此变量，但如果内存很少，可以将其设置为客户端发送语句的预期长度。如果语句超过此长度，连接缓冲区会自动扩大。可以将`net_buffer_length`设置的最大值为 1MB。

    此变量的会话值是只读的。

+   `net_read_timeout`

    | 命令行格式 | `--net-read-timeout=#` |
    | --- | --- |
    | 系统变量 | `net_read_timeout` |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `30` |
    | 最小值 | `1` |
    | 最大值 | `31536000` |
    | 单位 | 秒 |

    从连接中等待更多数据的秒数，然后中止读取。当服务器从客户端读取数据时，`net_read_timeout` 是控制何时中止的超时值。当服务器向客户端写入数据时，`net_write_timeout` 是控制何时中止的超时值。另请参阅 `replica_net_timeout` 和 `slave_net_timeout`。

+   `net_retry_count`

    | 命令行格式 | `--net-retry-count=#` |
    | --- | --- |
    | 系统变量 | `net_retry_count` |
    | 作用范围 | 全局, 会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `10` |
    | 最小值 | `1` |
    | 最大值（64 位平台） | `18446744073709551615` |
    | 最大值（32 位平台） | `4294967295` |

    如果通信端口上的读取或写入被中断，重试此次数后放弃。在 FreeBSD 上应将此值设置得很高，因为内部中断会发送给所有线程。

+   `net_write_timeout`

    | 命令行格式 | `--net-write-timeout=#` |
    | --- | --- |
    | 系统变量 | `net_write_timeout` |
    | 作用范围 | 全局, 会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `60` |
    | 最小值 | `1` |
    | 最大值 | `31536000` |
    | 单位 | 秒 |

    等待将块写入连接的秒数，然后中止写入。另请参阅 `net_read_timeout`。

+   `new`

    | 命令行格式 | `--new[={OFF&#124;ON}]` |
    | --- | --- |
    | 已弃用 | 8.0.35 |
    | 系统变量 | `new` |
    | 作用范围 | 全局, 会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 禁用 | `skip-new` |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    此变量在 MySQL 4.0 中用于启用一些 4.1 特性，并为向后兼容性而保留。其值始终为 `OFF`。

    此变量在 MySQL 8.0.35 中已弃用，并可能在将来的版本中被移除。

    在 NDB Cluster 中，将此变量设置为`ON`可以使用除`KEY`或`LINEAR KEY`之外的分区类型与`NDB`表。此实验性功能不支持生产环境，并且现已弃用，因此可能在将来的版本中被移除。有关更多信息，请参见用户定义的分区和 NDB 存储引擎（NDB Cluster）。

+   `ngram_token_size`

    | 命令行格式 | `--ngram-token-size=#` |
    | --- | --- |
    | 系统变量 | `ngram_token_size` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `2` |
    | 最小值 | `1` |
    | 最大值 | `10` |

    定义 n-gram 全文解析器的 n-gram 令牌大小。`ngram_token_size`选项是只读的，只能在启动时修改。默认值为 2（二元组）。最大值为 10。

    有关如何配置此变量的更多信息，请参见第 14.9.8 节，“ngram 全文解析器”。

+   `offline_mode`

    | 命令行格式 | `--offline-mode[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `offline_mode` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    在离线模式下，MySQL 实例会断开客户端用户的连接，除非他们具有相关权限，并且不允许他们发起新连接。被拒绝访问的客户端会收到一个`ER_SERVER_OFFLINE_MODE`错误。

    要将服务器置于离线模式，请将`offline_mode`系统变量的值从`OFF`更改为`ON`。要恢复正常操作，请将`offline_mode`从`ON`更改为`OFF`。要控制离线模式，管理员帐户必须具有`SYSTEM_VARIABLES_ADMIN`权限和`CONNECTION_ADMIN`权限（或已弃用的`SUPER`权限，涵盖这两个权限）。从 MySQL 8.0.31 开始需要`CONNECTION_ADMIN`，并建议在所有版本中使用以防止意外锁定。

    离线模式具有以下特征：

    +   没有`CONNECTION_ADMIN`权限（或已弃用的`SUPER`权限）的连接客户端用户在下一个请求时将被断开连接，并显示适当的错误。断开连接包括终止正在运行的语句和释放锁。这些客户端也不能发起新连接，并收到适当的错误。

    +   拥有`CONNECTION_ADMIN`或`SUPER`权限的连接客户端用户不会被断开连接，并且可以发起新连接以管理服务器。

    +   从 MySQL 8.0.30 开始，如果将服务器置于脱机模式的用户没有`SYSTEM_USER`权限，那么拥有`SYSTEM_USER`权限的连接客户端用户也不会被断开连接。但是，在服务器处于脱机模式时，这些用户不能发起新连接，除非他们也具有`CONNECTION_ADMIN`或`SUPER`权限。只有他们现有的连接不能被终止，因为终止正在使用`SYSTEM_USER`权限执行的会话或语句需要`SYSTEM_USER`权限。

    +   复制线程被允许继续向服务器应用数据。

+   `old`

    | 命令行格式 | `--old[={OFF&#124;ON}]` |
    | --- | --- |
    | 弃用 | 8.0.35 |
    | 系统变量 | `old` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    `old`是一个兼容性变量。默认情况下禁用，但可以在启动时启用以恢复服务器在旧版本中存在的行为。

    当启用`old`时，它将更改索引提示的默认范围，使其与 MySQL 5.1.17 之前使用的范围相同。也就是说，没有`FOR`子句的索引提示仅适用于索引在行检索中的使用，而不适用于`ORDER BY`或`GROUP BY`子句的解析。（参见第 10.9.4 节，“索引提示”。）在复制设置中启用此选项时要小心。使用基于语句的二进制日志记录，源和副本之间具有不同模式可能导致复制错误。

    该变量自 MySQL 8.0.35 起已被弃用，并可能在未来的版本中被移除。

+   `old_alter_table`

    | 命令行格式 | `--old-alter-table[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `old_alter_table` |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `关闭` |

    当启用此变量时，服务器不会使用优化的方法来处理`ALTER TABLE`操作。它会回退到使用临时表、复制数据，然后将临时表重命名为原始表的方法，就像 MySQL 5.0 及更早版本所使用的那样。有关`ALTER TABLE`操作的更多信息，请参见第 15.1.9 节，“ALTER TABLE 语句”。

    使用`old_alter_table=ON`的`ALTER TABLE ... DROP PARTITION`会重建分区表，并尝试将删除的分区数据移动到具有兼容`PARTITION ... VALUES`定义的另一个分区。无法移动到另一个分区的数据将被删除。在早期版本中，使用`old_alter_table=ON`的`ALTER TABLE ... DROP PARTITION`会删除存储在分区中的数据并删除该分区。

+   `open_files_limit`

    | 命令行格式 | `--open-files-limit=#` |
    | --- | --- |
    | 系统变量 | `open_files_limit` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `5000，可能会调整` |
    | 最小值 | `0` |
    | 最大值 | `平台相关` |

    操作系统提供给**mysqld**的文件描述符数量：

    +   在启动时，**mysqld**使用`setrlimit()`保留描述符，使用直接设置此变量的值或使用`--open-files-limit`选项给**mysqld_safe**。如果**mysqld**出现`打开文件过多`的错误，请尝试增加`open_files_limit`的值。在内部，此变量的最大值是最大无符号整数值，但实际最大值取决于平台。

    +   在运行时，`open_files_limit`的值表示操作系统实际允许**mysqld**使用的文件描述符数量，这可能与启动时请求的值不同。如果在启动期间无法分配请求的文件描述符数量，**mysqld**会向错误日志写入警告。

    有效的`open_files_limit`值基于系统启动时指定的值（如果有）以及`max_connections`和`table_open_cache`的值，使用以下公式：

    +   `10 + max_connections + (table_open_cache * 2)`。使用这些变量的默认值得到 8161。

        仅在 Windows 上，会将 2048（C 运行时库文件描述符最大值）添加到此数字中。这将总计为 10209，再次使用指定系统变量的默认值。

    +   `max_connections * 5`

    +   MySQL 8.0.19 及更高版本：操作系统限制。

    +   MySQL 8.0.19 之前：

        +   如果该限制为正但不是无穷大，则为操作系统限制。

        +   如果操作系统限制为无穷大：如果在启动时指定了`open_files_limit`值，则为该值，否则为 5000。

    服务器尝试使用这些值中的最大值获取文件描述符数量，上限为最大无符号整数值。如果无法获取那么多描述符，则服务器尝试获取系统允许的数量。

    在 MySQL 无法更改打开文件数量的系统上，有效值为 0。

    在 Unix 上，该值不能大于**ulimit -n**命令显示的值。在使用`systemd`的 Linux 系统上，该值不能大于`LimitNOFILE`（如果未设置`LimitNOFILE`，则为`DefaultLimitNOFILE`）；否则，在 Linux 上，`open_files_limit`的值不能超过**ulimit -n**。

+   `optimizer_prune_level`

    | 命令行格式 | `--optimizer-prune-level=#` |
    | --- | --- |
    | 系统变量 | `optimizer_prune_level` |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 是 |
    | 类型 | 整数 |
    | 默认值 | `1` |
    | 最小值 | `0` |
    | 最大值 | `1` |

    控制查询优化期间应用的启发式方法，以从优化器搜索空间中剪枝不太有前途的部分计划。值为 0 会禁用启发式方法，使优化器执行详尽搜索。值为 1 会导致优化器根据中间计划检索的行数来剪枝计划。

+   `optimizer_search_depth`

    | 命令行格式 | `--optimizer-search-depth=#` |
    | --- | --- |
    | 系统变量 | `optimizer_search_depth` |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 是 |
    | 类型 | 整数 |
    | 默认值 | `62` |
    | 最小值 | `0` |
    | 最大值 | `62` |

    查询优化器执行的搜索的最大深度。比查询结果中的关系数更大的值会产生更好的查询计划，但生成查询执行计划的时间更长。比查询中的关系数更小的值会更快地返回执行计划，但结果计划可能远非最佳。如果设置为 0，系统会自动选择一个合理的值。

+   `optimizer_switch`

    | 命令行格式 | `--optimizer-switch=value` |
    | --- | --- |
    | 系统变量 | `optimizer_switch` |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 是 |
    | 类型 | 集合 |
    | 有效值（≥ 8.0.22） | `batched_key_access={on&#124;off}``block_nested_loop={on&#124;off}``condition_fanout_filter={on&#124;off}``derived_condition_pushdown={on&#124;off}``derived_merge={on&#124;off}``duplicateweedout={on&#124;off}``engine_condition_pushdown={on&#124;off}``firstmatch={on&#124;off}``hash_join={on&#124;off}``index_condition_pushdown={on&#124;off}``index_merge={on&#124;off}``index_merge_intersection={on&#124;off}``index_merge_sort_union={on&#124;off}``index_merge_union={on&#124;off}``loosescan={on&#124;off}``materialization={on&#124;off}``mrr={on&#124;off}``mrr_cost_based={on&#124;off}``prefer_ordering_index={on&#124;off}``semijoin={on&#124;off}``skip_scan={on&#124;off}``subquery_materialization_cost_based={on&#124;off}``subquery_to_derived={on&#124;off}``use_index_extensions={on&#124;off}``use_invisible_indexes={on&#124;off}` |
    | 有效值（≥ 8.0.21） | `batched_key_access={on&#124;off}``block_nested_loop={on&#124;off}``condition_fanout_filter={on&#124;off}``derived_merge={on&#124;off}``duplicateweedout={on&#124;off}``engine_condition_pushdown={on&#124;off}``firstmatch={on&#124;off}``hash_join={on&#124;off}``index_condition_pushdown={on&#124;off}``index_merge={on&#124;off}``index_merge_intersection={on&#124;off}``index_merge_sort_union={on&#124;off}``index_merge_union={on&#124;off}``loosescan={on&#124;off}``materialization={on&#124;off}``mrr={on&#124;off}``mrr_cost_based={on&#124;off}``prefer_ordering_index={on&#124;off}``semijoin={on&#124;off}``skip_scan={on&#124;off}``subquery_materialization_cost_based={on&#124;off}``subquery_to_derived={on&#124;off}``use_index_extensions={on&#124;off}``use_invisible_indexes={on&#124;off}` |
    | 有效值（≥ 8.0.18） | `batched_key_access={on&#124;off}``block_nested_loop={on&#124;off}``condition_fanout_filter={on&#124;off}``derived_merge={on&#124;off}``duplicateweedout={on&#124;off}``engine_condition_pushdown={on&#124;off}``firstmatch={on&#124;off}``hash_join={on&#124;off}``index_condition_pushdown={on&#124;off}``index_merge={on&#124;off}``index_merge_intersection={on&#124;off}``index_merge_sort_union={on&#124;off}``index_merge_union={on&#124;off}``loosescan={on&#124;off}``materialization={on&#124;off}``mrr={on&#124;off}``mrr_cost_based={on&#124;off}``semijoin={on&#124;off}``skip_scan={on&#124;off}``subquery_materialization_cost_based={on&#124;off}``use_index_extensions={on&#124;off}``use_invisible_indexes={on&#124;off}` |
    | 有效值（≥ 8.0.13） | `batched_key_access={on&#124;off}``block_nested_loop={on&#124;off}``condition_fanout_filter={on&#124;off}``derived_merge={on&#124;off}``duplicateweedout={on&#124;off}``engine_condition_pushdown={on&#124;off}``firstmatch={on&#124;off}``index_condition_pushdown={on&#124;off}``index_merge={on&#124;off}``index_merge_intersection={on&#124;off}``index_merge_sort_union={on&#124;off}``index_merge_union={on&#124;off}``loosescan={on&#124;off}``materialization={on&#124;off}``mrr={on&#124;off}``mrr_cost_based={on&#124;off}``semijoin={on&#124;off}``skip_scan={on&#124;off}``subquery_materialization_cost_based={on&#124;off}``use_index_extensions={on&#124;off}``use_invisible_indexes={on&#124;off}` |
    | 有效值（≤ 8.0.12） | `batched_key_access={on&#124;off}``block_nested_loop={on&#124;off}``condition_fanout_filter={on&#124;off}``derived_merge={on&#124;off}``duplicateweedout={on&#124;off}``engine_condition_pushdown={on&#124;off}``firstmatch={on&#124;off}``index_condition_pushdown={on&#124;off}``index_merge={on&#124;off}``index_merge_intersection={on&#124;off}``index_merge_sort_union={on&#124;off}``index_merge_union={on&#124;off}``loosescan={on&#124;off}``materialization={on&#124;off}``mrr={on&#124;off}``mrr_cost_based={on&#124;off}``semijoin={on&#124;off}``subquery_materialization_cost_based={on&#124;off}``use_index_extensions={on&#124;off}``use_invisible_indexes={on&#124;off}` |

    `optimizer_switch` 系统变量可控制优化器行为。该变量的值是一组标志，每个标志的值为`on`或`off`，表示相应的优化器行为是否启用或禁用。此变量具有全局和会话值，并且可以在运行时更改。全局默认值可以在服务器启动时设置。

    要查看当前的优化器标志集，请选择变量值：

    ```sql
    mysql> SELECT @@optimizer_switch\G
    *************************** 1\. row ***************************
    @@optimizer_switch: index_merge=on,index_merge_union=on,
                        index_merge_sort_union=on,index_merge_intersection=on,
                        engine_condition_pushdown=on,index_condition_pushdown=on,
                        mrr=on,mrr_cost_based=on,block_nested_loop=on,
                        batched_key_access=off,materialization=on,semijoin=on,
                        loosescan=on,firstmatch=on,duplicateweedout=on,
                        subquery_materialization_cost_based=on,
                        use_index_extensions=on,condition_fanout_filter=on,
                        derived_merge=on,use_invisible_indexes=off,skip_scan=on,
                        hash_join=on,subquery_to_derived=off,
                        prefer_ordering_index=on,hypergraph_optimizer=off,
                        derived_condition_pushdown=on
    ```

    有关此变量的语法和其控制的优化器行为的更多信息，请参见第 10.9.2 节，“可切换优化”。

+   `optimizer_trace`

    | 命令行格式 | `--optimizer-trace=value` |
    | --- | --- |
    | 系统变量 | `optimizer_trace` |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 字符串 |

    此变量控制优化器跟踪。详情请参阅 MySQL 内部：跟踪优化器。

+   `optimizer_trace_features`

    | 命令行格式 | `--optimizer-trace-features=value` |
    | --- | --- |
    | 系统变量 | `optimizer_trace_features` |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 字符串 |

    此变量启用或禁用选定的优化器跟踪功能。详情请参阅 MySQL 内部：跟踪优化器。

+   `optimizer_trace_limit`

    | 命令行格式 | `--optimizer-trace-limit=#` |
    | --- | --- |
    | 系统变量 | `optimizer_trace_limit` |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `1` |
    | 最小值 | `0` |
    | 最大值 | `2147483647` |

    最大要显示的优化器跟踪数量。详情请参阅 MySQL 内部：跟踪优化器。

+   `optimizer_trace_max_mem_size`

    | 命令行格式 | `--optimizer-trace-max-mem-size=#` |
    | --- | --- |
    | 系统变量 | `optimizer_trace_max_mem_size` |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 是 |
    | 类型 | 整数 |
    | 默认值 | `1048576` |
    | 最小值 | `0` |
    | 最大值 | `4294967295` |
    | 单位 | 字节 |

    最大累积存储优化器跟踪的大小。详情请参阅 MySQL 内部：跟踪优化器。

+   `optimizer_trace_offset`

    | 命令行格式 | `--optimizer-trace-offset=#` |
    | --- | --- |
    | 系统变量 | `optimizer_trace_offset` |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `-1` |
    | 最小值 | `-2147483647` |
    | 最大值 | `2147483647` |

    优化器跟踪的偏移量。详情请参阅 MySQL Internals: Tracing the Optimizer。

+   `performance_schema_*`xxx`*`

    性能模式系统变量列在 第 29.15 节，“性能模式系统变量” 中。这些变量可用于配置性能模式操作。

+   `parser_max_mem_size`

    | 命令行格式 | `--parser-max-mem-size=#` |
    | --- | --- |
    | 系统变量 | `parser_max_mem_size` |
    | 作用域 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR` Hint Applies | 否 |
    | 类型 | 整数 |
    | 默认值（64 位平台） | `18446744073709551615` |
    | 默认值（32 位平台） | `4294967295` |
    | 最小值 | `10000000` |
    | 最大值（64 位平台） | `18446744073709551615` |
    | 最大值（32 位平台） | `4294967295` |
    | 单位 | 字节 |

    解析器可用的最大内存量。默认值不限制可用内存。该值可以减少，以防止解析长或复杂的 SQL 语句导致的内存不足情况。

+   `partial_revokes`

    | 命令行格式 | `--partial-revokes[={OFF&#124;ON}]` |
    | --- | --- |
    | 引入版本 | 8.0.16 |
    | 系统变量 | `partial_revokes` |
    | 作用域 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` Hint Applies | 否 |
    | 类型 | 布尔 |
    | 默认值 | `OFF`（如果部分撤销不存在）`ON`（如果部分撤销存在） |

    启用此变量可以部分撤销权限。具体来说，对于在全局级别具有权限的用户，`partial_revokes` 允许撤销特定模式的权限，同时保留其他模式的权限。例如，具有全局 `UPDATE` 权限的用户可以限制在 `mysql` 系统模式上行使此权限。换句话说，用户可以在所有模式上行使 `UPDATE` 权限，除了 `mysql` 模式。从这个意义上说，用户的全局 `UPDATE` 权限被部分撤销。

    一旦启用，如果任何帐户有权限限制，则无法禁用 `partial_revokes`。如果存��任何此类帐户，则禁用 `partial_revokes` 将失败：

    +   尝试在启动时禁用`partial_revokes`时，服务器会记录错误消息并启用`partial_revokes`。

    +   尝试在运行时禁用`partial_revokes`时，会发生错误并且`partial_revokes`的值保持不变。

    要在此情况下禁用`partial_revokes`，首先修改每个具有部分撤销权限的帐户，重新授予权限或删除帐户。

    注意

    在权限分配中，启用`partial_revokes`会导致 MySQL 将模式名称中未转义的`_`和`%` SQL 通配符字符解释为文字字符，就好像它们已经被转义为`\_`和`\%`一样。因为这会改变 MySQL 解释权限的方式，建议在可能启用`partial_revokes`的安装中避免未转义的通配符字符在权限分配中出现。

    此外，在 MySQL 8.0.35 中，作为授权中通配符字符的`_`和`%`的使用已被弃用，并且您应该预期在将来的 MySQL 版本中将删除对它们的支持。

    欲了解更多信息，包括有关删除部分撤销的说明，请参阅第 8.2.12 节，“使用部分撤销进行权限限制”。

+   `password_history`

    | 命令行格式 | `--password-history=#` |
    | --- | --- |
    | 系统变量 | `password_history` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `0` |
    | 最小值 | `0` |
    | 最大值 | `4294967295` |

    此变量定义了基于所需最小密码更改次数控制重用先前密码的全局策略。对于先前使用过的帐户密码，此变量指示必须发生多少次后续帐户密码更改才能重新使用密码。如果值为 0（默认值），则没有基于密码更改次数的重用限制。

    对此变量的更改立即应用于所有使用`PASSWORD HISTORY DEFAULT`选项定义的帐户。

    可以通过`CREATE USER`和`ALTER USER`语句的`PASSWORD HISTORY`选项，根据需要为单个账户覆盖全局的更改次数密码重用策略。参见第 8.2.15 节，“密码管理”。

+   `password_require_current`

    | 命令行格式 | `--password-require-current[={OFF&#124;ON}]` |
    | --- | --- |
    | 引入版本 | 8.0.13 |
    | 系统变量 | `password_require_current` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    这个变量定义了全局策略，控制更改账户密码的尝试是否必须指定要替换的当前密码。

    对该变量的更改立即应用于所有使用`PASSWORD REQUIRE CURRENT DEFAULT`选项定义的账户。

    可以通过`CREATE USER`和`ALTER USER`语句的`PASSWORD REQUIRE`选项，根据需要为单个账户覆盖全局的验证要求策略。参见第 8.2.15 节，“密码管理”。

+   `password_reuse_interval`

    | 命令行格式 | `--password-reuse-interval=#` |
    | --- | --- |
    | 系统变量 | `password_reuse_interval` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `0` |
    | 最小值 | `0` |
    | 最大值 | `4294967295` |
    | 单位 | 天 |

    这个变量定义了基于经过的时间控制先前密码重用的全局策略。对于先前使用过的账户密码，该变量指示必须经过多少天才能重新使用密码。如果值为 0（默认值），则没有基于经过时间的重用限制。

    对该变量的更改立即应用于所有使用`PASSWORD REUSE INTERVAL DEFAULT`选项定义的账户。

    可以通过`CREATE USER`和`ALTER USER`语句的`PASSWORD REUSE INTERVAL`选项，根据需要为单个账户覆盖全局的经过时间密码重用策略。参见第 8.2.15 节，“密码管理”。

+   `persisted_globals_load`

    | 命令行格式 | `--persisted-globals-load[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `persisted_globals_load` |
    | 作用范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `ON` |

    是否从数据目录中的 `mysqld-auto.cnf` 文件加载持久化配置设置。服务器通常在启动后处理此文件，位于所有其他选项文件之后（参见 Section 6.2.2.2, “使用选项文件”）。禁用 `persisted_globals_load` 会导致服务器启动顺序跳过 `mysqld-auto.cnf`。

    若要修改 `mysqld-auto.cnf` 的内容，请使用 `SET PERSIST`、`SET PERSIST_ONLY` 和 `RESET PERSIST` 语句。参见 Section 7.1.9.3, “持久化系统变量”。

+   `persist_only_admin_x509_subject`

    | 命令行格式 | `--persist-only-admin-x509-subject=string` |
    | --- | --- |
    | 引入版本 | 8.0.14 |
    | 系统变量 | `persist_only_admin_x509_subject` |
    | 作用范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 字符串 |
    | 默认值 | `空字符串` |

    `SET PERSIST` 和 `SET PERSIST_ONLY` 允许系统变量持久化到数据目录中的 `mysqld-auto.cnf` 选项文件中（参见 Section 15.7.6.1, “变量赋值的 SET 语法”）。持久化系统变量使得运行时配置更改会影响后续服务器重启，这对于远程管理而无需直接访问 MySQL 服务器主机选项文件非常方便。然而，一些系统变量是不可持久化的，或者只能在某些限制条件下持久化。

    `persist_only_admin_x509_subject` 系统变量指定用户必须具有的 SSL 证书 X.509 主题值，以便能够持久化受限制的系统变量。默认值为空字符串，这会禁用主题检查，因此受限制的系统变量无法被任何用户持久化。

    如果 `persist_only_admin_x509_subject` 不为空，则使用加密连接连接到服务器并提供具有指定主题值的 SSL 证书的用户可以使用 `SET PERSIST_ONLY` 来持久化受限制的系统变量。有关受限制的持久化系统变量的信息以及配置 MySQL 以启用 `persist_only_admin_x509_subject` 的说明，请参见 第 7.1.9.4 节，“不可持久化和受限制的持久化系统变量”。

+   `persist_sensitive_variables_in_plaintext`

    | 命令行格式 | `--persist_sensitive_variables_in_plaintext[={OFF&#124;ON}]` |
    | --- | --- |
    | 引���版本 | 8.0.29 |
    | 系统变量 | `persist_sensitive_variables_in_plaintext` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `ON` |

    `persist_sensitive_variables_in_plaintext` 控制着服务器是否允许以未加密的格式存储敏感系统变量的值，如果在使用 `SET PERSIST` 设置系统变量的值时，当时没有可用的 keyring 组件支持。它还控制着服务器是否能够在无法解密加密值时启动。请注意，keyring 插件不支持对敏感系统变量进行安全存储；必须在 MySQL 服务器实例上启用一个 keyring 组件（参见 第 8.4.4 节，“MySQL Keyring”）以支持安全存储。

    默认设置为 `ON`，如果有 keyring 组件支持，则加密值，并在没有支持时持久化未加密的值（并发出警告）。下次设置任何持久化系统变量时，如果在那时有 keyring 支持，服务器会加密任何未加密的敏感系统变量的值。`ON` 设置还允许服务器在无法解密加密系统变量值时启动，在这种情况下会发出警告，并使用系统变量的默认值。在那种情况下，直到可以解密它们为止，它们的值不能被更改。

    最安全的设置为 `OFF`，意味着如果没有 keyring 组件支持，则敏感系统变量值无法持久化。`OFF` 设置还意味着如果无法解密加密系统变量值，则服务器不会启动。

    更多信息，请参阅 持久化敏感系统变量。

+   `pid_file`

    | 命令行格式 | `--pid-file=file_name` |
    | --- | --- |
    | 系统变量 | `pid_file` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 文件名 |

    服务器写入其进程 ID 的文件的路径名。除非给定绝对路径名以指定不同的目录，否则服务器将在数据目录中创建该文件。如果指定了此变量，则必须指定一个值。如果未指定此变量，MySQL 将使用默认值`*`host_name`*.pid`，其中*`host_name`*是主机机器的名称。

    进程 ID 文件用于其他程序（如 **mysqld_safe**）确定服务器的进程 ID。在 Windows 上，此变量还会影响默认错误日志文件名。请参阅 第 7.4.2 节，“错误日志”。

+   `plugin_dir`

    | 命令行格式 | `--plugin-dir=dir_name` |
    | --- | --- |
    | 系统变量 | `plugin_dir` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 目录名 |
    | 默认值 | `BASEDIR/lib/plugin` |

    插件目录的路径名。

    如果插件目录可被服务器写入，用户可能可以使用 `SELECT ... INTO DUMPFILE` 将可执行代码写入目录中的文件。可以通过将 `plugin_dir` 设置为服务器只读或将 `secure_file_priv` 设置为可以安全写入 `SELECT` 的目录来防止这种情况。

+   `port`

    | 命令行格式 | `--port=port_num` |
    | --- | --- |
    | 系统变量 | `port` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `3306` |
    | 最小值 | `0` |
    | 最大值 | `65535` |

    服务器监听 TCP/IP 连接的端口号。此变量可以使用 `--port` 选项设置。

+   `preload_buffer_size`

    | 命令行格式 | `--preload-buffer-size=#` |
    | --- | --- |
    | 系统变量 | `preload_buffer_size` |
    | 作用范围 | 全局, 会话 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `32768` |
    | 最小值 | `1024` |
    | 最大值 | `1073741824` |
    | 单位 | 字节 |

    预加载索引时分配的缓冲区大小。

    截至 MySQL 8.0.27，设置此系统变量的会话值是受限制的操作。会话用户必须具有足够权限来设置受限制的会话变量。参见第 7.1.9.1 节，“系统变量权限”。

+   `print_identified_with_as_hex`

    | 命令行格式 | `--print-identified-with-as-hex[={OFF&#124;ON}]` |
    | --- | --- |
    | 引入版本 | 8.0.17 |
    | 系统变量 | `print_identified_with_as_hex` |
    | 作用范围 | 全局, 会话 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    在`SHOW CREATE USER`输出的`IDENTIFIED WITH`子句中显示的密码哈希值可能包含对终端显示和其他环境产生不良影响的不可打印字符。启用`print_identified_with_as_hex`会导致`SHOW CREATE USER`以十六进制字符串而不是常规字符串文字显示这些哈希值。即使启用此变量，不包含不可打印字符的哈希值仍会显��为常规字符串文字。

+   `profiling`

    如果设置为 0 或`OFF`（默认值），则禁用语句分析。如果设置为 1 或`ON`，则启用语句分析，并且`SHOW PROFILE`和`SHOW PROFILES`语句提供对分析信息的访问。参见第 15.7.7.31 节，“SHOW PROFILES 语句”。

    此变量已被弃用；预计将在未来的 MySQL 版本中删除。

+   `profiling_history_size`

    如果启用了`profiling`，则维护分析信息的语句数量。默认值为 15。最大值为 100。将值设置为 0 将有效地禁用分析。参见第 15.7.7.31 节，“SHOW PROFILES 语句”。

    此变量已被弃用；预计在未来的 MySQL 版本中将被移除。

+   `protocol_compression_algorithms`

    | 命令行格式 | `--protocol-compression-algorithms=value` |
    | --- | --- |
    | 引入版本 | 8.0.18 |
    | 系统变量 | `protocol_compression_algorithms` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 集合 |
    | 默认值 | `zlib,zstd,uncompressed` |
    | 有效值 | `zlib``zstd``uncompressed` |

    服务器允许传入连接的压缩算法。这些包括客户端程序的连接以及参与源/副本复制或组复制的服务器的连接。压缩不适用于`FEDERATED`表的连接。

    `protocol_compression_algorithms` 不控制 X 协议的连接压缩。请参阅第 22.5.5 节，“X 插件的连接压缩”以了解其操作方式。

    变量值是一个或多个逗号分隔的压缩算法名称列表，任意顺序，可从以下项目中选择（不区分大小写）：

    +   `zlib`: 允许使用`zlib`压缩算法的连接。

    +   `zstd`: 允许使用`zstd`压缩算法的连接。

    +   `uncompressed`: 允许未压缩的连接。如果此算法名称未包含在`protocol_compression_algorithms`值中，服务器将不允许未压缩的连接。它仅允许使用值中指定的其他算法的压缩连接，并且没有回退到未压缩的连接。

    `zlib,zstd,uncompressed`的默认值表示服务器允许所有压缩算法。

    有关更多信息，请参阅第 6.2.8 节，“连接压缩控制”。

+   `protocol_version`

    | 系统变量 | `protocol_version` |
    | --- | --- |
    | 作用范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `10` |
    | 最小值 | `0` |
    | 最大值 | `4294967295` |

    MySQL 服务器使用的客户端/服务器协议版本。

+   `proxy_user`

    | 系统变量 | `proxy_user` |
    | --- | --- |
    | 作用范围 | 会话 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 字符串 |

    如果当前客户端是另一个用户的代理，则此变量是代理用户帐户名称。否则，此变量为`NULL`。请参阅第 8.2.19 节“代理用户”。

+   `pseudo_replica_mode`

    | 引入版本 | 8.0.26 |
    | --- | --- |
    | 系统变量 | `pseudo_replica_mode` |
    | 范围 | 会话 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔值 |

    从 MySQL 8.0.26 开始，`pseudo_replica_mode`取代了从该版本开始已弃用的`pseudo_slave_mode`。操作和效果相同，只是术语已更改。

    `pseudo_replica_mode`用于内部服务器使用。它有助于正确处理在比当前处理它们的服务器更旧或更新的服务器上起源的事务。**mysqlbinlog**在执行任何 SQL 语句之前将`pseudo_replica_mode`的值设置为 true。

    设置`pseudo_replica_mode`的会话值是受限制的操作。会话用户必须具有`REPLICATION_APPLIER`权限（请参阅第 19.3.3 节“复制权限检查”），或具有足以设置受限制会话变量的权限（请参阅第 7.1.9.1 节“系统变量权限”）。但是，请注意，该变量不是供用户设置的；它是由复制基础设施自动设置的。

    `pseudo_replica_mode`对准备的 XA 事务的处理产生以下影响，这些事务可以附加到或从处理会话中分离（默认情况下，发出`XA START`的会话）：

    +   如果为真，并且处理会话已执行内部使用的`BINLOG`语句，则一旦事务的第一部分直到`XA PREPARE`完成，XA 事务将自动从会话中分离，以便任何具有`XA_RECOVER_ADMIN`权限的会话可以提交或回滚。

    +   如果为 false，则 XA 事务会一直附加到处理会话，只要该会话仍然存在，期间其他会话无法提交事务。只有在会话断开连接或服务器重新启动时，准备好的事务才会被分离。

    `pseudo_replica_mode`对`original_commit_timestamp`复制延迟时间戳和`original_server_version`系统变量有以下影响：

    +   如果为 true，则假定未显式设置`original_commit_timestamp`或`original_server_version`的事务起源于另一个未知服务器，因此将值 0（表示未知）分配给时间戳和系统变量。

    +   如果为 false，则假定未显式设置`original_commit_timestamp`或`original_server_version`的事务起源于当前服务器，因此将当前时间戳和当前服务器版本分配给时间戳和系统变量。

    在 MySQL 8.0.14 及更高版本中，`pseudo_replica_mode`对设置一个或多个不支持（已移除或未知）SQL 模式的语句的处理有以下影响：

    +   如果为 true，则服务器会忽略不支持的模式并发出警告。

    +   如果为 false，则服务器会拒绝带有`ER_UNSUPPORTED_SQL_MODE`的语句。

+   `pseudo_slave_mode`

    | 已弃用 | 8.0.26 |
    | --- | --- |
    | 系统变量 | `pseudo_slave_mode` |
    | 范围 | 会话 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔值 |

    从 MySQL 8.0.26 开始，`pseudo_slave_mode`已被弃用，取而代之的是别名`pseudo_replica_mode`。`pseudo_slave_mode`用于内部服务器使用。它有助于正确处理在比当前处理它们的服务器更旧或更新的服务器上起源的事务。**mysqlbinlog**在执行任何 SQL 语句之前将`pseudo_slave_mode`的值设置为 true。

    设置此系统变量的会话值是受限操作。会话用户必须具有`REPLICATION_APPLIER`权限（参见第 19.3.3 节，“复制权限检查”），或具有足够权限设置受限会话变量（参见第 7.1.9.1 节，“系统变量权限”）。但是，请注意，该变量不是供用户设置的；它是由复制基础设施自动设置的。

    请参阅`pseudo_replica_mode`系统变量的描述，了解`pseudo_slave_mode`的影响。

+   `pseudo_thread_id`

    | 系统变量 | `pseudo_thread_id` |
    | --- | --- |
    | 作用域 | 会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `2147483647` |
    | 最小值 | `0` |
    | 最大值 | `2147483647` |

    此变量供内部服务器使用。

    警告

    更改`pseudo_thread_id`系统变量的会话值会更改`CONNECTION_ID()`函数返回的值。

    截至 MySQL 8.0.14，设置此系统变量的会话值是受限操作。会话用户必须具有足够权限设置受限会话变量。请参见第 7.1.9.1 节，“系统变量权限”。

+   `query_alloc_block_size`

    | 命令行格式 | `--query-alloc-block-size=#` |
    | --- | --- |
    | 系统变量 | `query_alloc_block_size` |
    | 作用域 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `8192` |
    | 最小值 | `1024` |
    | 最大值 | `4294966272` |
    | 单位 | 字节 |
    | 块大小 | `1024` |

    在语句解析和执行期间为创建的对象分配的内存块的字节大小。如果遇到内存碎片问题，增加此参数可能有所帮助。

    字节编号的块大小为 1024。在存储系统变量的值之前，MySQL 服务器会将不是块大小的整数倍的值向下舍入到块大小的下一个较低的整数倍。解析器允许值达到平台的最大无符号整数值（32 位系统为 4294967295 或 2³²−1，64 位系统为 18446744073709551615 或 2⁶⁴−1），但实际最大值是较低的块大小。

+   `query_prealloc_size`

    | 命令行格式 | `--query-prealloc-size=#` |
    | --- | --- |
    | 已弃用 | 8.0.29 |
    | 系统变量 | `query_prealloc_size` |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `8192` |
    | 最小值 | `8192` |
    | 最大值（64 位平台） | `18446744073709550592` |
    | 最大值（32 位平台） | `4294966272` |
    | 单位 | 字节 |
    | 块大小 | `1024` |

    *MySQL 8.0.28 及更早版本*：这设置了用于语句解析和执行的持久缓冲区的大小（以字节为单位）。该缓冲区在语句之间不会被释放。如果您运行复杂查询，较大的 `query_prealloc_size` 值可能有助于提高性能，因为它可以减少服务器在执行查询操作期间进行内存分配的需求。您应该意识到，这并不一定完全消除分配；服务器在某些情况下仍可能分配内存，例如与事务相关的操作或存储程序。

    截至 MySQL 8.0.29，`query_prealloc_size` 已弃用，并设置它不再产生任何效果；您应该期待在将来的 MySQL 版本中删除它。

+   `rand_seed1`

    | 系统变量 | `rand_seed1` |
    | --- | --- |
    | 范围 | 会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `N/A` |
    | 最小值 | `0` |
    | 最大值 | `4294967295` |

    `rand_seed1` 和 `rand_seed2` 变量仅存在于会话变量中，可以设置但不能读取。这些变量（但不是它们的值）会显示在 `SHOW VARIABLES` 的输出中。

    这些变量的目的是支持`RAND()`函数的复制。对于调用`RAND()`的语句，源传递两个值给副本，在副本中用于初始化随机数生成器。副本使用这些值来设置会话变量`rand_seed1`和`rand_seed2`，以便副本上的`RAND()`生成与源上相同的值。

+   `rand_seed2`

    参见`rand_seed1`的描述。

+   `range_alloc_block_size`

    | 命令行格式 | `--range-alloc-block-size=#` |
    | --- | --- |
    | 系统变量 | `range_alloc_block_size` |
    | 作用范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 是 |
    | 类型 | 整数 |
    | 默认值 | `4096` |
    | 最小值 | `4096` |
    | 最大值（64 位平台） | `18446744073709550592` |
    | 最大值 | `4294966272` |
    | 单位 | 字节 |
    | 块大小 | `1024` |

    在进行范围优化时分配的块的大小（以字节为单位）。

    字节编号的块大小为 1024。在存储系统变量值之前，MySQL 服务器会将不是块大小的整数倍的值向下舍入到下一个较低的块大小的整数倍。解析器允许值达到平台的最大无符号整数值（32 位系统为 4294967295 或 2³²−1，64 位系统为 18446744073709551615 或 2⁶⁴−1），但实际最大值是较低的块大小。

+   `range_optimizer_max_mem_size`

    | 命令行格式 | `--range-optimizer-max-mem-size=#` |
    | --- | --- |
    | 系统变量 | `range_optimizer_max_mem_size` |
    | 作用范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `8388608` |
    | 最小值 | `0` |
    | 最大值 | `18446744073709551615` |
    | 单位 | 字节 |

    范围优化器的内存消耗限制。值为 0 表示“无限制”。如果优化器考虑的执行计划使用范围访问方法，但优化器估计所需的内存量将超过限制，则放弃该计划并考虑其他计划。有关更多信息，请参见限制范围优化器的内存使用。

+   `rbr_exec_mode`

    | 系统变量 | `rbr_exec_mode` |
    | --- | --- |
    | 范围 | 会话 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 枚举 |
    | 默认值 | `STRICT` |
    | 有效值 | `STRICT``IDEMPOTENT` |

    供**mysqlbinlog**内部使用。此变量在`IDEMPOTENT`模式和`STRICT`模式之间切换服务器。`IDEMPOTENT`模式导致在**mysqlbinlog**生成的`BINLOG`语句中抑制重复键和未找到键的错误。当在导致与现有数据冲突的服务器上重放基于行的二进制日志时，此模式很有用。**mysqlbinlog**通过在输出中写入以下内容来设置此模式，当您指定`--idempotent`选项时：

    ```sql
    SET SESSION RBR_EXEC_MODE=IDEMPOTENT;
    ```

    从 MySQL 8.0.18 开始，设置此系统变量的会话值不再是受限制的操作。

+   `read_buffer_size`

    | 命令行格式 | `--read-buffer-size=#` |
    | --- | --- |
    | 系统变量 | `read_buffer_size` |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 是 |
    | 类型 | 整数 |
    | 默认值 | `131072` |
    | 最小值 | `8192` |
    | 最大值 | `2147479552` |
    | 单位 | 字节 |
    | 块大小 | `4096` |

    每个为`MyISAM`表执行顺序扫描的线程都为其扫描的每个表分配了这个大小的缓冲区（以字节为单位）。如果进行许多顺序扫描，您可能希望增加此值，默认值为 131072。此变量的值应为 4KB 的倍数。如果设置为不是 4KB 的倍数的值，则其值将向下舍入到最接近的 4KB 的倍数。

    该选项还用于以下上下文中的所有其他存储引擎，除了`InnoDB`：

    +   用于在临时文件中缓存索引（而不是临时表）以便对行进行`ORDER BY`排序。

    +   用于批量插入到分区中。

    +   用于缓存嵌套查询的结果。

    `read_buffer_size`还以另一种存储引擎特定的方式使用：确定`MEMORY`表的内存块大小。

    从 MySQL 8.0.22 开始，`select_into_buffer_size`的值用于替代执行`SELECT INTO DUMPFILE`和`SELECT INTO OUTFILE`语句时使用的 I/O 缓存缓冲区的`read_buffer_size`的值。（在其他情况下，`read_buffer_size`用于 I/O 缓存缓冲区大小。）

    有关不同操作期间内存使用的更多信息，请参见第 10.12.3.1 节，“MySQL 如何使用内存”。

+   `read_only`

    | 命令行格式 | `--read-only[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `read_only` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    如果启用`read_only`系统变量，则服务器不允许客户端更新，除非是具有`CONNECTION_ADMIN`权限（或已弃用的`SUPER`权限）的用户。此变量默认情况下禁用。

    服务器还支持一个`super_read_only`系统变量（默认情况下禁用），具有以下效果：

    +   如果启用`super_read_only`，服务器禁止客户端更新，即使是具有`CONNECTION_ADMIN`或`SUPER`权限的用户。

    +   将`super_read_only`设置为`ON`会隐式地将`read_only`设置为`ON`。

    +   将`read_only`设置为`OFF`会隐式地将`super_read_only`设置为`OFF`。

    当启用`read_only`和`super_read_only`时，服务器仍允许执行以下操作：

    +   由复制线程执行的更新，如果服务器是副本。在复制设置中，可以在副本服务器上启用`read_only`以确保副本仅接受来自源服务器而不是来自客户端的更新。

    +   写入系统表`mysql.gtid_executed`，该表存储未在当前二进制日志文件中出现的已执行事务的 GTID。

    +   使用`ANALYZE TABLE`或`OPTIMIZE TABLE`语句。只读模式的目的是防止对表结构或内容的更改。分析和优化不符合此类更改。这意味着，例如，可以使用**mysqlcheck** `--all-databases` `--analyze` 在只读副本上执行一致性检查。

    +   使用`FLUSH STATUS`语句，这些语句始终写入二进制日志。

    +   对`TEMPORARY`表的操作。

    +   插入到日志表（`mysql.general_log`和`mysql.slow_log`）；参见第 7.4.1 节，“选择通用查询日志和慢查询日志输出目的地”。

    +   对性能模式表的更新，例如`UPDATE`或`TRUNCATE TABLE`操作。

    对于复制源服务器上的`read_only`的更改不会被复制到副本服务器。该值可以在副本上独立设置，与源设置无关。

    尝试启用`read_only`（包括由启用`super_read_only`导致的隐式尝试）时，以下条件适用：

    +   如果存在任何显式锁（使用`LOCK TABLES`获取的锁）或存在未决事务，则尝试失败并出现错误。

    +   如果其他客户端有任何正在进行的语句、活动的`LOCK TABLES WRITE`或正在进行的提交，则尝试将被阻塞，直到锁被释放并语句和事务结束。在等待启用`read_only`期间，其他客户端的表锁请求或开始事务的请求也会被阻塞，直到`read_only`被设置。

    +   如果存在持有元数据锁的活动事务，则尝试将被阻塞，直到这些事务结束。

    +   当你持有全局读锁（通过 `FLUSH TABLES WITH READ LOCK` 获取）时，可以启用 `read_only`，因为这不涉及表锁。

+   `read_rnd_buffer_size`

    | 命令行格式 | `--read-rnd-buffer-size=#` |
    | --- | --- |
    | 系统变量 | `read_rnd_buffer_size` |
    | 作用范围 | 全局, 会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 是 |
    | 类型 | 整数 |
    | 默认值 | `262144` |
    | 最小值 | `1` |
    | 最大值 | `2147483647` |
    | 单位 | 字节 |

    此变量用于从 `MyISAM` 表中读取行，并且对于任何存储引擎，也用于多范围读取优化。

    在按键排序操作后按顺序从 `MyISAM` 表中读取行时，通过此缓冲区读取行以避免磁盘查找。请参见 第 10.2.1.16 节, “ORDER BY 优化”。将变量设置为一个较大的值可以大大提高 `ORDER BY` 的性能。但是，这是为每个客户端分配的缓冲区，因此不应将全局变量设置为一个较大的值。相反，只应在需要运行大查询的客户端内更改会话变量。

    有关不同操作期间内存使用的更多信息，请参见 第 10.12.3.1 节, “MySQL 如何使用内存”。有关多范围读取优化的信息，请参见 第 10.2.1.11 节, “多范围读取优化”。

+   `regexp_stack_limit`

    | 命令行格式 | `--regexp-stack-limit=#` |
    | --- | --- |
    | 系统变量 | `regexp_stack_limit` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `8000000` |
    | 最小值 | `0` |
    | 最大值 | `2147483647` |
    | 单位 | 字节 |

    用于由 `REGEXP_LIKE()` 和类似函数执行的正则表达式匹配操作的内部堆栈的最大可用内存（请参见 第 14.8.2 节, “正则表达式”）。

+   `regexp_time_limit`

    | 命令行格式 | `--regexp-time-limit=#` |
    | --- | --- |
    | 系统变量 | `regexp_time_limit` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `32` |
    | 最小值 | `0` |
    | 最大值 | `2147483647` |

    由`REGEXP_LIKE()`和类似函数执行的正则表达式匹配操作的时间限制（参见第 14.8.2 节，“正则表达式”）。此限制表示匹配引擎执行的最大允许步骤数，因此仅间接影响执行时间。通常，它在毫秒级别。

+   `require_row_format`

    | 引入版本 | 8.0.19 |
    | --- | --- |
    | 系统变量 | `require_row_format` |
    | 作用范围 | 会话 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    此变量仅供复制和**mysqlbinlog**内部服务器使用。它将会话中执行的 DML 事件限制为仅以行为基础的二进制日志格式编码的事件，并且不能创建临时表。不遵守限制的查询将失败。

    将此系统变量的会话值设置为`ON`不需要特权。将此系统变量的会话值设置为`OFF`是一项受限操作，会话用户必须具有足够的特权来设置受限会话变量。参见第 7.1.9.1 节，“系统变量特权”。

+   `require_secure_transport`

    | 命令行格式 | `--require-secure-transport[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `require_secure_transport` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    客户端连接到服务器是否需要使用某种形式的安全传输。当启用此变量时，服务器仅允许使用 TLS/SSL 加密的 TCP/IP 连接，或者使用套接字文件（在 Unix 上）或共享内存（在 Windows 上）的连接。服务器拒绝非安全连接尝试，这些尝试将因`ER_SECURE_TRANSPORT_REQUIRED`错误而失败。

    此功能补充了每个帐户的 SSL 要求，后者优先。例如，如果一个帐户被定义为`REQUIRE SSL`，启用`require_secure_transport`不会使该帐户能够使用 Unix 套接字文件进行连接。

    服务器可能没有可用的安全传输。例如，如果在没有指定任何 SSL 证书或密钥文件的情况下启动 Windows 上的服务器，并且禁用了`shared_memory`系统变量，则不支持任何安全传输。在这些条件下，尝试在启动时启用`require_secure_transport`会导致服务器写入错误日志并退出。尝试在运行时启用该变量会导致`ER_NO_SECURE_TRANSPORTS_CONFIGURED`错误。

    另请参阅将加密连接配置为强制性。

+   `resultset_metadata`

    | 系统变量 | `resultset_metadata` |
    | --- | --- |
    | 范围 | 会话 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 枚举 |
    | 默认值 | `FULL` |
    | 有效值 | `FULL``NONE` |

    对于元数据传输是可选的连接，客户端设置`resultset_metadata`系统变量以控制服务器是否返回结果集元数据。允许的值为`FULL`（返回所有元数据；这是默认值）和`NONE`（不返回元数据）。

    对于不支持元数据的连接，将`resultset_metadata`设置为`NONE`会产生错误。

    有关管理结果集元数据传输的详细信息，请参见可选结果集元数据。

+   `secondary_engine_cost_threshold`

    | 引入版本 | 8.0.16 |
    | --- | --- |
    | 系统变量 | `secondary_engine_cost_threshold` |
    | 范围 | 会话 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 是 |
    | 类型 | 数值 |
    | 默认值 | `100000.000000` |
    | 最小值 | `0` |
    | 最大值 | `DBL_MAX（最大双精度值）` |

    查询转移到辅助引擎的优化器成本阈值。

    用于 HeatWave。参见 MySQL HeatWave 用户指南。

+   `schema_definition_cache`

    | 命令行格式 | `--schema-definition-cache=#` |
    | --- | --- |
    | 系统变量 | `schema_definition_cache` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `256` |
    | 最小值 | `256` |
    | 最大值 | `524288` |

    定义了字典对象缓存中可以保留的模式定义对象的数量限制，包括已使用和未使用的对象。

    未使用的模式定义对象仅在使用数量少于由 `schema_definition_cache` 定义的容量时才保留在字典对象缓存中。

    设置为 `0` 表示模式定义对象仅在使用时保留在字典对象缓存中。

    更多信息，请参见 第 16.4 节，“字典对象缓存”。

+   `secure_file_priv`

    | 命令行格式 | `--secure-file-priv=dir_name` |
    | --- | --- |
    | 系统变量 | `secure_file_priv` |
    | 作用域 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 字符串 |
    | 默认值 | `平台特定` |
    | 有效值 | `空字符串``dirname``NULL` |

    此变量用于限制数据导入和导出操作的影响，例如由 `LOAD DATA` 和 `SELECT ... INTO OUTFILE` 语句以及 `LOAD_FILE()` 函数执行的操作。这些操作仅允许具有 `FILE` 权限的用户执行。

    `secure_file_priv` 可以设置如下：

    +   如果为空，该变量没有任何效果。这不是一个安全的设置。

    +   如果设置为目录的名称，服务器将限制导入和导出操作仅使用该目录中的文件。该目录必须存在；服务器不会创��它。

    +   如果设置为`NULL`，服务器将禁用导入和导出操作。

    默认值是平台特定的，取决于 `INSTALL_LAYOUT` **CMake** 选项的值，如下表所示。如果您正在从源代码构建，请使用 `INSTALL_SECURE_FILE_PRIVDIR` **CMake** 选项显式指定默认的 `secure_file_priv` 值。

    | `INSTALL_LAYOUT` 值 | 默认的 `secure_file_priv` 值 |
    | --- | --- |
    | `STANDALONE` | 空 |
    | `DEB`, `RPM`, `SVR4` | `/var/lib/mysql-files` |
    | 否则 | 在 `CMAKE_INSTALL_PREFIX` 值下的 `mysql-files` 目录 |

    服务器在启动时检查`secure_file_priv`的值，并在值不安全时向错误日志写入警告。如果非`NULL`值被视为不安全，当它为空时，或者值为数据目录或其子目录，或者是所有用户都可以访问的目录时。如果`secure_file_priv`设置为不存在的路径，则服务器将向错误日志写入错误消息并退出。

+   `select_into_buffer_size`

    | 命令行格式 | `--select-into-buffer-size=#` |
    | --- | --- |
    | 引入版本 | 8.0.22 |
    | 系统变量 | `select_into_buffer_size` |
    | 作用范围 | 全局, 会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 是 |
    | 类型 | 整数 |
    | 默认值 | `131072` |
    | 最小值 | `8192` |
    | 最大值 | `2147479552` |
    | 单位 | 字节 |
    | 块大小 | `4096` |

    在使用`SELECT INTO OUTFILE`或`SELECT INTO DUMPFILE`将数据转储到一个或多个文件以进行备份创建、数据迁移或其他目的时，写入通常会被缓冲，然后触发大量写入 I/O 活动到磁盘或其他存储设备，并阻塞对延迟更敏感的其他查询。您可以使用此变量来控制用于将数据写入存储设备的缓冲区的大小，以确定何时应发生缓冲区同步，从而防止发生上述类型的写入阻塞。

    `select_into_buffer_size`会覆盖为`read_buffer_size`设置的任何值。(`select_into_buffer_size`和`read_buffer_size`具有相同的默认值、最大值和最小值。) 您还可以使用`select_into_disk_sync_delay`来设置稍后要遵守的超时，每次发生同步时。

    截至 MySQL 8.0.27，设置此系统变量的会话值是受限制的操作。会话用户必须具有足够的权限来设置受限制的会话变量。请参阅第 7.1.9.1 节，“系统变量权限”。

+   `select_into_disk_sync`

    | 命令行格式 | `--select-into-disk-sync={ON&#124;OFF}` |
    | --- | --- |
    | 引入版本 | 8.0.22 |
    | 系统变量 | `select_into_disk_sync` |
    | 作用范围 | 全局, 会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 是 |
    | 类型 | 布尔 |
    | 默认值 | `OFF` |
    | 有效值 | `OFF``ON` |

    当设置为`ON`时，通过使用`select_into_buffer_size`，启用对输出文件的写入缓冲区同步，该文件由长时间运行的`SELECT INTO OUTFILE`或`SELECT INTO DUMPFILE`语句写入。

+   `select_into_disk_sync_delay`

    | 命令行格式 | `--select-into-disk-sync-delay=#` |
    | --- | --- |
    | 引入版本 | 8.0.22 |
    | 系统变量 | `select_into_disk_sync_delay` |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 是 |
    | 类型 | 整数 |
    | 默认值 | `0` |
    | 最小值 | `0` |
    | 最大值 | `31536000` |
    | 单位 | 毫秒 |

    当通过长时间运行的`SELECT INTO OUTFILE`或`SELECT INTO DUMPFILE`语句启用对输出文件的写入缓冲区同步时，由`select_into_disk_sync`设置此变量，该变量设置同步后的可选延迟时间（以毫秒为单位）。`0`（默认值）表示没有延迟。

    截至 MySQL 8.0.27，设置此系统变量的会话值是受限制的操作。会话用户必须具有足够权限来设置受限制的会话变量。请参阅第 7.1.9.1 节，“系统变量权限”。

+   `session_track_gtids`

    | 命令行格式 | `--session-track-gtids=value` |
    | --- | --- |
    | 系统变量 | `session_track_gtids` |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 枚举 |
    | 默认值 | `OFF` |
    | 有效值 | `OFF``OWN_GTID``ALL_GTIDS` |

    控制服务器是否将 GTID 返回给客户端，使客户端能够使用它们来跟踪服务器状态。根据变量值，在执行每个事务结束时，服务器的 GTID 被捕获并作为确认的一部分返回给客户端。`session_track_gtids`的可能值如下：

    +   `OFF`：服务器不会将 GTID 返回给客户端。这是默认值。

    +   `OWN_GTID`：服务器返回客户端在当前会话中成功提交的所有事务的 GTID。通常，这是最后一个提交的事务的单个 GTID，但如果单个客户端请求导致多个事务，则服务器返回包含所有相关 GTID 的 GTID 集。

    +   `ALL_GTIDS`：服务器返回其`gtid_executed`系统变量的全局值，该值在事务成功提交后读取。除了刚刚提交的事务的 GTID 外，此 GTID 集还包括服务器上由任何客户端提交的所有事务，并且可以包括在当前被确认的事务提交后提交的事务。

    在事务上下文中无法设置`session_track_gtids`。

    有关会话状态跟踪的更多信息，请参见第 7.1.18 节，“服务器跟踪客户端会话状态”。

+   `session_track_schema`

    | 命令行格式 | `--session-track-schema[={OFF | ON}]` |
    | --- | --- | --- |
    | 系统变量 | `session_track_schema` |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `ON` |

    控制服务器是否跟踪当前会话中设置默认模式（数据库）的更改，并通知客户端使模式名称可用。

    如果启用了模式名称跟踪器，则每次设置默认模式时都会发生名称通知，即使新模式名称与旧名称相同。

    有关会话状态跟踪的更多信息，请参见第 7.1.18 节，“服务器跟踪客户端会话状态”。

+   `session_track_state_change`

    | 命令行格式 | `--session-track-state-change[={OFF | ON}]` |
    | --- | --- | --- |
    | 系统变量 | `session_track_state_change` |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    控制服务器是否跟踪当前会话状态的更改，并在状态更改发生时通知客户端。客户端会话状态的这些属性可能会报告更改：

    +   默认模式（数据库）。

    +   系统变量的会话特定值。

    +   用户定义变量。

    +   临时表。

    +   预编译语句。

    如果启用了会话状态跟踪器，则会在涉及跟踪的会话属性的每次更改时进行通知，即使新属性值与旧值相同。例如，将用户定义变量设置为其当前值会导致通知。

    `session_track_state_change`变量仅控制更改发生时的通知，而不是更改的内容。例如，当设置默认模式或分配跟踪会话系统变量时，会发生状态更改通知，但通知不包括模式名称或变量值。要接收模式名称或会话系统变量值的通知，请分别使用`session_track_schema`或`session_track_system_variables`系统变量。

    注意

    将值分配给`session_track_state_change`本身不被视为状态更改，也不会报告为此。但是，如果其名称在`session_track_system_variables`的值中列出，对其的任何分配确实会导致通知新值。

    有关会话状态跟踪的更多信息，请参阅 Section 7.1.18, “服务器跟踪客户端会话状态”。

+   `session_track_system_variables`

    | 命令行格式 | `--session-track-system-variables=#` |
    | --- | --- |
    | 系统变量 | `session_track_system_variables` |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 字符串 |
    | 默认值 | `time_zone, autocommit, character_set_client, character_set_results, character_set_connection` |

    控制服务器是否跟踪会话系统变量的分配并通知客户端每个分配变量的名称和值。变量值是一个逗号分隔的变量列表，用于跟踪分配。默认情况下，通知已启用`time_zone`, `autocommit`, `character_set_client`, `character_set_results`, 和`character_set_connection`。（后三个变量是受`SET NAMES`影响的变量。）

    要启用每个处理语句的语句 ID 的显示，请使用`statement_id`变量。例如：

    ```sql
    mysql>  SET @@SESSION.session_track_system_variables='statement_id'
    mysql>  SELECT 1;
    +---+
    | 1 |
    +---+
    | 1 |
    +---+
    1 row in set (0.0006 sec)
    Statement ID: 603835
    ```

    特殊值`*`使服务器跟踪对所有会话变量的赋值。如果给出此值，必须单独指定，而不带有特定的系统变量名称。此值还使每个成功处理的语句的语句 ID 的显示变得可能。

    要禁用会话变量赋值的通知，请将`session_track_system_variables`设置为空字符串。

    如果启用了会话系统变量跟踪，即使新值与旧值相同，对跟踪的会话变量的所有赋值也会发出通知。

    有关会话状态跟踪的更多信息，请参见第 7.1.18 节，“服务器跟踪客户端会话状态”。

+   `session_track_transaction_info`

    | 命令行格式 | `--session-track-transaction-info=value` |
    | --- | --- |
    | 系统变量 | `session_track_transaction_info` |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 枚举 |
    | 默认值 | `OFF` |
    | 有效值 | `OFF``STATE``CHARACTERISTICS` |

    控制服务器是否跟踪当前会话中事务的状态和特征，并通知客户端使此信息可用。允许使用这些`session_track_transaction_info`值：

    +   `OFF`：禁用事务状态跟踪。这是默认设置。

    +   `STATE`：启用事务状态跟踪而不跟踪特征。状态跟踪使客户端能够确定事务是否正在进行中，以及是否可以将其移动到另一个会话而不会被回滚。

    +   `CHARACTERISTICS`：启用事务状态跟踪，包括特征跟踪。特征跟踪使客户端能够确定如何在另一个会话中重新启动事务，以使其具有与原始会话相同的特征。以下特征对此目的相关：

        ```sql
        ISOLATION LEVEL
        READ ONLY
        READ WRITE
        WITH CONSISTENT SNAPSHOT
        ```

    为了安全地将事务重新定位到另一个会话，客户端必须不仅跟踪事务状态，还必须跟踪事务特征。此外，客户端必须跟踪`transaction_isolation`和`transaction_read_only`系统变量，以正确确定会话默认值。（要跟踪这些变量，请在`session_track_system_variables`系统变量的值中列出它们。）

    有关会话状态跟踪的更多信息，请参见第 7.1.18 节，“服务器跟踪客户端会话状态”。

+   `sha256_password_auto_generate_rsa_keys`

    | 命令行格式 | `--sha256-password-auto-generate-rsa-keys[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `sha256_password_auto_generate_rsa_keys` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `ON` |

    服务器使用此变量来确定是否在数据目录中自动生成 RSA 私钥/公钥对文件（如果它们尚不存在）。

    在启动时，如果满足以下所有条件，服务器将自动在数据目录中生成 RSA 私钥/公钥对文件：启用了`sha256_password_auto_generate_rsa_keys`或`caching_sha2_password_auto_generate_rsa_keys`系统变量；未指定 RSA 选项；数据目录中缺少 RSA 文件。这些密钥对文件使得通过 RSA 在未加密连接上进行安全密码交换成为可能，用于由`sha256_password`或`caching_sha2_password`插件认证的帐户；参见第 8.4.1.3 节，“SHA-256 可插拔认证”和第 8.4.1.2 节，“缓存 SHA-2 可插拔认证”。

    有关 RSA 文件自动生成的更多信息，包括文件名和特性，请参见第 8.3.3.1 节，“使用 MySQL 创建 SSL 和 RSA 证书和密钥”

    `auto_generate_certs`系统变量相关，但控制着用于使用 SSL��行安全连接所需的 SSL 证书和密钥文件的自动生成。

+   `sha256_password_private_key_path`

    | 命令行格式 | `--sha256-password-private-key-path=file_name` |
    | --- | --- |
    | 系统变量 | `sha256_password_private_key_path` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 文件名 |
    | 默认值 | `private_key.pem` |

    此变量的值是`sha256_password`认证插件的 RSA 私钥文件的路径名。如果文件以相对路径命名，则解释为相对于服务器数据目录。文件必须采用 PEM 格式。

    重要

    因为此文件存储私钥，其访问模式应受限制，以便只有 MySQL 服务器可以读取它。

    有关`sha256_password`的信息，请参见第 8.4.1.3 节，“SHA-256 可插拔认证”。

+   `sha256_password_proxy_users`

    | 命令行格式 | `--sha256-password-proxy-users[={OFF | ON}]` |
    | --- | --- | --- |
    | 系统变量 | `sha256_password_proxy_users` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    此变量控制`sha256_password`内置认证插件是否支持代理用户。除非启用了`check_proxy_users`系统变量，否则不起作用。有关用户代理的信息，请参见第 8.2.19 节，“代理用户”。

+   `sha256_password_public_key_path`

    | 命令行格式 | `--sha256-password-public-key-path=file_name` |
    | --- | --- |
    | 系统变量 | `sha256_password_public_key_path` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 文件名 |
    | 默认值 | `public_key.pem` |

    此变量的值是`sha256_password`认证插件的 RSA 公钥文件的路径名。如果文件以相对路径命名，则解释为相对于服务器数据目录。文件必须采用 PEM 格式。因为此文件存储公钥，所以可以自由分发给客户端用户。（使用 RSA 密码加密连接到服务器时明确指定公钥的客户端必须使用与服务器使用的公钥相同的公钥。）

    有关`sha256_password`的信息，包括客户端如何指定 RSA 公钥的信息，���参见第 8.4.1.3 节，“SHA-256 可插拔认证”。

+   `shared_memory`

    | 命令行格式 | `--shared-memory[={OFF | ON}]` |
    | --- | --- | --- |
    | 系统变量 | `shared_memory` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 平台特定 | Windows |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    (仅限 Windows。) 服务器是否允许共享内存连接。

+   `shared_memory_base_name`

    | 命令行格式 | `--shared-memory-base-name=name` |
    | --- | --- |
    | 系统变量 | `shared_memory_base_name` |
    | 作用范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 平台特定 | Windows |
    | 类型 | 字符串 |
    | 默认值 | `MYSQL` |

    (仅限 Windows。) 用于共享内存连接的共享内存的名称。在单个物理机器上运行多个 MySQL 实例时很有用。默认名称为`MYSQL`。名称区分大小写。

    此变量仅在服务器启动时启用了支持共享内存连接的`shared_memory`系统变量时适用。

+   `show_create_table_skip_secondary_engine`

    | 命令行格式 | `--show-create-table-skip-secondary-engine[={OFF&#124;ON}]` |
    | --- | --- |
    | 引入版本 | 8.0.18 |
    | 系统变量 | `show_create_table_skip_secondary_engine` |
    | 作用范围 | 会话 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 是 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    启用`show_create_table_skip_secondary_engine`会导致`SECONDARY ENGINE`子句在`SHOW CREATE TABLE`输出和由**mysqldump**实用程序转储的`CREATE TABLE`语句中被排除。

    **mysqldump**提供了`--show-create-skip-secondary-engine`选项。当指定时，它会在转储操作的持续时间内启用`show_create_table_skip_secondary_engine`系统变量。

    尝试在不支持`show_create_table_skip_secondary_engine`变量的 MySQL 8.0.18 之前的版本上使用带有`--show-create-skip-secondary-engine`选项的**mysqldump**操作会导致错误。

    用于 HeatWave。请参阅 MySQL HeatWave 用户指南。

+   `show_create_table_verbosity`

    | 命令行格式 | `--show-create-table-verbosity[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `show_create_table_verbosity` |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    `SHOW CREATE TABLE`通常不会显示`ROW_FORMAT`表选项，如果行格式是默认格式。启用此变量会导致`SHOW CREATE TABLE`无论是否为默认格式都显示`ROW_FORMAT`。

+   `show_gipk_in_create_table_and_information_schema`

    | 命令行格式 | `--show-gipk-in-create-table-and-information-schema[={OFF&#124;ON}]` |
    | --- | --- |
    | 引入版本 | 8.0.30 |
    | 系统变量 | `show_gipk_in_create_table_and_information_schema` |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `ON` |

    生成的不可见主键在`SHOW`语句的输出和信息模式表中是否可见。当此变量设置为`OFF`时，这些键不会显示。

    此变量不会被复制。

    更多信息，请参见第 15.1.20.11 节，“生成的不可见主键”。

+   `show_old_temporals`

    | 命令行格式 | `--show-old-temporals[={OFF&#124;ON}]` |
    | --- | --- |
    | 已弃用 | 是 |
    | 系统变量 | `show_old_temporals` |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    `SHOW CREATE TABLE` 输出是否包含用于标记发现为 5.6.4 版本之前格式的时间列（`TIME`、`DATETIME` 和 `TIMESTAMP` 列不支持分数秒精度）。默认情况下，此变量已禁用。如果启用，`SHOW CREATE TABLE` 输出如下所示：

    ```sql
    CREATE TABLE `mytbl` (
      `ts` timestamp /* 5.5 binary format */ NOT NULL DEFAULT CURRENT_TIMESTAMP,
      `dt` datetime /* 5.5 binary format */ DEFAULT NULL,
      `t` time /* 5.5 binary format */ DEFAULT NULL
    ) DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci
    ```

    `INFORMATION_SCHEMA COLUMNS` 表的 `COLUMN_TYPE` 列的输出受到类似影响。

    此变量已被弃用，并可能在未来的 MySQL 版本中被移除。

    截至 MySQL 8.0.27 版本，设置此系统变量的会话值是受限制的操作。会话用户必须具有足够的权限来设置受限制的会话变量。请参见 第 7.1.9.1 节，“系统变量权限”。

+   `skip_external_locking`

    | 命令行格式 | `--skip-external-locking[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `skip_external_locking` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔 |
    | 默认值 | `ON` |

    如果 **mysqld** 使用外部锁定（系统锁定），则此值为 `OFF`，如果禁用外部锁定，则为 `ON`。这仅影响 `MyISAM` 表的访问。

    此变量由 `--external-locking` 或 `--skip-external-locking` 选项设置。默认情况下禁用外部锁定。

    外部锁定仅影响 `MyISAM` 表的访问。有关更多信息，包括可以使用和不可使用的条件，请参见 第 10.11.5 节，“外部锁定”。

+   `skip_name_resolve`

    | 命令行格式 | `--skip-name-resolve[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `skip_name_resolve` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔 |
    | 默认值 | `OFF` |

    在检查客户端连接时是否解析主机名。如果此变量为`OFF`，**mysqld**在检查客户端连接时解析主机名。如果为`ON`，**mysqld**仅使用 IP 地址；在这种情况下，授权表中的所有`Host`列值必须是 IP 地址。请参阅 Section 7.1.12.3, “DNS Lookups and the Host Cache”。

    根据系统的网络配置和账户的`Host`值，客户端可能需要使用显式的`--host`选项进行连接，例如`--host=127.0.0.1`或`--host=::1`。

    尝试连接到主机`127.0.0.1`通常会解析为`localhost`账户。但是，如果服务器启用了`skip_name_resolve`，这将失败。如果您打算这样做，请确保存在一个可以接受连接的账户。例如，要能够使用`--host=127.0.0.1`或`--host=::1`连接为`root`，请创建这些账户：

    ```sql
    CREATE USER 'root'@'127.0.0.1' IDENTIFIED BY '*root-password*';
    CREATE USER 'root'@'::1' IDENTIFIED BY '*root-password*';
    ```

+   `skip_networking`

    | ��令行格式 | `--skip-networking[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `skip_networking` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    此变量控制服务器是否允许 TCP/IP 连接。默认情况下，它被禁用（允许 TCP 连接）。如果启用，服务器仅允许本地（非 TCP/IP）连接，并且所有与**mysqld**的交互必须使用命名管道或共享内存（在 Windows 上）或 Unix 套接字文件（在 Unix 上）进行。这个选项强烈建议用于只允许本地客户端的系统。请参阅 Section 7.1.12.3, “DNS Lookups and the Host Cache”。

    因为使用`--skip-grant-tables`启动服务器会禁用身份验证检查，服务器在这种情况下还会通过启用`skip_networking`来禁用远程连接。

+   `skip_show_database`

    | 命令行格式 | `--skip-show-database` |
    | --- | --- |
    | 系统变量 | `skip_show_database` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔 |
    | 默认值 | `OFF` |

    这可以防止没有`SHOW DATABASES`权限的人使用`SHOW DATABASES`语句。如果您担心用户能够看到其他用户拥有的数据库，这可以提高安全性。其效果取决于`SHOW DATABASES`权限：如果变量值为`ON`，则只有拥有`SHOW DATABASES`权限的用户才允许使用`SHOW DATABASES`语句，并且该语句显示所有数据库名称。如果值为`OFF`，则允许所有用户使用`SHOW DATABASES`语句，但只显示用户具有`SHOW DATABASES`或其他权限的数据库名称。

    注意

    因为任何静态全局权限都被视为所有数据库的权限，任何静态全局权限都使用户能够使用`SHOW DATABASES`或通过检查`INFORMATION_SCHEMA`的`SCHEMATA`表来查看所有数据库名称，除了在数据库级别通过部分撤销限制的数据库。

+   `slow_launch_time`

    | 命令行格式 | `--slow-launch-time=#` |
    | --- | --- |
    | 系统变量 | `slow_launch_time` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `2` |
    | 最小值 | `0` |
    | 最大值 | `31536000` |
    | 单位 | 秒 |

    如果创建一个线程花费的时间超过这么多秒，服务器会增加`Slow_launch_threads`状态变量。

+   `slow_query_log`

    | 命令行格式 | `--slow-query-log[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `slow_query_log` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔 |
    | 默认值 | `OFF` |

    慢查询日志是否启用。该值可以为 0（或`OFF`）以禁用日志，或者为 1（或`ON`）以启用日志。日志输出的目的地由`log_output`系统变量控制；如果该值为`NONE`，即使启用了日志，也不会写入任何日志条目。

    “慢查询”由`long_query_time`变量的值确定。请参阅第 7.4.5 节，“慢查询日志”。

+   `slow_query_log_file`

    | 命令行格式 | `--slow-query-log-file=file_name` |
    | --- | --- |
    | 系统变量 | `slow_query_log_file` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 文件名 |
    | 默认值 | `host_name-slow.log` |

    慢查询日志文件的名称。默认值为`*`host_name`*-slow.log`，但初始值可以通过`--slow_query_log_file`选项更改。

+   `socket`

    | 命令行格式 | `--socket={file_name&#124;pipe_name}` |
    | --- | --- |
    | 系统变量 | `socket` |
    | 作用范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 字符串 |
    | 默认值（Windows） | `MySQL` |
    | 默认值（其他） | `/tmp/mysql.sock` |

    在 Unix 平台上，此变量是用于本地客户端连接的套接字文件的名称。默认值为`/tmp/mysql.sock`。（对于某些发行格式，目录可能不同，例如 RPM 的`/var/lib/mysql`。）

    在 Windows 上，此变量是用于本地客户端连接的命名管道的名称。默认值为`MySQL`（不区分大小写）。

+   `sort_buffer_size`

    | 命令行格式 | `--sort-buffer-size=#` |
    | --- | --- |
    | 系统变量 | `sort_buffer_size` |
    | 作用范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 是 |
    | 类型 | 整数 |
    | 默认值 | `262144` |
    | 最小值 | `32768` |
    | 最大值（Windows） | `4294967295` |
    | 最大值（其他，64 位平台） | `18446744073709551615` |
    | 最大值（其他，32 位平台） | `4294967295` |
    | 单位 | 字节 |

    每个必须执行排序的会话都会分配这个大小的缓冲区。`sort_buffer_size` 不针对任何存储引擎具体，而是以一般方式应用于优化。至少，`sort_buffer_size` 的值必须足够大，以容纳排序缓冲区中的十五个元组。此外，增加 `max_sort_length` 的值可能需要增加 `sort_buffer_size` 的值。有关更多信息，请参见 Section 10.2.1.16, “ORDER BY Optimization”

    如果在 `SHOW GLOBAL STATUS` 输出中每秒看到许多 `Sort_merge_passes`，您可以考虑增加 `sort_buffer_size` 的值，以加快无法通过查询优化或改进索引来改善的 `ORDER BY` 或 `GROUP BY` 操作的速度。

    优化器会尝试计算所需空间，但可以分配更多，直至达到限制。在全局设置比所需更大的情况下，会减慢大多数执行排序的查询。最好将其增加为会话设置，仅适用于需要更大尺寸的会话。在 Linux 上，存在 256KB 和 2MB 的阈值，超过这些值可能会显著减慢内存分配速度，因此您应考虑保持在这些值以下。尝试实验以找到适合您工作负载的最佳值。请参见 Section B.3.3.5, “Where MySQL Stores Temporary Files”。

    `sort_buffer_size` 的最大允许设置为 4GB−1。对于 64 位平台，允许使用更大的值（除了 64 位 Windows，对于该平台，大值会被截断为 4GB−1 并附带警告）。

+   `sql_auto_is_null`

    | 系统变量 | `sql_auto_is_null` |
    | --- | --- |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 是 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    如果启用了此变量，则在成功插入自动生成的 `AUTO_INCREMENT` 值的语句之后，您可以通过发出以下形式的语句找到该值：

    ```sql
    SELECT * FROM *tbl_name* WHERE *auto_col* IS NULL
    ```

    如果语句返回一行，则返回的值与调用`LAST_INSERT_ID()`函数的值相同。有关详细信息，包括多行插入后的返回值，请参阅第 14.15 节，“信息函数”。如果没有成功插入`AUTO_INCREMENT`值，则`SELECT`语句不返回任何行。

    通过使用`IS NULL`比较来检索`AUTO_INCREMENT`值的行为被一些 ODBC 程序（如 Access）使用。请参阅获取自增值。通过将`sql_auto_is_null`设置为`OFF`可以禁用此行为。

    在 MySQL 8.0.16 之前，将`WHERE *`auto_col`* IS NULL`转换为`WHERE *`auto_col`* = LAST_INSERT_ID()`仅在执行语句时执行，因此在执行期间`sql_auto_is_null`的值决定了查询是否被转换。在 MySQL 8.0.16 及更高版本中，转换在语句准备期间执行。

    `sql_auto_is_null`的默认值为`OFF`。

+   `sql_big_selects`

    | 系统变量 | `sql_big_selects` |
    | --- | --- |
    | 作用范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 是 |
    | 类型 | 布尔值 |
    | 默认值 | `ON` |

    如果设置为`OFF`，MySQL 将中止可能需要很长时间才能执行的`SELECT`语句（即，优化器估计检查的行数超过`max_join_size`值的语句）。当发出不明智的`WHERE`语句时，这是有用的。新连接的默认值为`ON`，允许所有`SELECT`语句。

    如果您将`max_join_size`系统变量设置为`DEFAULT`以外的值，则`sql_big_selects`将被设置为`OFF`。

+   `sql_buffer_result`

    | 系统变量 | `sql_buffer_result` |
    | --- | --- |
    | 作用范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 是 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    如果启用，`sql_buffer_result` 强制将 `SELECT` 语句的结果放入临时表中。这有助于 MySQL 尽早释放表锁，并且在向客户端发送结果需要很长时间的情况下可能是有益的。默认值为`OFF`。

+   `sql_generate_invisible_primary_key`

    | 命令行格式 | `--sql-generate-invisible-primary-key[={OFF&#124;ON}]` |
    | --- | --- |
    | 引入版本 | 8.0.30 |
    | 系统变量 | `sql_generate_invisible_primary_key` |
    | 作用范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    此服务器是否为创建时没有主键的任何`InnoDB`表添加生成的不可见主键。

    此变量不会被复制。此外，即使在副本上设置了该变量，复制应用程序线程也会忽略它；这意味着，默认情况下，副本不会为源上创建时没有主键的任何复制表生成主键。在 MySQL 8.0.32 及更高版本中，您可以通过在`CHANGE REPLICATION SOURCE TO`语句中设置`REQUIRE_TABLE_PRIMARY_KEY_CHECK = GENERATE`来导致副本为这些表生成不可见主键，可选择指定一个复制通道。

    有关更多信息和示例，请参见第 15.1.20.11 节，“生成不可见主键”。

+   `sql_log_off`

    | 系统变量 | `sql_log_off` |
    | --- | --- |
    | 作用范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |
    | 有效值 | `OFF`（启用记录）`ON`（禁用记录） |

    此变量控制当前会话是否禁用对一般查询日志的记录（假设一般查询日志本身已启用）。默认值为`OFF`（即启用记录）。要为当前会话禁用或启用一般查询日志，请将会话`sql_log_off`变量设置为`ON`或`OFF`。

    设置此系统变量的会话值是受限操作。会话用户必须具有足够的权限来设置受限会话变量。请参见第 7.1.9.1 节，“系统变量权限”。

+   `sql_mode`

    | 命令行格式 | `--sql-mode=name` |
    | --- | --- |
    | 系统变量 | `sql_mode` |
    | 作用范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 是 |
    | 类型 | 集合 |
    | 默认值 | `ONLY_FULL_GROUP_BY STRICT_TRANS_TABLES NO_ZERO_IN_DATE NO_ZERO_DATE ERROR_FOR_DIVISION_BY_ZERO NO_ENGINE_SUBSTITUTION` |
    | 有效值 | `ALLOW_INVALID_DATES``ANSI_QUOTES``ERROR_FOR_DIVISION_BY_ZERO``HIGH_NOT_PRECEDENCE``IGNORE_SPACE``NO_AUTO_VALUE_ON_ZERO``NO_BACKSLASH_ESCAPES``NO_DIR_IN_CREATE``NO_ENGINE_SUBSTITUTION``NO_UNSIGNED_SUBTRACTION``NO_ZERO_DATE``NO_ZERO_IN_DATE``ONLY_FULL_GROUP_BY``PAD_CHAR_TO_FULL_LENGTH``PIPES_AS_CONCAT``REAL_AS_FLOAT``STRICT_ALL_TABLES``STRICT_TRANS_TABLES``TIME_TRUNCATE_FRACTIONAL` |

    当前服务器 SQL 模式，可以动态设置。详情请参见第 7.1.11 节，“服务器 SQL 模式”。

    注意

    MySQL 安装程序可能在安装过程中配置 SQL 模式。

    如果 SQL 模式与默认值或您期望的值不同，请检查服务器在启动时读取的选项文件中的设置。

+   `sql_notes`

    | 系统变量 | `sql_notes` |
    | --- | --- |
    | 作用范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `ON` |

    如果启用（默认情况下），`Note`级别的诊断会增加`warning_count`，服务器会记录它们。如果禁用，`Note`诊断不会增加`warning_count`，服务器也不会记录它们。**mysqldump**包含输出以禁用此变量，以便重新加载转储文件时不会因不影响重新加载操作完整性的事件而产生警告。

+   `sql_quote_show_create`

    | 系统变量 | `sql_quote_show_create` |
    | --- | --- |
    | 作用范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `ON` |

    如果启用（默认情况下），服务器会为`SHOW CREATE TABLE`和`SHOW CREATE DATABASE`语句中的标识符加引号。如果禁用，则不会加引号。默认情况下启用此选项，以便对需要引号的标识符进行复制。请参阅 Section 15.7.7.10, “SHOW CREATE TABLE Statement”和 Section 15.7.7.6, “SHOW CREATE DATABASE Statement”。

+   `sql_require_primary_key`

    | 命令行格式 | `--sql-require-primary-key[={OFF&#124;ON}]` |
    | --- | --- |
    | 引入版本 | 8.0.13 |
    | 系统变量 | `sql_require_primary_key` |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 是 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    创建新表或更改现有表结构的语句是否强制要求表具有主键。

    设置此系统变量的会话值是受限制的操作。会话用户必须具有足够权限来设置受限制的会话变量。请参阅 Section 7.1.9.1, “System Variable Privileges”。

    启用此变量有助于避免在表没有主键时可能出现的基于行的复制性能问题。假设一张表没有主键，更新或删除操作会修改多行。在复制源服务器上，此操作可以使用单个表扫描执行，但在使用基于行的复制进行复制时，会导致在副本上要修改的每行都进行表扫描。有了主键，这些表扫描就不会发生。

    `sql_require_primary_key` 适用于基本表和`TEMPORARY`表，其值的更改会被复制到副本服务器。从 MySQL 8.0.18 开始，它仅适用于可以参与复制的存储引擎。

    启用时，`sql_require_primary_key` 会产生以下效果：

    +   尝试创建没有主键的新表会失败并显示错误。这包括`CREATE TABLE ... LIKE`。也包括`CREATE TABLE ... SELECT`，除非`CREATE TABLE`部分包含主键定义。

    +   尝试从现有表中删除主键会失败并显示错误，但在同一`ALTER TABLE`语句中删除主键并添加主键是允许的。

        即使表中还包含`UNIQUE NOT NULL`索引，删除主键也会失败。

    +   尝试导入没有主键的表会失败并显示错误。

    `CHANGE REPLICATION SOURCE TO` 语句（MySQL 8.0.23 及更高版本）或 `CHANGE MASTER TO` 语句（MySQL 8.0.23 之前）的 `REQUIRE_TABLE_PRIMARY_KEY_CHECK` 选项使得复制品可以选择自己的主键检查策略。当为复制通道设置该选项为 `ON` 时，复制品在复制操作中始终使用 `sql_require_primary_key` 系统变量的值为 `ON`，需要主键。当该选项设置为 `OFF` 时，复制品在复制操作中始终使用 `sql_require_primary_key` 系统变量的值为 `OFF`，即使源需要主键，也不需要主键。当 `REQUIRE_TABLE_PRIMARY_KEY_CHECK` 选项设置为 `STREAM` 时，这是默认值，复制品使用从源复制的每个事务的值。对于 `REQUIRE_TABLE_PRIMARY_KEY_CHECK` 选项的 `STREAM` 设置，如果在复制通道中使用权限检查，则 `PRIVILEGE_CHECKS_USER` 帐户需要具有足够权限来设置受限制的会话变量，以便它可以设置 `sql_require_primary_key` 系统变量的会话值。对于 `ON` 或 `OFF` 设置，帐户不需要这些权限。更多信息，请参见 第 19.3.3 节，“复制权限检查”。

+   `sql_safe_updates`

    | 系统变量 | `sql_safe_updates` |
    | --- | --- |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 是 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    如果启用了此变量，那么在 `UPDATE` 和 `DELETE` 语句中，如果没有在 `WHERE` 子句或 `LIMIT` 子句中使用键，则会产生错误。这样可以捕捉那些未正确使用键且可能更改或删除大量行的 `UPDATE` 和 `DELETE` 语句。默认值为 `OFF`。

    对于**mysql**客户端，可以通过使用`--safe-updates`选项启用`sql_safe_updates`。更多信息，请参阅使用安全更新模式（--safe-updates）。

+   `sql_select_limit`

    | 系统变量 | `sql_select_limit` |
    | --- | --- |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 是 |
    | 类型 | 整数 |
    | 默认值 | `18446744073709551615` |
    | 最小值 | `0` |
    | 最大值 | `18446744073709551615` |

    从`SELECT`语句返回的最大行数。更多信息，请参阅使用安全更新模式（--safe-updates）。

    新连接的默认值是服务器允许每个表的最大行数。典型的默认值为(2³²)−1 或(2⁶⁴)−1。如果已更改限制，则可以通过分配`DEFAULT`值来恢复默认值。

    如果`SELECT`有`LIMIT`子句，则`LIMIT`优先于`sql_select_limit`的值。

+   `sql_warnings`

    | 系统变量 | `sql_warnings` |
    | --- | --- |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    此变量控制单行`INSERT`语句在出现警告时是否生成信息字符串。默认值为`OFF`。将值设置为`ON`以生成信息字符串。

+   `ssl_ca`

    | 命令行格式 | `--ssl-ca=file_name` |
    | --- | --- |
    | 系统变量 | `ssl_ca` |
    | 范围 | 全局 |
    | 动态（≥ 8.0.16） | 是 |
    | 动态（≤ 8.0.15） | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 文件名 |
    | 默认值 | `NULL` |

    证书颁发机构（CA）证书文件的 PEM 格式路径名。该文件包含一组受信任的 SSL 证书颁发机构。

    截至 MySQL 8.0.16 版本，此变量是动态的，可以在运行时进行修改，以影响服务器在执行`ALTER INSTANCE RELOAD TLS`后建立的新连接所使用的 TLS 上下文，或者在变量值被持久化后重新启动。请参见用于加密连接的服务器端运行时配置和监控。在 MySQL 8.0.16 之前，此变量只能在服务器启动时设置。

+   `ssl_capath`

    | 命令行格式 | `--ssl-capath=dir_name` |
    | --- | --- |
    | 系统变量 | `ssl_capath` |
    | 作用域 | 全局 |
    | 动态（≥ 8.0.16） | 是 |
    | 动态（≤ 8.0.15） | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 目录名称 |
    | 默认值 | `NULL` |

    包含受信任 SSL 证书颁发机构（CA）证书文件的 PEM 格式的目录路径名。

    截至 MySQL 8.0.16 版本，此变量是动态的，可以在运行时进行修改，以影响服务器在执行`ALTER INSTANCE RELOAD TLS`后建立的新连接所使用的 TLS 上下文，或者在变量值被持久化后重新启动。请参见用于加密连接的服务器端运行时配置和监控。在 MySQL 8.0.16 之前，此变量只能在服务器启动时设置。

+   `ssl_cert`

    | 命令行格式 | `--ssl-cert=file_name` |
    | --- | --- |
    | 系统变量 | `ssl_cert` |
    | 作用域 | 全局 |
    | 动态（≥ 8.0.16） | 是 |
    | 动态（≤ 8.0.15） | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 文件名 |
    | 默认值 | `NULL` |

    服务器 SSL 公钥证书文件的 PEM 格式的路径名。

    如果服务器启动时使用`ssl_cert`设置为使用任何受限制的密码或密码类别的证书，服务器将以禁用加密连接的方式启动。有关密码限制的信息，请参见连接密码配置。

    从 MySQL 8.0.16 开始，此变量是动态的，可以在运行时修改，以影响服务器在执行`ALTER INSTANCE RELOAD TLS`后或重新启动后用于新连接的 TLS 上下文，如果变量值已持久化。请参阅服务器端运行时配置和监控加密连接。在 MySQL 8.0.16 之前，此变量只能在服务器启动时设置。

    注意

    链式 SSL 证书支持在 v8.0.30 中添加；以前只读取第一个证书。

+   `ssl_cipher`

    | 命令行格式 | `--ssl-cipher=name` |
    | --- | --- |
    | 系统变量 | `ssl_cipher` |
    | 范围 | 全局 |
    | 动态（≥ 8.0.16） | 是 |
    | 动态（≤ 8.0.15） | 否 |
    | `SET_VAR` Hint Applies | 否 |
    | 类型 | 字符串 |
    | 默认值 | `NULL` |

    允许的加密密码列表，用于使用 TLS 协议直至 TLSv1.2 的连接。如果列表中没有支持的密码，则使用这些 TLS 协议的加密连接将无法工作。

    为了最大的可移植性，密码列表应该是一个或多个密码名称的列表，用冒号分隔。以下示例显示了两个密码名称，用冒号分隔：

    ```sql
    [mysqld]
    ssl_cipher="DHE-RSA-AES128-GCM-SHA256:AES128-SHA"
    ```

    OpenSSL 支持在 OpenSSL 文档中描述的指定密码的语法，网址为[`www.openssl.org/docs/manmaster/man1/ciphers.html`](https://www.openssl.org/docs/manmaster/man1/ciphers.html)。

    有关 MySQL 支持的加密密码的信息，请参阅第 8.3.2 节，“加密连接 TLS 协议和密码”。

    从 MySQL 8.0.16 开始，此变量是动态的，可以在运行时修改，以影响服务器在执行`ALTER INSTANCE RELOAD TLS`后或重新启动后用于新连接的 TLS 上下文，如果变量值已持久化。请参阅服务器端运行时配置和监控加密连接。在 MySQL 8.0.16 之前，此变量只能在服务器启动时设置。

+   `ssl_crl`

    | 命令行格式 | `--ssl-crl=file_name` |
    | --- | --- |
    | 系统变量 | `ssl_crl` |
    | 范围 | 全局 |
    | 动态（≥ 8.0.16） | 是 |
    | 动态（≤ 8.0.15） | 否 |
    | `SET_VAR` Hint Applies | 否 |
    | 类型 | 文件名称 |
    | 默认值 | `NULL` |

    包含以 PEM 格式的证书吊销列表的文件路径名。

    截至 MySQL 8.0.16 版本，此变量是动态的，可以在运行时修改，以影响服务器在执行`ALTER INSTANCE RELOAD TLS`后或重启后用于新连接的 TLS 上下文。参见服务器端运行时配置和加密连接监控。在 MySQL 8.0.16 之前，此变量只能在服务器启动时设置。

+   `ssl_crlpath` 

    | 命令行格式 | `--ssl-crlpath=dir_name` |
    | --- | --- |
    | 系统变量 | `ssl_crlpath` |
    | 作用域 | 全局 |
    | 动态（≥ 8.0.16） | 是 |
    | 动态（≤ 8.0.15） | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 目录名称 |
    | 默认值 | `NULL` |

    包含以 PEM 格式的证书吊销列表文件的目录路径。

    截至 MySQL 8.0.16 版本，此变量是动态的，可以在运行时修改，以影响服务器在执行`ALTER INSTANCE RELOAD TLS`后或重启后用于新连接的 TLS 上下文。参见服务器端运行时配置和加密连接监控。在 MySQL 8.0.16 之前，此变量只能在服务器启动时设置。

+   `ssl_fips_mode`

    | 命令行格式 | `--ssl-fips-mode={OFF&#124;ON&#124;STRICT}` |
    | --- | --- |
    | 已弃用 | 8.0.34 |
    | 系统变量 | `ssl_fips_mode` |
    | 作用域 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 枚举 |
    | 默认值 | `OFF` |
    | 有效值 | `OFF`（或 0）`ON`（或 1）`STRICT`（或 2） |

    控制是否在服务器端启用 FIPS 模式。`ssl_fips_mode` 系统变量与其他 `ssl_*`xxx`*` 系统变量不同，它不用于控制服务器是否允许加密连接，而是影响允许进行的加密操作。参见第 8.8 节，“FIPS 支持”。

    允许使用以下 `ssl_fips_mode` 值：

    +   `OFF`（或 0）：禁用 FIPS 模式。

    +   `ON`（或 1）：启用 FIPS 模式。

    +   `STRICT`（或 2）：启用“严格”FIPS 模式。

    注意

    如果 OpenSSL FIPS 对象模块不可用，则`ssl_fips_mode`的唯一允许值是`OFF`。在这种情况下，在启动时将`ssl_fips_mode`设置为`ON`或`STRICT`会导致服务器生成错误消息并退出。

    截至 MySQL 8.0.34 版本，此选项已被弃用并设置为只读。预计在未来的 MySQL 版本中将其移除。

+   `ssl_key`

    | 命令行格式 | `--ssl-key=file_name` |
    | --- | --- |
    | 系统变量 | `ssl_key` |
    | 范围 | 全局 |
    | 动态（≥ 8.0.16） | 是 |
    | 动态（≤ 8.0.15） | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 文件名 |
    | 默认值 | `NULL` |

    服务器 SSL 私钥文件的路径名，采用 PEM 格式。为了更好的安全性，请使用至少 2048 位的 RSA 密钥大小的证书。

    如果密钥文件受密码保护，服务器会提示用户输入密码。密码必须以交互方式提供；不能存储在文件中。如果密码不正确，程序会继续执行，就好像无法读取密钥一样。

    截至 MySQL 8.0.16 版本，此变量是动态的，可以在运行时修改，以影响服务器在执行`ALTER INSTANCE RELOAD TLS`后或重启后用于新连接的 TLS 上下文。参见服务器端运行时配置和加密连接监控。在 MySQL 8.0.16 之前，此变量只能在服务器启动时设置。

+   `ssl_session_cache_mode`

    | 命令行格式 | `--ssl_session_cache_mode={ON&#124;OFF}` |
    | --- | --- |
    | 引入 | 8.0.29 |
    | 系统变量 | `ssl_session_cache_mode` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `ON` |
    | 有效值 | `ON``OFF` |

    控制是否在服务器端启用内存中的会话缓存以及服务器端会话票据生成。默认模式为`ON`（启用会话缓存模式）。对`ssl_session_cache_mode`系统���量的更改只有在执行`ALTER INSTANCE RELOAD TLS`语句后生效，或者在变量值被持久化后重新启动后才会生效。

    这些`ssl_session_cache_mode`值是允许的：

    +   `ON`: 启用会话缓存模式。

    +   `OFF`: 禁用会话缓存模式。

    如果此系统变量的值为`OFF`，服务器将不会宣传其支持会话恢复。在运行 OpenSSL 1.0.`x`时，会话票据始终会生成，但当`ssl_session_cache_mode`启���时，这些票据是不可用的。

    可以通过`Ssl_session_cache_mode`状态变量观察当前生效的`ssl_session_cache_mode`的值。

+   `ssl_session_cache_timeout`

    | 命令行格式 | `--ssl_session_cache_timeout` |
    | --- | --- |
    | 引入版本 | 8.0.29 |
    | 系统变量 | `ssl_session_cache_timeout` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `300` |
    | 最小值 | `0` |
    | 最大值 | `84600` |
    | 单位 | 秒 |

    设置一个时间段，在建立新的加密连接到服务器时允许重用先前会话，前提是`ssl_session_cache_mode`系统变量已启用并且先前的会话数据可用。如果会话超时到期，则会话将无法再次重用。

    默认值为 300 秒，最大值为 84600（或一天的秒数）。对`ssl_session_cache_timeout`系统变量的更改仅在执行`ALTER INSTANCE RELOAD TLS`语句后生效，或者如果变量值已持久化，则在重新启动后生效。可以通过`Ssl_session_cache_timeout`状态变量观察当前生效的`ssl_session_cache_timeout`的值。

+   `stored_program_cache`

    | 命令行格式 | `--stored-program-cache=#` |
    | --- | --- |
    | 系统变量 | `stored_program_cache` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `256` |
    | 最小值 | `16` |
    | 最大值 | `524288` |

    为每个连接设置了缓存的存储过程的数量的软上限。此变量的值是指 MySQL 服务器为存储过程和存储函数分别维护的两个缓存中每个缓存中保存的存储过程数量。

    每当执行存储过程时，都会在解析存储过程的第一个或顶层语句之前检查此缓存大小；如果同一类型的存储过程（根据执行的是存储过程还是存储函数）的数量超过此变量指定的限制，相应的缓存将被清空，并释放先前为缓存对象分配的内存。这样即使存在存储过程之间的依赖关系，也可以安全地清空缓存。

    存储过程和存储函数缓存与存储程序定义缓存分区并行存在于字典对象缓存中。存储过程和存储函数缓存是每个连接的，而存储程序定义缓存是共享的。存储过程和存储函数缓存中对象的存在与存储程序定义缓存中对象的存在没有依赖关系，反之亦然。有关更多信息，请参见第 16.4 节，“字典对象缓存”。

+   `stored_program_definition_cache`

    | 命令行格式 | `--stored-program-definition-cache=#` |
    | --- | --- |
    | 系统变量 | `stored_program_definition_cache` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `256` |
    | 最小值 | `256` |
    | 最大值 | `524288` |

    定义了可以保留在字典对象缓存中的已使用和未使用的存储程序定义对象的数量限制。

    当使用中的数量小于`stored_program_definition_cache`定义的容量时，未使用的存储程序定义对象仅保留在字典对象缓存中。

    设置为 0 意味着只有在使用时才会将存储程序定义对象保留在字典对象缓存中。

    存储程序定义缓存分区与使用`stored_program_cache`选项配置的存储过程和存储函数缓存并行存在。

    `stored_program_cache`选项设置了每个连接缓存的存储过程或函数的软上限，并且每次连接执行存储过程或函数时都会检查该限制。另一方面，存储程序定义缓存分区是一个共享缓存，用于存储其他目的的存储程序定义对象。存储程序定义缓存分区中的对象的存在与存储过程缓存或存储函数缓存中的对象的存在没有依赖关系，反之亦然。

    有关相关信息，请参阅第 16.4 节，“字典对象缓存”。

+   `super_read_only`

    | 命令行格式 | `--super-read-only[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `super_read_only` |
    | 作用域 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    如果启用了`read_only`系统变量，则服务器不允许客户端更新，除非是具有`CONNECTION_ADMIN`权限（或已弃用的`SUPER`权限）的用户。如果还启用了`super_read_only`系统变量，则服务器甚至禁止具有`CONNECTION_ADMIN`或`SUPER`权限的用户进行客户端更新。有关只读模式的描述以及`read_only`和`super_read_only`之间的交互信息，请参阅`read_only`系统变量的描述。

    当启用`super_read_only`时，阻止客户端更新的操作包括不一定看起来是更新的操作，例如`CREATE FUNCTION`（安装可加载函数）、`INSTALL PLUGIN`和`INSTALL COMPONENT`。这些操作被禁止，因为它们涉及对`mysql`系统模式中的表的更改。

    同样，如果启用了事件调度器，启用`super_read_only`系统变量会阻止其在`events`数据字典表中更新事件“上次执行”时间戳。这会导致事件调度器在下次尝试执行计划事件时停止，并在写入服务器错误日志后停止。（在这种情况下，`event_scheduler`系统变量不会从`ON`更改为`OFF`。一个含义是，此变量拒绝了 DBA *意图*，即事件调度器启用或禁用，其实际状态为启动或停止可能是不同的。）如果在启用后随后禁用`super_read_only`，服务器会根据需要自动重新启动事件调度器，截至 MySQL 8.0.26。在 MySQL 8.0.26 之前，需要通过重新启用来手动重新启动事件调度器。

    在复制源服务器上对`super_read_only`的更改不会被复制到副本服务器。可以在副本上独立��源设置该值。

+   `syseventlog.facility`

    | 命令行格式 | `--syseventlog.facility=value` |
    | --- | --- |
    | 引入 | 8.0.13 |
    | 系统变量 | `syseventlog.facility` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 字符串 |
    | 默认值 | `daemon` |

    写入`syslog`的错误日志输出的设施（发送消息的程序类型）。除非安装了`log_sink_syseventlog`错误日志组件，否则此变量不可用。参见第 7.4.2.8 节，“将错误日志记录到系统日志”。

    允许的值可能因操作系统而异；请参考您的系统`syslog`文档。

    此变量在 Windows 上不存在。

+   `syseventlog.include_pid`

    | 命令行格式 | `--syseventlog.include-pid[={OFF&#124;ON}]` |
    | --- | --- |
    | 引入 | 8.0.13 |
    | 系统变量 | `syseventlog.include_pid` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔 |
    | 默认值 | `ON` |

    是否在写入`syslog`的错误日志输出的每一行中包含服务器进程 ID。除非安装了`log_sink_syseventlog`错误日志组件，否则此变量不可用。参见第 7.4.2.8 节，“将错误日志记录到系统日志”。

    此变量在 Windows 上不存在。

+   `syseventlog.tag`

    | 命令行格式 | `--syseventlog.tag=tag` |
    | --- | --- |
    | 引入 | 8.0.13 |
    | 系统变量 | `syseventlog.tag` |
    | 范围 | 全局 |
    | Dynamic | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 字符串 |
    | Default Value | `空字符串` |

    要添加到写入`syslog`或 Windows 事件日志的错误日志输出中的服务器标识符的标签。除非安装了`log_sink_syseventlog`错误日志组件，否则无法使用此变量。请参阅 Section 7.4.2.8, “Error Logging to the System Log”。

    默认情况下，未设置任何标签，因此在 Windows 上服务器标识符仅为`MySQL`，在其他平台上为`mysqld`。如果指定了*`tag`*的标签值，则将其附加到具有前导连字符的服务器标识符中，从而得到`syslog`标识符为`mysqld-*`tag`*`（或在 Windows 上为`MySQL-*`tag`*`）。

    在 Windows 上，要使用尚不存在的标签，服务器必须从具有管理员权限的帐户运行，以允许为标签创建注册表条目。如果标签已经存在，则不需要提升权限。

+   `system_time_zone`

    | 系统变量 | `system_time_zone` |
    | --- | --- |
    | 范围 | 全局 |
    | Dynamic | No |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 字符串 |

    服务器系统时区。当服务器开始执行时，它会从机器默认值继承一个时区设置，可能会被用于运行服务器或启动脚本的帐户的环境修改。该值用于设置`system_time_zone`。要明确指定系统时区，请设置`TZ`环境变量或使用**mysqld_safe**脚本的`--timezone`选项。

    截至 MySQL 8.0.26 版本，除了启动时初始化外，如果服务器主机的时区发生变化（例如，由于夏令时），`system_time_zone`会反映这种变化，这对应用程序有以下影响：

    +   引用`system_time_zone`的查询在夏令时更改前后会得到不同的值。

    +   对于在夏令时更改之前开始执行并在更改后结束的查询，`system_time_zone`在查询中保持恒定，因为该值通常在执行开始时被缓存。

    `system_time_zone`变量与`time_zone`变量不同。尽管它们可能具有相同的值，但后者变量用于初始化每个连接的客户端的时区。参见第 7.1.15 节，“MySQL 服务器时区支持”。

+   `table_definition_cache`

    | 命令行格式 | `--table-definition-cache=#` |
    | --- | --- |
    | 系统变量 | `table_definition_cache` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `-1`（表示自动调整大小；不要分配此文字值） |
    | 最小值 | `400` |
    | 最大值 | `524288` |

    可以存储在表定义缓存中的表定义数量。如果使用大量表格，可以创建一个大的表定义缓存以加快表格打开速度。表定义缓存占用空间较小，不使用文件描述符，不像普通表缓存。最小值为 400。默认值基于以下公式，上限为 2000：

    ```sql
    MIN(400 + table_open_cache / 2, 2000)
    ```

    对于`InnoDB`，`table_definition_cache` 设置充当字典对象缓存中表实例数量和一次可以打开的文件-每个表表空间数量的软限制。

    如果字典对象缓存中的表实例数量超过`table_definition_cache`限制，LRU 机制开始标记表实例以驱逐并最终从字典对象缓存中删除它们。由于具有外键关系的表实例不会放置在 LRU 列表上，具有缓存元数据的打开表数量可能高于`table_definition_cache`限制。

    一次可以打开的基于文件的表空间数量受`table_definition_cache`和`innodb_open_files`设置的限制。如果两个变量都设置了，将使用较高的设置。如果两个变量都没有设置，则使用具有更高默认值的`table_definition_cache`设置。如果打开的表空间数量超过由`table_definition_cache`或`innodb_open_files`定义的限制，LRU 机制将在 LRU 列表中搜索完全刷新且当前未扩展的表空间文件。每次打开新表空间时执行此过程。只有非活动表空间会被关闭。

    表定义缓存与字典对象缓存的表定义缓存分区并行存在。这两个缓存存储表定义，但为 MySQL 服务器的不同部分提供服务。一个缓存中的对象不依赖于另一个缓存中对象的存在。有关更多信息，请参见第 16.4 节，“字典对象缓存”。

+   `table_encryption_privilege_check`

    | 命令行格式 | `--table-encryption-privilege-check[={OFF&#124;ON}]` |
    | --- | --- |
    | 引入版本 | 8.0.16 |
    | 系统变量 | `table_encryption_privilege_check` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    控制在创建或更改具有与`default_table_encryption`设置不同的加密的模式或通用表空间时发生的`TABLE_ENCRYPTION_ADMIN`权限检查，或者在创建或更改具有与默认模式加密不同的加密设置的表时发生的检查。默认情况下，此检查已禁用。

    在运行时设置`table_encryption_privilege_check`需要`SUPER`权限。

    `table_encryption_privilege_check`支持`SET PERSIST`和`SET PERSIST_ONLY`语法。请参见第 7.1.9.3 节，“持久化系统变量”。

    更多信息，请参见为模式和通用表空间定义加密默认值。

+   `table_open_cache`

    | 命令行格式 | `--table-open-cache=#` |
    | --- | --- |
    | 系统变量 | `table_open_cache` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `4000` |
    | 最小值 | `1` |
    | 最大值 | `524288` |

    所有线程的打开表数。增加此值会增加**mysqld**所需的文件描述符数。此变量的有效值为`open_files_limit`的有效值 `- 10 -` 与`max_connections`的有效值 `/ 2` 中的较大者，以及 400；即

    ```sql
    MAX(
        (open_files_limit - 10 - max_connections) / 2,
        400
       )
    ```

    您可以通过检查`Opened_tables`状态变量来检查是否需要增加表缓存。如果`Opened_tables`的值很大，并且您不经常使用`FLUSH TABLES`（这只是强制关闭并重新打开所有表），那么您应该增加`table_open_cache`变量的值。有关表缓存的更多信息，请参见第 10.4.3.1 节，“MySQL 如何打开和关闭表”。

+   `table_open_cache_instances`

    | 命令行格式 | `--table-open-cache-instances=#` |
    | --- | --- |
    | 系统变量 | `table_open_cache_instances` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `16` |
    | 最小值 | `1` |
    | 最大值 | `64` |

    打开表缓存实例的数量。为了通过将打开表缓存分割为大小为`table_open_cache` / `table_open_cache_instances`的几个较小的缓存实例来减少会话之间的争用以提高可伸缩性。会话只需要锁定一个实例来访问它以进行 DML 语句。这将在实例之间分段缓存访问，允许在许多会话访问表时使用缓存时获得更高的性能。（DDL 语句仍然需要锁定整个缓存，但这类语句比 DML 语句频率低得多。）

    在通常使用 16 个或更多核心的系统上，建议值为 8 或 16。但是，如果您的表上有许多大型触发器导致内存负载较高，则默认设置的`table_open_cache_instances`可能会导致过多的内存使用。在这种情况下，将`table_open_cache_instances`设置为 1 可能有助于限制内存使用。

+   `tablespace_definition_cache`

    | 命令行格式 | `--tablespace-definition-cache=#` |
    | --- | --- |
    | 系统变量 | `tablespace_definition_cache` |
    | 作用域 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `256` |
    | 最小值 | `256` |
    | 最大值 | `524288` |

    定义了可以保留在字典对象缓存中的表空间定义对象数量的限制，包括已使用和未使用的对象。

    当未使用的表空间定义对象数量少于`tablespace_definition_cache`定义的容量时，这些对象仅保留在字典对象缓存中。

    当设置为`0`时，表空间定义对象仅在使用时保留在字典对象缓存中。

    更多信息，请参见第 16.4 节，“字典对象缓存”。

+   `temptable_max_mmap`

    | 命令行格式 | `--temptable-max-mmap=#` |
    | --- | --- |
    | 引入版本 | 8.0.23 |
    | 系统变量 | `temptable_max_mmap` |
    | 作用域 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `1073741824` |
    | 最小值 | `0` |
    | 最大值 | `2⁶⁴-1` |
    | 单位 | 字节 |

    定义了在将数据存储到`InnoDB`内部临时表之前，TempTable 存储引擎允许从内存映射临时文件中分配的最大内存量（以字节为单位）。设置为 0 会禁用从内存映射临时文件中分配内存。更多信息，请参见 Section 10.4.4, “MySQL 中的内部临时表使用”。

+   `temptable_max_ram`

    | 命令行格式 | `--temptable-max-ram=#` |
    | --- | --- |
    | 系统变量 | `temptable_max_ram` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `1073741824` |
    | 最小值 | `2097152` |
    | 最大值 | `2⁶⁴-1` |
    | 单位 | 字节 |

    定义了在 TempTable 存储引擎开始将数据存储到磁盘之前，可以占用的最大内存量。默认值为 1073741824 字节（1GiB）。更多信息，请参见 Section 10.4.4, “MySQL 中的内部临时表使用”。

+   `temptable_use_mmap`

    | 命令行格式 | `--temptable-use-mmap[={OFF&#124;ON}]` |
    | --- | --- |
    | 引入版本 | 8.0.16 |
    | 已弃用 | 8.0.26 |
    | 系统变量 | `temptable_use_mmap` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `ON` |

    定义了当 TempTable 存储引擎占用的内存量超过`temptable_max_ram`变量定义的限制时，TempTable 存储引擎是否将为内部内存中的临时表分配空间作为内存映射临时文件。当禁用`temptable_use_mmap`时，TempTable 存储引擎会改用`InnoDB`磁盘上的内部临时表。更多信息，请参见 Section 10.4.4, “MySQL 中的内部临时表使用”。

+   `thread_cache_size`

    | 命令行格式 | `--thread-cache-size=#` |
    | --- | --- |
    | 系统变量 | `thread_cache_size` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `-1`（表示自动调整大小；不要分配此字面值） |
    | 最小值 | `0` |
    | 最大值 | `16384` |

    服务器应该缓存多少线程以便重复使用。当客户端断开连接时，如果`thread_cache_size`线程少于缓存中，客户端的线程将放入缓存中。如果可能，请求线程将通过重用从缓存中取出的线程来满足，并且只有在缓存为空时才会创建新线程。如果您有很多新连接，可以增加此变量以提高性能。通常情况下，如果您有一个良好的线程实现，这并不会提供显著的性能改进。但是，如果您的服务器每秒看到数百个连接，通常应将`thread_cache_size`设置得足够高，以便大多数新连接使用缓存的线程。通过检查`Connections`和`Threads_created`状态变量之间的差异，您可以看到线程缓存的效率如何。有关详细信息，请参见第 7.1.10 节，“服务器状态变量”。

    默认值基于以下公式，上限为 100：

    ```sql
    8 + (max_connections / 100)
    ```

+   `thread_handling`

    | 命令行格式 | `--thread-handling=name` |
    | --- | --- |
    | 系统变量 | `线程处理` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 枚举 |
    | 默认值 | `one-thread-per-connection` |
    | 有效值 | `no-threads``one-thread-per-connection``loaded-dynamically` |

    服务器用于连接线程的线程处理模型。允许的值为`no-threads`（服务器使用单个线程处理一个连接），`one-thread-per-connection`（服务器使用一个线程处理每个客户端连接）和`loaded-dynamically`（线程池插件初始化时设置）。`no-threads`在 Linux 下进行调试很有用；请参见第 7.9 节，“调试 MySQL”。

+   `thread_pool_algorithm`

    | 命令行格式 | `--thread-pool-algorithm=#` |
    | --- | --- |
    | 系统变量 | `thread_pool_algorithm` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `0` |
    | 最小值 | `0` |
    | 最大值 | `1` |

    此变量控制线程池插件使用的算法：

    +   `0`：使用保守的低并发��法。

    +   `1`: 使用一种高并发算法，当线程计数达到最佳值时性能更好，但如果连接数达到极高值，性能可能会下降。

    仅当启用线程池插件时才可用此变量。请参见第 7.6.3 节，“MySQL 企业线程池”。

+   `thread_pool_dedicated_listeners`

    | 命令行格式 | `--thread-pool-dedicated-listeners` |
    | --- | --- |
    | 引入 | 8.0.23 |
    | 系统变量 | `thread_pool_dedicated_listeners` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    将每个线程组中的一个监听线程专用于监听分配给该组的连接传入的语句。

    +   `OFF`:（默认）禁用专用监听线程。

    +   `ON`: 将每个线程组中的一个监听线程专用于监听分配给该组的连接传入的语句。专用监听线程不执行查询。

    仅在通过 `thread_pool_max_transactions_limit` 定义了事务限制时，启用 `thread_pool_dedicated_listeners` 才有用。否则，不应启用 `thread_pool_dedicated_listeners`。

    MySQL HeatWave 服务在 MySQL 8.0.23 中引入了此变量。从 MySQL 8.0.31 起，它可在 MySQL 企业版中使用。

+   `thread_pool_high_priority_connection`

    | 命令行格式 | `--thread-pool-high-priority-connection=#` |
    | --- | --- |
    | 系统变量 | `thread_pool_high_priority_connection` |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `0` |
    | 最小值 | `0` |
    | 最大值 | `1` |

    此变量影响执行前新语句的排队。如果值为 0（假，即默认值），语句排队将同时使用低优先级队列和高优先级队列。如果值为 1（真），排队的语句始终进入高优先级队列。

    仅当启用线程池插件时才可用此变量。请参见第 7.6.3 节，“MySQL 企业线程池”。

+   `thread_pool_max_active_query_threads`

    | 命令行格式 | `--thread-pool-max-active-query-threads` |
    | --- | --- |
    | 引入 | 8.0.19 |
    | 系统变量 | `thread_pool_max_active_query_threads` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `0` |
    | 最小值 | `0` |
    | 最大值 | `512` |

    每个组允许的活动（运行中）查询线程的最大数量。如果值为 0，则线程池插件将使用尽可能多的可用线程。

    此变量仅在启用线程池插件时可用。请参阅 第 7.6.3 节，“MySQL 企业线程池”。

+   `thread_pool_max_transactions_limit`

    | 命令行格式 | `--thread-pool-max-transactions-limit` |
    | --- | --- |
    | 引入版本 | 8.0.23 |
    | 系统变量 | `thread_pool_max_transactions_limit` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `0` |
    | 最小值 | `0` |
    | 最大值 | `1000000` |

    线程池插件允许的最大事务数。定义事务限制将一个线程绑定到一个事务，直到提交，这有助于在高并发情况下稳定吞吐量。

    默认值为 0 表示没有事务限制。该变量是动态的，但无法在运行时从 0 更改为更高的值，反之亦然。在启动时设置为非零值允许在运行时进行动态配置。需要`CONNECTION_ADMIN`权限才能在运行时配置`thread_pool_max_transactions_limit`。

    当您定义事务限制时，启用`thread_pool_dedicated_listeners`会在每个线程组中创建一个专用的监听线程。额外的专用监听线程会消耗更多资源并影响线程池性能。因此，应谨慎使用`thread_pool_dedicated_listeners`。

    当达到由`thread_pool_max_transactions_limit`定义的限制时，新连接似乎会挂起，直到一个或多个现有事务完成。当尝试在现有连接上启动新事务时也会发生相同情况。如果现有连接被阻塞或运行时间较长，则可能需要特权连接才能访问服务器以增加限制、移除限制或终止运行中的事务。请参阅 特权连接。

    MySQL HeatWave 服务在 MySQL 8.0.23 中引入了该变量。它从 MySQL 8.0.31 开始与 MySQL 企业版一起提供。

+   `thread_pool_max_unused_threads`

    | 命令行格式 | `--thread-pool-max-unused-threads=#` |
    | --- | --- |
    | 系统变量 | `thread_pool_max_unused_threads` |
    | 作用域 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `0` |
    | 最小值 | `0` |
    | 最大值 | `4096` |

    线程池中允许的未使用线程的最大数量。该变量使得可以限制睡眠线程使用的内存量。

    值为 0（默认值）表示睡眠线程数量没有限制。值为*`N`*，其中*`N`*大于 0 表示 1 个消费者线程和*`N`*−1 个保留线程。在这种情况下，如果一个线程准备进入睡眠状态，但睡眠线程数量已达到最大值，则该线程将退出而不是进入睡眠状态。

    一个睡眠线程可以是消费者线程或保留线程。线程池允许一个线程在睡眠时成为消费者线程。如果一个线程进入睡眠状态且没有现有的消费者线程，则它将作为消费者线程睡眠。当需要唤醒一个线程时，如果存在消费者线程，则选择消费者线程。只有在没有消费者线程需要唤醒时才选择保留线程。

    该变量仅在启用线程池插件时可用。参见第 7.6.3 节，“MySQL 企业线程池”。

+   `thread_pool_prio_kickup_timer`

    | 命令行格式 | `--thread-pool-prio-kickup-timer=#` |
    | --- | --- |
    | 系统变量 | `thread_pool_prio_kickup_timer` |
    | 作用域 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `1000` |
    | 最小值 | `0` |
    | 最大值 | `4294967294` |
    | 单位 | 毫秒 |

    该变量影响等待在低优先级队列中执行的语句。该值是等待语句移动到高优先级队列之前的毫秒数。默认值为 1000（1 秒）。

    该变量仅在启用线程池插件时可用。参见第 7.6.3 节，“MySQL 企业线程池”。

+   `thread_pool_query_threads_per_group`

    | 命令行格式 | `--thread-pool-query-threads-per-group` |
    | --- | --- |
    | 引入版本 | 8.0.31 |
    | 系统变量 | `thread_pool_query_threads_per_group` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `1` |
    | 最小值 | `1` |
    | 最大值 | `4096` |

    线程组中允许的最大查询线程数。最大值为 4096，但如果设置了 `thread_pool_max_transactions_limit`，则 `thread_pool_query_threads_per_group` 不得超过该值。

    默认值为 1 意味着每个线程组中有一个活动查询线程，在许多负载情况下运行良好。当您使用高并发线程池算法（`thread_pool_algorithm = 1`）时，如果由于长时间运行的事务导致响应时间变慢，请考虑增加该值。

    在运行时需要 `CONNECTION_ADMIN` 权限来配置 `thread_pool_query_threads_per_group`。

    如果在运行时减少 `thread_pool_query_threads_per_group` 的值，则当前正在运行用户查询的线程将被允许完成，然后移至保留池或终止。如果在运行时增加值，并且线程组需要更多线程，则如果可能，这些线程将从保留池中获取，否则将创建新线程。

    从 MySQL 8.0.31 版本开始，在 MySQL HeatWave Service 和 MySQL Enterprise Edition 中可用此变量。

+   `thread_pool_size`

    | 命令行格式 | `--thread-pool-size=#` |
    | --- | --- |
    | 系统变量 | `thread_pool_size` |
    | 作用范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `16` |
    | 最小值 | `1` |
    | 最大值（≥ 8.0.19） | `512` |
    | 最大值（≤ 8.0.18） | `64` |

    线程池中线程组的数量。这是控制线程池性能最重要的参数。它影响可以同时执行多少条语句。如果指定了超出允许值范围的值，则线程池插件不会加载，并且服务器会向错误日志写入消息。

    仅当线程池插件启用时才可用此变量。请参阅 Section 7.6.3, “MySQL Enterprise Thread Pool”。

+   `thread_pool_stall_limit`

    | 命令行格式 | `--thread-pool-stall-limit=#` |
    | --- | --- |
    | 系统变量 | `thread_pool_stall_limit` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `6` |
    | 最小值 | `4` |
    | 最大值 | `600` |
    | 单位 | 毫秒 * 10 |

    此变量影响执行语句。该值是语句在开始执行后完成之前必须完成的时间，此时线程池允许线程组开始执行另一个语句。该值以 10 毫秒单位计量，因此默认值 6 表示 60 毫秒。较短的等待值允许线程更快地启动。较短的值也更有利于避免死锁情况。较长的等待值对包含长时间运行语句的工作负载有用，以避免在当前语句执行时启动太多新语句。

    仅当线程池插件启用时才可用此变量。请参阅第 7.6.3 节，“MySQL 企业线程池”。

+   `thread_pool_transaction_delay`

    | 命令行格式 | `--thread-pool-transaction-delay` |
    | --- | --- |
    | 引入 | 8.0.31 |
    | 系统变量 | `thread_pool_transaction_delay` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `0` |
    | 最小值 | `0` |
    | 最大值 | `300000` |

    执行新事务之前的延迟时间，单位为毫秒。最大值为 300000（5 分钟）。

    在并行事务影响其他操作性能的情况下，可以使用事务延迟。例如，如果并行事务影响索引创建或在线缓冲池调整操作，您可以配置事务延迟以减少资源争用，同时运行这些操作。

    工作线程在执行新事务之前会休眠指定的`thread_pool_transaction_delay`毫秒数。

    `thread_pool_transaction_delay`设置不影响从特权连接（分配给`Admin`线程组的连接）发出的查询。这些查询不受配置的事务延迟的影响。

+   `thread_stack`

    | 命令行格式 | `--thread-stack=#` |
    | --- | --- |
    | 系统变量 | `thread_stack` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值（64 位平台，≥ 8.0.27） | `1048576` |
    | 默认值（64 位平台，≤ 8.0.26�� | `286720` |
    | 默认值（32 位平台，≥ 8.0.27） | `1048576` |
    | 默认值（32 位平台，≤ 8.0.26） | `221184` |
    | 最小值 | `131072` |
    | 最大值（64 位平台） | `18446744073709550592` |
    | 最大值（32 位平台） | `4294966272` |
    | 单位 | 字节 |
    | 块大小 | `1024` |

    每个线程的堆栈大小。默认值足够大以进行正常操作。如果线程堆栈大小太小，则会限制服务器可以处理的 SQL 语句的复杂性，存储过程的递归深度以及其他消耗内存的操作。

+   `time_zone`

    | 系统变量 | `time_zone` |
    | --- | --- |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用（≥ 8.0.17） | 是 |
    | `SET_VAR` 提示适用（≤ 8.0.16） | 否 |
    | 类型 | 字符串 |
    | 默认值 | `SYSTEM` |
    | 最小值（≥ 8.0.19） | `-13:59` |
    | 最小值（≤ 8.0.18） | `-12:59` |
    | 最大值（≥ 8.0.19） | `+14:00` |
    | 最大值（≤ 8.0.18） | `+13:00` |

    当前时区。此变量用于为每个连接的客户端初始化时区。默认情况下，此的初始值为 `'SYSTEM'`（表示“使用`system_time_zone`的值”）。可以在服务器启动时使用 `--default-time-zone` 选项显式指定值。请参阅 Section 7.1.15, “MySQL Server Time Zone Support”。

    注意

    如果设置为 `SYSTEM`，则每次需要时区计算的 MySQL 函数调用都会进行系统库调用以确定当前系统时区。此调用可能受全局互斥锁保护，导致争用。

+   `timestamp`

    | 系统变量 | `timestamp` |
    | --- | --- |
    | 范围 | 会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 是 |
    | 类型 | 数值 |
    | 默认值 | `UNIX_TIMESTAMP()` |
    | 最小值 | `1` |
    | 最大值 | `2147483647` |

    设置此客户端的时间。如果您使用二进制日志来恢复行，则会使用原始时间戳。*`timestamp_value`* 应为 Unix 纪元时间戳（类似于`UNIX_TIMESTAMP()`返回的值，而不是`'*`YYYY-MM-DD hh:mm:ss`*'`格式的值）或 `DEFAULT`。

    将`timestamp`设置为一个常量值会使其保留该值，直到再次更改。将`timestamp`设置为`DEFAULT`会使其值为在访问时的当前日期和时间。

    `timestamp`是一个`DOUBLE`而不是`BIGINT`，因为其值包括微秒部分。最大值对应于 UTC 时区的`'2038-01-19 03:14:07'`，与`TIMESTAMP`数据类型相同。

    `SET timestamp`会影响`NOW()`返回的值，但不会影响`SYSDATE()`。这意味着二进制日志中的时间戳设置不会影响对`SYSDATE()`的调用。可以使用`--sysdate-is-now`选项启动服务器，使`SYSDATE()`成为`NOW()`的同义词，这样`SET timestamp`会影响这两个函数。

+   `tls_ciphersuites`

    | 命令行格式 | `--tls-ciphersuites=ciphersuite_list` |
    | --- | --- |
    | 引入版本 | 8.0.16 |
    | 系统变量 | `tls_ciphersuites` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` Hint Applies | No |
    | 类型 | 字符串 |
    | 默认值 | `NULL` |

    服务器允许用于使用 TLSv1.3 的加密连接的密码套件。该值是一个由零个或多个以冒号分隔的密码套件名称组成的列表。

    可以为此变量命名的密码套件取决于用于编译 MySQL 的 SSL 库。如果未设置此变量，则其默认值为`NULL`，这意味着服务器允许默认密码套件。如果将变量设置为空字符串，则不启用任何密码套件，无法建立加密连接。有关更多信息，请参见第 8.3.2 节，“加密连接 TLS 协议和密码套件”。

+   `tls_version`

    | 命令行格式 | `--tls-version=protocol_list` |
    | --- | --- |
    | 系统变量 | `tls_version` |
    | 作用范围 | 全局 |
    | 动态（≥ 8.0.16） | 是 |
    | 动态（≤ 8.0.15） | 否 |
    | `SET_VAR` Hint Applies | No |
    | 类型 | 字符串 |
    | 默认值（≥ 8.0.28） | `TLSv1.2,TLSv1.3` |
    | 默认值（≥ 8.0.16, ≤ 8.0.27） | `TLSv1,TLSv1.1,TLSv1.2,TLSv1.3` |
    | 默认值（≤ 8.0.15） | `TLSv1,TLSv1.1,TLSv1.2` |

    服务器允许加密连接的协议。该值是一个或多个逗号分隔的协议名称列表，不区分大小写。可以为此变量命名的协议取决于用于编译 MySQL 的 SSL 库。应选择允许的协议，以避免在列表中留下“漏洞”。有关详细信息，请参见第 8.3.2 节，“加密连接 TLS 协议和密码”。

    自 MySQL 8.0.16 起，此变量是动态的，可以在运行时修改，以影响服务器用于新连接的 TLS 上下文。请参见用于加密连接的服务器端运行时配置和监控。在 MySQL 8.0.16 之前，此变量只能在服务器启动时设置。

    重要

    +   自 MySQL 8.0.28 起，MySQL 服务器已移除对 TLSv1 和 TLSv1.1 连接协议的支持。这些协议自 MySQL 8.0.26 起已被弃用。详细信息请参见移除对 TLSv1 和 TLSv1.1 协议的支持。

    +   自 MySQL 8.0.16 起，MySQL 服务器支持 TLSv1.3 协议，前提是 MySQL 服务器使用了 OpenSSL 1.1.1 或更高版本进行编译。服务器在启动时检查 OpenSSL 的版本，如果低于 1.1.1，则从系统变量的默认值中移除 TLSv1.3。在这种情况下，默认值为“`TLSv1,TLSv1.1,TLSv1.2`”，直到 MySQL 8.0.27，从 MySQL 8.0.28 开始为“`TLSv1.2`”。

+   `tmp_table_size`

    | 命令行格式 | `--tmp-table-size=#` |
    | --- | --- |
    | 系统变量 | `tmp_table_size` |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 是 |
    | 类型 | 整数 |
    | 默认值 | `16777216` |
    | 最小值 | `1024` |
    | 最大值 | `18446744073709551615` |
    | 单位 | 字节 |

    定义了由`MEMORY`存储引擎和自 MySQL 8.0.28 起的`TempTable`存储引擎创建的内部内存临时表的最大大小。如果内部内存临时表超过此大小，它将自动转换为磁盘上的内部临时表。

    `tmp_table_size`变量不适用于用户创建的`MEMORY`表。不支持用户创建的`TempTable`表。

    当在内存中使用`MEMORY`存储引擎用于临时表时，实际大小限制为`tmp_table_size`和`max_heap_table_size`中较小的一个。`max_heap_table_size`设置不适用于`TempTable`表。

    当您执行许多高级`GROUP BY`查询并且内存充足时，增加`tmp_table_size`的值（以及在使用`MEMORY`存储引擎用于内存中临时表时必要时增加`max_heap_table_size`的值）。

    您可以通过比较`Created_tmp_disk_tables`和`Created_tmp_tables`的值来比较创建的内部磁盘临时表的数量与创建的内部临时表的总数。

    参见 Section 10.4.4, “MySQL 中的内部临时表使用”。

+   `tmpdir`

    | 命令行格式 | `--tmpdir=dir_name` |
    | --- | --- |
    | 系统变量 | `tmpdir` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 目录名称 |

    用于创建临时文件的目录路径。如果默认的`/tmp`目录位于一个无法容纳临时表的分区上，这可能会有用。此变量可以设置为一系列路径，这些路径会循环使用。在 Unix 上应该用冒号字符（`:`）分隔路径，在 Windows 上应该用分号字符（`;`）分隔路径。

    `tmpdir`可以是一个非永久性位置，比如基于内存的文件系统上的目录或者在服务器主机重新启动时清除的目录。如果 MySQL 服务器充当副本，并且您使用非永久性位置作为`tmpdir`，考虑使用`replica_load_tmpdir`或`slave_load_tmpdir`变量为副本设置不同的临时目录。对于副本，用于复制`LOAD DATA`语句的临时文件存储在此目录中，因此在永久位置下它们可以在机器重新启动后继续存在，尽管如果临时文件已被删除，则复制现在可以在重新启动后继续。

    关于临时文件存储位置的更多信息，请参见 Section B.3.3.5，“MySQL 存储临时文件的位置”。

+   `transaction_alloc_block_size`

    | 命令行格式 | `--transaction-alloc-block-size=#` |
    | --- | --- |
    | 系统变量 | `transaction_alloc_block_size` |
    | 范围 | 全局、会话 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `8192` |
    | 最小值 | `1024` |
    | 最大值 | `131072` |
    | 单位 | 字节 |
    | 块大小 | `1024` |

    增加每个事务内存池所需内存的字节数。请参阅`transaction_prealloc_size`的描述。

+   `transaction_isolation`

    | 命令行格式 | `--transaction-isolation=name` |
    | --- | --- |
    | 系统变量 | `transaction_isolation` |
    | 范围 | 全局、会话 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 枚举 |
    | 默认值 | `REPEATABLE-READ` |
    | 有效值 | `READ-UNCOMMITTED``READ-COMMITTED``REPEATABLE-READ``SERIALIZABLE` |

    事务隔离级别。默认值为`REPEATABLE-READ`。

    事务隔离级别有三个范围：全局、会话和下一个事务。这种三范围实现导致了一些非标准的隔离级别分配语义，稍后会描述。

    要在启动时设置全局事务隔离级别，请使用`--transaction-isolation`服务器选项。

    在运行时，隔离级别可以直接使用`SET`语句设置`transaction_isolation`系统变量的值，或间接使用`SET TRANSACTION`语句。如果您直接将`transaction_isolation`设置为包含空格的隔离级别名称，则名称应该用引号括起来，并用破折号替换空格。例如，使用以下`SET`语句设置全局值：

    ```sql
    SET GLOBAL transaction_isolation = 'READ-COMMITTED';
    ```

    设置全局`transaction_isolation`值会为所有后续会话设置隔离级别。现有会话不受影响。

    要设置会话或下一个级别的`transaction_isolation`值，请使用`SET`语句。对于大多数会话系统变量，这些语句是设置值的等效方式：

    ```sql
    SET @@SESSION.*var_name* = *value*;
    SET SESSION *var_name* = *value*;
    SET *var_name* = *value*;
    SET @@*var_name* = *value*;
    ```

    如前所述，事务隔离级别具有下一个事务范围，除了全局和会话范围。为了启用设置下一个事务范围，用于分配会话系统变量值的`SET`语法对于`transaction_isolation`具有非标准语义：

    +   要设置会话隔离级别，请使用以下任何语法： 

        ```sql
        SET @@SESSION.transaction_isolation = *value*;
        SET SESSION transaction_isolation = *value*;
        SET transaction_isolation = *value*;
        ```

        对于这些语法，这些语义适用：

        +   为会话中执行的所有后续事务设置隔���级别。

        +   允许在事务内执行，但不会影响当前正在进行的事务。

        +   如果在事务之间执行，则会覆盖任何设置下一个事务隔离级别的先前语句。

        +   对应于`SET SESSION TRANSACTION ISOLATION LEVEL`（带有`SESSION`关键字）。

    +   要设置下一个事务隔离级别，请使用以下语法：

        ```sql
        SET @@transaction_isolation = *value*;
        ```

        对于该语法，这些语义适用：

        +   仅为会话中执行的下一个单个事务设置隔离级别。

        +   后续事务将恢复到会话隔离级别。

        +   不允许在事务内执行。

        +   对应于`SET TRANSACTION ISOLATION LEVEL`（不带`SESSION`关键字）。

    有关`SET TRANSACTION`及其与`transaction_isolation`系统变量的关系的更多信息，请参见 Section 15.3.7, “SET TRANSACTION Statement”。

+   `transaction_prealloc_size`

    | 命令行格式 | `--transaction-prealloc-size=#` |
    | --- | --- |
    | 已弃用 | 8.0.29 |
    | 系统变量 | `transaction_prealloc_size` |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `4096` |
    | 最小值 | `1024` |
    | 最大值 | `131072` |
    | 单位 | 字节 |
    | 块大小 | `1024` |

    有一个每个事务的内存池，用于进行各种与事务相关的分配。池的初始大小以字节为单位为`transaction_prealloc_size`。对于每个无法从池中满足的分配，因为它没有足够的可用内存，池会增加`transaction_alloc_block_size`字节。当事务结束时，池会被截断为`transaction_prealloc_size`字节。通过使`transaction_prealloc_size`足够大，以包含单个事务中的所有语句，您可以避免许多`malloc()`调用。

    从 MySQL 8.0.29 开始，`transaction_prealloc_size`已被弃用；事务内存池的初始大小是固定的，设置此变量不再起作用。（`transaction_alloc_block_size`的功能不受此更改影响。）预计`transaction_prealloc_size`将在未来的 MySQL 版本中移除。

+   `transaction_read_only`

    | 命令行格式 | `--transaction-read-only[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `transaction_read_only` |
    | 作用域 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    事务访问模式。该值可以是`OFF`（读/写；默认值）或`ON`（只读）。

    事务访问模式有三个作用域：全局，会话和下一个事务。这种三作用域实现导致了一些非标准的访问模式分配语义，稍后会描述。

    若要在启动时设置全局事务访问模式，请使用`--transaction-read-only`服务器选项。

    在运行时，可以直接使用`SET`语句为`transaction_read_only`系统变量分配值，或间接使用`SET TRANSACTION`语句。例如，使用此`SET`语句设置全局值：

    ```sql
    SET GLOBAL transaction_read_only = ON;
    ```

    设置全局`transaction_read_only`值会为所有后续会话设置访问模式。现有会话不受影响。

    若要设置会话或下一级别的`transaction_read_only`值，请使用`SET`语句。对于大多数会话系统变量，这些语句是设置值的等效方式：

    ```sql
    SET @@SESSION.*var_name* = *value*;
    SET SESSION *var_name* = *value*;
    SET *var_name* = *value*;
    SET @@*var_name* = *value*;
    ```

    如前所述，事务访问模式具有下一个事务范围，除了全局和会话范围。为了启用设置下一个事务范围，`SET`用于分配会话系统变量值的语法对于`transaction_read_only`具有非标准语义，

    +   要设置会话访问模式，请使用以下任何语法：

        ```sql
        SET @@SESSION.transaction_read_only = *value*;
        SET SESSION transaction_read_only = *value*;
        SET transaction_read_only = *value*;
        ```

        对于每个语法，这些语义适用：

        +   为会话中执行的所有后续事务设置访问模式。

        +   允许在事务中执行，但不影响当前正在进行的事务。

        +   如果在事务之间执行，则会覆盖任何设置下一个事务访问模式的前面语句。

        +   对应于`SET SESSION TRANSACTION {READ WRITE | READ ONLY}`（带有`SESSION`关键字）。

    +   要设置下一个事务访问模式，请使用此语法：

        ```sql
        SET @@transaction_read_only = *value*;
        ```

        对于该语法，这些语义适用：

        +   仅为会话中执行的下一个单个事务设置访问模式。

        +   后续事务将恢复为会话访问模式。

        +   不允许在事务中执行。

        +   对应于`SET TRANSACTION {READ WRITE | READ ONLY}`（不带`SESSION`关键字）。

    有关`SET TRANSACTION`及其与`transaction_read_only`系统变量的关系的更多信息，请参见 Section 15.3.7, “SET TRANSACTION Statement”。

+   `unique_checks`

    | 系统变量 | `unique_checks` |
    | --- | --- |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR`提示适�� | 是 |
    | 类型 | 布尔值 |
    | 默认值 | `ON` |

    如果设置为 1（默认值），则在`InnoDB`表中执行次要索引的唯一性检查。如果设置为 0，则允许存储引擎假定输入数据中不存在重复键。如果您确定您的数据不包含唯一性违规，可以将其设置为 0 以加快将大表导入到`InnoDB`中。

    将此变量设置为 0 不*要求*存储引擎忽略重复键。引擎仍然可以检查它们并在检测到它们时发出重复键错误。

+   `updatable_views_with_limit`

    | 命令行格式 | `--updatable-views-with-limit[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `updatable_views_with_limit` |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 是 |
    | 类型 | 布尔值 |
    | 默认值 | `1` |

    此变量控制当视图不包含底层表中定义的主键的所有列时，是否可以在更新语句包含`LIMIT`子句时进行视图更新（此类更新通常由 GUI 工具生成）。更新是`UPDATE`或`DELETE`语句。这里的主键是指`PRIMARY KEY`，或者一个不包含`NULL`的`UNIQUE`索引。

    该变量可以有两个值：

    +   `1`或`YES`：仅发出警告（而不是错误消息）。这是默认值。

    +   `0`或`NO`：禁止更新。

+   `use_secondary_engine`

    | 引入版本 | 8.0.13 |
    | --- | --- |
    | 系统变量 | `use_secondary_engine` |
    | 作用域 | 会话 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 是 |
    | 类型 | 枚举 |
    | 默认值 | `ON` |
    | 有效值 | `OFF``ON``FORCED` |

    供将来使用。

    是否使用辅助引擎执行查询。

    用于 HeatWave。请参阅 MySQL HeatWave 用户指南。

+   `validate_password.*`xxx`*`

    `validate_password`组件实现了一组具有形式为`validate_password.*`xxx`*`的系统变量。这些变量通过该组件影响密码测试；请参阅 Section 8.4.3.2, “Password Validation Options and Variables”。

+   `version`

    服务器的版本号。该值可能还包括指示服务器构建或配置信息的后缀。`-debug`表示服务器已启用调试支持进行构建。

+   `version_comment`

    | 系统变量 | `version_comment` |
    | --- | --- |
    | 作用域 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 字符串 |

    **CMake**配置程序具有一个`COMPILATION_COMMENT_SERVER`选项，允许在构建 MySQL 时指定注释。该变量包含该注释的值。（在 MySQL 8.0.14 之前，`version_comment`由`COMPILATION_COMMENT`选项设置。）请参阅 Section 2.8.7, “MySQL Source-Configuration Options”。

+   `version_compile_machine`

    | 系统变量 | `version_compile_machine` |
    | --- | --- |
    | 作用域 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 字符串 |

    服务器二进制文件的类型。

+   `version_compile_os`

    | 系统变量 | `version_compile_os` |
    | --- | --- |
    | 作用域 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 字符串 |

    MySQL 构建的操作系统类型。

+   `version_compile_zlib`

    | 系统变量 | `version_compile_zlib` |
    | --- | --- |
    | 作用域 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 字符串 |

    编译进 `zlib` 库的版本。

+   `wait_timeout`

    | 命令行格式 | `--wait-timeout=#` |
    | --- | --- |
    | 系统变量 | `wait_timeout` |
    | 作用域 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `28800` |
    | 最小值 | `1` |
    | 最大值（Windows） | `2147483` |
    | 最大值（其他） | `31536000` |
    | 单位 | 秒 |

    服务器在关闭非交互式连接之前等待活动的秒数。

    在线程启动时，会话 `wait_timeout` 值从全局 `wait_timeout` 值或全局 `interactive_timeout` 值初始化，取决于客户端类型（由 `CLIENT_INTERACTIVE` 连接选项到 `mysql_real_connect()` 定义）。另请参阅 `interactive_timeout`。

+   `warning_count`

    上一条生成消息的最后一条语句导致的错误、警告和注释数量。此变量只读。参见 第 15.7.7.42 节，“SHOW WARNINGS 语句”。

+   `windowing_use_high_precision`

    | 命令行格式 | `--windowing-use-high-precision[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `windowing_use_high_precision` |
    | 作用域 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 是 |
    | 类型 | 布尔 |
    | 默认值 | `ON` |

    是否计算窗口操作而不丢失精度。请参阅 Section 10.2.1.21, “窗口函数优化”。

+   `xa_detach_on_prepare`

    | 命令行格式 | `--xa-detach-on-prepare[={OFF&#124;ON}]` |
    | --- | --- |
    | 引入版本 | 8.0.29 |
    | 系统变量 | `xa_detach_on_prepare` |
    | 作用域 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔 |
    | 默认值 | `ON` |

    当设置为`ON`（启用）时，所有 XA 事务都会在`XA PREPARE`的一部分中与连接（会话）分离（断开）。这意味着即使原始连接尚未终止，其他连接也可以提交或回滚 XA 事务，并且此连接可以启动新事务。

    临时表不能在分离的 XA 事务中使用。

    当此设置为`OFF`（禁用）时，XA 事务严格与同一连接关联，直到会话断开。建议您允许其启用（默认行为）以用于复制。

    有关更多信息，请参阅 Section 15.3.8.2, “XA 事务状态”。
