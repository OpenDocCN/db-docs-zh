# 7.1.7 服务器命令选项

> 原文：[`dev.mysql.com/doc/refman/8.0/en/server-options.html`](https://dev.mysql.com/doc/refman/8.0/en/server-options.html)

当启动 **mysqld** 服务器时，可以使用 Section 6.2.2, “指定程序选项” 中描述的任何方法指定程序选项。最常见的方法是在选项文件或命令行中提供选项。然而，在大多数情况下，最好确保服务器每次运行时使用相同的选项。确保这一点的最佳方法是在选项文件中列出它们。参见 Section 6.2.2.2, “使用选项文件”。该部分还描述了选项文件的格式和语法。

**mysqld** 从 `[mysqld]` 和 `[server]` 组中读取选项。**mysqld_safe** 从 `[mysqld]`, `[server]`, `[mysqld_safe]`, 和 `[safe_mysqld]` 组中读取选项。**mysql.server** 从 `[mysqld]` 和 `[mysql.server]` 组中读取选项。

**mysqld** 接受许多命令选项。要获得简要摘要，请执行此命令：

```sql
mysqld --help
```

要查看完整列表，请使用此命令：

```sql
mysqld --verbose --help
```

列表中的一些项目实际上是可以在服务器启动时设置的系统变量。这些可以使用 `SHOW VARIABLES` 语句在运行时显示。前述 **mysqld** 命令显示的一些项目不会出现在 `SHOW VARIABLES` 输出中；这是因为它们只是选项，而不是系统变量。

以下列表显示了一些最常见的服务器选项。其他选项在其他章节中描述：

+   影响安全性的选项：参见 Section 8.1.4, “与安全相关的 mysqld 选项和变量”。

+   与 SSL 相关的选项：参见 加密连接的命令选项。

+   二进制日志控制选项：参见 Section 7.4.4, “二进制日志”。

+   复制相关选项：参见 Section 19.1.6, “复制和二进制日志选项和变量”。

+   用于加载插件（如可插拔存储引擎）的选项：参见 Section 7.6.1, “安装和卸载插件”。

+   特定于特定存储引擎的选项：参见第 17.14 节，“InnoDB 启动选项和系统变量”和第 18.2.1 节，“MyISAM 启动选项”。

一些选项控制缓冲区或缓存的大小。对于给定的缓冲区，服务器可能需要分配内部数据结构。这些结构通常从分配给缓冲区的总内存中分配，并且所需空间的量可能取决于平台。这意味着当你为控制缓冲区大小的选项分配一个值时，实际可用的空间量可能与分配的值不同。在某些情况下，该量可能小于分配的值。还有可能服务器将值调整向上。例如，如果你为最小值为 1024 的选项分配了一个值为 0，服务器会将该值设置为 1024。

缓冲区大小、长度和堆栈大小的值以字节为单位，除非另有说明。

一些选项接受文件名值。除非另有说明，如果值是相对路径名，则默认文件位置是数据目录。要明确指定位置，请使用绝对路径名。假设数据目录是`/var/mysql/data`。如果文件值选项给定为相对路径名，则位于`/var/mysql/data`下。如果值是绝对路径名，则其位置如路径名所示。

你也可以通过使用变量名作为选项在服务器启动时设置服务器系统变量的值。要为服务器系统变量分配一个值，使用`--*`var_name`*=*`value`*`的形式的选项。例如，`--sort_buffer_size=384M`将`sort_buffer_size`变量设置为 384MB 的值。

当你给一个变量赋值时，MySQL 可能会自动校正该值以保持在给定范围内，或者如果只允许特定值，则调整该值为最接近的可允许值。

为了限制系统变量在运行时可以设置的最大值，可以使用`SET`语句，并通过`--maximum-*`var_name`*=*`value`*`的形式指定最大值。

你可以使用`SET`语句在运行时更改大多数系统变量的值。参见第 15.7.6.1 节，“变量赋值的 SET 语法”。

第 7.1.8 节，“服务器系统变量”，提供了所有变量的完整描述，以及有关在服务器启动和运行时设置它们的附加信息。有关更改系统变量的信息，请参阅 第 7.1.1 节，“配置服务器”。

+   `--help`, `-?`

    | 命令行格式 | `--help` |
    | --- | --- |

    显示简短的帮助消息并退出。同时使用 `--verbose` 和 `--help` 选项以查看完整消息。

+   `--admin-ssl`, `--skip-admin-ssl`

    | 命令行格式 | `--admin-ssl[={OFF&#124;ON}]` |
    | --- | --- |
    | 引入版本 | 8.0.21 |
    | 已弃用 | 8.0.26 |
    | 类型 | 布尔值 |
    | 默认值 | `ON` |

    `--admin-ssl` 选项类似于 `--ssl` 选项，只是它适用于管理连接接口而不是主连接接口。有关这些接口的信息，请参阅 第 7.1.12.1 节，“连接接口”。

    `--admin-ssl` 选项指定服务器允许但不要求在管理接口上进行加密连接。此选项默认启用。

    `--admin-ssl` 可以以否定形式指定为 `--skip-admin-ssl` 或同义词 (`--admin-ssl=OFF`, `--disable-admin-ssl`)。在这种情况下，该选项指定服务器*不*允许加密连接，而不管 `admin_tsl_*`xxx`*` 和 `admin_ssl_*`xxx`*` 系统变量的设置如何。

    `--admin-ssl` 选项仅在服务器启动时对管理接口是否支持加密连接产生影响。在运行时，它被忽略且不会影响 `ALTER INSTANCE RELOAD TLS` 的操作。例如，您可以使用 `--admin-ssl=OFF` 来启动具有禁用加密连接的管理接口，然后重新配置 TLS 并执行 `ALTER INSTANCE RELOAD TLS FOR CHANNEL mysql_admin` 来在运行时启用加密连接。

    关于配置连接加密支持的一般信息，请参阅第 8.3.1 节，“配置 MySQL 使用加密连接”。该讨论是针对主连接接口编写的，但参数名称对于管理连接接口是相似的。考虑在服务器端至少设置`admin_ssl_cert`和`admin_ssl_key`系统变量，以及在客户端端设置`--ssl-ca`（或`--ssl-capath`）选项。有关管理接口的更多信息，请参阅加密连接的管理接口支持。

    因为默认情况下启用了对加密连接的支持，通常不需要指定`--admin-ssl`。从 MySQL 8.0.26 开始，`--admin-ssl`已被弃用，并可能在未来的 MySQL 版本中被移除。如果希望禁用加密连接，可以在不指定`--admin-ssl`的情况下以否定形式进行。将`admin_tls_version`系统变量设置为空值，表示不支持任何 TLS 版本。例如，以下内容在服务器`my.cnf`文件中禁用了加密连接：

    ```sql
    [mysqld]
    admin_tls_version=''
    ```

+   `--allow-suspicious-udfs`

    | 命令行格式 | `--allow-suspicious-udfs[={OFF&#124;ON}]` |
    | --- | --- |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    此选项控制是否可以加载仅具有主函数`xxx`符号的可加载函数。默认情况下，该选项处于关闭状态，只能加载具有至少一个辅助符号的可加载函数；这可以防止尝试从包含合法函数的共享对象文件之外的文件加载函数。请参阅可加载函数安全预防措施。

+   `--ansi`

    | 命令行格式 | `--ansi` |
    | --- | --- |

    使用标准（ANSI）SQL 语法，而不是 MySQL 语法。要对服务器 SQL 模式进行更精确的控制，请使用`--sql-mode`选项。请参阅第 1.6 节，“MySQL 标准兼容性”和第 7.1.11 节，“服务器 SQL 模式”。

+   `--basedir=*`dir_name`*`, `-b *`dir_name`*`

    | 命令行格式 | `--basedir=dir_name` |
    | --- | --- |
    | 系统变量 | `basedir` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 目录名称 |
    | 默认值 | `mysqld 安装目录的父目录` |

    MySQL 安装目录的路径。此选项设置`basedir`系统变量。

    服务器可执行文件在启动时确定自己的完整路径名，并使用其所在目录的父目录作为默认的`basedir`值。这反过来使服务器能够在搜索包含错误消息的`share`目录等与服务器相关信息时使用该`basedir`。

+   `--character-set-client-handshake`

    | 命令行格式 | `--character-set-client-handshake[={OFF&#124;ON}]` |
    | --- | --- |
    | 已弃用 | 8.0.35 |
    | 类型 | 布尔值 |
    | 默认值 | `ON` |

    不要忽略客户端发送的字符集信息。要忽略客户端信息并使用默认服务器字符集，请使用`--skip-character-set-client-handshake`。

    此选项在 MySQL 8.0.35 及更高版本的 MySQL 8.0 发布版中已弃用，每当使用它时都会发出警告，并将在将来的 MySQL 版本中删除。依赖于此选项的应用程序应尽快开始迁移。

+   `--chroot=*`dir_name`*`, `-r *`dir_name`*`

    | 命令行格式 | `--chroot=dir_name` |
    | --- | --- |
    | 类型 | 目录名称 |

    通过使用`chroot()`系统调用，在启动时将**mysqld**服务器置于封闭环境中。这是一项推荐的安全措施。使用此选项会在一定程度上限制`LOAD DATA`和`SELECT ... INTO OUTFILE`。

+   `--console`

    | 命令行格式 | `--console` |
    | --- | --- |
    | 特定平台 | Windows |

    （仅适用于 Windows。）导致默认错误日志目的地为控制台。这会影响那些将自己的输出目的地基于默认目的地的日志接收器。参见第 7.4.2 节，“错误日志”。**mysqld**如果使用此选项，则不会关闭控制台窗口。

    `--console`优先于`--log-error`。

+   `--core-file`

    | 命令行格式 | `--core-file` |
    | --- | --- |

    当使用此选项时，如果**mysqld**崩溃，则会生成一个核心文件；不需要（或接受）任何参数。核心文件的名称和位置取决于系统。在 Linux 上，会在进程的当前工作目录中写入一个名为`core.*`pid`*`的核心文件，对于**mysqld**来说，这是数据目录。*`pid`*代表服务器进程的进程 ID。在 macOS 上，会将名为`core.*`pid`*`的核心文件写入`/cores`目录。在 Solaris 上，使用**coreadm**命令指定核心文件的写入位置和命名方式。

    对于某些系统，要获取核心文件，还必须指定`--core-file-size`选项给**mysqld_safe**。请参见 Section 6.3.2, “mysqld_safe — MySQL Server Startup Script”。在某些系统上，如 Solaris，如果还使用`--user`选项，则不会生成核心文件。可能会有额外的限制或限制。例如，在启动服务器之前可能需要执行**ulimit -c unlimited**。请查阅系统文档。

    `innodb_buffer_pool_in_core_file`变量可用于在支持的操作系统上减小核心文件的大小。更多信息，请参见 Section 17.8.3.7, “Excluding Buffer Pool Pages from Core Files”。

+   `--daemonize`, `-D`

    | 命令行格式 | `--daemonize[={OFF&#124;ON}]` |
    | --- | --- |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    此选项使服务器作为传统的分叉守护程序运行，允许它与使用 systemd 进行进程控制的操作系统一起工作。更多信息，请参见 Section 2.5.9, “Managing MySQL Server with systemd”。

    `--daemonize`与`--initialize`和`--initialize-insecure`是互斥的。

    如果服务器使用`--daemonize`选项启动且未连接到 tty 设备，则在没有显式日志选项的情况下，将使用默认错误日志选项`--log-error=""`，将错误输出定向到默认日志文件。

    `-D`是`--daemonize`的同义词。

+   `--datadir=*`dir_name`*`, `-h *`dir_name`*`

    | 命令行格式 | `--datadir=dir_name` |
    | --- | --- |
    | 系统变量 | `datadir` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 目录名称 |

    MySQL 服务器数据目录的路径。此选项设置了 `datadir` 系统变量。请参阅该变量的描述。

+   [`--debug[=*`debug_options`*]`](server-options.html#option_mysqld_debug), `-# [*`debug_options`*]`

    | 命令行格式 | `--debug[=debug_options]` |
    | --- | --- |
    | 系统变量 | `debug` |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 字符串 |
    | 默认值（Unix） | `d:t:i:o,/tmp/mysqld.trace` |
    | 默认值（Windows） | `d:t:i:O,\mysqld.trace` |

    如果 MySQL 配置了 `-DWITH_DEBUG=1` **CMake** 选项，您可以使用此选项获取 **mysqld** 的跟踪文件，了解其正在执行的操作。典型的 *`debug_options`* 字符串是 `d:t:o,*`file_name`*`。在 Unix 上默认为 `d:t:i:o,/tmp/mysqld.trace`，在 Windows 上为 `d:t:i:O,\mysqld.trace`。

    使用 `-DWITH_DEBUG=1` 配置 MySQL 支持调试功能，使您可以在启动服务器时使用 `--debug="d,parser_debug"` 选项。这会导致用于处理 SQL 语句的 Bison 解析器将解析跟踪转储到服务器的标准错误输出。通常，此输出会写入错误日志。

    此选项可以多次给出。以 `+` 或 `-` 开头的值将添加到或从先前的值中减去。例如，`--debug=T` `--debug=+P` 将值设置为 `P:T`。

    更多信息，请参阅 第 7.9.4 节，“DBUG 包”。

+   [`--debug-sync-timeout[=*`N`*]`](server-options.html#option_mysqld_debug-sync-timeout)

    | 命令行格式 | `--debug-sync-timeout[=#]` |
    | --- | --- |
    | 类型 | 整数 |

    控制是否启用用于测试和调试的 Debug Sync 设施。使用 Debug Sync 需要将 MySQL 配置为具有 `-DWITH_DEBUG=ON` **CMake** 选项（参见 第 2.8.7 节，“MySQL 源配置选项”）；否则，此选项不可用。选项值是以秒为单位的超时时间。默认值为 0，表示禁用 Debug Sync。要启用它，请指定大于 0 的值；此值也成为各个同步点的默认超时时间。如果给出该选项而没有值，则超时设置为 300 秒。

    有关 Debug Sync 设施及如何使用同步点的描述，请参阅 MySQL 内部：测试同步。

+   `--default-time-zone=*`timezone`*`

    | 命令行格式 | `--default-time-zone=name` |
    | --- | --- |
    | 类型 | 字符串 |

    设置默认服务器时区。此选项设置全局 `time_zone` 系统变量。如果未提供此选项，则默认时区与系统时区相同（由 `system_time_zone` 系统变量的值给出）。

    `system_time_zone` 变量与 `time_zone` 变量不同。尽管它们可能具有相同的值，但后者变量用于初始化每个连接的客户端的时区。参见 第 7.1.15 节，“MySQL 服务器时区支持”。

+   `--defaults-extra-file=*`file_name`*`

    在全局选项文件之后但（在 Unix 上）在用户选项文件之前读取此选项文件。如果文件不存在或无法访问，则会发生错误。如果 *`file_name`* 不是绝对路径名，则将其解释为相对于当前目录。如果使用该选项，则必须在命令行上的第一个选项。

    有关此选项文件选项及其他选项文件选项的其他信息，请参见 第 6.2.2.3 节，“影响选项文件处理的命令行选项”。

+   `--defaults-file=*`file_name`*`

    仅读取给定的选项文件。如果文件不存在或无法访问，则会发生错误。如果 *`file_name`* 不是绝对路径名，则将其解释为相对于当前目录。

    例外：即使使用 `--defaults-file`，**mysqld** 也会读取 `mysqld-auto.cnf`。

    注意

    如果使用此选项，则必须将其作为命令行上的第一个选项，除非服务器是使用`--defaults-file`和`--install`（或`--install-manual`）选项启动的，此时`--install`（或`--install-manual`）必须放在第一位。

    有关此选项和其他选项文件选项的更多信息，请参见第 6.2.2.3 节，“影响选项文件处理的命令行选项”。

+   `--defaults-group-suffix=*`str`*`

    读取不仅是通常的选项组，还有带有通常名称和后缀为*`str`*的组。例如，**mysqld**通常读取`[mysqld]`组。如果此选项设置为`--defaults-group-suffix=_other`，**mysqld**还会读取`[mysqld_other]`组。

    有关此选项和其他选项文件选项的更多信息，请参见第 6.2.2.3 节，“影响选项文件处理的命令行选项”。

+   `--early-plugin-load=*`plugin_list`*`

    | 命令行格式 | `--early-plugin-load=plugin_list` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `空字符串` |

    此选项告诉服务器在加载强制内置插件和存储引擎初始化之前加载哪些插件。仅支持使用`PLUGIN_OPT_ALLOW_EARLY`编译的插件进行早期加载。如果给出多个`--early-plugin-load`选项，则只有最后一个适用。

    选项值是以分号分隔的*`plugin_library`*和*`name`*`=`*`plugin_library`*值的列表。每个*`plugin_library`*是包含插件代码的库文件的名称，每个*`name`*是要加载的插件的名称。如果插件库的名称没有任何前置插件名称，服务器将加载库中的所有插件。如果有前置插件名称，服务器将仅从库中加载指定的插件。服务器在由`plugin_dir`系统变量命名的目录中查找插件库文件。

    例如，如果插件`myplug1`和`myplug2`包含在插件库文件`myplug1.so`和`myplug2.so`中，可以使用此选项执行早期插件加载：

    ```sql
    mysqld --early-plugin-load="myplug1=myplug1.so;myplug2=myplug2.so"
    ```

    引号包围参数值，否则某些命令解释器会将分号（`;`）解释为特殊字符。（例如，Unix shell 将其视为命令终止符。）

    每个命名的插件仅在**mysqld**的单次调用中提前加载。重新启动后，除非再次使用`--early-plugin-load`，否则插件不会提前加载。

    如果服务器是使用`--initialize`或`--initialize-insecure`启动的，则不会加载由`--early-plugin-load`指定的插件。

    如果服务器使用`--help`运行，则会加载由`--early-plugin-load`指定的插件，但不会初始化。此行为确保插件选项显示在帮助消息中。

    `InnoDB`表空间加密依赖于 MySQL Keyring 进行加密密钥管理，并且要使用的密钥环插件必须在存储引擎初始化之前加载，以便为加密表的`InnoDB`恢复提供支持。例如，希望在启动时加载`keyring_file`插件的管理员应使用带有适当选项值的`--early-plugin-load`（例如，在 Unix 和类 Unix 系统上使用`keyring_file.so`，在 Windows 上使用`keyring_file.dll`）。

    有关`InnoDB`表空间加密的信息，请参阅第 17.13 节，“InnoDB 数据静态加密”。有关插件加载的一般信息，请参阅第 7.6.1 节，“安装和卸载插件”。

    注意

    对于 MySQL Keyring，此选项仅在密钥库使用密钥环插件进行管理时使用。如果密钥库管理使用密钥环组件而不是插件，请使用清单文件指定组件加载；参见第 8.4.4.2 节，“密钥环组件安装”。

+   [`--exit-info[=*`flags`*]`](server-options.html#option_mysqld_exit-info)，`-T [*`flags`*]`

    | 命令行格式 | `--exit-info[=flags]` |
    | --- | --- |
    | 类型 | 整数 |

    这是一个不同标志的位掩码，您可以用它来调试**mysqld**服务器。除非您*确切地*知道它的作用，否则不要使用此选项！

+   `--external-locking`

    | 命令行格式 | `--external-locking[={OFF&#124;ON}]` |
    | --- | --- |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    启用外部锁定（系统锁定），默认情况下是禁用的。如果在`lockd`无法完全工作的系统上使用此选项（例如 Linux），那么**mysqld**很容易发生死锁。

    要显式禁用外部锁定，请使用`--skip-external-locking`。

    外部锁定仅影响`MyISAM`表访问。有关更多信息，包括可以和不可以使用的条件，请参见 Section 10.11.5, “External Locking”。

+   `--flush`

    | 命令行格式 | `--flush[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `flush` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    在每个 SQL 语句后将所有更改同步到磁盘。通常，MySQL 仅在每个 SQL 语句后将所有更改写入磁盘，并让操作系统处理同步到磁盘。请参见 Section B.3.3.3, “What to Do If MySQL Keeps Crashing”。

    注意

    如果指定了`--flush`，则`flush_time`的值无关紧要，对`flush_time`的更改不会影响刷新行为。

+   `--gdb`

    | 命令行格式 | `--gdb[={OFF&#124;ON}]` |
    | --- | --- |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    为`SIGINT`安装中断处理程序（用于使用`^C`停止**mysqld**设置断点）并禁用堆栈跟踪和核心文件处理。请参见 Section 7.9.1.4, “Debugging mysqld under gdb”。

    在 Windows 上，此选项还抑制了用于实现`RESTART`语句的分叉：分叉使一个进程充当监视器，另一个进程充当服务器。然而，分叉使得确定要附加调试的服务器进程更加困难，因此使用`--gdb`启动服务器会抑制分叉。对于使用此选项启动的服务器，`RESTART`只是简单地退出而不会重新启动。

    在非调试设置中，可以使用`--no-monitor`来抑制监视进程的分叉。

+   `--initialize`, `-I`

    | 命令行格式 | `--initialize[={OFF&#124;ON}]` |
    | --- | --- |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    此选项用于通过创建数据目录并填充`mysql`系统模式中的表来初始化 MySQL 安装。有关更多信息，请参见 Section 2.9.1, “Initializing the Data Directory”。

    此选项限制了 MySQL 服务器的一些其他启动选项的影响，或者与其不兼容。这类问题中最常见的一些问题如下：

    +   我们强烈建议，在使用`--initialize`初始化数据目录时，除了`--datadir`指定的其他选项外，不要指定任何其他选项，其他用于设置目录位置的选项，如`--basedir`，以及可能的`--user`，如果需要的话。在初始化完成并且**mysqld**关闭后，可以在启动 MySQL 服务器时指定运行选项。当使用`--initialize-insecure`代替`--initialize`时，也适用这一规则。

    +   使用`--initialize`启动服务器时，某些功能不可用，限制了任何由`init_file`系统变量命名的文件中允许的语句。有关更多信息，请参阅该变量的描述。此外，`disabled_storage_engines`系统变量无效。

    +   当与`--initialize`一起使用时，`--ndbcluster`选项被忽略。

    +   `--initialize`与`--bootstrap`和`--daemonize`是互斥的。

    上述列表中的项目在使用`--initialize-insecure`选项初始化服务器时也适用。

+   `--initialize-insecure`

    | 命令行格式 | `--initialize-insecure[={OFF&#124;ON}]` |
    | --- | --- |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    此选项用于通过创建数据目录并填充`mysql`系统模式中的表来初始化 MySQL 安装。此选项意味着`--initialize`，并且适用相同的限制和限制；有关更多信息，请参阅该选项的描述，以及第 2.9.1 节，“初始化数据目录”。

    警告

    此选项创建一个具有空密码的 MySQL `root`用户，这是不安全的。因此，在生产环境中不要使用它，除非手动设置此密码。有关如何执行此操作的信息，请参阅初始化后的 root 密码分配。

+   `--innodb-*`xxx`*`

    设置`InnoDB`存储引擎的选项。`InnoDB`选项列在 Section 17.14, “InnoDB Startup Options and System Variables”中。

+   [`--install [*`service_name`*]`](server-options.html#option_mysqld_install)

    | 命令行格式 | `--install [service_name]` |
    | --- | --- |
    | 特定平台 | Windows |

    (仅限 Windows) 将服务器安装为 Windows 服务，并在 Windows 启动时自动启动。如果没有给出*`service_name`*值，则默认服务名称为`MySQL`。有关更多信息，请参见 Section 2.3.4.8, “Starting MySQL as a Windows Service”。

    注意

    如果服务器使用`--defaults-file`和`--install`选项启动，则必须首先使用`--install`。

+   [`--install-manual [*`service_name`*]`](server-options.html#option_mysqld_install-manual)

    | 命令行格式 | `--install-manual [service_name]` |
    | --- | --- |
    | 特定平台 | Windows |

    (仅限 Windows) 将服务器安装为需要手动启动的 Windows 服务。它不会在 Windows 启动时自动启动。如果没有给出*`service_name`*值，则默认服务名称为`MySQL`。有关更多信息，请参见 Section 2.3.4.8, “Starting MySQL as a Windows Service”。

    注意

    如果服务器使用`--defaults-file`和`--install-manual`选项启动，则必须首先使用`--install-manual`。

+   `--language=*`lang_name`*, -L *`lang_name`*`

    | 命令行格式 | `--language=name` |
    | --- | --- |
    | 已弃用 | 是；请改用`lc-messages-dir` |
    | 系统变量 | `language` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 目录名称 |
    | 默认值 | `/usr/local/mysql/share/mysql/english/` |

    用于错误消息的语言。*`lang_name`*可以作为语言名称或作为安装语言文件的目录的完整路径名给出。请参见 Section 12.12, “Setting the Error Message Language”。

    应该使用`--lc-messages-dir`和`--lc-messages`，而不是已被弃用的`--language`（并被视为`--lc-messages-dir`的同义词）。您应该期望`--language`选项在未来的 MySQL 版本中被移除。

+   `--large-pages`

    | 命令行格式 | `--large-pages[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `large_pages` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` Hint Applies | 否 |
    | 平台特定 | Linux |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    一些硬件/操作系统架构支持大于默认值（通常为 4KB）的内存页。此支持的实际实现取决于底层硬件和操作系统。执行大量内存访问的应用程序可能通过使用大页获得性能改进，因为减少了转换查找缓冲区（TLB）的缺失。

    MySQL 支持 Linux 实现的大页支持（在 Linux 中称为 HugeTLB）。请参阅第 10.12.3.3 节，“启用大页支持”。有关 Solaris 对大页的支持，请参阅`--super-large-pages`选项的描述。

    `--large-pages`默认情况下是禁用的。

+   `--lc-messages=*`locale_name`*`

    | 命令行格式 | `--lc-messages=name` |
    | --- | --- |
    | 系统变量 | `lc_messages` |
    | 范围 | 全局, 会话 |
    | 动态 | 是 |
    | `SET_VAR` Hint Applies | 否 |
    | 类型 | 字符串 |
    | 默认值 | `en_US` |

    用于错误消息的区域设置。默认值为`en_US`。服务器将参数转换为语言名称，并将其与`--lc-messages-dir`的值结合起来生成错误消息文件的位置。请参阅第 12.12 节，“设置错误消息语言”。

+   `--lc-messages-dir=*`dir_name`*`

    | 命令行格式 | `--lc-messages-dir=dir_name` |
    | --- | --- |
    | 系统变量 | `lc_messages_dir` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` Hint Applies | 否 |
    | 类型 | 目录名称 |

    存放错误消息的目录。服务器使用该值与`--lc-messages`的值一起生成错误消息文件的位置。参见第 12.12 节，“设置错误消息语言”。

+   `--local-service`

    | 命令行格式 | `--local-service` |
    | --- | --- |

    （仅限 Windows）在服务名称后面跟着`--local-service`选项会导致服务器使用具有有限系统特权的`LocalService` Windows 帐户运行。如果在服务名称后面分别给出`--defaults-file`和`--local-service`，它们可以以任何顺序出现。参见第 2.3.4.8 节，“将 MySQL 作为 Windows 服务启动”。

+   [`--log-error[=*`file_name`*]`](server-options.html#option_mysqld_log-error)

    | 命令行格式 | `--log-error[=file_name]` |
    | --- | --- |
    | 系统变量 | `log_error` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 文件名 |

    将默认的错误日志目的地设置为指定的文件。这会影响那些基于默认目的地的日志输出目的地的日志接收器。参见第 7.4.2 节，“错误日志”。

    如果选项未指定文件，则在 Unix 和类 Unix 系统上，默认的错误日志目的地是数据目录中名为`*`host_name`*.err`的文件。在 Windows 上，默认目的地相同，除非指定了`--pid-file`选项。在这种情况下，文件名是数据目录中带有`.err`后缀的 PID 文件基本名称。

    如果选项指定了一个文件，则默认目的地是该文件（如果名称没有后缀，则添加`.err`后缀），位于数据目录下���除非给出绝对路径名以指定不同的位置。

    如果无法将错误日志输出重定向到错误日志文件，则会发生错误并导致启动失败。

    在 Windows 上，如果同时给出`--console`和`--log-error`，则`--console`优先于`--log-error`。在这种情况下，默认的错误日志目的地是控制台而不是文件。

+   [`--log-isam[=*`file_name`*]`](server-options.html#option_mysqld_log-isam)

    | 命令行格式 | `--log-isam[=file_name]` |
    | --- | --- |
    | 类型 | 文件名 |

    记录所有`MyISAM`更改到此文件中（仅在调试`MyISAM`时使用）。

+   `--log-raw`

    | 命令行格式 | `--log-raw[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量（≥ 8.0.19） | `log_raw` |
    | 范围（≥ 8.0.19） | 全局 |
    | 动态（≥ 8.0.19） | 是 |
    | `SET_VAR` 提示适用（≥ 8.0.19） | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    某些语句中的密码写入到一般查询日志、慢查询日志和二进制日志时，服务器会对密码进行重写，以避免明文出现。可以通过使用`--log-raw`选项来抑制一般查询日志中的密码重写。这个选项可能对诊断目的有用，以查看服务器接收到的语句的确切文本，但出于安全原因，不建议在生产环境中使用。

    如果安装了查询重写插件，则`--log-raw`选项会影响语句记录如下：

    +   没有使用`--log-raw`，服务器记录查询重写插件返回的语句。这可能与接收到的语句不同。

    +   使用`--log-raw`，服务器记录接收到的原始语句。

    更多信息，请参见第 8.1.2.3 节，“密码和日志记录”。

+   `--log-short-format`

    | 命令行格式 | `--log-short-format[={OFF&#124;ON}]` |
    | --- | --- |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    如果已激活，减少慢查询日志中的信息记录。

+   `--log-tc=*`file_name`*`

    | 命令行格式 | `--log-tc=file_name` |
    | --- | --- |
    | 类型 | 文件名 |
    | 默认值 | `tc.log` |

    内存映射事务协调器日志文件的名称（用于在二进制日志禁用时影响多个存储引擎的 XA 事务）。默认名称为`tc.log`。如果未提供完整路径名，则文件将在数据目录下创建。此选项未使用。

+   `--log-tc-size=*`size`*`

    | 命令行格式 | `--log-tc-size=#` |
    | --- | --- |
    | 类型 | 整数 |
    | 默认值 | `6 * 页面大小` |
    | 最小值 | `6 * 页面大小` |
    | 最大值（64 位平台） | `18446744073709551615` |
    | 最大值（32 位平��） | `4294967295` |

    内存映射事务协调器日志的字节大小。默认和最小值为页面大小的 6 倍，且该值必须是页面大小的倍数。

+   `--memlock`

    | 命令行格式 | `--memlock[={OFF&#124;ON}]` |
    | --- | --- |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    将**mysqld**进程锁定在内存中。如果您遇到操作系统导致**mysqld**交换到磁盘的问题，此选项可能有所帮助。

    `--memlock` 适用于支持`mlockall()`系统调用的系统；这包括 Solaris、大多数使用 2.4 或更高内核的 Linux 发行版，以及其他 Unix 系统。在 Linux 系统上，您可以通过检查系统`mman.h`文件中是否定义了`mlockall()`（因此也支持此选项）来判断`mlockall()`是否受支持，如下所示：

    ```sql
    $> grep mlockall /usr/include/sys/mman.h
    ```

    如果支持`mlockall()`，您应该在上一个命令的输出中看到类似以下内容：

    ```sql
    extern int mlockall (int __flags) __THROW;
    ```

    重要

    使用此选项可能需要您以`root`身份运行服务器，出于安全原因，通常不是一个好主意。请参阅第 8.1.5 节，“如何以普通用户身份运行 MySQL”。

    在 Linux 和其他系统上，您可以通过更改`limits.conf`文件来避免以`root`身份运行服务器的需要。请参阅第 10.12.3.3 节，“启用大页支持”中有关`memlock`限制的说明。

    您不应在不支持`mlockall()`系统调用的系统上使用此选项；如果这样做，**mysqld** 在您尝试启动时很可能会立即退出。

+   `--myisam-block-size=*`N`*`

    | 命令行格式 | `--myisam-block-size=#` |
    | --- | --- |
    | 类型 | 整数 |
    | 默认值 | `1024` |
    | 最小值 | `1024` |
    | 最大值 | `16384` |

    用于`MyISAM`索引页的块大小。

+   `--no-defaults`

    不要读取任何选项文件。如果由于从选项文件中读取未知选项而导致程序启动失败，可以使用`--no-defaults`来防止读取这些选项。如果使用此选项，它必须是命令行中的第一个选项。

    有关此选项和其他选项文件选项的更多信息，请参见第 6.2.2.3 节，“影响选项文件处理的命令行选项”。

+   `--no-dd-upgrade`

    | 命令行格式 | `--no-dd-upgrade[={OFF&#124;ON}]` |
    | --- | --- |
    | 弃用 | 8.0.16 |
    | 类型 | 布尔 |
    | 默认值 | `OFF` |

    注意

    该选项自 MySQL 8.0.16 起已被弃用。它被`--upgrade`选项取代，该选项提供了对数据字典和服务器升级行为的更精细控制。

    防止 MySQL 服务器启动过程中自动升级数据字典表。通常在将现有安装升级到更新的 MySQL 版本后启动 MySQL 服务器时使用此选项，其中可能包括对数据字典表定义的更改。

    当指定了`--no-dd-upgrade` 并且服务器发现其期望的数据字典版本与数据字典本身存储的版本不同时，启动将因数据字典升级被禁止而失败；

    ```sql
    [ERROR] [MY-011091] [Server] Data dictionary upgrade prohibited by the
    command line option '--no_dd_upgrade'.
    [ERROR] [MY-010020] [Server] Data Dictionary initialization failed.
    ```

    在正常启动期间，服务器的数据字典版本将与数据字典中存储的版本进行比较，以确定是否应升级数据字典表定义。如果需要升级且支持，服务器将使用更新的定义创建数据字典表，将持久化的元数据复制到新表中，原子性地用新表替换旧表，并重新初始化数据字典。如果不需要升级，则启动将继续而不更新数据字典表。

+   `--no-monitor`

    | 命令行格式 | `--no-monitor[={OFF&#124;ON}]` |
    | --- | --- |
    | 引入版本 | 8.0.12 |
    | 平台特定 | Windows |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    （仅适用于 Windows）。此选项抑制了用于实现`RESTART`语句的分叉：分叉使一个进程充当另一个进程的监视器，后者充当服务器。对于使用此选项启动的服务器，`RESTART`只是退出而不会重新启动。

    `--no-monitor` 在 MySQL 8.0.12 版本之前不可用。可以使用`--gdb`选项作为一种解决方法。

+   `--old-style-user-limits`

    | 命令行格式 | `--old-style-user-limits[={OFF&#124;ON}]` |
    | --- | --- |
    | 已弃用 | 8.0.30 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    启用旧式用户限制。（在 MySQL 5.0.3 之前，帐户资源限制是针对用户连接的每个主机单独计算的，而不是每个`user`表中的帐户行。）请参阅第 8.2.21 节，“设置帐户资源限制”。

    此选项已被弃用，在 MySQL 8.0.30 版本中，如果在命令行或选项文件中使用它，MySQL 将发出警告。预计此选项将在未来的版本中被移除；在此之前，您应该立即检查您的应用程序是否使用了`--old-style-user-limits`，并在此之前删除任何可能依赖它的内容。

+   `--performance-schema-xxx`

    配置性能模式选项。详情请参见第 29.14 节，“性能模式命令选项”。

+   `--plugin-load=*`plugin_list`*`

    | 命令行格式 | `--plugin-load=plugin_list` |
    | --- | --- |
    | 类型 | 字符串 |

    此选项告诉服务器在启动时加载指定的插件。如果给出多个`--plugin-load`选项，则只有最后一个有效。可以使用`--plugin-load-add`选项指定要加载的其他插件。

    选项值是一个以分号分隔的*`plugin_library`*和*`name`*`=`*`plugin_library`*值的列表。每个*`plugin_library`*是包含插件代码的库文件的名称，每个*`name`*是要加载的插件的名称。如果插件库的名称没有任何前置插件名称，服务器将加载库中的所有插件。有了前置插件名称，服务器将仅从库中加载指定的插件。服务器通过`plugin_dir`系统变量指定的目录中查找插件库文件。

    例如，如果插件库文件`myplug1.so`和`myplug2.so`中包含名为`myplug1`和`myplug2`的插件，请使用此选项执行早期插件加载：

    ```sql
    mysqld --plugin-load="myplug1=myplug1.so;myplug2=myplug2.so"
    ```

    引号包围参数值，否则某些命令解释器会将分号（`;`）解释为特殊字符。（例如，Unix shell 将其视为命令终止符。）

    每个命名的插件仅在单次调用**mysqld**时加载。重启后，除非再次使用`--plugin-load`，否则插件不会被加载。这与`INSTALL PLUGIN`相反，后者会向`mysql.plugins`表添加条目，以使插件在每次正常服务器启动时加载。

    在正常启动序列期间，服务器通过读取`mysql.plugins`系统表来确定要加载哪些插件。如果服务器使用`--skip-grant-tables`选项启动，则在`mysql.plugins`表中注册的插件不会被加载且不可用。`--plugin-load`使插件即使在给定`--skip-grant-tables`的情况下也能够加载。`--plugin-load`还使得无法在运行时加载的插件能够在启动时加载。

    此选项不设置相应的系统变量。`SHOW PLUGINS`的输出提供有关已加载插件的信息。更详细的信息可以在信息模式的`PLUGINS`表中找到。参见 Section 7.6.2, “Obtaining Server Plugin Information”。

    有关插件加载的更多信息，请参见第 7.6.1 节，“安装和卸载插件”。

+   `--plugin-load-add=*`plugin_list`*`

    | 命令行格式 | `--plugin-load-add=plugin_list` |
    | --- | --- |
    | 类型 | 字符串 |

    此选项是`--plugin-load`选项的补充。`--plugin-load-add`在启动时向要加载的插件集合中添加一个或多个插件。参数格式与`--plugin-load`相同。`--plugin-load-add`可用于避免将大量插件作为单个冗长的`--plugin-load`参数指定。

    可以在没有`--plugin-load`的情况下给出`--plugin-load-add`，但是在出现在`--plugin-load`之前的任何`--plugin-load-add`实例都不起作用，因为`--plugin-load`会重置要加载的插件集合。换句话说，这些选项：

    ```sql
    --plugin-load=x --plugin-load-add=y
    ```

    等同于此选项：

    ```sql
    --plugin-load="x;y"
    ```

    但这些选项：

    ```sql
    --plugin-load-add=y --plugin-load=x
    ```

    等同于此选项：

    ```sql
    --plugin-load=x
    ```

    此选项不设置相应的系统变量。`SHOW PLUGINS`的输出提供有关已加载插件的信息。更详细的信息可以在信息模式`PLUGINS`表中找到。请参见第 7.6.2 节，“获取服务器插件信息”。

    有关插件加载的更多信息，请参见第 7.6.1 节，“安装和卸载插件”。

+   `--plugin-*`xxx`*`

    指定与服务器插件相关的选项。例如，许多存储引擎可以作为插件构建，对于这些引擎，可以使用`--plugin`前缀指定其选项。因此，`InnoDB`的`--innodb-file-per-table`选项可以指定为`--plugin-innodb-file-per-table`。

    对于可以启用或禁用的布尔选项，还支持`--skip`前缀和其他替代格式（参见第 6.2.2.4 节，“程序选项修饰符”）。例如，`--skip-plugin-innodb-file-per-table`禁用`innodb-file-per-table`。

    使用`--plugin`前缀的理由是，如果与内置服务器选项存在名称冲突，它可以使插件选项得以明确指定。例如，如果插件作者将插件命名为“sql”并实现“mode”选项，则选项名称可能是`--sql-mode`，这将与同名的内置选项冲突。在这种情况下，引用冲突名称将优先解析为内置选项。为避免歧义，用户可以将插件选项指定为`--plugin-sql-mode`。建议为插件选项使用`--plugin`前缀，以避免任何歧义问题。

+   `--port=*`port_num`*`, `-P *`port_num`*`

    | 命令行格式 | `--port=port_num` |
    | --- | --- |
    | 系统变量 | `port` |
    | 作用范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `3306` |
    | 最小值 | `0` |
    | 最大值 | `65535` |

    在监听 TCP/IP 连接时要使用的端口号。在 Unix 和类 Unix 系统上，端口号必须是 1024 或更高，除非服务器由`root`操作系统用户启动。将此选项设置为 0 会使用默认值。

+   `--port-open-timeout=*`num`*`

    | 命令行格式 | `--port-open-timeout=#` |
    | --- | --- |
    | 类型 | 整数 |
    | 默认值 | `0` |

    在某些系统上，当服务器停止时，TCP/IP 端口可能不会立即变为可用。如果服务器随后快速重新启动，则重新打开端口的尝试可能会失败。此选项指示服务器在无法打开端口时等待多少秒，直到 TCP/IP 端口变为空闲。默认情况下不等待。

+   `--print-defaults`

    打印程序名称以及从选项文件中获取的所有选项。密码值被掩盖。如果使用此选项，则必须是命令行中的第一个选项，除非它紧接着`--defaults-file`或`--defaults-extra-file`之后使用。

    有关此选项文件和其他选项文件选项的附加信息，请参见第 6.2.2.3 节，“影响选项文件处理的命令行选项”。

+   [`--remove [*`service_name`*]`](server-options.html#option_mysqld_remove)

    | 命令行格式 | `--remove [service_name]` |
    | --- | --- |
    | 特定平台 | Windows |

    （仅限 Windows）移除 MySQL Windows 服务。如果未提供*`service_name`*值，则默认服务名称为`MySQL`。有关更多信息，请参见第 2.3.4.8 节，“将 MySQL 作为 Windows 服务启动”。

+   `--safe-user-create`

    | 命令行格式 | `--safe-user-create[={OFF&#124;ON}]` |
    | --- | --- |
    | 已弃用 | 是 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    此选项已弃用，并在 MySQL 8.0.11 中被忽略。有关相关信息，请参见服务器更改。

    如果启用此选项，用户无法使用`GRANT`语句创建新的 MySQL 用户，除非用户对`mysql.user`系统表或表中任何列具有`INSERT`权限。如果您希望用户能够创建具有用户有权授予的那些权限的新用户，您应授予用户以下权限：

    ```sql
    GRANT INSERT(user) ON mysql.user TO '*user_name*'@'*host_name*';
    ```

    这确保用户无法直接更改任何权限列，而必须使用`GRANT`语句为其他用户授予权限。

+   `--skip-grant-tables`

    | 命令行格式 | `--skip-grant-tables[={OFF&#124;ON}]` |
    | --- | --- |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    此选项会影响服务器启动顺序：

    +   `--skip-grant-tables`导致服务器不读取`mysql`系统模式中的授权表，因此在启动时根本不使用权限系统。这使得任何访问服务器的人都可以*无限制地访问所有数据库*。

        因为使用`--skip-grant-tables`启动服务器会禁用身份验证检查，因此服务器在这种情况下还会通过启用`skip_networking`来禁用远程连接。

        要使使用`--skip-grant-tables`启动的服务器在运行时加载授权表，请执行特权刷新操作，可以通过以下方式完成：

        +   连接到服务器后，发出 MySQL `FLUSH PRIVILEGES`语句。

        +   从命令行执行 **mysqladmin flush-privileges** 或 **mysqladmin reload** 命令。

        特权刷新也可能隐式发生在启动后执行的其他操作中，从而导致服务器开始使用授权表。例如，如果服务器在启动序列中执行升级，则会刷新权限。

    +   `--skip-grant-tables` 禁用了失败登录跟踪和临时账户锁定，因为这些功能依赖于授权表。参见 第 8.2.15 节，“密码管理”。

    +   `--skip-grant-tables` 导致服务器不加载数据字典或 `mysql` 系统模式中注册的某些���他对象：

        +   使用 `CREATE EVENT` 安装并在 `events` 数据字典表中注册的定时事件。

        +   使用 `INSTALL PLUGIN` 安装并在 `mysql.plugin` 系统表中注册的插件。

            若要在使用 `--skip-grant-tables` 时加载插件，使用 `--plugin-load` 或 `--plugin-load-add` 选项。

        +   使用 `CREATE FUNCTION` 安装并在 `mysql.func` 系统表中注册的可加载函数。

        `--skip-grant-tables` 在启动过程中*不*抑制组件的加载。

    +   `--skip-grant-tables` 导致 `disabled_storage_engines` 系统变量失效。

+   `--skip-host-cache`

    | 命令行格式 | `--skip-host-cache` |
    | --- | --- |
    | 弃用 | 8.0.30 |

    禁用内部主机缓存以加快名称到 IP 地址的解析速度。禁用缓存后，服务器在每次客户端连接时执行 DNS 查找。

    使用 `--skip-host-cache` 类似于将 `host_cache_size` 系统变量设置为 0，但 `host_cache_size` 更灵活，因为它还可以用于在运行时调整、启用或禁用主机缓存，而不仅仅在服务器启动时。

    从 MySQL 8.0.30 开始，此选项已弃用；您应该使用`SET GLOBAL host_cache_size = 0`代替。

    使用`--skip-host-cache`启动服务器不会阻止对`host_cache_size`值的运行时更改，但这些更改不起作用，即使将`host_cache_size`设置为大于 0，缓存也不会重新启用。

    有关主机缓存工作原理的更多信息，请参见第 7.1.12.3 节，“DNS 查找和主机缓存”。

+   `--skip-innodb`

    禁用`InnoDB`存储引擎。在这种情况下，由于默认存储引擎是`InnoDB`，除非您还使用`--default-storage-engine`和`--default-tmp-storage-engine`将默认值设置为其他引擎，否则服务器将无法启动，用于永久表和`TEMPORARY`表。

    无法禁用`InnoDB`存储引擎，`--skip-innodb`选项已弃用且无效。使用该选项会产生警告。预计此选项将在未来的 MySQL 版本中被移除。

+   `--skip-new`

    | 命令行格式 | `--skip-new` |
    | --- | --- |
    | 已弃用 | 8.0.35 |

    此选项禁用（曾被视为）新的、可能不安全的行为。它导致这些设置：`delay_key_write=OFF`、`concurrent_insert=NEVER`、`automatic_sp_privileges=OFF`。它还导致对于不支持`OPTIMIZE TABLE`的存储引擎，将`OPTIMIZE TABLE`映射到`ALTER TABLE`。

    从 MySQL 8.0.35 开始，此选项已弃用，并可能在将来的版本中被移除。

+   `--skip-show-database`

    | 命令行格式 | `--skip-show-database` |
    | --- | --- |
    | 系统变量 | `skip_show_database` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    此选项设置了`skip_show_database`系统变量，该变量控制谁被允许使用`SHOW DATABASES`语句。请参阅第 7.1.8 节，“服务器系统变量”。

+   `--skip-stack-trace`

    | 命令行格式 | `--skip-stack-trace` |
    | --- | --- |

    不要写堆栈跟踪。当您在调试器下运行**mysqld**时，此选项很有用。在某些系统上，您还必须使用此选项才能获得核心文件。请参阅第 7.9 节，“调试 MySQL”。

+   `--slow-start-timeout=*`超时`*`

    | 命令行格式 | `--slow-start-timeout=#` |
    | --- | --- |
    | 类型 | 整数 |
    | 默认值 | `15000` |

    此选项控制 Windows 服务控制管理器的服务启动超时。该值是服务控制管理器在尝试在启动期间终止 Windows 服务之前等待的最大毫秒数。默认值为 15000（15 秒）。如果 MySQL 服务启动时间过长，您可能需要增加此值。值为 0 表示没有超时。

+   `--socket=*`路径`*`

    | 命令行格式 | `--socket={文件名&#124;管道名称}` |
    | --- | --- |
    | 系统变量 | `socket` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 字符串 |
    | 默认值（Windows） | `MySQL` |
    | 默认值（其他） | `/tmp/mysql.sock` |

    在 Unix 上，此选项指定用于监听本地连接时要使用的 Unix 套接字文件。默认值为`/tmp/mysql.sock`。如果给定此选项，服务器将在数据目录中创建文件，除非给定绝对路径名以指定不同的目录。在 Windows 上，该选项指定用于监听使用命名管道的本地连接时要使用的管道名称。默认值为`MySQL`（不区分大小写）。

+   [`--sql-mode=*`值`*[,*`值`*[,*`值`*...]]`](server-options.html#option_mysqld_sql-mode)

    | 命令行格式 | `--sql-mode=name` |
    | --- | --- |
    | 系统变量 | `sql_mode` |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 是 |
    | 类型 | 集合 |
    | 默认值 | `ONLY_FULL_GROUP_BY STRICT_TRANS_TABLES NO_ZERO_IN_DATE NO_ZERO_DATE ERROR_FOR_DIVISION_BY_ZERO NO_ENGINE_SUBSTITUTION` |
    | 有效值 | `ALLOW_INVALID_DATES``ANSI_QUOTES``ERROR_FOR_DIVISION_BY_ZERO``HIGH_NOT_PRECEDENCE``IGNORE_SPACE``NO_AUTO_VALUE_ON_ZERO``NO_BACKSLASH_ESCAPES``NO_DIR_IN_CREATE``NO_ENGINE_SUBSTITUTION``NO_UNSIGNED_SUBTRACTION``NO_ZERO_DATE``NO_ZERO_IN_DATE``ONLY_FULL_GROUP_BY``PAD_CHAR_TO_FULL_LENGTH``PIPES_AS_CONCAT``REAL_AS_FLOAT``STRICT_ALL_TABLES``STRICT_TRANS_TABLES``TIME_TRUNCATE_FRACTIONAL` |

    设置 SQL 模式。请参见 Section 7.1.11, “Server SQL Modes”。

    注意

    MySQL 安装程序可能会在安装过程中配置 SQL 模式。

    如果 SQL 模式与默认设置或您期望的设置不同，请检查服务器在启动时读取的选项文件中的设置。

+   `--ssl`, `--skip-ssl`

    | 命令行格式 | `--ssl[={OFF&#124;ON}]` |
    | --- | --- |
    | 已弃用 | 8.0.26 |
    | 被禁用者 | `skip-ssl` |
    | 类型 | 布尔值 |
    | 默认值 | `ON` |

    `--ssl` 选项指定服务器在主连接接口上允许但不要求加密连接。此选项默认启用。

    一个类似的选项，`--admin-ssl`，类似于 `--ssl`，但适用于管理连接接口而不是主连接接口。有关这些接口的信息，请参见 Section 7.1.12.1, “Connection Interfaces”。

    `--ssl` 可以以否定形式指定为 `--skip-ssl` 或同义词（`--ssl=OFF`, `--disable-ssl`）。在这种情况下，该选项指定服务器*不*允许加密连接，而不管 `tls_*`xxx`*` 和 `ssl_*`xxx`*` 系统变量的设置如何。

    `--ssl` 选项仅在服务器启动时对服务器是否支持加密连接产生影响。在运行时，它被忽略且不会影响 `ALTER INSTANCE RELOAD TLS` 的操作。例如，您可以使用 `--ssl=OFF` 来启动禁用加密连接的服务器，然后重新配置 TLS 并执行 `ALTER INSTANCE RELOAD TLS` 来在运行时启用加密连接。

    有关配置服务器是否允许客户端使用 SSL 连接以及指示 SSL 密钥和证书位置的更多信息，请参见第 8.3.1 节，“配置 MySQL 使用加密连接”，该节还描述了服务器自动生成和自动发现证书和密钥文件的功能。考虑在服务器端至少设置 `ssl_cert` 和 `ssl_key` 系统变量以及客户端端的 `--ssl-ca`（或 `--ssl-capath`）选项。

    因为默认情况下启用了对加密连接的支持，通常不需要指定 `--ssl`。从 MySQL 8.0.26 开始，`--ssl` 已弃用，并可能在将来的 MySQL 版本中删除。如果希望禁用加密连接，可以在否定形式下执行，而无需指定 `--ssl`。将 `tls_version` 系统变量设置为空值，表示不支持任何 TLS 版本。例如，以下内容在服务器的 `my.cnf` 文件中禁用了加密连接：

    ```sql
    [mysqld]
    tls_version=''
    ```

+   `--standalone`

    | 命令行格式 | `--standalone` |
    | --- | --- |
    | 特定平台 | Windows |

    仅在 Windows 上可用；指示 MySQL 服务器不作为服务运行。

+   `--super-large-pages`

    | 命令行格式 | `--super-large-pages[={OFF&#124;ON}]` |
    | --- | --- |
    | 特定平台 | Solaris |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    MySQL 中标准使用大页尝试使用支持的最大大小，最高可达 4MB。在 Solaris 下，“超大页”功能使得可以使用高达 256MB 的页面。此功能适用于最近的 SPARC 平台。可以通过使用 `--super-large-pages` 或 `--skip-super-large-pages` 选项来启用或禁用此功能。

+   `--symbolic-links`, `--skip-symbolic-links`

    | 命令行格式 | `--symbolic-links[={OFF&#124;ON}]` |
    | --- | --- |
    | 已弃用 | 是 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    启用或禁用符号链接支持。在 Unix 上，启用符号链接意味着您可以使用`CREATE TABLE`语句的`INDEX DIRECTORY`或`DATA DIRECTORY`选项将`MyISAM`索引文件或数据文件链接到另一个目录。如果删除或重命名表，其符号链接指向的文件也将被删除或重命名。请参阅第 10.12.2.2 节，“在 Unix 上为 MyISAM 表使用符号链接”。

    注意

    符号链接支持以及控制它的`--symbolic-links`选项已被弃用；您应该期望它在 MySQL 的将来版本中被移除。此外，默认情况下该选项已禁用。相关的`have_symlink`系统变量也已被弃用；您应该期望它在 MySQL 的将来版本中被移除。

    此选项在 Windows 上没有意义。

+   `--sysdate-is-now`

    | 命令行格式 | `--sysdate-is-now[={OFF&#124;ON}]` |
    | --- | --- |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    `SYSDATE()` 默认返回执行时的时间，而不是语句开始执行时的时间。这与`NOW()`的行为不同。此选项使`SYSDATE()`成为`NOW()`的同义词。有关二进制日志和复制的影响，请参阅第 14.7 节，“日期和时间函数”中对`SYSDATE()`的描述以及第 7.1.8 节，“服务器系统变量”中对`SET TIMESTAMP`的描述。

+   `--tc-heuristic-recover={COMMIT|ROLLBACK}`

    | 命令行格式 | `--tc-heuristic-recover=name` |
    | --- | --- |
    | 类型 | 枚举 |
    | 默认值 | `OFF` |
    | 有效数值 | `OFF``COMMIT``ROLLBACK` |

    决定在手动启发式恢复中使用。

    如果指定了`--tc-heuristic-recover`选项，则无论手动启发式恢复是否成功，服务器都会退出。

    在具有多个支持两阶段提交的存储引擎的系统上，`ROLLBACK`选项是不安全的，并导致恢复停止并显示以下错误：

    ```sql
    [ERROR] --tc-heuristic-recover rollback
    strategy is not safe on systems with more than one 2-phase-commit-capable
    storage engine. Aborting crash recovery.
    ```

+   `--transaction-isolation=*`level`*`

    | 命令行格式 | `--transaction-isolation=name` |
    | --- | --- |
    | 系统变量 | `transaction_isolation` |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 枚举 |
    | 默认值 | `REPEATABLE-READ` |
    | 有效数值 | `READ-UNCOMMITTED``READ-COMMITTED``REPEATABLE-READ``SERIALIZABLE` |

    设置默认事务隔离级别。`level`值可以是`READ-UNCOMMITTED`、`READ-COMMITTED`、`REPEATABLE-READ`或`SERIALIZABLE`。请参见第 15.3.7 节，“SET TRANSACTION 语句”。

    默认事务隔离级别也可以通过使用`SET TRANSACTION`语句或设置`transaction_isolation`系统变量在运行时进行设置。

+   `--transaction-read-only`

    | 命令行格式 | `--transaction-read-only[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `transaction_read_only` |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    设置默认事务访问模式。默认情况下，只读模式被禁用，因此模式为读/写。

    要在运行时设置默认事务访问模式，请使用`SET TRANSACTION`语句或设置`transaction_read_only`系统变量。请参见第 15.3.7 节，“SET TRANSACTION 语句”。

+   `--tmpdir=*`dir_name`*`, `-t *`dir_name`*`

    | 命令行格式 | `--tmpdir=dir_name` |
    | --- | --- |
    | 系统变量 | `tmpdir` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 目录名称 |

    用于创建临时文件的目录路径。如果默认的`/tmp`目录位于无法容纳临时表的分区上，则可能会有用。此选项接受几个路径，这些路径以轮询方式使用。在 Unix 上应使用冒号字符（`:`）分隔路径，在 Windows 上应使用分号字符（`;`）分隔路径。

    `--tmpdir` 可以是一个非永久性的位置，比如基于内存的文件系统上的目录，或者是在服务器主机重新启动时清空的目录。如果 MySQL 服务器充当副本，并且您正在使用非永久性位置作为 `--tmpdir`，请考虑使用 `replica_load_tmpdir` 或 `slave_load_tmpdir` 系统变量为副本设置不同的临时目录。对于副本，用于复制 `LOAD DATA` 语句的临时文件存储在此目录中，因此在永久位置下，它们可以在机器重新启动后继续存在，尽管如果临时文件已被删除，则复制可以在重新启动后继续进行。

    有关临时文件存储位置的更多信息，请参阅 Section B.3.3.5, “Where MySQL Stores Temporary Files”。

+   `--upgrade=*`value`*`

    | 命令行格式 | `--upgrade=value` |
    | --- | --- |
    | 引入版本 | 8.0.16 |
    | 类型 | 枚举 |
    | 默认数值 | `AUTO` |
    | 有效数值 | `AUTO``NONE``MINIMAL``FORCE` |

    此选项控制服务器在启动时是否以及如何执行自动升级。自动升级包括两个步骤：

    +   步骤 1：数据字典升级。

        此步骤升级：

        +   `mysql` 模式中的数据字典表。如果实际数据字典版本低于当前预期版本，则服务器将升级数据字典。如果无法升级，或者被阻止这样做，则服务器无法运行。

        +   性能模式和 `INFORMATION_SCHEMA`。

    +   步骤 2：服务器升级。

        此步骤包括所有其他升级任务。如果现有安装数据的 MySQL 版本低于服务器期望的版本，���必须进行升级：

        +   `mysql` 模式中的系统表（剩余的非数据字典表）。

        +   `sys` 模式。

        +   用户模式。

    有关升级步骤 1 和 2 的详细信息，请参阅 Section 3.4, “What the MySQL Upgrade Process Upgrades”。

    这些 `--upgrade` 选项值是允许的：

    +   `AUTO`

        服务器对发现的任何过时内容执行自动升级（步骤 1 和 2）。如果未明确指定 `--upgrade`，则这是默认操作。

    +   `NONE`

        服务器在启动过程中不执行任何自动升级步骤（跳过步骤 1 和 2）。因为此选项值阻止数据字典升级，如果发现数据字典过时，则服务器将以错误退出：

        ```sql
        [ERROR] [MY-013381] [Server] Server shutting down because upgrade is
        required, yet prohibited by the command line option '--upgrade=NONE'.
        [ERROR] [MY-010334] [Server] Failed to initialize DD Storage Engine
        [ERROR] [MY-010020] [Server] Data Dictionary initialization failed.
        ```

    +   `MINIMAL`

        如果需要，服务器升级数据字典、性能模式和`INFORMATION_SCHEMA`（第 1 步）。请注意，使用此选项进行升级后，无法启动组复制，因为复制内部依赖的系统表未更新，并且在其他领域也可能出现功能减少。

    +   `FORCE`

        如果需要，服务器升级数据字典、性能模式和`INFORMATION_SCHEMA`（第 1 步）。此外，服务器强制升级其他所有内容（第 2 步）。由于服务器检查所有模式中的所有对象，使用此选项会导致服务器启动时间较长。

        `FORCE` 用于强制执行第 2 步操作，如果服务器认为这些操作是不必要的。例如，您可能认为系统表丢失或损坏，想要强制修复。

    以下表总结了服务器针对每个选项值采取的操作。

    | 选项值 | 服务器执行第 1 步？ | 服务器执行第 2 步？ |
    | --- | --- | --- |
    | `AUTO` | 如果需要 | 如果需要 |
    | `NONE` | 否 | 否 |
    | `MINIMAL` | 如果需要 | 否 |
    | `FORCE` | 如果需要 | 是 |

+   `--user={*`user_name`*|*`user_id`*}`，`-u {*`user_name`*|*`user_id`*}`

    | 命令行格式 | `--user=name` |
    | --- | --- |
    | 类型 | 字符串 |

    将**mysqld** 服务器作为具有名称*`user_name`*或数字用户 ID*`user_id`*的用户运行。 （此上下文中的“用户”指的是系统登录帐户，而不是授予表中列出的 MySQL 用户。）

    当以`root`身份启动**mysqld**时，此选项是*强制*的。服务器在启动序列期间更改其用户 ID，使其以特定用户身份而不是`root`身份运行。请参阅第 8.1.1 节，“安全准则”。

    为了避免可能的安全漏洞，用户在`my.cnf`文件中添加`--user=root`选项（导致服务器以`root`身份运行），**mysqld** 仅使用指定的第一个`--user`选项，并在存在多个`--user`选项时产生警告。在处理命令行选项之前，`/etc/my.cnf`和`$MYSQL_HOME/my.cnf`中的选项会被处理，因此建议您在`/etc/my.cnf`中放置一个`--user`选项，并指定一个不是`root`的值。在任何其他`--user`选项之前找到`/etc/my.cnf`中的选项，确保服务器以非`root`用户身份运行，并且如果找到任何其他`--user`选项，则会产生警告。

+   `--validate-config`

    | 命令行格式 | `--validate-config[={OFF | ON}]` |
    | --- | --- | --- |
    | 引入版本 | 8.0.16 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    验证服务器启动配置。如果未发现错误，则服务器以退出代码 0 终止。如果发现错误，则服务器显示诊断消息并以退出代码 1 终止。根据`log_error_verbosity`的值，还可能显示警告和信息消息，但不会立即产生验证终止或退出代码 1。有关更多信息，请参见第 7.1.3 节，“服务器配置验证”。

+   [`--validate-user-plugins[={OFF|ON}]`](server-options.html#option_mysqld_validate-user-plugins)

    | 命令行格式 | `--validate-user-plugins[={OFF | ON}]` |
    | --- | --- | --- |
    | 类型 | 布尔值 |
    | 默认值 | `ON` |

    如果启用了此选项（默认情况下），服务器将检查每个用户账户，并在发现会使账户无法使用的条件时产生警告：

    +   账户需要一个未加载的认证插件。

    +   账户需要`sha256_password`或`caching_sha2_password`认证插件，但服务器未启用 SSL 或 RSA，这是插件所需的。 

    启用`--validate-user-plugins`会减慢服务器初始化和`FLUSH PRIVILEGES`。如果不需要额外的检查，您可以在启动时禁用此选项以避免性能下降。

+   `--verbose`, `-v`

    与`--help`选项一起使用此选项以获取详细帮助。

+   `--version`, `-V`

    显示版本信息并退出。
