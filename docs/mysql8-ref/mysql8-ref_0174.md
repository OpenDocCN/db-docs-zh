# 6.4.2 mysql_secure_installation — 改善 MySQL 安装安全性

> 译文：[`dev.mysql.com/doc/refman/8.0/en/mysql-secure-installation.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-secure-installation.html)

该程序使您能够通过以下方式改善 MySQL 安装的安全性：

+   您可以为`root`帐户设置密码。

+   您可以删除可以从本地主机外部访问的`root`帐户。

+   您可以删除匿名用户帐户。

+   您可以删除`test`数据库（默认情况下可以被所有用户访问，甚至是匿名用户），以及允许任何人访问以`test_`开头的数据库的权限。

**mysql_secure_installation**帮助您实施类似于第 2.9.4 节“保护初始 MySQL 帐户”中描述的安全建议。

正常使用是连接到本地 MySQL 服务器；无参数调用**mysql_secure_installation**：

```sql
mysql_secure_installation
```

当执行时，**mysql_secure_installation**会提示您确定要执行哪些操作。

`validate_password`组件可用于检查密码强度。如果未安装插件，**mysql_secure_installation**会提示用户是否安装。稍后输入的任何密码如果启用插件，则会使用插件进行检查。

大多数常见的 MySQL 客户端选项，如`--host`和`--port`可以在命令行和选项文件中使用。例如，要连接到本地服务器的 IPv6 端口 3307，请使用以下命令：

```sql
mysql_secure_installation --host=::1 --port=3307
```

**mysql_secure_installation**支持以下选项，可以在命令行或选项文件的`[mysql_secure_installation]`和`[client]`组中指定。有关 MySQL 程序使用的选项文件的信息，请参见第 6.2.2.2 节“使用选项文件”。

**表 6.9 mysql_secure_installation 选项**

| 选项名称 | 描述 | 引入 | 废弃 |
| --- | --- | --- | --- |
| --defaults-extra-file | 读取指定的选项文件以及通常的选项文件 |  |  |
| --defaults-file | 仅读取指定的选项文件 |  |  |
| --defaults-group-suffix | 选项组后缀值 |  |  |
| --help | 显示帮助信息并退出 |  |  |
| --host | MySQL 服务器所在的主机 |  |  |
| --no-defaults | 不读取任何选项文件 |  |  |
| --password | 被接受但始终被忽略。每当调用 mysql_secure_installation 时，用户都会被提示输入密码 |  |  |
| --port | 连接的 TCP/IP 端口号 |  |  |
| --print-defaults | 打印默认选项 |  |  |
| --protocol | 要使用的传输协议 |  |  |
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
| --ssl-session-data-continue-on-failed-reuse | 是否在会话重用失败时建立连接 | 8.0.29 |  |
| --tls-ciphersuites | 用于加密连接的允许的 TLSv1.3 密码套件 | 8.0.16 |  |
| --tls-version | 用于加密连接的允许的 TLS 协议 |  |  |
| --use-default | 无需用户交互执行 |  |  |
| --user | 连接到服务器时要使用的 MySQL 用户名 |  |  |
| 选项名称 | 描述 | 引入 | 废弃 |

+   `--help`, `-?`

    | 命令行格式 | `--help` |
    | --- | --- |

    显示帮助消息并退出。

+   `--defaults-extra-file=*`file_name`*`

    | 命令行格式 | `--defaults-extra-file=file_name` |
    | --- | --- |
    | 类型 | 文件名 |

    在全局选项文件之后但（在 Unix 上）在用户选项文件之前读取此选项文件。如果文件不存在或无法访问，将会出现错误。如果*`file_name`*不是绝对路径名，则将其解释为相对于当前目录的路径。

    有关此选项和其他选项文件选项的更多信息，请参见第 6.2.2.3 节，“影响选项文件处理的命令行选项”。

+   `--defaults-file=*`file_name`*`

    | 命令行格式 | `--defaults-file=file_name` |
    | --- | --- |
    | 类型 | 文件名 |

    仅使用给定的选项文件。如果文件不存在或无法访问，将会出现错误。如果*`file_name`*不是绝对路径名，则将其解释为相对于当前目录的路径。

    有关此选项和其他选项文件选项的更多信息，请参见第 6.2.2.3 节，“影响选项文件处理的命令行选项”。

+   `--defaults-group-suffix=*`str`*`

    | 命令行格式 | `--defaults-group-suffix=str` |
    | --- | --- |
    | 类型 | 字符串 |

    不仅读取通常的选项组，还读取具有通常名称和后缀*`str`*的组。例如，**mysql_secure_installation**通常会读取`[client]`和`[mysql_secure_installation]`组。如果将此选项指定为`--defaults-group-suffix=_other`，**mysql_secure_installation**还会读取`[client_other]`和`[mysql_secure_installation_other]`组。

    有关此选项和其他选项文件选项的更多信息，请参见第 6.2.2.3 节，“影响选项文件处理的命令行选项”。

+   `--host=*`host_name`*`, `-h *`host_name`*`

    | 命令行格式 | `--host` |
    | --- | --- |

    连接到给定主机上的 MySQL 服务器。

+   `--no-defaults`

    | 命令行格式 | `--no-defaults` |
    | --- | --- |

    不要读取任何选项文件。如果由于从选项文件中读取未知选项而导致程序启动失败，可以使用`--no-defaults`来防止读取它们。

    例外情况是，无论何时都会读取`.mylogin.cnf`文件（如果存在）。即使使用`--no-defaults`，这允许以比在命令行上更安全的方式指定密码。要创建`.mylogin.cnf`，请使用**mysql_config_editor**实用程序。请参见第 6.6.7 节，“mysql_config_editor — MySQL 配置实用程序”。

    有关此选项和其他选项文件选项的更多信息，请参见第 6.2.2.3 节，“影响选项文件处理的命令行选项”。

+   `--password=*`password`*`, `-p *`password`*`

    | 命令行格式 | `--password=password` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `[none]` |

    此选项被接受但被忽略。无论是否使用此选项，**mysql_secure_installation**始终提示用户输入密码。

+   `--port=*`port_num`*`, `-P *`port_num`*`

    | 命令行格式 | `--port=port_num` |
    | --- | --- |
    | 类型 | 数值 |
    | 默认值 | `3306` |

    对于 TCP/IP 连接，要使用的端口号。

+   `--print-defaults`

    | 命令行格式 | `--print-defaults` |
    | --- | --- |

    打印程序名称以及从选项文件获取的所有选项。

    有关此选项和其他选项文件选项的更多信息，请参阅第 6.2.2.3 节，“影响选项文件处理的命令行选项”。

+   `--protocol={TCP|SOCKET|PIPE|MEMORY}`

    | 命令行格式 | `--protocol=type` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `[见文本]` |
    | 有效值 | `TCP``SOCKET``PIPE``MEMORY` |

    用于连接到服务器的传输协议。当其他连接参数通常导致使用不同于您想要的协议时，这很有用。有关允许值的详细信息，请参阅第 6.2.7 节，“连接传输协议”。

+   `--socket=*`路径`*`, `-S *`路径`*`

    | 命令行格式 | `--socket={file_name&#124;pipe_name}` |
    | --- | --- |
    | 类型 | 字符串 |

    对于连接到`localhost`的连接，使用的 Unix 套接字文件，或者在 Windows 上使用的命名管道的名称。

    在 Windows 上，此选项仅在服务器启动时启用了支持命名管道连接的`named_pipe`系统变量时才适用。此外，连接必须是由`named_pipe_full_access_group`系统变量指定的 Windows 组的成员。

+   `--ssl*`

    以`--ssl`开头的选项指定是否使用加密连接到服务器，并指示 SSL 密钥和证书的位置。请参阅加密连接的命令选项。

+   `--ssl-fips-mode={OFF|ON|STRICT}`

    | 命令行格式 | `--ssl-fips-mode={OFF&#124;ON&#124;STRICT}` |
    | --- | --- |
    | 已弃用 | 8.0.34 |
    | 类型 | 枚举 |
    | 默认值 | `OFF` |
    | 有效值 | `OFF``ON``STRICT` |

    控制是否在客户端启用 FIPS 模式。`--ssl-fips-mode`选项与其他`--ssl-*`xxx`*`选项不同，它不用于建立加密连接，而是用于影响允许的加密操作。请参阅第 8.8 节，“FIPS 支持”。

    允许使用这些`--ssl-fips-mode`值：

    +   `OFF`: 禁用 FIPS 模式。

    +   `ON`: 启用 FIPS 模式。

    +   `STRICT`: 启用“严格” FIPS 模式。

    注意

    如果 OpenSSL FIPS 对象模块不可用，`--ssl-fips-mode` 的唯一允许值为 `OFF`。在这种情况下，将 `--ssl-fips-mode` 设置为 `ON` 或 `STRICT` 会导致客户端在启动时产生警告并在非 FIPS 模式下运行。

    从 MySQL 8.0.34 开始，此选项已被弃用。预计在将来的 MySQL 版本中将其移除。

+   `--tls-ciphersuites=*`ciphersuite_list`*`

    | 命令行格式 | `--tls-ciphersuites=ciphersuite_list` |
    | --- | --- |
    | 引入版本 | 8.0.16 |
    | 类型 | 字符串 |

    用于使用 TLSv1.3 的加密连接的允许的密码套件。该值是一个或多个以冒号分隔的密码套件名称列表。可以为此选项命名的密码套件取决于用于编译 MySQL 的 SSL 库。有关详细信息，请参见第 8.3.2 节，“加密连接 TLS 协议和密码”。

    此选项是在 MySQL 8.0.16 中添加的。

+   `--tls-version=*`protocol_list`*`

    | 命令行格式 | `--tls-version=protocol_list` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 (≥ 8.0.16) | `TLSv1,TLSv1.1,TLSv1.2,TLSv1.3` (OpenSSL 1.1.1 或更高版本)`TLSv1,TLSv1.1,TLSv1.2` (否则) |
    | 默认值 (≤ 8.0.15) | `TLSv1,TLSv1.1,TLSv1.2` |

    用于加密连接的允许的 TLS 协议。该值是一个或多个逗号分隔的协议名称列表。可以为此选项命名的协议取决于用于编译 MySQL 的 SSL 库。有关详细信息，请参见第 8.3.2 节，“加密连接 TLS 协议和密码”。

+   `--use-default`

    | 命令行格式 | `--use-default` |
    | --- | --- |
    | 类型 | 布尔值 |

    非交互执行。此选项可用于无人值守安装操作。

+   `--user=*`user_name`*`, `-u *`user_name`*`

    | 命令行格式 | `--user=user_name` |
    | --- | --- |
    | 类型 | 字符串 |

    用于连接到服务器的 MySQL 帐户的用户名。
