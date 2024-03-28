# 6.5.6 mysqlpump — 一个数据库备份程序

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysqlpump.html`](https://dev.mysql.com/doc/refman/8.0/en/mysqlpump.html)

+   mysqlpump 调用语法

+   mysqlpump 选项摘要

+   mysqlpump 选项描述

+   mysqlpump 对象选择

+   mysqlpump 并行处理

+   mysqlpump 限制

**mysqlpump**客户端实用程序执行逻辑备份，生成一组可以执行的 SQL 语句，以重现原始数据库对象定义和表数据。它为备份或传输到另一个 SQL 服务器转储一个或多个 MySQL 数据库。

注意

**mysqlpump**在 MySQL 8.0.34 中已弃用；预计将在未来的 MySQL 版本中删除。您可以使用 MySQL 程序，如**mysqldump**和 MySQL Shell 来执行逻辑备份、转储数据库和类似任务。

提示

考虑使用 MySQL Shell 转储实用程序，提供多线程并行转储、文件压缩和进度信息显示，以及云功能，如 Oracle Cloud Infrastructure Object Storage 流式传输和 MySQL HeatWave Service 兼容性检查和修改。转储可以轻松导入到 MySQL 服务器实例或 MySQL HeatWave Service DB 系统中，使用 MySQL Shell 加载转储实用程序。MySQL Shell 的安装说明可以在这里找到。

**mysqlpump**的功能包括：

+   并行处理数据库和数据库内的对象，以加快转储过程

+   更好地控制要转储的数据库和数据库对象（表、存储过程、用户帐户）

+   转储用户帐户作为帐户管理语句（`CREATE USER`，`GRANT`），而不是作为插入到`mysql`系统数据库中

+   创建压缩输出的能力

+   进度指示器（值为估计值）

+   对于通过在插入行后添加索引来加快`InnoDB`表的辅助索引创建的转储文件重新加载

注意

**mysqlpump** 使用了 MySQL 5.7 中引入的功能，因此假定与 MySQL 5.7 或更高版本一起使用。

**mysqlpump** 至少需要对被转储表具有 `SELECT` 权限，对被转储视图具有 `SHOW VIEW` 权限，对被转储触发器具有 `TRIGGER` 权限，如果未使用 `--single-transaction` 选项，则需要 `LOCK TABLES` 权限。需要对 `mysql` 系统数据库具有 `SELECT` 权限以转储用户定义。某些选项可能需要其他权限，如选项描述中所述。

要重新加载转储文件，您必须具有执行其中包含的语句所需的权限，例如这些语句创建的对象所需的适当 `CREATE` 权限。

注意

在 Windows 上使用 PowerShell 进行输出重定向创建的转储文件具有 UTF-16 编码：

```sql
mysqlpump [options] > dump.sql
```

但是，不允许使用 UTF-16 作为连接字符集（请参阅 第 12.4 节，“连接字符集和校对规则”），因此无法正确加载转储文件。要解决此问题，请使用 `--result-file` 选项，以 ASCII 格式创建输出：

```sql
mysqlpump [options] --result-file=dump.sql
```

#### mysqlpump 调用语法

默认情况下，**mysqlpump** 转储所有数据库（在 mysqlpump 限制 中有特定例外）。要明确指定此行为，请使用 `--all-databases` 选项：

```sql
mysqlpump --all-databases
```

要转储单个数据库或该数据库中的某些表，请在命令行上命名数据库，可选地跟随表名：

```sql
mysqlpump *db_name*
mysqlpump *db_name tbl_name1 tbl_name2 ...*
```

要将所有名称参数视为数据库名称，请使用 `--databases` 选项：

```sql
mysqlpump --databases *db_name1 db_name2* ...
```

默认情况下，**mysqlpump** 不会转储用户帐户定义，即使您转储包含授权表的 `mysql` 系统数据库。要以逻辑定义的形式转储授权表内容，如 `CREATE USER` 和 `GRANT` 语句，使用 `--users` 选项并禁止所有数据库转储：

```sql
mysqlpump --exclude-databases=% --users
```

在上述命令中，`%` 是一个通配符，用于匹配所有数据库名称以供 `--exclude-databases` 选项使用。

**mysqlpump** 支持几个选项，用于包括或排除数据库、表、存储程序和用户定义。请参阅 mysqlpump 对象选择。

要重新加载转储文件，请执行其中包含的语句。例如，使用 **mysql** 客户端：

```sql
mysqlpump [options] > dump.sql
mysql < dump.sql
```

以下讨论提供了额外的 **mysqlpump** 使用示例。

要查看 **mysqlpump** 支持的选项列表，请发出命令 **mysqlpump --help**。

#### mysqlpump 选项摘要

**mysqlpump** 支持以下选项，可以在命令行或选项文件的 `[mysqlpump]` 和 `[client]` 组中指定。（在 MySQL 8.0.20 之前，**mysqlpump** 读取 `[mysql_dump]` 组而不是 `[mysqlpump]`。从 8.0.20 开始，仍然接受 `[mysql_dump]` 但已被弃用。）有关 MySQL 程序使用的选项文件的信息，请参见 6.2.2.2 使用选项文件。

**表 6.17 mysqlpump 选项**

| 选项名称 | 描述 | 引入版本 | 废弃版本 |
| --- | --- | --- | --- |
| --add-drop-database | 在每个 CREATE DATABASE 语句之前添加 DROP DATABASE 语句 |  |  |
| --add-drop-table | 在每个 CREATE TABLE 语句之前添加 DROP TABLE 语句 |  |  |
| --add-drop-user | 在每个 CREATE USER 语句之前添加 DROP USER 语句 |  |  |
| --add-locks | 在每个表转储周围加上 LOCK TABLES 和 UNLOCK TABLES 语句 |  |  |
| --all-databases | 转储所有数据库 |  |  |
| --bind-address | 使用指定的网络接口连接到 MySQL 服务器 |  |  |
| --character-sets-dir | 安装字符集的目录 |  |  |
| --column-statistics | 写入 ANALYZE TABLE 语句以生成统计直方图 |  |  |
| --complete-insert | 使用包括列名的完整 INSERT 语句 |  |  |
| --compress | 压缩客户端和服务器之间发送的所有信息 |  | 8.0.18 |
| --compress-output | 输出压缩算法 |  |  |
| --compression-algorithms | 允许连接到服务器的压缩算法 | 8.0.18 |  |
| --databases | 将所有名称参数解释为数据库名称 |  |  |
| --debug | 写入调试日志 |  |  |
| --debug-check | 在程序退出时打印调试信息 |  |  |
| --debug-info | 在程序退出时打印调试信息、内存和 CPU 统计信息 |  |  |
| --default-auth | 要使用的认证插件 |  |  |
| --default-character-set | 指定默认字符集 |  |  |
| --default-parallelism | 并行处理的默认线程数 |  |  |
| --defaults-extra-file | 除了通常的选项文件外，还读取指定的选项文件 |  |  |
| --defaults-file | 仅读取指定的选项文件 |  |  |
| --defaults-group-suffix | 选项组后缀值 |  |  |
| --defer-table-indexes | 重新加载时，推迟索引创建，直到加载表行后 |  |  |
| --events | 从转储的数据库中转储事件 |  |  |
| --exclude-databases | 从转储中排除的数据库 |  |  |
| --exclude-events | 从转储中排除的事件 |  |  |
| --exclude-routines | 从转储中排除的例程 |  |  |
| --exclude-tables | 从转储中排除的表 |  |  |
| --exclude-triggers | 从转储中排除的触发器 |  |  |
| --exclude-users | 从转储中排除的用户 |  |  |
| --extended-insert | 使用多行 INSERT 语法 |  |  |
| --get-server-public-key | 从服务器请求 RSA 公钥 |  |  |
| --help | 显示帮助消息并退出 |  |  |
| --hex-blob | 使用十六进制表示法转储二进制列 |  |  |
| --host | MySQL 服务器所在的主机 |  |  |
| --include-databases | 转储中包含的数据库 |  |  |
| --include-events | 转储中要包含的事件 |  |  |
| --include-routines | 转储中要包含的例程 |  |  |
| --include-tables | 转储中要包含的表 |  |  |
| --include-triggers | 转储中要包含的触发器 |  |  |
| --include-users | 转储中要包含的用户 |  |  |
| --insert-ignore | 写入 INSERT IGNORE 而不是 INSERT 语句 |  |  |
| --log-error-file | 将警告和错误追加到指定文件 |  |  |
| --login-path | 从.mylogin.cnf 中读取登录路径选项 |  |  |
| --max-allowed-packet | 发送到服务器或从服务器接收的最大数据包长度 |  |  |
| --net-buffer-length | TCP/IP 和套接字通信的缓冲区大小 |  |  |
| --no-create-db | 不写入 CREATE DATABASE 语句 |  |  |
| --no-create-info | 不写入重新创建每个转储表的 CREATE TABLE 语句 |  |  |
| --no-defaults | 不读取任何选项文件 |  |  |
| --parallel-schemas | 指定模式处理的并行性 |  |  |
| --password | 连接到服务器时要使用的密码 |  |  |
| --password1 | 连接到服务器时要使用的第一个多因素身份验证密码 | 8.0.27 |  |
| --password2 | 连接到服务器时要使用的第二个多因素身份验证密码 | 8.0.27 |  |
| --password3 | 连接到服务器时要使用的第三个多因素身份验证密码 | 8.0.27 |  |
| --plugin-dir | 安装插件的目录 |  |  |
| --port | 连接的 TCP/IP 端口号 |  |  |
| --print-defaults | 打印默认选项 |  |  |
| --protocol | 要使用的传输协议 |  |  |
| --replace | 写入 REPLACE 语句而不是 INSERT 语句 |  |  |
| --result-file | 直接输出到指定文件 |  |  |
| --routines | 从转储的数据库中转储存储例程（过程和函数） |  |  |
| --server-public-key-path | 包含 RSA 公钥的文件路径名 |  |  |
| --set-charset | 将 SET NAMES default_character_set 添加到输出 |  |  |
| --set-gtid-purged | 是否将 SET @@GLOBAL.GTID_PURGED 添加到输出 |  |  |
| --single-transaction | 在单个事务中转储表 |  |  |
| --skip-definer | 从视图和存储程序 CREATE 语句中省略 DEFINER 和 SQL SECURITY 子句 |  |  |
| --skip-dump-rows | 不转储表行 |  |  |
| --skip-generated-invisible-primary-key | 不转储生成的不可见主键的信息 | 8.0.30 |  |
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
| --triggers | 为每个转储表转储触发器 |  |  |
| --tz-utc | 将 SET TIME_ZONE='+00:00' 添加到转储文件 |  |  |
| --user | 连接到服务器时要使用的 MySQL 用户名 |  |  |
| --用户 | 转储用户账户 |  |  |
| --version | 显示版本信息并退出 |  |  |
| --watch-progress | 显示进度指示器 |  |  |
| --zstd-compression-level | 用于使用 zstd 压缩的服务器连接的压缩级别 | 8.0.18 |  |
| 选项名称 | 描述 | 引入 | 废弃 |

#### mysqlpump 选项描述

+   `--help`，`-?`

    | 命令行格式 | `--help` |
    | --- | --- |

    显示帮助信息并退出。

+   `--add-drop-database`

    | 命令行格式 | `--add-drop-database` |
    | --- | --- |

    在每个 `CREATE DATABASE` 语句之前写一个 `DROP DATABASE` 语句。

    注意

    在 MySQL 8.0 中，`mysql` 模式被视为系统模式，不能被最终用户删除。如果与 `--all-databases` 或 `--databases` 一起使用 `--add-drop-database`，其中要转储的模式列表包括 `mysql`，则转储文件包含一个导致重新加载转储文件时出错的 `DROP DATABASE `mysql`` 语句。

    相反，要使用 `--add-drop-database`，请使用 `--databases` 与要转储的模式列表，其中列表不包括 `mysql`。

+   `--add-drop-table`

    | 命令行格式 | `--add-drop-table` |
    | --- | --- |

    在每个 `CREATE TABLE` 语句之前写一个 `DROP TABLE` 语句。

+   `--add-drop-user`

    | 命令行格式 | `--add-drop-user` |
    | --- | --- |

    在每个 `CREATE USER` 语句之前写一个 `DROP USER` 语句。

+   `--add-locks`

    | 命令行格式 | `--add-locks` |
    | --- | --- |

    将每个表转储用 `LOCK TABLES` 和 `UNLOCK TABLES` 语句包围起来。这样在重新加载转储文件时插入速度更快。参见 Section 10.2.5.1, “Optimizing INSERT Statements”。

    该选项与并行性不兼容，因为来自不同表的 `INSERT` 语句可能会交错，并且在一个表的插入结束后跟随的 `UNLOCK TABLES` 可能会释放仍然存在插入的表上的锁。

    `--add-locks` 和 `--single-transaction` 是互斥的。

+   `--all-databases`, `-A`

    | 命令行格式 | `--all-databases` |
    | --- | --- |

    转储所有数据库（有一些例外情况，请参阅 mysqlpump 限制）。如果没有明确指定其他行为，则这是默认行为。

    `--all-databases` 和 `--databases` 是互斥的。

    注意

    有关该选项与 `--all-databases` 不兼容的信息，请参阅 `--add-drop-database` 描述。

    在 MySQL 8.0 之前，使用 `--all-databases` 选项时，**mysqldump** 和 **mysqlpump** 的 `--routines` 和 `--events` 选项不需要包括存储过程和事件的定义：转储包括 `mysql` 系统数据库，因此也包括包含存储过程和事件定义的 `mysql.proc` 和 `mysql.event` 表。从 MySQL 8.0 开始，不再使用 `mysql.event` 和 `mysql.proc` 表。相应对象的定义存储在数据字典表中，但这些表不会被转储。要在使用 `--all-databases` 进行转储时包括存储过程和事件，请明确使用 `--routines` 和 `--events` 选项。

+   `--bind-address=*`ip_address`*`

    | 命令行格式 | `--bind-address=ip_address` |
    | --- | --- |

    在具有多个网络接口的计算机上，使用此选项选择连接到 MySQL 服务器的接口。

+   `--character-sets-dir=*`path`*`

    | 命令行格式 | `--character-sets-dir=dir_name` |
    | --- | --- |
    | 类型 | 目录名称 |

    安装字符集的目录。请参阅 第 12.15 节，“字符集配置”。

+   `--column-statistics`

    | 命令行格式 | `--column-statistics` |
    | --- | --- |
    | 类型 | 布尔 |
    | 默认值 | `OFF` |

    在重新加载转储文件时，向输出中添加`ANALYZE TABLE`语句，为转储表生成直方图统计信息。默认情况下，此选项已禁用，因为为大型表生成直方图可能需要很长时间。

+   `--complete-insert`

    | 命令行格式 | `--complete-insert` |
    | --- | --- |

    编写包含列名的完整`INSERT`语句。

+   `--compress`, `-C`

    | 命令行格式 | `--compress[={OFF&#124;ON}]` |
    | --- | --- |
    | 已弃用 | 8.0.18 |
    | 类型 | 布尔 |
    | 默认值 | `OFF` |

    如果可能，压缩客户端和服务器之间发送的所有信息。请参阅第 6.2.8 节，“连接压缩控制”。

    从 MySQL 8.0.18 开始，此选项已被弃用。预计在未来的 MySQL 版本中将其移除。请参阅配置传统连接压缩。

+   `--compress-output=*`algorithm`*`

    | 命令行格式 | `--compress-output=algorithm` |
    | --- | --- |
    | 类型 | 枚举 |
    | 有效值 | `LZ4``ZLIB` |

    默认情况下，**mysqlpump**不会压缩输出。���选项指定使用指定算法进行输出压缩。允许的算法为`LZ4`和`ZLIB`。

    要解压缩压缩的输出，您必须拥有适当的实用程序。如果系统命令**lz4**和**openssl zlib**不可用，MySQL 发行版包括可用于解压缩使用`--compress-output=LZ4`和`--compress-output=ZLIB`选项压缩的**mysqlpump**输出的**lz4_decompress**和**zlib_decompress**实用程序。有关更多信息，请参阅第 6.8.1 节，“lz4_decompress — 解压缩 mysqlpump LZ4 压缩输出”和第 6.8.3 节，“zlib_decompress — 解压缩 mysqlpump ZLIB 压缩输出”。

+   `--compression-algorithms=*`value`*`

    | 命令行格式 | `--compression-algorithms=value` |
    | --- | --- |
    | 引入 | 8.0.18 |
    | 类型 | 设置 |
    | 默认值 | `uncompressed` |
    | 有效值 | `zlib``zstd``uncompressed` |

    用于与服务器连接的允许压缩算法。可用算法与`protocol_compression_algorithms`系统变量相同。默认值为`uncompressed`。

    查看更多信息，请参阅第 6.2.8 节，“连接压缩控制”。

    此选项在 MySQL 8.0.18 中添加。

+   `--databases`，`-B`

    | 命令行格式 | `--databases` |
    | --- | --- |

    通常，**mysqlpump**将命令行上的第一个名称参数视为数据库名称，后续名称视为表名称。使用此选项，它将所有名称参数视为数据库名称。在每个新数据库之前，输出中包括`CREATE DATABASE`语句。

    `--all-databases`和`--databases`是互斥的。

    注意

    有关该选项与`--databases`不兼容的信息，请参阅`--add-drop-database`描述。

+   [`--debug[=*`debug_options`*]`](mysqlpump.html#option_mysqlpump_debug)，`-# [*`debug_options`*]`

    | 命令行格式 | `--debug[=debug_options]` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `d:t:O,/tmp/mysqlpump.trace` |

    写入调试日志。典型的*`debug_options`*字符串为`d:t:o,*`file_name`*`。默认值为`d:t:O,/tmp/mysqlpump.trace`。

    仅当 MySQL 使用`WITH_DEBUG`构建时才可用。由 Oracle 提供的 MySQL 发布二进制文件*不*使用此选项构建。

+   `--debug-check`

    | 命令行格式 | `--debug-check` |
    | --- | --- |
    | 类型 | 布尔值 |
    | 默认值 | `FALSE` |

    在程序退出时打印一些调试信息。

    仅当 MySQL 使用`WITH_DEBUG`构建时才可用。由 Oracle 提供的 MySQL 发布二进制文件*不*使用此选项构建。

+   `--debug-info`，`-T`

    | 命令行格式 | `--debug-info` |
    | --- | --- |
    | 类型 | 布尔值 |
    | 默认值 | `FALSE` |

    在程序退出时打印调试信息以及内存和 CPU 使用统计信息。

    仅当 MySQL 使用`WITH_DEBUG`构建时才可用。由 Oracle 提供的 MySQL 发布二进制文件*不*使用此选项构建。

+   `--default-auth=*`plugin`*`

    | 命令行格式 | `--default-auth=plugin` |
    | --- | --- |
    | 类型 | 字符串 |

    有关要使用的客户端端身份验证插件的提示。参见第 8.2.17 节，“可插拔认证”。

+   `--default-character-set=*`charset_name`*`

    | 命令行格式 | `--default-character-set=charset_name` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `utf8` |

    将*`charset_name`*作为默认字符集。参见第 12.15 节，“字符集配置”。如果未指定字符集，**mysqlpump**将使用`utf8mb4`。

+   `--default-parallelism=*`N`*`

    | 命令行格式 | `--default-parallelism=N` |
    | --- | --- |
    | 类型 | 整数 |
    | 默认值 | `2` |

    每个并行处理队列的默认线程数。默认值为 2。

    `--parallel-schemas`选项也会影响并行性，并可用于覆盖默认线程数。有关更多信息，请参见 mysqlpump 并行处理。

    使用`--default-parallelism=0`和没有`--parallel-schemas`选项时，**mysqlpump**将作为单线程进程运行，并不创建队列。

    启用并行性后，不同数据库的输出可能会交错。

+   `--defaults-extra-file=*`file_name`*`

    | 命令行格式 | `--defaults-extra-file=file_name` |
    | --- | --- |
    | 类型 | 文件名 |

    在全局选项文件之后但（在 Unix 上）在用户选项文件之前读取此选项文件。如果文件不存在或无法访问，将会出现错误。如果*`file_name`*不是绝对路径名，则将其解释为相对于当前目录。

    有关此选项文件和其他选项文件选项的其他信息，请参见第 6.2.2.3 节，“影响选项文件处理的命令行选项”。

+   `--defaults-file=*`file_name`*`

    | 命令行格式 | `--defaults-file=file_name` |
    | --- | --- |
    | 类型 | 文件名 |

    仅使用给定的选项文件。如果文件不存在或无法访问，将会出现错误。如果*`file_name`*不是绝对路径名，则将其解释为相对于当前目录。

    例外：即使使用了`--defaults-file`，客户端程序也会读取`.mylogin.cnf`。

    有关此选项和其他选项文件选项的更多信息，请参见第 6.2.2.3 节，“影响选项文件处理的命令行选项”。

+   `--defaults-group-suffix=*`str`*`

    | 命令行格式 | `--defaults-group-suffix=str` |
    | --- | --- |
    | 类型 | 字符串 |

    不仅读取常规选项组，还读取具有常规名称和后缀*`str`*的组。例如，**mysqlpump**通常会读取`[client]`和`[mysqlpump]`组。如果此选项给定为`--defaults-group-suffix=_other`，**mysqlpump**还会读取`[client_other]`和`[mysqlpump_other]`组。

    有关此选项和其他选项文件选项的更多信息，请参见第 6.2.2.3 节，“影响选项文件处理的命令行选项”。

+   `--延迟表索引`

    | 命令行格式 | `--defer-table-indexes` |
    | --- | --- |
    | 类型 | 布尔 |
    | 默认值 | `TRUE` |

    在转储输出中，延迟为每个表创建索引，直到其行加载完毕。这适用于所有存储引擎，但对于`InnoDB`仅适用于二级索引。

    此选项默认启用；使用`--skip-defer-table-indexes`来禁用它。

+   `--事件`

    | 命令行格式 | `--events` |
    | --- | --- |
    | 类型 | 布尔 |
    | 默认值 | `TRUE` |

    在输出中包含转储数据库中的事件调度器事件。事件转储需要这些数据库的`EVENT`权限。

    使用`--事件`生成的输出包含`CREATE EVENT`语句来创建事件。

    此选项默认启用；使用`--skip-events`来禁用它。

+   `--排除数据库=*`db_list`*`

    | 命令行格式 | `--exclude-databases=db_list` |
    | --- | --- |
    | 类型 | 字符串 |

    不转储*`db_list`*中的数据库，其中*`db_list`*是一个或多个逗号分隔的数据库名称列表。此选项的多个实例是累加的。有关更多信息，请参见 mysqlpump 对象选择。

+   `--排除事件=*`event_list`*`

    | 命令行格式 | `--exclude-events=event_list` |
    | --- | --- |
    | 类型 | 字符串 |

    不要转储*`event_list`*中的数据库，这是一个包含一个或多个逗号分隔的事件名称的列表。此选项的多个实例是累加的。有关更多信息，请参见 mysqlpump 对象选择。

+   `--exclude-routines=*`routine_list`*`

    | 命令行格式 | `--exclude-routines=routine_list` |
    | --- | --- |
    | 类型 | 字符串 |

    不要转储*`routine_list`*中的事件，这是一个包含一个或多个逗号分隔的例程（存储过程或函数）名称的列表。此选项的多个实例是累加的。有关更多信息，请参见 mysqlpump 对象选择。

+   `--exclude-tables=*`table_list`*`

    | 命令行格式 | `--exclude-tables=table_list` |
    | --- | --- |
    | 类型 | 字符串 |

    不要转储*`table_list`*中的表，这是一个包含一个或多个逗号分隔的表名称的列表。此选项的多个实例是累加的。有关更多信息，请参见 mysqlpump 对象选择。

+   `--exclude-triggers=*`trigger_list`*`

    | 命令行格式 | `--exclude-triggers=trigger_list` |
    | --- | --- |
    | 类型 | 字符串 |

    不要转储*`trigger_list`*中的触发器，这是一个包含一个或多个逗号分隔的触发器名称的列表。此选项的多个实例是累加的。有关更多信息，请参见 mysqlpump 对象选择。

+   `--exclude-users=*`user_list`*`

    | 命令行格式 | `--exclude-users=user_list` |
    | --- | --- |
    | 类型 | 字符串 |

    不要转储*`user_list`*中的用户账户，这是一个包含一个或多个逗号分隔的账户名称的列表。此选项的多个实例是累加的。有关更多信息，请参见 mysqlpump 对象选择。

+   `--extended-insert=*`N`*`

    | 命令行格式 | `--extended-insert=N` |
    | --- | --- |

    使用包含多个`VALUES`列表的多行语法编写`INSERT`语句。这样可以产生较小的转储文件，并在重新加载文件时加快插入速度。

    选项值表示每个`INSERT`语句中要包含的行数。默认值为 250。值为 1 会产生每个表行一个`INSERT`语句。

+   `--get-server-public-key`

    | 命令行格式 | `--get-server-public-key` |
    | --- | --- |
    | 类型 | 布尔值 |

    从服务器请求用于 RSA 密钥对密码交换所需的公钥。此选项适用于使用`caching_sha2_password`认证插件进行身份验证的客户端。对于该插件，除非请求，否则服务器不会发送公钥。对于不使用该插件进行身份验证的帐户，此选项将被忽略。如果不使用基于 RSA 的密码交换，例如客户端使用安全连接连接到服务器时，此选项也将被忽略。

    如果给定`--server-public-key-path=*`file_name`*`并指定有效的公钥文件，则优先于`--get-server-public-key`。

    有关`caching_sha2_password`插件的信息，请参阅第 8.4.1.2 节，“缓存 SHA-2 可插拔认证”。

+   `--hex-blob`

    | 命令行格式 | `--hex-blob` |
    | --- | --- |

    使用十六进制表示法转储二进制列（例如，`'abc'`变为`0x616263`）。受影响的数据类型包括`BINARY`、`VARBINARY`、`BLOB`类型、`BIT`、所有空间数据类型以及其他非二进制数据类型（与`binary`字符集一起使用时）。

+   `--host=*`host_name`*`, `-h *`host_name`*`

    | 命令行格式 | `--host` |
    | --- | --- |

    从给定主机上的 MySQL 服务器转储数据。

+   `--include-databases=*`db_list`*`

    | 命令行格式 | `--include-databases=db_list` |
    | --- | --- |
    | 类型 | 字符串 |

    转储*`db_list`*中的数据库，这是一个包含一个或多个逗号分隔数据库名称的列表。转储包括命名数据库中的所有对象。此选项的多个实例是累加的。有关更多信息，请参阅 mysqlpump 对象选择。

+   `--include-events=*`event_list`*`

    | 命令行格式 | `--include-events=event_list` |
    | --- | --- |
    | 类型 | 字符串 |

    转储*`event_list`*中的事件，这是一个包含一个或多个逗号分隔事件名称的列表。此选项的多个实例是累加的。有关更多信息，请参阅 mysqlpump 对象选择。

+   `--include-routines=*`routine_list`*`

    | 命令行格式 | `--include-routines=routine_list` |
    | --- | --- |
    | 类型 | 字符串 |

    转储*`routine_list`*中的例程，这是一个包含一个或多个逗号分隔的例程（存储过程或函数）名称的列表。此选项的多个实例是累加的。有关更多信息，请参见 mysqlpump 对象选择。

+   `--include-tables=*`table_list`*`

    | 命令行格式 | `--include-tables=table_list` |
    | --- | --- |
    | 类型 | 字符串 |

    转储*`table_list`*中的表，这是一个包含一个或多个逗号分隔的表名称的列表。此选项的多个实例是累加的。有关更多信息，请参见 mysqlpump 对象选择。

+   `--include-triggers=*`trigger_list`*`

    | 命令行格式 | `--include-triggers=trigger_list` |
    | --- | --- |
    | 类型 | 字符串 |

    转储*`trigger_list`*中的触发器，这是一个包含一个或多个逗号分隔的触发器名称的列表。此选项的多个实例是累加的。有关更多信息，请参见 mysqlpump 对象选择。

+   `--include-users=*`user_list`*`

    | 命令行格式 | `--include-users=user_list` |
    | --- | --- |
    | 类型 | 字符串 |

    转储*`user_list`*中的用户帐户，这是一个包含一个或多个逗号分隔的用户名称的列表。此选项的多个实例是累加的。有关更多信息，请参见 mysqlpump 对象选择。

+   `--insert-ignore`

    | 命令行格式 | `--insert-ignore` |
    | --- | --- |

    写入`INSERT IGNORE`语句而不是`INSERT`语句。

+   `--log-error-file=*`file_name`*`

    | 命令行格式 | `--log-error-file=file_name` |
    | --- | --- |
    | 类型 | 文件名 |

    通过将警告和错误附加到指定的文件名来记录警告和错误。如果未提供此选项，**mysqlpump**将警告和错误写入标准错误输出。

+   `--login-path=*`name`*`

    | 命令行格式 | `--login-path=name` |
    | --- | --- |
    | 类型 | 字符串 |

    从`.mylogin.cnf`登录路径文件中的命名登录路径中读取选项。 “登录路径”是一个包含指定要连接到哪个 MySQL 服务器以及要作为哪个帐户进行身份验证的选项的选项组。要创建或修改登录路径文件，请使用**mysql_config_editor**实用程序。请参见���6.6.7 节，“mysql_config_editor — MySQL 配置实用程序”。 

    有关此选项和其他选项文件选项的更多信息，请参见第 6.2.2.3 节，“影响选项文件处理的命令行选项”。

+   `--max-allowed-packet=*`N`*`

    | 命令行格式 | `--max-allowed-packet=N` |
    | --- | --- |
    | 类型 | 数值 |
    | 默认值 | `25165824` |

    客户端/服务器通信缓冲区的最大大小。默认值为 24MB，最大值为 1GB。

+   `--net-buffer-length=*`N`*`

    | 命令行格式 | `--net-buffer-length=N` |
    | --- | --- |
    | 类型 | 数值 |
    | 默认值 | `1047552` |

    客户端/服务器通信缓冲区的初始大小。在创建多行`INSERT`语句（如使用`--extended-insert`选项时），**mysqlpump**创建长度最长达到*`N`*字节的行。如果您使用此选项来增加值，请确保 MySQL 服务器的`net_buffer_length`系统变量至少具有这么大的值。

+   `--no-create-db`

    | 命令行格式 | `--no-create-db` |
    | --- | --- |

    抑制可能包含在输出中的任何`CREATE DATABASE`语句。

+   `--no-create-info`，`-t`

    | 命令行格式 | `--no-create-info` |
    | --- | --- |

    不要编写创建每个转储表的`CREATE TABLE`语句。

+   `--no-defaults`

    | 命令行格式 | `--no-defaults` |
    | --- | --- |

    不要读取任何选项文件。如果由于从选项文件中读取未知选项而导致程序启动失败，可以使用`--no-defaults`来防止读取它们。

    例外情况是，如果存在`.mylogin.cnf`文件，则在所有情况下都会读取该文件。即使使用`--no-defaults`，这使得可以以比在命令行上更安全的方式指定密码。要创建`.mylogin.cnf`，请使用**mysql_config_editor**实用程序。请参见第 6.6.7 节，“mysql_config_editor — MySQL 配置实用程序”。

    有关此选项和其他选项文件选项的更多信息，请参见第 6.2.2.3 节，“影响选项文件处理的命令行选项”。

+   [`--parallel-schemas=[*`N`*:]*`db_list`*`](mysqlpump.html#option_mysqlpump_parallel-schemas)

    | 命令行格式 | `--parallel-schemas=[N:]schema_list` |
    | --- | --- |
    | 类型 | 字符串 |

    创建一个用于处理 *`db_list`* 中的数据库的队列，其中 *`db_list`* 是一个或多个逗号分隔的数据库名称列表。如果给定 *`N`*，则队列使用 *`N`* 个线程。如果未给定 *`N`*，则 `--default-parallelism` 选项确定队列线程的数量。

    此选项的多个实例会创建多个队列。**mysqlpump** 还会创建一个默认队列，用于未在任何 `--parallel-schemas` 选项中命名的数据库，以及在命令选项选择它们时用于转储用户定义。有关更多信息，请参见 mysqlpump 并行处理。

+   [`--password[=*`password`*]`](mysqlpump.html#option_mysqlpump_password), `-p[*`password`*]`

    | 命令行格式 | `--password[=password]` |
    | --- | --- |
    | 类型 | 字符串 |

    用于连接到服务器的 MySQL 帐户的密码。密码值是可选的。如果未提供，**mysqlpump** 会提示输入一个。如果提供了密码，则 `--password=` 或 `-p` 和后面的密码之间 *不能有空格*。如果未指定密码选项，则默认情况下不发送密码。

    在命令行上指定密码应被视为不安全。为了避免在命令行上提供密码，请使用选项文件。请参阅 第 8.1.2.1 节，“终端用户密码安全指南”。

    要明确指定没有密码，并且 **mysqlpump** 不应提示输入密码，请使用 `--skip-password` 选项。

+   [`--password1[=*`pass_val`*]`](mysqlpump.html#option_mysqlpump_password1)

    用于连接到服务器的 MySQL 帐户的多因素身份验证因子 1 的密码。密码值是可选的。如果未提供，**mysqlpump** 会提示输入一个。如果提供了密码，则 `--password1=` 和后面的密码之间 *不能有空格*。如果未指定密码选项，则默认情况下不发送密码。

    在命令行上指定密码应被视为不安全。为了避免在命令行上提供密码，请使用选项文件。请参阅 第 8.1.2.1 节，“终端用户密码安全指南”。

    明确指定没有密码，并且**mysqlpump**不���提示输入密码，请使用`--skip-password1`选项。

    `--password1`和`--password`是同义词，`--skip-password1`和`--skip-password`也是。

+   [`--password2[=*`pass_val`*]`](mysqlpump.html#option_mysqlpump_password2)

    用于连接到服务器的 MySQL 帐户的多因素认证因子 2 的密码。此选项的语义类似于`--password1`的语义；有关详细信息，请参见该选项的描述。

+   [`--password3[=*`pass_val`*]`](mysqlpump.html#option_mysqlpump_password3)

    用于连接到服务器的 MySQL 帐户的多因素认证因子 3 的密码。此选项的语义类似于`--password1`的语义；有关详细信息，请参见该选项的描述。

+   `--plugin-dir=*`dir_name`*`

    | 命令行格式 | `--plugin-dir=dir_name` |
    | --- | --- |
    | 类型 | 目录名称 |

    查找插件的目录。如果使用`--default-auth`选项指定身份验证插件但**mysqlpump**找不到它，请指定此选项。参见第 8.2.17 节，“可插拔认证”。

+   `--port=*`port_num`*`, `-P *`port_num`*`

    | 命令行格式 | `--port=port_num` |
    | --- | --- |
    | 类型 | 数字 |
    | 默认值 | `3306` |

    用于 TCP/IP 连接的端口号。

+   `--print-defaults`

    | 命令行格式 | `--print-defaults` |
    | --- | --- |

    打印程序名称以及从选项文件获取的所有选项。

    有关此选项和其他选项文件选项的其他信息，请参见第 6.2.2.3 节，“影响选项文件处理的命令行选项”。

+   `--protocol={TCP|SOCKET|PIPE|MEMORY}`

    | 命令行格式 | `--protocol=type` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `[见文本]` |
    | 有效值 | `TCP``SOCKET``PIPE``MEMORY` |

    用于连接到服务器的传输协议。当其他连接参数通常导致使用不希望使用的协议时，这很有用。有关允许值的详细信息，请参见第 6.2.7 节，“连接传输协议”。

+   `--replace`

    | 命令行格式 | `--replace` |
    | --- | --- |

    使用`REPLACE`语句而不是`INSERT`语句。

+   `--result-file=*`file_name`*`

    | 命令行格式 | `--result-file=file_name` |
    | --- | --- |
    | 类型 | 文件名 |

    将输出直接定向到指定的文件。即使在生成转储时发生错误，结果文件也会被创建并覆盖其先前的内容。

    此选项应在 Windows 上使用，以防止换行符`\n`被转换为`\r\n`回车/换行序列。

+   `--routines`

    | 命令行格式 | `--routines` |
    | --- | --- |
    | 类型 | 布尔值 |
    | 默认值 | `TRUE` |

    在输出中包含已存储的例程（过程和函数）的转储数据库。此选项需要全局`SELECT`权限。

    使用`--routines`生成的输出包含`CREATE PROCEDURE`和`CREATE FUNCTION`语句以创建例程。

    此选项默认启用；使用`--skip-routines`来禁用它。

+   `--server-public-key-path=*`file_name`*`

    | 命令行格式 | `--server-public-key-path=file_name` |
    | --- | --- |
    | 类型 | 文件名 |

    PEM 格式文件的路径名，其中包含服务器所需的客户端端公钥的副本，用于 RSA 密钥对密码交换。此选项适用于使用`sha256_password`或`caching_sha2_password`认证插件进行身份验证的客户端。对于不使用这些插件进行身份验证的帐户，此选项将被忽略。如果不使用基于 RSA 的密码交换，例如客户端使用安全连接连接到服务器时，也将被忽略。

    如果给定`--server-public-key-path=*`file_name`*`并指定有效的公钥文件，则优先于`--get-server-public-key`。

    对于`sha256_password`，此选项仅在 MySQL 使用 OpenSSL 构建时适用。

    有关`sha256_password`和`caching_sha2_password`插件的信息，请参见第 8.4.1.3 节，“SHA-256 可插拔认证”和第 8.4.1.2 节，“缓存 SHA-2 可插拔认证”。

+   `--set-charset`

    | 命令行格式 | `--set-charset` |
    | --- | --- |

    将`SET NAMES *`default_character_set`*`写入输出。

    此选项默认启用。要禁用它并抑制`SET NAMES`语句，请使用`--skip-set-charset`。

+   `--set-gtid-purged=*`value`*`

    | 命令行格式 | `--set-gtid-purged=value` |
    | --- | --- |
    | 类型 | 枚举 |
    | 默认值 | `AUTO` |
    | 有效值 | `OFF``ON``AUTO` |

    此选项通过指示是否向输出添加`SET @@GLOBAL.gtid_purged`语句，使得可以控制写入转储文件的全局事务 ID（GTID）信息。此选项还可能导致在重新加载转储文件时向输出写入一个禁用二进制日志的语句。

    下表显示了允许的选项值。默认值为`AUTO`。

    | 值 | 含义 |
    | --- | --- |
    | `OFF` | 不向输出添加任何`SET`语句。  |
    | `ON` | 向输出添加一个`SET`语句。如果服务器未启用 GTIDs，则会发生错误。 |
    | `AUTO` | 如果服务器启用了 GTIDs，则向输出添加一个`SET`语句。 |

    当重新加载转储文件时，`--set-gtid-purged`选项对二进制日志的影响如下：

    +   `--set-gtid-purged=OFF`：不向输出添加`SET @@SESSION.SQL_LOG_BIN=0;`。

    +   `--set-gtid-purged=ON`：向输出添加`SET @@SESSION.SQL_LOG_BIN=0;`。

    +   `--set-gtid-purged=AUTO`：如果您正在备份的服务器启用了 GTIDs（即`AUTO`评估为`ON`），则向输出添加`SET @@SESSION.SQL_LOG_BIN=0;`。

+   `--single-transaction`

    | 命令行格式 | `--single-transaction` |
    | --- | --- |

    此选项将事务隔离模式设置为`REPEATABLE READ`，并在转储数据之前向服务器发送一个`START TRANSACTION` SQL 语句。仅对诸如`InnoDB`之类的事务表有用，因为它会在发出`START TRANSACTION`时转储数据库的一致状态，而不会阻塞任何应用程序。

    在使用此选项时，应该记住只有`InnoDB`表会以一致的状态进行转储。例如，使用此选项时转储的任何`MyISAM`或`MEMORY`表可能仍会改变状态。

    在进行`--single-transaction`转储时，为确保有效的转储文件（正确的表内容和二进制日志坐标），不应有其他连接使用以下语句：`ALTER TABLE`、`CREATE TABLE`、`DROP TABLE`、`RENAME TABLE`、`TRUNCATE TABLE`。一致性读取与这些语句不隔离，因此在要转储的表上使用它们可能导致由**mysqlpump**执行的用于检索表内容的`SELECT`获取不正确的内容或失败。

    `--add-locks`和`--single-transaction`是互斥的。

+   `--skip-definer`

    | 命令行格式 | `--skip-definer` |
    | --- | --- |
    | 类型 | 布尔值 |
    | 默认值 | `FALSE` |

    从视图和存储程序的`CREATE`语句中省略`DEFINER`和`SQL SECURITY`子句。重新加载转储文件时，创建使用默认`DEFINER`和`SQL SECURITY`值的对象。参见第 27.6 节，“存储对象访问控制”。

+   `--skip-dump-rows`, `-d`

    | 命令行格式 | `--skip-dump-rows` |
    | --- | --- |
    | 类型 | 布尔值 |
    | 默认值 | `FALSE` |

    不要转储表行。

+   `--skip-generated-invisible-primary-key`

    | 命令行格式 | `--skip-generated-invisible-primary-key` |
    | --- | --- |
    | 引入版本 | 8.0.30 |
    | 类型 | 布尔值 |
    | 默认值 | `FALSE` |

    此选项从 MySQL 8.0.30 开始可用，导致生成的不可见主键（GIPKs）在转储中被排除。有关 GIPK 和 GIPK 模式的更多信息，请参见第 15.1.20.11 节，“生成的不可见主键”。

+   `--socket=*`path`*`, `-S *`path`*`

    | 命令行格式 | `--socket={file_name&#124;pipe_name}` |
    | --- | --- |
    | 类型 | 字符串 |

    对于连接到`localhost`的情况，要使用的 Unix 套接字文件，或者在 Windows 上要使用的命名管道的名称。

    在 Windows 上，此选项仅在服务器启用了`named_pipe`系统变量以支持命名管道连接时才适用。此外，进行连接的用户必须是由`named_pipe_full_access_group`系统变量指定的 Windows 组的成员。

+   `--ssl*`

    以 `--ssl` 开头的选项指定是否使用加密连接连接到服务器，并指示 SSL 密钥和证书的位置。参见 加密连接的命令选项。

+   `--ssl-fips-mode={OFF|ON|STRICT}`

    | 命令行格式 | `--ssl-fips-mode={OFF&#124;ON&#124;STRICT}` |
    | --- | --- |
    | 弃用 | 8.0.34 |
    | 类型 | 枚举 |
    | 默认值 | `OFF` |
    | 有效值 | `OFF``ON``STRICT` |

    控制是否在客户端端启用 FIPS 模式。`--ssl-fips-mode` 选项与其他 `--ssl-*`xxx`*` 选项不同，它不用于建立加密连接，而是影响允许哪些加密操作。参见 第 8.8 节，“FIPS 支持”。

    允许使用以下 `--ssl-fips-mode` 值：

    +   `OFF`: 禁用 FIPS 模式。

    +   `ON`: 启用 FIPS 模式。

    +   `STRICT`: 启用“严格” FIPS 模式。

    注意

    如果 OpenSSL FIPS 对象模块不可用，则 `--ssl-fips-mode` 的唯一允许值为 `OFF`。在这种情况下，将 `--ssl-fips-mode` 设置为 `ON` 或 `STRICT` 会导致客户端在启动时产生警告并在非 FIPS 模式下运行。

    从 MySQL 8.0.34 开始，此选项已弃用。预计在将来的 MySQL 版本中将其移除。

+   `--tls-ciphersuites=*`ciphersuite_list`*`

    | 命令行格式 | `--tls-ciphersuites=ciphersuite_list` |
    | --- | --- |
    | 引入版本 | 8.0.16 |
    | 类型 | 字符串 |

    使用 TLSv1.3 的加密连接的允许密码套件。该值是一个或多个以冒号分隔的密码套件名称列表。可以为此选项命名的密码套件取决于用于编译 MySQL 的 SSL 库。有关详细信息，请参见 第 8.3.2 节，“加密连接 TLS 协议和密码套件”。

    此选项在 MySQL 8.0.16 中添加。

+   `--tls-version=*`protocol_list`*`

    | 命令行格式 | `--tls-version=protocol_list` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值（≥ 8.0.16） | `TLSv1,TLSv1.1,TLSv1.2,TLSv1.3`（OpenSSL 1.1.1 或更高版本）`TLSv1,TLSv1.1,TLSv1.2`（否则） |
    | 默认值（≤ 8.0.15） | `TLSv1,TLSv1.1,TLSv1.2` |

    用于加密连接的允许的 TLS 协议。该值是一个或多个以逗号分隔的协议名称列表。可以为此选项命名的协议取决于用于编译 MySQL 的 SSL 库。有关详细信息，请��见 第 8.3.2 节，“加密连接 TLS 协议和密码套件”。

+   `--triggers`

    | 命令行格式 | `--triggers` |
    | --- | --- |
    | 类型 | 布尔 |
    | 默认值 | `TRUE` |

    在输出中为每个转储表包括触发器。

    默认情况下启用此选项；使用`--skip-triggers`来禁用它。

+   `--tz-utc`

    | 命令行格式 | `--tz-utc` |
    | --- | --- |

    此选项允许在不同时区的服务器之间转储和重新加载`TIMESTAMP`列。**mysqlpump**将其连接时区设置为 UTC，并在转储文件中添加`SET TIME_ZONE='+00:00'`。如果没有此选项，`TIMESTAMP`列将在源服务器和目标服务器的本地时区中转储和重新加载，如果服务器位于不同的时区，可能会导致值发生变化。`--tz-utc`也可以防止由于夏令时变化而导致的变化。

    默认情况下启用此选项；使用`--skip-tz-utc`来禁用它。

+   `--user=*`user_name`*`, `-u *`user_name`*`

    | 命令行格式 | `--user=user_name` |
    | --- | --- |
    | 类型 | 字符串 |

    用于连接到服务器的 MySQL 帐户的用户名。

    如果您在 MySQL 8.0.31 或更高版本中使用`Rewriter`插件，则应授予此用户`SKIP_QUERY_REWRITE`权限。

+   `--users`

    | 命令行格式 | `--users` |
    | --- | --- |
    | 类型 | 布尔 |
    | 默认值 | `FALSE` |

    将用户帐户转储为`CREATE USER`和`GRANT`语句的逻辑定义。

    用户定义存储在`mysql`系统数据库的授权表中。默认情况下，**mysqlpump**不包括`mysql`数据库转储中的授权表。要将授权表的内容作为逻辑定义转储，请使用`--users`选项并禁止所有数据库转储：

    ```sql
    mysqlpump --exclude-databases=% --users
    ```

+   `--version`, `-V`

    | 命令行格式 | `--version` |
    | --- | --- |

    显示版本信息并退出。

+   `--watch-progress`

    | 命令行格式 | `--watch-progress` |
    | --- | --- |
    | 类型 | 布尔 |
    | 默认值 | `TRUE` |

    定期显示有关已完成和总表、行和其他对象数量的进度指示器。

    默认情况下启用此选项；使用`--skip-watch-progress`来禁用它。

+   `--zstd-compression-level=*`level`*`

    | 命令行格式 | `--zstd-compression-level=#` |
    | --- | --- |
    | 引入版本 | 8.0.18 |
    | 类型 | 整数 |

    用于使用 `zstd` 压缩算法连接到服务器的压缩级别。允许的级别从 1 到 22，较大的值表示较高级别的压缩。默认的 `zstd` 压缩级别为 3。压缩级别设置对不使用 `zstd` 压缩的连接没有影响。

    更多信息，请参见 6.2.8 节，“连接压缩控制”。 

    此选项在 MySQL 8.0.18 版本中添加。

#### mysqlpump 对象选择

**mysqlpump** 具有一组包含和排除选项，可过滤多种对象类型，并提供对要转储的对象进行灵活控制：

+   `--include-databases` 和 `--exclude-databases` 适用于数据库及其内部的所有对象。

+   `--include-tables` 和 `--exclude-tables` 适用于表。这些选项还会影响与表相关的触发器，除非给出了特定于触发器的选项。

+   `--include-triggers` 和 `--exclude-triggers` 适用于触发器。

+   `--include-routines` 和 `--exclude-routines` 适用于存储过程和函数。如果一个例程选项匹配存储过程名称，它也会匹配同名的存储函数。

+   `--include-events` 和 `--exclude-events` 适用于事件调度器事件。

+   `--include-users` 和 `--exclude-users` 适用于用户账户。

任何包含或排除选项都可以多次给出。效果是累加的。这些选项的顺序不重要。

每个包含和排除选项的值是逗号分隔的适当对象类型的名称列表。例如：

```sql
--exclude-databases=test,world
--include-tables=customer,invoice
```

通配符字符允许在对象名称中使用：

+   `%` 匹配零个或多个字符的任意序列。

+   `_` 匹配任意单个字符。

例如，`--include-tables=t%,__tmp` 匹配所有以 `t` 开头的表名和以 `tmp` 结尾的五个字符表名。

对于用户，没有主机部分指定的名称会被解释为隐含主机`%`。例如，`u1`和`u1@%`是等效的。这与 MySQL 通常适用的等效性相同（参见第 8.2.4 节，“指定帐户名称”）。

包含和排除选项的交互如下：

+   默认情况下，没有包含或排除选项，**mysqlpump**会转储所有数据库（特定例外情况在 mysqlpump 限制中有说明）。

+   如果在没有排除选项的情况下给出了包含选项，则只有被包含命名的对象会被转储。

+   如果在没有包含选项的情况下给出了排除选项，则所有对象都会被转储，除了被排除命名的对象。

+   如果给出了包含和排除选项，则所有被排除的对象和未被包含的对象都不会被转储。所有其他对象都会被转储。

如果正在转储多个数据库，则可以通过使用数据库名称限定对象名称来命名特定数据库中的表、触发器和例程。以下命令转储数据库`db1`和`db2`，但排除表`db1.t1`和`db2.t2`：

```sql
mysqlpump --include-databases=db1,db2 --exclude-tables=db1.t1,db2.t2
```

以下选项提供了指定要转储的数据库的替代方法：

+   `--all-databases`选项会转储所有数据库（特定例外情况在 mysqlpump 限制中有说明）。这等同于根本不指定任何对象选项（默认**mysqlpump**的操作是转储所有内容）。

    `--include-databases=%`类似于`--all-databases`，但选择所有数据库进行转储，即使它们是`--all-databases`的例外情况。

+   `--databases`选项会导致**mysqlpump**将所有名称参数视为要转储的数据库名称。这等同于一个`--include-databases`选项，命名相同的数据库。

#### mysqlpump 并行处理

**mysqlpump**可以使用并行处理来实现并发处理。您可以选择数据库之间的并发性（同时转储多个数据库）和数据库内的并发性（同时转储给定数据库中的多个对象）。

默认情况下，**mysqlpump**设置一个队列和两个线程。您可以创建额外的队列并控制分配给每个队列的线程数，包括默认队列：

+   `--default-parallelism=*`N`*`指定每个队列使用的默认线程数。如果没有此选项，*`N`*为 2。

    默认队列始终使用默认线程数。除非另有说明，否则其他队列使用默认线程数。

+   [`--parallel-schemas=[*`N`*:]*`db_list`*`](mysqlpump.html#option_mysqlpump_parallel-schemas)设置一个处理队列，用于转储*`db_list`*中命名的数据库，并可选择指定队列使用的线程数。*`db_list`*是逗号分隔的数据库名称列表。如果选项参数以`*`N`*:`开头，则队列使用*`N`*个线程。否则，`--default-parallelism`选项确定队列线程数。

    多个`--parallel-schemas`选项实例会创建多个队列。

    数据库列表中的名称允许包含与过滤选项支持的`%`和`_`通配符相同的字符（请参阅 mysqlpump 对象选择）。

**mysqlpump**使用默认队列来处理未明确命名的任何数据库，并在命令选项选择它们时用于转储用户定义。

通常，对于多个队列，**mysqlpump**在队列处理的数据库集之间使用并行性，以同时转储多个数据库。对于使用多个线程的队列，**mysqlpump**在数据库内部使用并行性，以同时转储给定数据库中的多个对象。也可能会发生异常；例如，**mysqlpump**可能会在从服务器获取数据库对象列表时阻塞队列。

启用并行性后，不同数据库的输出可能会交错。例如，并行转储的多个表中的`INSERT`语句可能会交错；这些语句不会按任何特定顺序编写。这不会影响重新加载，因为输出语句会使用数据库名称限定对象名称，或根据需要在之前加上`USE`语句。

并行性的粒度是单个数据库对象。例如，单个表不能使用多个线程并行转储。

示例：

```sql
mysqlpump --parallel-schemas=db1,db2 --parallel-schemas=db3
```

**mysqlpump**设置一个队列来处理`db1`和`db2`，另一个队列来处理`db3`，以及一个默认队列来处理所有其他数据库。所有队列都使用两个线程。

```sql
mysqlpump --parallel-schemas=db1,db2 --parallel-schemas=db3
          --default-parallelism=4
```

这与前面的示例相同，只是所有队列都使用四个线程。

```sql
mysqlpump --parallel-schemas=5:db1,db2 --parallel-schemas=3:db3
```

`db1` 和 `db2` 的队列使用五个线程，`db3` 的队列使用三个线程，而默认队列使用默认的两个线程。

作为一个特例，使用 `--default-parallelism=0` 和没有 `--parallel-schemas` 选项时，**mysqlpump** 作为单线程进程运行，并且不创建任何队列。

#### mysqlpump 限制

**mysqlpump** 默认不会转储 `performance_schema`、`ndbinfo` 或 `sys` 模式。要转储其中任何一个，请在命令行上明确命名它们。您还可以使用 `--databases` 或 `--include-databases` 选项来命名它们。

**mysqlpump** 不会转储 `INFORMATION_SCHEMA` 模式。

**mysqlpump** 不会转储 `InnoDB` `CREATE TABLESPACE` 语句。

**mysqlpump** 使用 `CREATE USER` 和 `GRANT` 语句以逻辑形式转储用户帐户（例如，当您使用 `--include-users` 或 `--users` 选项时）。因此，默认情况下，对 `mysql` 系统数据库的转储不包括包含用户定义的授权表：`user`、`db`、`tables_priv`、`columns_priv`、`procs_priv` 或 `proxies_priv`。要转储其中任何授权表，请在 `mysql` 数据库后面命名表名：

```sql
mysqlpump mysql user db ...
```
