# 6.5.7 mysqlshow — 显示数据库、表和列信息

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysqlshow.html`](https://dev.mysql.com/doc/refman/8.0/en/mysqlshow.html)

**mysqlshow**客户端可用于快速查看存在哪些数据库、它们的表，或表的列或索引。

**mysqlshow**提供了一个命令行接口来执行几个 SQL `SHOW`语句。请参阅 Section 15.7.7, “SHOW 语句”。相同的信息可以通过直接使用这些语句来获取。例如，您可以从**mysql**客户端程序中发出它们。

调用像这样的**mysqlshow**：

```sql
mysqlshow [*options*] [*db_name* [*tbl_name* [*col_name*]]]
```

+   如果没有给出数据库，则显示数据库名称列表。

+   如果没有给出表，则显示数据库中所有匹配的表。

+   如果没有给出列，则显示表中所有匹配的列和列类型。

输出仅显示您具有某些权限的那些数据库、表或列的名称。

如果最后一个参数包含 shell 或 SQL 通配符(`*`, `?`, `%`, 或 `_`)，则只显示与通配符匹配的名称。如果数据库名称包含任何下划线，那么应该用反斜杠（某些 Unix shell 需要两个）对其进行转义，以获取正确的表或列列表。`*` 和 `?` 字符会转换为 SQL `%` 和 `_` 通配符。当您尝试显示具有名称中带有 `_` 的表的列时，这可能会导致一些混淆，因为在这种情况下，**mysqlshow**只会显示与模式匹配的表名。这很容易通过在命令行最后添加额外的 `%` 作为单独参数来解决。

**mysqlshow**支持以下选项，这些选项可以在命令行或选项文件的`[mysqlshow]`和`[client]`组中指定。有关 MySQL 程序使用的选项文件的信息，请参阅 Section 6.2.2.2, “使用选项文件”。

**表 6.18 mysqlshow 选项**

| 选项名称 | 描述 | 引入 | 废弃 |
| --- | --- | --- | --- |
| --bind-address | 使用指定的网络接口连接到 MySQL 服务器 |  |  |
| --character-sets-dir | 字符集所在目录 |  |  |
| --compress | 在客户端和服务器之间发送的所有信息进行压缩 |  | 8.0.18 |
| --compression-algorithms | 连接到服务器时允许的压缩算法 | 8.0.18 |  |
| --count | 每个表的行数 |  |  |
| --debug | 写入调试日志 |  |  |
| --debug-check | 程序退出时打印调试信息 |  |  |
| --debug-info | 程序退出时打印调试信息、内存和 CPU 统计信息 |  |  |
| --default-auth | 要使用的认证插件 |  |  |
| --default-character-set | 指定默认字符集 |  |  |
| --defaults-extra-file | 除了通常的选项文件之外，读取指定的选项文件 |  |  |
| --defaults-file | 仅读取指定的选项文件 |  |  |
| --defaults-group-suffix | 选项组后缀值 |  |  |
| --enable-cleartext-plugin | 启用明文认证插件 |  |  |
| --get-server-public-key | 从服务器请求 RSA 公钥 |  |  |
| --help | 显示帮助信息并退出 |  |  |
| --host | MySQL 服务器所在的主机 |  |  |
| --keys | 显示表索引 |  |  |
| --login-path | 从.mylogin.cnf 文件中读取登录路径选项 |  |  |
| --no-defaults | 不读取任何选项文件 |  |  |
| --password | 连接到服务器时使用的密码 |  |  |
| --password1 | 连接到服务器时使用的第一个多因��身份验证密码 | 8.0.27 |  |
| --password2 | 连接到服务器时使用的第二个多因素身份验证密码 | 8.0.27 |  |
| --password3 | 连接到服务器时使用的第三个多因素身份验证密码 | 8.0.27 |  |
| --pipe | 使用命名管道连接到服务器（仅限 Windows） |  |  |
| --plugin-dir | 插件安装目录 |  |  |
| --port | 连接的 TCP/IP 端口号 |  |  |
| --print-defaults | 打印默认选项 |  |  |
| --protocol | 使用的传输协议 |  |  |
| --server-public-key-path | 包含 RSA 公钥的文件路径名 |  |  |
| --shared-memory-base-name | 共享内存连接的共享内存名称（仅限 Windows） |  |  |
| --show-table-type | 显示指示表类型的列 |  |  |
| --socket | 要使用的 Unix 套接字文件或 Windows 命名管道 |  |  |
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
| --status | 显示有关每个表的额外信息 |  |  |
| --tls-ciphersuites | 用于加密连接的可接受的 TLSv1.3 密码套件 | 8.0.16 |  |
| --tls-version | 用于加密连接的可接受的 TLS 协议 |  |  |
| --user | 连接到服务器时要使用的 MySQL 用户名 |  |  |
| --verbose | 详细模式 |  |  |
| --version | 显示版本信息并退出 |  |  |
| --zstd-compression-level | 用于使用 zstd 压缩的服务器连接的压缩级别 | 8.0.18 |  |
| 选项名称 | 描述 | 引入版本 | 废弃版本 |

+   `--help`, `-?`

    | 命令行格式 | `--help` |
    | --- | --- |

    显示帮助信息并退出。

+   `--bind-address=*`ip_address`*`

    | 命令行格式 | `--bind-address=ip_address` |
    | --- | --- |

    在具有多个网络接口的计算机上，使用此选项选择连接到 MySQL 服务器的接口。

+   `--character-sets-dir=*`dir_name`*`

    | 命令行格式 | `--character-sets-dir=path` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `[none]` |

    安装字符集的目录。请参见第 12.15 节，“字符集配置”。

+   `--compress`, `-C`

    | 命令行格式 | `--compress[={OFF&#124;ON}]` |
    | --- | --- |
    | 弃用 | 8.0.18 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    如果可能，压缩客户端和服务器之间发送的所有信息。请参见第 6.2.8 节，“连接压缩控制”。

    从 MySQL 8.0.18 开始，此选项已被弃用。预计在未来的 MySQL 版本中将被移除。请参见配置传统连接压缩。

+   `--compression-algorithms=*`value`*`

    | 命令行格式 | `--compression-algorithms=value` |
    | --- | --- |
    | 引入版本 | 8.0.18 |
    | 类型 | 集合 |
    | 默认值 | `uncompressed` |
    | 有效值 | `zlib``zstd``uncompressed` |

    连接到服务器的允许压缩算法。可用算法与`protocol_compression_algorithms`系统变量相同。默认值为`uncompressed`。

    更多信息，请参见第 6.2.8 节，“连接压缩控制”。

    此选项是在 MySQL 8.0.18 中添加的。

+   `--count`

    | 命令行格式 | `--count` |
    | --- | --- |

    显示每个表的行数。对于非`MyISAM`表来说可能会很慢。

+   [`--debug[=*`debug_options`*]`](mysqlshow.html#option_mysqlshow_debug), `-# [*`debug_options`*]`

    | 命令行格式 | `--debug[=debug_options]` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `d:t:o` |

    写入调试日志。典型的*`debug_options`*字符串是`d:t:o,*`file_name`*`。默认值为`d:t:o`。

    只有在使用`WITH_DEBUG`构建 MySQL 时才可用此选项。由 Oracle 提供的 MySQL 发布二进制文件*不*使用此选项构建。

+   `--debug-check`

    | 命令行格式 | `--debug-check` |
    | --- | --- |
    | 类型 | 布尔值 |
    | 默认值 | `FALSE` |

    程序退出时打印一些调试信息。

    只有在使用`WITH_DEBUG`构建 MySQL 时才可用此选项。由 Oracle 提供的 MySQL 发布二进制文件*不*使用此选项构建。

+   `--debug-info`

    | 命令行格式 | `--debug-info` |
    | --- | --- |
    | 类型 | 布尔值 |
    | 默认值 | `FALSE` |

    在程序退出时打印调试信息以及内存和 CPU 使用统计信息。

    只有在使用`WITH_DEBUG`构建 MySQL 时才可用此选项。由 Oracle 提供的 MySQL 发布二进制文件*不*使用此选项构建。

+   `--default-character-set=*`charset_name`*`

    | 命令行格式 | `--default-character-set=charset_name` |
    | --- | --- |
    | 类型 | 字符串 |

    将*`charset_name`*作为默认字符集。请参见第 12.15 节，“字符集配置”。

+   `--default-auth=*`plugin`*`

    | 命令行格式 | `--default-auth=plugin` |
    | --- | --- |
    | 类型 | 字符串 |

    关于要使用哪个客户端端身份验证插件的提示。请参见第 8.2.17 节，“可插拔认证”。

+   `--defaults-extra-file=*`file_name`*`

    | 命令行格式 | `--defaults-extra-file=file_name` |
    | --- | --- |
    | 类型 | 文件名 |

    在全局选项文件之后但（在 Unix 上）在用户选项文件之前读取此选项文件。如果文件不存在或无法访问，将会出现错误。如果*`file_name`*不是绝对路径名，则将其解释为相对于当前目录。

    有关此选项文件选项和其他选项的更多信息，请参见第 6.2.2.3 节，“影响选项文件处理的命令行选项”。

+   `--defaults-file=*`file_name`*`

    | 命令行格式 | `--defaults-file=file_name` |
    | --- | --- |
    | 类型 | 文件名 |

    仅使用给定的选项文件。如果文件不存在或无法访问，将会出现错误。如果*`file_name`*不是绝对路径名，则将其解释为相对于当前目录。

    例外：即使使用`--defaults-file`，客户端程序也会读取`.mylogin.cnf`。

    有关此选项文件选项和其他选项的更多信息，请参见第 6.2.2.3 节，“影响选项文件处理的命令行选项”。

+   `--defaults-group-suffix=*`str`*`

    | 命令行格式 | `--defaults-group-suffix=str` |
    | --- | --- |
    | 类型 | 字符串 |

    不仅读取通常的选项组，还读取具有通常名称和后缀*`str`*的组。例如，**mysqlshow**通常会读取`[client]`和`[mysqlshow]`组。如果此选项被指定为`--defaults-group-suffix=_other`，**mysqlshow**还会读取`[client_other]`和`[mysqlshow_other]`组。

    有关此选项和其他选项文件选项的更多信息，请参见第 6.2.2.3 节，“影响选项文件处理的命令行选项”。

+   `--enable-cleartext-plugin`

    | 命令行格式 | `--enable-cleartext-plugin` |
    | --- | --- |
    | 类型 | 布尔值 |
    | 默认值 | `FALSE` |

    启用`mysql_clear_password`明文认证插件。（参见第 8.4.1.4 节，“客户端明文插件认证”。）

+   `--get-server-public-key`

    | 命令行格式 | `--get-server-public-key` |
    | --- | --- |
    | 类型 | 布尔值 |

    从服务器请求用于基于密钥对的密码交换的 RSA 公钥。此选项适用于使用通过`caching_sha2_password`认证插件进行身份验证的帐户连接到服务器的客户端。对于通过这些帐户连接的连接，除非请求，否则服务器不会将公钥发送给客户端。对于不使用该插件进行身份验证的帐户，此选项将被忽略。如果不需要基于 RSA 的密码交换，例如客户端使用安全连接连接到服务器时，也将被忽略。

    如果`--server-public-key-path=*`file_name`*`被指定并指定了有效的公钥文件，则它优先于`--get-server-public-key`。

    有关`caching_sha2_password`插件的信息，请参见第 8.4.1.2 节，“缓存 SHA-2 可插拔认证”。

+   `--host=*`host_name`*`，`-h *`host_name`*`

    | 命令行格式 | `--host=host_name` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `localhost` |

    连接到给定主机上的 MySQL 服务器。

+   `--keys`，`-k`

    | 命令行格式 | `--keys` |
    | --- | --- |

    显示表索引。

+   `--login-path=*`name`*`

    | 命令行格式 | `--login-path=name` |
    | --- | --- |
    | 类型 | 字符串 |

    从`.mylogin.cnf`登录路径文件中读取命名登录路径的选项。 “登录路径”是一个包含指定要连接到哪个 MySQL 服务器以及要作为哪个帐户进行身份验证的选项的选项组。要创建或修改登录路径文件，请使用**mysql_config_editor**实用程序。请参见第 6.6.7 节，“mysql_config_editor — MySQL 配置实用程序”。

    有关此选项和其他选项文件选项的更多信息，请参见第 6.2.2.3 节，“影响选项文件处理的命令行选项”。

+   `--no-defaults`

    | 命令行格式 | `--no-defaults` |
    | --- | --- |

    不要读取任何选项文件。如果由于从选项文件中读取未知选项而导致程序启动失败，则可以使用`--no-defaults`来阻止读取这些选项。

    例外情况是，如果存在`.mylogin.cnf`文件，则在所有情况下都会读取该文件。即使使用`--no-defaults`，这也允许以比在命令行上更安全的方式指定密码。要创建`.mylogin.cnf`，请使用**mysql_config_editor**实用程序。请参见第 6.6.7 节，“mysql_config_editor — MySQL 配置实用程序”。

    有关此选项和其他选项文件选项的更多信息，请参见第 6.2.2.3 节，“影响选项文件处理的命令行选项”。

+   [`--password[=*`password`*]`](mysqlshow.html#option_mysqlshow_password), `-p[*`password`*]`

    | 命令行格式 | `--password[=password]` |
    | --- | --- |
    | 类型 | 字符串 |

    用于连接服务器的 MySQL 帐户的密码。密码值是可选的。如果未提供，**mysqlshow**会提示输入密码。如果提供了密码，则`--password=`或`-p`后面必须*没有空格*。如果未指定密码选项，则默认不发送密码。

    在命令行上指定密码应被视为不安全。为了避免在命令行上提供密码，请使用选项文件。请参见第 8.1.2.1 节，“密码安全的最终用户指南”。

    要明确指定没有密码，并且**mysqlshow** 不应提示输入密码，请使用`--skip-password`选项。

+   [`--password1[=*`pass_val`*]`](mysqlshow.html#option_mysqlshow_password1)

    MySQL 账户用于连接服务器的多因素认证因子 1 的密码。密码值是可选的。如果未提供，**mysqlshow** 将提示输入密码。如果提供，`--password1=` 和后面的密码之间*不能有空格*。如果未指定密码选项，则默认不发送密码。

    在命令行上指定密码应被视为不安全。为了避免在命令行上提供密码，请使用选项文件。请参阅 Section 8.1.2.1, “End-User Guidelines for Password Security”。

    要明确指定没有密码，并且**mysqlshow** 不应提示输入密码，请使用`--skip-password1`选项。

    `--password1` 和 `--password` 是同义词，`--skip-password1` 和 `--skip-password` 也是同义词。

+   [`--password2[=*`pass_val`*]`](mysqlshow.html#option_mysqlshow_password2)

    MySQL 账户用于连接服务器的多因素认证因子 2 的密码。此选项的语义与`--password1`的语义类似；有关详细信息，请参阅该选项的描述。

+   [`--password3[=*`pass_val`*]`](mysqlshow.html#option_mysqlshow_password3)

    MySQL 账户用于连接服务器的多因素认证因子 3 的密码。此选项的语义与`--password1`的语义类似；有关详细信息，请参阅该选项的描述。

+   `--pipe`, `-W`

    | 命令行格式 | `--pipe` |
    | --- | --- |
    | 类型 | 字符串 |

    在 Windows 上，使用命名管道连接到服务器。此选项仅适用于服务器启用了`named_pipe`系统变量以支持命名管道连接。此外，进行连接的用户必须是由`named_pipe_full_access_group`系统变量指定的 Windows 组的成员。

+   `--plugin-dir=*`dir_name`*`

    | 命令行格式 | `--plugin-dir=dir_name` |
    | --- | --- |
    | 类型 | 目录名称 |

    查找插件的目录。如果使用`--default-auth`选项指定了认证插件但**mysqlshow**找不到它，则指定此选项。参见第 8.2.17 节，“可插拔认证”。

+   `--port=*`port_num`*`, `-P *`port_num`*`

    | 命令行格式 | `--port=port_num` |
    | --- | --- |
    | 类型 | 数字 |
    | 默认值 | `3306` |

    对于 TCP/IP 连接，要使用的端口号。

+   `--print-defaults`

    | 命令行格式 | `--print-defaults` |
    | --- | --- |

    打印程序名称以及从选项文件获取的所有选项。

    有关此选项和其他选项文件选项的更多信息，请参见第 6.2.2.3 节，“影响选项文件处理的命令行选项”。

+   `--protocol={TCP|SOCKET|PIPE|MEMORY}`

    | 命令行格式 | `--protocol=type` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `[见文本]` |
    | 有效值 | `TCP``SOCKET``PIPE``MEMORY` |

    用于连接到服务器的传输协议。当其他连接参数通常导致使用您不想要的协议时，这很有用。有关允许值的详细信息，请参见第 6.2.7 节，“连接传输协议”。

+   `--server-public-key-path=*`file_name`*`

    | 命令行格式 | `--server-public-key-path=file_name` |
    | --- | --- |
    | 类型 | 文件名 |

    PEM 格式文件中包含客户端公钥的路径名，该公钥是服务器用于 RSA 密钥对密码交换的必需部分。此选项适用于使用`sha256_password`或`caching_sha2_password`认证插件进行身份验证的客户端。对于不使用这些插件进行身份验证的帐户，此选项将被忽略。如果不使用基于 RSA 的密码交换，例如客户端通过安全连接连接到服务器时，此选项也将被忽略。

    如果给定`--server-public-key-path=*`file_name`*`并指定有效的公钥文件，则它优先于`--get-server-public-key`。

    对于`sha256_password`，此选项仅在 MySQL 使用 OpenSSL 构建时适用。

    有关`sha256_password`和`caching_sha2_password`插件的信息，请参阅第 8.4.1.3 节，“SHA-256 可插拔认证”和第 8.4.1.2 节，“缓存 SHA-2 可插拔认证”。

+   `--shared-memory-base-name=*`name`*`

    | 命令行格式 | `--shared-memory-base-name=name` |
    | --- | --- |
    | 平台特定 | Windows |

    在 Windows 上，用于使用共享内存连接到本地服务器的共享内存名称。默认值为`MYSQL`。共享内存名称区分大小写。

    此选项仅在���务器启动时启用了`shared_memory`系统变量以支持共享内存连接时才适用。

+   `--show-table-type`, `-t`

    | 命令行格式 | `--show-table-type` |
    | --- | --- |

    显示指示表类型的列，如`SHOW FULL TABLES`。类型为`BASE TABLE`或`VIEW`。

+   `--socket=*`path`*`, `-S *`path`*`

    | 命令行格式 | `--socket={file_name&#124;pipe_name}` |
    | --- | --- |
    | 类型 | 字符串 |

    对于对`localhost`的连接，要使用的 Unix 套接字文件，或者在 Windows 上，要使用的命名管道的名称。

    在 Windows 上，此选项仅在服务器启动时启用了`named_pipe`系统变量以支持命名管道连接时才适用。此外，进行连接的用户必须是由`named_pipe_full_access_group`系统变量指定的 Windows 组的成员。

+   `--ssl*`

    以`--ssl`开头的选项指定是否使用加密连接到服务器，并指示 SSL 密钥和证书的位置。请参阅加密连接的命令选项。

+   `--ssl-fips-mode={OFF|ON|STRICT}`

    | 命令行格式 | `--ssl-fips-mode={OFF&#124;ON&#124;STRICT}` |
    | --- | --- |
    | 已弃用 | 8.0.34 |
    | 类型 | 枚举 |
    | 默认值 | `OFF` |
    | 有效数值 | `OFF``ON``STRICT` |

    控制是否在客户端端启用 FIPS 模式。`--ssl-fips-mode`选项与其他`--ssl-*`xxx`*`选项不同，它不用于建立加密连接，而是影响允许进行的加密操作。请参阅第 8.8 节，“FIPS 支持”。

    这些`--ssl-fips-mode`值是允许的：

    +   `OFF`：禁用 FIPS 模式。

    +   `ON`：启用 FIPS 模式。

    +   `STRICT`: 启用“严格”FIPS 模式。

    注意

    如果 OpenSSL FIPS 对象模块不可用，则`--ssl-fips-mode`的唯一允许值为`OFF`。在这种情况下，将`--ssl-fips-mode`设置为`ON`或`STRICT`会导致客户端在启动时产生警告并在非 FIPS 模式下运行。

    从 MySQL 8.0.34 开始，此选项已被弃用。预计在未来的 MySQL 版本中将被移除。

+   `--status`, `-i`

    | 命令行格式 | `--status` |
    | --- | --- |

    显示每个表的额外信息。

+   `--tls-ciphersuites=*`ciphersuite_list`*`

    | 命令行格式 | `--tls-ciphersuites=ciphersuite_list` |
    | --- | --- |
    | 引入版本 | 8.0.16 |
    | 类型 | 字符串 |

    用于使用 TLSv1.3 的加密连接的允许密码套件。该值是一个或多个以冒号分隔的密码套件名称列表。可以为此选项命名的密码套件取决于用于编译 MySQL 的 SSL 库。有关详细信息，请参见第 8.3.2 节，“加密连接 TLS 协议和密码套件”。

    此选项是在 MySQL 8.0.16 中添加的。

+   `--tls-version=*`protocol_list`*`

    | 命令行格式 | `--tls-version=protocol_list` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值（≥ 8.0.16） | `TLSv1,TLSv1.1,TLSv1.2,TLSv1.3`（OpenSSL 1.1.1 或更高版本）`TLSv1,TLSv1.1,TLSv1.2`（否则） |
    | 默认值（≤ 8.0.15） | `TLSv1,TLSv1.1,TLSv1.2` |

    用于加密连接的允许的 TLS 协议。该值是一个或多个以逗号分隔的协议名称列表。可以为此选项命名的协议取决于用于编译 MySQL 的 SSL 库。有关详细信息，请参见第 8.3.2 节，“加密连接 TLS 协议和密码套件”。

+   `--user=*`user_name`*`, `-u *`user_name`*`

    | 命令行格式 | `--user=user_name,` |
    | --- | --- |
    | 类型 | 字符串 |

    用于连接到服务器的 MySQL 帐户的用户名。

+   `--verbose`, `-v`

    | 命令行格式 | `--verbose` |
    | --- | --- |

    详细模式。打印程序执行的更多信息。可以多次使用此选项以增加信息量。

+   `--version`, `-V`

    | 命令行格式 | `--version` |
    | --- | --- |

    显示版本信息并退出。

+   `--zstd-compression-level=*`level`*`

    | 命令行格式 | `--zstd-compression-level=#` |
    | --- | --- |
    | 引入版本 | 8.0.18 |
    | 类型 | 整数 |

    用于使用`zstd`压缩算法连接到服务器的压缩级别。允许的级别从 1 到 22，较大的值表示更高级别的压缩。默认的`zstd`压缩级别为 3。压缩级别设置对不使用`zstd`压缩的连接没有影响。

    欲了解更多信息，请参阅第 6.2.8 节，“连接压缩控制”。

    此选项是在 MySQL 8.0.18 中添加的。
