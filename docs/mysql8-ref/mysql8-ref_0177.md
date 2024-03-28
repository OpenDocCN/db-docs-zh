# 6.4.5 mysql_upgrade — 检查和升级 MySQL 表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-upgrade.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-upgrade.html)

注意

从 MySQL 8.0.16 开始，MySQL 服务器执行以前由 **mysql_upgrade** 处理的升级任务（有关详细信息，请参见 Section 3.4, “What the MySQL Upgrade Process Upgrades”）。因此，**mysql_upgrade** 不再需要，并且在该版本中已被弃用；预计在将来的 MySQL 版本中将其移除。由于 **mysql_upgrade** 不再执行升级任务，它无条件地以状态 0 退出。

每次升级 MySQL 时，您应该执行 **mysql_upgrade**，它会查找与升级后的 MySQL 服务器不兼容的地方：

+   它升级 `mysql` 模式中的系统表，以便您可以利用可能已添加的新权限或功能。

+   它升级了性能模式、`INFORMATION_SCHEMA` 和 `sys` 模式。

+   它检查用户模式。

如果 **mysql_upgrade** 发现表存在可能的不兼容性，它将执行表检查，并在发现问题时尝试进行表修复。如果表无法修复，请参见 Section 3.14, “Rebuilding or Repairing Tables or Indexes” 了解手动表修复策略。

**mysql_upgrade** 直接与 MySQL 服务器通信，发送执行升级所需的 SQL 语句。

警告

在执行升级之前，您应始终备份当前的 MySQL 安装。请参见 Section 9.2, “Database Backup Methods”。

一些升级不兼容性可能需要在升级 MySQL 安装和运行 **mysql_upgrade** 之前进行特殊处理。请参见 Chapter 3, *Upgrading MySQL*，了解如何确定是否有任何这样的不兼容性适用于您的安装以及如何处理它们。

使用 **mysql_upgrade** 如下：

1.  确保服务器正在运行。

1.  调用 **mysql_upgrade** 来升级 `mysql` 模式中的系统表，并检查和修复其他模式中的表：

    ```sql
    mysql_upgrade [*options*]
    ```

1.  停止服务器并重新启动，以使任何系统表更改生效。

如果有多个要升级的 MySQL 服务器实例，请使用适合连接到每个所需服务器的连接参数调用 **mysql_upgrade**。例如，对于在本地主机上运行在端口 3306 到 3308 上的服务器，请通过连接到相应端口来升级它们：

```sql
mysql_upgrade --protocol=tcp -P 3306 [*other_options*]
mysql_upgrade --protocol=tcp -P 3307 [*other_options*]
mysql_upgrade --protocol=tcp -P 3308 [*other_options*]
```

对于 Unix 上的本地主机连接，`--protocol=tcp` 选项强制使用 TCP/IP 进行连接，而不是使用 Unix 套接字文件。

默认情况下，**mysql_upgrade** 以 MySQL `root` 用户身份运行。如果在运行 **mysql_upgrade** 时 `root` 密码已过期，则会显示密码已过期的消息，并且 **mysql_upgrade** 失败。要纠正这个问题，重置 `root` 密码以取消过期，并再次运行 **mysql_upgrade**。首先，以 `root` 身份连接到服务器：

```sql
$> mysql -u root -p
Enter password: ****  <- enter root password here
```

使用 `ALTER USER` 重置密码：

```sql
mysql> ALTER USER USER() IDENTIFIED BY '*root-password*';
```

然后退出 **mysql** 并再次运行 **mysql_upgrade**：

```sql
$> mysql_upgrade [*options*]
```

注意

如果使用 `disabled_storage_engines` 系统变量运行服务器以禁用某些存储引擎（例如 `MyISAM`），**mysql_upgrade** 可能会因错误而失败，如下所示：

```sql
mysql_upgrade: [ERROR] 3161: Storage engine MyISAM is disabled
(Table creation is disallowed).
```

为了处理这个问题，重新启动服务器以禁用 `disabled_storage_engines`。然后你应该能够成功运行 **mysql_upgrade**。之后，重新启动服务器并将 `disabled_storage_engines` 设置为其原始值。

除非使用 `--upgrade-system-tables` 选项调用，**mysql_upgrade** 将根据需要处理所有用户模式中的所有表。表检查可能需要很长时间才能完成。每个表都会被锁定，因此在处理过程中对其他会话不可用。检查和修复操作可能需要很长时间，特别是对于大表。表检查使用 `CHECK TABLE` 语句的 `FOR UPGRADE` 选项。有关此选项的详细信息，请参见 第 15.7.3.2 节，“CHECK TABLE 语句”。

**mysql_upgrade** 使用当前 MySQL 版本号标记所有已检查和修复的表。这确保下次使用相同服务器版本运行 **mysql_upgrade** 时，可以确定是否有必要再次检查或修复给定表。

**mysql_upgrade** 将 MySQL 版本号保存在名为 `mysql_upgrade_info` 的文件中，该文件位于数据目录中。这用于快速检查是否已为此版本检查了所有表，以便可以跳过表检查。要忽略此文件并执行检查，请使用 `--force` 选项。

注意

`mysql_upgrade_info` 文件已被弃用；预计在未来的 MySQL 版本中将被移除。

**mysql_upgrade** 检查 `mysql.user` 系统表行，并对任何具有空 `plugin` 列的行，如果凭据使用与该插件兼容的哈希格式，则将该列设置为 `'mysql_native_password'`。必须手动升级具有早期 4.1 版本密码哈希的行。

**mysql_upgrade** 不会升级时区表或帮助表的内容。有关升级说明，请参见 第 7.1.15 节，“MySQL 服务器时区支持”，以及 第 7.1.17 节，“服务器端帮助支持”。

除非使用 `--skip-sys-schema` 选项调用，否则 **mysql_upgrade** 如果尚未安装 `sys` 模式，则安装它，并在其他情况下将其升级到当前版本。如果存在 `sys` 模式但没有 `version` 视图，则会发生错误，假设其不存在表示用户创建的模式：

```sql
A sys schema exists with no sys.version view. If
you have a user created sys schema, this must be renamed for the
upgrade to succeed.
```

在这种情况下升级，首先删除或重命名现有的`sys`模式。

**mysql_upgrade** 支持以下选项，可以在命令行或选项文件的`[mysql_upgrade]`和`[client]`组中指定。有关 MySQL 程序使用的选项文件的信息，请参见 Section 6.2.2.2, “使用选项文件”。

**表 6.11 mysql_upgrade 选项**

| 选项名称 | 描述 | 引入 | 废弃 |
| --- | --- | --- | --- |
| --bind-address | 使用指定的网络接口连接到 MySQL 服务器 |  |  |
| --character-sets-dir | 安装字符集的目录 |  |  |
| --compress | 压缩客户端和服务器之间发送的所有信息 |  | 8.0.18 |
| --compression-algorithms | 连接到服务器的允许压缩算法 | 8.0.18 |  |
| --debug | 写入调试日志 |  |  |
| --debug-check | 程序退出时打印调试信息 |  |  |
| --debug-info | 程序退出时打印调试信息、内存和 CPU 统计信息 |  |  |
| --default-auth | 要使用的认证插件 |  |  |
| --default-character-set | 指定默认字符集 |  |  |
| --defaults-extra-file | 除了通常的选项文件外，还读取指定的选项文件 |  |  |
| --defaults-file | 仅读取指定的选项文件 |  |  |
| --defaults-group-suffix | 选项组后缀值 |  |  |
| --force | 即使 mysql_upgrade 已经为当前 MySQL 版本执行过，也要强制执行 |  |  |
| --get-server-public-key | 从服务器请求 RSA 公钥 |  |  |
| --help | 显示帮助消息并退出 |  |  |
| --host | MySQL 服务器所在的主机 |  |  |
| --login-path | 从.mylogin.cnf 中读取登录路径选项 |  |  |
| --max-allowed-packet | 发送到服务器或从服务器接收的最大数据包长度 |  |  |
| --net-buffer-length | TCP/IP 和套接字通信的缓冲区大小 |  |  |
| --no-defaults | 不读取任何选项文件 |  |  |
| --password | 连接到服务��时要使用的密码 |  |  |
| --pipe | 使用命名管道连接服务器（仅限 Windows） |  |  |
| --plugin-dir | 安装插件的目录 |  |  |
| --port | 连接的 TCP/IP 端口号 |  |  |
| --print-defaults | 打印默认选项 |  |  |
| --protocol | 要使用的传输协议 |  |  |
| --server-public-key-path | 包含 RSA 公钥的文件路径名 |  |  |
| --shared-memory-base-name | 共享内存连接的共享内存名称（仅限 Windows） |  |  |
| --skip-sys-schema | 不安装或升级 sys schema |  |  |
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
| --tls-ciphersuites | 用于加密连接的允许的 TLSv1.3 密码套件 | 8.0.16 |  |
| --tls-version | 用于加密连接的允许的 TLS 协议 |  |  |
| --upgrade-system-tables | 仅更新系统表，不更新用户模式 |  |  |
| --user | 连接到服务器时要使用的 MySQL 用户名 |  |  |
| --verbose | 详细模式 |  |  |
| --version-check | 检查正确的服务器版本 |  |  |
| --write-binlog | 将所有语句写入二进制日志 |  |  |
| --zstd-compression-level | 用于使用 zstd 压缩连接的服务器的压缩级别 | 8.0.18 |  |
| 选项名称 | 描述 | 引入 | 废弃 |

+   `--help`

    | 命令行格式 | `--help` |
    | --- | --- |

    显示简短的帮助消息并退出。

+   `--bind-address=*`ip_address`*`

    | 命令行格式 | `--bind-address=ip_address` |
    | --- | --- |

    在具有多个网络接口的计算机上，使用此选项选择连接到 MySQL 服务器的接口。

+   `--character-sets-dir=*`dir_name`*`

    | 命令行格式 | `--character-sets-dir=dir_name` |
    | --- | --- |
    | 类型 | 目录名称 |

    字符集安装目录。参见第 12.15 节，“字符集配置”。

+   `--compress`, `-C`

    | 命令行格式 | `--compress[={OFF&#124;ON}]` |
    | --- | --- |
    | 废弃 | 8.0.18 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    尽可能压缩客户端和服务器之间发送的所有信息。参见第 6.2.8 节，“连接压缩控制”。

    从 MySQL 8.0.18 开始，此选项已被弃用。预计在将来的 MySQL 版本中将其移除。参见配置传统连接压缩。

+   `--compression-algorithms=*`value`*`

    | 命令行格式 | `--compression-algorithms=value` |
    | --- | --- |
    | 引入 | 8.0.18 |
    | 类型 | 集合 |
    | 默认值 | `uncompressed` |
    | 有效值 | `zlib``zstd``uncompressed` |

    连接到服务器的允许的压缩算法。可用的算法与`protocol_compression_algorithms`系统变量相同。默认值为`uncompressed`。

    更多信息，请参见第 6.2.8 节，“连接压缩控制”。

    此选项在 MySQL 8.0.18 中添加。

+   [`--debug[=*`debug_options`*]`](mysql-upgrade.html#option_mysql_upgrade_debug), `-# [*`debug_options`*]`

    | 命令行格式 | `--debug[=#]` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `d:t:O,/tmp/mysql_upgrade.trace` |

    写入调试日志。典型的*`debug_options`*字符串是`d:t:o,*`file_name`*`。默认值为`d:t:O,/tmp/mysql_upgrade.trace`。

+   `--debug-check`

    | 命令行格式 | `--debug-check` |
    | --- | --- |
    | 类型 | 布尔值 |

    程序退出时打印一些调试信息。

+   `--debug-info`, `-T`

    | 命令行格式 | `--debug-info` |
    | --- | --- |
    | 类型 | 布尔值 |
    | 默认值 | `FALSE` |

    在程序退出时打印调试信息以及内存和 CPU 使用统计信息。

+   `--default-auth=*`plugin`*`

    | 命令行格式 | `--default-auth=plugin` |
    | --- | --- |
    | 类型 | 字符串 |

    提示使用哪个客户端身份验证插件。参见第 8.2.17 节，“可插拔身份验证”。

+   `--default-character-set=*`charset_name`*`

    | 命令行格式 | `--default-character-set=name` |
    | --- | --- |
    | 类型 | 字符串 |

    使用*`charset_name`*作为默认字符集。参见第 12.15 节，“字符集配置”。

+   `--defaults-extra-file=*`file_name`*`

    | 命令行格式 | `--defaults-extra-file=file_name` |
    | --- | --- |
    | 类型 | 文件名 |

    在全局选项文件之后（在 Unix 上）但在用户选项文件之前读取此选项文件。如果文件不存在或无法访问，将出现错误。如果*`file_name`*不是绝对路径名，则将其解释为相对于当前目录。

    关于此选项文件和其他选项文件选项的更多信息，请参见第 6.2.2.3 节，“影响选项文件处理的命令行选项”。

+   `--defaults-file=*`file_name`*`

    | 命令行格式 | `--defaults-file=file_name` |
    | --- | --- |
    | 类型 | 文件名 |

    仅使用给定的选项文件。如果文件不存在或无法访问，将出现错误。如果*`file_name`*不是绝对路径名，则将其解释为相对于当前目录。

    有关此选项文件选项和其他选项的更多信息，请参见第 6.2.2.3 节，“影响选项文件处理的命令行选项”。

+   `--defaults-group-suffix=*`str`*`

    | 命令行格式 | `--defaults-group-suffix=str` |
    | --- | --- |
    | 类型 | 字符串 |

    读取不仅是通常的选项组，还有带有通常名称和后缀为*`str`*的组。例如，**mysql_upgrade**通常会读取`[client]`和`[mysql_upgrade]`组。如果此选项给定为`--defaults-group-suffix=_other`，**mysql_upgrade**还会读取`[client_other]`和`[mysql_upgrade_other]`组。

    有关此选项文件选项和其他选项的更多信息，请参见第 6.2.2.3 节，“影响选项文件处理的命令行选项”。

+   `--force`

    | 命令行格式 | `--force` |
    | --- | --- |
    | 类型 | 布尔值 |

    忽略`mysql_upgrade_info`文件，并强制执行，即使**mysql_upgrade**已经为当前版本的 MySQL 执行过。

+   `--get-server-public-key`

    | 命令行格式 | `--get-server-public-key` |
    | --- | --- |
    | 类型 | 布尔值 |

    从服务器请求用于 RSA 密钥对密码交换所需的公钥。此选项适用于使用`caching_sha2_password`认证插件进行身份验证的客户端。对于该插件，除非请求，否则服务器不会发送公钥。对于不使用该插件进行身份验证的帐户，此选项将被忽略。如果不使用基于 RSA 的密码交换，例如客户端使用安全连接连接到服务器时，此选项也将被忽略。

    如果`--server-public-key-path=*`file_name`*`被给定并指定了有效的公钥文件，则优先于`--get-server-public-key`。

    有关`caching_sha2_password`插件的信息，请参见第 8.4.1.2 节，“Caching SHA-2 Pluggable Authentication”。

+   `--host=*`host_name`*`, `-h *`host_name`*`

    | 命令行格式 | `--host=name` |
    | --- | --- |
    | 类型 | 字符串 |

    连接到给定主机上的 MySQL 服务器。

+   `--login-path=*`name`*`

    | 命令行格式 | `--login-path=name` |
    | --- | --- |
    | 类型 | 字符串 |

    从`.mylogin.cnf`登录路径文件中的指定登录路径读取选项。 “登录路径”是一个包含指定要连接到哪个 MySQL 服务器以及要作为哪个帐户进行身份验证的选项的选项组。要创建或修改登录路径文件，请使用**mysql_config_editor**实用程序。请参阅 Section 6.6.7, “mysql_config_editor — MySQL Configuration Utility”。

    有关此选项和其他选项文件选项的其他信息，请参阅 Section 6.2.2.3, “Command-Line Options that Affect Option-File Handling”。

+   `--max-allowed-packet=*`value`*`

    | 命令行格式 | `--max-allowed-packet=value` |
    | --- | --- |
    | 类型 | 整数 |
    | 默认值 | `25165824` |
    | 最小值 | `4096` |
    | 最大值 | `2147483648` |

    客户端/服务器通信缓冲区的最大大小。默认值为 24MB。最小值和最大值分别为 4KB 和 2GB。

+   `--net-buffer-length=*`value`*`

    | 命令行格式 | `--net-buffer-length=value` |
    | --- | --- |
    | 类型 | 整数 |
    | 默认值 | `1047552` |
    | 最小值 | `4096` |
    | 最大值 | `16777216` |

    客户端/服务器通信缓冲区的初始大小。默认值为 1MB − 1KB。最小值和最大值分别为 4KB 和 16MB。

+   `--no-defaults`

    | 命令行格式 | `--no-defaults` |
    | --- | --- |

    不要读取任何选项文件。如果由于从选项文件中读取未知选项而导致程序启动失败，则可以使用`--no-defaults`来防止读取它们。

    例外情况是，如果存在`.mylogin.cnf`文件，则在所有情况下都会读取该文件。即使使用`--no-defaults`，也可以以比在命令行上更安全的方式指定密码。要创建`.mylogin.cnf`，请使用**mysql_config_editor**实用程序。请参阅 Section 6.6.7, “mysql_config_editor — MySQL Configuration Utility”。

    有关此选项和其他选项文件选项的其他信息，请参阅 Section 6.2.2.3, “Command-Line Options that Affect Option-File Handling”。

+   [`--password[=*`password`*]`](mysql-upgrade.html#option_mysql_upgrade_password), `-p[*`password`*]`

    | 命令行格式 | `--password[=name]` |
    | --- | --- |
    | 类型 | 字符串 |

    用于连接到服务器的 MySQL 帐户的密码。密码值是可选的。如果未提供，**mysql_upgrade**会提示输入密码。如果提供，`--password=`或`-p`与后面的密码之间*不能有空格*。如果未指定密码选项，则默认不发送密码。

    在命令行上指定密码应被视为不安全。为了避免在命令行上提供密码，请使用选项文件。参见第 8.1.2.1 节，“密码安全的最终用户指南”。

    明确指定没有密码，并且**mysql_upgrade**不应提示输入密码，使用`--skip-password`选项。

+   `--pipe`, `-W`

    | 命令行格式 | `--pipe` |
    | --- | --- |
    | 类型 | 字符串 |

    在 Windows 上，使用命名管道连接服务器。此选项仅在服务器启动时启用了`named_pipe`系统变量以支持命名管道连接时适用。此外，进行连接的用户必须是由`named_pipe_full_access_group`系统变量指定的 Windows 组的成员。

+   `--plugin-dir=*`dir_name`*`

    | 命令行格式 | `--plugin-dir=dir_name` |
    | --- | --- |
    | 类型 | 目录名称 |

    查找插件的目录。如果使用`--default-auth`选项指定身份验证插件但**mysql_upgrade**找不到它，则指定此选项。参见第 8.2.17 节，“可插拔认证”。

+   `--port=*`port_num`*`, `-P *`port_num`*`

    | 命令行格式 | `--port=#` |
    | --- | --- |
    | 类型 | 数字 |

    用于 TCP/IP 连接的端口号。

+   `--print-defaults`

    | 命令行格式 | `--print-defaults` |
    | --- | --- |

    打印程序名称以及从选项文件获取的所有选项。

+   `--protocol={TCP|SOCKET|PIPE|MEMORY}`

    | 命令行格式 | `--protocol=name` |
    | --- | --- |
    | 类型 | 字符串 |

    用于连接到服务器的传输协议。当其他连接参数通常导致使用不希望使用的协议时，这很有用。有关允许值的详细信息，请参见第 6.2.7 节，“连接传输协议”。

+   `--server-public-key-path=*`file_name`*`

    | 命令行格式 | `--server-public-key-path=file_name` |
    | --- | --- |
    | 类型 | 文件名 |

    以 PEM 格式的文件路径名，其中包含服务器所需的用于 RSA 密钥对密码交换的客户端端公钥的副本。此选项适用于使用`sha256_password`或`caching_sha2_password`认证插件进行身份验证的客户端。对于不使用这些插件进行身份验证的帐户，此选项将被忽略。如果不使用基于 RSA 的密码交换（例如客户端使用安全连接连接到服务器时），此选项也将被忽略。

    如果`--server-public-key-path=*`file_name`*`被指定并指定了有效的公钥文件，则优先于`--get-server-public-key`。

    对于`sha256_password`，此选项仅在 MySQL 使用 OpenSSL 构建时适用。

    有关`sha256_password`和`caching_sha2_password`插件的信息，请参见第 8.4.1.3 节，“SHA-256 可插拔认证”和第 8.4.1.2 节，“缓存 SHA-2 可插拔认证”。

+   `--shared-memory-base-name=*`name`*`

    | 命令行格式 | `--shared-memory-base-name=name` |
    | --- | --- |
    | 平台特定 | Windows |

    在 Windows 上，用于通过共享内存连接到本地服务器的共享内存名称。默认值为`MYSQL`。共享内存名称区分大小写。

    仅当服务器启用了`shared_memory`系统变量以支持共享内存连��时，此选项才适用。

+   `--skip-sys-schema`

    | 命令行格式 | `--skip-sys-schema` |
    | --- | --- |
    | 类型 | 布尔值 |
    | 默认值 | `FALSE` |

    默认情况下，**mysql_upgrade**如果未安装`sys`模式，则安装它，并将其升级到当前版本。`--skip-sys-schema`选项会抑制此行为。

+   `--socket=*`path`*`, `-S *`path`*`

    | 命令行格式 | `--socket={file_name&#124;pipe_name}` |
    | --- | --- |
    | 类型 | 字符串 |

    对于连接到`localhost`的连接，要使用的 Unix 套接字文件，或者在 Windows 上，要使用的命名管道的名称。

    在 Windows 上，只有在服务器启动时启用了`named_pipe`系统变量以支持命名管道连接时，此选项才适用。此外，进行连接的用户必须是由`named_pipe_full_access_group`系统变量指定的 Windows 组的成员。

+   `--ssl*`

    以`--ssl`开头的选项指定是否使用加密连接连接到服务器，并指示 SSL 密钥和证书的位置。请参见加密连接的命令选项。

+   `--ssl-fips-mode={OFF|ON|STRICT}`

    | 命令行格式 | `--ssl-fips-mode={OFF&#124;ON&#124;STRICT}` |
    | --- | --- |
    | 已弃用 | 8.0.34 |
    | 类型 | 枚举 |
    | 默认值 | `OFF` |
    | 有效值 | `OFF``ON``STRICT` |

    控制是否在客户端端启用 FIPS 模式。`--ssl-fips-mode`选项与其他`--ssl-*`xxx`*`选项不同，它不用于建立加密连接，而是影响允许哪些加密操作。请参见第 8.8 节，“FIPS 支持”。

    允许使用这些`--ssl-fips-mode`值：

    +   `OFF`：禁用 FIPS 模式。

    +   `ON`：启用 FIPS 模式。

    +   `STRICT`：启用“严格” FIPS 模式。

    注意

    如果 OpenSSL FIPS Object 模块不可用，则`--ssl-fips-mode`的唯一允许值是`OFF`。在这种情况下，将`--ssl-fips-mode`设置为`ON`或`STRICT`会导致客户端在启动时产生警告并在非 FIPS 模式下运行。

    从 MySQL 8.0.34 开始，此选项已弃用。预计在将来的 MySQL 版本中将其移除。

+   `--tls-ciphersuites=*`ciphersuite_list`*`

    | 命令行格式 | `--tls-ciphersuites=ciphersuite_list` |
    | --- | --- |
    | 引入版本 | 8.0.16 |
    | 类型 | 字符串 |

    用于使用 TLSv1.3 的加密连接的允许密码套件。该值是一个或多个以冒号分隔的密码套件名称列表。可以为此选项命名的密码套件取决于用于编译 MySQL 的 SSL 库。有关详细信息，请参见第 8.3.2 节，“加密连接 TLS 协议和密码”。

    此选项是在 MySQL 8.0.16 中添加的。

+   `--tls-version=*`protocol_list`*`

    | 命令行格式 | `--tls-version=protocol_list` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 (≥ 8.0.16) | `TLSv1,TLSv1.1,TLSv1.2,TLSv1.3` (OpenSSL 1.1.1 或更高版本)`TLSv1,TLSv1.1,TLSv1.2` (否则) |
    | 默认值 (≤ 8.0.15) | `TLSv1,TLSv1.1,TLSv1.2` |

    加密连接的可接受 TLS 协议。该值是一个或多个逗号分隔的协议名称列表。可以为此选项命名的协议取决于用于编译 MySQL 的 SSL 库。有关详细信息，请参见 Section 8.3.2, “Encrypted Connection TLS Protocols and Ciphers”。

+   `--upgrade-system-tables`, `-s`

    | 命令行格式 | `--upgrade-system-tables` |
    | --- | --- |
    | 类型 | 布尔值 |

    仅升级 `mysql` 模式中的系统表，不升级用户模式。

+   `--user=*`user_name`*`, `-u *`user_name`*`

    | 命令行格式 | `--user=name` |
    | --- | --- |
    | 类型 | 字符串 |

    用于连接到服务器的 MySQL 帐户的用户名。默认用户名为 `root`。

+   `--verbose`

    | 命令行格式 | `--verbose` |
    | --- | --- |
    | 类型 | 布尔值 |

    详细模式。打印有关程序操作的更多信息。

+   `--version-check`, `-k`

    | 命令行格式 | `--version-check` |
    | --- | --- |
    | 类型 | 布尔值 |

    检查 **mysql_upgrade** 连接的服务器版本，以验证其与构建 **mysql_upgrade** 的版本相同。如果不同，**mysql_upgrade** 将退出。此选项默认启用；要禁用检查，请使用 `--skip-version-check`。

+   `--write-binlog`

    | 命令行格式 | `--write-binlog` |
    | --- | --- |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    默认情况下，**mysql_upgrade** 的二进制日志记录是禁用的。如果希望将其操作写入二进制日志，请使用 `--write-binlog`。

    当服务器启用全局事务标识符（GTIDs）时（`gtid_mode=ON`），不要通过 **mysql_upgrade** 启用二进制日志记录。

+   `--zstd-compression-level=*`level`*`

    | 命令行格式 | `--zstd-compression-level=#` |
    | --- | --- |
    | 引入版本 | 8.0.18 |
    | 类型 | 整数 |

    用于使用`zstd`压缩算法连接到服务器的压缩级别。允许的级别从 1 到 22，较大的值表示较高级别的压缩。默认的`zstd`压缩级别为 3。压缩级别设置对不使用`zstd`压缩的连接没有影响。

    有关更多信息，请参见第 6.2.8 节，“连接压缩控制”。

    此选项是在 MySQL 8.0.18 中添加的。
