# 6.9 环境变量

> 原文：[`dev.mysql.com/doc/refman/8.0/en/environment-variables.html`](https://dev.mysql.com/doc/refman/8.0/en/environment-variables.html)

本节列出了 MySQL 直接或间接使用的环境变量。这些大部分也可以在本手册的其他地方找到。

命令行上的选项优先于选项文件和环境变量中指定的值，选项文件中的值优先于环境变量中的值。在许多情况下，最好使用选项文件而不是环境变量来修改 MySQL 的行为。请参阅 Section 6.2.2.2, “Using Option Files”。

| 变量 | 描述 |
| --- | --- |
| `AUTHENTICATION_KERBEROS_CLIENT_LOG` | Kerberos 认证日志级别。 |
| `AUTHENTICATION_LDAP_CLIENT_LOG` | 客户端 LDAP 认证日志级别。 |
| `AUTHENTICATION_PAM_LOG` | PAM 认证插件调试日志设置。 |
| `CC` | 您的 C 编译器的名称（用于运行**CMake**）。 |
| `CXX` | 您的 C++编译器的名称（用于运行**CMake**）。 |
| `CC` | 您的 C 编译器的名称（用于运行**CMake**）。 |
| `DBI_USER` | Perl DBI 的默认用户名。 |
| `DBI_TRACE` | Perl DBI 的跟踪选项。 |
| `HOME` | **mysql**历史文件的默认路径是`$HOME/.mysql_history`。 |
| `LD_RUN_PATH` | 用于指定`libmysqlclient.so`的位置。 |
| `LIBMYSQL_ENABLE_CLEARTEXT_PLUGIN` | 启用`mysql_clear_password`认证插件；参见 Section 8.4.1.4, “Client-Side Cleartext Pluggable Authentication”。 |
| `LIBMYSQL_PLUGIN_DIR` | 查找客户端插件的目录。 |
| `LIBMYSQL_PLUGINS` | 预加载的客户端插件。 |
| `MYSQL_DEBUG` | 调试时的调试跟踪选项。 |
| `MYSQL_GROUP_SUFFIX` | 选项组后缀值（类似于指定`--defaults-group-suffix`）。 |
| `MYSQL_HISTFILE` | **mysql**历史文件的路径。如果设置了此变量，其值将覆盖`$HOME/.mysql_history`的默认值。 |
| `MYSQL_HISTIGNORE` | 指定不记录到`$HOME/.mysql_history`或`syslog`（如果给定`--syslog`）的语句模式。 |
| `MYSQL_HOME` | 服务器特定的`my.cnf`文件所在目录的路径。 |
| `MYSQL_HOST` | **mysql**命令行客户端使用的默认主机名。 |
| `MYSQL_OPENSSL_UDF_DH_BITS_THRESHOLD` | `create_dh_parameters()`的最大密钥长度。请参见 Section 8.6.3, “MySQL 企业加密用法和示例”。 |
| `MYSQL_OPENSSL_UDF_DSA_BITS_THRESHOLD` | `create_asymmetric_priv_key()`的最大 DSA 密钥长度。请参见 Section 8.6.3, “MySQL 企业加密用法和示例”。 |
| `MYSQL_OPENSSL_UDF_RSA_BITS_THRESHOLD` | `create_asymmetric_priv_key()`的最大 RSA 密钥长度。请参见 Section 8.6.3, “MySQL 企业加密用法和示例”。 |
| `MYSQL_PS1` | **mysql**命令行客户端中要使用的命令提示符。 |
| `MYSQL_PWD` | 连接到**mysqld**时的默认密码。使用此选项是不安全的。请参见表后面的注释。 |
| `MYSQL_TCP_PORT` | 默认的 TCP/IP 端口号。 |
| `MYSQL_TEST_LOGIN_FILE` | `.mylogin.cnf`登录路径文件的名称。 |
| `MYSQL_TEST_TRACE_CRASH` | 测试协议跟踪插件是否会使客户端崩溃。请参见表后面的注释。 |
| `MYSQL_TEST_TRACE_DEBUG` | 测试协议跟踪插件是否产生输出。请参见表后面的注释。 |
| `MYSQL_UNIX_PORT` | 默认的 Unix 套接字文件名；用于与`localhost`的连接。 |
| `MYSQLX_TCP_PORT` | X 插件默认的 TCP/IP 端口号。 |
| `MYSQLX_UNIX_PORT` | X 插件默认的 Unix 套接字文件名；用于与`localhost`的连接。 |
| `NOTIFY_SOCKET` | **mysqld** 用于与 systemd 通信的套接字。 |
| `PATH` | shell 用于查找 MySQL 程序。 |
| `PKG_CONFIG_PATH` | `mysqlclient.pc` **pkg-config**文件的位置。请参见表后面的注释。 |
| `TMPDIR` | 创建临时文件的目录。 |
| `TZ` | 这应该设置为您的本地时区。请参见 Section B.3.3.7, “时区问题”。 |
| `UMASK` | 创建文件时的用户文件创建模式。请参见表后面的注释。 |
| `UMASK_DIR` | 创建目录时的用户目录创建模式。请参见表后面的注释。 |
| `USER` | 连接到**mysqld**时 Windows 上的默认用户名。 |
| 变量 | 描述 |

有关**mysql**历史文件的信息，请参见 Section 6.5.1.3, “mysql 客户端日志记录”。

使用`MYSQL_PWD`指定 MySQL 密码被认为是*极其不安全*的，不应使用。一些版本的**ps**包括显示运行进程环境的选项。在某些系统上，如果设置了`MYSQL_PWD`，您的密码将暴露给运行**ps**的任何其他用户。即使在没有这种版本的**ps**的系统上，也不明智地假设没有其他用户可以检查进程环境的方法。

`MYSQL_PWD`在 MySQL 8.0 中已弃用；预计将在将来的 MySQL 版本中删除。

`MYSQL_TEST_LOGIN_FILE`是登录路径文件的路径名（由**mysql_config_editor**创建的文件）。如果未设置，默认值为 Windows 上的`%APPDATA%\MySQL\.mylogin.cnf`目录和非 Windows 系统上的`$HOME/.mylogin.cnf`。请参见 Section 6.6.7, “mysql_config_editor — MySQL 配置实用程序”。

`MYSQL_TEST_TRACE_DEBUG`和`MYSQL_TEST_TRACE_CRASH`变量控制测试协议跟踪客户端插件，如果 MySQL 构建时启用了该插件。有关更多信息，请参见使用测试协议跟踪插件。

默认的`UMASK`和`UMASK_DIR`值分别为`0640`和`0750`。MySQL 假定`UMASK`或`UMASK_DIR`的值以零开头时为八进制。例如，设置`UMASK=0600`等效于`UMASK=384`，因为 0600 八进制为 384 十进制。

`UMASK`和`UMASK_DIR`变量，尽管它们的名称是这样，但实际上是用作模式，而不是掩码：

+   如果设置了`UMASK`，**mysqld**将使用`($UMASK | 0600)`作为文件创建的模式，以便新创建的文件的模式在 0600 到 0666 的范围内（所有值均为八进制）。

+   如果设置了`UMASK_DIR`，**mysqld**将使用`($UMASK_DIR | 0700)`作为目录创建的基本模式，然后与`~(~$UMASK & 0666)`进行 AND 操作，以便新创建的目录的模式在 0700 到 0777 的范围内（所有值均为八进制）。AND 操作可能会从目录模式中删除读取和写入权限，但不会删除执行权限。

另请参阅 Section B.3.3.1, “文件权限问题”。

如果使用**pkg-config**构建 MySQL 程序，则可能需要设置`PKG_CONFIG_PATH`。请参见使用 pkg-config 构建 C API 客户端程序。
