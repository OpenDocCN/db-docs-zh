> 原文：[`dev.mysql.com/doc/refman/8.0/en/x-plugin-options-system-variables.html`](https://dev.mysql.com/doc/refman/8.0/en/x-plugin-options-system-variables.html)

#### 22.5.6.2 X Plugin 选项和系统变量

使用此选项来控制 X Plugin 的激活：

+   [`--mysqlx[=value]`](x-plugin-options-system-variables.html#option_mysqld_mysqlx)

    | 命令行格式 | `--mysqlx[=value]` |
    | --- | --- |
    | 类型 | 枚举 |
    | 默认值 | `ON` |
    | 有效值 | `ON``OFF``FORCE``FORCE_PLUS_PERMANENT` |

    此选项控制服务器在启动时如何加载 X Plugin。在 MySQL 8.0 中，默认情况下启用 X Plugin，但此选项可用于控制其激活状态。

    选项值应该是插件加载选项中可用的一个，如第 7.6.1 节，“安装和卸载插件”中所述。

如果启用了 X Plugin，它会公开几个系统变量，允许控制其操作：

+   `mysqlx_bind_address`

    | 命令行格式 | `--mysqlx-bind-address=addr` |
    | --- | --- |
    | 系统变量 | `mysqlx_bind_address` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 字符串 |
    | 默认值 | `*` |

    X Plugin 监听 TCP/IP 连接的网络地址。此变量不是动态的，只能在启动时配置。这是`bind_address`系统变量的 X Plugin 等效变量；有关更多信息，请参阅该变量描述。

    默认情况下，X Plugin 接受所有服务器主机 IPv4 接口上的 TCP/IP 连接，并且如果服务器主机支持 IPv6，则接受所有 IPv6 接口上的连接。如果指定了`mysqlx_bind_address`，其值必须满足这些要求：

    +   在 MySQL 8.0.21 之前，`mysqlx_bind_address`接受单个地址值，该值可以指定单个非通配符 IP 地址（IPv4 或 IPv6）、主机名或允许监听多个网络接口的通配符地址格式之一（`*`、`0.0.0.0`或`::`）。

    +   在 MySQL 8.0.21 中，`mysqlx_bind_address`接受单个值或逗号分隔值列表。当变量命名多个值列表时，每个值必须指定单个非通配符 IP 地址（IPv4 或 IPv6）或主机名。通配符地址格式（`*`、`0.0.0.0`或`::`）不允许在值列表中使用。

    +   截至 MySQL 8.0.22，该值可能包括网络命名空间说明符。

    IP 地址可以指定为 IPv4 或 IPv6 地址。对于任何作为主机名的值，X Plugin 将名称解析为 IP 地址并绑定到该地址。如果主机名解析为多个 IP 地址，则 X Plugin 将使用第一个 IPv4 地址（如果有）或否则使用第一个 IPv6 地址。

    X Plugin 对不同类型的地址处理如下：

    +   如果地址是`*`，X Plugin 将在所有服务器主机 IPv4 接口上接受 TCP/IP 连接，并且，如果服务器主机支持 IPv6，则在所有 IPv6 接口上接受连接。使用此地址允许 X Plugin 进行 IPv4 和 IPv6 连接。这是默认值。如果变量指定了多个值的列表，则不允许此值。

    +   如果地址是`0.0.0.0`，X Plugin 将在所有服务器主机 IPv4 接口上接受 TCP/IP 连接。如果变量指定了多个值的列表，则不允许此值。

    +   如果地址是`::`，X Plugin 将在所有服务器主机 IPv4 和 IPv6 接口上接受 TCP/IP 连接。如果变量指定了多个值的列表，则不允许此值。

    +   如果地址是 IPv4 映射地址，则 X Plugin 接受该地址的 TCP/IP 连接，无论是 IPv4 还是 IPv6 格式。例如，如果 X Plugin 绑定到`::ffff:127.0.0.1`，则诸如 MySQL Shell 之类的客户端可以使用`--host=127.0.0.1`或`--host=::ffff:127.0.0.1`进行连接。

    +   如果地址是“常规”IPv4 或 IPv6 地址（例如`127.0.0.1`或`::1`），X Plugin 仅接受针对该 IPv4 或 IPv6 地址的 TCP/IP 连接。

    这些规则适用于为地址指定网络命名空间：

    +   可以为 IP 地址或主机名指定网络命名空间。

    +   无法为通配符 IP 地址指定网络命名空间。

    +   对于给定的地址，网络命名空间是可选的。如果给定，必须将其指定为地址后紧跟着的`/*`ns`*`后缀。

    +   没有`/*`ns`*`后缀的地址使���主机系统全局命名空间。因此，全局命名空间是默认值。

    +   带有`/*`ns`*`后缀的地址使用命名为*`ns`*的命名空间。

    +   主机系统必须支持网络命名空间，并且每个命名空间必须事先设置好。命名不存在的命名空间会产生错误。

    +   如果变量值指定了多个地址，它可以包括全局命名空间中的地址，命名空间中的地址，或者混合使用。

    有关网络命名空间的更多信息，请参见第 7.1.14 节，“网络命名空间支持”。

    重要

    由于 X 插件不是强制性插件，因此如果指定地址或地址列表中存在错误（就像 MySQL 服务器对`bind_address`错误所做的那样），它不会阻止服务器启动。对于 X 插件，如果无法解析列表中的某个地址或 X 插件无法绑定到它，该地址将被跳过，将记录错误消息，并且 X 插件尝试绑定到剩余的每个地址。X 插件的`Mysqlx_address`状态变量仅显示成功绑定的列表中的地址。如果列表中没有任何地址导致成功绑定，或者如果单个指定的地址失败，X 插件将记录错误消息`ER_XPLUGIN_FAILED_TO_PREPARE_IO_INTERFACES`，指出无法使用 X 协议。`mysqlx_bind_address`不是动态的，因此要解决任何问题，您必须停止服务器，更正系统变量值，然后重新启动服务器。

+   `mysqlx_compression_algorithms`

    | 命令行格式 | `--mysqlx-compression-algorithms=value` |
    | --- | --- |
    | 引入版本 | 8.0.19 |
    | 系统变量 | `mysqlx_compression_algorithms` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 集合 |
    | 默认值 | `deflate_stream, lz4_message, zstd_stream` |
    | 有效值 | `deflate_stream``lz4_message``zstd_stream` |

    允许在 X 协议连接上使用的压缩算法。默认情况下，允许使用 Deflate、LZ4 和 zstd 算法。要禁止任何算法，请将`mysqlx_compression_algorithms`设置为仅包含您允许的算法。算法名称`deflate_stream`、`lz4_message`和`zstd_stream`可以以任何组合指定，顺序和大小写不重要。如果将系统变量设置为空字符串，则不允许任何压缩算法，并且仅使用未压缩的连接。使用特定于算法的系统变量来调整每个允许算法的默认和最大压缩级别。有关更多详细信息，以及有关 X 协议连接压缩与 MySQL 服务器等效设置的关系的信息，请参见第 22.5.5 节，“X 插件连接压缩”。

+   `mysqlx_connect_timeout`

    | 命令行格式 | `--mysqlx-connect-timeout=#` |
    | --- | --- |
    | 系统变量 | `mysqlx_connect_timeout` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `30` |
    | 最小值 | `1` |
    | 最大值 | `1000000000` |
    | 单位 | 秒 |

    X 插件等待新连接客户端接收第一个数据包的秒数。这是 X 插件对应的`connect_timeout`；有关更多信息，请参阅该变量描述。

+   `mysqlx_deflate_default_compression_level`

    | 命令行格式 | `--mysqlx_deflate_default_compression_level=#` |
    | --- | --- |
    | 引入版本 | 8.0.20 |
    | 系统变量 | `mysqlx_deflate_default_compression_level` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `3` |
    | 最小值 | `1` |
    | 最大值 | `9` |

    服务器在 X 协议连接上使用的默认 Deflate 算法压缩级别。将级别指定为整数，从 1（最低压缩力度）到 9（最高力度）。如果客户端在能力协商期间未请求压缩级别，则使用此级别。如果您未指定此系统变量，服务器将使用级别 3 作为默认值。有关更多信息，请参阅第 22.5.5 节，“X 插件连接压缩”。

+   `mysqlx_deflate_max_client_compression_level`

    | 命令行格式 | `--mysqlx_deflate_max_client_compression_level=#` |
    | --- | --- |
    | 引入版本 | 8.0.20 |
    | 系统变量 | `mysqlx_deflate_max_client_compression_level` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `5` |
    | 最小值 | `1` |
    | 最大值 | `9` |

    服务器允许在 X 协议连接上使用 Deflate 算法的最大压缩级别。该范围与此算法的默认压缩级别相同。如果客户端请求比此更高的压缩级别，服务器将使用您在此处设置的级别。如果您未指定此系统变量，服务器将设置最大压缩级别为 5。

+   `mysqlx_document_id_unique_prefix`

    | 命令行格式 | `--mysqlx-document-id-unique-prefix=#` |
    | --- | --- |
    | 系统变量 | `mysqlx_document_id_unique_prefix` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `0` |
    | 最小值 | `0` |
    | 最大值 | `65535` |

    设置服务器在将文档添加到集合时生成的文档 ID 的前 4 个字节。通过为每个实例设置此变量的唯一值，您可以确保文档 ID 在实例之间是唯一的。参见理解文档 ID。

+   `mysqlx_enable_hello_notice`

    | 命令行格式 | `--mysqlx-enable-hello-notice[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `mysqlx_enable_hello_notice` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔 |
    | 默认值 | `ON` |

    控制发送给尝试通过 X 协议连接的经典 MySQL 协议客户端的消息。启用时，不支持 X 协议的客户端尝试连接到服务器 X 协议端口的客户端会收到一个错误，解释他们正在使用错误的协议。

+   `mysqlx_idle_worker_thread_timeout`

    | 命令行格式 | `--mysqlx-idle-worker-thread-timeout=#` |
    | --- | --- |
    | 系统变量 | `mysqlx_idle_worker_thread_timeout` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `60` |
    | 最小值 | `0` |
    | 最大值 | `3600` |
    | 单位 | 秒 |

    空闲工作线程终止的秒数。

+   `mysqlx_interactive_timeout`

    | 命令行格式 | `--mysqlx-interactive-timeout=#` |
    | --- | --- |
    | 系统变量 | `mysqlx_interactive_timeout` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提���适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `28800` |
    | 最小值 | `1` |
    | 最大值 | `2147483` |
    | 单位 | 秒 |

    交互客户端的`mysqlx_wait_timeout`会话变量的默认值。（等待交互客户端超时的秒数。）

+   `mysqlx_lz4_default_compression_level`

    | 命令行格式 | `--mysqlx_lz4_default_compression_level=#` |
    | --- | --- |
    | 引入版本 | 8.0.20 |
    | 系统变量 | `mysqlx_lz4_default_compression_level` |
    | 作用域 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `2` |
    | 最小值 | `0` |
    | 最大值 | `16` |

    服务器在 X 协议连接上使用的 LZ4 算法的默认压缩级别。将级别指定为从 0（最低压缩力度）到 16（最高力度）的整数。如果客户端在能力协商期间未请求压缩级别，则使用此级别。如果您没有指定此系统变量，服务器将使用级别 2 作为默认值。有关更多信息，请参见 第 22.5.5 节，“X 插件连接压缩”。

+   `mysqlx_lz4_max_client_compression_level`

    | 命令行格式 | `--mysqlx_lz4_max_client_compression_level=#` |
    | --- | --- |
    | 引入版本 | 8.0.20 |
    | 系统变量 | `mysqlx_lz4_max_client_compression_level` |
    | 作用域 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `8` |
    | 最小值 | `0` |
    | 最大值 | `16` |

    服务器在 X 协议连接上允许的 LZ4 算法的最大压缩级别。该范围与此算法的默认压缩级别相同。如果客户端请求比此更高的压缩级别，服务器将使用您在此处设置的级别。如果您没有指定此系统变量，服务器将设置最大压缩级别为 8。

+   `mysqlx_max_allowed_packet`

    | 命令行格式 | `--mysqlx-max-allowed-packet=#` |
    | --- | --- |
    | 系统变量 | `mysqlx_max_allowed_packet` |
    | 作用域 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `67108864` |
    | 最小值 | `512` |
    | 最大值 | `1073741824` |
    | 单位 | 字节 |

    X 插件可以接收的网络数据包的最大大小。当连接使用压缩时，此限制也适用，因此在消息被解压缩后，网络数据包必须小于此大小。这是 X 插件等同于`max_allowed_packet`的变量；有关更多信息，请参阅该变量描述。

+   `mysqlx_max_connections`

    | 命令行格式 | `--mysqlx-max-connections=#` |
    | --- | --- |
    | 系统变量 | `mysqlx_max_connections` |
    | 作用域 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `100` |
    | 最小值 | `1` |
    | 最大值 | `65535` |

    X 插件可以接受的最大并发客户端连接数。这是 X 插件等同于`max_connections`的变量；有关更多信息，请参阅该变量描述。

    对于此变量的修改，如果新值小于当前连接数，则新限制仅对新连接生效。

+   `mysqlx_min_worker_threads`

    | 命令行格式 | `--mysqlx-min-worker-threads=#` |
    | --- | --- |
    | 系统变量 | `mysqlx_min_worker_threads` |
    | 作用域 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `2` |
    | 最小值 | `1` |
    | 最大值 | `100` |

    X 插件用于处理客户端请求的工作线程的最小数量。

+   `mysqlx_port`

    | 命令行格式 | `--mysqlx-port=port_num` |
    | --- | --- |
    | 系统变量 | `mysqlx_port` |
    | 作用域 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `33060` |
    | 最小值 | `1` |
    | 最大值 | `65535` |

    X 插件用于监听 TCP/IP 连接的网络端口。这是 X 插件等同于`port`的变量；有关更多信息，请参阅该变量描述。

+   `mysqlx_port_open_timeout`

    | 命令行格式 | `--mysqlx-port-open-timeout=#` |
    | --- | --- |
    | 系统变量 | `mysqlx_port_open_timeout` |
    | 作用域 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `0` |
    | 最小值 | `0` |
    | 最大值 | `120` |
    | 单位 | 秒 |

    X 插件等待 TCP/IP 端口空闲的秒数。

+   `mysqlx_read_timeout`

    | 命令行格式 | `--mysqlx-read-timeout=#` |
    | --- | --- |
    | 系统变量 | `mysqlx_read_timeout` |
    | 范围 | 会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `30` |
    | 最小值 | `1` |
    | 最大值 | `2147483` |
    | 单位 | 秒 |

    X 插件等待阻塞读操作完成的秒数。超过此时间后，如果读操作不成功，X 插件将关闭连接并向客户端应用程序返回带有错误代码 ER_IO_READ_ERROR 的警告通知。

+   `mysqlx_socket`

    | 命令行格式 | `--mysqlx-socket=file_name` |
    | --- | --- |
    | 系统变量 | `mysqlx_socket` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 字符串 |
    | 默认值 | `/tmp/mysqlx.sock` |

    X 插件用于连接的 Unix 套接字文件的路径。此设置仅在运行在 Unix 操作系统上的 MySQL 服务器时使用。客户端可以使用此套接字通过 X 插件连接到 MySQL 服务器。

    默认的`mysqlx_socket`路径和文件名基于 MySQL 服务器主套接字文件的默认路径和文件名，文件名后附加了一个 `x`。主套接字文件的默认路径和文件名为 `/tmp/mysql.sock`，因此 X 插件套接字文件的默认路径和文件名为 `/tmp/mysqlx.sock`。

    如果您在服务器启动时使用`socket`系统变量指定了主套接字文件的替代路径和文件名，则这不会影响 X 插件套接字文件的默认值。在这种情况下，如果您希望将两个套接字存储在单个路径上，您还必须设置`mysqlx_socket`系统变量。例如，在配置文件中：

    ```sql
    socket=/home/sockets/mysqld/mysql.sock
    mysqlx_socket=/home/sockets/xplugin/xplugin.sock
    ```

    如果您在编译时使用`MYSQL_UNIX_ADDR`编译选项更改主套接字文件的默认路径和文件名，则会影响 X 插件套接字文件的默认值，该值是通过在`MYSQL_UNIX_ADDR`文件名后附加一个`x`形成的。如果您想在编译时为 X 插件套接字文件设置不同的默认值，请使用`MYSQLX_UNIX_ADDR`编译选项。

    `MYSQLX_UNIX_PORT`环境变量也可用于在服务器启动时设置 X 插件套接字文件的默认值（请参阅第 6.9 节，“环境变量”）。如果设置了此环境变量，则会覆盖编译的`MYSQLX_UNIX_ADDR`值，但会被`mysqlx_socket`值覆盖。

+   `mysqlx_ssl_ca`

    | 命令行格式 | `--mysqlx-ssl-ca=file_name` |
    | --- | --- |
    | 系统变量 | `mysqlx_ssl_ca` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 文件名 |
    | 默认值 | `NULL` |

    `mysqlx_ssl_ca`系统变量类似于`ssl_ca`，不同之处在于它适用于 X 插件而不是 MySQL 服务器的主连接接口。有关为 X 插件配置加密支持的信息，请参见第 22.5.3 节，“使用 X 插件进行加密连接”。

+   `mysqlx_ssl_capath`

    | 命令行格式 | `--mysqlx-ssl-capath=dir_name` |
    | --- | --- |
    | 系统变量 | `mysqlx_ssl_capath` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 目���名 |
    | 默认值 | `NULL` |

    `mysqlx_ssl_capath`系统变量类似于`ssl_capath`，不同之处在于它适用于 X 插件而不是 MySQL 服务器的主连接接口。有关为 X 插件配置加密支持的信息，请参见第 22.5.3 节，“使用 X 插件进行加密连接”。

+   `mysqlx_ssl_cert`

    | 命令行格式 | `--mysqlx-ssl-cert=file_name` |
    | --- | --- |
    | 系统变量 | `mysqlx_ssl_cert` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 文件名 |
    | 默认值 | `NULL` |

    `mysqlx_ssl_cert`系统变量类似于`ssl_cert`，不同之处在于它适用于 X Plugin 而不是 MySQL 服务器的主连接接口。有关为 X Plugin 配置加密���持的信息，请参见第 22.5.3 节，“使用 X Plugin 进行加密连接”。

+   `mysqlx_ssl_cipher`

    | 命令行格式 | `--mysqlx-ssl-cipher=name` |
    | --- | --- |
    | 系统变量 | `mysqlx_ssl_cipher` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 字符串 |
    | 默认值 | `NULL` |

    `mysqlx_ssl_cipher`系统变量类似于`ssl_cipher`，不同之处在于它适用于 X Plugin 而不是 MySQL 服务器的主连接接口。有关为 X Plugin 配置加密支持的信息，请参见第 22.5.3 节，“使用 X Plugin 进行加密连接”。

+   `mysqlx_ssl_crl`

    | 命令行格式 | `--mysqlx-ssl-crl=file_name` |
    | --- | --- |
    | 系统变量 | `mysqlx_ssl_crl` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 文件名 |
    | 默认值 | `NULL` |

    `mysqlx_ssl_crl`系统变量类似于`ssl_crl`，不同之处在于它适用于 X Plugin 而不是 MySQL 服务器的主连接接口。有关为 X Plugin 配置加密支持的信息，请参见第 22.5.3 节，“使用 X Plugin 进行加密连接”。

+   `mysqlx_ssl_crlpath`

    | 命令行格式 | `--mysqlx-ssl-crlpath=dir_name` |
    | --- | --- |
    | 系统变量 | `mysqlx_ssl_crlpath` |
    | 作用范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 目录名称 |
    | 默认值 | `NULL` |

    `mysqlx_ssl_crlpath` 系统变量类似于 `ssl_crlpath`，不同之处在于它适用于 X Plugin 而不是 MySQL 服务器的主连接接口。有关为 X Plugin 配置加密支持的信息，请参见 第 22.5.3 节，“使用 X Plugin 进行加密连接”。

+   `mysqlx_ssl_key`

    | 命令行格式 | `--mysqlx-ssl-key=file_name` |
    | --- | --- |
    | 系统变量 | `mysqlx_ssl_key` |
    | 作用范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 文件名 |
    | 默认值 | `NULL` |

    `mysqlx_ssl_key` 系统变量类似于 `ssl_key`，不同之处在于它适用于 X Plugin 而不是 MySQL 服务器的主连接接口。有关为 X Plugin 配置加密支持的信息，请参见 第 22.5.3 节，“使用 X Plugin 进行加密连接”。

+   `mysqlx_wait_timeout`

    | 命令行格式 | `--mysqlx-wait-timeout=#` |
    | --- | --- |
    | 系统变量 | `mysqlx_wait_timeout` |
    | 作用范围 | 会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `28800` |
    | 最小值 | `1` |
    | 最大值 | `2147483` |
    | 单位 | 秒 |

    X Plugin 等待连接上的活动的秒数。超过此时间后，如果读操作不成功，X Plugin 将关闭连接。如果客户端是非交互式的，则会将会话变量的初始值从全局 `mysqlx_wait_timeout` 变量复制过来。对于交互式客户端，初始值从会话 `mysqlx_interactive_timeout` 复制过来。

+   `mysqlx_write_timeout`

    | 命令行格式 | `--mysqlx-write-timeout=#` |
    | --- | --- |
    | 系统变量 | `mysqlx_write_timeout` |
    | 作用范围 | 会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `60` |
    | 最小值 | `1` |
    | 最大值 | `2147483` |
    | 单位 | 秒 |

    X 插件等待阻塞写操作完成的秒数。超过此时间，如果写操作不成功，X 插件将关闭连接。

+   `mysqlx_zstd_default_compression_level`

    | 命令行格式 | `--mysqlx_zstd_default_compression_level=#` |
    | --- | --- |
    | 引入版本 | 8.0.20 |
    | 系统变量 | `mysqlx_zstd_default_compression_level` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `3` |
    | 最小值 | `-131072` |
    | 最大值 | `22` |

    服务器在 X 协议连接上使用的 zstd 算法的默认压缩级别。对于 zstd 库版本 1.4.0，您可以设置从 1 到 22 的正值（最高压缩力度），或代表逐渐降低力度的负值。值为 0 转换为值为 1。对于较早版本的 zstd 库，您只能指定值为 3。如果客户端在能力协商期间未请求压缩级别，则使用此级别。如果您未指定此系统变量，服务器将使用级别 3 作为默认值。有关更多信息，请参见 第 22.5.5 节，“X 插件连接压缩”。

+   `mysqlx_zstd_max_client_compression_level`

    | 命令行格式 | `--mysqlx_zstd_max_client_compression_level=#` |
    | --- | --- |
    | 引入版本 | 8.0.20 |
    | 系统变量 | `mysqlx_zstd_max_client_compression_level` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `11` |
    | 最小值 | `-131072` |
    | 最大值 | `22` |

    服务器允许在 X 协议连接上使用 zstd 算法的最大压缩级别。范围与此算法的默认压缩级别相同。如果客户端请求比此更高的压缩级别，服务器将使用您在此处设置的级别。如果您未指定此系统变量，服务器将设置最大压缩级别为 11。
