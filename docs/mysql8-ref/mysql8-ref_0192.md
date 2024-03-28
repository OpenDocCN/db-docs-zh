# 6.5.8 mysqlslap — A Load Emulation Client

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysqlslap.html`](https://dev.mysql.com/doc/refman/8.0/en/mysqlslap.html)

**mysqlslap**是一个诊断程序，旨在模拟 MySQL 服务器的客户端负载，并报告每个阶段的时间。它的工作方式就像多个客户端正在访问服务器。

像这样调用**mysqlslap**：

```sql
mysqlslap [*options*]
```

一些选项，如`--create`或`--query`使您能够指定包含 SQL 语句的字符串或包含语句的文件。如果指定文件，默认情况下每行必须包含一个语句。（也就是说，隐式语句分隔符是换行符。）使用`--delimiter`选项指定不同的分隔符，这使您能够指定跨越多行的语句或将多个语句放在一行上。您不能在文件中包含注释；**mysqlslap**无法理解它们。

**mysqlslap**分为三个阶段：

1.  创建模式、表，以及可选的任何存储程序或用于测试的数据。此阶段使用单个客户端连接。

1.  运行负载测试。此阶段可以使用多个客户端连接。

1.  清理（断开连接，如果指定了则删除表）。此阶段使用单个客户端连接。

示例：

提供您自己的创建和查询 SQL 语句，使用 50 个客户端查询，每个客户端进行 200 次选择（在一行上输入命令）：

```sql
mysqlslap --delimiter=";"
  --create="CREATE TABLE a (b int);INSERT INTO a VALUES (23)"
  --query="SELECT * FROM a" --concurrency=50 --iterations=200
```

让**mysqlslap**构建具有两个`INT` - INTEGER, INT, SMALLINT, TINYINT, MEDIUMINT, BIGINT")列和三个`VARCHAR`列的表的查询 SQL 语句。使用五个客户端，每个客户端查询 20 次。不要创建表或插入数据（即使用先前测试的模式和数据）：

```sql
mysqlslap --concurrency=5 --iterations=20
  --number-int-cols=2 --number-char-cols=3
  --auto-generate-sql
```

告诉程序从指定文件加载创建、插入和查询 SQL 语句，其中`create.sql`文件包含多个以`';'`分隔的表创建语句和多个以`';'`分隔的插入语句。`--query`文件应包含多个以`';'`分隔的查询。运行所有加载语句，然后使用五个客户端（每个客户端运行五次）运行查询文件中的所有查询：

```sql
mysqlslap --concurrency=5
  --iterations=5 --query=query.sql --create=create.sql
  --delimiter=";"
```

**mysqlslap** 支持以下选项，可以在命令行或选项文件的 `[mysqlslap]` 和 `[client]` 组中指定。有关 MySQL 程序使用的选项文件的信息，请参见 Section 6.2.2.2, “Using Option Files”。

**表 6.19 mysqlslap 选项**

| 选项名称 | 描述 | 引入版本 | 废弃版本 |
| --- | --- | --- | --- |
| --auto-generate-sql | 当未提供文件或使用命令选项时自动生成 SQL 语句 |  |  |
| --auto-generate-sql-add-autoincrement | 为自动生成的表添加 AUTO_INCREMENT 列 |  |  |
| --auto-generate-sql-execute-number | 指定自动生成的查询数量 |  |  |
| --auto-generate-sql-guid-primary | 为自动生成的表添加基于 GUID 的主键 |  |  |
| --auto-generate-sql-load-type | 指定测试负载类型 |  |  |
| --auto-generate-sql-secondary-indexes | 指定自动生成的表中要添加多少个次要索引 |  |  |
| --auto-generate-sql-unique-query-number | 为自动测试生成多少不同的查询 |  |  |
| --auto-generate-sql-unique-write-number | 为 --auto-generate-sql-write-number 生成多少不同的查询 |  |  |
| --auto-generate-sql-write-number | 每个线程执行的行插入数量 |  |  |
| --commit | 在提交之前执行多少个语句 |  |  |
| --compress | 在客户端和服务器之间发送的所有信息进行压缩 |  | 8.0.18 |
| --compression-algorithms | 与服务器连接时允许的压缩算法 | 8.0.18 |  |
| --concurrency | 发出 SELECT 语句时模拟的客户端数量 |  |  |
| --create | 包含用于创建表的语句的文件或字符串 |  |  |
| --create-schema | 运行测试的模式 |  |  |
| --csv | 以逗号分隔值格式生成输出 |  |  |
| --debug | 写入调试日志 |  |  |
| --debug-check | 程序退出时打印调试信息 |  |  |
| --debug-info | 在程序退出时打印调试信息、内存和 CPU 统计信息 |  |  |
| --default-auth | 要使用的身份验证插件 |  |  |
| --defaults-extra-file | 除了通常的选项文件外，读取指定的选项文件 |  |  |
| --defaults-file | 仅读取指定的选项文件 |  |  |
| --defaults-group-suffix | 选项组后缀值 |  |  |
| --delimiter | SQL 语句中使用的分隔符 |  |  |
| --detach | 每个 N 条语句后分离（关闭并重新打开）每个连接 |  |  |
| --enable-cleartext-plugin | 启用明文身份验证插件 |  |  |
| --engine | 用于创建表的存储引擎 |  |  |
| --get-server-public-key | 从服务器请求 RSA 公钥 |  |  |
| --help | 显示帮助信息并退出 |  |  |
| --host | MySQL 服务器所在的主机 |  |  |
| --iterations | 运行测试的次数 |  |  |
| --login-path | 从 .mylogin.cnf 文件中读取登录路径选项 |  |  |
| --no-defaults | 不读取任何选项文件 |  |  |
| --no-drop | 不删除测试运行期间创建的任何模式 |  |  |
| --number-char-cols | 如果指定了 --auto-generate-sql，使用的 VARCHAR 列数 |  |  |
| --number-int-cols | 如果指定了 --auto-generate-sql，使用的 INT 列数 |  |  |
| --number-of-queries | 将每个客户端限制为大约这个查询数 |  |  |
| --only-print | 不连接到数据库。mysqlslap 只打印它将要执行的操作 |  |  |
| --password | 连接到服务器时使用的密码 |  |  |
| --password1 | 连接到服务器时使用的第一个多因素身份验证密码 | 8.0.27 |  |
| --password2 | 连接到服务器时使用的第二个多因素身份验证密码 | 8.0.27 |  |
| --password3 | 连接到服务器时要使用的第三个多因素身份验证密码 | 8.0.27 |  |
| --pipe | 使用命名管道连接到服务器（仅限 Windows） |  |  |
| --plugin-dir | 安装插件的目录 |  |  |
| --port | 连接的 TCP/IP 端口号 |  |  |
| --post-query | 包含在测试完成后执行的语句的文件或字符串 |  |  |
| --post-system | 在测试完成后使用 system() 执行的字符串 |  |  |
| --pre-query | 在运行测试之前执行的语句的文件或字符串 |  |  |
| --pre-system | 在运行测试之前使用 system() 执行的字符串 |  |  |
| --print-defaults | 打印默认选项 |  |  |
| --protocol | 要使用的传输协议 |  |  |
| --query | 包含用于检索数据的 SELECT 语句的文件或字符串 |  |  |
| --server-public-key-path | 包含 RSA 公钥的文件的路径名 |  |  |
| --shared-memory-base-name | 共享内存连接的共享内存名称（仅限 Windows） |  |  |
| --silent | 静默模式 |  |  |
| --socket | 要使用的 Unix 套接字文件或 Windows 命名管道 |  |  |
| --sql-mode | 为客户端会话设置 SQL 模式 |  |  |
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
| --user | 连接到服务器时要使用的 MySQL 用户名 |  |  |
| --verbose | 详细模式 |  |  |
| --version | 显示版本信息并退出 |  |  |
| --zstd-compression-level | 使用 zstd 压缩连接到服务器的压缩级别 | 8.0.18 |  |
| 选项名称 | 描述 | 引入 | 废弃 |

+   `--help`, `-?`

    | 命令行格式 | `--help` |
    | --- | --- |

    显示帮助��息并退出。

+   `--auto-generate-sql`, `-a`

    | 命令行格式 | `--auto-generate-sql` |
    | --- | --- |
    | 类型 | 布尔值 |
    | 默认值 | `FALSE` |

    在文件中未提供 SQL 语句或使用命令选项时自动生成 SQL 语句。

+   `--auto-generate-sql-add-autoincrement`

    | 命令行格式 | `--auto-generate-sql-add-autoincrement` |
    | --- | --- |
    | 类型 | 布尔值 |
    | 默认值 | `FALSE` |

    向自动生成的表添加一个 `AUTO_INCREMENT` 列。

+   `--auto-generate-sql-execute-number=*`N`*`

    | 命令行格式 | `--auto-generate-sql-execute-number=#` |
    | --- | --- |
    | 类型 | 数值 |

    指定自动生成多少个查询。

+   `--auto-generate-sql-guid-primary`

    | 命令行格式 | `--auto-generate-sql-guid-primary` |
    | --- | --- |
    | 类型 | 布尔值 |
    | 默认值 | `FALSE` |

    向自动生成的表添加基于 GUID 的主键。

+   `--auto-generate-sql-load-type=*`type`*`

    | 命令行格式 | `--auto-generate-sql-load-type=type` |
    | --- | --- |
    | 类型 | 枚举 |
    | 默认值 | `mixed` |
    | 有效值 | `read``write``key``update``mixed` |

    指定测试负载类型。允许的值为 `read`（扫描表）、`write`（插入表）、`key`（读取主键）、`update`（更新主键）或 `mixed`（一半插入，一半扫描选择）。默认为 `mixed`。

+   `--auto-generate-sql-secondary-indexes=*`N`*`

    | 命令行格式 | `--auto-generate-sql-secondary-indexes=#` |
    | --- | --- |
    | 类型 | 数值 |
    | 默认值 | `0` |

    指定自动生成表中要添加多少个辅助索引。默认情况下，不添加任何索引。

+   `--auto-generate-sql-unique-query-number=*`N`*`

    | 命令行格式 | `--auto-generate-sql-unique-query-number=#` |
    | --- | --- |
    | 类型 | 数字 |
    | 默认值 | `10` |

    为自动测试生成多少个不同的查询。例如，如果运行一个执行 1000 次选择的`key`测试，您可以使用此选项并设置值为 1000 来运行 1000 个唯一查询，或者设置为 50 来执行 50 个不同的选择。默认值为 10。

+   `--auto-generate-sql-unique-write-number=*`N`*`

    | 命令行格式 | `--auto-generate-sql-unique-write-number=#` |
    | --- | --- |
    | 类型 | 数字 |
    | 默认值 | `10` |

    为`--auto-generate-sql-write-number`生成多少个不同的查询。默认值为 10。

+   `--auto-generate-sql-write-number=*`N`*`

    | 命令行格式 | `--auto-generate-sql-write-number=#` |
    | --- | --- |
    | 类型 | 数字 |
    | 默认值 | `100` |

    要执行多少行插入。默认值为 100。

+   `--commit=*`N`*`

    | 命令行格式 | `--commit=#` |
    | --- | --- |
    | 类型 | 数字 |
    | 默认值 | `0` |

    在提交之前要执行多少条语句。默认值为 0（不执行提交）。

+   `--compress`, `-C`

    | 命令行格式 | `--compress[={OFF&#124;ON}]` |
    | --- | --- |
    | 已弃用 | 8.0.18 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    如果可能，压缩客户端和服务器之间发送的所有信息。请参阅第 6.2.8 节，“连接压缩控制”。

    从 MySQL 8.0.18 开始，此选项已弃用。预计将在将来的 MySQL 版本中删除。请参阅配置传统连接压缩。

+   `--compression-algorithms=*`value`*`

    | 命令行格式 | `--compression-algorithms=value` |
    | --- | --- |
    | 引入版本 | 8.0.18 |
    | 类型 | 设置 |
    | 默认值 | `uncompressed` |
    | 有效值 | `zlib``zstd``uncompressed` |

    用于与服务器连接的允许的压缩算法。可用的算法与`protocol_compression_algorithms`系统变量相同。默认值为`uncompressed`。

    有关更多信息，请参阅第 6.2.8 节，“连接压缩控制”。

    此选项在 MySQL 8.0.18 中添加。

+   `--concurrency=*`N`*`, `-c *`N`*`

    | 命令行格式 | `--concurrency=#` |
    | --- | --- |
    | 类型 | 数值 |

    要模拟的并行客户端数量。

+   `--create=*`value`*`

    | 命令行格式 | `--create=value` |
    | --- | --- |
    | 类型 | 字符串 |

    用于创建表的语句的文件或字符串。

+   `--create-schema=*`value`*`

    | 命令行格式 | `--create-schema=value` |
    | --- | --- |
    | 类型 | 字符串 |

    要运行测试的模式。

    注意

    如果还提供了 `--auto-generate-sql` 选项，**mysqlslap** 在测试运行结束时会删除模式。为了避免这种情况，也要使用 `--no-drop` 选项。

+   [`--csv[=*`file_name`*]`](mysqlslap.html#option_mysqlslap_csv)

    | 命令行格式 | `--csv=[file]` |
    | --- | --- |
    | 类型 | 文件名 |

    以逗号分隔值格式生成输出。输出到指定文件，如果没有给出文件，则输出到标准输出。

+   [`--debug[=*`debug_options`*]`](mysqlslap.html#option_mysqlslap_debug), `-# [*`debug_options`*]`

    | 命令行格式 | `--debug[=debug_options]` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `d:t:o,/tmp/mysqlslap.trace` |

    写入调试日志。典型的 *`debug_options`* 字符串是 `d:t:o,*`file_name`*`。默认值是 `d:t:o,/tmp/mysqlslap.trace`。

    此选项仅在使用 `WITH_DEBUG` 构建 MySQL 时可用。由 Oracle 提供的 MySQL 发行版二进制文件 *不* 使用此选项构建。

+   `--debug-check`

    | 命令行格式 | `--debug-check` |
    | --- | --- |
    | 类型 | 布尔值 |
    | 默认值 | `FALSE` |

    在程序退出时打印一些调试信息。

    此选项仅在使用 `WITH_DEBUG` 构建 MySQL 时可用。由 Oracle 提供的 MySQL 发行版二进制文件 *不* 使用此选项构建。

+   `--debug-info`, `-T`

    | 命令行格式 | `--debug-info` |
    | --- | --- |
    | 类型 | 布尔值 |
    | 默认值 | `FALSE` |

    在程序退出时打印调试信息以及内存和 CPU 使用统计信息。

    此选项仅在使用 `WITH_DEBUG` 构建 MySQL 时可用。由 Oracle 提供的 MySQL 发行版二进制文件 *不* 使用此选项构建。

+   `--default-auth=*`plugin`*`

    | 命令行格式 | `--default-auth=plugin` |
    | --- | --- |
    | 类型 | 字符串 |

    关于使用哪个客户端端身份验证插件的提示。参见 第 8.2.17 节，“可插拔认证”。

+   `--defaults-extra-file=*`file_name`*`

    | 命令行格式 | `--defaults-extra-file=file_name` |
    | --- | --- |
    | 类型 | 文件名 |

    在全局选项文件之后（在 Unix 上）但在用户选项文件之前读取此选项文件。如果文件不存在或无法访问，将出现错误。如果*`file_name`*不是绝对路径名，则将其解释为相对于当前目录。

    有关此选项和其他选项文件选项的更多信息，请参见第 6.2.2.3 节，“影响选项文件处理的命令行选项”。

+   `--defaults-file=*`file_name`*`

    | 命令行格式 | `--defaults-file=file_name` |
    | --- | --- |
    | 类型 | 文件名 |

    仅使用给定的选项文件。如果文件不存在或无法访问，将出现错误。如果*`file_name`*不是绝对路径名，则将其解释为相对于当前目录。

    例外：即使使用`--defaults-file`，客户端程序也会读取`.mylogin.cnf`。

    有关此选项和其他选项文件选项的更多信息，请参见第 6.2.2.3 节，“影响选项文件处理的命令行选项”��

+   `--defaults-group-suffix=*`str`*`

    | 命令行格式 | `--defaults-group-suffix=str` |
    | --- | --- |
    | 类型 | 字符串 |

    不仅读取常规选项组，还读取具有常规名称和*`str`*后缀的选项组。例如，**mysqlslap**通常会读取`[client]`和`[mysqlslap]`组。如果给定此选项作为`--defaults-group-suffix=_other`，**mysqlslap**还会读取`[client_other]`和`[mysqlslap_other]`组。

    有关此选项和其他选项文件选项的更多信息，请参见第 6.2.2.3 节，“影响选项文件处理的命令行选项”。

+   `--delimiter=*`str`*`, `-F *`str`*`

    | 命令行格式 | `--delimiter=str` |
    | --- | --- |
    | 类型 | 字符串 |

    SQL 语句中使用的分隔符，可以在文件中提供或使用命令选项。

+   `--detach=*`N`*`

    | 命令行格式 | `--detach=#` |
    | --- | --- |
    | 类型 | 数值 |
    | 默认值 | `0` |

    每*`N`*个语句后分离（关闭并重新打开）每个连接。默认值为 0（连接不分离）。

+   `--enable-cleartext-plugin`

    | 命令行格式 | `--enable-cleartext-plugin` |
    | --- | --- |
    | 类型 | 布尔值 |
    | 默认值 | `FALSE` |

    启用`mysql_clear_password`明文认证插件。（请参阅 Section 8.4.1.4, “Client-Side Cleartext Pluggable Authentication”。）

+   `--engine=*`engine_name`*`, `-e *`engine_name`*`

    | 命令行格式 | `--engine=engine_name` |
    | --- | --- |
    | 类型 | 字符串 |

    用于创建表的存储引擎。

+   `--get-server-public-key`

    | 命令行格式 | `--get-server-public-key` |
    | --- | --- |
    | 类型 | 布尔 |

    从服务器请求用于基于密钥对密码交换的 RSA 公钥。此选项适用于使用通过`caching_sha2_password`认证插件进行身份验证的帐户连接到服务器的客户端。对于通过这些帐户连接的连接，除非请求，否则服务器不会将公钥发送给客户端。对于不使用该插件进行身份验证的帐户，此选项将被忽略。如果不需要基于 RSA 的密码交换，例如客户端使用安全连接连接到服务器时，也将被忽略。

    如果给定`--server-public-key-path=*`file_name`*`并指定有效的公钥文件，则它将优先于`--get-server-public-key`。

    有关`caching_sha2_password`插件的信息，请参阅 Section 8.4.1.2, “Caching SHA-2 Pluggable Authentication”。

+   `--host=*`host_name`*`, `-h *`host_name`*`

    | 命令行格式 | `--host=host_name` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `localhost` |

    连接到给定主机上的 MySQL 服务器。

+   `--iterations=*`N`*`, `-i *`N`*`

    | 命令行格式 | `--iterations=#` |
    | --- | --- |
    | 类型 | 数字 |

    运行测试的次数。

+   `--login-path=*`name`*`

    | 命令行格式 | `--login-path=name` |
    | --- | --- |
    | 类型 | 字符串 |

    从`.mylogin.cnf`登录路径文件中的指定登录路径读取选项。 “登录路径”是一个包含指定要连接到哪个 MySQL 服务器以及要作为哪个帐户进行身份验证的选项的选项组。要创建或修改登录路径文件，请使用**mysql_config_editor**实用程序。请参阅 Section 6.6.7, “mysql_config_editor — MySQL Configuration Utility”。

    有关此选项和其他选项文件选项的其他信息，请参阅 Section 6.2.2.3, “Command-Line Options that Affect Option-File Handling”。

+   `--no-drop`

    | 命令行格式 | `--no-drop` |
    | --- | --- |
    | 类型 | 布尔值 |
    | 默认值 | `FALSE` |

    防止 **mysqlslap** 在测试运行期间删除其创建的任何模式。

+   `--no-defaults`

    | 命令行格式 | `--no-defaults` |
    | --- | --- |

    不读取任何选项文件。如果由于从选项文件中读取未知选项而导致程序启动失败，则可以使用 `--no-defaults` 防止读取这些选项。

    例外情况是，无论何种情况下都会读取 `.mylogin.cnf` 文件（如果存在）。这使得可以以比在命令行上更安全的方式指定密码，即使使用了 `--no-defaults`。要创建 `.mylogin.cnf`，请使用 **mysql_config_editor** 实用程序。请参阅 第 6.6.7 节，“mysql_config_editor — MySQL 配置实用程序”。

    有关此选项和其他选项文件选项的更多信息，请参阅 第 6.2.2.3 节，“影响选项文件处理的命令行选项”。

+   `--number-char-cols=*`N`*`, `-x *`N`*`

    | 命令行格式 | `--number-char-cols=#` |
    | --- | --- |
    | 类型 | 数值 |

    如果指定了 `--auto-generate-sql`，则使用的 `VARCHAR` 列的数量。

+   `--number-int-cols=*`N`*`, `-y *`N`*`

    | 命令行格式 | `--number-int-cols=#` |
    | --- | --- |
    | 类型 | 数值 |

    如果指定了 `--auto-generate-sql`，则使用的 `INT` 列的数量。

+   `--number-of-queries=*`N`*`

    | 命令行格式 | `--number-of-queries=#` |
    | --- | --- |
    | 类型 | 数值 |

    将每个客户端限制在大约这么多查询。查询计数考虑了语句分隔符。例如，如果像下面这样调用 **mysqlslap**，则会识别 `;` 分隔符，使得查询字符串的每个实例计为两个查询。因此，插入了 5 行（而不是 10 行）。

    ```sql
    mysqlslap --delimiter=";" --number-of-queries=10
              --query="use test;insert into t values(null)"
    ```

+   `--only-print`

    | 命令行格式 | `--only-print` |
    | --- | --- |
    | 类型 | 布尔值 |
    | 默认值 | `FALSE` |

    不连接到数据库。**mysqlslap** 只打印它本应该执行的操作。

+   [`--password[=*`password`*]`](mysqlslap.html#option_mysqlslap_password), `-p[*`password`*]`

    | 命令行格式 | `--password[=password]` |
    | --- | --- |
    | 类型 | 字符串 |

    用于连接到服务器的 MySQL 帐户的密码。密码值是可选的。如果未提供，**mysqlslap**会提示输入密码。如果提供了密码，则`--password=`或`-p`之后的密码之间*不能有空格*。如果未指定密码选项，则默认不发送密码。

    在命令行上指定密码应被视为不安全。为了避免在命令行上提供密码，请使用选项文件。请参阅第 8.1.2.1 节，“密码安全的最终用户指南”。

    要明确指定没有密码，并且**mysqlslap**不应提示密码，请使用`--skip-password`选项。

+   [`--password1[=*`pass_val`*]`](mysqlslap.html#option_mysqlslap_password1)

    用于连接到服务器的 MySQL 帐户的多因素认证因子 1 的密码。密码值是可选的。如果未提供，**mysqlslap**会提示输入密码。如果提供了密码，则`--password1=`和其后的密码之间*不能有空格*。如果未指定密码选项，则默认不发送密码。

    在命令行上指定密码应被视为不安全。为了避免在命令行上提供密码，请使用选项文件。请参阅第 8.1.2.1 节，“密码安全的最终用户指南”。

    要明确指定没有密码，并且**mysqlslap**不应提示密码，请使用`--skip-password1`选项。

    `--password1`和`--password`是同义词，`--skip-password1`和`--skip-password`也是。

+   [`--password2[=*`pass_val`*]`](mysqlslap.html#option_mysqlslap_password2)

    用于连接到服务器的 MySQL 帐户的多因素认证因子 2 的密码。此选项的语义类似于`--password1`的语义；有关详细信息，请参阅该选项的描述。

+   [`--password3[=*`pass_val`*]`](mysqlslap.html#option_mysqlslap_password3)

    用于连接到服务器的 MySQL 帐户的多因素认证因子 3 的密码。此选项的语义类似于 `--password1` 的语义；有关详细信息，请参阅该选项的描述。

+   `--pipe`, `-W`

    | 命令行格式 | `--pipe` |
    | --- | --- |
    | 类型 | 字符串 |

    在 Windows 上，使用命名管道连接到服务器。此选项仅在服务器启动时启用了支持命名管道连接的 `named_pipe` 系统变量时才适用。此外，进行连接的用户必须是由 `named_pipe_full_access_group` 系统变量指定的 Windows 组的成员。

+   `--plugin-dir=*`dir_name`*`

    | 命令行格式 | `--plugin-dir=dir_name` |
    | --- | --- |
    | 类型 | 目录名称 |

    查找插件的目录。如果使用 `--default-auth` 选项指定身份验证插件但 **mysqlslap** 找不到它，则指定此选项。参见 Section 8.2.17, “Pluggable Authentication”。

+   `--port=*`port_num`*`, `-P *`port_num`*`

    | 命令行格式 | `--port=port_num` |
    | --- | --- |
    | 类型 | 数字 |
    | 默认值 | `3306` |

    用于 TCP/IP 连接的端口号。

+   `--post-query=*`value`*`

    | 命令行格式 | `--post-query=value` |
    | --- | --- |
    | 类型 | 字符串 |

    包含在测试完成后要执行的语句的文件或字符串。此执行不计入计时目的。

+   `--post-system=*`str`*`

    | 命令行格式 | `--post-system=str` |
    | --- | --- |
    | 类型 | 字符串 |

    在测试完成后使用 `system()` 执行的字符串。此执行不计入计时目的。

+   `--pre-query=*`value`*`

    | 命令行格式 | `--pre-query=value` |
    | --- | --- |
    | 类型 | 字符串 |

    包含在运行测试之前要执行的语句的文件或字符串。此执行不计入计时目的。

+   `--pre-system=*`str`*`

    | 命令行格式 | `--pre-system=str` |
    | --- | --- |
    | 类型 | 字符串 |

    在运行测试之前使用 `system()` 执行的字符串。此执行不计入计时目的。

+   `--print-defaults`

    | 命令行格式 | `--print-defaults` |
    | --- | --- |

    打印程序名称以及从选项文件获取的所有选项。

    有关此选项和其他选项文件选项的附加信息，请参见第 6.2.2.3 节，“影响选项文件处理的命令行选项”。

+   `--protocol={TCP|SOCKET|PIPE|MEMORY}`

    | 命令行格式 | `--protocol=type` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `[见文本]` |
    | 有效值 | `TCP``SOCKET``PIPE``MEMORY` |

    用于连接服务器的传输协议。当其他连接参数通常导致使用不希望的协议时，这将非常有用。有关允许值的详细信息，请参见第 6.2.7 节，“连接传输协议”。

+   `--query=*`value`*`, `-q *`value`*`

    | 命令行格式 | `--query=value` |
    | --- | --- |
    | 类型 | 字符串 |

    包含`SELECT`语句的文件或字符串，用于检索数据。

+   `--server-public-key-path=*`file_name`*`

    | 命令行格式 | `--server-public-key-path=file_name` |
    | --- | --- |
    | 类型 | 文件名 |

    以 PEM 格式包含客户端端公钥的文件路径名，该公钥是服务器用于 RSA 密钥对密码交换所需的。此选项适用于使用`sha256_password`或`caching_sha2_password`认证插件进行身份验证的客户端。对于不使用这些插件进行身份验证的帐户，此选项将被忽略。如果不使用基于 RSA 的密码交换（例如客户端使用安全连接连接到服务器时），此选项也将被忽略。

    如果给定`--server-public-key-path=*`file_name`*`并指定有效的公钥文件，则它将优先于`--get-server-public-key`。

    对于`sha256_password`，仅当 MySQL 使用 OpenSSL 构建时，此选项才适用。

    有关`sha256_password`和`caching_sha2_password`插件的信息，请参见第 8.4.1.3 节，“SHA-256 可插拔认证”和第 8.4.1.2 节，“缓存 SHA-2 可插拔认证”。

+   `--shared-memory-base-name=*`name`*`

    | 命令行格式 | `--shared-memory-base-name=name` |
    | --- | --- |
    | 平台特定 | Windows |

    在 Windows 上，用于使用共享内存进行到本地服务器的连接的共享内存名称。默认值为`MYSQL`。共享内存名称区分大小写。

    此选项仅在服务器启动时启用了支持共享内存连接的`shared_memory`系统变量时才适用。

+   `--silent`, `-s`

    | 命令行格式 | `--silent` |
    | --- | --- |

    静默模式。无输出。

+   `--socket=*`path`*`, `-S *`path`*`

    | 命令行格式 | `--socket={file_name&#124;pipe_name}` |
    | --- | --- |
    | 类型 | 字符串 |

    对于对`localhost`的连接，要使用的 Unix 套接字文件，或者在 Windows 上要使用的命名管道的名称。

    在 Windows 上，仅当服务器启动时启用了支持命名管道连接的`named_pipe`系统变量时，此选项才适用。此外，进行连接的用户必须是由`named_pipe_full_access_group`系统变量指定的 Windows 组的成员。

+   `--sql-mode=*`mode`*`

    | 命令行格式 | `--sql-mode=mode` |
    | --- | --- |
    | 类型 | 字符串 |

    设置客户端会话的 SQL 模式。

+   `--ssl*`

    以`--ssl`开头的选项指定是否使用加密连接到服务器，并指示 SSL 密钥和证书的位置。请参阅加密连接的命令选项。

+   `--ssl-fips-mode={OFF|ON|STRICT}`

    | 命令行格式 | `--ssl-fips-mode={OFF&#124;ON&#124;STRICT}` |
    | --- | --- |
    | 已弃用 | 8.0.34 |
    | 类型 | 枚举 |
    | 默认值 | `OFF` |
    | 有效值 | `OFF``ON``STRICT` |

    控制是否在客户端端启用 FIPS 模式。`--ssl-fips-mode`选项与其他`--ssl-*`xxx`*`选项不同，它不用于建立加密连接，而是影响允许哪些加密操作。请参阅第 8.8 节，“FIPS 支持”。

    允许使用这些`--ssl-fips-mode`值： 

    +   `OFF`: 禁用 FIPS 模式。

    +   `ON`: 启用 FIPS 模式。

    +   `STRICT`: 启用“严格” FIPS 模式。

    注意

    如果 OpenSSL FIPS 对象模块不可用，则`--ssl-fips-mode`的唯一允许值为`OFF`。在这种情况下，将`--ssl-fips-mode`设置为`ON`或`STRICT`会导致客户端在启动时产生警告并在非 FIPS 模式下运行。

    从 MySQL 8.0.34 开始，此选项已弃用。预计在将来的 MySQL 版本中将其移除。

+   `--tls-ciphersuites=*`ciphersuite_list`*`

    | 命令行格式 | `--tls-ciphersuites=ciphersuite_list` |
    | --- | --- |
    | 引入版本 | 8.0.16 |
    | 类型 | 字符串 |

    用于使用 TLSv1.3 的加密连接的允许密码套件。该值是一个或多个以冒号分隔的密码套件名称列表。可以为此选项命名的密码套件取决于用于编译 MySQL 的 SSL 库。有关详细信息，请参见第 8.3.2 节，“加密连接 TLS 协议和密码”。

    此选项已添加到 MySQL 8.0.16 中。

+   `--tls-version=*`protocol_list`*`

    | 命令行格式 | `--tls-version=protocol_list` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值（≥ 8.0.16） | `TLSv1,TLSv1.1,TLSv1.2,TLSv1.3`（OpenSSL 1.1.1 或更高版本）`TLSv1,TLSv1.1,TLSv1.2`（否则） |
    | 默认值（≤ 8.0.15） | `TLSv1,TLSv1.1,TLSv1.2` |

    可用于加密连接的 TLS 协议。该值是一个或多个逗号分隔的协议名称列表。可以为此选项命名的协议取决于用于编译 MySQL 的 SSL 库。有关详细信息，请参见第 8.3.2 节，“加密连接 TLS 协议和密码”。

+   `--user=*`user_name`*`, `-u *`user_name`*`

    | 命令行格式 | `--user=user_name,` |
    | --- | --- |
    | 类型 | 字符串 |

    用于连接到服务器的 MySQL 帐户的用户名。

+   `--verbose`, `-v`

    | 命令行格式 | `--verbose` |
    | --- | --- |

    详细模式。打印程序执行的更多信息。此选项可多次使用以增加信息量。

+   `--version`, `-V`

    | 命令行格式 | `--version` |
    | --- | --- |

    显示版本信息并退出。

+   `--zstd-compression-level=*`level`*`

    | 命令行格式 | `--zstd-compression-level=#` |
    | --- | --- |
    | 引入版本 | 8.0.18 |
    | 类型 | 整数 |

    用于使用 `zstd` 压缩算法连接到服务器的压缩级别。允许的级别从 1 到 22，较大的值表示较高级别的压缩。默认的 `zstd` 压缩级别为 3。压缩级别设置对不使用 `zstd` 压缩的连接没有影响。

    有关更多信息，请参见第 6.2.8 节，“连接压缩控制”。

    此选项已添加到 MySQL 8.0.18 中。
