# 6.5.2 mysqladmin — A MySQL Server Administration Program

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysqladmin.html`](https://dev.mysql.com/doc/refman/8.0/en/mysqladmin.html)

**mysqladmin** 是用于执行管理操作的客户端。您可以使用它来检查服务器的配置和当前状态，创建和删除数据库等。

像这样调用**mysqladmin**：

```sql
mysqladmin [*options*] *command* [*command-arg*] [*command* [*command-arg*]] ...
```

**mysqladmin** 支持以下命令。一些命令在命令名称后面需要一个参数。

+   `create *`db_name`*`

    创建一个名为*`db_name`*的新数据库。

+   `debug`

    在 MySQL 8.0.20 之前，告诉服务器将调试信息写入错误日志。连接的用户必须具有`SUPER`权限。此信息的格式和内容可能会发生变化。

    这包括有关事件调度程序的信息。参见 Section 27.4.5, “Event Scheduler Status”。

+   `drop *`db_name`*`

    删除名为*`db_name`*的数据库及其所有表。

+   `extended-status`

    显示服务器状态变量及其值。

+   `flush-hosts`

    刷新主机缓存中的所有信息。参见 Section 7.1.12.3, “DNS Lookups and the Host Cache”。

+   `flush-logs [*`log_type`* ...]`

    刷新所有日志。

    **mysqladmin flush-logs** 命令允许提供可选的日志类型，以指定要刷新的日志。在`flush-logs`命令之后，您可以提供一个空格分隔的日志类型列表，其中包括以下一个或多个日志类型：`binary`、`engine`、`error`、`general`、`relay`、`slow`。这些对应于可以为`FLUSH LOGS` SQL 语句指定的日志类型。

+   `flush-privileges`

    重新加载授权表（与`reload`相同）。

+   `flush-status`

    清除状态变量。

+   `flush-tables`

    刷新所有表。

+   `flush-threads`

    刷新线程缓存。

+   `kill *`id`*,*`id`*,...`

    终止服务器线程。如果给出多个线程 ID 值，则列表中不能有空格。

    要终止属于其他用户的线程，连接的用户必须具有`CONNECTION_ADMIN`权限（或已弃用的`SUPER`权限）。

+   `password *`new_password`*`

    设置一个新密码。这将为您在使用**mysqladmin**连接到服务器的帐户更改密码为*`new_password`*。因此，下次您使用相同帐户调用**mysqladmin**（或任何其他客户端程序）时，您必须指定新密码。

    警告

    使用**mysqladmin**设置密码应被视为*不安全*。在某些系统上，您的密码会对系统状态程序（如**ps**）可见，其他用户可能调用这些程序来显示命令行。MySQL 客户端通常在初始化序列期间用零覆盖命令行密码参数。但是，在这个值可见的瞬间仍然存在。此外，在某些系统上，这种覆盖策略是无效的，密码仍然对**ps**可见。（SystemV Unix 系统和其他系统可能存在此问题。）

    如果*`new_password`*值包含空格或其他对您的命令解释器特殊的字符，则需要将其放在引号内。在 Windows 上，请确保使用双引号而不是单引号；单引号不会从密码中删除，而是被解释为密码的一部分。例如：

    ```sql
    mysqladmin password "my new password"
    ```

    在`password`命令后可以省略新密码。在这种情况下，**mysqladmin**会提示输入密码值，这样您就可以避免在命令行中指定密码。只有在`password`是**mysqladmin**命令行上的最后一个命令时才应省略密码值。否则，下一个参数将被视为密码。

    注意

    如果服务器是使用`--skip-grant-tables`选项启动的，请勿使用此命令。密码更改不会生效。即使您在`password`命令之前在同一命令行上使用`flush-privileges`重新启用授权表，因为刷新操作发生在连接之后。但是，您可以使用**mysqladmin flush-privileges**重新启用授权表，然后使用单独的**mysqladmin password**命令更改密码。

+   `ping`

    检查服务器是否可用。从**mysqladmin**返回的状态为 0 表示服务器正在运行，为 1 表示未运行。即使出现`拒绝访问`等错误，返回状态也为 0，因为这意味着服务器正在运行但拒绝连接，这与服务器未运行不同。

+   `processlist`

    显示活动服务器线程的列表。这类似于`SHOW PROCESSLIST`语句的输出。如果给出了`--verbose`选项，则输出类似于`SHOW FULL PROCESSLIST`的输出。（参见第 15.7.7.29 节，“SHOW PROCESSLIST Statement”.）

+   `reload`

    重新加载授权表。

+   `refresh`

    刷新所有表并关闭和打开日志文件。

+   `关闭`

    停止服务器。

+   `start-replica`

    在副本服务器上启动复制。使用 MySQL 8.0.26 中的此命令。

+   `start-slave`

    在副本服务器上启动复制。使用 MySQL 8.0.26 之前的此命令。

+   `status`

    显示简短的服务器状态消息。

+   `stop-replica`

    在副本服务器上停止复制。使用 MySQL 8.0.26 中的此命令。

+   `stop-slave`

    在 MySQL 8.0.26 之前，在副本服务器上停止复制。 

+   `变量`

    显示服务器系统变量及其值。

+   `版本`

    显示服务器的版本信息。

所有命令都可以缩写为任何唯一前缀。例如：

```sql
$> mysqladmin proc stat
+----+-------+-----------+----+---------+------+-------+------------------+
| Id | User  | Host      | db | Command | Time | State | Info             |
+----+-------+-----------+----+---------+------+-------+------------------+
| 51 | jones | localhost |    | Query   | 0    |       | show processlist |
+----+-------+-----------+----+---------+------+-------+------------------+
Uptime: 1473624  Threads: 1  Questions: 39487
Slow queries: 0  Opens: 541  Flush tables: 1
Open tables: 19  Queries per second avg: 0.0268
```

**mysqladmin status** 命令结果显示以下数值：

+   `运行时间`

    MySQL 服务器已运行的秒数。

+   `线程`

    活动线程（客户端）的数量。

+   `查询数`

    从服务器启动以来客户端发出的查询数。

+   `慢查询`

    超过`long_query_time`秒的查询数。参见第 7.4.5 节，“慢查询日志”.

+   `打开`

    服务器打开的表的数量。

+   `Flush tables`

    服务器执行的`flush-*`、`refresh`和`reload`命令数。

+   `打开的表`

    当前打开的表的数量。

如果在使用 Unix 套接字文件连接到本地服务器时执行**mysqladmin shutdown**，**mysqladmin** 会等到服务器的进程 ID 文件被移除，以确保服务器已正确停止。

**mysqladmin** 支持以下选项，可以在命令行或选项文件的`[mysqladmin]`和`[client]`组中指定。有关 MySQL 程序使用的选项文件的信息，请参阅 6.2.2.2 节，“使用选项文件”。

**表 6.13 mysqladmin 选项**

| 选项名称 | 描述 | 引入 | 废弃 |
| --- | --- | --- | --- |
| --bind-address | 使用指定的网络接口连接到 MySQL 服务器 |  |  |
| --character-sets-dir | 字符集所在目录 |  |  |
| --compress | 在客户端和服务器之间发送的所有信息进行压缩 |  | 8.0.18 |
| --compression-algorithms | 用于与服务器连接的允许压缩算法 | 8.0.18 |  |
| --connect-timeout | 连接超时前的秒数 |  |  |
| --count | 重复执行命令的迭代次数 |  |  |
| --debug | 写入调试日志 |  |  |
| --debug-check | 程序退出时打印调试信息 |  |  |
| --debug-info | 程序退出时打印调试信息、内存和 CPU 统计信息 |  |  |
| --default-auth | 要使用的认证插件 |  |  |
| --default-character-set | 指定默认字符集 |  |  |
| --defaults-extra-file | 除了通常的选项文件外，还读取指定的选项文件 |  |  |
| --defaults-file | 仅读取指定的选项文件 |  |  |
| --defaults-group-suffix | 选项组后缀值 |  |  |
| --enable-cleartext-plugin | 启用明文认证插件 |  |  |
| --force | 即使发生 SQL 错误也继续 |  |  |
| --get-server-public-key | 从服务器请求 RSA 公钥 |  |  |
| --help | 显示帮助信息并退出 |  |  |
| --host | MySQL 服务器所在的主机 |  |  |
| --login-path | 从 .mylogin.cnf 文件中读取登录路径选项 |  |  |
| --no-beep | 发生错误时不发出蜂鸣声 |  |  |
| --no-defaults | 不读取任何选项文件 |  |  |
| --password | 连接到服务器时要使用的密码 |  |  |
| --password1 | 连接到服务器时要使用的第一个多因素身份验证密码 | 8.0.27 |  |
| --password2 | 连接到服务器时要使用的第二个多因素身份验证密码 | 8.0.27 |  |
| --password3 | 连接到服务器时要使用的第三个多因素身份验证密码 | 8.0.27 |  |
| --pipe | 使用命名管道连接到服务器（仅限 Windows） |  |  |
| --plugin-dir | 插件安装目录 |  |  |
| --port | 连接的 TCP/IP 端口号 |  |  |
| --print-defaults | 打印默认选项 |  |  |
| --protocol | 要使用的传输协议 |  |  |
| --relative | 与--sleep 选项一起使用时显示当前值与先前值之间的差异 |  |  |
| --server-public-key-path | 包含 RSA 公钥的文件的路径名 |  |  |
| --shared-memory-base-name | 共享内存连接的共享内存名称（仅限 Windows） |  |  |
| --show-warnings | 在语句执行后显示警告 |  |  |
| --shutdown-timeout | 等待服务器关闭的最大秒数 |  |  |
| --silent | 静默模式 |  |  |
| --sleep | 重复执行命令，在每次执行之间休眠 delay 秒 |  |  |
| --socket | 要使用的 Unix 套接字文件或 Windows 命名管道 |  |  |
| --ssl-ca | 包含受信任的 SSL 证书颁发机构列表的文件 |  |  |
| --ssl-capath | 包含受信任的 SSL 证书颁发机构证书文件的目录 |  |  |
| --ssl-cert | 包含 X.509 证书的文件 |  |  |
| --ssl-cipher | 连接加密的可允许密码 |  |  |
| --ssl-crl | 包含证书吊销列表的文件 |  |  |
| --ssl-crlpath | 包含证书吊销列表文件的目录 |  |  |
| --ssl-fips-mode | 是否在客户端启用 FIPS 模式 |  | 8.0.34 |
| --ssl-key | 包含 X.509 密钥的文件 |  |  |
| --ssl-mode | 与服务器连接的期望安全状态 |  |  |
| --ssl-session-data | 包含 SSL 会话数据的文件 | 8.0.29 |  |
| --ssl-session-data-continue-on-failed-reuse | 如果会话重用失败，则是否建立连接 | 8.0.29 |  |
| --tls-ciphersuites | 用于加密连接的 TLSv1.3 密码套件 | 8.0.16 |  |
| --tls-version | 可用于加密连接的 TLS 协议 |  |  |
| --user | 连接到服务器时要使用的 MySQL 用户名 |  |  |
| --verbose | 详细模式 |  |  |
| --version | 显示版本信息并退出 |  |  |
| --vertical | 垂直打印查询输出行（每列值一行） |  |  |
| --wait | 如果无法建立连接，则等待并重试，而不是中止 |  |  |
| --zstd-compression-level | 用于使用 zstd 压缩连接到服务器的连接的压缩级别 | 8.0.18 |  |
| 选项名称 | 描述 | 引入 | 废弃 |

+   `--help`, `-?`

    | 命令行格式 | `--help` |
    | --- | --- |

    显示帮助信息并退出。

+   `--bind-address=*`ip_address`*`

    | 命令行格式 | `--bind-address=ip_address` |
    | --- | --- |

    在具有多个网络接口的计算机上，使用此选项选择要用于连接到 MySQL 服务器的接口。

+   `--character-sets-dir=*`dir_name`*`

    | 命令行格式 | `--character-sets-dir=path` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `[none]` |

    安装字符集的目录。请参阅 第 12.15 节，“字符集配置”。

+   `--compress`, `-C`

    | 命令行格式 | `--compress[={OFF&#124;ON}]` |
    | --- | --- |
    | 废弃 | 8.0.18 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    如果可能，压缩客户端和服务器之间发送的所有信息。请参阅 第 6.2.8 节，“连接压缩控制”。

    从 MySQL 8.0.18 开始，此选项已被弃用。预计在未来的 MySQL 版本中将会移除。请参见配置传统连接压缩。

+   `--compression-algorithms=*`value`*`

    | 命令行格式 | `--compression-algorithms=value` |
    | --- | --- |
    | 引入版本 | 8.0.18 |
    | 类型 | 集合 |
    | 默认值 | `uncompressed` |
    | 有效值 | `zlib``zstd``uncompressed` |

    允许用于与服务器连接的压缩算法。可用的算法与`protocol_compression_algorithms`系统变量相同。默认值为`uncompressed`。

    更多信息，请参见第 6.2.8 节，“连接压缩控制”。

    此选项在 MySQL 8.0.18 中添加。

+   `--connect-timeout=*`value`*`

    | 命令行格式 | `--connect-timeout=value` |
    | --- | --- |
    | 类型 | 数值 |
    | 默认值 | `43200` |

    连接超时之前的最大秒数。默认值为 43200（12 小时）。

+   `--count=*`N`*`, `-c *`N`*`

    | 命令行格式 | `--count=#` |
    | --- | --- |

    如果给定了`--sleep`选项，则重复执行命令的次数。

+   [`--debug[=*`debug_options`*]`](mysqladmin.html#option_mysqladmin_debug), `-# [*`debug_options`*]`

    | 命令行格式 | `--debug[=debug_options]` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `d:t:o,/tmp/mysqladmin.trace` |

    写入调试日志。典型的*`debug_options`*字符串是`d:t:o,*`file_name`*`。默认值为`d:t:o,/tmp/mysqladmin.trace`。

    此选项仅在使用`WITH_DEBUG`构建 MySQL 时可用。Oracle 提供的 MySQL 发布二进制文件*不*使用此选项构建。

+   `--debug-check`

    | 命令行格式 | `--debug-check` |
    | --- | --- |
    | 类型 | 布尔值 |
    | 默认值 | `FALSE` |

    在程序退出时打印一些调试信息。

    此选项仅在使用`WITH_DEBUG`构建 MySQL 时可用。Oracle 提供的 MySQL 发布二进制文件*不*使用此选项构建。

+   `--debug-info`

    | 命令行格式 | `--debug-info` |
    | --- | --- |
    | 类型 | 布尔值 |
    | 默认值 | `FALSE` |

    在程序退出时打印调试信息以及内存和 CPU 使用统计信息。

    此选项仅在使用`WITH_DEBUG`构建 MySQL 时可用。Oracle 提供的 MySQL 发布二进制文件*不*使用此选项构建。

+   `--default-auth=*`plugin`*`

    | 命令行格式 | `--default-auth=plugin` |
    | --- | --- |
    | 类型 | 字符串 |

    一个关于使用哪个客户端端身份验证插件的提示。参见第 8.2.17 节，“可插拔身份验证”。

+   `--default-character-set=*`charset_name`*`

    | 命令行格式 | `--default-character-set=charset_name` |
    | --- | --- |
    | 类型 | 字符串 |

    将*`charset_name`*用作默认字符集。参见第 12.15 节，“字符集配置”。

+   `--defaults-extra-file=*`file_name`*`

    | 命令行格式 | `--defaults-extra-file=file_name` |
    | --- | --- |
    | 类型 | 文件名 |

    在全局选项文件之后但（在 Unix 上）在用户选项文件之前读取此选项文件。如果文件不存在或无法访问，将会出现错误。如果*`file_name`*不是绝对路径名，则将其解释为相对于当前目录。

    有关此选项和其他选项文件选项的其他信息，请参见第 6.2.2.3 节，“影响选项文件处理的命令行选项”。

+   `--defaults-file=*`file_name`*`

    | 命令行格式 | `--defaults-file=file_name` |
    | --- | --- |
    | 类型 | 文件名 |

    仅使用给定的选项文件。如果文件不存在或无法访问，将会出现错误。如果*`file_name`*不是绝对路径名，则将其解释为相对于当前目录。

    例外情况：即使使用`--defaults-file`，客户端程序也会读取`.mylogin.cnf`。

    有关此选项和其他选项文件选项的其他信息，请参见第 6.2.2.3 节，“影响选项文件处理的命令行选项”。

+   `--defaults-group-suffix=*`str`*`

    | 命令行格式 | `--defaults-group-suffix=str` |
    | --- | --- |
    | 类型 | 字符串 |

    不仅读取通常的选项组，还读取具有通常名称和后缀*`str`*的组。例如，**mysqladmin**通常会读取`[client]`和`[mysqladmin]`组。如果给定此选项作为`--defaults-group-suffix=_other`，**mysqladmin**还会读取`[client_other]`和`[mysqladmin_other]`组。

    有关此选项和其他选项��件选项的更多信息，请参见第 6.2.2.3 节，“影响选项文件处理的命令行选项”。

+   `--enable-cleartext-plugin`

    | 命令行格式 | `--enable-cleartext-plugin` |
    | --- | --- |
    | 类型 | 布尔值 |
    | 默认值 | `FALSE` |

    启用`mysql_clear_password`明文认证插件。（参见第 8.4.1.4 节，“客户端明文可插拔认证”。）

+   `--force`, `-f`

    | 命令行格式 | `--force` |
    | --- | --- |

    不要为`drop *`db_name`*`命令请求确认。对于多个命令，即使发生错误，也继续执行。

+   `--get-server-public-key`

    | 命令行格式 | `--get-server-public-key` |
    | --- | --- |
    | 类型 | 布尔值 |

    从服务器请求用于 RSA 密钥对密码交换所需的公钥。此选项适用于使用`caching_sha2_password`认证插件进行身份验证的客户端。对于该插件，除非请求，否则服务器不会发送公钥。对于不使用该插件进行身份验证的帐户，此选项将被忽略。如果不使用基于 RSA 的密码交换，例如客户端使用安全连接连接到服务器时，此选项也将被忽略。

    如果`--server-public-key-path=*`file_name`*`已给出并指定了有效的公钥文件，则优先于`--get-server-public-key`。

    有关`caching_sha2_password`插件的更多信息，请参见第 8.4.1.2 节，“缓存 SHA-2 可插拔认证”。

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

    从`.mylogin.cnf`登录路径文件中的命名登录路径读取选项。 “登录路径”是一个包含指定要连接到哪个 MySQL 服务器以及要作为哪个帐户进行身份验证的选项组。要创建或修改登录路径文件，请使用**mysql_config_editor**实用程序。请参见第 6.6.7 节，“mysql_config_editor — MySQL 配置实用程序”。

    关于这个和其他选项文件选项的更多信息，请参见第 6.2.2.3 节，“影响选项文件处理的命令行选项”。

+   `--no-beep`, `-b`

    | 命令行格式 | `--no-beep` |
    | --- | --- |

    抑制默认情况下发出的警告蜂鸣声，例如连接到服务器失败时发出的警告。

+   `--no-defaults`

    | 命令行格式 | `--no-defaults` |
    | --- | --- |

    不要阅读任何选项文件。如果程序启动失败是因为从选项文件中读取了未知选项，可以使用`--no-defaults`来防止它们被读取。

    例外情况是，如果存在`.mylogin.cnf`文件，则在所有情况下都会读取该文件。即使使用`--no-defaults`，这也允许以比在命令行上更安全的方式指定密码。要创建`.mylogin.cnf`，请使用**mysql_config_editor**实用程序。请参见第 6.6.7 节，“mysql_config_editor — MySQL 配置实用程序”。

    关于这个和其他选项文件选项的更多信息，请参见第 6.2.2.3 节，“影响选项文件处理的命令行选项”。

+   [`--password[=*`password`*]`](mysqladmin.html#option_mysqladmin_password), `-p[*`password`*]`

    | 命令行格式 | `--password[=password]` |
    | --- | --- |
    | 类型 | 字符串 |

    用于连接到服务器的 MySQL 帐户的密码。密码值是可选的。如果没有给出，**mysqladmin**会提示输入一个。如果给出，`--password=`或`-p`后面必须*没有空格*跟着密码。如果未指定密码选项，则默认情况是不发送密码。

    在命令行上指定密码应被视为不安全。为了避免在命令行上提供密码，请使用选项文件。请参见第 8.1.2.1 节，“密码安全的最终用户指南”。

    为明确指定没有密码，并且**mysqladmin**不应提示输入密码，请使用`--skip-password`选项。

+   [`--password1[=*`pass_val`*]`](mysqladmin.html#option_mysqladmin_password1)

    用于连接到服务器的 MySQL 帐户的多因素认证因子 1 的密码。密码值是可选的。如果未提供，**mysql**会提示输入密码。如果提供，`--password1=`和后面的密码之间*不能有空格*。如果未指定密码选项，则默认不发送密码。

    在命令行上指定密码应被视为不安全。为了避免在命令行上提供密码，请使用一个选项文件。请参阅第 8.1.2.1 节，“密码安全的最终用户指南”。

    明确指定没有密码，并且**mysqladmin**不应提示输入密码，使用`--skip-password1`选项。

    `--password1`和`--password`是同义词，`--skip-password1`和`--skip-password`也是同义词。

+   [`--password2[=*`pass_val`*]`](mysqladmin.html#option_mysqladmin_password2)

    用于连接到服务器的 MySQL 帐户的多因素认证因子 2 的密码。此选项的语义与`--password1`的语义类似；有关详细信息，请参阅该选项的描述。

+   [`--password3[=*`pass_val`*]`](mysqladmin.html#option_mysqladmin_password3)

    用于连接到服务器的 MySQL 帐户的多因素认证因子 3 的密码。此选项的语义与`--password1`的语义类似；有关详细信息，请参阅该选项的描述。

+   `--pipe`, `-W`

    | 命令行格式 | `--pipe` |
    | --- | --- |
    | 类型 | 字符串 |

    在 Windows 上，使用命名管道连接到服务器。此选项仅在服务器启动时启用了`named_pipe`系统变量以支持命名管道连接时适用。此外，进行连接的用户必须是由`named_pipe_full_access_group`系统变量指定的 Windows 组的成员。

+   `--plugin-dir=*`dir_name`*`

    | 命令行格式 | `--plugin-dir=dir_name` |
    | --- | --- |
    | 类型 | 目录名称 |

    查找插件的目录。如果使用 `--default-auth` 选项指定身份验证插件但 **mysqladmin** 找不到它，请指定此选项。请参见 第 8.2.17 节，“可插拔认证”。

+   `--port=*`port_num`*`, `-P *`port_num`*`

    | 命令行格式 | `--port=port_num` |
    | --- | --- |
    | 类型 | 数值 |
    | 默认值 | `3306` |

    对于 TCP/IP 连接，需要使用的端口号。

+   `--print-defaults`

    | 命令行格式 | `--print-defaults` |
    | --- | --- |

    打印程序名称以及从选项文件获取的所有选项。

    关于此选项文件和其他选项文件选项的更多信息，请参见 第 6.2.2.3 节，“影响选项文件处理的命令行选项”。

+   `--protocol={TCP|SOCKET|PIPE|MEMORY}`

    | 命令行格式 | `--protocol=type` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `[查看文本]` |
    | 有效值 | `TCP``SOCKET``PIPE``MEMORY` |

    用于连接到服务器的传输协议。当其他连接参数通常导致使用不是您想要的协议时，这将非常有用。有关允许值的详细信息，请参见 第 6.2.7 节，“连接传输协议”。

+   `--relative`, `-r`

    | 命令行格式 | `--relative` |
    | --- | --- |

    与 `--sleep` 选项一起使用时，显示当前值与先前值之间的差异。此选项仅适用于 `extended-status` 命令。

+   `--server-public-key-path=*`file_name`*`

    | 命令行格式 | `--server-public-key-path=file_name` |
    | --- | --- |
    | 类型 | 文件名 |

    包含客户端端公钥的 PEM 格式文件的路径名，该公钥是服务器用于 RSA 密钥对密码交换所需的。此选项适用于使用 `sha256_password` 或 `caching_sha2_password` 认证插件进行身份验证的客户端。对于不使用这些插件进行身份验证的帐户，此选项将被忽略。如果不使用基于 RSA 的密码交换（例如客户端使用安全连接连接到服务器时），也将被忽略。

    如果给定 `--server-public-key-path=*`file_name`*` 并指定有效的公钥文件，则它优先于 `--get-server-public-key`。

    对于 `sha256_password`，仅当 MySQL 使用 OpenSSL 构建时才适用此选项。

    有关`sha256_password`和`caching_sha2_password`插件的信息，请参见第 8.4.1.3 节，“SHA-256 可插入式认证”和第 8.4.1.2 节，“缓存 SHA-2 可插入式认证”。

+   `--shared-memory-base-name=*`name`*`

    | 命令行格式 | `--shared-memory-base-name=name` |
    | --- | --- |
    | 特定平台 | Windows |

    在 Windows 上，用于使用共享内存连接到本地服务器的共享内存名称。默认值为`MYSQL`。共享内存名称区分大小写。

    仅当服务器启用了支持共享内存连接的`shared_memory`系统变量时，此选项才适用。

+   `--show-warnings`

    | 命令行格式 | `--show-warnings` |
    | --- | --- |

    显示由发送到服务器的语句执行引起的警告。

+   `--shutdown-timeout=*`value`*`

    | 命令行格式 | `--shutdown-timeout=seconds` |
    | --- | --- |
    | 类型 | 数字 |
    | 默认值 | `3600` |

    等待服务器关闭的最大秒数。默认值为 3600（1 小时）。

+   `--silent`, `-s`

    | 命令行格式 | `--silent` |
    | --- | --- |

    如果无法建立与服务器的连接，则静默退出。

+   `--sleep=*`delay`*`, `-i *`delay`*`

    | 命令行格式 | `--sleep=delay` |
    | --- | --- |

    重复执行命令，在*`delay`*秒之间休眠。`--count`选项确定迭代次数。如果未提供`--count`，**mysqladmin**将无限期执行命令，直到被中断。

+   `--socket=*`path`*`, `-S *`path`*`

    | 命令行格式 | `--socket={file_name&#124;pipe_name}` |
    | --- | --- |
    | 类型 | 字符串 |

    对于连接到`localhost`的连接，要使用的 Unix 套接字文件，或者在 Windows 上要使用的命名管道的名称。

    在 Windows 上，仅当服务器启用了支持命名管道连接的`named_pipe`系统变量时，此选项才适用。此外，进行连接的用户必须是由`named_pipe_full_access_group`系统变量指定的 Windows 组的成员。

+   `--ssl*`

    以 `--ssl` 开头的选项指定是否使用加密连接连接到服务器，并指示 SSL 密钥和证书的位置。请参见加密连接的命令选项。

+   `--ssl-fips-mode={OFF|ON|STRICT}`

    | 命令行格式 | `--ssl-fips-mode={OFF&#124;ON&#124;STRICT}` |
    | --- | --- |
    | 已弃用 | 8.0.34 |
    | 类型 | 枚举 |
    | 默认值 | `OFF` |
    | 有效值 | `OFF``ON``STRICT` |

    控制是否在客户端上启用 FIPS 模式。`--ssl-fips-mode` 选项与其他 `--ssl-*`xxx`*` 选项不同，它不用于建立加密连接，而是影响允许哪些加密操作。请参见第 8.8 节，“FIPS 支持”。

    允许使用的`--ssl-fips-mode` 值为：

    +   `OFF`: 禁用 FIPS 模式。

    +   `ON`: 启用 FIPS 模式。

    +   `STRICT`: 启用“严格” FIPS 模式。

    注意

    如果 OpenSSL FIPS 对象模块不可用，则`--ssl-fips-mode` 的唯一允许值为 `OFF`。在这种情况下，将`--ssl-fips-mode` 设置为 `ON` 或 `STRICT` 会导致客户端在启动时产生警告并在非 FIPS 模式下运行。

    截至 MySQL 8.0.34，此选项已弃用。预计在将来的 MySQL 版本中将其移除。

+   `--tls-ciphersuites=*`ciphersuite_list`*`

    | 命令行格式 | `--tls-ciphersuites=ciphersuite_list` |
    | --- | --- |
    | 引入版本 | 8.0.16 |
    | 类型 | 字符串 |

    用于使用 TLSv1.3 的加密连接的可用密码套件。该值是一个或多个以冒号分隔的密码套件名称列表。可以为此选项命名的密码套件取决于用于编译 MySQL 的 SSL 库。有关详细信息，请参见第 8.3.2 节，“加密连接 TLS 协议和密码”。

    此选项已于 MySQL 8.0.16 中添加。

+   `--tls-version=*`protocol_list`*`

    | 命令行格式 | `--tls-version=protocol_list` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 (≥ 8.0.16) | `TLSv1,TLSv1.1,TLSv1.2,TLSv1.3` (OpenSSL 1.1.1 或更高版本)`TLSv1,TLSv1.1,TLSv1.2` (否则) |
    | 默认值 (≤ 8.0.15) | `TLSv1,TLSv1.1,TLSv1.2` |

    可用于加密连接的 TLS 协议。该值是一个或多个逗号分隔的协议名称列表。可以为此选项命名的协议取决于用于编译 MySQL 的 SSL 库。有关详细信息，请参见第 8.3.2 节，“加密连接 TLS 协议和密码”。

+   `--user=*`user_name`*`, `-u *`user_name`*`

    | 命令行格式 | `--user=user_name,` |
    | --- | --- |
    | 类型 | 字符串 |

    用于连接到服务器的 MySQL 帐户的用户名。

    如果您在 MySQL 8.0.31 或更高版本中使用`Rewriter`插件，则应授予该用户`SKIP_QUERY_REWRITE`权限。

+   `--verbose`, `-v`

    | 命令行格式 | `--verbose` |
    | --- | --- |

    详细模式。打印程序执行的更多信息。

+   `--version`, `-V`

    | 命令行格式 | `--version` |
    | --- | --- |

    显示版本信息并退出。

+   `--vertical`, `-E`

    | 命令行格式 | `--vertical` |
    | --- | --- |

    垂直打印输出。类似于`--relative`，但是垂直打印输出。

+   [`--wait[=*`count`*]`](mysqladmin.html#option_mysqladmin_wait), `-w[*`count`*]`

    | 命令行格式 | `--wait` |
    | --- | --- |

    如果无法建立连接，则等待并重试，而不是中止。如果给定了*`count`*值，则表示重试次数。默认值为一次。

+   `--zstd-compression-level=*`level`*`

    | 命令行格式 | `--zstd-compression-level=#` |
    | --- | --- |
    | 引入版本 | 8.0.18 |
    | 类型 | 整数 |

    用于使用`zstd`压缩算法连接到服务器的压缩级别。允许的级别从 1 到 22，较大的值表示较高级别的压缩。默认的`zstd`压缩级别为 3。压缩级别设置对不使用`zstd`压缩的连接没有影响。

    有关更多信息，请参阅第 6.2.8 节，“连接压缩控制”。

    此选项在 MySQL 8.0.18 中添加。
