> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-command-options.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-command-options.html)

#### 6.5.1.1 mysql 客户端选项

[**mysql**](https://dev.mysql.com/doc/refman/8.0/en/mysql.html "6.5.1 mysql — MySQL 命令行客户端") 支持以下选项，可以在命令行或选项文件的 `[mysql]` 和 `[client]` 组中指定。有关 MySQL 程序使用的选项文件的信息，请参见 [6.2.2.2 使用选项文件](https://dev.mysql.com/doc/refman/8.0/en/option-files.html "6.2.2.2 使用选项文件")。

**表 6.12 mysql 客户端选项**

| 选项名称 | 描述 | 引入 | 废弃 |
| --- | --- | --- | --- |
| [--auto-rehash](https://dev.mysql.com/doc/refman/8.0/en/mysql-command-options.html#option_mysql_auto-rehash) | 启用自动重新哈希 |  |  |
| [--auto-vertical-output](https://dev.mysql.com/doc/refman/8.0/en/mysql-command-options.html#option_mysql_auto-vertical-output) | 启用自动垂直结果集显示 |  |  |
| [--batch](https://dev.mysql.com/doc/refman/8.0/en/mysql-command-options.html#option_mysql_batch) | 不使用历史文件 |  |  |
| [--binary-as-hex](https://dev.mysql.com/doc/refman/8.0/en/mysql-command-options.html#option_mysql_binary-as-hex) | 以十六进制表示显示二进制值 |  |  |
| [--binary-mode](https://dev.mysql.com/doc/refman/8.0/en/mysql-command-options.html#option_mysql_binary-mode) | 禁用 \r\n - 到 - \n 的转换和将 \0 视为查询结束 |  |  |
| [--bind-address](https://dev.mysql.com/doc/refman/8.0/en/mysql-command-options.html#option_mysql_bind-address) | 使用指定的网络接口连接到 MySQL 服务器 |  |  |
| [--character-sets-dir](https://dev.mysql.com/doc/refman/8.0/en/mysql-command-options.html#option_mysql_character-sets-dir) | 安装字符集的目录 |  |  |
| [--column-names](https://dev.mysql.com/doc/refman/8.0/en/mysql-command-options.html#option_mysql_column-names) | 在结果中写入列名 |  |  |
| [--column-type-info](https://dev.mysql.com/doc/refman/8.0/en/mysql-command-options.html#option_mysql_column-type-info) | 显示结果集元数据 |  |  |
| [--comments](https://dev.mysql.com/doc/refman/8.0/en/mysql-command-options.html#option_mysql_comments) | 是否保留或剥离发送到服务器的语句中的注释 |  |  |
| [--compress](https://dev.mysql.com/doc/refman/8.0/en/mysql-command-options.html#option_mysql_compress) | 压缩客户端和服务器之间发送的所有信息 |  | 8.0.18 |
| [--compression-algorithms](https://dev.mysql.com/doc/refman/8.0/en/mysql-command-options.html#option_mysql_compression-algorithms) | 连接到服务器的允许压缩算法 | 8.0.18 |  |
| [--connect-expired-password](https://dev.mysql.com/doc/refman/8.0/en/mysql-command-options.html#option_mysql_connect-expired-password) | 指示服务器客户端可以处理过期密码沙盒模式 |  |  |
| [--connect-timeout](https://dev.mysql.com/doc/refman/8.0/en/mysql-command-options.html#option_mysql_connect-timeout) | 连接超时前的秒数 |  |  |
| [--database](https://dev.mysql.com/doc/refman/8.0/en/mysql-command-options.html#option_mysql_database) | 要使用的数据库 |  |  |
| [--debug](https://dev.mysql.com/doc/refman/8.0/en/mysql-command-options.html#option_mysql_debug) | 写入调试日志；仅在 MySQL 构建时支持调试时可用 |  |  |
| [--debug-check](https://dev.mysql.com/doc/refman/8.0/en/mysql-command-options.html#option_mysql_debug-check) | 程序退出时打印调试信息 |  |  |
| --debug-info | 程序退出时打印调试信息、内存和 CPU 统计 |  |  |
| --default-auth | 要使用的认证插件 |  |  |
| --default-character-set | 指定默认字符集 |  |  |
| --defaults-extra-file | 除了通常的选项文件外，读取命名的选项文件 |  |  |
| --defaults-file | 仅读取命名的选项文件 |  |  |
| --defaults-group-suffix | 选项组后缀值 |  |  |
| --delimiter | 设置语句分隔符 |  |  |
| --dns-srv-name | 使用 DNS SRV 查找主机信息 | 8.0.22 |  |
| --enable-cleartext-plugin | 启用明文认证插件 |  |  |
| --execute | 执行语句并退出 |  |  |
| --fido-register-factor | 必须进行注册的多因素认证因素 | 8.0.27 | 8.0.35 |
| --force | 即使发生 SQL 错误也继续 |  |  |
| --get-server-public-key | 从服务器请求 RSA 公钥 |  |  |
| --help | 显示帮助信息并退出 |  |  |
| --histignore | 指定要忽略记录的语句模式 |  |  |
| --host | MySQL 服务器所在的主机 |  |  |
| --html | 生成 HTML 输出 |  |  |
| --ignore-spaces | 忽略函数名后的空格 |  |  |
| --init-command | 连接后要执行的 SQL 语句 |  |  |
| --line-numbers | 为错误写入行号 |  |  |
| --load-data-local-dir | LOAD DATA LOCAL 语句中命名文件的目录 | 8.0.21 |  |
| --local-infile | 启用或禁用 LOAD DATA 的 LOCAL 功能 |  |  |
| --login-path | 从.mylogin.cnf 中读取登录路径选项 |  |  |
| --max-allowed-packet | 发送到服务器或从服务器接收的最大数据包长度 |  |  |
| --max-join-size | 在使用--safe-updates 时，联接中的行的自动限制 |  |  |
| --named-commands | 启用命名的 mysql 命令 |  |  |
| --net-buffer-length | TCP/IP 和套接字通信的缓冲区大小 |  |  |
| --network-namespace | 指定网络命名空间 | 8.0.22 |  |
| --no-auto-rehash | 禁用自动重新哈希 |  |  |
| --no-beep | 发生错误时不发出蜂鸣声 |  |  |
| --no-defaults | 不读取任何选项文件 |  |  |
| --oci-config-file | 定义 Oracle Cloud Infrastructure CLI 配置文件的替代位置。 | 8.0.27 |  |
| --one-database | 忽略除了命令行上指定的默认数据库之外的语句 |  |  |
| --pager | 用于分页查询输出的给定命令 |  |  |
| --password | 连接到服务器时使用的密码 |  |  |
| --password1 | 连接到服务器时使用的第一个多因素身份验证密码 | 8.0.27 |  |
| --password2 | 连接到服务器时使用的第二个多因素身份验证密码 | 8.0.27 |  |
| --password3 | 连接到服务器时使用的第三个多因素身份验证密码 | 8.0.27 |  |
| --pipe | 使用命名管道连接服务器（仅限 Windows） |  |  |
| --plugin-authentication-kerberos-client-mode | 允许在 Windows 上通过 MIT Kerberos 库进行 GSSAPI 可插拔身份验证 | 8.0.32 |  |
| --plugin-dir | 插件安装目录 |  |  |
| --port | 连接的 TCP/IP 端口号 |  |  |
| --print-defaults | 打印默认选项 |  |  |
| --prompt | 设置指定格式的提示符 |  |  |
| --protocol | 使用的传输协议 |  |  |
| --quick | 不缓存每个查询结果 |  |  |
| --raw | 写入列值而不进行转义转换 |  |  |
| --reconnect | 如果与服务器的连接丢失，自动尝试重新连接 |  |  |
| --safe-updates, --i-am-a-dummy | 仅允许指定键值的 UPDATE 和 DELETE 语句 |  |  |
| --select-limit | 在使用 --safe-updates 时 SELECT 语句的自动限制 |  |  |
| --server-public-key-path | 包含 RSA 公钥的文件的路径名 |  |  |
| --shared-memory-base-name | 共享内存连接的共享内存名称（仅限 Windows） |  |  |
| --show-warnings | 如果有任何警告，则在每个语句后显示警告 |  |  |
| --sigint-ignore | 忽略 SIGINT 信号（通常是键入 Control+C 的结果） |  |  |
| --silent | 静默模式 |  |  |
| --skip-auto-rehash | 禁用自动重新哈希 |  |  |
| --skip-column-names | 在结果中不写入列名 |  |  |
| --skip-line-numbers | 跳过错误的行号 |  |  |
| --skip-named-commands | 禁用命名的 mysql 命令 |  |  |
| --skip-pager | 禁用分��� |  |  |
| --skip-reconnect | 禁用重新连接 |  |  |
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
| --syslog | 将交互式语句记录到 syslog |  |  |
| --table | 以表格格式显示输出 |  |  |
| --tee | 将输出的副本追加到指定文件 |  |  |
| --tls-ciphersuites | 加密连接的允许的 TLSv1.3 密码套件 | 8.0.16 |  |
| --tls-version | 加密连接的允许的 TLS 协议 |  |  |
| --unbuffered | 在每个查询后刷新缓冲区 |  |  |
| --user | 连接到服务器时要使用的 MySQL 用户名 |  |  |
| --verbose | 详细模式 |  |  |
| --version | 显示版本信息并退出 |  |  |
| --vertical | 垂直打印查询输出行（每列值一行） |  |  |
| --wait | 如果无法建立连接，则等待并重试，而不是中止 |  |  |
| --xml | 生成 XML 输出 |  |  |
| --zstd-compression-level | 使用 zstd 压缩连接到服务器的压缩级别 | 8.0.18 |  |
| 选项名称 | 描述 | 引入 | 废弃 |

+   `--help`, `-?`

    | 命令行格式 | `--help` |
    | --- | --- |

    显示帮助信息并退出。

+   `--auto-rehash`

    | 命令行格式 | `--auto-rehash` |
    | --- | --- |
    | 禁用方式 | `skip-auto-rehash` |

    启用自动 rehash。默认情况下，此选项已启用，可实现数据库、表和列名的自动补全。使用`--disable-auto-rehash`来禁用 rehash。这会使**mysql**启动更快，但如果要使用名称补全，则必须发出`rehash`命令或其`\#`快捷方式。

    要完成一个名称，输入第一部分并按 Tab 键。如果名称是明确的，**mysql**会自动完成。否则，您可以再次按 Tab 键查看以您已输入的内容开头的可能名称。如果没有默认数据库，则不会发生自动完成。

    注意

    此功能需要使用**readline**库编译的 MySQL 客户端。通常，**readline**库在 Windows 上不可用。

+   `--auto-vertical-output`

    | 命令行格式 | `--auto-vertical-output` |
    | --- | --- |

    如果结果集对当前窗口太宽，则使其以垂直方式显示，否则使用正常的表格格式显示。（这适用于以`;`或`\G`结尾的语句。）

+   `--batch`, `-B`

    | 命令行格式 | `--batch` |
    | --- | --- |

    使用制表符作为列分隔符打印结果，每行显示在新行上。使用此选项，**mysql**不使用历史文件。

    批处理模式导致非表格输出格式和特殊字符的转义。可以通过使用原始模式来禁用转义；请参阅`--raw`选项的描述。

+   `--binary-as-hex`

    | 命令行格式 | `--binary-as-hex` |
    | --- | --- |
    | 类型 | 布尔值 |
    | 默认值（≥ 8.0.19） | `非交互模式下为 FALSE` |
    | 默认值（≤ 8.0.18） | `FALSE` |

    当给定此选项时，**mysql**会使用十六进制表示法（`0x*`value`*`）显示二进制数据。无论整体输出显示格式是表格、垂直、HTML 还是 XML，都会发生这种情况。

    当启用`--binary-as-hex`选项时，会影响所有二进制字符串的显示，包括由`CHAR()`和`UNHEX()`等函数返回的字符串。以下示例演示了使用 ASCII 码`A`（65 十进制，41 十六进制）的情况：

    +   禁用`--binary-as-hex`：

        ```sql
        mysql> SELECT CHAR(0x41), UNHEX('41');
        +------------+-------------+
        | CHAR(0x41) | UNHEX('41') |
        +------------+-------------+
        | A          | A           |
        +------------+-------------+
        ```

    +   启用`--binary-as-hex`：

        ```sql
        mysql> SELECT CHAR(0x41), UNHEX('41');
        +------------------------+--------------------------+
        | CHAR(0x41)             | UNHEX('41')              |
        +------------------------+--------------------------+
        | 0x41                   | 0x41                     |
        +------------------------+--------------------------+
        ```

    要编写一个二进制字符串表达式，以便无论是否启用`--binary-as-hex`，都将其显示为字符字符串，请使用以下技术：

    +   `CHAR()`函数有一个`USING *`charset`*`子句：

        ```sql
        mysql> SELECT CHAR(0x41 USING utf8mb4);
        +--------------------------+
        | CHAR(0x41 USING utf8mb4) |
        +--------------------------+
        | A                        |
        +--------------------------+
        ```

    +   更一般地，使用`CONVERT()`将表达式转换为给定的字符集：

        ```sql
        mysql> SELECT CONVERT(UNHEX('41') USING utf8mb4);
        +------------------------------------+
        | CONVERT(UNHEX('41') USING utf8mb4) |
        +------------------------------------+
        | A                                  |
        +------------------------------------+
        ```

    截至 MySQL 8.0.19 版本，当**mysql**以交互模式运行时，默认情况下启用此选项。此外，当隐式或显式启用该选项时，`status`（或 `\s`）命令的输出包括以下行：

    ```sql
    Binary data as: Hexadecimal
    ```

    要禁用十六进制表示法，请使用`--skip-binary-as-hex`。

+   `--binary-mode`

    | 命令行格式 | `--binary-mode` |
    | --- | --- |

    此选项有助于处理可能包含`BLOB`值的**mysqlbinlog**输出。默认情况下，**mysql**将语句字符串中的 `\r\n` 转换为 `\n`，并将 `\0` 解释为语句终止符。`--binary-mode`禁用这两个功能。它还在非交互模式下禁用所有**mysql**命令，除了在输入通过管道传输到**mysql**或使用 `source` 命令加载时的 `charset` 和 `delimiter`。

+   `--bind-address=*`ip_address`*`

    | 命令行格式 | `--bind-address=ip_address` |
    | --- | --- |

    在具有多个网络接口的计算机上，使用此选项选择连接到 MySQL 服务器的接口。

+   `--character-sets-dir=*`dir_name`*`

    | 命令行格式 | `--character-sets-dir=dir_name` |
    | --- | --- |
    | 类型 | 目录���称 |

    安装字符集的目录。请参阅第 12.15 节，“字符集配置”。

+   `--column-names`

    | 命令行格式 | `--column-names` |
    | --- | --- |

    写入结果中的列名。

+   `--column-type-info`

    | 命令行格式 | `--column-type-info` |
    | --- | --- |

    显示结果集元数据。此信息对应于 C API `MYSQL_FIELD` 数据结构的内容。请参阅 C API 基本数据结构。

+   `--comments`，`-c`

    | 命令行格式 | `--comments` |
    | --- | --- |
    | 类型 | 布尔值 |
    | 默认值 | `FALSE` |

    是否剥离或保留发送到服务器的语句中的注释。默认为`--skip-comments`（剥离注释），启用则使用`--comments`（保留注释）。

    注意

    无论是否给出此选项，**mysql**客户端始终向服务器传递优化器提示。

    注释剥离已弃用。预计此功能及其控制选项将在未来的 MySQL 版本中被移除。

+   `--compress`, `-C`

    | 命令行格式 | `--compress[={OFF&#124;ON}]` |
    | --- | --- |
    | 已弃用 | 8.0.18 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    如果可能的话，尽量压缩客户端和服务器之间发送的所有信息。参见 第 6.2.8 节，“连接压缩控制”。

    从 MySQL 8.0.18 开始，此选项已弃用。预计在未来的 MySQL 版本中将其移除。请参见 配置传统连接压缩。

+   `--compression-algorithms=*`value`*`

    | 命令行格式 | `--compression-algorithms=value` |
    | --- | --- |
    | 引入版本 | 8.0.18 |
    | 类型 | 集合 |
    | 默认值 | `uncompressed` |
    | 有效数值 | `zlib``zstd``uncompressed` |

    连接到服务器的允许的压缩算法。可用的算法与 `protocol_compression_algorithms` 系统变量相同。默认值为 `uncompressed`。

    有关更多信息，请参见 第 6.2.8 节，“连接压缩控制”。

    此选项在 MySQL 8.0.18 中添加。

+   `--connect-expired-password`

    | 命令行格式 | `--connect-expired-password` |
    | --- | --- |

    表示客户端可以处理沙盒模式，如果用于连接的帐户密码已过期。这对于非交互式调用 **mysql** 可能很有用，因为通常情况下，服务器会断开尝试使用帐户密码已过期的帐户连接的非交互式客户端。（参见 第 8.2.16 节，“服务器处理过期密码”。）

+   `--connect-timeout=*`value`*`

    | 命令行格式 | `--connect-timeout=value` |
    | --- | --- |
    | 类型 | 数值 |
    | 默认值 | `0` |

    连接超时之前的秒数。（默认值为 `0`。）

+   `--database=*`db_name`*`, `-D *`db_name`*`

    | 命令行格式 | `--database=dbname` |
    | --- | --- |
    | 类型 | 字符串 |

    要使用的数据库。这主要在选项文件中很有用。

+   [`--debug[=*`debug_options`*]`](mysql-command-options.html#option_mysql_debug), `-# [*`debug_options`*]`

    | 命令行格式 | `--debug[=debug_options]` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `d:t:o,/tmp/mysql.trace` |

    写一个调试日志。一个典型的*`debug_options`*字符串是`d:t:o,*`file_name`*`。默认值是`d:t:o,/tmp/mysql.trace`。

    只有在使用`WITH_DEBUG`构建 MySQL 时才可用。由 Oracle 提供的 MySQL 发布二进制文件*不*使用此选项构建。

+   `--debug-check`

    | 命令行格式 | `--debug-check` |
    | --- | --- |
    | 类型 | 布尔值 |
    | 默认值 | `FALSE` |

    在程序退出时打印一些调试信息。

    只有在使用`WITH_DEBUG`构建 MySQL 时才可用。由 Oracle 提供的 MySQL 发布二进制文件*不*使用此选项构建。

+   `--debug-info`, `-T`

    | 命令行格式 | `--debug-info` |
    | --- | --- |
    | 类型 | 布尔值 |
    | 默认值 | `FALSE` |

    在程序退出时打印调试信息以及内存和 CPU 使用统计信息。

    只有在使用`WITH_DEBUG`构建 MySQL 时才可用。由 Oracle 提供的 MySQL 发布二进制文件*不*使用此选项构建。

+   `--default-auth=*`plugin`*`

    | 命令行格式 | `--default-auth=plugin` |
    | --- | --- |
    | 类型 | 字符串 |

    一个关于使用哪个客户端端身份验证插件的提示。请参阅第 8.2.17 节，“可插拔认证”。

+   `--default-character-set=*`charset_name`*`

    | 命令行格式 | `--default-character-set=charset_name` |
    | --- | --- |
    | 类型 | 字符串 |

    将*`charset_name`*用作客户端和连接的默认字符集。

    如果操作系统使用一个字符集，而**mysql**客户端默认使用另一个字符集，则此选项可能很有用。在这种情况下，输出可能格式不正确。通常可以通过使用此选项强制客户端使用系统字符集来解决此类问题。

    欲了解更多信息，请参阅第 12.4 节，“连接字符集和校对规则”，以及第 12.15 节，“字符集配置”。

+   `--defaults-extra-file=*`file_name`*`

    | 命令行格式 | `--defaults-extra-file=file_name` |
    | --- | --- |
    | 类型 | 文件名 |

    在全局选项文件之后但（在 Unix 上）在用户选项文件之前读取此选项文件。如果文件不存在或无法访问，将会出现错误。如果*`file_name`*不是绝对路径名，则将其解释为相对于当前目录。

    关于此选项文件和其他选项文件选项的更多信息，请参见第 6.2.2.3 节，“影响选项文件处理的命令行选项”。

+   `--defaults-file=*`file_name`*`

    | 命令行格式 | `--defaults-file=file_name` |
    | --- | --- |
    | 类型 | 文件名 |

    仅使用给定的选项文件。如果文件不存在或无法访问，将会出现错误。如果*`file_name`*不是绝对路径名，则将其解释为相对于当前目录。

    例外：即使使用`--defaults-file`，客户端程序也会读取`.mylogin.cnf`。

    关于此选项文件和其他选项文件选项的更多信息，请参见第 6.2.2.3 节，“影响选项文件处理的命令行选项”。

+   `--defaults-group-suffix=*`str`*`

    | 命令行格式 | `--defaults-group-suffix=str` |
    | --- | --- |
    | 类型 | 字符串 |

    不仅读取通常的选项组，还读取具有通常名称和后缀*`str`*的组。例如，**mysql**通常会读取`[client]`和`[mysql]`组。如果给定此选项为`--defaults-group-suffix=_other`，**mysql**还会读取`[client_other]`和`[mysql_other]`组。

    关于此选项文件和其他选项文件选项的更多信息，请参见第 6.2.2.3 节，“影响选项文件处理的命令行选项”。

+   `--delimiter=*`str`*`

    | 命令行格式 | `--delimiter=str` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `;` |

    设置语句分隔符。默认为分号字符(`;`)。

+   `--disable-named-commands`

    禁用命名命令。仅使用`\*`形式，或仅在以分号(`;`)结尾的行的开头使用命名命令。**mysql**默认启用此选项。但是，即使使用此选项，长格式命令仍然可以从第一行起作用。请参见第 6.5.1.2 节，“mysql 客户端命令”。

+   `--dns-srv-name=*`name`*`

    | 命令行格式 | `--dns-srv-name=name` |
    | --- | --- |
    | 引入版本 | 8.0.22 |
    | 类型 | 字符串 |

    指定确定用于建立与 MySQL 服务器的连接的候选主机的 DNS SRV 记录的名称。有关 MySQL 中 DNS SRV 支持的信息，请参见 第 6.2.6 节，“使用 DNS SRV 记录连接到服务器”。

    假设 DNS 配置了 `example.com` 域的此 SRV 信息：

    ```sql
    Name                     TTL   Class   Priority Weight Port Target
    _mysql._tcp.example.com. 86400 IN SRV  0        5      3306 host1.example.com
    _mysql._tcp.example.com. 86400 IN SRV  0        10     3306 host2.example.com
    _mysql._tcp.example.com. 86400 IN SRV  10       5      3306 host3.example.com
    _mysql._tcp.example.com. 86400 IN SRV  20       5      3306 host4.example.com
    ```

    要使用该 DNS SRV 记录，请像这样调用 **mysql**：

    ```sql
    mysql --dns-srv-name=_mysql._tcp.example.com
    ```

    **mysql** 然后尝试连接组中的每个服务器，直到建立成功的连接。仅当无法建立到任何服务器的连接时才会发生连接失败。DNS SRV 记录中的优先级和权重值确定应尝试的服务器顺序。

    当使用 `--dns-srv-name` 调用时，**mysql** 仅尝试建立 TCP 连接。

    如果同时给出 `--dns-srv-name` 选项和 `--host` 选项，则 `--dns-srv-name` 选项优先于 `--host` 选项。如果在 **mysql** 启动时同时给出了 `--dns-srv-name` 选项以指定 DNS SRV 记录，并且随后在运行时使用 `connect` 命令并指定主机名参数，则该主机名优先于任何 `--dns-srv-name` 选项。

    此选项是在 MySQL 8.0.22 中添加的。

+   `--enable-cleartext-plugin`

    | 命令行格式 | `--enable-cleartext-plugin` |
    | --- | --- |
    | 类型 | 布尔 |
    | 默认值 | `FALSE` |

    启用 `mysql_clear_password` 明文认证插件。（请参阅 第 8.4.1.4 节，“客户端端明文插件认证”。）

+   `--execute=*`statement`*`, `-e *`statement`*`

    | 命令行格式 | `--execute=statement` |
    | --- | --- |
    | 类型 | 字符串 |

    执行语句并退出。默认输出格式类似于使用 `--batch` 生成的格式。有关示例，请参见 第 6.2.2.1 节，“在命令行上使用选项”。使用此选项，**mysql** 不使用历史文件。

+   `--fido-register-factor=*`value`*`

    | 命令行格式 | `--fido-register-factor=value` |
    | --- | --- |
    | 引入 | 8.0.27 |
    | 已弃用 | 8.0.35 |
    | 类型 | 字符串 |

    注意

    从 MySQL 8.0.35 开始，此选项已弃用，并可能在未来的 MySQL 发行版中删除。

    必须执行 FIDO 设备注册的因素或因素。此选项值必须是单个值，或用逗号分隔的两个值。每个值必须是 2 或 3，因此允许的选项值为`'2'`、`'3'`、`'2,3'`和`'3,2'`。

    例如，需要为第三个认证因素注册的帐户调用 **mysql** 客户端如下：

    ```sql
    mysql --user=*user_name* --fido-register-factor=3
    ```

    需要为第二个和第三个认证因素注册的帐户调用 **mysql** 客户端如下：

    ```sql
    mysql --user=*user_name* --fido-register-factor=2,3
    ```

    如果注册成功，将建立连接。如果存在待注册的认证因素，在尝试连接到服务器时，连接将进入待注册模式。在这种情况下，断开连接并重新连接，使用正确的`--fido-register-factor`值完成注册。

    注册是一个包括 *启动注册* 和 *完成注册* 步骤的两步过程。启动注册步骤执行以下语句：

    ```sql
    ALTER USER *user* *factor* INITIATE REGISTRATION
    ```

    该语句返回一个包含 32 字节挑战、用户名和依赖方 ID（参见 `authentication_fido_rp_id`）的结果集。

    完成注册步骤执行以下语句：

    ```sql
    ALTER USER *user* *factor* FINISH REGISTRATION SET CHALLENGE_RESPONSE AS '*auth_string*'
    ```

    该语句完成注册并将以下信息作为 *`auth_string`* 的一部分发送到服务器：认证器数据，X.509 格式的可选证书，以及签名。

    必须在单个连接中执行启动和注册步骤，因为客户端在启动步骤期间接收到的挑战会保存到客户端连接处理程序中。如果注册步骤由不同的连接执行，注册将失败。`--fido-register-factor` 选项执行启动和注册步骤，避免了上述失败场景，并避免了手动执行 `ALTER USER` 启动和注册语句。

    `--fido-register-factor` 选项仅适用于 **mysql** 客户端和 MySQL Shell。其他 MySQL 客户端程序不支持该选项。

    有关相关信息，请参阅使用 FIDO 认证。

+   `--force`, `-f`

    | 命令行格式 | `--force` |
    | --- | --- |

    即使发生 SQL 错误，也继续执行。

+   `--get-server-public-key`

    | 命令行格式 | `--get-server-public-key` |
    | --- | --- |
    | 类型 | 布尔值 |

    从服务器请求用于 RSA 密钥对密码交换所需的公钥。此选项适用于使用`caching_sha2_password`认证插件进行身份验证的客户端。对于该插件，除非请求，否则服务器不会发送公钥。对于不使用该插件进行身份验证的帐户，此选项将被忽略。如果不使用基于 RSA 的密码交换（例如客户端使用安全连接连接到服务器时），此选项也将被忽略。

    如果提供了`--server-public-key-path=*`file_name`*`并指定了有效的公钥文件，则它优先于`--get-server-public-key`。

    有关`caching_sha2_password`插件的信息，请参阅第 8.4.1.2 节，“缓存 SHA-2 可插拔认证”。

+   `--histignore`

    | 命令行格式 | `--histignore=pattern_list` |
    | --- | --- |
    | 类型 | 字符串 |

    一个或多个以冒号分隔的模式列表，用于指定在记录日志时要忽略的语句。这些模式将添加到默认模式列表(`"*IDENTIFIED*:*PASSWORD*"`)中。为此选项指定的值会影响写入历史文件和`syslog`（如果给出了`--syslog`选项）的语句的记录。有关更多信息，请参阅第 6.5.1.3 节，“mysql 客户端日志记录”。

+   `--host=*`host_name`*`, `-h *`host_name`*`

    | 命令行格式 | `--host=host_name` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `localhost` |

    连接到给定主机上的 MySQL 服务器。

    如果同时给出 `--dns-srv-name` 选项和 `--host` 选项，则 `--dns-srv-name` 选项优先于 `--host` 选项。`--dns-srv-name` 导致连接建立使用 `mysql_real_connect_dns_srv()` C API 函数而不是 `mysql_real_connect()`。但是，如果在运行时随后使用 `connect` 命令并指定主机名参数，则该主机名优先于在 **mysql** 启动时给出的任何 `--dns-srv-name` 选项以指定 DNS SRV 记录。

+   `--html`, `-H`

    | 命��行格式 | `--html` |
    | --- | --- |

    生成 HTML 输出。

+   `--ignore-spaces`, `-i`

    | 命令行格式 | `--ignore-spaces` |
    | --- | --- |

    忽略函数名后的空格。其效果在`IGNORE_SPACE` SQL 模式的讨论中有描述（参见第 7.1.11 节，“服务器 SQL 模式”）。

+   `--init-command=str`

    | 命令行格式 | `--init-command=str` |
    | --- | --- |

    连接到服务器后要执行的单个 SQL 语句。如果启用了自动重新连接，则在重新连接发生后再次执行该语句。

+   `--line-numbers`

    | 命令行格式 | `--line-numbers` |
    | --- | --- |
    | 禁用者 | `skip-line-numbers` |

    为错误写入行号。使用 `--skip-line-numbers` 禁用此功能。

+   `--load-data-local-dir=*`dir_name`*`

    | 命令行格式 | `--load-data-local-dir=dir_name` |
    | --- | --- |
    | 引入版本 | 8.0.21 |
    | 类型 | 目录名称 |
    | 默认值 | `空字符串` |

    此选项影响客户端端的 `LOCAL` 能力，用于 `LOAD DATA` 操作。它指定了 `LOAD DATA LOCAL` 语句中命名的文件必须位于的目录。`--load-data-local-dir` 的效果取决于是否启用或禁用了 `LOCAL` 数据加载：

    +   如果启用了 `LOCAL` 数据加载，无论是在 MySQL 客户端库中默认启用还是通过指定 [`--local-infile[=1]`](mysql-command-options.html#option_mysql_local-infile)，都会忽略 `--load-data-local-dir` 选项。

    +   如果禁用了`LOCAL`数据加载，无论是默认在 MySQL 客户端库中还是通过指定`--local-infile=0`，都会应用`--load-data-local-dir`选项。

    当应用`--load-data-local-dir`时，选项值指定了必须位于其中的本地数据文件的目录。无论底层文件系统的大小写敏感性如何，目录路径名和要加载的文件的路径名的比较都是区分大小写的。如果选项值为空字符串，则不指定任何目录，结果是不允许进行本地数据加载。

    例如，要显式禁用除位于`/my/local/data`目录中的文件之外的���地数据加载，请像这样调用**mysql**：

    ```sql
    mysql --local-infile=0 --load-data-local-dir=/my/local/data
    ```

    当同时给出`--local-infile`和`--load-data-local-dir`时，它们给出的顺序并不重要。

    在**mysql**中成功使用`LOCAL`加载操作还需要服务器允许本地加载；请参阅第 8.1.6 节，“LOAD DATA LOCAL 的安全考虑”

    `--load-data-local-dir`选项是在 MySQL 8.0.21 中添加的。

+   [`--local-infile[={0|1}]`](mysql-command-options.html#option_mysql_local-infile)

    | 命令行格式 | `--local-infile[={0 | 1}]` |
    | --- | --- | --- |
    | 类型 | 布尔值 |
    | 默认值 | `FALSE` |

    默认情况下，`LOAD DATA`的`LOCAL`功能由编译到 MySQL 客户端库中的默认值确定。要显式启用或禁用`LOCAL`数据加载，请使用`--local-infile`选项。当不带值给出时，该选项启用`LOCAL`数据加载。当作为`--local-infile=0`或`--local-infile=1`给出时，该选项禁用或启用`LOCAL`数据加载。

    如果禁用了`LOCAL`功能，则可以使用`--load-data-local-dir`选项来允许在指定目录中的文件进行受限制的本地加载。

    在**mysql**中成功使用`LOCAL`加载操作还需要服务器允许本地加载；请参阅第 8.1.6 节，“LOAD DATA LOCAL 的安全考虑”

+   `--login-path=*`name`*`

    | 命令行格式 | `--login-path=name` |
    | --- | --- |
    | 类型 | 字符串 |

    从`.mylogin.cnf`登录路径文件中的命名登录路径读取选项。 “登录路径”是一个包含指定要连接的 MySQL 服务器和要进行身份验证的帐户的选项组。要创建或修改登录路径文件，请使用**mysql_config_editor**实用程序。请参见第 6.6.7 节，“mysql_config_editor — MySQL 配置实用程序”。

    有关此选项和其他选项文件选项的更多信息，请参见第 6.2.2.3 节，“影响选项文件处理的命令行选项”。

+   `--max-allowed-packet=*`value`*`

    | 命令行格式 | `--max-allowed-packet=value` |
    | --- | --- |
    | 类型 | 数值 |
    | 默认值 | `16777216` |

    客户端/服务器通信缓冲区的最大大小。默认值为 16MB，最大值为 1GB。

+   `--max-join-size=*`value`*`

    | 命令行格式 | `--max-join-size=value` |
    | --- | --- |
    | 类型 | 数值 |
    | 默认值 | `1000000` |

    在使用`--safe-updates`时，连接中联接的行的自动限制。（默认值为 1,000,000。）

+   `--named-commands`, `-G`

    | 命令行格式 | `--named-commands` |
    | --- | --- |
    | 被`skip-named-commands`禁用 |

    启用命名**mysql**命令。允许长格式命令，而不仅仅是短格式命令。例如，`quit`和`\q`都被识别。使用`--skip-named-commands`来禁用命名命令。请参见第 6.5.1.2 节，“mysql 客户端命令”。

+   `--net-buffer-length=*`value`*`

    | 命令行格式 | `--net-buffer-length=value` |
    | --- | --- |
    | 类型 | 数值 |
    | 默认值 | `16384` |

    TCP/IP 和套接字通信的缓冲区大小。（默认值为 16KB。）

+   `--network-namespace=*`name`*`

    | 命令行格式 | `--network-namespace=name` |
    | --- | --- |
    | 引入版本 | 8.0.22 |
    | 类型 | 字符串 |

    用于 TCP/IP 连接的网络命名空间。如果省略，连接将使用默认（全局）命名空间。有关网络命名空间的信息，请参见第 7.1.14 节，“网络命名空间支持”。

    此选项已添加到 MySQL 8.0.22 中。仅在实现网络命名空间支持的平台上可用。

+   `--no-auto-rehash`, `-A`

    | 命令行格式 | `--no-auto-rehash` |
    | --- | --- |
    | 已弃用 | 是 |

    这与`--skip-auto-rehash`具有相同的效果。请参阅`--auto-rehash`的描述。

+   `--no-beep`, `-b`

    | 命令行格式 | `--no-beep` |
    | --- | --- |

    当发生错误时不发出蜂鸣声。

+   `--no-defaults`

    | 命令行格式 | `--no-defaults` |
    | --- | --- |

    不要读取任何选项文件。如果程序启动失败是因为从选项文件中读取了未知选项，可以使用`--no-defaults`来阻止它们被读取。

    例外情况是，如果存在`.mylogin.cnf`文件，则在所有情况下都会读取该文件。这允许以比在命令行上更安全的方式指定密码，即使使用`--no-defaults`也是如此。要创建`.mylogin.cnf`，请使用**mysql_config_editor**实用程序。请参阅第 6.6.7 节，“mysql_config_editor — MySQL 配置实用程序”。

    有关此选项和其他选项文件选项的附加信息，请参阅第 6.2.2.3 节，“影响选项文件处理的命令行选项”。

+   `--one-database`, `-o`

    | 命令行格式 | `--one-database` |
    | --- | --- |

    忽略除了在默认数据库为命令行上指定的数据库时发生的语句之外的所有语句。此选项是基本的，应谨慎使用。语句过滤仅基于`USE`语句。

    最初，**mysql**执行输入中的语句，因为在命令行上指定一个数据库*`db_name`*等同于在输入开头插入`USE *`db_name`*`。然后，对于每个遇到的`USE`语句，**mysql**根据命令行上的数据库名接受或拒绝后续语句。语句的内容并不重要。

    假设**mysql**被调用来处理这组语句：

    ```sql
    DELETE FROM db2.t2;
    USE db2;
    DROP TABLE db1.t1;
    CREATE TABLE db1.t1 (i INT);
    USE db1;
    INSERT INTO t1 (i) VALUES(1);
    CREATE TABLE db2.t1 (j INT);
    ```

    如果命令行是**mysql --force --one-database db1**，**mysql**处理输入如下：

    +   `DELETE` 语句会被执行，因为默认数据库是 `db1`，尽管该语句指定了不同数据库中的表。

    +   `DROP TABLE` 和 `CREATE TABLE` 语句不会被执行，因为默认数据库不是 `db1`，尽管这些语句指定了 `db1` 中的表。

    +   `INSERT` 和 `CREATE TABLE` 语句会被执行，因为默认数据库是 `db1`，尽管 `CREATE TABLE` 语句指定了不同数据库中的表。

+   [`--pager[=*`command`*]`](mysql-command-options.html#option_mysql_pager)

    | 命令行格式 | `--pager[=command]` |
    | --- | --- |
    | 被 `skip-pager` 禁用 | `--password[=password]` |
    | 类型 | 字符串 |

    使用给定的命令来分页查询输出。如果省略命令，则默认分页器是您的 `PAGER` 环境变量的值。有效的分页器包括 **less**、**more**、**cat [> filename]** 等。此选项仅在 Unix 上以及仅在交互模式下有效。要禁用分页，请使用 `--skip-pager`。Section 6.5.1.2, “mysql Client Commands” 进一步讨论了输出分页。

+   [`--password[=*`password`*]`](mysql-command-options.html#option_mysql_password), `-p[*`password`*]`

    | 命令行格式 | `--password[=password]` |
    | --- | --- |
    | 类型 | 字符串 |

    用于连接到服务器的 MySQL 账户的密码。密码值是可选的。如果没有提供，**mysql** 会提示输入密码。如果提供了密码，`--password=` 或 `-p` 与后面的密码之间*不能有空格*。如果未指定密码选项，则默认情况下不发送密码。

    在命令行上指定密码应被视为不安全。为了避免在命令行上提供密码，请使用选项文件。参见 Section 8.1.2.1, “End-User Guidelines for Password Security”。

    要明确指定没有密码，并且 **mysql** 不应提示输入密码，使用 `--skip-password` 选项。

+   [`--password1[=*`pass_val`*]`](mysql-command-options.html#option_mysql_password1)

    用于连接到服务器的 MySQL 帐户的多因素认证因子 1 的密码。密码值是可选的。如果未给出，则**mysql**会提示输入密码。如果给出，则`--password1=`和后面的密码之间*不能有空格*。如果未指定密码选项，则默认情况下不发送密码。

    在命令行上指定密码应被视为不安全。为了避免在命令行上提供密码，请使用选项文件。请参阅第 8.1.2.1 节，“密码安全的最终用户指南”。

    要明确指定没有密码，并且**mysql**不应提示密码，请使用`--skip-password1`选项。

    `--password1`和`--password`是同义词，`--skip-password1`和`--skip-password`也是同义词。

+   [`--password2[=*`pass_val`*]`](mysql-command-options.html#option_mysql_password2)

    用于连接到服务器的 MySQL 帐户的多因素认证因子 2 的密码。此选项的语义与`--password1`的语义类似；有关详细信息，请参阅该选项的描述。

+   [`--password3[=*`pass_val`*]`](mysql-command-options.html#option_mysql_password3)

    用于连接到服务器的 MySQL 帐户的多因素认证因子 3 的密码。此选项的语义与`--password1`的语义类似；有关详细信息，请参阅该选项的描述。

+   `--pipe`, `-W`

    | 命令行格式 | `--pipe` |
    | --- | --- |
    | 类型 | 字符串 |

    在 Windows 上，使用命名管道连接到服务器。此选项仅在服务器启动时启用了支持命名管道连接的`named_pipe`系统变量时才适用。此外，进行连接的用户必须是由`named_pipe_full_access_group`系统变量指定的 Windows 组的成员。

+   `--plugin-authentication-kerberos-client-mode=*`value`*`

    | 命令行格式 | `--plugin-authentication-kerberos-client-mode` |
    | --- | --- |
    | 引入版本 | 8.0.32 |
    | 类型 | 字符串 |
    | 默认值 | `SSPI` |
    | 有效值 | `GSSAPI``SSPI` |

    在 Windows 上，`authentication_kerberos_client`认证插件支持此插件选项。它提供了两个客户端用户可以在运行时设置的可能值：`SSPI`和`GSSAPI`。

    客户端端插件选项的默认值使用了 Security Support Provider Interface（SSPI），它能够从 Windows 内存缓存中获取凭据。另外，客户端用户可以选择支持通过 Windows 上的 MIT Kerberos 库使用 Generic Security Service Application Program Interface（GSSAPI）的模式。GSSAPI 能够获取先前使用**kinit**命令生成的缓存凭据。

    更多信息，请参见在 GSSAPI 模式下的 Windows 客户端命令。

+   `--plugin-dir=*`dir_name`*`

    | 命令行格式 | `--plugin-dir=dir_name` |
    | --- | --- |
    | 类型 | 目录名称 |

    查找插件的目录。如果使用`--default-auth`选项指定了一个认证插件但**mysql**找不到它，请指定此选项。参见第 8.2.17 节，“可插拔认证”。

+   `--port=*`port_num`*`, `-P *`port_num`*`

    | 命令行格式 | `--port=port_num` |
    | --- | --- |
    | 类型 | 数字 |
    | 默认值 | `3306` |

    用于 TCP/IP 连接的端口号。

+   `--print-defaults`

    | 命令行格式 | `--print-defaults` |
    | --- | --- |

    打印程序名称以及从选项文件中获取的所有选项。

    有关此选项和其他选项文件选项的更多信息，请参见第 6.2.2.3 节，“影响选项文件处理的命令行选项”。

+   `--prompt=*`format_str`*`

    | 命令行格式 | `--prompt=format_str` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `mysql>` |

    将提示设置为指定的格式。默认值为`mysql>`。提示可以包含的特殊序列在第 6.5.1.2 节，“mysql 客户端命令”中描述。

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

    不要缓存每个查询结果，按接收顺序打印每行。如果输出被暂停，这可能会减慢服务器速度。使用此选项，**mysql** 不会使用历史文件。

    默认情况下，**mysql** 在生成任何输出之前会获取所有结果行；在存储这些行时，它会从每一列的实际值中依次计算出最大列长度。在打印输出时，它会使用这个最大值来格式化输出。当指定 `--quick` 时，**mysql** 在开始之前没有行来计算长度，因此使用最大长度。在下面的示例中，表 `t1` 有一个类型为 `BIGINT` - INTEGER, INT, SMALLINT, TINYINT, MEDIUMINT, BIGINT") 的单列，包含 4 行。默认输出宽度为 9 个字符；这个宽度等于返回行中任何列值中的最大字符数（5），再加上用作填充的空格和用作列分隔符的 `|` 字符，每个都是 2 个字符）。使用 `--quick` 选项时的输出宽度为 25 个字符；这等于表示 `-9223372036854775808` 所需的字符数，这是可以存储在（有符号的）`BIGINT` 列中的最长可能值，或者是 19 个字符，再加上用于填充和列分隔符的 4 个字符。可以在这里看到差异：

    ```sql
    $> mysql -t test -e "SELECT * FROM t1"
    +-------+
    | c1    |
    +-------+
    |   100 |
    |  1000 |
    | 10000 |
    |    10 |
    +-------+

    $> mysql --quick -t test -e "SELECT * FROM t1"
    +----------------------+
    | c1                   |
    +----------------------+
    |                  100 |
    |                 1000 |
    |                10000 |
    |                   10 |
    +----------------------+
    ```

+   `--raw`, `-r`

    | 命令行格式 | `--raw` |
    | --- | --- |

    对于表格输出，围绕列的“框”使得一个列值可以与另一个区分开。对于非表格输出（例如在批处理模式下生成的输出或给出 `--batch` 或 `--silent` 选项时生成的输出），特殊字符在输出中被转义，以便可以轻松识别它们。换行符、制表符、`NUL` 和反斜杠分别被写为 `\n`、`\t`、`\0` 和 `\\`。`--raw` 选项禁用了这种字符转义。

    以下示例演示了表格与非表格输出以及使用原始模式禁用转义的情况：

    ```sql
    % mysql
    mysql> SELECT CHAR(92);
    +----------+
    | CHAR(92) |
    +----------+
    | \        |
    +----------+

    % mysql -s
    mysql> SELECT CHAR(92);
    CHAR(92)
    \\

    % mysql -s -r
    mysql> SELECT CHAR(92);
    CHAR(92)
    \
    ```

+   `--reconnect`

    | 命令行格式 | `--reconnect` |
    | --- | --- |
    | 禁用者 | `skip-reconnect` |

    如果与服务器的连接丢失，自动尝试重新连接。每次连接丢失时都会尝试重新连接一次。要抑制重新连接行为，请使用 `--skip-reconnect`。

+   `--safe-updates`, `--i-am-a-dummy`, `-U`

    | 命令行格式 | `--safe-updates``--i-am-a-dummy` |
    | --- | --- |
    | 类型 | 布尔值 |
    | 默认值 | `FALSE` |

    如果启用此选项，则不使用 `WHERE` 子句中的键或 `LIMIT` 子句的 `UPDATE` 和 `DELETE` 语句将产生错误。此外，对于产生（或估计产生）非常大结果集的 `SELECT` 语句也会施加限制。如果在选项文件中设置了此选项，则可以在命令行上使用 `--skip-safe-updates` 来覆盖它。有关此选项的更多信息，请参见 使用安全更新模式 (--safe-updates)")。

+   `--select-limit=*`value`*`

    | 命令行格式 | `--select-limit=value` |
    | --- | --- |
    | 类型 | 数值 |
    | 默认值 | `1000` |

    当使用 `--safe-updates` 时，`SELECT` 语句的自动限制。（默认值为 1,000）。

+   `--server-public-key-path=*`file_name`*`

    | 命令行格式 | `--server-public-key-path=file_name` |
    | --- | --- |
    | 类型 | 文件名 |

    PEM 格式文件路径名，其中包含服务器所需的客户端端公钥的副本，用于 RSA 密钥对密码交换。此选项适用于使用 `sha256_password` 或 `caching_sha2_password` 认证插件进行身份验证的客户端。对于不使用这些插件进行身份验证的帐户，此选项将被忽略。如果不使用基于 RSA 的密码交换，例如客户端使用安全连接连接到服务器时，此选项也将被忽略。

    如果给定 `--server-public-key-path=*`file_name`*` 并指定有效的公钥文件，则优先于 `--get-server-public-key`。

    对于 `sha256_password`，此选项仅在使用 OpenSSL 构建 MySQL 时适用。

    有关 `sha256_password` 和 `caching_sha2_password` 插件的信息，请参见 第 8.4.1.3 节，“SHA-256 可插拔认证” 和 第 8.4.1.2 节，“缓存 SHA-2 可插拔认证”。

+   `--shared-memory-base-name=*`name`*`

    | 命令行格式 | `--shared-memory-base-name=name` |
    | --- | --- |
    | 特定平台 | Windows |

    在 Windows 上，用于使用共享内存进行到本地服务器的连接的共享内存名称。默认值为`MYSQL`。共享内存名称区分大小写。

    只有在服务器启动时启用了支持共享内存连接的`shared_memory`系统变量时，此选项才适用。

+   `--show-warnings`

    | 命令行格式 | `--show-warnings` |
    | --- | --- |

    导致在每个语句后显示警告（如果有）。此选项适用于交互和批处理模式。

+   `--sigint-ignore`

    | 命令行格式 | `--sigint-ignore` |
    | --- | --- |

    忽略`SIGINT`信号（通常是键入**Control+C**的结果）。

    如果没有此选项，键入**Control+C**会中断当前语句（如果有的话），否则会取消任何部分输入行。

+   `--silent`，`-s`

    | 命令行格式 | `--silent` |
    | --- | --- |

    静默模式。产生较少输出。可以多次使用此选项以产生越来越少的输出。

    此选项会导致非表格输出格式和特殊字符的转义。可以通过使用原始模式来禁用转义；请参阅`--raw`选项的描述。

+   `--skip-column-names`，`-N`

    | 命令行格式 | `--skip-column-names` |
    | --- | --- |

    不在结果中写入列名。使用此选项会导致输出右对齐，如下所示：

    ```sql
    $> echo "SELECT * FROM t1" | mysql -t test
    +-------+
    | c1    |
    +-------+
    | a,c,d |
    | c     |
    +-------+
    $> echo "SELECT * FROM t1" | ./mysql -uroot -Nt test
    +-------+
    | a,c,d |
    |     c |
    +-------+
    ```

+   `--skip-line-numbers`，`-L`

    | 命令行格式 | `--skip-line-numbers` |
    | --- | --- |

    不为错误写入行号。在想要比较包含错误消息的结果文件时很有用。

+   `--socket=*`路径`*`，`-S *`路径`*`

    | 命令行格式 | `--socket={file_name&#124;pipe_name}` |
    | --- | --- |
    | 类型 | 字符串 |

    对于连接到`localhost`的连接，要使用的 Unix 套接字文件，或者在 Windows 上，要使用的命名管道的名称。

    在 Windows 上，只有在服务器启动时启用了支持命名管道连接的`named_pipe`系统变量时，此选项才适用。此外，进行连接的用户必须是由`named_pipe_full_access_group`系统变量指定的 Windows 组的成员。

+   `--ssl*`

    以`--ssl`开头的选项指定是否使用加密连接到服务器，并指示在哪里找到 SSL 密钥和证书。请参阅加密连接的命令选项。

+   `--ssl-fips-mode={OFF|ON|STRICT}`

    | 命令行格式 | `--ssl-fips-mode={OFF&#124;ON&#124;STRICT}` |
    | --- | --- |
    | 已弃用 | 8.0.34 |
    | 类型 | 枚举 |
    | 默认值 | `OFF` |
    | 有效值 | `OFF``ON``STRICT` |

    控制是否在客户端端启用 FIPS 模式。`--ssl-fips-mode` 选项与其他 `--ssl-*`xxx`*` 选项不同，它不用于建立加密连接，而是用于影响允许哪些加密操作。请参阅 第 8.8 节，“FIPS 支持”。 

    允许使用这些 `--ssl-fips-mode` 值：

    +   `OFF`: 禁用 FIPS 模式。

    +   `ON`: 启用 FIPS 模式。

    +   `STRICT`: 启用“严格” FIPS 模式。

    注意

    如果 OpenSSL FIPS 对象模块不可用，则 `--ssl-fips-mode` 的唯一允许值是 `OFF`。在这种情况下，将 `--ssl-fips-mode` 设置为 `ON` 或 `STRICT` 会导致客户端在启动时产生警告并在非 FIPS 模式下运行。

    从 MySQL 8.0.34 开始，此选项已弃用。预计在将来的 MySQL 版本中将其移除。

+   `--syslog`, `-j`

    | 命令行格式 | `--syslog` |
    | --- | --- |

    此选项导致 **mysql** 将交互语句发送到系统日志设施。在 Unix 上，这是 `syslog`；在 Windows 上，这是 Windows 事件日志。记录消息出现的目的地取决于系统。在 Linux 上，目的地通常是 `/var/log/messages` 文件。

    这是在 Linux 上使用 `--syslog` 生成的输出示例。此输出已经过格式化以便阅读；每个记录的消息实际上占据一行。

    ```sql
    Mar  7 12:39:25 myhost MysqlClient[20824]:
      SYSTEM_USER:'oscar', MYSQL_USER:'my_oscar', CONNECTION_ID:23,
      DB_SERVER:'127.0.0.1', DB:'--', QUERY:'USE test;'
    Mar  7 12:39:28 myhost MysqlClient[20824]:
      SYSTEM_USER:'oscar', MYSQL_USER:'my_oscar', CONNECTION_ID:23,
      DB_SERVER:'127.0.0.1', DB:'test', QUERY:'SHOW TABLES;'
    ```

    更多信息，请参阅 第 6.5.1.3 节，“mysql 客户端日志记录”。

+   `--table`, `-t`

    | 命令行格式 | `--table` |
    | --- | --- |

    以表格格式显示输出。这是交互使用的默认设置，但也可用于在批处理模式下生成表格输出。

+   `--tee=*`file_name`*`

    | 命令行格式 | `--tee=file_name` |
    | --- | --- |
    | 类型 | 文件名 |

    将输出的副本附加到给定文件。此选项仅在交互模式下有效。第 6.5.1.2 节，“mysql 客户端命令”，进一步讨论了 tee 文件。

+   `--tls-ciphersuites=*`ciphersuite_list`*`

    | 命令行格式 | `--tls-ciphersuites=ciphersuite_list` |
    | --- | --- |
    | 引入 | 8.0.16 |
    | 类型 | 字符串 |

    用于使用 TLSv1.3 的加密连接的可接受密码套件。该值是一个或多个冒号分隔的密码套件名称列表。可以为此选项命名的密码套件取决于用于编译 MySQL 的 SSL 库。有关详细信息，请参见第 8.3.2 节，“加密连接 TLS 协议和密码”。

    此选项在 MySQL 8.0.16 中添加。

+   `--tls-version=*`protocol_list`*`

    | 命令行格式 | `--tls-version=protocol_list` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值（≥ 8.0.16） | `TLSv1,TLSv1.1,TLSv1.2,TLSv1.3`（OpenSSL 1.1.1 或更高版本）`TLSv1,TLSv1.1,TLSv1.2`（否则） |
    | 默认值（≤ 8.0.15） | `TLSv1,TLSv1.1,TLSv1.2` |

    用于加密连接的可接受 TLS 协议。该值是一个或多个逗号分隔的协议名称列表。可以为此选项命名的协议取决于用于编译 MySQL 的 SSL 库。有关详细信息，请参见第 8.3.2 节，“加密连接 TLS 协议和密码”。

+   `--unbuffered`, `-n`

    | 命令行格式 | `--unbuffered` |
    | --- | --- |

    在每个查询后刷新缓冲区。

+   `--user=*`user_name`*`, `-u *`user_name`*`

    | 命令行格式 | `--user=user_name` |
    | --- | --- |
    | 类型 | 字符串 |

    用于连接到服务器的 MySQL 帐户的用户名。

+   `--verbose`, `-v`

    | 命令行格式 | `--verbose` |
    | --- | --- |

    冗长模式。产生关于程序操作的更多输出。可以多次使用此选项以产生更多输出。（例如，`-v -v -v`即使在批处理模式下也会产生表格输出格式。）

+   `--version`, `-V`

    | 命令行格式 | `--version` |
    | --- | --- |

    显示版本信息并退出。

+   `--vertical`, `-E`

    | 命令行格式 | `--vertical` |
    | --- | --- |

    将查询输出行垂直显示（每列值一行）。如果不使用此选项，可以通过在语句末尾加上`\G`来指定单个语句的垂直输出。

+   `--wait`, `-w`

    | 命令行格式 | `--wait` |
    | --- | --- |

    如果无法建立连接，则等待并重试，而不是中止。

+   `--xml`, `-X`

    | 命令行格式 | `--xml` |
    | --- | --- |

    生成 XML 输出。

    ```sql
    <field name="*column_name*">NULL</field>
    ```

    当使用 `--xml` 与 **mysql** 一起使用时，输出与 **mysqldump** `--xml` 的匹配。详情请参见 第 6.5.4 节，“mysqldump — 数据库备份程序”。

    XML 输出还使用 XML 命名空间，如下所示：

    ```sql
    $> mysql --xml -uroot -e "SHOW VARIABLES LIKE 'version%'"
    <?xml version="1.0"?>

    <resultset statement="SHOW VARIABLES LIKE 'version%'" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
    <row>
    <field name="Variable_name">version</field>
    <field name="Value">5.0.40-debug</field>
    </row>

    <row>
    <field name="Variable_name">version_comment</field>
    <field name="Value">Source distribution</field>
    </row>

    <row>
    <field name="Variable_name">version_compile_machine</field>
    <field name="Value">i686</field>
    </row>

    <row>
    <field name="Variable_name">version_compile_os</field>
    <field name="Value">suse-linux-gnu</field>
    </row>
    </resultset>
    ```

+   `--zstd-compression-level=*`level`*`

    | 命令行格式 | `--zstd-compression-level=#` |
    | --- | --- |
    | 引入版本 | 8.0.18 |
    | 类型 | 整数 |

    用于使用`zstd`压缩算法连接到服务器的连接的压缩级别。允许的级别为 1 到 22，较大的值表示较高级别的压缩。默认的`zstd`压缩级别为 3。压缩级别设置对不使用`zstd`压缩的连接没有影响。

    更多信息，请参见 第 6.2.8 节，“连接压缩控制”。

    此选项在 MySQL 8.0.18 中添加。
