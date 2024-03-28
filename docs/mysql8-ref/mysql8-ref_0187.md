# 6.5.3 mysqlcheck — A Table Maintenance Program

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysqlcheck.html`](https://dev.mysql.com/doc/refman/8.0/en/mysqlcheck.html)

**mysqlcheck**客户端执行表维护：检查、修复、优化或分析表。

每个表在处理过程中被锁定，因此在处理过程中对其他会话不可用，尽管对于检查操作，表仅使用`READ`锁定（请参阅第 15.3.6 节“LOCK TABLES 和 UNLOCK TABLES 语句”，了解有关`READ`和`WRITE`锁定的更多信息）。表维护操作可能耗时，特别是对于大表。如果使用`--databases`或`--all-databases`选项来处理一个或多个数据库中的所有表，那么调用**mysqlcheck**可能需要很长时间。（如果 MySQL 升级过程确定需要进行表检查，因为它以相同方式处理表，这也适用。）

**mysqlcheck** 必须在**mysqld**服务器运行时使用，这意味着您无需停止服务器即可执行表维护。

**mysqlcheck** 使用 SQL 语句`CHECK TABLE`、`REPAIR TABLE`、`ANALYZE TABLE`和`OPTIMIZE TABLE`以一种方便用户的方式进行操作。它确定要执行的操作所需的语句，然后将这些语句发送到服务器执行。有关每个语句适用的存储引擎的详细信息，请参阅第 15.7.3 节“表维护语句”中这些语句的描述。

并非所有存储引擎都必然支持所有四种维护操作。在这种情况下，会显示错误消息。例如，如果`test.t`是一个`MEMORY`表，尝试检查它会产生这样的结果：

```sql
$> mysqlcheck test t
test.t
note     : The storage engine for the table doesn't support check
```

如果**mysqlcheck**无法修复表格，请参阅第 3.14 节，“重建或修复表格或索引”以获取手动表格修复策略。例如，对于`InnoDB`表格，可以使用`CHECK TABLE`进行检查，但无法使用`REPAIR TABLE`进行修复。

注意

在执行表格修复操作之前最好备份表格；在某些情况下，该操作可能导致数据丢失。可能的原因包括但不限于文件系统错误。

有三种常见的调用**mysqlcheck**的方法：

```sql
mysqlcheck [*options*] *db_name* [*tbl_name* ...]
mysqlcheck [*options*] --databases *db_name* ...
mysqlcheck [*options*] --all-databases
```

如果在*`db_name`*后没有命名任何表格，或者使用`--databases`或`--all-databases`选项，则会检查整个数据库。

与其他客户端程序相比，**mysqlcheck**具有特殊功能。检查表格的默认行为（`--check`）可以通过重命名二进制文件来更改。如果您希望默认情况下修复表格的工具，只需复制一个名为**mysqlrepair**的**mysqlcheck**副本，或者创建一个指向名为**mysqlrepair**的**mysqlcheck**的符号链接。如果调用**mysqlrepair**，它会修复表格。

下表中显示的名称可用于更改**mysqlcheck**的默认行为。

| 命令 | 含义 |
| --- | --- |
| **mysqlrepair** | 默认选项是`--repair` |
| **mysqlanalyze** | 默认选项是`--analyze` |
| **mysqloptimize** | 默认选项是`--optimize` |

**mysqlcheck**支持以下选项，可以在命令行或选项文件的`[mysqlcheck]`和`[client]`组中指定。有关 MySQL 程序使用的选项文件的信息，请参阅第 6.2.2.2 节，“使用选项文件”。

**表格 6.14 mysqlcheck 选项**

| 选项名称 | 描述 | 引入 | 废弃 |
| --- | --- | --- | --- |
| --all-databases | 检查所有数据库中的所有表 |  |  |
| --all-in-1 | 为每个命名了该数据库中所有表格的数据库执行单个语句 |  |  |
| --analyze | 分析表 |  |  |
| --auto-repair | 如果���查的表损坏，自动修复 |  |  |
| --bind-address | 使用指定的网络接口连接到 MySQL 服务器 |  |  |
| --character-sets-dir | 安装字符集的目录 |  |  |
| --check | 检查表中的错误 |  |  |
| --check-only-changed | 仅检查自上次检查以来发生变化的表 |  |  |
| --check-upgrade | 使用 FOR UPGRADE 选项调用 CHECK TABLE |  |  |
| --compress | 在客户端和服务器之间发送的所有信息进行压缩 |  | 8.0.18 |
| --compression-algorithms | 与服务器连接时允许的压缩算法 | 8.0.18 |  |
| --databases | 将所有参数解释为数据库名称 |  |  |
| --debug | 写调试日志 |  |  |
| --debug-check | 程序退出时打印调试信息 |  |  |
| --debug-info | 程序退出时打印调试信息、内存和 CPU 统计信息 |  |  |
| --default-auth | 要使用的认证插件 |  |  |
| --default-character-set | 指定默认字符集 |  |  |
| --defaults-extra-file | 除了通常的选项文件外，读取指定的选项文件 |  |  |
| --defaults-file | 仅读取指定的选项文件 |  |  |
| --defaults-group-suffix | 选项组后缀值 |  |  |
| --enable-cleartext-plugin | 启用明文认证插件 |  |  |
| --extended | 检查和修复表 |  |  |
| --fast | 仅检查未正确关闭的表 |  |  |
| --force | 即使发生 SQL 错误也继续 |  |  |
| --get-server-public-key | 从服务器请求 RSA 公钥 |  |  |
| --help | 显示帮助信息并退出 |  |  |
| --host | MySQL 服务器所在的主机 |  |  |
| --login-path | 从.mylogin.cnf 文件中读取登录路径选项 |  |  |
| --medium-check | 执行比--extended 操作更快的检查 |  |  |
| --no-defaults | 不读取任何选项文件 |  |  |
| --optimize | 优��表 |  |  |
| --password | 连接到服务器时使用的密码 |  |  |
| --password1 | 连接到服务器时使用的第一个多因素身份验证密码 | 8.0.27 |  |
| --password2 | 连接到服务器时使用的第二个多因素身份验证密码 | 8.0.27 |  |
| --password3 | 连接到服务器时使用的第三个多因素身份验证密码 | 8.0.27 |  |
| --pipe | 使用命名管道连接到服务器（仅限 Windows） |  |  |
| --plugin-dir | 安装插件的目录 |  |  |
| --port | 连接的 TCP/IP 端口号 |  |  |
| --print-defaults | 打印默认选项 |  |  |
| --protocol | 要使用的传输协议 |  |  |
| --quick | 检查的最快方法 |  |  |
| --repair | 执行修复操作，可以修复几乎所有问题，除了不唯一的唯一键 |  |  |
| --server-public-key-path | 包含 RSA 公钥的文件路径名 |  |  |
| --shared-memory-base-name | 共享内存连接的共享内存名称（仅限 Windows） |  |  |
| --silent | 静默模式 |  |  |
| --skip-database | 在执行操作时跳过此数据库 |  |  |
| --socket | 要使用的 Unix 套接字文件或 Windows 命名管道 |  |  |
| --ssl-ca | 包含受信任 SSL 证书颁发机构列表的文件 |  |  |
| --ssl-capath | 包含受信任 SSL 证书颁发机构证书文件的目录 |  |  |
| --ssl-cert | 包含 X.509 证书的文件 |  |  |
| --ssl-cipher | 连接加密的允许密码 |  |  |
| --ssl-crl | 包含证书吊销列表的文件 |  |  |
| --ssl-crlpath | 包含证书吊销列表文件的目录 |  |  |
| --ssl-fips-mode | 是否在客户端启用 FIPS 模式 |  | 8.0.34 |
| --ssl-key | 包含 X.509 密钥的文件 |  |  |
| --ssl-mode | 与服务器连接的期望安全状态 |  |  |
| --ssl-session-data | 包含 SSL 会话数据的文件 | 8.0.29 |  |
| --ssl-session-data-continue-on-failed-reuse | 如果会话重用失败是否建立连接 | 8.0.29 |  |
| --tables | 覆盖 --databases 或 -B 选项 |  |  |
| --tls-ciphersuites | 用于加密连接的允许的 TLSv1.3 密码套件 | 8.0.16 |  |
| --tls-version | 用于加密连接的允许的 TLS 协议 |  |  |
| --use-frm | 用于对 MyISAM 表进行修复操作 |  |  |
| --user | 连接到服务器时要使用的 MySQL 用户名 |  |  |
| --verbose | 详细模式 |  |  |
| --version | 显示版本信息并退出 |  |  |
| --write-binlog | 将 ANALYZE、OPTIMIZE、REPAIR 语句记录到二进制日志。--skip-write-binlog 将 NO_WRITE_TO_BINLOG 添加到这些语句中 |  |  |
| --zstd-compression-level | 用于使用 zstd 压缩连接到服务器的压缩级别 | 8.0.18 |  |
| 选项名称 | 描述 | 引入 | 废弃 |

+   `--help`, `-?`

    | 命令行格式 | `--help` |
    | --- | --- |

    显示帮助信息并退出。

+   `--all-databases`, `-A`

    | 命令行格式 | `--all-databases` |
    | --- | --- |

    检查所有数据库中的所有表。这与使用 `--databases` 选项并在命令行上命名所有数据库相同，只是不会检查 `INFORMATION_SCHEMA` 和 `performance_schema` 数据库。可以通过显式使用 `--databases` 选项来检查它们。

+   `--all-in-1`, `-1`

    | 命令行格式 | `--all-in-1` |
    | --- | --- |

    与为每个表发出语句不同，执行一个单独的语句，其中包含要处理的来自该数据库的所有表的名称。

+   `--analyze`, `-a`

    | 命令行格式 | `--analyze` |
    | --- | --- |

    分析表格。

+   `--auto-repair`

    | 命令行格式 | `--auto-repair` |
    | --- | --- |

    如果检查的表损坏，将自动修复。所有必要的修复将在所有表检查完成后进行。

+   `--bind-address=*`ip_address`*`

    | 命令行格式 | `--bind-address=ip_address` |
    | --- | --- |

    在具有多个网络接口的计算机上，使用此选项选择连接到 MySQL 服务器的接口。

+   `--character-sets-dir=*`dir_name`*`

    | 命令行格式 | `--character-sets-dir=dir_name` |
    | --- | --- |
    | 类型 | 目录名称 |

    安装字符集的目录。请参阅第 12.15 节，“字符集配置”。

+   `--check`, `-c`

    | 命令行格式 | `--check` |
    | --- | --- |

    检查表格是否有错误。这是默认操作。

+   `--check-only-changed`, `-C`

    | 命令行格式 | `--check-only-changed` |
    | --- | --- |

    仅检查自上次检查以来发生更改或未正确关闭的表。

+   `--check-upgrade`, `-g`

    | 命令行格式 | `--check-upgrade` |
    | --- | --- |

    使用`FOR UPGRADE`选项调用`CHECK TABLE`以检查表格与当前服务器版本的不兼容性。

+   `--compress`

    | 命令行格式 | `--compress[={OFF&#124;ON}]` |
    | --- | --- |
    | 弃用 | 8.0.18 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    尽可能压缩客户端和服务器之间发送的所有信息。请参阅第 6.2.8 节，“连接压缩控制”。

    从 MySQL 8.0.18 开始，此选项已被弃用。预计在未来的 MySQL 版本中将其移除。请参阅配置传统连接压缩。

+   `--compression-algorithms=*`value`*`

    | 命令行格式 | `--compression-algorithms=value` |
    | --- | --- |
    | 引入 | 8.0.18 |
    | 类型 | 集合 |
    | 默认值 | `uncompressed` |
    | 有效值 | `zlib``zstd``uncompressed` |

    连接到服务器的允许压缩算法。可用的算法与`protocol_compression_algorithms`系统变量相同。默认值为`uncompressed`。

    有关更多信息，请参阅第 6.2.8 节，“连接压缩控制”。

    此选项在 MySQL 8.0.18 中添加。

+   `--databases`, `-B`

    | 命令行格式 | `--databases` |
    | --- | --- |

    处理命名数据库中的所有表。通常，**mysqlcheck**将命令行上的第一个名称参数视为数据库名称，任何后续名称视为表名���。使用此选项，它将所有名称参数视为数据库名称。

+   [`--debug[=*`debug_options`*]`](mysqlcheck.html#option_mysqlcheck_debug), `-# [*`debug_options`*]`

    | 命令行格式 | `--debug[=debug_options]` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `d:t:o` |

    写入调试日志。典型的*`debug_options`*字符串是`d:t:o,*`file_name`*`。默认值为`d:t:o`。

    此选项仅在使用`WITH_DEBUG`构建 MySQL 时可用。由 Oracle 提供的 MySQL 发布二进制文件*不*使用此选项构建。

+   `--debug-check`

    | 命令行格式 | `--debug-check` |
    | --- | --- |
    | 类型 | 布尔值 |
    | 默认值 | `FALSE` |

    在程序退出时打印一些调试信息。

    此选项仅在使用`WITH_DEBUG`构建 MySQL 时可用。由 Oracle 提供的 MySQL 发布二进制文件*不*使用此选项构建。

+   `--debug-info`

    | 命令行格式 | `--debug-info` |
    | --- | --- |
    | 类型 | 布尔值 |
    | 默认值 | `FALSE` |

    在程序退出时打印调试信息以及内存和 CPU 使用统计信息。

    此选项仅在使用`WITH_DEBUG`构建 MySQL 时可用。由 Oracle 提供的 MySQL 发布二进制文件*不*使用此选项构建。

+   `--default-character-set=*`charset_name`*`

    | 命令行格式 | `--default-character-set=charset_name` |
    | --- | --- |
    | 类型 | 字符串 |

    使用*`charset_name`*作为默认字符集。参见第 12.15 节，“字符集配置”。

+   `--defaults-extra-file=*`file_name`*`

    | 命令行格式 | `--defaults-extra-file=file_name` |
    | --- | --- |
    | 类型 | 文件名 |

    在全局选项文件之后但（在 Unix 上）在用户选项文件之前读取此选项文件。如果文件不存在或无法访问，则会出现错误。如果*`file_name`*不是绝对路径名，则将其解释为相对于当前目录的路径。

    有关此选项和其他选项文件选项的更多信息，请参见第 6.2.2.3 节，“影响选项文件处理的命令行选项”。

+   `--defaults-file=*`file_name`*`

    | 命令行格式 | `--defaults-file=file_name` |
    | --- | --- |
    | 类型 | 文件名 |

    仅使用给定的选项文件。如果文件不存在或无法访问，将出现错误。如果*`file_name`*不是绝对路径名，则将其解释为相对于当前目录。

    例外：即使使用`--defaults-file`，客户端程序也会读取`.mylogin.cnf`。

    有关此选项和其他选项文件选项的其他信息，请参阅第 6.2.2.3 节，“影响选项文件处理的命令行选项”。

+   `--defaults-group-suffix=*`str`*`

    | 命令行格式 | `--defaults-group-suffix=str` |
    | --- | --- |
    | 类型 | 字符串 |

    不仅读取通常的选项组，还读取具有通常名称和后缀为*`str`*的组。例如，**mysqlcheck**通常会读取`[client]`和`[mysqlcheck]`组。如果此选项给定为`--defaults-group-suffix=_other`，**mysqlcheck**还会读取`[client_other]`和`[mysqlcheck_other]`组。

    有关此选项和其他选项文件选项的其他信息，请参阅第 6.2.2.3 节，“影响选项文件处理的命令行选项”。

+   `--extended`，`-e`

    | 命令行格式 | `--extended` |
    | --- | --- |

    如果您正在使用此选项来检查表格，它会确保它们是 100%一致的，但需要很长时间。

    如果您正在使用此选项来修复表格，它会运行一个可能不仅执行时间长，而且可能产生大量垃圾行的扩展修复！

+   `--default-auth=*`plugin`*`

    | 命令行格式 | `--default-auth=plugin` |
    | --- | --- |
    | 类型 | 字符串 |

    关于要使用的客户端端身份验证插件的提示。请参阅第 8.2.17 节，“可插拔认证”。

+   `--enable-cleartext-plugin`

    | 命令行格式 | `--enable-cleartext-plugin` |
    | --- | --- |
    | 类型 | 布尔值 |
    | 默认值 | `FALSE` |

    启用`mysql_clear_password`明文身份验证插件。（请参阅第 8.4.1.4 节，“客户端明文插拔式身份验证”。）

+   `--fast`，`-F`

    | 命令行格式 | `--fast` |
    | --- | --- |

    仅检查未正确关闭的表格。

+   `--force`，`-f`

    | 命令行格式 | `--force` |
    | --- | --- |

    即使发生 SQL 错误，也要继续。

+   `--get-server-public-key`

    | 命令行格式 | `--get-server-public-key` |
    | --- | --- |
    | 类型 | 布尔值 |

    从服务器请求用于 RSA 密钥对密码交换所需的公钥。此选项适用于使用`caching_sha2_password`身份验证插件进行身份验证的客户端。对于该插件，除非请求，否则服务器不会发送公钥。对于不使用该插件进行身份验证的帐户，此选项将被忽略。如果不使用基于 RSA 的密码交换，例如客户端使用安全连接连接到服务器时，此选项也将被忽略。

    如果给定`--server-public-key-path=*`file_name`*`并指定有效的公钥文件，则它优先于`--get-server-public-key`。

    有关`caching_sha2_password`插件的信息，请参见第 8.4.1.2 节，“Caching SHA-2 Pluggable Authentication”。

+   `--host=*`host_name`*`, `-h *`host_name`*`

    | 命令行格式 | `--host=host_name` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `localhost` |

    连接到给定主机上的 MySQL 服务器。

+   `--login-path=*`name`*`

    | 命令行格式 | `--login-path=name` |
    | --- | --- |
    | 类型 | 字符串 |

    从`.mylogin.cnf`登录路径文件中的指定登录路径中读取选项。 “登录路径”是一个包含指定要连接到哪个 MySQL 服务器以及要作为哪个帐户进行身份验证的选项的选项组。要创建或修改登录路径文件，请使用**mysql_config_editor**实用程序。请参见第 6.6.7 节，“mysql_config_editor — MySQL Configuration Utility”。

    有关此选项和其他选项文件选项的更多信息，请参见第 6.2.2.3 节，“影响选项文件处理的命令行选项”。

+   `--medium-check`, `-m`

    | 命令行格式 | `--medium-check` |
    | --- | --- |

    进行比`--extended`操作更快的检查。这只能找到所有错误的 99.99%，在大多数情况下应该足够好。

+   `--no-defaults`

    | 命令行格式 | `--no-defaults` |
    | --- | --- |

    不要读取任何选项文件。如果程序启动失败是因为从选项文件中读取了未知选项，可以使用`--no-defaults`来防止它们被读取。

    例外情况是，如果存在`.mylogin.cnf`文件，则在所有情况下都会读取该文件。这允许以比在命令行上更安全的方式指定密码，即使使用`--no-defaults`。要创建`.mylogin.cnf`，请使用**mysql_config_editor**实用程序。参见第 6.6.7 节，“mysql_config_editor — MySQL 配置实用程序”。

    有关此选项文件选项和其他选项的更多信息，请参见第 6.2.2.3 节，“影响选项文件处理的命令行选项”。

+   `--optimize`, `-o`

    | 命令行格式 | `--optimize` |
    | --- | --- |

    优化表格。

+   [`--password[=*`password`*]`](mysqlcheck.html#option_mysqlcheck_password), `-p[*`password`*]`

    | 命令行格式 | `--password[=password]` |
    | --- | --- |
    | 类型 | 字符串 |

    用于连接到服务器的 MySQL 帐户的密码。密码值是可选的。如果未提供，**mysqlcheck**会提示输入密码。如果提供了密码，则`--password=`或`-p`后面必须*没有空格*。如果未指定���码选项，则默认不发送密码。

    在命令行上指定密码应被视为不安全。为了避免在命令行上输入密码，请使用一个选项文件。参见第 8.1.2.1 节，“密码安全的最终用户指南”。

    要明确指定没有密码，并且**mysqlcheck**不应提示密码，请使用`--skip-password`选项。

+   [`--password1[=*`pass_val`*]`](mysqlcheck.html#option_mysqlcheck_password1)

    用于连接到服务器的 MySQL 帐户的多因素认证因子 1 的密码。密码值是可选的。如果未提供，**mysqlcheck**会提示输入密码。如果提供了密码，则`--password1=`后面必须*没有空格*。如果未指定密码选项，则默认不发送密码。

    在命令行上指定密码应被视为不安全。为了避免在命令行上输入密码，请使用一个选项文件。参见第 8.1.2.1 节，“密码安全的最终用户指南”。

    要明确指定没有密码，并且**mysqlcheck**不应提示输入密码，请使用`--skip-password1`选项。

    `--password1`和`--password`是同义词，`--skip-password1`和`--skip-password`也是同义词。

+   [`--password2[=*`pass_val`*]`](mysqlcheck.html#option_mysqlcheck_password2)

    用于连接到服务器的 MySQL 帐户的多因素认证因子 2 的密码。此选项的语义类似于`--password1`的语义；有关详细信息，请参阅该选项的描述。

+   [`--password3[=*`pass_val`*]`](mysqlcheck.html#option_mysqlcheck_password3)

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

    查找插件的目录。如果使用`--default-auth`选项指定身份验证插件但**mysqlcheck**找不到它，请指定此选项。请参阅第 8.2.17 节，“可插拔认证”。

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

    用于连接到服务器的传输协议。当其他连接参数通常导致使用不希望使用的协议时，这很有用。有关允许值的详细信息，请参见第 6.2.7 节，“连接传输协议”。

+   `--quick`, `-q`

    | 命令行格式 | `--quick` |
    | --- | --- |

    如果您使用此选项来检查表格，则可以防止检查扫描行以检查不正确的链接。这是最快的检查方法。

    如果您使用此选项来修复表格，则尝试仅修复索引树。这是最快的修复方法。

+   `--repair`, `-r`

    | 命令行格式 | `--repair` |
    | --- | --- |

    执行一个修复操作，可以修复几乎所有问题，除了不唯一的唯一键。

+   `--server-public-key-path=*`file_name`*`

    | 命令行格式 | `--server-public-key-path=file_name` |
    | --- | --- |
    | 类型 | 文件名 |

    以 PEM 格式的文件路径名，其中包含服务器所需的用于 RSA 密钥对密码交换的客户端端公钥的副本。此选项适用于使用`sha256_password`或`caching_sha2_password`身份验证插件进行身份验证的客户端。对于不使用这些插件进行身份验证的帐户，此选项将被忽略。如果不使用基于 RSA 的密码交换，例如客户端使用安全连接连接到服务器时，此选项也将被忽略。

    如果给定`--server-public-key-path=*`file_name`*`并指定有效的公钥文件，则它优先于`--get-server-public-key`。

    对于`sha256_password`，此选项仅在 MySQL 使用 OpenSSL 构建时适用。

    有关`sha256_password`和`caching_sha2_password`插件的信息，请参见第 8.4.1.3 节，“SHA-256 可插拔认证”和第 8.4.1.2 节，“缓存 SHA-2 可插拔认证”。

+   `--shared-memory-base-name=*`name`*`

    | 命令行格式 | `--shared-memory-base-name=name` |
    | --- | --- |
    | 特定平台 | Windows |

    在 Windows 上，用于通过共享内存连接到本地服务器的共享内存名称。默认值为 `MYSQL`。共享内存名称区分大小写。

    此选项仅在服务器启用了 `shared_memory` 系统变量以支持共享内存连接时适用。

+   `--silent`, `-s`

    | 命令行格式 | `--silent` |
    | --- | --- |

    静默模式。仅打印错误消息。

+   `--skip-database=*`db_name`*`

    | 命令行格式 | `--skip-database=db_name` |
    | --- | --- |

    在 **mysqlcheck** 执行的操作中不包括命名数据库（区分大小写）。

+   `--socket=*`path`*`, `-S *`path`*`

    | 命令行格式 | `--socket={file_name&#124;pipe_name}` |
    | --- | --- |
    | 类型 | 字符串 |

    对于连接到 `localhost` 的连接，要使用的 Unix 套接字文件，或者在 Windows 上要使用的命名管道的名称。

    在 Windows 上，此选项仅在服务器启用了 `named_pipe` 系统变量以支持命名管道连接时适用。此外，进行连接的用户必须是由 `named_pipe_full_access_group` 系统变量指定的 Windows 组的成员。

+   `--ssl*`

    以 `--ssl` 开头的选项指定是否使用加密连接到服务器，并指示 SSL 密钥和证书的位置。参见 加密连接的命令选项。

+   `--ssl-fips-mode={OFF|ON|STRICT}`

    | 命令行格式 | `--ssl-fips-mode={OFF&#124;ON&#124;STRICT}` |
    | --- | --- |
    | 已弃用 | 8.0.34 |
    | 类型 | 枚举 |
    | 默认值 | `OFF` |
    | 有效数值 | `OFF``ON``STRICT` |

    控制是否在客户端启用 FIPS 模式。`--ssl-fips-mode` 选项与其他 `--ssl-*`xxx`*` 选项不同，它不用于建立加密连接，而是影响允许哪些加密操作。参见 第 8.8 节，“FIPS 支持”。

    允许使用以下 `--ssl-fips-mode` 值：

    +   `OFF`: 禁用 FIPS 模式。

    +   `ON`: 启用 FIPS 模式。

    +   `STRICT`: 启用“严格” FIPS 模式。

    注意

    如果 OpenSSL FIPS Object 模块不可用，则 `--ssl-fips-mode` 的唯一允许值是 `OFF`。在这种情况下，将 `--ssl-fips-mode` 设置为 `ON` 或 `STRICT` 会导致客户端在启动时产生警告并在非 FIPS 模式下运行。

    截至 MySQL 8.0.34，此选项已被弃用。预计将在未来的 MySQL 版本中移除。

+   `--tables`

    | 命令行格式 | `--tables` |
    | --- | --- |

    覆盖`--databases`或`-B`选项。选项后面的所有名称参数都被视为表名。

+   `--tls-ciphersuites=*`ciphersuite_list`*`

    | 命令行格式 | `--tls-ciphersuites=ciphersuite_list` |
    | --- | --- |
    | 引入 | 8.0.16 |
    | 类型 | 字符串 |

    用于使用 TLSv1.3 的加密连接的可允许的密码套件。该值是一个或多个以冒号分隔的密码套件名称列表。可以为此选项命名的密码套件取决于用于编译 MySQL 的 SSL 库。有关详细信息，请参见第 8.3.2 节，“加密连接 TLS 协议和密码”。

    此选项是在 MySQL 8.0.16 中添加的。

+   `--tls-version=*`protocol_list`*`

    | 命令行格式 | `--tls-version=protocol_list` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值（≥ 8.0.16） | `TLSv1,TLSv1.1,TLSv1.2,TLSv1.3`（OpenSSL 1.1.1 或更高版本）`TLSv1,TLSv1.1,TLSv1.2`（否则） |
    | 默认值（≤ 8.0.15） | `TLSv1,TLSv1.1,TLSv1.2` |

    用于加密连接的可允许的 TLS 协议。该值是一个或多个逗号分隔的协议名称列表。可以为此选项命名的协议取决于用于编译 MySQL 的 SSL 库。有关详细信息，请参见第 8.3.2 节，“加密连接 TLS 协议和密码”。

+   `--use-frm`

    | 命令行格式 | `--use-frm` |
    | --- | --- |

    对于`MyISAM`表的修复操作，从数据字典获取表结构，以便即使`.MYI`头文件损坏，也可以修复表。

+   `--user=*`user_name`*`，`-u *`user_name`*`

    | 命令行格式 | `--user=user_name,` |
    | --- | --- |
    | 类型 | 字符串 |

    用于连接到服务器的 MySQL 帐户的用户名。

+   `--verbose`，`-v`

    | 命令行格式 | `--verbose` |
    | --- | --- |

    详细模式。打印有关程序操作各个阶段的信息。

+   `--version`，`-V`

    | 命令行格式 | `--version` |
    | --- | --- |

    显示版本信息并退出。

+   `--write-binlog`

    | 命令行格式 | `--write-binlog` |
    | --- | --- |

    默认情况下启用此选项，因此由 **mysqlcheck** 生成的 `ANALYZE TABLE`、`OPTIMIZE TABLE` 和 `REPAIR TABLE` 语句将被写入二进制日志。使用 `--skip-write-binlog` 可以添加 `NO_WRITE_TO_BINLOG` 到语句中，以便不记录这些语句。当这些语句不应发送到副本或在使用二进制日志从备份中恢复时运行时，请使用 `--skip-write-binlog`。

+   `--zstd-compression-level=*`level`*`

    | 命令行格式 | `--zstd-compression-level=#` |
    | --- | --- |
    | 引入版本 | 8.0.18 |
    | 类型 | 整数 |

    用于使用 `zstd` 压缩算法连接到服务器的压缩级别。允许的级别从 1 到 22，较大的值表示较高级别的压缩。默认的 `zstd` 压缩级别为 3。压缩级别设置对不使用 `zstd` 压缩的连接没有影响。

    有关更多信息，请参见 第 6.2.8 节，“连接压缩控制”。

    此选项是在 MySQL 8.0.18 中添加的。
