# 6.6.9 mysqlbinlog — 用于处理二进制日志文件的实用程序

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysqlbinlog.html`](https://dev.mysql.com/doc/refman/8.0/en/mysqlbinlog.html)

6.6.9.1 mysqlbinlog 十六进制转储格式

6.6.9.2 mysqlbinlog 行事件显示

6.6.9.3 使用 mysqlbinlog 备份二进制日志文件

6.6.9.4 指定 mysqlbinlog 服务器 ID

服务器的二进制日志由包含描述数据库内容修改的“事件”的文件组成。服务器以二进制格式编写这些文件。要以文本格式显示它们的内容，请使用**mysqlbinlog**实用程序。您还可以使用**mysqlbinlog**来显示由复制设置中的副本服务器写入的中继日志文件的内容，因为中继日志与二进制日志具有相同的格式。有关二进制日志和中继日志的进一步讨论，请参见第 7.4.4 节，“二进制日志”和第 19.2.4 节，“中继日志和复制元数据存储库”。

调用**mysqlbinlog**的方法如下：

```sql
mysqlbinlog [*options*] *log_file* ...
```

例如，要显示名为`binlog.000003`的二进制日志文件的内容，请使用以下命令：

```sql
mysqlbinlog binlog.000003
```

输出包括`binlog.000003`中包含的事件。对于基于语句的日志记录，事件信息包括 SQL 语句、执行该语句的服务器 ID、执行语句时的时间戳、花费的时间等。对于基于行的日志记录，事件指示的是行更改而不是 SQL 语句。有关日志记录模式的信息，请参见第 19.2.1 节，“复制格式”。

事件前面有提供额外信息的头部注释。例如：

```sql
# at 141
#100309  9:28:36 server id 123  end_log_pos 245
  Query thread_id=3350  exec_time=11  error_code=0
```

在第一行中，`at`后面的数字表示二进制日志文件中事件的文件偏移量或起始位置。

第二行以日期和时间开头，指示语句在事件发生的服务器上开始执行的时间。对于复制，此时间戳会传播到副本服务器。`server id`是事件发生的服务器的`server_id`值。`end_log_pos`指示下一个事件从哪里开始（即，当前事件的结束位置+1）。`thread_id`指示执行事件的线程。`exec_time`是在复制源服务器上执行事件所花费的时间。在副本上，它是副本上结束执行时间减去源上开始执行时间的差值。这个差值作为指示复制滞后源的指标。`error_code`指示执行事件的结果。零表示没有发生错误。

注意

在使用事件组时，事件的文件偏移量可能会被分组在一起，事件的注释可能会被分组在一起。不要将这些分组事件误认为是空白文件偏移量。

**mysqlbinlog**的输出可以重新执行（例如，通过将其用作**mysql**的输入）以重做日志中的语句。这对于意外服务器退出后的恢复操作非常有用。有关其他用法示例，请参见本节后面的讨论以及第 9.5 节，“时间点（增量）恢复”。要执行**mysqlbinlog**使用的内部使用`BINLOG`语句，用户需要`BINLOG_ADMIN`权限（或已弃用的`SUPER`权限），或者`REPLICATION_APPLIER`权限加上��行每个日志事件所需的适当权限。

你可以使用**mysqlbinlog**直接读取二进制日志文件并将其应用于本地 MySQL 服务器。您还可以通过使用`--read-from-remote-server`选项从远程服务器读取二进制日志。要读取远程二进制日志，可以提供连接参数选项以指示如何连接到服务器。这些选项是`--host`，`--password`，`--port`，`--protocol`，`--socket`和`--user`。

当二进制日志文件已加密时，可以从 MySQL 8.0.14 开始执行，**mysqlbinlog** 无法直接读取它们，但可以使用`--read-from-remote-server`选项从服务器读取它们。当服务器的`binlog_encryption`系统变量设置为`ON`时，二进制日志文件会被加密。`SHOW BINARY LOGS` 语句显示特定二进制日志文件是加密还是未加密。加密和未加密的二进制日志文件也可以通过加密日志文件头部的魔数来区分（加密日志文件为`0xFD62696E`，与未加密日志文件使用的不同为`0xFE62696E`）。请注意，从 MySQL 8.0.14 开始，**mysqlbinlog** 如果尝试直接读取加密的二进制日志文件，则会返回适当的错误，但旧版本的**mysqlbinlog** 完全不会将文件识别为二进制日志文件。有关二进制日志加密的更多信息，请参见 Section 19.3.2, “Encrypting Binary Log Files and Relay Log Files”。

当二进制日志事务负载已经被压缩时，可以从 MySQL 8.0.20 开始执行，**mysqlbinlog** 版本从该版本开始会自动解压缩和解码事务负载，并将其打印为未压缩事件。旧版本的**mysqlbinlog** 无法读取压缩的事务负载。当服务器的`binlog_transaction_compression`系统变量设置为`ON`时，事务负载会被压缩，然后作为单个事件（`Transaction_payload_event`）写入服务器的二进制日志文件。使用`--verbose`选项，**mysqlbinlog** 添加注释，说明使用的压缩算法，最初接收到的压缩负载大小，以及解压缩后的负载大小。

注意

**mysqlbinlog** 对于作为压缩事务负载的一部分的单个事件所述的结束位置（`end_log_pos`）与原始压缩负载的结束位置相同。因此，多个解压缩事件可能具有相同的结束位置。

**mysqlbinlog**自身的连接压缩如果事务负载已经被压缩，则效果较小，但仍会对未压缩的事务和标头进行操作。

有关二进制日志事务压缩的更多信息，请参见第 7.4.4.5 节，“二进制日志事务压缩”。

运行**mysqlbinlog**时，针对大型二进制日志，请注意文件系统是否有足够的空间来存储生成的文件。要配置**mysqlbinlog**用于临时文件的目录，请使用`TMPDIR`环境变量。

**mysqlbinlog** 在执行任何 SQL 语句之前将`pseudo_replica_mode`或`pseudo_slave_mode`的值设置为 true。此系统变量影响 XA 事务的处理，`original_commit_timestamp`复制延迟时间戳和`original_server_version`系统变量以及不受支持的 SQL 模式。

**mysqlbinlog** 支持以下选项，可以在命令行或选项文件的`[mysqlbinlog]`和`[client]`组中指定。有关 MySQL 程序使用的选项文件的信息，请参见第 6.2.2.2 节，“使用选项文件”。

**表 6.23 mysqlbinlog ���项**

| 选项名称 | 描述 | 引入 | 废弃 |
| --- | --- | --- | --- |
| --base64-output | 使用 base-64 编码打印二进制日志条目 |  |  |
| --bind-address | 使用指定的网络接口连接到 MySQL 服务器 |  |  |
| --binlog-row-event-max-size | 二进制日志最大事件大小 |  |  |
| --character-sets-dir | 安装字符集的目录 |  |  |
| --compress | 压缩客户端和服务器之间发送的所有信息 | 8.0.17 | 8.0.18 |
| --compression-algorithms | 连接到服务器的允许的压缩算法 | 8.0.18 |  |
| --connection-server-id | 用于测试和调试。有关适用的默认值和其他详细信息，请参阅文本 |  |  |
| --database | 仅列出此数据库的条目 |  |  |
| --debug | 写入调试日志 |  |  |
| --debug-check | 在程序退出时打印调试信息 |  |  |
| --debug-info | 在程序退出时打印调试信息、内存和 CPU 统计信息 |  |  |
| --default-auth | 要使用的认证插件 |  |  |
| --defaults-extra-file | 除了通常的选项文件外，读取指定的命名选项文件 |  |  |
| --defaults-file | 仅读取指定的选项文件 |  |  |
| --defaults-group-suffix | 选项组后缀值 |  |  |
| --disable-log-bin | 禁用二进制日志记录 |  |  |
| --exclude-gtids | 不显示提供的 GTID 集合中的任何组 |  |  |
| --force-if-open | 即使打开或未正确关闭，也读取二进制日志文件 |  |  |
| --force-read | 如果 mysqlbinlog 读取一个无法识别的��进制日志事件，它会打印警告 |  |  |
| --get-server-public-key | 从服务器请求 RSA 公钥 |  |  |
| --help | 显示帮助消息并退出 |  |  |
| --hexdump | 在注释中显示日志的十六进制转储 |  |  |
| --host | MySQL 服务器所在的主机 |  |  |
| --idempotent | 使服务器在仅从此会话处理二进制日志更新时使用幂等模式 |  |  |
| --include-gtids | 仅显示提供的 GTID 集合中的组 |  |  |
| --local-load | 在指定目录为 LOAD DATA 准备本地临时文件 |  |  |
| --login-path | 从.mylogin.cnf 中读取登录路径选项 |  |  |
| --no-defaults | 不读取任何选项文件 |  |  |
| --offset | 跳过日志中的前 N 个条目 |  |  |
| --password | 连接到服务器时使用的密码 |  |  |
| --plugin-dir | 安装插件的目录 |  |  |
| --port | 连接的 TCP/IP 端口号 |  |  |
| --print-defaults | 打印默认选项 |  |  |
| --print-table-metadata | 打印表元数据 |  |  |
| --protocol | 使用的传输协议 |  |  |
| --raw | 将事件以原始（二进制）格式写入输出文件 |  |  |
| --read-from-remote-master | 从 MySQL 复制源服务器读取二进制日志，而不是读取本地日志文件 |  | 8.0.26 |
| --read-from-remote-server | 从 MySQL 服务器读取二进制日志，而不是本地日志文件 |  |  |
| --read-from-remote-source | 从 MySQL 复制源服务器读取二进制日志，而不是读取本地日志文件 | 8.0.26 |  |
| --require-row-format | 要求行级二进制日志格式 | 8.0.19 |  |
| --result-file | 将输出直接定向到指定文件 |  |  |
| --rewrite-db | 在以行格式编写的日志中回放时为数据库创建重写规则。可以多次使用 |  |  |
| --server-id | 仅提取由具有给定服务器 ID 的服务器创建的事件 |  |  |
| --server-id-bits | 告诉 mysqlbinlog 如何解释二进制日志中的服务器 ID，当日志由将其 server-id-bits 设置为小于最大值的 mysqld 编写时；仅由 MySQL Cluster 版本的 mysqlbinlog 支持 |  |  |
| --server-public-key-path | 包含 RSA 公钥的文件路径名 |  |  |
| --set-charset | 在输出中添加一个 SET NAMES charset_name 语句 |  |  |
| --shared-memory-base-name | 共享内存连接的共享内存名称（仅限 Windows） |  |  |
| --short-form | 仅显示日志中包含的语句 |  |  |
| --skip-gtids | 在输出转储文件中不包括二进制日志文件中的 GTID |  |  |
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
| --ssl-session-data-continue-on-failed-reuse | 是否在会话重用失败时建立连接 | 8.0.29 |  |
| --start-datetime | 从时间戳等于或晚于日期时间参数的第一个事件读取二进制日志 |  |  |
| --start-position | 从位置等于或大于参数的第一个事件解码二进制日志 |  |  |
| --stop-datetime | 在时间戳等于或大于日期时间参数的第一个事件处停止读取二进制日志 |  |  |
| --stop-never | 在读取最后一个二进制日志文件后保持与服务器的连接 |  |  |
| --stop-never-slave-server-id | 连接到服务器时报告的从服务器 ID |  |  |
| --stop-position | 在位置等于或大于参数的第一个事件处停止解码二进制日志 |  |  |
| --tls-ciphersuites | 用于加密连接的允许的 TLSv1.3 密码套件 | 8.0.16 |  |
| --tls-version | 用于加密连接的允许的 TLS 协议 |  |  |
| --to-last-log | 不要在从 MySQL 服务器请求的二进制日志末尾停止，而是继续打印到最后一个二进制日志的末尾 |  |  |
| --user | 连接到服务器时要使用的 MySQL 用户名 |  |  |
| --verbose | 将行事件重构为 SQL 语句 |  |  |
| --verify-binlog-checksum | 验证二进制日志中的校验和 |  |  |
| --version | 显示版本信息并退出 |  |  |
| --zstd-compression-level | 用于使用 zstd 压缩连接的服务器的压缩级别 | 8.0.18 |  |
| 选项名称 | 描述 | 引入 | 废弃 |

+   `--help`，`-?`

    | 命令行格式 | `--help` |
    | --- | --- |

    显示帮助消息并退出。

+   `--base64-output=*`value`*`

    | 命令行格式 | `--base64-output=value` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `AUTO` |
    | 有效值 | `AUTO``NEVER``DECODE-ROWS` |

    此选项确定何时应使用`BINLOG`语句将事件显示为经过 base-64 编码的字符串。该选项具有以下可允许的值（不区分大小写）：

    +   `AUTO`（"自动"）或`UNSPEC`（"未指定"）在必要时自动显示`BINLOG`语句（即，用于格式描述事件和行事件）。如果没有给出`--base64-output`选项，则效果与`--base64-output=AUTO`相同。

        注意

        如果您打算使用**mysqlbinlog**的输出重新执行二进制日志文件内容，则自动显示`BINLOG`是唯一安全的行为。其他选项值仅用于调试或测试目的，因为它们可能生成不包含所有事件的可执行形式的输出。

    +   `NEVER`导致不显示`BINLOG`语句。如果发现必须使用`BINLOG`显示的行事件，则**mysqlbinlog**将退出并显示错误。

    +   `DECODE-ROWS` 指定给 **mysqlbinlog**，你打算将行事件解码并显示为注释的 SQL 语句，同时还要指定 `--verbose` 选项。与 `NEVER` 类似，`DECODE-ROWS` 抑制显示 `BINLOG` 语句，但与 `NEVER` 不同的是，如果找到行事件，它不会因错误而退出。

    有关 `--base64-output` 和 `--verbose` 对行事件输出的影响的示例，请参见第 6.6.9.2 节，“mysqlbinlog 行事件显示”。

+   `--bind-address=*`ip_address`*`

    | 命令行格式 | `--bind-address=ip_address` |
    | --- | --- |

    在具有多个网络接口的计算机上，使用此选项选择连接到 MySQL 服务器的网络接口。

+   `--binlog-row-event-max-size=*`N`*`

    | 命令行格式 | `--binlog-row-event-max-size=#` |
    | --- | --- |
    | 类型 | 数字 |
    | 默认值 | `4294967040` |
    | 最小值 | `256` |
    | 最大值 | `18446744073709547520` |

    指定基于行的二进制日志事件的最大大小，以字节为单位。如果可能，将行分组为小于此大小的事件。该值应为 256 的倍数。默认值为 4GB。

+   `--character-sets-dir=*`dir_name`*`

    | 命令行格式 | `--character-sets-dir=dir_name` |
    | --- | --- |
    | 类型 | 目录名称 |

    安装字符集的目录。参见第 12.15 节，“字符集配置”。

+   `--compress`

    | 命令行格式 | `--compress[={OFF&#124;ON}]` |
    | --- | --- |
    | 引入版本 | 8.0.17 |
    | 已弃用 | 8.0.18 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    尽可能压缩客户端和服务器之间发送的所有信息。参见第 6.2.8 节，“连接压缩控制”。

    此选项在 MySQL 8.0.17 版本中添加。截至 MySQL 8.0.18 版本，该选项已被弃用。预计在未来的 MySQL 版本中将会移除。参见配置传统连接压缩。

+   `--compression-algorithms=*`value`*`

    | 命令行格式 | `--compression-algorithms=value` |
    | --- | --- |
    | 引入版本 | 8.0.18 |
    | 类型 | 集合 |
    | 默认值 | `uncompressed` |
    | 有效值 | `zlib``zstd``uncompressed` |

    连接到服务器的允许压缩算法。可用算法与`protocol_compression_algorithms`系统变量相同。默认值为`uncompressed`。

    查看更多信息，请参阅第 6.2.8 节，“连接压缩控制”。

    此选项在 MySQL 8.0.18 中添加。

+   `--connection-server-id=*`server_id`*`

    | 命令行格式 | `--connection-server-id=#]` |
    | --- | --- |
    | 类型 | 整数 |
    | 默认值 | `0 (1)` |
    | 最小值 | `0 (1)` |
    | 最大值 | `4294967295` |

    `--connection-server-id`指定**mysqlbinlog**连接到服务器时报告的服务器 ID。它可用于避免与复制服务器或另一个**mysqlbinlog**进程的 ID 冲突。

    如果指定了`--read-from-remote-server`选项，**mysqlbinlog**报告服务器 ID 为 0，告诉服务器在发送最后一个日志文件后断开连接（非阻塞行为）。如果还指定了`--stop-never`选项以保持与服务器的连接，**mysqlbinlog**默认报告服务器 ID 为 1 而不是 0，并且如果需要，可以使用`--connection-server-id`替换该服务器 ID。参见第 6.6.9.4 节，“指定 mysqlbinlog 服务器 ID”。

+   `--database=*`db_name`*`, `-d *`db_name`*`

    | 命令行格式 | `--database=db_name` |
    | --- | --- |
    | 类型 | 字符串 |

    此选项导致**mysqlbinlog**输出二进制日志（仅本地日志）中在*`db_name`*被选择为默认数据库时发生的条目，`USE`语句。

    **mysqlbinlog**的`--database`选项类似于**mysqld**的`--binlog-do-db`选项，但只能用于指定一个数据库。如果`--database`被多次给出，只有最后一个实例会被使用。

    此选项的效果取决于是否使用基于语句或基于行的日志格式，就像`--binlog-do-db`的效果取决于是否使用基于语句或基于行的日志一样。

    **基于语句的日志记录。** `--database`选项的工作方式如下：

    +   当*`db_name`*是默认数据库时，无论是修改*`db_name`*中的表还是其他数据库中的表，语句都会输出。

    +   除非*`db_name`*被选为默认数据库，否则不会输出语句，即使它们修改了*`db_name`*中的表。

    +   对于`CREATE DATABASE`、`ALTER DATABASE`和`DROP DATABASE`有一个例外。在确定是否输出语句时，被*创建、修改或删除*的数据库被视为默认数据库。

    假设二进制日志是通过使用基于语句记录执行这些语句创建的：

    ```sql
    INSERT INTO test.t1 (i) VALUES(100);
    INSERT INTO db2.t2 (j)  VALUES(200);
    USE test;
    INSERT INTO test.t1 (i) VALUES(101);
    INSERT INTO t1 (i)      VALUES(102);
    INSERT INTO db2.t2 (j)  VALUES(201);
    USE db2;
    INSERT INTO test.t1 (i) VALUES(103);
    INSERT INTO db2.t2 (j)  VALUES(202);
    INSERT INTO t2 (j)      VALUES(203);
    ```

    **mysqlbinlog --database=test** 不会输出前两个`INSERT`语句，因为没有默认数据库。它会输出`USE test`后面的三个`INSERT`语句，但不会输出`USE db2`后面的三个`INSERT`语句。

    **mysqlbinlog --database=db2** 不会输出前两个`INSERT`语句，因为没有默认数据库。它不会输出`USE test`后面的三个`INSERT`语句，但会输出`USE db2`后面的三个`INSERT`语句。

    **基于行的日志记录。** **mysqlbinlog** 仅输出更改属于*`db_name`*的表的条目。默认数据库对此没有影响。假设刚才描述的二进制日志是使用基于行的日志记录而不是基于语句的日志记录创建的。**mysqlbinlog --database=test** 仅输出修改测试数据库中的`t1`的条目，无论是否发出了`USE`或默认数据库是什么。

    如果服务器正在以`MIXED`设置`binlog_format`运行，并且您希望能够使用**mysqlbinlog**与`--database`选项，则必须确保被修改的表位于由`USE`选择的数据库中。（特别是，不应使用跨数据库更新。）

    与`--rewrite-db`选项一起使用时，`--rewrite-db`选项首先应用；然后应用`--database`选项，使用重写后的数据库名称。在提供选项的顺序方面没有任何区别。

+   [`--debug[=*`debug_options`*]`](mysqlbinlog.html#option_mysqlbinlog_debug), `-# [*`debug_options`*]`

    | 命令行格式 | `--debug[=debug_options]` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `d:t:o,/tmp/mysqlbinlog.trace` |

    写入调试日志。典型的*`debug_options`*字符串是`d:t:o,*`file_name`*`。默认值为`d:t:o,/tmp/mysqlbinlog.trace`。

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

+   `--default-auth=*`plugin`*`

    | 命令行格式 | `--default-auth=plugin` |
    | --- | --- |
    | 类型 | 字符串 |

    关于要使用哪个客户端端身份验证插件的提示。请参见第 8.2.17 节，“可插拔身份验证”。

+   `--defaults-extra-file=*`file_name`*`

    | 命令行格式 | `--defaults-extra-file=file_name` |
    | --- | --- |
    | 类型 | 文件名 |

    在全局选项文件之后但在用户选项文件之前读取此选项文件。如果文件不存在或无法访问，将会出现错误。如果*`file_name`*不是绝对路径名，则将其解释为相对于当前目录。

    有关此选项和其他选项文件选项的更多信息，请参见第 6.2.2.3 节，“影响选项文件处理的命令行选项”。

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

    不仅读取通常的选项组，还读取具有通常名称和后缀*`str`*的组。例如，**mysqlbinlog**通常会读取`[client]`和`[mysqlbinlog]`组。如果给定此选项作为`--defaults-group-suffix=_other`，**mysqlbinlog**还会读取`[client_other]`和`[mysqlbinlog_other]`组。

    有关此选项和其他选项文件选项的更多信息，请参见第 6.2.2.3 节，“影响选项文件处理的命令行选项”。

+   `--disable-log-bin`, `-D`

    | 命令行格式 | `--disable-log-bin` |
    | --- | --- |

    禁用二进制日志记录。如果您使用`--to-last-log`选项并将输出发送到同一 MySQL 服务器，这对于避免无限循环很有用。此选项在意外退出后进行恢复时也很有用，以避免重复记录的语句。

    此选项会导致**mysqlbinlog**在输出中包含一个`SET sql_log_bin = 0`语句，以禁用剩余输出的二进制日志记录。操纵`sql_log_bin`系统变量的会话值是一项受限操作，因此此选项要求您具有足够权限设置受限会话变量。请参阅 Section 7.1.9.1, “System Variable Privileges”。

+   `--exclude-gtids=*`gtid_set`*`

    | 命令行格式 | `--exclude-gtids=gtid_set` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 |  |

    不显示*`gtid_set`*中列出的任何组。

+   `--force-if-open`, `-F`

    | 命令行格式 | `--force-if-open` |
    | --- | --- |

    读取二进制日志文件，即使它们是打开的或未正确关闭（`IN_USE`标志已设置）；如果文件以截断事件结束，则不会失败。

    `IN_USE`标志仅针对当前由服务器写入的二进制日志设置；如果服务器崩溃，该标志将保持设置，直到服务器再次启动并恢复二进制日志。没有此选项，**mysqlbinlog**拒绝处理具有此标志设置的文件。由于服务器可能正在写入文件，因此截断最后一个事件被视为正常。

+   `--force-read`, `-f`

    | 命令行格式 | `--force-read` |
    | --- | --- |

    使用此选项，如果**mysqlbinlog**读取一个无法识别的二进制日志事件，它会打印警告，忽略该事件并继续。如果没有此选项，**mysqlbinlog**会在读取此类事件时停止。

+   `--get-server-public-key`

    | 命令行格式 | `--get-server-public-key` |
    | --- | --- |
    | 类型 | 布尔值 |

    从服务器请求用于 RSA 密钥对密码交换的公钥。此选项适用于使用`caching_sha2_password`认证插件进行身份验证的客户端。对于该插件，除非请求，否则服务器不会发送公钥。对于不使用该插件进行身份验证的帐户，此选项将被忽略。如果不使用基于 RSA 的密码交换，例如客户端使用安全连接连接到服务器时，此选项也将被忽略。

    如果提供了`--server-public-key-path=*`file_name`*`并指定了有效的公钥文件，则它优先于`--get-server-public-key`。

    有关`caching_sha2_password`插件的信息，请参阅第 8.4.1.2 节，“缓存 SHA-2 可插拔认证”。

+   `--hexdump`, `-H`

    | 命令行格式 | `--hexdump` |
    | --- | --- |

    在注释中显示日志的十六进制转储，如第 6.6.9.1 节，“mysqlbinlog 十六进制转储格式”所述。十六进制输出对于复制调试可能很有帮助。

+   `--host=*`host_name`*`, `-h *`host_name`*`

    | 命令行格式 | `--host=host_name` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `localhost` |

    从给定主机的 MySQL 服务器获取二进制日志。

+   `--幂等`

    | 命令行格式 | `--幂等` |
    | --- | --- |
    | 类型 | 布尔值 |
    | 默认值 | `true` |

    告诉 MySQL 服务器在处理更新时使用幂等模式；这会导致服务器在处理更新时遇到当前会话中的任何重复键或找不到键错误时进行抑制。每当需要或必须重放一个或多个二进制日志到可能不包含日志所指数据的 MySQL 服务器时，此选项可能会很有用。

    此选项的影响范围仅包括当前的**mysqlbinlog**客户端和会话。

+   `--include-gtids=*`gtid_set`*`

    | 命令行格式 | `--include-gtids=gtid_set` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 |  |

    仅显示*`gtid_set`*中列出的组。

+   `--local-load=*`dir_name`*`, `-l *`dir_name`*`

    | 命令行格式 | `--local-load=dir_name` |
    | --- | --- |
    | 类型 | 目录名称 |

    对应于`LOAD DATA`语句的数据加载操作，**mysqlbinlog**从二进制日志事件中提取文件，将它们写入本地文件系统作为临时文件，并写入`LOAD DATA LOCAL`语句以导入文件。 默认情况下，**mysqlbinlog**将这些临时文件写入操作系统特定的目录。 可以使用`--local-load`选项来明确指定**mysqlbinlog**应准备本地临时文件的目录。

    因为其他进程可以将文件写入默认的操作系统特定目录，建议使用`--local-load`选项来指定**mysqlbinlog**的不同目录用于数据文件，并在处理来自**mysqlbinlog**输出时通过指定`--load-data-local-dir`选项来指定相同目录给**mysql**。 例如：

    ```sql
    mysqlbinlog --local-load=/my/local/data ...
        | mysql --load-data-local-dir=/my/local/data ...
    ```

    重要提示

    这些临时文件不会被**mysqlbinlog**或任何其他 MySQL 程序自动删除。

+   `--login-path=*`name`*`

    | 命令行格式 | `--login-path=name` |
    | --- | --- |
    | 类型 | 字符串 |

    从`.mylogin.cnf`登录路径文件中读取指定登录路径中的选项。 “登录路径”是一个包含指定要连接的 MySQL 服务器和要进行身份验证的帐户的选项的选项组。 要创建或修改登录路径文件，请使用**mysql_config_editor**实用程序。 请参阅第 6.6.7 节，“mysql_config_editor — MySQL 配置实用程序”。

    有关此选项文件选项和其他选项的其他信息，请参阅第 6.2.2.3 节，“影响选项文件处理的命令行选项”。

+   `--no-defaults`

    | 命令行格式 | `--no-defaults` |
    | --- | --- |

    不要读取任何选项文件。如果由于从选项文件中读取未知选项而导致程序启动失败，可以使用`--no-defaults`来防止读取它们。

    例外情况是无论何种情况下都会读取`.mylogin.cnf`文件，如果存在的话。这允许以比在命令行上更安全的方式指定密码，即使使用了`--no-defaults`。要创建`.mylogin.cnf`，请使用**mysql_config_editor**实用程序。参见第 6.6.7 节，“mysql_config_editor — MySQL 配置实用程序”。

    有关此选项文件选项和其他选项的更多信息，请参见第 6.2.2.3 节，“影响选项文件处理的命令行选项”。

+   `--offset=*`N`*`, `-o *`N`*`

    | 命令行格式 | `--offset=#` |
    | --- | --- |
    | 类型 | 数字 |

    跳过日志中的第*`N`*条目。

+   `--open-files-limit=*`N`*`

    | 命令行格式 | `--open-files-limit=#` |
    | --- | --- |
    | 类型 | 数字 |
    | 默认值 | `8` |
    | 最小值 | `1` |
    | 最大值 | `[依赖于平台]` |

    指定要保留的打开文件描述符的数量。

+   [`--password[=*`password`*]`](mysqlbinlog.html#option_mysqlbinlog_password), `-p[*`password`*]`

    | 命令行格式 | `--password[=password]` |
    | --- | --- |
    | 类型 | 字符串 |

    用于连接到服务器的 MySQL 帐户的密码。密码值是可选的。如果未提供，**mysqlbinlog**会提示输入密码。如果提供了密码，则`--password=`或`-p`后面不能有*空格*。如果未指定密码选项，则默认不发送密码。

    在命令行上指定密码应被视为不安全。为了避免在命令行上提供密码，请使用一个选项文件。参见第 8.1.2.1 节，“密码安全的最终用户指南”。

    要明确指定没有密码，并且**mysqlbinlog**不应提示输入密码，请使用`--skip-password`选项。

+   `--plugin-dir=*`dir_name`*`

    | 命令行格式 | `--plugin-dir=dir_name` |
    | --- | --- |
    | 类型 | 目录名称 |

    查找插件的目录。如果使用`--default-auth`选项指定身份验证插件但**mysqlbinlog**找不到它，请指定此选项。请参阅第 8.2.17 节，“可插拔认证”。

+   `--port=*`port_num`*`, `-P *`port_num`*`

    | 命令行格式 | `--port=port_num` |
    | --- | --- |
    | 类型 | 数值 |
    | 默认值 | `3306` |

    用于连接到远程服务器的 TCP/IP 端口号。

+   `--print-defaults`

    | 命令行格式 | `--print-defaults` |
    | --- | --- |

    打印程序名称以及从选项文件获取的所有选项。

    有关此选项文件和其他选项文件选项的更多信息，请参见第 6.2.2.3 节，“影响选项文件处理的命令行选项”。

+   `--print-table-metadata`

    | 命令行格式 | `--print-table-metadata` |
    | --- | --- |

    从二进制日志中打印与表相关的元数据。使用`binlog-row-metadata`配置二进制日志中记录的表相关元数据的数量。

+   `--protocol={TCP|SOCKET|PIPE|MEMORY}`

    | 命令行格式 | `--protocol=type` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `[查看文本]` |
    | 有效值 | `TCP``SOCKET``PIPE``MEMORY` |

    用于连接到服务器的传输协议。当其他连接参数通常导致使用您不想要的协议时，这很有用。有关允许值的详细信息，请参见第 6.2.7 节，“连接传输协议”。

+   `--raw`

    | 命令行格式 | `--raw` |
    | --- | --- |
    | 类型 | 布尔值 |
    | 默认值 | `FALSE` |

    默认情况下，**mysqlbinlog** 读取二进制日志文件并以文本格式写入事件。`--raw` 选项告诉 **mysqlbinlog** 以原始二进制格式写入它们。其使用需要同时使用 `--read-from-remote-server`，因为文件是从服务器请求的。**mysqlbinlog** 为从服务器读取的每个文件写入一个输出文件。`--raw` 选项可用于备份服务器的二进制日志。使用 `--stop-never` 选项，备份是“实时”的，因为 **mysqlbinlog** 保持与服务器的连接。默认情况下，输出文件写入当前目录，与原始日志文件具有相同的名称。可以使用 `--result-file` 选项修改输出文件名。有关更多信息，请参见 Section 6.6.9.3, “Using mysqlbinlog to Back Up Binary Log Files”。

+   `--read-from-remote-source=*`type`*`

    | 命令行格式 | `--read-from-remote-source=type` |
    | --- | --- |
    | 引入版本 | 8.0.26 |

    从 MySQL 8.0.26 开始，使用 `--read-from-remote-source`，在 MySQL 8.0.26 之前，使用 `--read-from-remote-master`。这两个选项具有相同的效果。这些选项通过将选项值设置为 `BINLOG-DUMP-NON-GTIDS` 或 `BINLOG-DUMP-GTIDS`，使用 `COM_BINLOG_DUMP` 或 `COM_BINLOG_DUMP_GTID` 命令从 MySQL 服务器读取二进制日志。如果 `--read-from-remote-source=BINLOG-DUMP-GTIDS` 或 `--read-from-remote-master=BINLOG-DUMP-GTIDS` 与 `--exclude-gtids` 结合使用，可以在源端过滤事务，避免不必要的网络流量。

    连接参数选项与这些选项或`--read-from-remote-server`选项一起使用。这些选项是`--host`、`--password`、`--port`、`--protocol`、`--socket`和`--user`。如果没有指定任何远程选项，则连接参数选项将被忽略。

    使用这些选项需要`REPLICATION SLAVE`权限。

+   `--read-from-remote-master=*`type`*`

    | 命令行格式 | `--read-from-remote-master=type` |
    | --- | --- |
    | 已弃用 | 8.0.26 |

    在 MySQL 8.0.26 之前，请使用此选项，而不是`--read-from-remote-source`。这两个选项具有相同的效果。

+   `--read-from-remote-server=*`file_name`*`, `-R`

    | 命令行格式 | `--read-from-remote-server=file_name` |
    | --- | --- |

    从 MySQL 服务器读取二进制日志，而不是读取本地日志文件。此选项要求远程服务器正在运行。它仅适用于远程服务器上的二进制日志文件，而不适用于中继日志文件。它接受二进制日志文件名（包括数字后缀），不包括文件路径。

    连接参数选项与此选项或`--read-from-remote-master`选项一起使用。这些选项是`--host`、`--password`、`--port`、`--protocol`、`--socket`和`--user`。如果没有指定任何远程选项，则连接参数选项将被忽略。

    使用此选项需要`REPLICATION SLAVE`权限。

    此选项类似于`--read-from-remote-master=BINLOG-DUMP-NON-GTIDS`。

+   `--result-file=*`name`*`, `-r *`name`*`

    | 命令行格式 | `--result-file=name` |
    | --- | --- |

    没有使用`--raw`选项，此选项表示**mysqlbinlog**写入文本输出的文件。使用`--raw`，**mysqlbinlog**为从服务器传输的每个日志文件写入一个二进制输出文件，默认情况下将它们写入当前目录，使用与原始日志文件相同的名称。在这种情况下，`--result-file`选项值被视为修改输出文件名的前缀。

+   `--require-row-format`

    | 命令行格式 | `--require-row-format` |
    | --- | --- |
    | 引入版本 | 8.0.19 |
    | 类型 | 布尔值 |
    | 默认值 | `false` |

    要求事件使用基于行的二进制日志格式。此选项强制**mysqlbinlog**输出使用基于行的复制事件。使用此选项生成的事件流将被使用`REQUIRE_ROW_FORMAT`选项的`CHANGE REPLICATION SOURCE TO`语句（从 MySQL 8.0.23 开始）或`CHANGE MASTER TO`语句（在 MySQL 8.0.23 之前）保护的复制通道接受。必须在写入二进制日志的服务器上设置`binlog_format=ROW`。当指定此选项时，**mysqlbinlog**在遇到任何不符合`REQUIRE_ROW_FORMAT`限制的事件时会停止，并打印错误消息，包括`LOAD DATA INFILE`指令，创建或删除临时表，`INTVAR`，`RAND`或`USER_VAR`事件，以及在 DML 事务中的非基于行的事件。**mysqlbinlog**还会在输出开头打印一个`SET @@session.require_row_format`语句，以在执行输出时应用限制，并不打印`SET @@session.pseudo_thread_id`语句。

    此选项在 MySQL 8.0.19 中添加。

+   `--rewrite-db='*`from_name`*->*`to_name`*'`

    | 命令行格式 | `--rewrite-db='oldname->newname'` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `[none]` |

    当从基于行或基于语句的日志中读取时，请将所有出现的*`from_name`*重写为*`to_name`*。对于基于行的日志，重写是在行上进行的，对于基于语句的日志，重写是在`USE`子句上进行的。

    警告

    使用此选项时，带有数据库名称限定的表名的语句不会被重写以使用新名称。

    作为此选项值的重写规则是一个字符串，其形式为`'*`from_name`*->*`to_name`*'`，如前所示，因此必须用引号括起来。

    要使用多个重写规则，请多次指定该选项，如下所示：

    ```sql
    mysqlbinlog --rewrite-db='dbcurrent->dbold' --rewrite-db='dbtest->dbcurrent' \
        binlog.00001 > /tmp/statements.sql
    ```

    与 `--database` 选项一起使用时，`--rewrite-db` 选项首先应用；然后使用重写后的数据库名称应用 `--database` 选项。在这方面，提供选项的顺序不会有任何影响。

    这意味着，例如，如果使用 `--rewrite-db='mydb->yourdb' --database=yourdb` 启动 **mysqlbinlog**，那么所有对数据库 `mydb` 和 `yourdb` 中任何表的更新都包含在输出中。另一方面，如果使用 `--rewrite-db='mydb->yourdb' --database=mydb` 启动，则 **mysqlbinlog** 根本不输出任何语句：因为所有对 `mydb` 的更新首先被重写为对 `yourdb` 的更新，然后应用 `--database` 选项，因此没有与 `--database=mydb` 匹配的更新。

+   `--server-id=*`id`*`

    | 命令行格式 | `--server-id=id` |
    | --- | --- |
    | 类型 | 数字 |

    仅显示由具有给定服务器 ID 的服务器创建的事件。

+   `--server-id-bits=*`N`*`

    | 命令行格式 | `--server-id-bits=#` |
    | --- | --- |
    | 类型 | 数字 |
    | 默认值 | `32` |
    | 最小值 | `7` |
    | 最大值 | `32` |

    仅使用`server_id`的前 *`N`* 位来标识服务器。如果二进制日志是由将 server-id-bits 设置为小于 32 并且用户数据存储在最高位的 **mysqld** 写入的，则运行 **mysqlbinlog** 并将 `--server-id-bits` 设置为 32 可以查看此数据。

    此选项仅由 NDB Cluster 分发的 **mysqlbinlog** 版本或带有 NDB Cluster 支持构建的版本支持。

+   `--server-public-key-path=*`file_name`*`

    | 命令行格式 | `--server-public-key-path=file_name` |
    | --- | --- |
    | 类型 | 文件名 |

    以 PEM 格式包含服务器所需的公钥的客户端端副本的文件路径名，用于基于 RSA 密钥对的密码交换。此选项适用于使用`sha256_password`或`caching_sha2_password`认证插件进行身份验证的客户端。对于不使用这些插件进行身份验证的帐户，此选项将被忽略。如果不使用基于 RSA 的密码交换，例如客户端使用安全连接连接到服务器时，此选项也将被忽略。

    如果提供了`--server-public-key-path=*`file_name`*`并指定了有效的公钥文件，则它优先于`--get-server-public-key`。

    对于`sha256_password`，此选项仅在 MySQL 使用 OpenSSL 构建时适用。

    有关`sha256_password`和`caching_sha2_password`插件的信息，请参见 Section 8.4.1.3, “SHA-256 Pluggable Authentication”和 Section 8.4.1.2, “Caching SHA-2 Pluggable Authentication”。

+   `--set-charset=*`charset_name`*`

    | 命令行格式 | `--set-charset=charset_name` |
    | --- | --- |
    | 类型 | 字符串 |

    向输出添加一个`SET NAMES *`charset_name`*`语句，指定用于处理日志文件的字符集。

+   `--shared-memory-base-name=*`name`*`

    | 命令行格式 | `--shared-memory-base-name=name` |
    | --- | --- |
    | 平台特定 | Windows |

    在 Windows 上，用于使用共享内存连接到本地服务器的共享内存名称。默认值为`MYSQL`。共享内存名称区分大小写。

    仅当服务器启用了`shared_memory`系统变量以支持共享内存连接时，此选项才适用。

+   `--short-form`, `-s`

    | 命令行格式 | `--short-form` |
    | --- | --- |

    仅显示日志中包含的语句，不包含任何额外信息或基于行的事件。仅用于测试，不应在生产系统中使用。已弃用，预计将在将来的版本中删除。

+   [`--skip-gtids[=(true|false)]`](mysqlbinlog.html#option_mysqlbinlog_skip-gtids)

    | 命令行格式 | `--skip-gtids[=true&#124;false]` |
    | --- | --- |
    | 类型 | 布尔值 |
    | 默认值 | `false` |

    不要在输出转储文件中包含二进制日志文件中的 GTIDs。例如：

    ```sql
    mysqlbinlog --skip-gtids binlog.000001 >  /tmp/dump.sql
    mysql -u root -p -e "source /tmp/dump.sql"
    ```

    通常不应在生产环境或恢复中使用此选项，除非在特定且罕见的情况下不需要 GTID。例如，管理员可能希望从一个部署复制选定的事务（如表定义）到另一个不相关的部署，该部署不会与原始部署进行复制。在这种情况下，`--skip-gtids` 可用于使管理员能够将事务应用为新事务，并确保部署保持不相关。但是，只有在 GTID 的包含对您的用例造成已知问题时才应使用此选项。

+   `--socket=*`路径`*`, `-S *`路径`*`

    | 命令行格式 | `--socket={file_name&#124;pipe_name}` |
    | --- | --- |
    | 类型 | 字符串 |

    对于连接到 `localhost` 的连接，使用的 Unix 套接字文件，或者在 Windows 上使用的命名管道的名称。

    在 Windows 上，此选项仅在服务器启动时启用了支持命名管道连接的 `named_pipe` 系统变量时才适用。此外，进行连接的用户必须是由 `named_pipe_full_access_group` 系统变量指定的 Windows 组的成员。

+   `--ssl*`

    以`--ssl`开头的选项指定是否使用加密连接到服务器，并指示 SSL 密钥和证书的位置。请参阅加密连接的命令选项。

+   `--ssl-fips-mode={OFF|ON|STRICT}`

    | 命令行格式 | `--ssl-fips-mode={OFF&#124;ON&#124;STRICT}` |
    | --- | --- |
    | 弃用 | 8.0.34 |
    | 类型 | 枚举 |
    | 默认值 | `OFF` |
    | 有效值 | `OFF``ON``STRICT` |

    控制是否在客户端端启用 FIPS 模式。`--ssl-fips-mode` 选项与其他 `--ssl-*`xxx`*` 选项不同，它不用于建立加密连接，而是影响允许哪些加密操作。请参阅第 8.8 节，“FIPS 支持”。

    允许使用以下 `--ssl-fips-mode` 值：

    +   `OFF`: 禁用 FIPS 模式。

    +   `ON`: 启用 FIPS 模式。

    +   `STRICT`: 启用“严格” FIPS 模式。

    注意

    如果 OpenSSL FIPS 对象模块不可用，则 `--ssl-fips-mode` 的唯一允许值为 `OFF`。在这种情况下，将 `--ssl-fips-mode` 设置为 `ON` 或 `STRICT` 会导致客户端在启动时产生警告并在非 FIPS 模式下运行。

    从 MySQL 8.0.34 开始，此选项已弃用。预计在将来的 MySQL 版本中将其移除。

+   `--start-datetime=*`datetime`*`

    | 命令行格式 | `--start-datetime=datetime` |
    | --- | --- |
    | 类型 | 日期时间 |

    从第一个具有时间戳等于或晚于 *`datetime`* 参数的事件开始读取二进制日志。*`datetime`* 值相对于运行 **mysqlbinlog** 的机器上的本地时区。该值应为 `DATETIME` 或 `TIMESTAMP` 数据类型接受的格式。例如：

    ```sql
    mysqlbinlog --start-datetime="2005-12-25 11:25:56" binlog.000003
    ```

    此选项对于点对点恢复很有用。请参阅 第 9.5 节，“时点（增量）恢复” Recovery")。

+   `--start-position=*`N`*`, `-j *`N`*`

    | 命令行格式 | `--start-position=#` |
    | --- | --- |
    | 类型 | 数值 |

    从日志位置 *`N`* 开始解码二进制日志，将输出任何从位置 *`N`* 或之后开始的事件。该位置是日志文件中的字节位置，而不是事件计数器；它需要指向一个事件的起始位置才能生成有用的输出。此选项适用于命令行中命名的第一个日志文件。

    在 MySQL 8.0.33 之前，此选项支持的最大值为 4294967295（2³²-1）。在 MySQL 8.0.33 及更高版本中，除非也使用 `--read-from-remote-server` 或 `--read-from-remote-source`，否则最大值为 18446744073709551616（2⁶⁴-1）。

    此选项对于点对点恢复很有用。请参阅 第 9.5 节，“时点（增量）恢复” Recovery")。

+   `--stop-datetime=*`datetime`*`

    | 命令行格式 | `--stop-datetime=datetime` |
    | --- | --- |

    在第一个具有时间戳等于或晚于 *`datetime`* 参数的事件处停止读取二进制日志。有关 *`datetime`* 值的信息，请参阅 `--start-datetime` 选项的描述。

    此选项对于点对点恢复很有用。请参阅 第 9.5 节，“时点（增量）恢复” Recovery")。

+   `--stop-never`

    | 命令行格式 | `--stop-never` |
    | --- | --- |
    | 类型 | 布尔值 |
    | 默认值 | `FALSE` |

    此选项与`--read-from-remote-server`一起使用。它告诉**mysqlbinlog**保持与服务器的连接。否则，当最后一个日志文件从服务器传输时，**mysqlbinlog**将退出。`--stop-never`意味着`--to-last-log`，因此只需要在命令行上命名要传输的第一个日志文件。

    `--stop-never`通常与`--raw`一起用于进行实时二进制日志备份，但也可以在不使用`--raw`的情况下使用，以保持服务器生成的日志事件的连续文本显示。

    使用`--stop-never`，默认情况下，**mysqlbinlog**在连接到服务器时报告服务器 ID 为 1。使用`--connection-server-id`来明确指定要报告的替代 ID。这可用于避免与复制服务器或另一个**mysqlbinlog**进程的 ID 冲突。参见第 6.6.9.4 节，“指定 mysqlbinlog 服务器 ID”。

+   `--stop-never-slave-server-id=*`id`*`

    | 命令行格式 | `--stop-never-slave-server-id=#` |
    | --- | --- |
    | 类型 | 数字 |
    | 默认值 | `65535` |
    | 最小值 | `1` |

    此选项已被弃用；预计将在将来的版本中删除。请改用`--connection-server-id`选项来指定**mysqlbinlog**报告的服务器 ID。

+   `--stop-position=*`N`*`

    | 命令行格式 | `--stop-position=#` |
    | --- | --- |
    | 类型 | 数字 |

    在日志位置 *`N`* 处停止解码二进制日志，从输出中排除从位置 *`N`* 开始的任何事件。该位置是日志文件中的一个字节点，而不是事件计数器；它需要指向最后要包含在输出中的事件的起始位置之后的位置。从位置 *`N`* 开始并在位置 *`N`* 或之后结束的事件是要处理的最后一个事件。此���项适用于命令行上命名的最后一个日志文件。

    此选项对于点对点恢复很有用。请参见第 9.5 节，“时点恢复（增量恢复）”。

+   `--tls-ciphersuites=*`ciphersuite_list`*`

    | 命令行格式 | `--tls-ciphersuites=ciphersuite_list` |
    | --- | --- |
    | 引入版本 | 8.0.16 |
    | 类型 | 字符串 |

    使用 TLSv1.3 的加密连接的可接受的密码套件。该值是一个或多个以冒号分隔的密码套件名称列表。可以为此选项命名的密码套件取决于用于编译 MySQL 的 SSL 库。有关详细信息，请参见第 8.3.2 节，“加密连接 TLS 协议和密码”。

    此选项是在 MySQL 8.0.16 中添加的。

+   `--tls-version=*`protocol_list`*`

    | 命令行格式 | `--tls-version=protocol_list` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值（≥ 8.0.16） | `TLSv1,TLSv1.1,TLSv1.2,TLSv1.3`（OpenSSL 1.1.1 或更高版本）`TLSv1,TLSv1.1,TLSv1.2`（否则） |
    | 默认值（≤ 8.0.15） | `TLSv1,TLSv1.1,TLSv1.2` |

    加密连接的可接受的 TLS 协议。该值是一个或多个逗号分隔的协议名称列表。可以为此选项命名的协议取决于用于编译 MySQL 的 SSL 库。有关详细信息，请参见第 8.3.2 节，“加密连接 TLS 协议和密码”。

+   `--to-last-log`, `-t`

    | 命令行格式 | `--to-last-log` |
    | --- | --- |

    不要在从 MySQL 服务器请求的二进制日志的末尾停止，而是继续打印直到最后一个二进制日志的末尾。如果将输出发送到同一台 MySQL 服务器，则可能导致无限循环。此选项需要`--read-from-remote-server`。

+   `--user=*`user_name`*`, `-u *`user_name`*`

    | 命令行格式 | `--user=user_name,` |
    | --- | --- |
    | 类型 | 字符串 |

    连接到远程服务器时要使用的 MySQL 帐户的用户名。

    如果您在 MySQL 8.0.31 或更高版本中使用 `Rewriter` 插件，则应授予该用户`SKIP_QUERY_REWRITE` 权限。

+   `--verbose`, `-v`

    | 命令行格式 | `--verbose` |
    | --- | --- |

    重建行事件并将其显示为带有表分区信息的注释 SQL 语句。如果此选项被传入两次（通过"-vv"或"--verbose --verbose"），输出将包括注释以指示列数据类型和一些元数据，以及信息日志事件，如`binlog_rows_query_log_events`系统变量设置为`TRUE`时的行查询日志事件。

    有关`--base64-output`和`--verbose`对行事件输出的影响的示例，请参见第 6.6.9.2 节，“mysqlbinlog 行事件显示”。

+   `--verify-binlog-checksum`, `-c`

    | 命令行格式 | `--verify-binlog-checksum` |
    | --- | --- |

    验证二进制日��文件中的校验和。

+   `--version`, `-V`

    | 命令行格式 | `--version` |
    | --- | --- |

    显示版本信息并退出。

    与 MySQL 先前版本不同，使用此选项时**mysqlbinlog**显示的版本号与 MySQL 服务器版本相同。

+   `--zstd-compression-level=*`level`*`

    | 命令行格式 | `--zstd-compression-level=#` |
    | --- | --- |
    | 引入版本 | 8.0.18 |
    | 类型 | 整数 |

    用于使用`zstd`压缩算法连接到服务器的压缩级别。允许的级别为 1 到 22，较大的值表示较高级别的压缩。默认的`zstd`压缩级别为 3。压缩级别设置对不使用`zstd`压缩的连接没有影响。

    更多信息，请参见第 6.2.8 节，“连接压缩控制”。

    此选项在 MySQL 8.0.18 中添加。

你可以将**mysqlbinlog**的输出导入到**mysql**客户端中，以执行二进制日志中包含的事件。这种技术用于在出现意外退出时从旧备份中恢复（参见第 9.5 节，“时间点（增量）恢复”）。例如：

```sql
mysqlbinlog binlog.000001 | mysql -u root -p
```

或者：

```sql
mysqlbinlog binlog.[0-9]* | mysql -u root -p
```

如果**mysqlbinlog**生成的语句可能包含`BLOB`值，当**mysql**处理它们时可能会出现问题。在这种情况下，使用`--binary-mode`选项调用**mysql**。

如果需要先修改语句日志（例如删除某些不想执行的语句），还可以将**mysqlbinlog**的输出重定向到文本文件。编辑文件后，通过将其作为输入传递给**mysql**程序来执行其中包含的语句：

```sql
mysqlbinlog binlog.000001 > tmpfile
... *edit tmpfile* ...
mysql -u root -p < tmpfile
```

当使用`--start-position`选项调用**mysqlbinlog**时，它仅显示二进制日志中偏移大于或等于给定位置的事件（给定位置必须与一个事件的开始匹配）。它还具有在看到具有给定日期和时间的事件时停止和开始的选项。这使您可以使用`--stop-datetime`选项执行时间点恢复（例如，能够说“将我的数据库回滚到今天上午 10:30 的状态”）。

**处理多个文件。** 如果您有多个二进制日志要在 MySQL 服务器上执行，安全的方法是使用单个连接处理它们。以下是一个示例，演示了可能是*不安全*的情况：

```sql
mysqlbinlog binlog.000001 | mysql -u root -p # DANGER!!
mysqlbinlog binlog.000002 | mysql -u root -p # DANGER!!
```

使用多个连接到服务器处理二进制日志会导致问题，如果第一个日志文件包含一个`CREATE TEMPORARY TABLE`语句，而第二个日志包含使用临时表的语句。当第一个**mysql**进程终止时，服务器会删除临时表。当第二个**mysql**进程尝试使用该表时，服务器会报告“未知表”。

为避免这样的问题，使用一个*单一的* **mysql** 进程执行您想要处理的所有二进制日志的内容。以下是一种方法：

```sql
mysqlbinlog binlog.000001 binlog.000002 | mysql -u root -p
```

另一种方法是将所有日志写入单个文件，然后处理该文件：

```sql
mysqlbinlog binlog.000001 >  /tmp/statements.sql
mysqlbinlog binlog.000002 >> /tmp/statements.sql
mysql -u root -p -e "source /tmp/statements.sql"
```

从 MySQL 8.0.12 开始，您还可以使用 shell 管道将多个二进制日志文件作为流输入提供给 **mysqlbinlog**。 可以解压缩压缩的二进制日志文件存档，并直接提供给 **mysqlbinlog**。 在这个例子中，`binlog-files_1.gz` 包含多个要处理的二进制日志文件。 管道提取 `binlog-files_1.gz` 的内容，将二进制日志文件传输到 **mysqlbinlog** 作为标准输入，并将 **mysqlbinlog** 的输出传输到 **mysql** 客户端以执行：

```sql
gzip -cd binlog-files_1.gz | ./mysqlbinlog - | ./mysql -uroot  -p
```

您可以指定多个存档文件，例如：

```sql
gzip -cd binlog-files_1.gz binlog-files_2.gz | ./mysqlbinlog - | ./mysql -uroot  -p
```

对于流输入，请勿使用 `--stop-position`，因为 **mysqlbinlog** 无法识别要应用此选项的最后一个日志文件。

**LOAD DATA 操作。** **mysqlbinlog** 可以生成一个输出，可以重现一个不带原始数据文件的 `LOAD DATA` 操作。 **mysqlbinlog** 将数据复制到一个临时文件中，并编写一个引用该文件的 `LOAD DATA LOCAL` 语句。 这些文件写入的默认目录位置是特定于系统的。 要明确指定目录，请使用 `--local-load` 选项。

因为 **mysqlbinlog** 将 `LOAD DATA` 语句转换为 `LOAD DATA LOCAL` 语句（即，它添加了 `LOCAL`），因此用于处理这些语句的客户端和服务器都必须配置为启用 `LOCAL` 能力。 请参阅 第 8.1.6 节，“LOAD DATA LOCAL 的安全注意事项”。

警告

为 `LOAD DATA LOCAL` 语句创建的临时文件 *不会* 自动删除，因为直到您实际执行这些语���之前都需要它们。 在您不再需要语句日志后，应自行删除临时文件。 这些文件可以在临时文件目录中找到，名称类似 *`original_file_name-#-#`*。
