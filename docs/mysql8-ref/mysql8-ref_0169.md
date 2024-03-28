# 6.3.2 mysqld_safe — MySQL 服务器启动脚本

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysqld-safe.html`](https://dev.mysql.com/doc/refman/8.0/en/mysqld-safe.html)

**mysqld_safe** 是在 Unix 上启动 **mysqld** 服务器的推荐方式。**mysqld_safe** 添加了一些安全功能，例如在发生错误时重新启动服务器并将运行时信息记录到错误日志中。本节稍后将介绍错误日志的描述。

注意

对于某些 Linux 平台，从 RPM 或 Debian 软件包安装 MySQL 包括 systemd 支持以管理 MySQL 服务器的启动和关闭。在这些平台上，**mysqld_safe** 不会安装，因为这是不必要的。有关更多信息，请参见 Section 2.5.9, “使用 systemd 管理 MySQL 服务器”。

在使用 systemd 进行服务器管理的平台上不使用 **mysqld_safe** 的一个含义是，选项文件中使用 `[mysqld_safe]` 或 `[safe_mysqld]` 部分不受支持，可能导致意外行为。

**mysqld_safe** 尝试启动名为 **mysqld** 的可执行文件。要覆盖默认行为并明确指定要运行的服务器的名称，请在 **mysqld_safe** 中指定 `--mysqld` 或 `--mysqld-version` 选项。您还可以使用 `--ledir` 指示 **mysqld_safe** 应该查找服务器的目录。

许多 **mysqld_safe** 的选项与 **mysqld** 的选项相同。请参阅 Section 7.1.7, “服务器命令选项”。

对于 **mysqld_safe** 不认识的选项，如果在命令行中指定，则传递给 **mysqld**，但如果在选项文件的 `[mysqld_safe]` 组中指定，则会被忽略。请参阅 Section 6.2.2.2, “使用选项文件”。

**mysqld_safe** 从选项文件的 `[mysqld]`、`[server]` 和 `[mysqld_safe]` 部分读取所有选项。例如，如果您指定了像这样的 `[mysqld]` 部分，**mysqld_safe** 将找到并使用 `--log-error` 选项：

```sql
[mysqld]
log-error=error.log
```

为了向后兼容，**mysqld_safe** 还会读取 `[safe_mysqld]` 部分，但为了保持最新，您应该将这些部分重命名为 `[mysqld_safe]`。

**mysqld_safe** 接受命令行和选项文件中的选项，如下表所述。有关 MySQL 程序使用的选项文件的信息，请参见 6.2.2.2 使用选项文件。

**表 6.7 mysqld_safe 选项**

| 选项名称 | 描述 |
| --- | --- |
| --basedir | MySQL 安装目录的路径 |
| --core-file-size | mysqld 可以创建的核心文件大小 |
| --datadir | 数据目录的路径 |
| --defaults-extra-file | 除了通常的选项文件外，还读取指定的选项文件 |
| --defaults-file | 仅读取指定的选项文件 |
| --help | 显示帮助信息并退出 |
| --ledir | 服务器所在目录的路径 |
| --log-error | 将错误日志写入指定文件 |
| --malloc-lib | 用于 mysqld 的替代 malloc 库 |
| --mysqld | 要启动的服务器程序的名称（在 ledir 目录中） |
| --mysqld-safe-log-timestamps | 用于日志记录的时间戳格式 |
| --mysqld-version | 服务器程序名称的后缀 |
| --nice | 使用 nice 程序设置服务器调度优先级 |
| --no-defaults | 不读取任何选项文件 |
| --open-files-limit | mysqld 可以打开的文件数 |
| --pid-file | 服务器进程 ID 文件的路径名 |
| --plugin-dir | 安装插件的目录 |
| --port | 用于监听 TCP/IP 连接的端口号 |
| --skip-kill-mysqld | 不尝试终止多余的 mysqld 进程 |
| --skip-syslog | 不将错误消息写入 syslog；使用错误日志文件 |
| --socket | 用于监听 Unix 套接字连接的套接字文件 |
| --syslog | 将错误消息写入 syslog |
| --syslog-tag | 写入 syslog 的消息标签后缀 |
| --timezone | 将 TZ 时区环境变量设置为指定值 |
| --user | 以用户名`user_name`或数字用户 ID`user_id`运行 mysqld |
| 选项名称 | 描述 |

+   `--help`

    | 命令行格式 | `--help` |
    | --- | --- |

    显示帮助消息并退出。

+   `--basedir=*`dir_name`*`

    | 命令行格式 | `--basedir=dir_name` |
    | --- | --- |
    | 类型 | 目录名称 |

    MySQL 安装目录的路径。

+   `--core-file-size=*`size`*`

    | 命令行格式 | `--core-file-size=size` |
    | --- | --- |
    | 类型 | 字符串 |

    **mysqld**应该能够创建的核心文件大小。选项值传递给**ulimit -c**。

    注意

    `innodb_buffer_pool_in_core_file`变量可用于减小支持此功能的操作系统上核心文件的大小。有关更多信息，请参见第 17.8.3.7 节，“从核心文件中排除缓冲池页”。

+   `--datadir=*`dir_name`*`

    | 命令行格式 | `--datadir=dir_name` |
    | --- | --- |
    | 类型 | 目录名称 |

    数据目录的路径。

+   `--defaults-extra-file=*`file_name`*`

    | 命令行格式 | `--defaults-extra-file=file_name` |
    | --- | --- |
    | 类型 | 文件名 |

    除了通常的选项文件外，还读取此选项文件。如果文件不存在或无法访问，服务器将以错误退出。如果*`file_name`*不是绝对路径名，则将其解释为相对于当前目录。如果使用此选项，它必须是命令行上的第一个选项。

    有关此选项文件和其他选项文件选项的更多信息，请参见第 6.2.2.3 节，“影响选项文件处理的命令行选项”。

+   `--defaults-file=*`file_name`*`

    | 命令行格式 | `--defaults-file=file_name` |
    | --- | --- |
    | 类型 | 文件名 |

    仅使用给定的选项文件。如果文件不存在或无法访问，服务器将以错误退出。如果*`file_name`*不是绝对路径名，则将其解释为相对于当前目录。如果使用此选项，它必须是命令行上的第一个选项。

    有关此选项和其他选项文件选项的更多信息，请参见第 6.2.2.3 节，“影响选项文件处理的命令行选项”。

+   `--ledir=*`dir_name`*`

    | 命令行格式 | `--ledir=dir_name` |
    | --- | --- |
    | 类型 | 目录名 |

    如果**mysqld_safe**找不到服务器，请使用此选项指示服务器所在目录的路径名。

    此选项仅在命令行上接受，而不在选项文件中。在使用 systemd 的平台上，该值可以在`MYSQLD_OPTS`的值中指定。请参见第 2.5.9 节，“使用 systemd 管理 MySQL 服务器”。

+   `--log-error=*`file_name`*`

    | 命令行格式 | `--log-error=file_name` |
    | --- | --- |
    | 类型 | 文件名 |

    将错误日志写入指定的文件。请参见第 7.4.2 节，“错误日志”。

+   `--mysqld-safe-log-timestamps`

    | 命令行格式 | `--mysqld-safe-log-timestamps=type` |
    | --- | --- |
    | 类型 | 枚举 |
    | 默认值 | `utc` |
    | 有效值 | `system``hyphen``legacy` |

    此选项控制由**mysqld_safe**生成的日志输出中的时间戳格式。以下列表描述了允许的值。对于任何其他值，**mysqld_safe**会记录警告并使用`UTC`格式。

    +   `UTC`, `协调世界时`

        ISO 8601 UTC 格式（与服务器的`--log_timestamps=UTC`相同）。这是默认值。

    +   `SYSTEM`, `系统`

        ISO 8601 本地时间格式（与服务器的`--log_timestamps=SYSTEM`相同）。

    +   `HYPHEN`, `连字符`

        *`YY-MM-DD h:mm:ss`* 格式，如**mysqld_safe**用于 MySQL 5.6。

    +   `LEGACY`, `传统`

        *`YYMMDD hh:mm:ss`* 格式，如**mysqld_safe**在 MySQL 5.6 之前。

+   [`--malloc-lib=[*`lib_name`*]`](mysqld-safe.html#option_mysqld_safe_malloc-lib)

    | 命令行格式 | `--malloc-lib=[lib-name]` |
    | --- | --- |
    | 类型 | 字符串 |

    用于内存分配的库的名称，而不是系统的`malloc()`库。选项值必须是目录之一`/usr/lib`，`/usr/lib64`，`/usr/lib/i386-linux-gnu`或`/usr/lib/x86_64-linux-gnu`。

    `--malloc-lib`选项通过修改`LD_PRELOAD`环境值来影响动态链接，以便加载程序在运行时找到内存分配库：

    +   如果未提供该选项，或者提供的值为空（`--malloc-lib=`)，则不会修改`LD_PRELOAD`，也不会尝试使用`tcmalloc`。

    +   在 MySQL 8.0.21 之前，如果选项给定为`--malloc-lib=tcmalloc`，**mysqld_safe**将在`/usr/lib`中查找`tcmalloc`库。如果找到`tmalloc`，则将其路径名添加到**mysqld**的`LD_PRELOAD`值的开头。如果未找到`tcmalloc`，**mysqld_safe**将因错误而中止。

        截至 MySQL 8.0.21，`tcmalloc`不是`--malloc-lib`选项的允许值。

    +   如果选项给定为`--malloc-lib=*`/path/to/some/library`*`，则该完整路径将添加到`LD_PRELOAD`值的开头。如果完整路径指向不存在或不可读文件，则**mysqld_safe**将因错误而中止。

    +   对于**mysqld_safe**添加路径名到`LD_PRELOAD`的情况，它将路径添加到变量已有值的开头。

    注意

    在使用 systemd 管理服务器的系统上，**mysqld_safe**不可用。相反，通过在`/etc/sysconfig/mysql`中设置`LD_PRELOAD`来指定分配库。

    Linux 用户可以通过将以下行添加到`my.cnf`文件，在任何已安装`tcmalloc`包的`/usr/lib`平台上使用`libtcmalloc_minimal.so`库：

    ```sql
    [mysqld_safe]
    malloc-lib=tcmalloc
    ```

    要使用特定的`tcmalloc`库，请指定其完整路径名。示例：

    ```sql
    [mysqld_safe]
    malloc-lib=/opt/lib/libtcmalloc_minimal.so
    ```

+   `--mysqld=*`prog_name`*`

    | 命令行格式 | `--mysqld=file_name` |
    | --- | --- |
    | 类型 | 文件名 |

    你想要启动的服务器程序的名称（在`ledir`目录中）。如果你使用 MySQL 二进制发行版但数据目录在二进制发行版之外，则需要此选项。如果**mysqld_safe**找不到服务器，请使用`--ledir`选项指示服务器所在目录的路径名。

    此选项仅在命令行上接受，而不在选项文件中接受。在使用 systemd 的��台上，该值可以在`MYSQLD_OPTS`的值中指定。请参阅第 2.5.9 节，“使用 systemd 管理 MySQL 服务器”。

+   `--mysqld-version=*`suffix`*`

    | 命令行格式 | `--mysqld-version=suffix` |
    | --- | --- |
    | 类型 | 字符串 |

    此选项类似于`--mysqld`选项，但您只需指定服务器程序名称的后缀。基本名称假定为**mysqld**。例如，如果您使用`--mysqld-version=debug`，**mysqld_safe**将在`ledir`目录中启动**mysqld-debug**程序。如果`--mysqld-version`的参数为空，**mysqld_safe**将在`ledir`目录中使用**mysqld**。

    此选项仅在命令行上接受，而不在选项文件中接受。在使用 systemd 的平台上，该值可以在`MYSQLD_OPTS`的值中指定。请参阅第 2.5.9 节，“使用 systemd 管理 MySQL 服务器”。

+   `--nice=*`priority`*`

    | 命令行格式 | `--nice=priority` |
    | --- | --- |
    | 类型 | 数字 |

    使用`nice`程序将服务器的调度优先级设置为给定值。

+   `--no-defaults`

    | 命令行格式 | `--no-defaults` |
    | --- | --- |
    | 类型 | 字符串 |

    不要读取任何选项文件。如果由于从选项文件中读取未知选项而导致程序启动失败，则可以使用`--no-defaults`来防止它们被读取。如果使用此选项，它必须是命令行上的第一个选项。

    有关此选项和其他选项文件选项的更多信息，请参见第 6.2.2.3 节，“影响选项文件处理的命令行选项”。

+   `--open-files-limit=*`count`*`

    | 命令行格式 | `--open-files-limit=count` |
    | --- | --- |
    | 类型 | 字符串 |

    **mysqld**应该能够打开的文件数。选项值传递给**ulimit -n**。 

    注意

    你必须以`root`身份启动**mysqld_safe**才能使其正常运行。

+   `--pid-file=*`file_name`*`

    | 命令行格式 | `--pid-file=file_name` |
    | --- | --- |
    | 类型 | 文件名 |

    **mysqld**应该用于其进程 ID 文件的路径名。

+   `--plugin-dir=*`dir_name`*`

    | 命令行格式 | `--plugin-dir=dir_name` |
    | --- | --- |
    | 类型 | 目录名 |

    插件目录的路径名。

+   `--port=*`port_num`*`

    | 命令行格式 | `--port=number` |
    | --- | --- |
    | 类型 | 数字 |

    服务器在监听 TCP/IP 连接时应该使用的端口号。端口号必须是 1024 或更高，除非服务器由`root`操作系统用户启动。

+   `--skip-kill-mysqld`

    | 命令行格式 | `--skip-kill-mysqld` |
    | --- | --- |

    不要在启动时尝试终止无关的**mysqld**进程。此选项仅在 Linux 上有效。

+   `--socket=*`path`*`

    | 命令行格式 | `--socket=file_name` |
    | --- | --- |
    | 类型 | 文件名 |

    服务器在监听本地连接时应该使用的 Unix 套接字文件。

+   `--syslog`, `--skip-syslog`

    | 命令行格�� | `--syslog` |
    | --- | --- |
    | 已弃用 | 是 |
    | 命令行格式 | `--skip-syslog` |
    | 已弃用 | 是 |

    `--syslog`导致错误消息被发送到支持**logger**程序的`syslog`。`--skip-syslog`抑制了`syslog`的使用；消息被写入错误日志文件。

    当使用`syslog`进行错误日志记录时，`daemon.err`设施/严重性用于所有日志消息。

    使用这些选项来控制**mysqld**的日志记录已经被弃用。要将错误日志输出写入系统日志，请使用第 7.4.2.8 节，“将错误日志记录到系统日志”中的说明。要控制设施，请使用服务器`log_syslog_facility`系统变量。

+   `--syslog-tag=*`tag`*`

    | 命令行格式 | `--syslog-tag=tag` |
    | --- | --- |
    | 已弃用 | 是 |

    对于写入`syslog`的日志，来自**mysqld_safe**和**mysqld**的消息分别带有`mysqld_safe`和`mysqld`的标识符。要为这些标识符指定后缀，请使用`--syslog-tag=*`tag`*`，这将修改标识符为`mysqld_safe-*`tag`*`和`mysqld-*`tag`*`。

    使用这个选项来控制**mysqld**的日志记录已被弃用。请使用服务器的`log_syslog_tag`系统变量。参见 Section 7.4.2.8, “Error Logging to the System Log”.

+   `--timezone=*`timezone`*`

    | 命令行格式 | `--timezone=timezone` |
    | --- | --- |
    | 类型 | 字符串 |

    将`TZ`时区环境变量设置为给定的选项值。请查阅您的操作系统文档以获取合法的时区规范格式。

+   `--user={*`user_name`*|*`user_id`*}`

    | 命令行格式 | `--user={user_name&#124;user_id}` |
    | --- | --- |
    | 类型 | 字符串 |
    | 类型 | 数值 |

    以*`user_name`*或数字用户 ID *`user_id`*的名称运行**mysqld**服务器。（这里的“用户”指的是系统登录帐户，而不是在授权表中列出的 MySQL 用户。）

如果你使用`--defaults-file`或`--defaults-extra-file`选项来执行**mysqld_safe**来命名一个选项文件，那么这个选项必须是命令行中给出的第一个选项，否则选项文件将不会被使用。例如，以下命令不会使用指定的选项文件：

```sql
mysql> mysqld_safe --port=*port_num* --defaults-file=*file_name*
```

相反，请使用以下命令：

```sql
mysql> mysqld_safe --defaults-file=*file_name* --port=*port_num*
```

**mysqld_safe**脚本被编写成通常可以启动从 MySQL 源或二进制发行版安装的服务器，即使这些类型的发行版通常会将服务器安装在略有不同的位置。（参见 Section 2.1.5, “Installation Layouts”.）**mysqld_safe**期望以下条件之一为真：

+   服务器和数据库可以相对于工作目录（从中调用**mysqld_safe**的目录）找到。对于二进制发行版，**mysqld_safe**在其工作目录下查找`bin`和`data`目录。对于源代码发行版，它查找`libexec`和`var`目录。如果您从 MySQL 安装目录（例如，对于二进制发行版为`/usr/local/mysql`）执行**mysqld_safe**，则应满足此条件。

+   如果服务器和数据库无法相对于工作目录找到，**mysqld_safe**将尝试通过绝对路径名来定位它们。典型位置是`/usr/local/libexec`和`/usr/local/var`。实际位置是根据构建时配置到发行版中的值确定的。如果 MySQL 安装在配置时指定的位置，则它们应该是正确的。

因为**mysqld_safe**试图相对于自己的工作目录找到服务器和数据库，所以您可以在任何地方安装 MySQL 的二进制发行版，只要您从 MySQL 安装目录运行**mysqld_safe**：

```sql
cd *mysql_installation_directory*
bin/mysqld_safe &
```

如果**mysqld_safe**失败，即使从 MySQL 安装目录调用，也要指定`--ledir`和`--datadir`选项，以指示服务器和数据库在系统中的位置。

**mysqld_safe**尝试使用**sleep**和**date**系统实用程序来确定每秒尝试启动的次数。如果这些实用程序存在，并且每秒尝试启动次数大于 5，**mysqld_safe**在再次启动之前等待 1 秒。这旨在防止在重复失败的情况下出现过多的 CPU 使用率。（Bug #11761530，Bug #54035）

当您使用**mysqld_safe**启动**mysqld**时，**mysqld_safe**会安排来自自身和**mysqld**的错误（和通知）消息发送到相同的目的地。

有几个用于控制这些消息目的地的**mysqld_safe**选项：

+   `--log-error=*`file_name`*`: 将错误消息写入指定的错误文件。

+   `--syslog`: 将错误消息写入支持**logger**程序的`syslog`系统。

+   `--skip-syslog`: 不将错误消息写入`syslog`。消息将被写入默认的错误日志文件（数据目录中的`*`host_name`*.err`），或者如果给定`--log-error`选项，则写入指定的文件。

如果没有给出这些选项中的任何一个，则默认为`--skip-syslog`。

当**mysqld_safe**写入消息时，通知会发送到日志目的地（`syslog`或错误日志文件）和`stdout`。错误会发送到日志目的地和`stderr`。

注意

从**mysqld_safe**控制**mysqld**日志记录已被弃用。请改用服务器的本机`syslog`支持。有关更多信息，请参见第 7.4.2.8 节，“将错误日志记录到系统日志”。
