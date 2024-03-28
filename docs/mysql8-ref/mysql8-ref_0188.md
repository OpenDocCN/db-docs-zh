# 6.5.4 mysqldump — 一个数据库备份程序

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysqldump.html`](https://dev.mysql.com/doc/refman/8.0/en/mysqldump.html)

**mysqldump**客户端实用程序执行逻辑备份，生成一组可以执行的 SQL 语句，以重现原始数据库对象定义和表数据。它为备份或传输到另一个 SQL 服务器转储一个或多个 MySQL 数据库。**mysqldump**命令还可以生成 CSV、其他分隔文本或 XML 格式的输出。

提示

考虑使用 MySQL Shell dump 工具，提供多线程并行转储、文件压缩、进度信息显示，以及云功能，如 Oracle Cloud 基础设施对象存储流式传输，以及 MySQL HeatWave 服务兼容性检查和修改。转储可以轻松导入到 MySQL Server 实例或 MySQL HeatWave 服务 DB 系统中，使用 MySQL Shell load dump 工具。MySQL Shell 的安装说明可以在这里找到。

+   性能和可伸缩性考虑

+   调用语法

+   选项语法 - 按字母顺序总结

+   连接选项

+   选项文件选项

+   DDL 选项

+   调试选项

+   帮助选项

+   国际化选项

+   复制选项

+   格式选项

+   过滤选项

+   性能选项

+   事务选项

+   选项组

+   示例

+   限制

**mysqldump**需要至少具有`SELECT`权限来备份表格，`SHOW VIEW`用于备份视图，`TRIGGER`用于备份触发器，如果未使用`--single-transaction`选项，则需要`LOCK TABLES`，如果未使用`--no-tablespaces`选项（自 MySQL 8.0.21 起），还需要`PROCESS`，如果同时使用了`gtid_mode=ON`和`gtid_purged=ON|AUTO`，则还需要（自 MySQL 8.0.32 起）具有`RELOAD`或`FLUSH_TABLES`权限和`--single-transaction`。某些选项可能需要其他权限，如选项描述中所述。

要重新加载转储文件，您必须具有执行其中包含的语句所需的权限，例如由这些语句创建的对象所需的适当`CREATE`权限。

**mysqldump**输出可能包括`ALTER DATABASE`语句来更改数据库排序规则。在转储存储程序以保留其字符编码时可能会使用这些语句。要重新加载包含此类语句的转储文件，需要受影响数据库的`ALTER`权限。

注意

在 Windows 上使用 PowerShell 进行输出重定向创建的转储文件具有 UTF-16 编码：

```sql
mysqldump [options] > dump.sql
```

然而，不允许使用 UTF-16 作为连接字符集（参见不允许的客户端字符集），因此无法正确加载转储文件。为解决此问题，请使用`--result-file`选项，该选项以 ASCII 格式创建输出：

```sql
mysqldump [options] --result-file=dump.sql
```

不建议在服务器启用 GTIDs（`gtid_mode=ON`）时加载转储文件，如果您的转储文件包含系统表。**mysqldump**为使用非事务性 MyISAM 存储引擎的系统表发出 DML 指令，而在启用 GTIDs 时不允许此组合。

#### 性能和可伸缩性考虑

`mysqldump`的优点包括在恢复之前查看或甚至编辑输出的便利性和灵活性。您可以为开发和 DBA 工作克隆数据库，或为测试生成现有数据库的轻微变体。它并不旨在用作备份大量数据的快速或可扩展解决方案。对于大数据量，即使备份步骤花费合理的时间，恢复数据可能会非常缓慢，因为重放 SQL 语句涉及磁盘 I/O 进行插入、索引创建等操作。

对于大规模备份和恢复，物理备份更为合适，可以复制数据文件的原始格式，以便快速恢复。

如果您的表主要是`InnoDB`表，或者如果您有`InnoDB`和`MyISAM`表的混合，请考虑使用**mysqlbackup**，它作为 MySQL Enterprise 的一部分提供。该工具为`InnoDB`备份提供高性能，几乎没有中断；它还可以备份`MyISAM`和其他存储引擎的表；它还提供许多方便的选项以适应不同的备份场景。请参见 Section 32.1, “MySQL Enterprise Backup Overview”。

**mysqldump**可以逐行检索和转储表内容，或者可以从表中检索整个内容并在转储之前在内存中缓冲它。如果要逐行转储表，请使用`--quick`选项（或`--opt`，它启用`--quick`）。`--opt`选项（因此`--quick`）默认启用，因此要启用内存缓冲，请使用`--skip-quick`。

如果您使用最新版本的**mysqldump**生成要重新加载到非常旧的 MySQL 服务器中的转储，请使用`--skip-opt`选项，而不是`--opt`或`--extended-insert`选项。

有关**mysqldump**的更多信息，请参见 Section 9.4, “Using mysqldump for Backups”。

#### 调用语法

通常有三种使用**mysqldump**的方式——用于转储一个或多个表、一个或多个完整数据库的集合，或整个 MySQL 服务器——如下所示：

```sql
mysqldump [*options*] *db_name* [*tbl_name* ...]
mysqldump [*options*] --databases *db_name* ...
mysqldump [*options*] --all-databases
```

要转储整个数据库，请不要在*`db_name`*后面命名任何表，或使用`--databases`或`--all-databases`选项。

要查看您的版本支持的**mysqldump**选项列表，请发出命令**mysqldump** `--help`。

#### 选项语法 - 按字母顺序总结

**mysqldump**支持以下选项，可以在命令行或选项文件的`[mysqldump]`和`[client]`组中指定。有关 MySQL 程序使用的选项���件的信息，请参见 Section 6.2.2.2, “Using Option Files”。

**表 6.15 mysqldump 选项**

| 选项名称 | 描述 | 引入 | 废弃 |
| --- | --- | --- | --- |
| --add-drop-database | 在每个 CREATE DATABASE 语句之前添加 DROP DATABASE 语句 |  |  |
| --add-drop-table | 在每个 CREATE TABLE 语句之前添加 DROP TABLE 语句 |  |  |
| --add-drop-trigger | 在每个 CREATE TRIGGER 语句之前添加 DROP TRIGGER 语句 |  |  |
| --add-locks | 将每个表转储包围在 LOCK TABLES 和 UNLOCK TABLES 语句中 |  |  |
| --all-databases | 转储所有数据库中的所有表 |  |  |
| --allow-keywords | 允许创建关键字列名 |  |  |
| --apply-replica-statements | 在输出的 CHANGE REPLICATION SOURCE TO 语句之前包括 STOP REPLICA，并在结束时包括 START REPLICA | 8.0.26 |  |
| --apply-slave-statements | 在输出的 CHANGE MASTER 语句之前包括 STOP SLAVE，并在结束时包括 START SLAVE |  | 8.0.26 |
| --bind-address | 使用指定的网络接口连接到 MySQL 服务器 |  |  |
| --character-sets-dir | 安装字符集的目录 |  |  |
| --column-statistics | 写入 ANALYZE TABLE 语句以生成统计直方图 |  |  |
| --comments | 向转储文件添加注释 |  |  |
| --compact | 生成更紧凑的输出 |  |  |
| --compatible | 生成与其他数据库系统或较旧的 MySQL 服务器更兼容的输出 |  |  |
| --complete-insert | 使用包含列名的完整 INSERT 语句 |  |  |
| --compress | 在客户端和服务器之间发送的所有信息进行压缩 |  | 8.0.18 |
| --compression-algorithms | 用于与服务器连接的允许的压缩算法 | 8.0.18 |  |
| --create-options | 在 CREATE TABLE 语句中包括所有特定于 MySQL 的表选项 |  |  |
| --databases | 将所有名称参数解释为数据库名称 |  |  |
| --debug | 写调试日志 |  |  |
| --debug-check | 程序退出时打印调试信息 |  |  |
| --debug-info | 程序退出时打印调试信息、内存和 CPU 统计信息 |  |  |
| --default-auth | 要使用的身份验证插件 |  |  |
| --default-character-set | 指定默认字符集 |  |  |
| --defaults-extra-file | 除了通常的选项文件外，还读取命名的选项文件 |  |  |
| --defaults-file | 仅读取命名的选项文件 |  |  |
| --defaults-group-suffix | 选项组后缀值 |  |  |
| --delete-master-logs | 在复制源服务器上，在执行转储操作后删除二进制日志 |  | 8.0.26 |
| --delete-source-logs | 在复制源服务器上，在执行转储操作后删除二进制日志 | 8.0.26 |  |
| --disable-keys | 对于每个表，用于禁用和启用键的语句包围 INSERT 语句 |  |  |
| --dump-date | 如果给定了--comments，则包括转储日期作为“Dump completed on”注释 |  |  |
| --dump-replica | 包括 CHANGE REPLICATION SOURCE TO 语句，列出副本源的二进制日志坐标 | 8.0.26 |  |
| --dump-slave | 包括列出副本源的二进制日志坐标的 CHANGE MASTER 语句 |  | 8.0.26 |
| --enable-cleartext-plugin | 启用明文认证插件 |  |  |
| --events | 转储来自已转储数据库的事件 |  |  |
| --extended-insert | 使用多行 INSERT 语法 |  |  |
| --fields-enclosed-by | 此选项与 --tab 选项一起使用，具有与 LOAD DATA 相应子句相同的含义 |  |  |
| --fields-escaped-by | 此选项与 --tab 选项一起使用，具有与 LOAD DATA 相应子句相同的含义 |  |  |
| --fields-optionally-enclosed-by | 此选项与 --tab 选项一起使用，具有与 LOAD DATA 相应子句相同的含义 |  |  |
| --fields-terminated-by | 此选项与 --tab 选项一起使用，具有与 LOAD DATA 相应子句相同的含义 |  |  |
| --flush-logs | 在开始转储之前刷新 MySQL 服务器日志文件 |  |  |
| --flush-privileges | 在转储 mysql 数据库后发出 FLUSH PRIVILEGES 语句 |  |  |
| --force | 即使在转储表时发生 SQL 错误也继续 |  |  |
| --get-server-public-key | 从服务器请求 RSA 公钥 |  |  |
| --help | 显示帮助消息并退出 |  |  |
| --hex-blob | 使用十六进制表示法转储二进制列 |  |  |
| --host | MySQL 服务器所在的主机 |  |  |
| --ignore-error | 忽略指定的错误 |  |  |
| --ignore-table | 不转储给定表 |  |  |
| --include-master-host-port | 在使用 --dump-slave 生成的 CHANGE MASTER 语句中包含 MASTER_HOST/MASTER_PORT 选项 |  | 8.0.26 |
| --include-source-host-port | 在使用 --dump-replica 生成的 CHANGE REPLICATION SOURCE TO 语句中包含 SOURCE_HOST 和 SOURCE_PORT 选项 | 8.0.26 |  |
| --insert-ignore | 写入 INSERT IGNORE 而不是 INSERT 语句 |  |  |
| --lines-terminated-by | 此选项与 --tab 选项一起使用，具有与 LOAD DATA 相应子句相同的含义 |  |  |
| --lock-all-tables | 锁定所有数据库中的所有表 |  |  |
| --lock-tables | 转储表之前锁定所有表 |  |  |
| --log-error | 将警告和错误追加到指定文件 |  |  |
| --login-path | 从.mylogin.cnf 中读取登录路径选项 |  |  |
| --master-data | 将二进制日志文件名和位置写入输出 |  | 8.0.26 |
| --max-allowed-packet | 发送到服务器或从服务器接收的最大数据包长度 |  |  |
| --mysqld-long-query-time | 慢查询阈值的会话值 | 8.0.30 |  |
| --net-buffer-length | TCP/IP 和套接字通信的缓冲区大小 |  |  |
| --network-timeout | 增加网络超时以允许更大的表转储 |  |  |
| --no-autocommit | 将每个转储表的 INSERT 语句括在 SET autocommit = 0 和 COMMIT 语句中 |  |  |
| --no-create-db | 不写入 CREATE DATABASE 语句 |  |  |
| --no-create-info | 不写入重新创建每个转储表的 CREATE TABLE 语句 |  |  |
| --no-data | 不转储表内容 |  |  |
| --no-defaults | 不读取任何选项文件 |  |  |
| --no-set-names | 与--skip-set-charset 相同 |  |  |
| --no-tablespaces | 在输出中不写入任何 CREATE LOGFILE GROUP 或 CREATE TABLESPACE 语句 |  |  |
| --opt | --add-drop-table --add-locks --create-options --disable-keys --extended-insert --lock-tables --quick --set-charset 的简写 |  |  |
| --order-by-primary | 按每个表的主键或第一个唯一索引对其行进行排序 |  |  |
| --password | 连接到服务器时使用的密码 |  |  |
| --password1 | 连接到服务器时使用的第一个多因素身份验证密码 | 8.0.27 |  |
| --password2 | 连接到服务器时使用的第二个多因素身份验证密码 | 8.0.27 |  |
| --password3 | 连接到服务器时使用的第三个多因素身份验证密码 | 8.0.27 |  |
| --pipe | 使用命名管道连接到服务器（仅限 Windows） |  |  |
| --plugin-authentication-kerberos-client-mode | 允许通过 Windows 上的 MIT Kerberos 库进行 GSSAPI 可插拔认证 | 8.0.32 |  |
| --plugin-dir | 插件安装目录 |  |  |
| --port | TCP/IP 连接的端口号 |  |  |
| --print-defaults | 打印默认选项 |  |  |
| --protocol | 要使用的传输协议 |  |  |
| --quick | 逐行从服务器检索表的行 |  |  |
| --quote-names | 在反引号字符内引用标识符 |  |  |
| --replace | 写入 REPLACE 语句而不是 INSERT 语句 |  |  |
| --result-file | 将输出直接定向到给定文件 |  |  |
| --routines | 从转储的数据库中转储存储过程（procedures）和函数（functions） |  |  |
| --server-public-key-path | 包含 RSA 公钥的文件路径名 |  |  |
| --set-charset | 将 SET NAMES default_character_set 添加到输出 |  |  |
| --set-gtid-purged | 是否将 SET @@GLOBAL.GTID_PURGED 添加到输出中 |  |  |
| --shared-memory-base-name | 共享内存连接的共享内存名称（仅限 Windows） |  |  |
| --show-create-skip-secondary-engine | 从 CREATE TABLE 语句中排除 SECONDARY ENGINE 子句 | 8.0.18 |  |
| --single-transaction | 在从服务器转储数据之前发出 BEGIN SQL 语句 |  |  |
| --skip-add-drop-table | 在每个 CREATE TABLE 语句之前不添加 DROP TABLE 语句 |  |  |
| --skip-add-locks | 不添加锁 |  |  |
| --skip-comments | 不向转储文件添加注释 |  |  |
| --skip-compact | 不生成更紧凑的输出 |  |  |
| --skip-disable-keys | 不禁用键 |  |  |
| --skip-extended-insert | 关闭扩展插入 |  |  |
| --skip-generated-invisible-primary-key | 不在转储文件中包含生成的不可见主键 | 8.0.30 |  |
| --skip-opt | 关闭--opt 设置的选项 |  |  |
| --skip-quick | 不从服务器逐行检索表的行 |  |  |
| --skip-quote-names | 不引用标识符 |  |  |
| --skip-set-charset | 不写入 SET NAMES 语句 |  |  |
| --skip-triggers | 不转储触发器 |  |  |
| --skip-tz-utc | 关闭 tz-utc |  |  |
| --socket | Unix 套接字文件或 Windows 命名管道 |  |  |
| --source-data | 将二进制日志文件名和位置写入输出 | 8.0.26 |  |
| --ssl-ca | 包含受信任 SSL 证书颁发机构列表的文件 |  |  |
| --ssl-capath | 包含受信任 SSL 证书颁发机构证书文件的目录 |  |  |
| --ssl-cert | 包含 X.509 证书的文件 |  |  |
| --ssl-cipher | 连接加密的可允许密码 |  |  |
| --ssl-crl | 包含证书吊销列表的文件 |  |  |
| --ssl-crlpath | 包含证书吊销列表文件的目录 |  |  |
| --ssl-fips-mode | 是否在客户端启用 FIPS 模式 |  | 8.0.34 |
| --ssl-key | 包含 X.509 密钥的文件 |  |  |
| --ssl-mode | 与服务器连接的期望安全状态 |  |  |
| --ssl-session-data | 包含 SSL 会话数据的文件 | 8.0.29 |  |
| --ssl-session-data-continue-on-failed-reuse | 如果会话重用失败是否建立连接 | 8.0.29 |  |
| --tab | 生成制表符分隔的数据文件 |  |  |
| --tables | 覆盖--databases 或-B 选项 |  |  |
| --tls-ciphersuites | 用于加密连接的 TLSv1.3 密码套件 | 8.0.16 |  |
| --tls-version | 用于加密连接的 TLS 协议 |  |  |
| --triggers | 为每个转储表转储触发器 |  |  |
| --tz-utc | 在转储文件中添加 SET TIME_ZONE='+00:00' |  |  |
| --user | 连接到服务器时要使用的 MySQL 用户名 |  |  |
| --verbose | 详细模式 |  |  |
| --version | 显示版本信息并退出 |  |  |
| --where | 仅转储由给定 WHERE 条件选择的行 |  |  |
| --xml | 生成 XML 输出 |  |  |
| --zstd-compression-level | 用于使用 zstd 压缩的服务器连接的压缩级别 | 8.0.18 |  |
| 选项名称 | 描述 | 引入 | 已弃用 |

#### 连接选项

**mysqldump** 命令登录到 MySQL 服务器以提取信息。以下选项指定如何连接到 MySQL 服务器，无论是在同一台机器上还是在远程系统上。

+   `--bind-address=*`ip_address`*`

    | 命令行格式 | `--bind-address=ip_address` |
    | --- | --- |

    在具有多个网络接口的计算机上，使用此选项选择要用于连接到 MySQL 服务器的接口。

+   `--compress`, `-C`

    | 命令行格式 | `--compress[={OFF&#124;ON}]` |
    | --- | --- |
    | 已弃用 | 8.0.18 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    尽可能压缩客户端和服务器之间发送的所有信息。请参见 Section 6.2.8, “Connection Compression Control”。

    从 MySQL 8.0.18 开始，此选项已弃用。预计将在将来的 MySQL 版本中删除。请参见 Configuring Legacy Connection Compression。

+   `--compression-algorithms=*`value`*`

    | 命令行格式 | `--compression-algorithms=value` |
    | --- | --- |
    | 引入 | 8.0.18 |
    | 类型 | 设置 |
    | 默认值 | `uncompressed` |
    | 有效值 | `zlib``zstd``uncompressed` |

    连接到服务器的允许的压缩算法。可用的算法与 `protocol_compression_algorithms` 系统变量相同。默认值为 `uncompressed`。

    更多信息，请参见 Section 6.2.8, “Connection Compression Control”。

    此选项已添加到 MySQL 8.0.18 中。

+   `--default-auth=*`plugin`*`

    | 命令行格式 | `--default-auth=plugin` |
    | --- | --- |
    | 类型 | 字符串 |

    有关要使用的客户端端身份验证插件的提示。请参见 Section 8.2.17, “Pluggable Authentication”。

+   `--enable-cleartext-plugin`

    | 命令行格式 | `--enable-cleartext-plugin` |
    | --- | --- |
    | 类型 | 布尔值 |
    | 默认值 | `FALSE` |

    启用`mysql_clear_password`明文认证插件。（参见第 8.4.1.4 节，“客户端明文可插拔认证”.）

+   `--get-server-public-key`

    | 命令行格式 | `--get-server-public-key` |
    | --- | --- |
    | 类型 | 布尔值 |

    从服务器请求用于 RSA 密钥对密码交换所需的公钥。此选项适用于使用`caching_sha2_password`认证插件进行身份验证的客户端。对于该插件，除非请求，否则服务器不会发送公钥。对于不使用该插件进行身份验证的帐户，此选项将被忽略。如果不使用基于 RSA 的密码交换，例如客户端使用安全连接连接到服务器时，此选项也将被忽略。

    如果`--server-public-key-path=*`file_name`*`已给出并指定了有效的公钥文件，则它将优先于`--get-server-public-key`.

    有关`caching_sha2_password`插件的信息，请参阅第 8.4.1.2 节，“缓存 SHA-2 可插拔认证”.

+   `--host=*`host_name`*`, `-h *`host_name`*`

    | 命令行格式 | `--host` |
    | --- | --- |

    从给定主机的 MySQL 服务器中转储数据。默认主机是`localhost`.

+   `--login-path=*`name`*`

    | 命令行格式 | `--login-path=name` |
    | --- | --- |
    | 类型 | 字符串 |

    从`.mylogin.cnf`登录路径文件中的指定登录路径读取选项。 “登录路径”是一个选项组，其中包含指定要连接到的 MySQL 服务器和要进行身份验证的帐户的选项。要创建或修改登录路径文件，请使用**mysql_config_editor**实用程序。请参阅第 6.6.7 节，“mysql_config_editor — MySQL 配置实用程序”.

    有关此选项和其他选项文件选项的更多信息，请参阅第 6.2.2.3 节，“影响选项文件处理的命令行选项”.

+   [`--password[=*`password`*]`](mysqldump.html#option_mysqldump_password), `-p[*`password`*]`

    | 命令行格式 | `--password[=password]` |
    | --- | --- |
    | 类型 | 字符串 |

    用于连接到服务器的 MySQL 帐户的密码。密码值是可选的。如果未提供，**mysqldump** 会提示输入密码。如果提供了密码，则`--password=`或`-p`后面必须*没有空格*。如果未指定密码选项，则默认不发送密码。

    在命令行上指定密码应被视为不安全。为了避免在命令行上提供密码，请使用选项文件。请参阅第 8.1.2.1 节，“密码安全的最终用户指南”。

    明确指定没有密码且**mysqldump**不应提示输入密码，请使用`--skip-password`选项。

+   [`--password1[=*`pass_val`*]`](mysqldump.html#option_mysqldump_password1)

    用于连接到服务器的 MySQL 帐户的多因素认证因子 1 的密码。密码值是可选的。如果未提供，**mysqldump** 会提示输入密码。如果提供了密码，则`--password1=`后面必须*没有空格*。如果未指定密码选项，则默认不发送密码。

    在命令行上指定密码应被视为不安全。为了避免在命令行上提供密码，请使用选项文件。请参阅第 8.1.2.1 节，“密码安全的最终用户指南”。

    明确指定没有密码且**mysqldump**不应提示输入密码，请使用`--skip-password1`选项。

    `--password1` 和 `--password` 是同义词，`--skip-password1` 和 `--skip-password` 也是同义词。

+   [`--password2[=*`pass_val`*]`](mysqldump.html#option_mysqldump_password2)

    用于连接到服务器的 MySQL 帐户的多因素认证因子 2 的密码。此选项的语义与`--password1`的语义类似；有关详细信息，请参阅该选项的描述。

+   [`--password3[=*`pass_val`*]`](mysqldump.html#option_mysqldump_password3)

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
    | 有效值 | `GSSAPI` |

    在 Windows 上，`authentication_kerberos_client`认证插件支持此插件选项。客户端用户可以在运行时设置两个可能的值：`SSPI`和`GSSAPI`。

    客户端端插件选项的默认值使用了 Security Support Provider Interface (SSPI)，它能够从 Windows 内存缓存中获取凭据。另外，客户端用户可以选择支持在 Windows 上通过 MIT Kerberos 库使用 Generic Security Service Application Program Interface (GSSAPI)的模式。GSSAPI 能够通过使用**kinit**命令之前生成的缓存凭据。

    有关更多信息，请参见 GSSAPI 模式下 Windows 客户端的命令。

+   `--plugin-dir=*`dir_name`*`

    | 命令行格式 | `--plugin-dir=dir_name` |
    | --- | --- |
    | 类型 | 目录名称 |

    查找插件的目录。如果使用`--default-auth`选项指定了一个认证插件，但**mysqldump**找不到它，那么请指定此选项。请参见第 8.2.17 节，“可插拔认证”。

+   `--port=*`port_num`*`, `-P *`port_num`*`

    | 命令行格式 | `--port=port_num` |
    | --- | --- |
    | 类型 | 数字 |
    | 默认值 | `3306` |

    用于 TCP/IP 连接的端口号。

+   `--protocol={TCP|SOCKET|PIPE|MEMORY}`

    | 命令行格式 | `--protocol=type` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `[查看文本]` |
    | 有效值 | `TCP``SOCKET``PIPE``MEMORY` |

    用于连接到服务器的传输协议。当其他连接参数通常导致使用不希望使用的协议时，这很有用。有关允许的值的详细信息，请参见第 6.2.7 节，“连接传输协议”。

+   `--server-public-key-path=*`file_name`*`

    | 命令行格式 | `--server-public-key-path=file_name` |
    | --- | --- |
    | 类型 | 文件名 |

    以 PEM 格式包含客户端端的服务器所需的公钥副本的文件路径名，用于 RSA 密钥对密码交换。此选项适用于使用 `sha256_password` 或 `caching_sha2_password` 认证插件进行身份验证的客户端。对于不使用这些插件进行身份验证的帐户，此选项将被忽略。如果不使用基于 RSA 的密码交换，例如客户端使用安全连接连接到服务器时，此选项也将被忽略。

    如果给定`--server-public-key-path=*`file_name`*`，并指定一个有效的公钥文件，则优先使用该文件，而不是`--get-server-public-key`。

    对于 `sha256_password`，此选项仅在使用 OpenSSL 构建 MySQL 时适用。

    有关 `sha256_password` 和 `caching_sha2_password` 插件的信息，请参阅第 8.4.1.3 节，“SHA-256 可插拔认证”，以及第 8.4.1.2 节，“缓存 SHA-2 可插拔认证”。

+   `--socket=*`path`*`, `-S *`path`*`

    | 命令行格式 | `--socket={file_name&#124;pipe_name}` |
    | --- | --- |
    | 类型 | 字符串 |

    对于连接到 `localhost` 的连接，使用的 Unix 套接字文件，或者在 Windows 上使用的命名管道的名称。

    在 Windows 上，此选项仅在服务器启动时启用了支持命名管道连接的 `named_pipe` 系统变量时才适用。此外，进行连接的用户必须是由 `named_pipe_full_access_group` 系统变量指定的 Windows 组的成员。

+   `--ssl*`

    以 `--ssl` 开头的选项指定是否使用加密连接到服务器，并指示 SSL 密钥和证书的位置。请参阅加密连接的命令选项。

+   `--ssl-fips-mode={OFF|ON|STRICT}`

    | 命令行格式 | `--ssl-fips-mode={OFF&#124;ON&#124;STRICT}` |
    | --- | --- |
    | 弃用 | 8.0.34 |
    | 类型 | 枚举 |
    | 默认值 | `OFF` |
    | 有效值 | `OFF``ON``STRICT` |

    控制是否在客户端端启用 FIPS 模式。`--ssl-fips-mode`选项与其他`--ssl-*`xxx`*`选项不同，它不用于建立加密连接，而是用于影响允许的加密操作。请参见第 8.8 节，“FIPS 支持”。

    允许使用的`--ssl-fips-mode`值为：

    +   `OFF`：禁用 FIPS 模式。

    +   `ON`：启用 FIPS 模式。

    +   `STRICT`：启用“严格” FIPS 模式。

    注意

    如果 OpenSSL FIPS 对象模块不可用，则`--ssl-fips-mode`的唯一允许值为`OFF`。在这种情况下，将`--ssl-fips-mode`设置为`ON`或`STRICT`会导致客户端在启动时产生警告并在非 FIPS 模式下运行。

    从 MySQL 8.0.34 开始，此选项已被弃用。预计将在将来的 MySQL 版本中删除。

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

    允许的加密连接的 TLS 协议。该值是一个或多个逗号分隔的协议名称列表。可以为此选项命名的协议取决于用于编译 MySQL 的 SSL 库。有关详细信息，请参见第 8.3.2 节，“加密连接 TLS 协议和密码”。

+   `--user=*`user_name`*`，`-u *`user_name`*`

    | 命令行格式 | `--user=user_name` |
    | --- | --- |
    | 类型 | 字符串 |

    用于连接到服务器的 MySQL 帐户的用户名。

    如果您正在使用 MySQL 8.0.31 或更高版本的`Rewriter`插件，则应授予该用户`SKIP_QUERY_REWRITE`权限。

+   `--zstd-compression-level=*`level`*`

    | 命令行格式 | `--zstd-compression-level=#` |
    | --- | --- |
    | 引入版本 | 8.0.18 |
    | 类型 | 整数 |

    用于使用`zstd`压缩算法连接到服务器的连接的压缩级别。允许的级别从 1 到 22，较大的值表示更高级别的压缩。默认的`zstd`压缩级别为 3。压缩级别设置对不使用`zstd`压缩的连接没有影响。

    更多信息，请参见第 6.2.8 节，“连接压缩控制”。

    此选项在 MySQL 8.0.18 中添加。

#### 选项文件选项

这些选项用于控制要读取哪些选项文件。

+   `--defaults-extra-file=*`file_name`*`

    | 命令行格式 | `--defaults-extra-file=file_name` |
    | --- | --- |
    | 类型 | 文件名 |

    在全局选项文件之后但（在 Unix 上）在用户选项文件之前读取此选项文件。如果文件不存在或无法访问，则会出现错误。如果*`file_name`*不是绝对路径名，则将其解释为相对于当前目录。

    有关此及其他选项文件选项的更多信息，请参见第 6.2.2.3 节，“影响选项文件处理的命令行选项”。

+   `--defaults-file=*`file_name`*`

    | 命令行格式 | `--defaults-file=file_name` |
    | --- | --- |
    | 类型 | 文件名 |

    仅使用给定的选项文件。如果文件不存在或无法访问，则会出现错误。如果*`file_name`*不是绝对路径名，则将其解释为相对于当前目录。

    例外：即使使用`--defaults-file`，客户端程序也会读取`.mylogin.cnf`。

    有关此及其他选项文件选项的更多信息，请参见第 6.2.2.3 节，“影响选项文件处理的命令行选项”。

+   `--defaults-group-suffix=*`str`*`

    | 命令行格式 | `--defaults-group-suffix=str` |
    | --- | --- |
    | 类型 | 字符串 |

    不仅读取通常的选项组，还读取具有通常名称和后缀*`str`*的组。例如，**mysqldump**通常会读取`[client]`和`[mysqldump]`组。如果此选项给出为`--defaults-group-suffix=_other`，**mysqldump**还会读取`[client_other]`和`[mysqldump_other]`组。

    有关此及其他选项文件选项的更多信息，请参见第 6.2.2.3 节，“影响选项文件处理的命令行选项”。

+   `--no-defaults`

    | 命令行格式 | `--no-defaults` |
    | --- | --- |

    不要读取任何选项文件。如果由于从选项文件中读取未知选项而导致程序启动失败，则可以使用`--no-defaults`来防止读取这些选项。

    例外情况是，如果存在`.mylogin.cnf`文件，则在所有情况下都会读取该文件。即使使用`--no-defaults`，也可以以比在命令行上更安全的方式指定密码。要创建`.mylogin.cnf`，请使用**mysql_config_editor**实用程序。请参阅第 6.6.7 节，“mysql_config_editor — MySQL 配置实用程序”。

    有关此选项和其他选项文件选项的其他信息，请参阅第 6.2.2.3 节，“影响选项文件处理的命令行选项”。

+   `--print-defaults`

    | 命令行格式 | `--print-defaults` |
    | --- | --- |

    打印程序名称以及从选项文件获取的所有选项。

    有关此选项和其他选项文件选项的其他信息，请参阅第 6.2.2.3 节，“影响选项文件处理的命令行选项”。

#### DDL 选项

**mysqldump**的使用场景包括设置一个全新的 MySQL 实例（包括数据库表），以及用现有数据库和表替换现有实例中的数据。以下选项允许您在恢复转储时指定要拆除和设置的内容，通过在转储文件中编码各种 DDL 语句。

+   `--add-drop-database`

    | 命令行格式 | `--add-drop-database` |
    | --- | --- |

    在每个`CREATE DATABASE`语句之前写入一个`DROP DATABASE`语句。此选项通常与`--all-databases`或`--databases`选项一起使用，因为除非指定了其中一个选项，否则不会写入任何`CREATE DATABASE`语句。

    注意

    在 MySQL 8.0 中，`mysql` 模式被视为无法被最终用户删除的系统模式。如果使用 `--add-drop-database` 与 `--all-databases` 或与包括 `mysql` 的要转储的模式列表的 `--databases` 一起使用，则转储文件包含一个导致重新加载转储文件时出错的 `DROP DATABASE `mysql`` 语句。

    相反，要使用 `--add-drop-database`，请使用带有要转储的模式列表的 `--databases` 选项，其中列表不包括 `mysql`。

+   `--add-drop-table`

    | 命令行格式 | `--add-drop-table` |
    | --- | --- |

    在每个 `CREATE TABLE` 语句之前写入一个 `DROP TABLE` 语句。

+   `--add-drop-trigger`

    | 命令行格式 | `--add-drop-trigger` |
    | --- | --- |

    在每个 `CREATE TRIGGER` 语句之前写入一个 `DROP TRIGGER` 语句。

+   `--all-tablespaces`, `-Y`

    | 命令行格式 | `--all-tablespaces` |
    | --- | --- |

    将用于创建 `NDB` 表的所有表空间的 SQL 语句添加到表转储中。这些信息在 **mysqldump** 的输出中通常不包含。此选项目前仅与 NDB Cluster 表相关。

+   `--no-create-db`, `-n`

    | 命令行格式 | `--no-create-db` |
    | --- | --- |

    如果给定了 `--databases` 或 `--all-databases` 选项，则在输出中包含的 `CREATE DATABASE` 语句将被抑制。

+   `--no-create-info`, `-t`

    | 命令行格式 | `--no-create-info` |
    | --- | --- |

    不要编写创建每个转储表的 `CREATE TABLE` 语句。

    注意

    此选项*不*排除从 **mysqldump** 输出中创建日志文件组或表空间的语句；但是，您可以使用 `--no-tablespaces` 选项来实现此目的。

+   `--no-tablespaces`, `-y`

    | 命令行格式 | `--no-tablespaces` |
    | --- | --- |

    此选项在**mysqldump**的输出中抑制所有`CREATE LOGFILE GROUP`和`CREATE TABLESPACE`语句。

+   `--replace`

    | 命令行格式 | `--replace` |
    | --- | --- |

    写入`REPLACE`语句而不是`INSERT`语句。

#### 调试选项

以下选项打印调试信息，在转储文件中编码调试信息，或让转储操作继续进行，而不考虑潜在问题。

+   `--allow-keywords`

    | 命令行格式 | `--allow-keywords` |
    | --- | --- |

    允许创建关键字列名。这通过在每个列名前缀中添加表名来实现。

+   `--comments`, `-i`

    | 命令行格式 | `--comments` |
    | --- | --- |

    在转储文件中写入额外信息，如程序版本、服务器版本和主机。此选项默认启用。要抑制此额外信息，请使用`--skip-comments`。

+   [`--debug[=*`debug_options`*]`](mysqldump.html#option_mysqldump_debug), `-# [*`debug_options`*]`

    | 命令行格式 | `--debug[=debug_options]` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `d:t:o,/tmp/mysqldump.trace` |

    写入调试日志。典型的*`debug_options`*字符串是`d:t:o,*`file_name`*`。默认值为`d:t:o,/tmp/mysqldump.trace`。

    该选项仅在使用`WITH_DEBUG`构建 MySQL 时才可用。由 Oracle 提供的 MySQL 发布二进制文件*不*使用此选项构建。

+   `--debug-check`

    | 命令行格式 | `--debug-check` |
    | --- | --- |
    | 类型 | 布尔值 |
    | 默认值 | `FALSE` |

    在程序退出时打印一些调试信息。

    该选项仅在使用`WITH_DEBUG`构建 MySQL 时才可用。由 Oracle 提供的 MySQL 发布二进制文件*不*使用此选项构建。

+   `--debug-info`

    | 命令行格式 | `--debug-info` |
    | --- | --- |
    | 类型 | 布尔值 |
    | 默认值 | `FALSE` |

    在程序退出时打印调试信息以及内存和 CPU 使用统计信息。

    该选项仅在使用`WITH_DEBUG`构建 MySQL 时才可用。由 Oracle 提供的 MySQL 发布二进制文件*不*使用此选项构建。

+   `--dump-date`

    | 命令行格式 | `--dump-date` |
    | --- | --- |
    | 类型 | 布尔值 |
    | 默认值 | `TRUE` |

    如果提供了`--comments`选项，**mysqldump**会在转储结束时生成以下形式的注释：

    ```sql
    -- Dump completed on *DATE*
    ```

    然而，日期会导致在不同时间生成的转储文件看起来不同，即使数据本身相同。`--dump-date`和`--skip-dump-date`控制是否将日期添加到注释中。默认是`--dump-date`（在注释中包含日期）。`--skip-dump-date`会禁止打印日期。

+   `--force`, `-f`

    | 命令行格式 | `--force` |
    | --- | --- |

    忽略所有错误；即使在表转储过程中发生 SQL 错误，也继续执行。

    此选项的一个用途是导致**mysqldump**在遇到因引用已删除的表而无效的视图时继续执行。没有`--force`，**mysqldump**会退出并显示错误消息。有了`--force`，**mysqldump**会打印错误消息，但也会将包含视图定义的 SQL 注释写入转储输出并继续执行。

    如果还提供了`--ignore-error`选项来忽略特定错误，则`--force`优先。

+   `--log-error=*`file_name`*`

    | 命令行格式 | `--log-error=file_name` |
    | --- | --- |
    | 类型 | 文件名 |

    通过将警告和错误附加到指定文件来记录。默认情况下不记录任何日志。

+   `--skip-comments`

    | 命令行格式 | `--skip-comments` |
    | --- | --- |

    请参阅`--comments`选项的描述。

+   `--verbose`, `-v`

    | 命令行格式 | `--verbose` |
    | --- | --- |

    详细模式。打印有关程序操作的更多信息。

#### 帮助选项

以下选项显示有关**mysqldump**命令本身的信息。

+   `--help`, `-?`

    | 命令行格式 | `--help` |
    | --- | --- |

    显示帮助消息并退出。

+   `--version`, `-V`

    | 命令行格式 | `--version` |
    | --- | --- |

    显示版本信息并退出。

#### 国际化选项

以下选项更改**mysqldump**命令如何使用国家语言设置表示字符数据。

+   `--character-sets-dir=*`dir_name`*`

    | 命令行格式 | `--character-sets-dir=dir_name` |
    | --- | --- |
    | 类型 | 目录名称 |

    安装字符集的目录。参见第 12.15 节，“字符集配置”。

+   `--default-character-set=*`charset_name`*`

    | 命令行格式 | `--default-character-set=charset_name` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `utf8` |

    使用*`charset_name`*作为默认字符集。参见第 12.15 节，“字符集配置”。如果未指定字符集，**mysqldump**将使用`utf8mb4`。

+   `--no-set-names`, `-N`

    | 命令行格式 | `--no-set-names` |
    | --- | --- |
    | 已弃用 | 是 |

    关闭`--set-charset`设置，与指定`--skip-set-charset`相同。

+   `--set-charset`

    | 命令行格式 | `--set-charset` |
    | --- | --- |
    | 禁用者 | `skip-set-charset` |

    将`SET NAMES *`default_character_set`*`写入输出。此选项默认启用。要禁止`SET NAMES`语句，请使用`--skip-set-charset`。

#### 复制选项

**mysqldump**命令经常用于在复制配置中的复制服务器上创建空实例或包含数据的实例。以下选项适用于在复制源服务器和副本上转储和恢复数据。

+   `--apply-replica-statements`

    | 命令行格式 | `--apply-replica-statements` |
    | --- | --- |
    | 引入版本 | 8.0.26 |
    | 类型 | 布尔值 |
    | 默认值 | `FALSE` |

    从 MySQL 8.0.26 开始，使用`--apply-replica-statements`，在 MySQL 8.0.26 之前，使用`--apply-slave-statements`。这两个选项具有相同的效果。对于使用`--dump-replica`或`--dump-slave`选项生成的副本转储，这些选项在具有二进制日志坐标的语句之前添加一个`STOP REPLICA`（在 MySQL 8.0.22 之前为`STOP SLAVE`）语句，并在输出末尾添加一个`START REPLICA`语句。

+   `--apply-slave-statements`

    | 命令行格式 | `--apply-slave-statements` |
    | --- | --- |
    | 已弃用 | 8.0.26 |
    | 类型 | 布尔值 |
    | 默认值 | `FALSE` |

    在 MySQL 8.0.26 之前，请使用此选项，而不是`--apply-replica-statements`。这两个选项具有相同的效果。

+   `--delete-source-logs`

    | 命令行格式 | `--delete-source-logs` |
    | --- | --- |
    | 引入版本 | 8.0.26 |

    从 MySQL 8.0.26 开始，请使用`--delete-source-logs`，在 MySQL 8.0.26 之前，请使用`--delete-master-logs`。这两个选项具有相同的效果。在复制源服务器上，这些选项通过在执行转储操作后向服务器发送`PURGE BINARY LOGS`语句来删除二进制日志。这些选项需要`RELOAD`权限以及足够执行该语句的权限。这些选项会自动启用`--source-data`或`--master-data`。

+   `--delete-master-logs`

    | 命令行格式 | `--delete-master-logs` |
    | --- | --- |
    | 已弃用 | 8.0.26 |

    在 MySQL 8.0.26 之前，请使用此选项，而不是`--delete-source-logs`。这两个选项具有相同的效果。

+   [`--dump-replica[=*`value`*]`](mysqldump.html#option_mysqldump_dump-replica)

    | 命令行格式 | `--dump-replica[=value]` |
    | --- | --- |
    | 引入版本 | 8.0.26 |
    | 类型 | 数值 |
    | 默认值 | `1` |
    | 有效值 | `1``2` |

    从 MySQL 8.0.26 开始，请使用`--dump-replica`，在 MySQL 8.0.26 之前，请使用`--dump-slave`。这两个选项具有相同的效果。这些选项类似于`--source-data`，只是用于转储副本服务器以生成可用于设置另一台具有与转储服务器相同源的服务器的转储文件。这些选项导致转储输出包括一个`CHANGE REPLICATION SOURCE TO`语句（从 MySQL 8.0.23 开始）或一个`CHANGE MASTER TO`语句（在 MySQL 8.0.23 之前），指示转储副本的源的二进制日志坐标（文件名和位置）。`CHANGE REPLICATION SOURCE TO`语句从`SHOW REPLICA STATUS`输出中读取`Relay_Master_Log_File`和`Exec_Master_Log_Pos`的值，并将它们分别用于`SOURCE_LOG_FILE`和`SOURCE_LOG_POS`。这些是副本开始复制的复制源服务器坐标。

    注意

    从已执行的中继日志事务序列中的不一致性可能导致使用错误的位置。有关更多信息，请参见第 19.5.1.34 节，“复制和事务不一致性”。

    `--dump-replica` 或 `--dump-slave` 导致使用源的坐标而不是备份服务器的坐标，就像`--source-data`或`--master-data`选项所做的那样。此外，指定此选项会覆盖使用的`--source-data`或 `--master-data` 选项，并有效地被忽略。

    警告

    如果将要应用备份的服务器使用了 `gtid_mode=ON` 和 `SOURCE_AUTO_POSITION=1` 或 `MASTER_AUTO_POSITION=1`，则不应使用 `--dump-replica` 或 `--dump-slave`。

    选项值的处理方式与`--source-data`相同。设置为无值或 1 会导致写入一个 `CHANGE REPLICATION SOURCE TO` 语句（从 MySQL 8.0.23 开始）或 `CHANGE MASTER TO` 语句（在 MySQL 8.0.23 之前）。设置为 2 会导致写入该语句，但用 SQL 注释括起来。在启用或禁用其他选项以及处理锁定方面，它与 `--source-data` 具有相同的效果。

    `--dump-replica` 或 `--dump-slave` 导致**mysqldump**在备份前停止复制 SQL 线程，并在备份后重新启动。

    `--dump-replica` 或 `--dump-slave` 向服务器发送一个 `SHOW REPLICA STATUS` 语句以获取信息，因此它们需要具有足够权限执行该语句。

    `--apply-replica-statements` 和 `--include-source-host-port` 选项可以与 `--dump-replica` 或 `--dump-slave` 一起使用。

+   [`--dump-slave[=*`value`*]`](mysqldump.html#option_mysqldump_dump-slave)

    | 命令行格式 | `--dump-slave[=value]` |
    | --- | --- |
    | 已弃用 | 8.0.26 |
    | 类型 | 数值 |
    | 默认值 | `1` |
    | 有效数值 | `1``2` |

    在 MySQL 8.0.26 之前，请使用这个选项，而不是`--dump-replica`。这两个选项具有相同的效果。

+   `--include-source-host-port`

    | 命令行格式 | `--include-source-host-port` |
    | --- | --- |
    | 引入版本 | 8.0.26 |
    | 类型 | 布尔值 |
    | 默认值 | `FALSE` |

    从 MySQL 8.0.26 开始，使用`--include-source-host-port`，在 MySQL 8.0.26 之前，使用`--include-master-host-port`。这两个选项具有相同的效果。这些选项为主机名和复制源的 TCP/IP 端口号添加了`SOURCE_HOST` | `MASTER_HOST` 和 `SOURCE_PORT` | `MASTER_PORT` 选项，以便在使用`--dump-replica`或`--dump-slave`选项生成的复制转储中的`CHANGE REPLICATION SOURCE TO`语句（从 MySQL 8.0.23 开始）或`CHANGE MASTER TO`语句（在 MySQL 8.0.23 之前）。

+   `--include-master-host-port`

    | 命令行格式 | `--include-master-host-port` |
    | --- | --- |
    | 已弃用 | 8.0.26 |
    | 类型 | 布尔值 |
    | 默认值 | `FALSE` |

    在 MySQL 8.0.26 之前，请使用此选项，而不是`--include-source-host-port`。这两个选项具有相同的效果。

+   [`--source-data[=*`value`*]`](mysqldump.html#option_mysqldump_source-data)

    | 命令���格式 | `--source-data[=value]` |
    | --- | --- |
    | 引入版本 | 8.0.26 |
    | 类型 | 数值 |
    | 默认值 | `1` |
    | 有效数值 | `1``2` |

    从 MySQL 8.0.26 开始，使用`--source-data`，在 MySQL 8.0.26 之前，使用`--master-data`。这两个选项具有相同的效果。这些选项用于转储复制源服务器，以生成一个可以用来设置另一台服务器作为源的副本的转储文件。这些选项导致转储输出包括一个`CHANGE REPLICATION SOURCE TO`语句（从 MySQL 8.0.23 开始）或`CHANGE MASTER TO`语句（在 MySQL 8.0.23 之前），指示了被转储服务器的二进制日志坐标（文件名和位置）。这些是副本在将转储文件加载到副本后应从中开始复制的复制源服务器坐标。

    如果选项值为 2，则`CHANGE REPLICATION SOURCE TO` | `CHANGE MASTER TO` 语句被写为 SQL 注释，因此仅具有信息性；在重新加载转储文件时不起作用。如果选项值为 1，则该语句不会被写为注释，并在重新加载转储文件时生效。如果未指定选项值，则默认值为 1。

    `--source-data`和`--master-data`向服务器发送`SHOW MASTER STATUS`语句以获取信息，因此它们需要具有足够执行该语句的权限。此选项还需要`RELOAD`权限，并且必须启用二进制日志。

    `--source-data`和`--master-data`会自动关闭`--lock-tables`。它们还会打开`--lock-all-tables`，除非还指定了`--single-transaction`，在这种情况下，全局读锁仅在转储开始时的短时间内获取（请参阅`--single-transaction`的描述）。在所有情况下，日志上的任何操作都发生在转储的确切时刻。

    通过使用`--dump-replica`或`--dump-slave`选项，可以通过转储源的现有副本来设置副本，这将覆盖`--source-data`和`--master-data`，并导致它们被忽略。

+   [`--master-data[=*`value`*]`](mysqldump.html#option_mysqldump_master-data)

    | 命令行格式 | `--master-data[=value]` |
    | --- | --- |
    | 已弃用 | 8.0.26 |
    | 类型 | 数值 |
    | 默认值 | `1` |
    | 有效值 | `1``2` |

    在 MySQL 8.0.26 之前，请使用此选项，而不是`--source-data`。这两个选项具有相同的效果。

+   `--set-gtid-purged=*`value`*`

    | 命令行格式 | `--set-gtid-purged=value` |
    | --- | --- |
    | 类型 | 枚举 |
    | 默认值 | `AUTO` |
    | 有效值 | `OFF``ON``AUTO` |

    这个选项适用于使用基于 GTID 的复制的服务器(`gtid_mode=ON`)。它控制在转储输出中包含一个`SET @@GLOBAL.gtid_purged`语句，该语句更新重新加载转储文件的服务器上的`gtid_purged`的值，以添加源服务器的`gtid_executed`系统变量中的 GTID 集。`gtid_purged`保存了已应用于服务器上的所有事务的 GTID，但在服务器上的任何二进制日志文件中不存在。因此，**mysqldump**会为在源服务器上执行的事务添加 GTID，以便目标服务器记录这些事务已应用，尽管它在其二进制日志中没有这些事务。`--set-gtid-purged`还控制在重新加载转储文件时包含一个`SET @@SESSION.sql_log_bin=0`语句，该语句在禁用二进制日志记录时防止生成新的 GTID 并将其分配给转储文件中执行的事务，以便使用原始事务的 GTID。

    如果不设置`--set-gtid-purged`选项，则默认情况下，如果您要备份的服务器启用了 GTID，并且全局值的`gtid_executed`系统变量中的 GTID 集不为空，则在转储输出中包含一个`SET @@GLOBAL.gtid_purged`语句。如果服务器启用了 GTID，则还包括一个`SET @@SESSION.sql_log_bin=0`语句。

    您可以将`gtid_purged`的值替换为指定的 GTID 集，或者在语句中添加加号(+)以将指定的 GTID 集附加到已由`gtid_purged`持有的 GTID 集。由**mysqldump**记录的`SET @@GLOBAL.gtid_purged`语句包含一个加号(`+`)的特定版本注释，使得 MySQL 将转储文件中的 GTID 集添加到现有的`gtid_purged`值中。

    需要注意的是，由 **mysqldump** 包含在 `SET @@GLOBAL.gtid_purged` 语句中的值包括服务器上 `gtid_executed` 集合中的所有事务的 GTID，甚至包括更改了数据库的被抑制部分或服务器上未包含在部分转储中的其他数据库的事务。这意味着在重放转储文件的服务器上更新了 `gtid_purged` 值后，可能存在与目标服务器上的任何数据无关的 GTID。如果在目标服务器上不再重放任何进一步的转储文件，则多余的 GTID 不会对服务器未来的操作造成任何问题，但它们使得在复制拓扑中的不同服务器上比较或协调 GTID 集合变得更加困难。如果在目标服务器上重放包含相同 GTID 的进一步转储文件（例如，来自相同原始服务器的另一个部分转储），第二个转储文件中的任何 `SET @@GLOBAL.gtid_purged` 语句都会失败。在这种情况下，在重放转储文件之前要么手动删除该语句，要么在输出转储文件时不包含该语句。

    在 MySQL 8.0.32 之前：在使用 `--single-transaction` 选项时，可能会导致输出不一致。如果需要 `--set-gtid-purged=ON`，可以与 `--lock-all-tables` 一起使用，但这可能会在运行 **mysqldump** 时阻止并行查询。

    如果 `SET @@GLOBAL.gtid_purged` 语句在目标服务器上不会产生预期结果，您可以将该语句从输出中排除，或者（从 MySQL 8.0.17 开始）包含该语句但将其注释掉，以便不会自动执行。您也可以包含该语句，但在转储文件中手动编辑以实现预期结果。

    `--set-gtid-purged` 选项的可能值如下：

    `AUTO`

    默认值。如果在您备份的服务器上启用了 GTID，并且 `gtid_executed` 不为空，则将 `SET @@GLOBAL.gtid_purged` 添加到输出中，其中包含来自 `gtid_executed` 的 GTID 集合。如果启用了 GTID，则将 `SET @@SESSION.sql_log_bin=0` 添加到输出中。如果在服务器上未启用 GTID，则不会添加这些语句到输出中。

    `OFF`

    `SET @@GLOBAL.gtid_purged` 不会被添加到输出中，`SET @@SESSION.sql_log_bin=0` 也不会被添加到输出中。对于不使用 GTIDs 的服务器，请使用此选项或 `AUTO`。仅对使用 GTIDs 的服务器使用此选项，如果您确定所需的 GTID 集已经存在于目标服务器的 `gtid_purged` 中并且不应更改，或者如果您计划手动识别并添加任何缺失的 GTIDs。

    `ON`

    如果服务器启用了 GTIDs，`SET @@GLOBAL.gtid_purged` 将被添加到输出中（除非 `gtid_executed` 为空），并且 `SET @@SESSION.sql_log_bin=0` 将被添加到输出中。如果设置了此选项但服务器未启用 GTIDs，则会发生错误。对于使用 GTIDs 的服务器，请使用此选项或 `AUTO`，除非您确定 `gtid_executed` 中的 GTIDs 在目标服务器上不需要。

    `COMMENTED`

    从 MySQL 8.0.17 开始可用。如果服务器启用了 GTIDs，`SET @@GLOBAL.gtid_purged` 将被添加到输出中（除非 `gtid_executed` 为空），但被注释掉。这意味着 `gtid_executed` 的值在输出中是可用的，但在重新加载转储文件时不会自动执行任何操作。`SET @@SESSION.sql_log_bin=0` 被添加到输出中，并且没有被注释掉。通过 `COMMENTED`，您可以手动或通过自动化控制 `gtid_executed` 集的使用。例如，如果您正在将数据迁移到另一个已经具有不同活动数据库的服务器上，您可能更喜欢这样做。

#### 格式选项

以下选项指定如何表示整个转储文件或转储文件中某些类型的数据。它们还控制是否将某些可选信息写入转储文件。

+   `--compact`

    | 命令行格式 | `--compact` |
    | --- | --- |

    生成更紧凑的输出。此选项启用 `--skip-add-drop-table`、`--skip-add-locks`、`--skip-comments`、`--skip-disable-keys` 和 `--skip-set-charset` 选项。

+   `--compatible=*`name`*`

    | 命令行格式 | `--compatible=name[,name,...]` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `''` |
    | 有效值 | `ansi``mysql323``mysql40``postgresql``oracle``mssql``db2``maxdb``no_key_options``no_table_options``no_key_options` |

    生���与其他数据库系统或较旧的 MySQL 服务器更兼容的输出。此选项的唯一允许值为`ansi`，其含义与设置服务器 SQL 模式的相应选项相同。参见 Section 7.1.11, “Server SQL Modes”。

+   `--complete-insert`, `-c`

    | 命令行格式 | `--complete-insert` |
    | --- | --- |

    使用包含列名的完整`INSERT`语句。

+   `--create-options`

    | 命令行格式 | `--create-options` |
    | --- | --- |

    在`CREATE TABLE`语句中包含所有特定于 MySQL 的表选项。

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

    这些选项与`--tab`选项一起使用，具有与`LOAD DATA`中相应`FIELDS`子句相同的含义。参见 Section 15.2.9, “LOAD DATA Statement”。

+   `--hex-blob`

    | 命令行格式 | `--hex-blob` |
    | --- | --- |

    使用十六进制表示法转储二进制列（例如，`'abc'`变为`0x616263`）。受影响的数据类型包括`BINARY`类型，`VARBINARY`类型，`BLOB`类型，`BIT`类型，所有空间数据类型，以及其他非二进制数据类型（当与`binary`字符集一起使用时）。

    当使用`--tab`时，将忽略`--hex-blob`选项。

+   `--lines-terminated-by=...`

    | 命令行格式 | `--lines-terminated-by=string` |
    | --- | --- |
    | 类型 | 字符串 |

    此选项与`--tab`选项一起使用，具有与`LOAD DATA`中相应`LINES`子句相同的含义。参见 Section 15.2.9, “LOAD DATA Statement”。

+   `--quote-names`, `-Q`

    | Command-Line Format | `--quote-names` |
    | --- | --- |
    | Disabled by | `skip-quote-names` |

    在```sql characters. If the `ANSI_QUOTES` SQL mode is enabled, identifiers are quoted within `"` characters. This option is enabled by default. It can be disabled with `--skip-quote-names`, but this option should be given after any option such as `--compatible` that may enable `--quote-names`.

*   `--result-file=*`file_name`*`, `-r *`file_name`*`

    | Command-Line Format | `--result-file=file_name` |
    | Type | File name |

    Direct output to the named file. The result file is created and its previous contents overwritten, even if an error occurs while generating the dump.

    This option should be used on Windows to prevent newline `\n` characters from being converted to `\r\n` carriage return/newline sequences.

*   `--show-create-skip-secondary-engine=*`value`*`

    | Command-Line Format | `--show-create-skip-secondary-engine` |
    | Introduced | 8.0.18 |

    Excludes the `SECONDARY ENGINE` clause from `CREATE TABLE` statements. It does so by enabling the `show_create_table_skip_secondary_engine` system variable for the duration of the dump operation. Alternatively, you can enable the `show_create_table_skip_secondary_engine` system variable prior to using **mysqldump**.

    This option was added in MySQL 8.0.18\. Attempting a **mysqldump** operation with the `--show-create-skip-secondary-engine` option on a release prior to MySQL 8.0.18 that does not support the `show_create_table_skip_secondary_engine` variable causes an error.

*   `--tab=*`dir_name`*`, `-T *`dir_name`*`

    | Command-Line Format | `--tab=dir_name` |
    | Type | Directory name |

    Produce tab-separated text-format data files. For each dumped table, **mysqldump** creates a `*`tbl_name`*.sql` file that contains the `CREATE TABLE` statement that creates the table, and the server writes a `*`tbl_name`*.txt` file that contains its data. The option value is the directory in which to write the files.

    Note

    This option should be used only when **mysqldump** is run on the same machine as the **mysqld** server. Because the server creates `*.txt` files in the directory that you specify, the directory must be writable by the server and the MySQL account that you use must have the `FILE` privilege. Because **mysqldump** creates `*.sql` in the same directory, it must be writable by your system login account.

    By default, the `.txt` data files are formatted using tab characters between column values and a newline at the end of each line. The format can be specified explicitly using the `--fields-*`xxx`*` and `--lines-terminated-by` options.

    Column values are converted to the character set specified by the `--default-character-set` option.

*   `--tz-utc`

    | Command-Line Format | `--tz-utc` |
    | Disabled by | `skip-tz-utc` |

    This option enables `TIMESTAMP` columns to be dumped and reloaded between servers in different time zones. **mysqldump** sets its connection time zone to UTC and adds `SET TIME_ZONE='+00:00'` to the dump file. Without this option, `TIMESTAMP` columns are dumped and reloaded in the time zones local to the source and destination servers, which can cause the values to change if the servers are in different time zones. `--tz-utc` also protects against changes due to daylight saving time. `--tz-utc` is enabled by default. To disable it, use `--skip-tz-utc`.

*   `--xml`, `-X`

    | Command-Line Format | `--xml` |

    Write dump output as well-formed XML.

    **`NULL`, `'NULL'`, and Empty Values**: For a column named *`column_name`*, the `NULL` value, an empty string, and the string value `'NULL'` are distinguished from one another in the output generated by this option as follows.

    | Value: | XML Representation: |
    | `NULL` (*unknown value*) | `<field name="*`column_name`*" xsi:nil="true" />` |
    | `''` (*empty string*) | `<field name="*`column_name`*"></field>` |
    | `'NULL'` (*string value*) | `<field name="*`column_name`*">NULL</field>` |

    The output from the **mysql** client when run using the `--xml` option also follows the preceding rules. (See Section 6.5.1.1, “mysql Client Options”.)

    XML output from **mysqldump** includes the XML namespace, as shown here:

    ```中引用标识符（如数据库、表和列名）

    $> mysqldump --xml -u root world City

    <?xml version="1.0"?>

    <mysqldump xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">

    <database name="world">

    <table_structure name="City">

    <field Field="ID" Type="int(11)" Null="NO" Key="PRI" Extra="auto_increment" />

    <field Field="Name" Type="char(35)" Null="NO" Key="" Default="" Extra="" />

    <field Field="CountryCode" Type="char(3)" Null="NO" Key="" Default="" Extra="" />

    <field Field="District" Type="char(20)" Null="NO" Key="" Default="" Extra="" />

    <field Field="Population" Type="int(11)" Null="NO" Key="" Default="0" Extra="" />

    <key Table="City" Non_unique="0" Key_name="PRIMARY" Seq_in_index="1" Column_name="ID"

    Collation="A" Cardinality="4079" Null="" Index_type="BTREE" Comment="" />

    <options Name="City" Engine="MyISAM" Version="10" Row_format="Fixed" Rows="4079"

    Avg_row_length="67" Data_length="273293" Max_data_length="18858823439613951"

    Index_length="43008" Data_free="0" Auto_increment="4080"

    Create_time="2007-03-31 01:47:01" Update_time="2007-03-31 01:47:02"

    Collation="latin1_swedish_ci" Create_options="" Comment="" />

    </table_structure>

    <table_data name="City">

    <row>

    <field name="ID">1</field>

    <field name="Name">Kabul</field>

    <field name="CountryCode">AFG</field>

    <field name="District">Kabol</field>

    <field name="Population">1780000</field>

    </row>

    *...* <row>

    <field name="ID">4079</field>

    <field name="Name">Rafah</field>

    <field name="CountryCode">PSE</field>

    <field name="District">Rafah</field>

    <field name="Population">92020</field>

    </row>

    </table_data>

    </database>

    </mysqldump>

    ```sql

#### Filtering Options

The following options control which kinds of schema objects are written to the dump file: by category, such as triggers or events; by name, for example, choosing which databases and tables to dump; or even filtering rows from the table data using a `WHERE` clause.

*   `--all-databases`, `-A`

    | Command-Line Format | `--all-databases` |

    Dump all tables in all databases. This is the same as using the `--databases` option and naming all the databases on the command line.

    Note

    See the `--add-drop-database` description for information about an incompatibility of that option with `--all-databases`.

    Prior to MySQL 8.0, the `--routines` and `--events` options for **mysqldump** and **mysqlpump** were not required to include stored routines and events when using the `--all-databases` option: The dump included the `mysql` system database, and therefore also the `mysql.proc` and `mysql.event` tables containing stored routine and event definitions. As of MySQL 8.0, the `mysql.event` and `mysql.proc` tables are not used. Definitions for the corresponding objects are stored in data dictionary tables, but those tables are not dumped. To include stored routines and events in a dump made using `--all-databases`, use the `--routines` and `--events` options explicitly.

*   `--databases`, `-B`

    | Command-Line Format | `--databases` |

    Dump several databases. Normally, **mysqldump** treats the first name argument on the command line as a database name and following names as table names. With this option, it treats all name arguments as database names. `CREATE DATABASE` and `USE` statements are included in the output before each new database.

    This option may be used to dump the `performance_schema` database, which normally is not dumped even with the `--all-databases` option. (Also use the `--skip-lock-tables` option.)

    Note

    See the `--add-drop-database` description for information about an incompatibility of that option with `--databases`.

*   `--events`, `-E`

    | Command-Line Format | `--events` |

    Include Event Scheduler events for the dumped databases in the output. This option requires the `EVENT` privileges for those databases.

    The output generated by using `--events` contains `CREATE EVENT` statements to create the events.

*   [`--ignore-error=*`error[,error]...`*`](mysqldump.html#option_mysqldump_ignore-error)

    | Command-Line Format | `--ignore-error=error[,error]...` |
    | Type | String |

    Ignore the specified errors. The option value is a list of comma-separated error numbers specifying the errors to ignore during **mysqldump** execution. If the `--force` option is also given to ignore all errors, `--force` takes precedence.

*   `--ignore-table=*`db_name.tbl_name`*`

    | Command-Line Format | `--ignore-table=db_name.tbl_name` |
    | Type | String |

    Do not dump the given table, which must be specified using both the database and table names. To ignore multiple tables, use this option multiple times. This option also can be used to ignore views.

*   `--no-data`, `-d`

    | Command-Line Format | `--no-data` |

    Do not write any table row information (that is, do not dump table contents). This is useful if you want to dump only the `CREATE TABLE` statement for the table (for example, to create an empty copy of the table by loading the dump file).

*   `--routines`, `-R`

    | Command-Line Format | `--routines` |

    Include stored routines (procedures and functions) for the dumped databases in the output. This option requires the global `SELECT` privilege.

    The output generated by using `--routines` contains `CREATE PROCEDURE` and `CREATE FUNCTION` statements to create the routines.

*   `--skip-generated-invisible-primary-key`

    | Command-Line Format | `--skip-generated-invisible-primary-key` |
    | Introduced | 8.0.30 |
    | Type | Boolean |
    | Default Value | `FALSE` |

    This option is available beginning with MySQL 8.0.30, and causes generated invisible primary keys to be excluded from the output. For more information, see Section 15.1.20.11, “Generated Invisible Primary Keys”.

*   `--tables`

    | Command-Line Format | `--tables` |

    Override the `--databases` or `-B` option. **mysqldump** regards all name arguments following the option as table names.

*   `--triggers`

    | Command-Line Format | `--triggers` |
    | Disabled by | `skip-triggers` |

    Include triggers for each dumped table in the output. This option is enabled by default; disable it with `--skip-triggers`.

    To be able to dump a table's triggers, you must have the `TRIGGER` privilege for the table.

    Multiple triggers are permitted. **mysqldump** dumps triggers in activation order so that when the dump file is reloaded, triggers are created in the same activation order. However, if a **mysqldump** dump file contains multiple triggers for a table that have the same trigger event and action time, an error occurs for attempts to load the dump file into an older server that does not support multiple triggers. (For a workaround, see Downgrade Notes; you can convert triggers to be compatible with older servers.)

*   `--where='*`where_condition`*'`, `-w '*`where_condition`*'`

    | Command-Line Format | `--where='where_condition'` |

    Dump only rows selected by the given `WHERE` condition. Quotes around the condition are mandatory if it contains spaces or other characters that are special to your command interpreter.

    Examples:

    ```

    --where="user='jimf'"

    -w"userid>1"

    -w"userid<1"

    ```sql

#### Performance Options

The following options are the most relevant for the performance particularly of the restore operations. For large data sets, restore operation (processing the `INSERT` statements in the dump file) is the most time-consuming part. When it is urgent to restore data quickly, plan and test the performance of this stage in advance. For restore times measured in hours, you might prefer an alternative backup and restore solution, such as MySQL Enterprise Backup for `InnoDB`-only and mixed-use databases.

Performance is also affected by the transactional options, primarily for the dump operation.

*   `--column-statistics`

    | Command-Line Format | `--column-statistics` |
    | Type | Boolean |
    | Default Value | `OFF` |

    Add `ANALYZE TABLE` statements to the output to generate histogram statistics for dumped tables when the dump file is reloaded. This option is disabled by default because histogram generation for large tables can take a long time.

*   `--disable-keys`, `-K`

    | Command-Line Format | `--disable-keys` |

    For each table, surround the `INSERT` statements with `/*!40000 ALTER TABLE *`tbl_name`* DISABLE KEYS */;` and `/*!40000 ALTER TABLE *`tbl_name`* ENABLE KEYS */;` statements. This makes loading the dump file faster because the indexes are created after all rows are inserted. This option is effective only for nonunique indexes of `MyISAM` tables.

*   `--extended-insert`, `-e`

    | Command-Line Format | `--extended-insert` |
    | Disabled by | `skip-extended-insert` |

    Write `INSERT` statements using multiple-row syntax that includes several `VALUES` lists. This results in a smaller dump file and speeds up inserts when the file is reloaded.

*   `--insert-ignore`

    | Command-Line Format | `--insert-ignore` |

    Write `INSERT IGNORE` statements rather than `INSERT` statements.

*   `--max-allowed-packet=*`value`*`

    | Command-Line Format | `--max-allowed-packet=value` |
    | Type | Numeric |
    | Default Value | `25165824` |

    The maximum size of the buffer for client/server communication. The default is 24MB, the maximum is 1GB.

    Note

    The value of this option is specific to **mysqldump** and should not be confused with the MySQL server's `max_allowed_packet` system variable; the server value cannot be exceeded by a single packet from **mysqldump**, regardless of any setting for the **mysqldump** option, even if the latter is larger.

*   `--mysqld-long-query-time=*`value`*`

    | Command-Line Format | `--mysqld-long-query-time=value` |
    | Introduced | 8.0.30 |
    | Type | Numeric |
    | Default Value | `Server global setting` |

    Set the session value of the `long_query_time` system variable. Use this option, which is available from MySQL 8.0.30, if you want to increase the time allowed for queries from **mysqldump** before they are logged to the slow query log file. **mysqldump** performs a full table scan, which means its queries can often exceed a global `long_query_time` setting that is useful for regular queries. The default global setting is 10 seconds.

    You can use `--mysqld-long-query-time` to specify a session value from 0 (meaning that every query from **mysqldump** is logged to the slow query log) to 31536000, which is 365 days in seconds. For **mysqldump**’s option, you can only specify whole seconds. When you do not specify this option, the server’s global setting applies to **mysqldump**’s queries.

*   `--net-buffer-length=*`value`*`

    | Command-Line Format | `--net-buffer-length=value` |
    | Type | Numeric |
    | Default Value | `16384` |

    The initial size of the buffer for client/server communication. When creating multiple-row `INSERT` statements (as with the `--extended-insert` or `--opt` option), **mysqldump** creates rows up to `--net-buffer-length` bytes long. If you increase this variable, ensure that the MySQL server `net_buffer_length` system variable has a value at least this large.

*   `--network-timeout`, `-M`

    | Command-Line Format | `--network-timeout[={0&#124;1}]` |
    | Type | Boolean |
    | Default Value | `TRUE` |

    Enable large tables to be dumped by setting `--max-allowed-packet` to its maximum value and network read and write timeouts to a large value. This option is enabled by default. To disable it, use `--skip-network-timeout`.

*   `--opt`

    | Command-Line Format | `--opt` |
    | Disabled by | `skip-opt` |

    This option, enabled by default, is shorthand for the combination of `--add-drop-table` `--add-locks` `--create-options` `--disable-keys` `--extended-insert` `--lock-tables` `--quick` `--set-charset`. It gives a fast dump operation and produces a dump file that can be reloaded into a MySQL server quickly.

    Because the `--opt` option is enabled by default, you only specify its converse, the `--skip-opt` to turn off several default settings. See the discussion of `mysqldump` option groups for information about selectively enabling or disabling a subset of the options affected by `--opt`.

*   `--quick`, `-q`

    | Command-Line Format | `--quick` |
    | Disabled by | `skip-quick` |

    This option is useful for dumping large tables. It forces **mysqldump** to retrieve rows for a table from the server a row at a time rather than retrieving the entire row set and buffering it in memory before writing it out.

*   `--skip-opt`

    | Command-Line Format | `--skip-opt` |

    See the description for the `--opt` option.

#### Transactional Options

The following options trade off the performance of the dump operation, against the reliability and consistency of the exported data.

*   `--add-locks`

    | Command-Line Format | `--add-locks` |

    Surround each table dump with `LOCK TABLES` and `UNLOCK TABLES` statements. This results in faster inserts when the dump file is reloaded. See Section 10.2.5.1, “Optimizing INSERT Statements”.

*   `--flush-logs`, `-F`

    | Command-Line Format | `--flush-logs` |

    Flush the MySQL server log files before starting the dump. This option requires the `RELOAD` privilege. If you use this option in combination with the `--all-databases` option, the logs are flushed *for each database dumped*. The exception is when using `--lock-all-tables`, `--source-data` or `--master-data`, or `--single-transaction`. In these cases, the logs are flushed only once, corresponding to the moment that all tables are locked by `FLUSH TABLES WITH READ LOCK`. If you want your dump and the log flush to happen at exactly the same moment, you should use `--flush-logs` together with `--lock-all-tables`, `--source-data` or `--master-data`, or `--single-transaction`.

*   `--flush-privileges`

    | Command-Line Format | `--flush-privileges` |

    Add a `FLUSH PRIVILEGES` statement to the dump output after dumping the `mysql` database. This option should be used any time the dump contains the `mysql` database and any other database that depends on the data in the `mysql` database for proper restoration.

    Because the dump file contains a `FLUSH PRIVILEGES` statement, reloading the file requires privileges sufficient to execute that statement.

    Note

    For upgrades to MySQL 5.7 or higher from older versions, do not use `--flush-privileges`. For upgrade instructions in this case, see Section 3.5, “Changes in MySQL 8.0”.

*   `--lock-all-tables`, `-x`

    | Command-Line Format | `--lock-all-tables` |

    Lock all tables across all databases. This is achieved by acquiring a global read lock for the duration of the whole dump. This option automatically turns off `--single-transaction` and `--lock-tables`.

*   `--lock-tables`, `-l`

    | Command-Line Format | `--lock-tables` |

    For each dumped database, lock all tables to be dumped before dumping them. The tables are locked with `READ LOCAL` to permit concurrent inserts in the case of `MyISAM` tables. For transactional tables such as `InnoDB`, `--single-transaction` is a much better option than `--lock-tables` because it does not need to lock the tables at all.

    Because `--lock-tables` locks tables for each database separately, this option does not guarantee that the tables in the dump file are logically consistent between databases. Tables in different databases may be dumped in completely different states.

    Some options, such as `--opt`, automatically enable `--lock-tables`. If you want to override this, use `--skip-lock-tables` at the end of the option list.

*   `--no-autocommit`

    | Command-Line Format | `--no-autocommit` |

    Enclose the `INSERT` statements for each dumped table within `SET autocommit = 0` and `COMMIT` statements.

*   `--order-by-primary`

    | Command-Line Format | `--order-by-primary` |

    Dump each table's rows sorted by its primary key, or by its first unique index, if such an index exists. This is useful when dumping a `MyISAM` table to be loaded into an `InnoDB` table, but makes the dump operation take considerably longer.

*   `--shared-memory-base-name=*`name`*`

    | Command-Line Format | `--shared-memory-base-name=name` |
    | Platform Specific | Windows |

    On Windows, the shared-memory name to use for connections made using shared memory to a local server. The default value is `MYSQL`. The shared-memory name is case-sensitive.

    This option applies only if the server was started with the `shared_memory` system variable enabled to support shared-memory connections.

*   `--single-transaction`

    | Command-Line Format | `--single-transaction` |

    This option sets the transaction isolation mode to `REPEATABLE READ` and sends a `START TRANSACTION` SQL statement to the server before dumping data. It is useful only with transactional tables such as `InnoDB`, because then it dumps the consistent state of the database at the time when `START TRANSACTION` was issued without blocking any applications.

    The `RELOAD` or `FLUSH_TABLES` privilege is required with `--single-transaction` if both `gtid_mode=ON` and `gtid_purged=ON|AUTO`. This requirement was added in MySQL 8.0.32.

    When using this option, you should keep in mind that only `InnoDB` tables are dumped in a consistent state. For example, any `MyISAM` or `MEMORY` tables dumped while using this option may still change state.

    While a `--single-transaction` dump is in process, to ensure a valid dump file (correct table contents and binary log coordinates), no other connection should use the following statements: `ALTER TABLE`, `CREATE TABLE`, `DROP TABLE`, `RENAME TABLE`, `TRUNCATE TABLE`. A consistent read is not isolated from those statements, so use of them on a table to be dumped can cause the `SELECT` that is performed by **mysqldump** to retrieve the table contents to obtain incorrect contents or fail.

    The `--single-transaction` option and the `--lock-tables` option are mutually exclusive because `LOCK TABLES` causes any pending transactions to be committed implicitly.

    Before 8.0.32: Using `--single-transaction` together with the `--set-gtid-purged` option was not recommended; doing so could lead to inconsistencies in the output of **mysqldump**.

    To dump large tables, combine the `--single-transaction` option with the `--quick` option.

#### Option Groups

*   The `--opt` option turns on several settings that work together to perform a fast dump operation. All of these settings are on by default, because `--opt` is on by default. Thus you rarely if ever specify `--opt`. Instead, you can turn these settings off as a group by specifying `--skip-opt`, then optionally re-enable certain settings by specifying the associated options later on the command line.

*   The `--compact` option turns off several settings that control whether optional statements and comments appear in the output. Again, you can follow this option with other options that re-enable certain settings, or turn all the settings on by using the `--skip-compact` form.

When you selectively enable or disable the effect of a group option, order is important because options are processed first to last. For example, `--disable-keys` `--lock-tables` `--skip-opt` would not have the intended effect; it is the same as `--skip-opt` by itself.

#### Examples

To make a backup of an entire database:

```

mysqldump *db_name* > *backup-file.sql*

```sql

To load the dump file back into the server:

```

mysql *db_name* < *backup-file.sql*

```sql

Another way to reload the dump file:

```

mysql -e "source */path-to-backup/backup-file.sql*" *db_name*

```sql

**mysqldump** is also very useful for populating databases by copying data from one MySQL server to another:

```

mysqldump --opt *db_name* | mysql --host=*remote_host* -C *db_name*

```sql

You can dump several databases with one command:

```

mysqldump --databases *db_name1* [*db_name2* ...] > my_databases.sql

```sql

To dump all databases, use the `--all-databases` option:

```

mysqldump --all-databases > all_databases.sql

```sql

For `InnoDB` tables, **mysqldump** provides a way of making an online backup:

```

mysqldump --all-databases --master-data --single-transaction > all_databases.sql

```sql

Or, in MySQL 8.0.26 and later:

```

mysqldump --all-databases --source-data --single-transaction > all_databases.sql

```sql

This backup acquires a global read lock on all tables (using `FLUSH TABLES WITH READ LOCK`) at the beginning of the dump. As soon as this lock has been acquired, the binary log coordinates are read and the lock is released. If long updating statements are running when the `FLUSH` statement is issued, the MySQL server may get stalled until those statements finish. After that, the dump becomes lock free and does not disturb reads and writes on the tables. If the update statements that the MySQL server receives are short (in terms of execution time), the initial lock period should not be noticeable, even with many updates.

For point-in-time recovery (also known as “roll-forward,” when you need to restore an old backup and replay the changes that happened since that backup), it is often useful to rotate the binary log (see Section 7.4.4, “The Binary Log”) or at least know the binary log coordinates to which the dump corresponds:

```

mysqldump --all-databases --master-data=2 > all_databases.sql

```sql

Or, in MySQL 8.0.26 and later:

```

mysqldump --all-databases --source-data=2 > all_databases.sql

```sql

Or:

```

mysqldump --all-databases --flush-logs --master-data=2 > all_databases.sql

```sql

Or, in MySQL 8.0.26 and later:

```

mysqldump --all-databases --flush-logs --source-data=2 > all_databases.sql

```

`--source-data`或`--master-data`选项可以与`--single-transaction`选项同时使用，这为使用`InnoDB`存储引擎存储表的情况下，在进行点时间恢复之前适用的在线备份提供了便利的方式。

有关备份的更多信息，请参阅 Section 9.2, “Database Backup Methods”和 Section 9.3, “Example Backup and Recovery Strategy”。

+   要选择`--opt`的效果，除了一些功能外，对于每个功能使用`--skip`选项。要禁用扩展插入和内存缓冲，请使用`--opt` `--skip-extended-insert` `--skip-quick`。（实际上，`--skip-extended-insert` `--skip-quick`就足够了，因为`--opt`默认开启。）

+   要对除了禁用索引和表锁定之外的所有功能进行反转`--opt`，请使用`--skip-opt` `--disable-keys` `--lock-tables`。

#### 限制

**mysqldump** 默认不会转储`performance_schema`或`sys`模式。要转储其中任何一个，请在命令行上明确指定它们的名称。您还可以使用`--databases`选项指定它们。对于`performance_schema`，还要使用`--skip-lock-tables`选项。

**mysqldump** 不会转储`INFORMATION_SCHEMA`模式。

**mysqldump** 不会转储`InnoDB`的`CREATE TABLESPACE`语句。

**mysqldump** 不会转储 NDB Cluster `ndbinfo`信息数据库。

**mysqldump** 包括重新创建`mysql`数据库中的`general_log`和`slow_query_log`表的语句。日志表内容不会被转储。

如果由于权限不足而遇到备份视图的问题，请参阅 Section 27.9, “Restrictions on Views”以找到解决方法。
