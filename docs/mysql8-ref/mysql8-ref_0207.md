# 6.6.8 mysql_migrate_keyring — 密钥迁移实用程序

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-migrate-keyring.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-migrate-keyring.html)

**mysql_migrate_keyring**实用程序在一个密钥环组件和另一个之间迁移密钥。它支持离线和在线迁移。

像这样调用**mysql_migrate_keyring**（在一行上输入命令）：

```sql
mysql_migrate_keyring
  --component-dir=*dir_name*
  --source-keyring=*name*
  --destination-keyring=*name*
  [*other options*]
```

有关密钥迁移的信息以及使用**mysql_migrate_keyring**和其他方法执行它们的说明，请参见第 8.4.4.14 节，“在密钥环密钥存储之间迁移密钥”。

**mysql_migrate_keyring**支持以下选项，可以在命令行或选项文件的`[mysql_migrate_keyring]`组中指定。有关 MySQL 程序使用的选项文件的信息，请参见第 6.2.2.2 节，“使用选项文件”。  

**表 6.22 mysql_migrate_keyring 选项**

| 选项名称 | 描述 | 引入 | 废弃 |
| --- | --- | --- | --- |
| --component-dir | 密钥环组件目录 |  |  |
| --defaults-extra-file | 除了通常的选项文件外，还读取命名选项文件 |  |  |
| --defaults-file | 仅读取命名选项文件 |  |  |
| --defaults-group-suffix | 选项组后缀值 |  |  |
| --destination-keyring | 目标密钥环组件名称 |  |  |
| --destination-keyring-configuration-dir | 目标密钥环组件配置目录 |  |  |
| --get-server-public-key | 从服务器请求 RSA 公钥 |  |  |
| --help | 显示帮助信息并退出 |  |  |
| --host | MySQL 服务器所在的主机 |  |  |
| --login-path | 从.mylogin.cnf 中读取登录路径选项 |  |  |
| --no-defaults | 不读取任何选项文件 |  |  |
| --online-migration | 迁移源是活动服务器 |  |  |
| --password | 连接到服务器时要使用的密码 |  |  |
| --port | 连接的 TCP/IP 端口号 |  |  |
| --print-defaults | 打印默认选项 |  |  |
| --server-public-key-path | 包含 RSA 公钥的文件路径名 |  |  |
| --socket | 要使用的 Unix 套接字文件或 Windows 命名管道 |  |  |
| --source-keyring | 源密钥环组件名称 |  |  |
| --source-keyring-configuration-dir | 源密钥环组件配置目录 |  |  |
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
| --tls-ciphersuites | 加密连接的允许的 TLSv1.3 密码套件 |  |  |
| --tls-version | 加密连接的允许 TLS 协议 |  |  |
| --user | 连接到服务器时要使用的 MySQL 用户名 |  |  |
| --verbose | 详细模式 |  |  |
| --version | 显示版本信息并退出 |  |  |
| 选项名称 | 描述 | 引入 | 废弃 |

+   `--help`, `-h`

    | 命令行格式 | `--help` |
    | --- | --- |

    显示帮助信息并退出。

+   `--component-dir=*`dir_name`*`

    | 命令行格式 | `--component-dir=dir_name` |
    | --- | --- |
    | 类型 | 目录名 |

    存放关键环组件的目录。这通常是本地 MySQL 服务器的`plugin_dir`系统变量的值。

    注意

    对于由**mysql_migrate_keyring**执行的所有关键环迁移操作，`--component-dir`、`--source-keyring`和`--destination-keyring`是强制的。此外，源组件和目标组件必须不同，并且两个组件必须配置正确，以便**mysql_migrate_keyring**可以加载和使用它们。

+   `--defaults-extra-file=*`file_name`*`

    | 命令行格式 | `--defaults-extra-file=file_name` |
    | --- | --- |
    | 类型 | 文件名 |

    在全局选项文件之后但（在 Unix 上）在用户选项文件之前读取此选项文件。如果文件不存在或无法访问，将会出现错误。如果*`file_name`*不是绝对路径名，则将其解释为相对于当前目录。

    有关此选项文件和其他选项文件选项的更多信息，请参见第 6.2.2.3 节，“影响选项文件处理的命令行选项”。

+   `--defaults-file=*`file_name`*`

    | 命令行格式 | `--defaults-file=file_name` |
    | --- | --- |
    | 类型 | 文件名 |

    仅使用给定的选项文件。如果文件不存在或无法访问，将会出现错误。如果*`file_name`*不是绝对路径名，则将其解释为相对于当前目录。

    例外：即使使用`--defaults-file`，客户端程序也会读取`.mylogin.cnf`。

    有关此选项和其他选项文件选项的更多信息，请参见第 6.2.2.3 节，“影响选项文件处理的命令行选项”。

+   `--defaults-group-suffix=*`str`*`

    | 命令行格式 | `--defaults-group-suffix=str` |
    | --- | --- |
    | 类型 | 字符串 |

    不仅读取通常的选项组，还读取具有通常名称和后缀*`str`*的组。例如，**mysql_migrate_keyring**通常会读取`[mysql_migrate_keyring]`组。如果将此选项给定为`--defaults-group-suffix=_other`，**mysql_migrate_keyring**还会读取`[mysql_migrate_keyring_other]`组。

    有关此选项和其他选项文件选项的更多信息，请参见第 6.2.2.3 节，“影响选项文件处理的命令行选项”。

+   `--destination-keyring=*`name`*`

    | 命令行格式 | `--destination-keyring=name` |
    | --- | --- |
    | 类型 | 字符串 |

    用于密钥迁移的目标密钥环组件。选项值的格式和解释与`--source-keyring`选项描述的相同。

    注意

    `--component-dir`，`--source-keyring`和`--destination-keyring`对由**mysql_migrate_keyring**执行的所有密钥环迁移操作都是强制性的。此外，源组件和目标组件必须不同，并且两个组件必须配置正确，以便**mysql_migrate_keyring**可以加载和使用它们。

+   `--destination-keyring-configuration-dir=*`dir_name`*`

    | 命令行格式 | `--destination-keyring-configuration-dir=dir_name` |
    | --- | --- |
    | 类型 | 目录名称 |

    仅当目标密钥环组件全局配置文件包含`"read_local_config": true`时，此选项才适用，表示组件配置包含在本地配置文件中。选项值指定包含该本地文件的目录。

+   `--get-server-public-key`

    | 命令行格式 | `--get-server-public-key` |
    | --- | --- |
    | 类型 | 布尔值 |

    从服务器请求用于 RSA 密钥对密码交换所需的公钥。此选项适用于使用`caching_sha2_password`认证插件进行身份验证的客户端。对于该插件，除非请求，否则服务器不会发送公钥。对于不使用该插件进行身份验证的帐户，此选项将被忽略。如果不使用基于 RSA 的密码交换，例如客户端使用安全连接连接到服务器时，此选项也将被忽略。

    如果给定`--server-public-key-path=*`file_name`*`并指定有效的公钥文件，则优先于`--get-server-public-key`。

    有关`caching_sha2_password`插件的信息，请参见第 8.4.1.2 节，“缓存 SHA-2 可插拔认证”。

+   `--host=*`host_name`*`，`-h *`host_name`*`

    | 命令行格式 | `--host=host_name` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `localhost` |

    当前正在使用其中一个密钥迁移密钥库的运行服务器的主机位置。迁移始终在本地主机上进行，因此该选项始终指定用于连接到本地服务器的值，例如`localhost`，`127.0.0.1`，`::1`或本地主机 IP 地址或主机名。

+   `--login-path=*`name`*`

    | 命令行格式 | `--login-path=name` |
    | --- | --- |
    | 类型 | 字符串 |

    从`.mylogin.cnf`登录路径文件中读取指定登录路径中的选项。 “登录路径”是一个包含指定要连接到哪个 MySQL 服务器以及要作为哪个帐户进行身份验证的选项的选项组。要创建或修改登录路径文件，请使用**mysql_config_editor**实用程序。请参见第 6.6.7 节，“mysql_config_editor — MySQL 配置实用程序”。

    有关此选项文件和其他选项文件选项的更多信息，请参见第 6.2.2.3 节，“影响选项文件处理的命令行选项”。

+   `--no-defaults`

    | 命令行格式 | `--no-defaults` |
    | --- | --- |

    不要读取任何选项文件。如果由于从选项文件中读取未知选项而导致程序启动失败，可以使用`--no-defaults`来防止读取它们。

    例外情况是无论何种情况下都会读取`.mylogin.cnf`文件，如果存在的话。这允许以比在命令行上更安全的方式指定密码，即使使用`--no-defaults`也是如此。要创建`.mylogin.cnf`，请使用**mysql_config_editor**实用程序。请���见 第 6.6.7 节，“mysql_config_editor — MySQL 配置实用程序”。

    有关此选项和其他选项文件选项的更多信息，请参见 第 6.2.2.3 节，“影响选项文件处理的命令行选项”。

+   `--online-migration`

    | 命令行格式 | `--online-migration` |
    | --- | --- |
    | 类型 | 布尔值 |
    | 默认值 | `FALSE` |

    当运行服务器使用密钥环时，此选项是强制性的。它告诉**mysql_migrate_keyring**执行在线密钥迁移。该选项具有以下效果：

    +   **mysql_migrate_keyring** 连接到服务器时使用指定的任何连接选项；否则这些选项将被忽略。

    +   在**mysql_migrate_keyring**连接到服务器后，它告诉服务器暂停密钥环操作。当密钥复制完成后，**mysql_migrate_keyring**告诉服务器可以在断开连接之前恢复密钥环操作。

+   [`--password[=*`password`*]`](mysql-migrate-keyring.html#option_mysql_migrate_keyring_password), `-p[*`password`*]`

    | 命令行格式 | `--password[=password]` |
    | --- | --- |
    | 类型 | 字符串 |

    用于连接到当前使用密钥迁移密钥库的运行服务器的 MySQL 帐户的密码。密码值是可选的。如果未提供，**mysql_migrate_keyring**会提示输入密码。如果提供了密码选项，则`--password=`或`-p`后面必须*没有空格*。如果未指定密码选项，则默认情况下不发送密码。

    在命令行上指定密码应被视为不安全。为了避免在命令行上输入密码，请使用一个选项文件。参见第 8.1.2.1 节，“密码安全的最终用户指南”。

    明确指定没有密码，并且**mysql_migrate_keyring**不应提示密码，使用`--skip-password`选项。

+   `--port=*`port_num`*`, `-P *`port_num`*`

    | 命令行格式 | `--port=port_num` |
    | --- | --- |
    | 类型 | 数值 |
    | 默认值 | `0` |

    对于 TCP/IP 连接，用于连接到当前使用其中一个密钥迁移密钥库的运行服务器的端口号。

+   `--print-defaults`

    | 命令行格式 | `--print-defaults` |
    | --- | --- |

    打印程序名称以及从选项文件获取的所有选项。

    有关此选项和其他选项文件选项的附加信息，请参见第 6.2.2.3 节，“影响选项文件处理的命令行选项”。

+   `--server-public-key-path=*`file_name`*`

    | 命令行格式 | `--server-public-key-path=file_name` |
    | --- | --- |
    | 类型 | 文件名 |

    PEM 格式文件的路径名，其中包含服务器所需的用于 RSA 密钥对密码交换的客户端端公钥的副本。此选项适用于使用`sha256_password`或`caching_sha2_password`认证插件进行身份验证的客户端。对于不使用这些插件进行身份验证的帐户，此选项将被忽略。如果不使用基于 RSA 的密码交换，即当客户端使用安全连接连接到服务器时，此选项也将被忽略。

    如果给定`--server-public-key-path=*`file_name`*`并指定一个有效的公钥文件，则它优先于`--get-server-public-key`。

    对于`sha256_password`，此选项仅在 MySQL 使用 OpenSSL 构建时适用。

    有关`sha256_password`和`caching_sha2_password`插件的信息，请参见第 8.4.1.3 节，“SHA-256 可插拔认证”和第 8.4.1.2 节，“缓存 SHA-2 可插拔认证”。

+   `--socket=*`path`*`, `-S *`path`*`

    | 命令行格式 | `--socket={file_name&#124;pipe_name}` |
    | --- | --- |
    | 类型 | 字符串 |

    对于 Unix 套接字文件或 Windows 命名管道连接，用于连接到当前使用其中一个密钥迁移密钥库的运行服务器的套接字文件或命名管道。

    在 Windows 上，只有在服务器启动时启用了`named_pipe`系统变量以支持命名管道连接时，此选项才会生效。此外，进行连接的用户必须是由`named_pipe_full_access_group`系统变量指定的 Windows 组的成员。

+   `--source-keyring=*`name`*`

    | 命令行格式 | `--source-keyring=name` |
    | --- | --- |
    | 类型 | 字符串 |

    用于密钥迁移的源密钥环组件。这是指定的组件库文件名，不带任何平台特定扩展名，如`.so`或`.dll`。例如，要使用库文件为`component_keyring_file.so`的组件，请将选项指定为`--source-keyring=component_keyring_file`。

    注意

    `--component-dir`、`--source-keyring`和`--destination-keyring`对于由**mysql_migrate_keyring**执行的所有密钥环迁移操作都是强制性的。此外，源组件和目标组件必须不同，并且两个组件必须正确配置，以便**mysql_migrate_keyring**可以加载和使用它们。

+   `--source-keyring-configuration-dir=*`dir_name`*`

    | 命令行格式 | `--source-keyring-configuration-dir=dir_name` |
    | --- | --- |
    | 类型 | 目录名称 |

    此选项仅在源密钥环组件全局配置文件包含`"read_local_config": true`时才生效，表示组件配置包含在本地配置文件中。选项值指定包含该本地文件的目录。

+   `--ssl*`

    以`--ssl`开头的选项指定是否使用加密连接到服务器，并指示 SSL 密钥和证书的位置。请参阅加密连接的命令选项。

+   `--ssl-fips-mode={OFF|ON|STRICT}`

    | 命令行格式 | `--ssl-fips-mode={OFF&#124;ON&#124;STRICT}` |
    | --- | --- |
    | 已弃用 | 8.0.34 |
    | 类型 | 枚举 |
    | 默认值 | `OFF` |
    | 有效值 | `OFF``ON``STRICT` |

    控制是否在客户端启用 FIPS 模式。`--ssl-fips-mode` 选项与其他 `--ssl-*`xxx`*` 选项不同，它不用于建立加密连接，而是用于影响允许的加密操作。请参见第 8.8 节，“FIPS 支持”。

    允许的`--ssl-fips-mode` 值为：

    +   `OFF`：禁用 FIPS 模式。

    +   `ON`：启用 FIPS 模式。

    +   `STRICT`：启用“严格” FIPS 模式。

    注意

    如果 OpenSSL FIPS 对象模块不可用，则`--ssl-fips-mode` 的唯一允许值是 `OFF`。在这种情况下，将`--ssl-fips-mode` 设置为 `ON` 或 `STRICT` 会导致客户端在启动时产生警告并在非 FIPS 模式下运行。

    从 MySQL 8.0.34 开始，此选项已弃用。预计在将来的 MySQL 版本中将其移除。

+   `--tls-ciphersuites=*`ciphersuite_list`*`

    | 命令行格式 | `--tls-ciphersuites=ciphersuite_list` |
    | --- | --- |
    | 类型 | 字符串 |

    用于使用 TLSv1.3 的加密连接的允许的密码套件。该值是一个或多个以冒号分隔的密码套件名称列表。可以为此选项命名的密码套件取决于用于编译 MySQL 的 SSL 库。有关详细信息，请参见第 8.3.2 节，“加密连接 TLS 协议和密码”。

+   `--tls-version=*`protocol_list`*`

    | 命令行格式 | `--tls-version=protocol_list` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `TLSv1,TLSv1.1,TLSv1.2,TLSv1.3`（OpenSSL 1.1.1 或更高版本）`TLSv1,TLSv1.1,TLSv1.2`（否则） |

    允许的加密连接的 TLS 协议。该值是一个或多个逗号分隔的协议名称列表。可以为此选项命名的协议取决于用于编译 MySQL 的 SSL 库。有关详细信息，请参见第 8.3.2 节，“加密连接 TLS 协议和密码”。

+   `--user=*`user_name`*`, `-u *`user_name`*`

    | 命令行格式 | `--user=user_name` |
    | --- | --- |
    | 类型 | 字符串 |

    用于连接到当前使用其中一个密钥迁移密钥库的运行服务器的 MySQL 帐户的用户名。

+   `--verbose`, `-v`

    | 命令行格式 | `--verbose` |
    | --- | --- |

    详细模式。输出关于程序操作的更多信息。

+   `--version`, `-V`

    | 命令行格式 | `--version` |
    | --- | --- |

    显示版本信息并退出。
