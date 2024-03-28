# 25.5.4 ndb_mgmd — NDB 集群管理服务器守护程序

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-mgmd.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-mgmd.html)

管理服务器是读取集群配置文件并将此信息分发给请求的集群中的所有节点的进程。它还维护着集群活动的日志。管理客户端可以连接到管理服务器并检查集群的状态。

所有可用于**ndb_mgmd**的选项均列在下表中。表后面会有额外的描述。

**表 25.26 与程序 ndb_mgmd 一起使用的命令行选项**

| 格式 | 描述 | 添加、弃用或移除 |
| --- | --- | --- |
| `--bind-address=host` | 本地绑定地址 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--character-sets-dir=path` | 包含字符集的目录 | 移除：8.0.31 |
| `--cluster-config-suffix=name` | 在读取 my.cnf 文件中的 cluster_config 部分时覆盖默认组后缀；用于测试 | 添加：8.0.24 |
| `[--config-cache[=TRUE&#124;FALSE]](mysql-cluster-programs-ndb-mgmd.html#option_ndb_mgmd_config-cache)` | 启用管理服务器配置缓存；默认为 true | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--config-file=file`,`-f file` | 指定集群配置文件；还要指定--reload 或--initial 以覆盖存在的配置缓存 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--configdir=directory`,`--config-dir=directory` | 指定集群管理服务器配置缓存目录 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--connect-retries=#` | 放弃之前重试连接的次数 | 移除：8.0.31 |
| `--connect-retry-delay=#` | 尝试联系管理服务器之间等待的秒数 | 移除：8.0.31 |
| `--connect-string=connection_string`,`-c connection_string` | 与--ndb-connectstring 相同 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--core-file` | 在错误时写入核心文件；用于调试 | 已移除：8.0.31 |
| `--daemon`,`-d` | 以守护进程模式运行 ndb_mgmd（默认） | (在所有基于 MySQL 8.0 的 NDB 版本中支持) |
| `--defaults-extra-file=path` | 在全局文件读取后读取给定文件 | (在所有基于 MySQL 8.0 的 NDB 版本中支持) |
| `--defaults-file=path` | 仅从给定文件中读取默认选项 | (在所有基于 MySQL 8.0 的 NDB 版本中支持) |
| `--defaults-group-suffix=string` | 还读取连接（group，suffix） | (在所有基于 MySQL 8.0 的 NDB 版本中支持) |
| `--help`,`-?` | 显示帮助文本并退出 | (在所有基于 MySQL 8.0 的 NDB 版本中支持) |
| `--initial` | 导致管理服务器从配置文件重新加载配置数据，绕过配置缓存 | (在所有基于 MySQL 8.0 的 NDB 版本中支持) |
| `[--install[=name]](mysql-cluster-programs-ndb-mgmd.html#option_ndb_mgmd_install)` | 用于将管理服务器进程安装为 Windows 服务；不适用于其他平台 | (在所有基于 MySQL 8.0 的 NDB 版本中支持) |
| `--interactive` | 以交互模式运行 ndb_mgmd（在生产中不受官方支持；仅用于测试目的） | (在所有基于 MySQL 8.0 的 NDB 版本中支持) |
| `--log-name=name` | 写入适用于此节点的集群日志消息时使用的名称 | (在所有基于 MySQL 8.0 的 NDB 版本中支持) |
| `--login-path=path` | 从登录文件中读取给定路径 | (在所有基于 MySQL 8.0 的 NDB 版本中支持) |
| `--mycnf` | 从 my.cnf 文件中读取集群配置数据 | (在所有基于 MySQL 8.0 的 NDB 版本中支持) |
| `--ndb-connectstring=connection_string`,`-c connection_string` | 设置连接到 ndb_mgmd 的连接字符串。语法："[nodeid=id;][host=]hostname[:port]"。覆盖 NDB_CONNECTSTRING 和 my.cnf 中的条目 | (在所有基于 MySQL 8.0 的 NDB 版本中支持) |
| `--ndb-mgmd-host=connection_string`,`-c connection_string` | 与 --ndb-connectstring 相同 | (适用于基于 MySQL 8.0 的所有 NDB 版本) |
| `--ndb-nodeid=#` | 为此节点设置节点 ID，覆盖 --ndb-connectstring 设置的任何 ID | (适用于基于 MySQL 8.0 的所有 NDB 版本) |
| `--ndb-optimized-node-selection` | 启用用于事务节点选择的优化。默认启用；使用 --skip-ndb-optimized-node-selection 来禁用 | 已移除：8.0.31 |
| `--no-defaults` | 不从登录文件以外的任何选项文件中读取默认选项 | (适用于基于 MySQL 8.0 的所有 NDB 版本) |
| `--no-nodeid-checks` | 不执行任何节点 ID 检查 | (适用于基于 MySQL 8.0 的所有 NDB 版本) |
| `--nodaemon` | 不以守护进程方式运行 ndb_mgmd | (适用于基于 MySQL 8.0 的所有 NDB 版本) |
| `--nowait-nodes=list` | 在启动此管理服务器时不等待指定的管理节点；需要 --ndb-nodeid 选项 | (适用于基于 MySQL 8.0 的所有 NDB 版本) |
| `--print-defaults` | 打印程序参数列表并退出 | (适用于基于 MySQL 8.0 的所��� NDB 版本) |
| `--print-full-config`,`-P` | 打印完整配置并退出 | (适用于基于 MySQL 8.0 的所有 NDB 版本) |
| `--reload` | 导致管理服务器将配置文件与配置缓存进行比较 | (适用于基于 MySQL 8.0 的所有 NDB 版本) |
| `[--remove[=name]](mysql-cluster-programs-ndb-mgmd.html#option_ndb_mgmd_remove)` | 用于删除先前安装为 Windows 服务的管理服务器进程，可选择指定要删除的服务名称；不适用于其他平台 | (适用于基于 MySQL 8.0 的所有 NDB 版本) |
| `--skip-config-file` | 不使用配置文件 | (适用于基于 MySQL 8.0 的所有 NDB 版本) |
| `--usage`,`-?` | 显示帮助文本并退出；与--help 相同 | （在基于 MySQL 8.0 的所有 NDB 版本中支持） |
| `--verbose`,`-v` | 写入额外信息到日志 | （在基于 MySQL 8.0 的所有 NDB 版本中支持） |
| `--version`,`-V` | 显示版本信息并退出 | （在基于 MySQL 8.0 的所有 NDB 版本中支持） |
| 格式 | 描述 | 添加、弃用或移除 |

+   `--bind-address=*`host`*`

    | 命令行格式 | `--bind-address=host` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `[none]` |

    导致管理服务器绑定到特定网络接口（主机名或 IP 地址）。此选项没有默认值。

+   `--character-sets-dir`

    | 命令行格式 | `--character-sets-dir=path` |
    | --- | --- |
    | 移除版本 | 8.0.31 |

    包含字符集的目录。

+   `cluster-config-suffix`

    | 命令行格式 | `--cluster-config-suffix=name` |
    | --- | --- |
    | 引入版本 | 8.0.24 |
    | 类型 | 字符串 |
    | 默认值 | `[none]` |

    在读取`my.cnf`中的集群配置部分时覆盖默认组后缀；用于测试。

+   `--config-cache`

    | 命令行格式 | `--config-cache[=TRUE&#124;FALSE]` |
    | --- | --- |
    | 类型 | 布尔值 |
    | 默认值 | `TRUE` |

    此选项的默认值为`1`（或`TRUE`，或`ON`），可用于禁用管理服务器的配置缓存，使其每次启动时从`config.ini`读取配置（参见第 25.4.3 节，“NDB 集群配置文件”）。您可以通过以下任一选项启动**ndb_mgmd**进程来实现：

    +   `--config-cache=0`

    +   `--config-cache=FALSE`

    +   `--config-cache=OFF`

    +   `--skip-config-cache`

    仅当管理服务器在启动时没有存储配置时，才能有效使用上述选项之一。如果管理服务器发现任何配置缓存文件，则`--config-cache`选项或`--skip-config-cache`选项将被忽略。因此，要禁用配置缓存，应该在管理服务器第一次启动时使用该选项。否则，即如果您希望为已经创建配置缓存的管理服务器禁用配置缓存，您必须停止管理服务器，手动删除任何现有的配置缓存文件，然后使用`--skip-config-cache`（或将`--config-cache`设置为 0、`OFF`或`FALSE`）重新启动管理服务器。

    配置缓存文件通常在安装目录下的名为`mysql-cluster`的目录中创建（除非使用`--configdir`选项覆盖了此位置）。每次管理服务器更新其配置数据时，都会写入一个新的缓存文件。这些文件按照创建顺序顺序命名，格式如下：

    ```sql
    ndb_*node-id*_config.bin.*seq-number*
    ```

    *`node-id`* 是管理服务器的节点 ID；*`seq-number`* 是一个从 1 开始的序列号。例如，如果管理服务器的节点 ID 是 5，则创建前三个配置缓存文件时，它们的名称将分别为`ndb_5_config.bin.1`、`ndb_5_config.bin.2`和`ndb_5_config.bin.3`。

    如果您的意图是在不实际禁用缓存的情况下清除或重新加载配置缓存，您应该使用`--reload`或`--initial`选项之一，而不是使用`--skip-config-cache`来启动**ndb_mgmd**。

    要重新启用配置缓存，只需重新启动管理服务器，但不使用之前用于禁用配置缓存的`--config-cache`或`--skip-config-cache`选项。

    **ndb_mgmd** 在使用`--skip-config-cache`时不会检查配置目录（`--configdir`），也不会尝试创建配置目录。（Bug #13428853）

+   `--config-file=*`filename`*`，`-f *`filename`*`

    | 命令行格式 | `--config-file=file` |
    | --- | --- |
    | 禁用方式 | `skip-config-file` |
    | 类型 | 文件名 |
    | 默认值 | `[无]` |

    指示管理服务器应使用哪个文件作为其配置文件。默认情况下，管理服务器在与**ndb_mgmd**可执行文件相同目录中查找名为`config.ini`的文件；否则，��须显式指定文件名和位置。

    此选项没有默认值，除非管理服务器被迫读取配置文件，要么因为**ndb_mgmd**是使用`--reload`或`--initial`选项启动的，要么因为管理服务器找不到任何配置缓存。从 NDB 8.0.26 开始，**ndb_mgmd** 如果指定了`--config-file`而没有指定`--initial`或`--reload`，则拒绝启动。

    如果**ndb_mgmd**是使用`--config-cache=OFF`启动的，则还会读取`--config-file`选项。有关更多信息，请参见第 25.4.3 节，“NDB 集群配置文件”。

+   `--configdir=*`dir_name`*`

    | 命令行格式 | `--configdir=directory``--config-dir=directory` |
    | --- | --- |
    | 类型 | 文件名 |
    | 默认值 | `$INSTALLDIR/mysql-cluster` |

    指定集群管理服务器的配置缓存目录。`--config-dir` 是此选项的别名。

    在 NDB 8.0.27 及更高版本中，这必须是绝对路径。否则，管理服务器将拒绝启动。

+   `--connect-retries`

    | 命令行格式 | `--connect-retries=#` |
    | --- | --- |
    | 移除 | 8.0.31 |
    | 类型 | 整数 |
    | 默认值 | `12` |
    | 最小值 | `0` |
    | 最大值 | `12` |

    在放弃之前重试连接的次数。

+   `--connect-retry-delay`

    | 命令行格式 | `--connect-retry-delay=#` |
    | --- | --- |
    | 移除 | 8.0.31 |
    | 类型 | 整数 |
    | 默认值 | `5` |
    | 最小值 | `0` |
    | 最大值 | `5` |

    尝试联系管理服务器之间等待的秒数。

+   `--connect-string`

    | 命令行格式 | `--connect-string=connection_string` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `[none]` |

    与--ndb-connectstring 相同。

+   `--core-file`

    | 命令行格式 | `--core-file` |
    | --- | --- |
    | 已移除 | 8.0.31 |

    在错误时写入核心文件；用于调试。

+   `--daemon`, `-d`

    | 命令行格式 | `--daemon` |
    | --- | --- |

    指示**ndb_mgmd**以守护进程方式启动。这是默认行为。

    在 Windows 平台上运行**ndb_mgmd**时，此选项无效。

+   `--defaults-extra-file`

    | 命令行格式 | `--defaults-extra-file=path` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `[none]` |

    在读取全局文件后读取给定文件。

+   `--defaults-file`

    | 命令行格式 | `--defaults-file=path` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `[none]` |

    仅从给定文件中读取默认选项。

+   `--defaults-group-suffix`

    | 命令行格式 | `--defaults-group-suffix=string` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `[none]` |

    也可以使用 concat(group, suffix) 读取组。

+   `--help`

    | 命令行格式 | `--help` |
    | --- | --- |

    显示帮助文本并退出。

+   `--initial`

    | 命令行格式 | `--initial` |
    | --- | --- |

    配置数据在内部缓存，而不是每次启动管理服务器时从集群全局配置文件中读取（请参阅第 25.4.3 节，“NDB 集群配置文件”）。使用`--initial`选项会覆盖此行为，强制管理服务器删除任何现有的缓存文件，然后从集群配置文件中重新读取配置数据并构建新的缓存。

    这与`--reload`选项有两种不同之处。首先，`--reload`���制服务器检查配置文件与缓存的差异，并仅在文件内容与缓存不同时重新加载数据。其次，`--reload`不会删除任何现有的缓存文件。

    如果使用`--initial`调用**ndb_mgmd**但找不到全局配置文件，则管理服务器无法启动。

    当管理服务器启动时，它会检查同一 NDB Cluster 中是否有另一个管理服务器，并尝试使用另一个管理服务器的配置数据。在执行具有多个管理节点的 NDB Cluster 的滚动重启时，此行为会产生影响。有关更多信息，请参见第 25.6.5 节，“执行 NDB Cluster 的滚动重启”。

    与`--config-file`选项一起使用时，仅在实际找到配置文件时才清除缓存。

+   [`--install[=*`name`*]`](mysql-cluster-programs-ndb-mgmd.html#option_ndb_mgmd_install)

    | 命令行格式 | `--install[=name]` |
    | --- | --- |
    | 平台特定 | Windows |
    | 类型 | 字符串 |
    | 默认值 | `ndb_mgmd` |

    使**ndb_mgmd**作为 Windows 服务安装。可选地，您可以为服务指定名称；如果未设置，则服务名称默认为`ndb_mgmd`。虽然最好在`my.ini`或`my.cnf`配置文件中指定其他**ndb_mgmd**程序选项，但也可以与`--install`一起使用。但是，在这种情况下，必须首先指定`--install`选项，然后再给出任何其他选项，以确保 Windows 服务安装成功。

    通常不建议将此选项与`--initial`选项一起使用，因为这会导致配置缓存在每次停止和启动服务时被清除并重建。如果您打算使用任何其他影响管理服务器启动的**ndb_mgmd**选项，应格外小心，并确保您充分理解并允许可能产生的任何后果。

    `--install` 选项在非 Windows 平台上没有任何效果。

+   `--interactive`

    | 命令行格式 | `--interactive` |
    | --- | --- |

    以交互模式启动**ndb_mgmd**；也就是说，一旦管理服务器运行，就会启动一个**ndb_mgm**客户端会话。此选项不会启动任何其他 NDB Cluster 节点。

+   `--log-name=*`name`*`

    | 命令行格式 | `--log-name=name` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `MgmtSrvr` |

    为此节点在集群日志中使用的名称。

+   `--login-path`

    | 命令行格式 | `--login-path=path` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `[none]` |

    从登录文件中读取给定路径。

+   `--mycnf`

    | 命令行格式 | `--mycnf` |
    | --- | --- |

    从`my.cnf`文件中读取配置数据。

+   `--ndb-connectstring`

    | 命令行格式 | `--ndb-connectstring=connection_string` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `[none]` |

    设���连接字符串。语法：`[nodeid=*`id`*;][host=]*`hostname`*[:*`port`*]`。覆盖`NDB_CONNECTSTRING`和`my.cnf`中的条目。如果指定了`--config-file`，则忽略；从 NDB 8.0.27 开始，当同时使用两个选项时会发出警告。

+   `--ndb-mgmd-host`

    | 命令行格式 | `--ndb-mgmd-host=connection_string` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `[none]` |

    与--ndb-connectstring 相同。

+   `--ndb-nodeid`

    | 命令行格式 | `--ndb-nodeid=#` |
    | --- | --- |
    | 类型 | 整数 |
    | 默认值 | `[none]` |

    设置此节点的节点 ID，覆盖`--ndb-connectstring`设置的任何 ID。

+   `--ndb-optimized-node-selection`

    | 命令行格式 | `--ndb-optimized-node-selection` |
    | --- | --- |
    | 已移除 | 8.0.31 |

    启用优化以选择事务节点。默认启用；使用`--skip-ndb-optimized-node-selection`来禁用。

+   `--no-nodeid-checks`

    | 命令行格式 | `--no-nodeid-checks` |
    | --- | --- |

    不执行任何节点 ID 的检查。

+   `--nodaemon`

    | 命令行格式 | `--nodaemon` |
    | --- | --- |

    指示**ndb_mgmd**不作为守护进程启动。

    在 Windows 上，**ndb_mgmd**的默认行为是在前台运行，因此在 Windows 平台上不需要此选项。

+   `--no-defaults`

    | 命令行格式 | `--no-defaults` |
    | --- | --- |

    不从登录文件以外的任何选项文件中读取默认选项。

+   `--nowait-nodes`

    | 命令行格式 | `--nowait-nodes=list` |
    | --- | --- |
    | 类型 | 数字 |
    | 默认值 | `[none]` |
    | 最小值 | `1` |
    | 最大值 | `255` |

    当启动配置为具有两个管理节点的 NDB Cluster 时，每个管理服务器通常会检查另一个**ndb_mgmd**是否也正在运行，并且另一个管理服务器的配置是否与其相同。但是，有时希望仅使用一个管理节点启动集群（并且可能稍后启动另一个**ndb_mgmd**）。此选项使管理节点绕过传递给此选项的任何其他管理节点的任何检查，允许集群启动，就好像配置为仅使用已启动的管理节点。

    为了举例说明，考虑`config.ini`文件的以下部分（我们省略了与此示例无关的大部分配置参数）：

    ```sql
    [ndbd]
    NodeId = 1
    HostName = 198.51.100.101

    [ndbd]
    NodeId = 2
    HostName = 198.51.100.102

    [ndbd]
    NodeId = 3
    HostName = 198.51.100.103

    [ndbd]
    NodeId = 4
    HostName = 198.51.100.104

    [ndb_mgmd]
    NodeId = 10
    HostName = 198.51.100.150

    [ndb_mgmd]
    NodeId = 11
    HostName = 198.51.100.151

    [api]
    NodeId = 20
    HostName = 198.51.100.200

    [api]
    NodeId = 21
    HostName = 198.51.100.201
    ```

    假设您希望仅使用具有节点 ID `10` 并在具有 IP 地址 198.51.100.150 的主机上运行的管理服务器启动此集群。（例如，假设您打算的主机计算机由于硬件故障暂时不可用，并且您正在等待修复。）要以这种方式启动集群，请在位于 198.51.100.150 的计算机上使用命令行输入以下命令：

    ```sql
    $> ndb_mgmd --ndb-nodeid=10 --nowait-nodes=11
    ```

    如前面的示例所示，当使用`--nowait-nodes`时，您还必须使用`--ndb-nodeid`选项来指定此**ndb_mgmd**进程的节点 ID。

    然后，您可以按照通常的方式启动集群的每个数据节点。如果您希望在稍后的某个时间启动并使用第二个管理服务器而无需重新启动数据节点，则必须使用引用两个管理服务器的连接字符串启动每个数据节点，如下所示：

    ```sql
    $> ndbd -c 198.51.100.150,198.51.100.151
    ```

    与希望作为连接到此集群的 NDB Cluster SQL 节点启动的任何**mysqld**进程使用的连接字符串相同。有关更多信息，请参见 Section 25.4.3.3, “NDB Cluster Connection Strings”。

    当与**ndb_mgmd**一起使用时，此选项仅影响管理节点与其他管理节点的行为。不要将其与与**ndbd**或**ndbmtd**")一起使用的`--nowait-nodes`选项混淆，以允许集群在少于其完整数据节点数量的情况下启动；当与数据节点一起使用时，此选项仅影响它们与其他数据节点的行为。

    可以将多个管理节点 ID 作为逗号分隔的列表传递给此选项。每个节点 ID 必须不小于 1 且不大于 255。实际上，对于同一 NDB 集群使用多个管理服务器（或有任何这样做的需要）是非常罕见的；在大多数情况下，您只需要将不希望在启动集群时使用的单个节点 ID 传递给此选项。

    注意

    当您稍后启动“丢失”的管理服务器时，其配置必须与集群中已经使用的管理服务器的配置相匹配。否则，它将无法通过现有管理服务器执行的配置检查，并且无法启动。

+   `--print-defaults`

    | 命令行格式 | `--print-defaults` |
    | --- | --- |

    打印程序参数列表并退出。

+   `--print-full-config`, `-P`

    | 命令行格式 | `--print-full-config` |
    | --- | --- |

    显示有关集群配置的扩展信息。使用此选项在命令行上，**ndb_mgmd**进程会打印有关集群设置的信息，包括集群配置部分的广泛列表以及参数及其值。通常与`--config-file` (`-f`)选项一起使用。

+   `--reload`

    | 命令行格式 | `--reload` |
    | --- | --- |

    NDB 集群配置数据是存储在内部而不是每次启动管理服务器时从集群全局配置文件中读取的（请参阅第 25.4.3 节，“NDB 集群配置文件”）。使用此选项会强制管理服务器检查其内部数据存储与集群配置文件的匹配情况，并在发现配置文件与缓存不匹配时重新加载配置。现有的配置缓存文件会被保留，但不会被使用。

    这与`--initial`选项有两点不同。首先，`--initial`会导致删除所有缓存文件。其次，`--initial`会强制管理服务器重新读取全局配置文件并构建新的缓存。

    如果管理服务器找不到全局配置文件，则`--reload`选项将被忽略。

    当使用`--reload`时，管理服务器必须能够与数据节点和集群中的任何其他管理服务器通信，然后才能尝试读取全局配置文件；否则，管理服务器将无法启动。这可能是由于网络环境的更改，例如节点的新 IP 地址或防火墙配置的更改。在这种情况下，您必须使用`--initial`来强制丢弃现有的缓存配置并从文件重新加载。有关更多信息，请参见第 25.6.5 节，“执行 NDB 集群的滚动重启”。

+   [`--remove[=name]`](mysql-cluster-programs-ndb-mgmd.html#option_ndb_mgmd_remove)

    | 命令行格式 | `--remove[=name]` |
    | --- | --- |
    | 特定平台 | Windows |
    | 类型 | 字符串 |
    | 默认值 | `ndb_mgmd` |

    删除已安装为 Windows 服务的管理服务器进程，可选择指定要删除的服务名称。仅适用于 Windows 平台。

+   `--skip-config-file`

    | 命令行格式 | `--skip-config-file` |
    | --- | --- |

    不读取集群配置文件；如果指定了`--initial`和`--reload`选项，则忽略。

+   `--usage`

    | 命令行格式 | `--usage` |
    | --- | --- |

    显示帮助文本并退出；与--help 相同。

+   `--verbose`, `-v`

    | 命令行格式 | `--verbose` |
    | --- | --- |

    删除已安装为 Windows 服务的管理服务器进程，可选择指定要删除的服务名称。仅适用于 Windows 平台。

+   `--version`

    | 命令行格式 | `--version` |
    | --- | --- |

    显示版本信息并退出。

在启动管理服务器时，不一定需要指定连接字符串。但是，如果您使用多个管理服务器，则应提供连接字符串，并且集群中的每个节点都应明确指定其节点 ID。

有关使用连接字符串的信息，请参见第 25.4.3.3 节“NDB 集群连接字符串”。第 25.5.4 节“ndb_mgmd — The NDB Cluster Management Server Daemon”描述了**ndb_mgmd**的其他选项。

以下文件由**ndb_mgmd**在其启动目录中创建或使用，并根据 `config.ini` 配置文件中指定的 `DataDir` 放置。在下面的列表中，*`node_id`* 是唯一的节点标识符。

+   `config.ini` 是整个集群的配置文件。此文件由用户创建并由管理服务器读取。第 25.4 节“NDB 集群配置”讨论了如何设置此文件。

+   `ndb_*`node_id`*_cluster.log` 是集群事件日志文件。此类事件的示例包括检查点启动和完成、节点启动事件、节点故障以及内存使用级别。可以在第 25.6 节“NDB 集群管理”中找到带有描述的完整集群事件列表。

    默认情况下，当集群日志大小达到一百万字节时，文件将重命名为 `ndb_*`node_id`*_cluster.log.*`seq_id`*`，其中 *`seq_id`* 是集群日志文件的序列号。（例如：如果已经存在序列号为 1、2 和 3 的文件，则下一个日志文件将使用编号 `4`。）您可以使用 `LogDestination` 配置参数更改集群日志的大小、文件数量和其他特性。

+   `ndb_*`node_id`*_out.log` 是在将管理服务器作为守护进程运行时用于 `stdout` 和 `stderr` 的文件。

+   `ndb_*`node_id`*.pid` 是在将管理服务器作为守护进程运行时使用的进程 ID 文件。
