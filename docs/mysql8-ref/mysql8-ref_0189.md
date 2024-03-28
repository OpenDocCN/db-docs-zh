# 6.5.5 mysqlimport — A Data Import Program

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysqlimport.html`](https://dev.mysql.com/doc/refman/8.0/en/mysqlimport.html)

**mysqlimport**客户端提供了一个命令行界面，用于`LOAD DATA` SQL 语句。大多数**mysqlimport**的选项直接对应于`LOAD DATA`语法的子句。请参阅 Section 15.2.9, “LOAD DATA Statement”。

调用**mysqlimport**的方法如下：

```sql
mysqlimport [*options*] *db_name* *textfile1* [*textfile2* ...]
```

对于命令行中命名的每个文本文件，**mysqlimport**会剥去文件名的任何扩展名，并使用结果确定将文件内容导入的表的名称。例如，名为`patient.txt`、`patient.text`和`patient`的文件都将导入到名为`patient`的表中。

**mysqlimport**支持以下选项，可以在命令行或选项文件的`[mysqlimport]`和`[client]`组中指定。有关 MySQL 程序使用的选项文件的信息，请参见 Section 6.2.2.2, “Using Option Files”。

**表 6.16 mysqlimport 选项**

| 选项名称 | 描述 | 引入 | 废弃 |
| --- | --- | --- | --- |
| --bind-address | 使用指定的网络接口连接到 MySQL 服务器 |  |  |
| --character-sets-dir | 字符集所在目录 |  |  |
| --columns | 此选项以逗号分隔的列名列表作为其值 |  |  |
| --compress | 压缩客户端和服务器之间发送的所有信息 |  | 8.0.18 |
| --compression-algorithms | 允许连接到服务器的压缩算法 | 8.0.18 |  |
| --debug | 写入调试日志 |  |  |
| --debug-check | 在程序退出时打印调试信息 |  |  |
| --debug-info | 在程序退出时打印调试信息、内存和 CPU 统计信息 |  |  |
| --default-auth | 要使用的认证插件 |  |  |
| --default-character-set | 指定默认字符集 |  |  |
| --defaults-extra-file | 除了通常的选项文件外，还读取命名的选项文件 |  |  |
| --defaults-file | 仅读取命名的选项文件 |  |  |
| --defaults-group-suffix | 选项组后缀值 |  |  |
| --delete | 导入文本文件之前清空表 |  |  |
| --enable-cleartext-plugin | 启用明文认证插件 |  |  |
| --fields-enclosed-by | 此选项与 LOAD DATA 的相应子句具有相同的含义 |  |  |
| --fields-escaped-by | 此选项与 LOAD DATA 的相应子句具有相同的含义 |  |  |
| --fields-optionally-enclosed-by | 此选项与 LOAD DATA 的相应子句具有相同的含义 |  |  |
| --fields-terminated-by | 此选项与 LOAD DATA 的相应子句具有相同的含义 |  |  |
| --force | 即使发生 SQL 错误也继续 |  |  |
| --get-server-public-key | 从服务器请求 RSA 公钥 |  |  |
| --help | 显示帮助信息并退出 |  |  |
| --host | MySQL 服务器所在的主机 |  |  |
| --ignore | 参见--replace 选项的描述 |  |  |
| --ignore-lines | 忽略数据文件的前 N 行 |  |  |
| --lines-terminated-by | 此选项与 LOAD DATA 的相应子句具有相同的含义 |  |  |
| --local | 从客户端主机本地读取输入文件 |  |  |
| --lock-tables | 在处理任何文本文件之前锁定所有表以进行写入 |  |  |
| --login-path | 从.mylogin.cnf 中读取登录路径选项 |  |  |
| --low-priority | 在加载表时使用 LOW_PRIORITY |  |  |
| --no-defaults | 不读取任何选项文件 |  |  |
| --password | 连接到服务器时要使用的密码 |  |  |
| --password1 | 连接到服务器时要使用的第一个多因素认证密码 | 8.0.27 |  |
| --password2 | 连接到服务器时要使用的第二个多因素身份验证密码 | 8.0.27 |  |
| --password3 | 连接到服务器时要使用的第三个多因素身份验证密码 | 8.0.27 |  |
| --pipe | 使用命名管道连接到服务器（仅限 Windows） |  |  |
| --plugin-dir | 安装插件的目录 |  |  |
| --port | 连接的 TCP/IP 端口号 |  |  |
| --print-defaults | 打印默认选项 |  |  |
| --protocol | 要使用的传输协议 |  |  |
| --replace | --replace 和 --ignore 选项控制对重复现有行的处理 |  |  |
| --server-public-key-path | 包含 RSA 公钥的文件路径名 |  |  |
| --shared-memory-base-name | 共享内存连接的共享内存名称（仅限 Windows） |  |  |
| --silent | 仅在发生错误时产生输出 |  |  |
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
| --tls-ciphersuites | 加密连接的允许 TLSv1.3 密码套件 | 8.0.16 |  |
| --tls-version | 加密连接的允许 TLS 协议 |  |  |
| --use-threads | 并行文件加载的线程数 |  |  |
| --user | 连接到服务器时要使用的 MySQL 用户名 |  |  |
| --verbose | 详细模式 |  |  |
| --version | 显示版本信息并退出 |  |  |
| --zstd-compression-level | 使用 zstd 压缩连接到服务器的压缩级别 | 8.0.18 |  |
| 选项名称 | 描述 | 引入版本 | 已弃用 |

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

    安装字符集的目录。参见 第 12.15 节，“字符集配置”。

+   `--columns=*`column_list`*`, `-c *`column_list`*`

    | 命令行格式 | `--columns=column_list` |
    | --- | --- |

    此选项以逗号分隔的列名列表作为其值。列名的顺序表示如何将数据文件列与表列匹配。

+   `--compress`, `-C`

    | 命令行格式 | `--compress[={OFF&#124;ON}]` |
    | --- | --- |
    | 已弃用 | 8.0.18 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    在可能的情况下压缩客户端和服务器之间发送的所有信息。参见 第 6.2.8 节，“连接压缩控制”。

    从 MySQL 8.0.18 开始，此选项已弃用。预计将在未来的 MySQL 版本中删除。请参见 配置传统连接压缩。

+   `--compression-algorithms=*`value`*`

    | 命令行格式 | `--compression-algorithms=value` |
    | --- | --- |
    | 引入版本 | 8.0.18 |
    | 类型 | 集合 |
    | 默认值 | `uncompressed` |
    | 有效值 | `zlib``zstd``uncompressed` |

    连接到服务器的允许压缩算法。可用的算法与 `protocol_compression_algorithms` 系统变量相同。默认值为 `uncompressed`。

    更多信息，请参见 第 6.2.8 节，“连接压缩控制”。

    此选项是在 MySQL 8.0.18 中添加的。

+   [`--debug[=*`debug_options`*]`](mysqlimport.html#option_mysqlimport_debug), `-# [*`debug_options`*]`

    | 命令行格式 | `--debug[=debug_options]` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `d:t:o` |

    写入调试日志。典型的 *`debug_options`* 字符串是 `d:t:o,*`file_name`*`。默认值为 `d:t:o`。

    只有在使用 `WITH_DEBUG` 构建 MySQL 时才可用。由 Oracle 提供的 MySQL 发行版二进制文件 *不* 使用此选项构建。

+   `--debug-check`

    | 命令行格式 | `--debug-check` |
    | --- | --- |
    | 类型 | 布尔值 |
    | 默认值 | `FALSE` |

    在程序退出时打印一些调试信息。

    只有在使用 `WITH_DEBUG` 构建 MySQL 时才可用。由 Oracle 提供的 MySQL 发行版二进制文件 *不* 使用此选项构建。

+   `--debug-info`

    | 命令行格式 | `--debug-info` |
    | --- | --- |
    | 类型 | 布尔值 |
    | 默认值 | `FALSE` |

    在程序退出时打印调试信息和内存和 CPU 使用统计信息。

    只有在使用 `WITH_DEBUG` 构建 MySQL 时才可用。由 Oracle 提供的 MySQL 发行版二进制文件 *不* 使用此选项构建。

+   `--default-character-set=*`charset_name`*`

    | 命令行格式 | `--default-character-set=charset_name` |
    | --- | --- |
    | 类型 | 字符串 |

    将 *`charset_name`* 用作默认字符集。请参见 第 12.15 节，“字符集配置”。

+   `--default-auth=*`plugin`*`

    | 命令行格式 | `--default-auth=plugin` |
    | --- | --- |
    | 类型 | 字符串 |

    关于使用哪种客户端端身份验证插件的提示。请参见 第 8.2.17 节，“可插拔认证”。

+   `--defaults-extra-file=*`file_name`*`

    | 命令行格式 | `--defaults-extra-file=file_name` |
    | --- | --- |
    | 类型 | 文件名 |

    在全局选项文件之后但（在 Unix 上）在用户选项文件之前读取此选项文件。如果文件不存在或无法访问，将会出现错误。如果 *`file_name`* 不是绝对路径名，则将其解释为相对于当前目录的路径。

    有关此选项文件和其他选项文件选项的附加信息，请参见第 6.2.2.3 节，“影响选项文件处理的命令行选项”。

+   `--defaults-file=*`file_name`*`

    | 命令行格式 | `--defaults-file=file_name` |
    | --- | --- |
    | 类型 | 文件名 |

    仅使用给定的选项文件。如果文件不存在或无法访问，将会出现错误。如果*`file_name`*不是绝对路径名，则将其解释为相对于当前目录的路径。

    例外：即使使用`--defaults-file`，客户端程序也会读取`.mylogin.cnf`。

    有关此选项文件和其他选项文件选项的附加信息，请参见第 6.2.2.3 节，“影响选项文件处理的命令行选项”。

+   `--defaults-group-suffix=*`str`*`

    | 命令行格式 | `--defaults-group-suffix=str` |
    | --- | --- |
    | 类型 | 字符串 |

    不仅读取通常的选项组，还读取具有通常名称和后缀为*`str`*的组。例如，**mysqlimport**通常会读取`[client]`和`[mysqlimport]`组。如果给定此选项作为`--defaults-group-suffix=_other`，**mysqlimport**还会读取`[client_other]`和`[mysqlimport_other]`组。

    有关此选项文件和其他选项文件选项的附加信息，请参见第 6.2.2.3 节，“影响选项文件处理的命令行选项”。

+   `--delete`, `-D`

    | 命令行格式 | `--delete` |
    | --- | --- |

    清空表格后再导入文本文件。

+   `--enable-cleartext-plugin`

    | 命令行格式 | `--enable-cleartext-plugin` |
    | --- | --- |
    | 类型 | 布尔值 |
    | 默认值 | `FALSE` |

    启用`mysql_clear_password`明文认证插件。（参见第 8.4.1.4 节，“客户端明文插件认证”。）

+   `--fields-terminated-by=...`, `--fields-enclosed-by=...`, `--fields-optionally-enclosed-by=...`, `--fields-escaped-by=...`

    | 命令行格式 | `--fields-terminated-by=string` |
    | --- | --- |
    | 类型 | 字符串 |
    | 命令行格式 | `--fields-enclosed-by=string` |
    | 类型 | 字符串 |
    | 命令行格式 | `--fields-optionally-enclosed-by=string` |
    | 类型 | 字符串 |
    | 命令行格式 | `--fields-escaped-by` |
    | 类型 | 字符串 |

    这些选项的含义与`LOAD DATA`的相应子句相同。请参阅第 15.2.9 节，“LOAD DATA Statement”。

+   `--force`, `-f`

    | 命令行格式 | `--force` |
    | --- | --- |

    忽略错误。例如，如果文本文件的表不存在，则继续处理任何剩余文件。如果表不存在，则没有`--force`，**mysqlimport**会退出。

+   `--get-server-public-key`

    | 命令行格式 | `--get-server-public-key` |
    | --- | --- |
    | 类型 | 布尔值 |

    从服务器请求用于基于 RSA 密钥对的密码交换的公钥。此选项适用于使用`caching_sha2_password`认证插件进行身份验证的客户端。对于该插件，除非请求，否则服务器不会发送公钥。对于不使用该插件进行身份验证的帐户，此选项将被忽略。如果不使用基于 RSA 的密码交换，例如客户端使用安全连接连接到服务器时，此选项也将被忽略。

    如果`--server-public-key-path=*`file_name`*`已给出并指定了有效的公钥文件，则它优先于`--get-server-public-key`。

    有关`caching_sha2_password`插件的信息，请参阅第 8.4.1.2 节，“Caching SHA-2 Pluggable Authentication”。

+   `--host=*`host_name`*`, `-h *`host_name`*`

    | 命令行格式 | `--host=host_name` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `localhost` |

    将数据导入到给定主机上的 MySQL 服务器。默认主机是`localhost`。

+   `--ignore`, `-i`

    | 命令行格式 | `--ignore` |
    | --- | --- |

    请参阅`--replace`选项的描述。

+   `--ignore-lines=*`N`*`

    | 命令行格式 | `--ignore-lines=#` |
    | --- | --- |
    | 类型 | 数字 |

    忽略数据文件的第一个*`N`*行。

+   `--lines-terminated-by=...`

    | 命令行格式 | `--lines-terminated-by=string` |
    | --- | --- |
    | 类型 | 字符串 |

    此选项与`LOAD DATA`的相应子句具有相同的含义。例如，要导入以回车/换行对终止的 Windows 文件，请使用`--lines-terminated-by="\r\n"`。（根据您的命令解释器的转义约定，您可能需要双反斜杠。）请参见第 15.2.9 节，“LOAD DATA 语句”。

+   `--local`, `-L`

    | 命令行格式 | `--local` |
    | --- | --- |
    | 类型 | 布尔值 |
    | 默认值 | `FALSE` |

    默认情况下，文件由服务器在服务器主机上读取。使用此选项，**mysqlimport**在客户端主机上本地读取输入文件。

    在**mysqlimport**中成功使用`LOCAL`加载操作还需要服务器允许本地加载；请参见第 8.1.6 节，“LOAD DATA LOCAL 的安全注意事项”

+   `--lock-tables`, `-l`

    | 命令行格式 | `--lock-tables` |
    | --- | --- |

    在处理任何文本文件之前，将*所有*表锁定为写入状态。这确保所有表在服务器上同步。

+   `--login-path=*`name`*`

    | 命令行格式 | `--login-path=name` |
    | --- | --- |
    | 类型 | 字符串 |

    从`.mylogin.cnf`登录路径文件中的命名登录路径读取选项。 “登录路径”是一个包含指定要连接到哪个 MySQL 服务器以及要作为哪个帐户进行身份验证的选项的选项组。要创建或修改登录路径文件，请使用**mysql_config_editor**实用程序。请参见第 6.6.7 节，“mysql_config_editor — MySQL 配置实用程序”。

    有关此选项文件和其他选项文件选项的更多信息，请参见第 6.2.2.3 节，“影响选项文件处理的命令行选项”。

+   `--low-priority`

    | 命令行格式 | `--low-priority` |
    | --- | --- |

    在加载表时使用`LOW_PRIORITY`。这仅影响仅使用表级锁定的存储引擎（如`MyISAM`，`MEMORY`和`MERGE`）。

+   `--no-defaults`

    | 命令行格式 | `--no-defaults` |
    | --- | --- |

    不要读取任何选项文件。如果由于从选项文件中读取未知选项而导致程序启动失败，可以使用`--no-defaults`来防止读取它们。

    唯一的例外是，如果存在`.mylogin.cnf`文件，则在所有情况下都会读取该文件。这使得可以以比在命令行上更安全的方式指定密码，即使使用`--no-defaults`。要创建`.mylogin.cnf`，请使用**mysql_config_editor**实用程序。参见第 6.6.7 节，“mysql_config_editor — MySQL 配置实用程序”。

    有关此选项和其他选项文件选项的其他信息，请参见第 6.2.2.3 节，“影响选项文件处理的命令行选项”。

+   [`--password[=*`password`*]`](mysqlimport.html#option_mysqlimport_password), `-p[*`password`*]`

    | 命令行格式 | `--password[=password]` |
    | --- | --- |
    | 类型 | 字符串 |

    用于连接到服务器的 MySQL 帐户的密码。密码值是可选的。如果未提供，**mysqlimport**会提示输入密码。如果提供，`--password=`或`-p`后面必须*没有空格*。如果未指定密码选项，则默认情况下不发送密码。

    在命令行上指定密码应被视为不安全。为了避免在命令行上提供密码，请使用选项文件。参见第 8.1.2.1 节，“密码安全的最终用户指南”。

    要明确指定没有密码，并且**mysqlimport**不应提示输入密码，请使用`--skip-password`选项。

+   [`--password1[=*`pass_val`*]`](mysqlimport.html#option_mysqlimport_password1)

    用于连接到服务器的 MySQL 帐户的多因素认证因素 1 的密码。密码值是可选的。如果未提供，**mysqlimport**会提示输入密码。如果提供，`--password1=`后面必须*没有空格*。如果未指定密码选项，则默认情况下不发送密码。

    在命令行上指定密码应被视为不安全。为了避免在命令行上提供密码，请使用选项文件。参见第 8.1.2.1 节，“密码安全的最终用户指南”。

    要明确指定没有密码，并且**mysqlimport**不应提示输入密码，请使用`--skip-password1`选项。

    `--password1`和`--password`是同义词，`--skip-password1`和`--skip-password`也是同义词。

+   [`--password2[=*`pass_val`*]`](mysqlimport.html#option_mysqlimport_password2)

    用于连接到服务器的 MySQL 帐户的多因素认证因子 2 的密码。此选项的语义类似于`--password1`的语义；有关详细信息，请参阅该选项的描述。

+   [`--password3[=*`pass_val`*]`](mysqlimport.html#option_mysqlimport_password3)

    用于连接到服务器的 MySQL 帐户的多因素认证因子 3 的密码。此选项的语义类似于`--password1`的语义；有关详细信息，请参阅该选项的描述。

+   `--pipe`, `-W`

    | 命令行格式 | `--pipe` |
    | --- | --- |
    | 类型 | 字符串 |

    在 Windows 上，使用命名管道连接到服务器。此选项仅适用于服务器启动时启用了`named_pipe`系统变量以支持命名管道连接的情况。此外，进行连接的用户必须是由`named_pipe_full_access_group`系统变量指定的 Windows 组的成员。

+   `--plugin-dir=*`dir_name`*`

    | 命令行格式 | `--plugin-dir=dir_name` |
    | --- | --- |
    | 类型 | 目录名称 |

    查找插件的目录。如果使用`--default-auth`选项指定身份验证插件但**mysqlimport**找不到它，则指定此选项。参见第 8.2.17 节，“可插拔认证”。

+   `--port=*`port_num`*`, `-P *`port_num`*`

    | 命令行格式 | `--port=port_num` |
    | --- | --- |
    | 类型 | 数字 |
    | 默认值 | `3306` |

    对于 TCP/IP 连接，要使用的端口号。

+   `--print-defaults`

    | 命令行格式 | `--print-defaults` |
    | --- | --- |

    打印程序名称以及从选项文件中获取的所有选项。

    有关此选项和其他选项文件选项的其他信息，请参见第 6.2.2.3 节，“影响选项文件处理的命令行选项”。

+   `--protocol={TCP|SOCKET|PIPE|MEMORY}`

    | 命令行格式 | `--protocol=type` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `[查看文本]` |
    | 有效值 | `TCP``SOCKET``PIPE``MEMORY` |

    用于连接到服务器的传输协议。当其他连接参数通常导致使用不希望使用的协议时，此选项很有用。有关允许的值的详细信息，请参见第 6.2.7 节，“连接传输协议”。

+   `--replace`，`-r`

    | 命令行格式 | `--replace` |
    | --- | --- |

    `--replace`和`--ignore`选项控制处理具有唯一键值的输入行重复现有行的方式。如果指定`--replace`，则新行将替换具有相同唯一键值的现有行。如果指定`--ignore`，则将跳过具有唯一键值上存在的现有行的输入行。如果两个选项都未指定，则在发现重复键值时会发生错误，并且将忽略文本文件的其余部分。

+   `--server-public-key-path=*`file_name`*`

    | 命令行格式 | `--server-public-key-path=file_name` |
    | --- | --- |
    | 类型 | 文件名 |

    PEM 格式文件的路径名，其中包含服务器所需的用于 RSA 密钥对密码交换的客户端端公钥的副本。此选项适用于使用`sha256_password`或`caching_sha2_password`认证插件进行身份验证的客户端。对于不使用这些插件进行身份验证的帐户，此选项将被忽略。如果不使用基于 RSA 的密码交换，例如客户端使用安全连接连接到服务器时，此选项也将被忽略。

    如果给定`--server-public-key-path=*`file_name`*`并指定有效的公钥文件，则它将优先于`--get-server-public-key`。

    对于`sha256_password`，只有当 MySQL 使用 OpenSSL 构建时，此选项才适用。

    有关`sha256_password`和`caching_sha2_password`插件的信息，请参见第 8.4.1.3 节，“SHA-256 可插拔认证”和第 8.4.1.2 节，“缓存 SHA-2��插拔认证”。

+   `--shared-memory-base-name=*`name`*`

    | 命令行格式 | `--shared-memory-base-name=name` |
    | --- | --- |
    | 平台特定 | Windows |

    在 Windows 上，用于使用共享内存连接到本地服务器的共享内存名称。默认值为`MYSQL`。共享内存名称区分大小写。

    此选项仅在服务器启动时启用了支持共享内存连接的`shared_memory`系统变量时才适用。

+   `--silent`, `-s`

    | 命令行格式 | `--silent` |
    | --- | --- |

    静默模式。仅在发生错误时产生输出。

+   `--socket=*`path`*`, `-S *`path`*`

    | 命令行格式 | `--socket={file_name | pipe_name}` |
    | --- | --- | --- |
    | 类型 | 字符串 |

    对于连接到`localhost`的连接，要使用的 Unix 套接字文件，或者在 Windows 上要使用的命名管道的名称。

    在 Windows 上，只有在服务器启动时启用了支持命名管道连接的`named_pipe`系统变量时，此选项才适用。此外，进行连接的用户必须是由`named_pipe_full_access_group`系统变量指定的 Windows 组的成员。

+   `--ssl*`

    以`--ssl`开头的选项指定是否使用加密连接到服务器，并指示 SSL 密钥和证书的位置。请参阅加密连接的命令选项。

+   `--ssl-fips-mode={OFF|ON|STRICT}`

    | 命令行格式 | `--ssl-fips-mode={OFF | ON | STRICT}` |
    | --- | --- | --- | --- |
    | 弃用 | 8.0.34 |
    | 类型 | 枚举 |
    | 默认值 | `OFF` |
    | 有效值 | `OFF``ON``STRICT` |

    控制是否在客户端端启用 FIPS 模式。`--ssl-fips-mode`选项与其他`--ssl-*`xxx`*`选项不同，它不用于建立加密连接，而是影响允许哪些加密操作。请参阅第 8.8 节，“FIPS 支持”。

    允许使用这些`--ssl-fips-mode`值：

    +   `OFF`: 禁用 FIPS 模式。

    +   `ON`: 启用 FIPS 模式。

    +   `STRICT`: 启用“严格”FIPS 模式。

    注意

    如果 OpenSSL FIPS 对象模块不可用，则`--ssl-fips-mode`的唯一允许值为`OFF`。在这种情况下，将`--ssl-fips-mode`设置为`ON`或`STRICT`会导致客户端在启动时产生警告并在非 FIPS 模式下运行。

    从 MySQL 8.0.34 开始，此选项已被弃用。预计在未来的 MySQL 版本中将会移除。

+   `--tls-ciphersuites=*`ciphersuite_list`*`

    | 命令行格式 | `--tls-ciphersuites=ciphersuite_list` |
    | --- | --- |
    | 引入版本 | 8.0.16 |
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

+   `--user=*`user_name`*`，`-u *`user_name`*`

    | 命令行格式 | `--user=user_name,` |
    | --- | --- |
    | 类型 | 字符串 |

    用于连接到服务器的 MySQL 帐户的用户名。

+   `--use-threads=*`N`*`

    | 命令行格式 | `--use-threads=#` |
    | --- | --- |
    | 类型 | 数字 |

    并行加载文件，使用*`N`*个线程。

+   `--verbose`，`-v`

    | 命令行格式 | `--verbose` |
    | --- | --- |

    详细模式。打印程序执行的更多信息。

+   `--version`，`-V`

    | 命令行格式 | `--version` |
    | --- | --- |

    显示版本信息并退出。

+   `--zstd-compression-level=*`level`*`

    | 命令行格式 | `--zstd-compression-level=#` |
    | --- | --- |
    | 引入版本 | 8.0.18 |
    | 类型 | 整数 |

    用于使用`zstd`压缩算法连接到服务器的连接的压缩级别。允许的级别为 1 到 22，较大的值表示较高级别的压缩。默认的`zstd`压缩级别为 3。压缩级别设置对不使用`zstd`压缩的连接没有影响。

    更多信息，请参见第 6.2.8 节，“连接压缩控制”。

    此选项是在 MySQL 8.0.18 中添加的。

这是一个演示如何使用**mysqlimport**的示例会话：

```sql
$> mysql -e 'CREATE TABLE imptest(id INT, n VARCHAR(30))' test
$> ed
a
100     Max Sydow
101     Count Dracula
.
w imptest.txt
32
q
$> od -c imptest.txt
0000000   1   0   0  \t   M   a   x       S   y   d   o   w  \n   1   0
0000020   1  \t   C   o   u   n   t       D   r   a   c   u   l   a  \n
0000040
$> mysqlimport --local test imptest.txt
test.imptest: Records: 2  Deleted: 0  Skipped: 0  Warnings: 0
$> mysql -e 'SELECT * FROM imptest' test
+------+---------------+
| id   | n             |
+------+---------------+
|  100 | Max Sydow     |
|  101 | Count Dracula |
+------+---------------+
```
