# 25.5.1 ndbd — NDB 集群数据节点守护进程

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndbd.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndbd.html)

**ndbd** 是用于处理使用 NDB Cluster 存储引擎的表中所有数据的进程。这是使数据节点能够完成分布式事务处理、节点恢复、写入磁盘的检查点、在线备份以及相关任务的进程。

在 NDB 集群中，一组 **ndbd** 进程协作处理数据。这些进程可以在同一台计算机（主机）上执行，也可以在不同计算机上执行。数据节点和集群主机之间的对应关系是完全可配置的。

可用于 **ndbd** 的选项显示在下表中。表后面会有额外的描述。

**表 25.24 与程序 ndbd 一起使用的命令行选项**

| 格式 | 描述 | 添加、弃用或移除 |
| --- | --- | --- |
| `--bind-address=name` | 本地绑定地址 | (适用于基于 MySQL 8.0 的所有 NDB 版本) |
| `--character-sets-dir=path` | 包含字符集的目录 | (适用于基于 MySQL 8.0 的所有 NDB 版本) |
| `--connect-delay=#` | --connect-retry-delay 的已弃用同义词，应使用该选项代替 | 移除：NDB 8.0.28 |
| `--connect-retries=#` | 设置在放弃之前重试连接的次数；0 表示仅尝试 1 次（无重试）；-1 表示持续无限重试 | (适用于基于 MySQL 8.0 的所有 NDB 版本) |
| `--connect-retry-delay=#` | 尝试联系管理服务器之间的等待时间，单位为秒；0 表示尝试之间不等待 | (适用于基于 MySQL 8.0 的所有 NDB 版本) |
| `--connect-string=connection_string`,`-c connection_string` | 与 --ndb-connectstring 相同 | (适用于基于 MySQL 8.0 的所有 NDB 版本) |
| `--core-file` | 在错误时写入核心文件；用于调试 | (适用于基于 MySQL 8.0 的所有 NDB 版本) |
| `--daemon`,`-d` | 以守护进程模式启动 ndbd（默认）；可使用 --nodaemon 参数覆盖 | (适用于基于 MySQL 8.0 的所有 NDB 版本) |
| `--defaults-extra-file=path` | 在全局文件读取后读取给定文件 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--defaults-file=path` | 仅从给定文件中读取默认选项 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--defaults-group-suffix=string` | 也读取连接(concat(group, suffix))的组 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--filesystem-password=password` | 节点文件系统加密的密码；可以从标准输入、终端或 my.cnf 文件中传递 | 新增：NDB 8.0.31 |
| `--filesystem-password-from-stdin={TRUE&#124;FALSE}` | 从标准输入获取节点文件系统加密的密码 | 新增：NDB 8.0.31 |
| `--foreground` | 在前台运行 ndbd，用于调试目的（意味着--nodaemon） | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--help`,`-?` | 显示帮助文本并退出 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--initial` | 执行 ndbd 的初始启动，包括文件系统清理；在使用此选项之前请参考文档 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--initial-start` | 执行部分初始启动（需要--nowait-nodes） | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `[--install[=name]](mysql-cluster-programs-ndbd.html#option_ndbd_install)` | 用于将数据节点进程安装为 Windows 服务；不适用于其他平台 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--logbuffer-size=#` | 控制日志缓冲区的大小；用于在生成许多日志消息时进行调试；默认值对于正常操作足够 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--login-path=path` | 从登录文件中读取给定路径 | (在基于 MySQL 8.0 的所有 NDB 版本���支持) |
| `--ndb-connectstring=connection_string`,`-c connection_string` | 设置连接到 ndb_mgmd 的连接字符串。语法："[nodeid=id;][host=]hostname[:port]"。覆盖 NDB_CONNECTSTRING 和 my.cnf 中的条目 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--ndb-mgmd-host=connection_string`,`-c connection_string` | 与--ndb-connectstring 相同 | (适用于基于 MySQL 8.0 的所有 NDB 版本) |
| `--ndb-nodeid=#` | 设置此节点的节点 ID，覆盖`--ndb-connectstring`设置的任何 ID | (适用于基于 MySQL 8.0 的所有 NDB 版本) |
| `--nodaemon` | 不将 ndbd 作为守护进程启动；供测试目的使用 | (适用于基于 MySQL 8.0 的所有 NDB 版本) |
| `--no-defaults` | 不从登录文件以外的任何选项文件中读取默认选项 | (适用于基于 MySQL 8.0 的所有 NDB 版本) |
| `--nostart`,`-n` | 不立即启动 ndbd；ndbd 等待从 ndb_mgm 启动的命令 | (适用于基于 MySQL 8.0 的所有 NDB 版本) |
| `--nowait-nodes=list` | 不等待这些数据节点启动（以逗号分隔的节点 ID 列表）；需要--ndb-nodeid | (适用于基于 MySQL 8.0 的所有 NDB 版本) |
| `--ndb-optimized-node-selection` | 启用用于事务节点选择的优化。默认启用；使用--skip-ndb-optimized-node-selection 来禁用 | 已移除：8.0.31 |
| `--print-defaults` | 打印程序参数列表并退出 | (适用于基于 MySQL 8.0 的所有 NDB 版本) |
| `[--remove[=name]](mysql-cluster-programs-ndbd.html#option_ndbd_remove)` | 用于删除先前安装为 Windows 服务的数据节点进程；不适用于其他平台 | (适用于基于 MySQL 8.0 的所有 NDB 版本) |
| `--usage`,`-?` | 显示帮助文本并退出；与--help 相同 | (适用于基于 MySQL 8.0 的所有 NDB 版本) |
| `--verbose`,`-v` | 将额外的调试信息写入节点日志 | (适用于基于 MySQL 8.0 的所有 NDB 版本) |
| `--version`,`-V` | 显示版本信息并退出 | (适用于基于 MySQL 8.0 的所有 NDB 版本) |
| 格式 | 描述 | 添加、弃用或移除 |

注意

所有这些选项也适用于此程序的多线程版本（**ndbmtd**")），您可以在本节中出现的“**ndbd**”处用“**ndbmtd**")”替换。

+   `--bind-address`

    | 命令行格式 | `--bind-address=name` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 |  |

    导致 **ndbd** 绑定到特定的网络接口（主机名或 IP 地址）。此选项没有默认值。

+   `--character-sets-dir`

    | 命令行格式 | `--character-sets-dir=path` |
    | --- | --- |

    包含字符集的目录。

+   `--connect-delay=*`#`*`

    | 命令行格式 | `--connect-delay=#` |
    | --- | --- |
    | 已弃用 | 是（在 8.0.28-ndb-8.0.28 中移除） |
    | 类型 | 数值 |
    | 默认值 | `5` |
    | 最小值 | `0` |
    | 最大值 | `3600` |

    确定在启动时尝试联系管理服务器之间等待的时间（尝试次数由 `--connect-retries` 选项控制）。默认值为 5 秒。

    此选项已弃用，并可能在将来的 NDB Cluster 版本中删除。请改用 `--connect-retry-delay`。

+   `--connect-retries=*`#`*`

    | 命令行格式 | `--connect-retries=#` |
    | --- | --- |
    | 类型 | 数值 |
    | 默认值 | `12` |
    | 最小值（≥ 8.0.28-ndb-8.0.28） | `-1` |
    | 最小值 | `-1` |
    | 最小值 | `-1` |
    | 最小值（≤ 8.0.27-ndb-8.0.27） | `0` |
    | 最大值 | `65535` |

    设置在放弃之前重试连接的次数；0 表示仅 1 次尝试（无重试）。默认值为 12 次尝试。尝试之间的等待时间由 `--connect-retry-delay` 选项控制。

    从 NDB 8.0.28 开始，您可以将此选项设置为 -1，在这种情况下，数据节点进程将持续不断地尝试连接。

+   `--connect-retry-delay=*`#`*`

    | 命令行格式 | `--connect-retry-delay=#` |
    | --- | --- |
    | 类型 | 数值 |
    | 默认值 | `5` |
    | 最小值 | `0` |
    | 最大值 | `4294967295` |

    确定在启动时尝试联系管理服务器之间等待的时间（尝试之间的时间由`--connect-retries`选项控制）。默认值为 5 秒。

    此选项取代了`--connect-delay`选项，该选项现已弃用并可能在将来的 NDB Cluster 版本中删除。

    此选项的短形式`-r`自 NDB 8.0.28 起已弃用，并可能在将来的 NDB Cluster 版本中删除。请改用长形式。

+   `--connect-string`

    | 命令行格式 | `--connect-string=connection_string` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `[none]` |

    与`--ndb-connectstring`相同。

+   `--core-file`

    | 命令行格式 | `--core-file` |
    | --- | --- |

    在错误时写入核心文件；用于调试。

+   `--daemon`, `-d`

    | 命令行格式 | `--daemon` |
    | --- | --- |

    指示**ndbd**或**ndbmtd**")作为守护进程执行。这是默认行为。`--nodaemon`可用于阻止进程作为守护进程运行。

    在 Windows 平台上运行**ndbd**或**ndbmtd**")时，此选项无效。

+   `--defaults-extra-file`

    | 命令行格式 | `--defaults-extra-file=path` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `[none]` |

    在全局文件读取后读取给定文件。

+   `--defaults-file`

    | 命令行格式 | `--defaults-file=path` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `[none]` |

    仅从给定文件读取默认选项。

+   `--defaults-group-suffix`

    | 命令行格式 | `--defaults-group-suffix=string` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `[none]` |

    还读取带有 concat(group, suffix)的组。

+   `--filesystem-password`

    | 命令行格式 | `--filesystem-password=password` |
    | --- | --- |
    | 引入 | 8.0.31-ndb-8.0.31 |

    通过`stdin`、`tty`或`my.cnf`文件将文件系统加密和解密密码传递给数据节点进程。

    需要`EncryptedFileSystem = 1`。

    更多信息，请参见第 25.6.14 节，“NDB 集群的文件系统加密”。

+   `--filesystem-password-from-stdin`

    | 命令行格式 | `--filesystem-password-from-stdin={TRUE&#124;FALSE}` |
    | --- | --- |
    | 引入版本 | 8.0.31-ndb-8.0.31 |

    将文件系统加密和解密密码传递给数据节点进程，仅从`stdin`。

    需要`EncryptedFileSystem = 1`。

    更多信息，请参见第 25.6.14 节，“NDB 集群的文件系统加密”。

+   `--foreground`

    | 命令行格式 | `--foreground` |
    | --- | --- |

    导致**ndbd**或**ndbmtd**")作为前台进程执行，主要用于调试目的。此选项意味着`--nodaemon`选项。

    在 Windows 平台上运行**ndbd**或**ndbmtd**")时，此选项不起作用。

+   `--help`

    | 命令行格式 | `--help` |
    | --- | --- |

    显示帮助文本并退出。

+   `--initial`

    | 命令行格式 | `--initial` |
    | --- | --- |

    指示**ndbd**执行初始启动。初始启动会擦除早期实例的恢复目的创建的任何文件，并重新创建恢复日志文件。在某些操作系统上，此过程可能需要相当长的时间。

    只有在非常特殊的情况下才应使用`--initial`启动，因为此选项会导致从 NDB 集群文件系统中删除所有文件并重新创建所有重做日志文件。这些情况列在这里：

    +   在执行更改了任何文件内容的软件升级时。

    +   在使用新版本的**ndbd**重新启动节点时。

    +   作为最后一招的措施，当由于某种原因节点重新启动或系统重启反复失败时。在这种情况下，请注意由于数据文件的破坏，此节点将不再能用于恢复数据。

    警告

    为避免最终可能发生的数据丢失，建议您*不要*将`--initial`选项与`StopOnError = 0`一起使用。相反，在集群启动后，只需在`config.ini`中将`StopOnError`设置为 0，然后正常重新启动数据节点，即不使用`--initial`选项。有关此问题的详细说明，请参阅`StopOnError`参数的描述。（Bug #24945638）

    使用此选项会阻止`StartPartialTimeout`和`StartPartitionedTimeout`配置参数产生任何效果。

    重要

    此选项*不会*影响已由受影响节点创建的备份文件。

    在 NDB 8.0.21 之前，`--initial`选项也不会影响任何磁盘数据文件。在 NDB 8.0.21 及更高版本中，当用于执行集群的初始重新启动时，该选项会导致删除与磁盘数据表空间相关的所有数据文件以及与先前存在于此数据节点上的日志文件组相关联的撤销日志文件（请参阅第 25.6.11 节，“NDB 集群磁盘数据表”）。

    此选项还不会影响数据节点在从已经运行的数据节点（除非它们也是作为初始重新启动的一部分而使用`--initial`启动）中自动恢复数据。在正常运行的 NDB 集群中，此数据恢复会自动发生，不需要用户干预。

    在第一次启动集群时（即在创建任何数据节点文件之前），可以使用此选项；但是，这*不是*必需的。

+   `--initial-start`

    | 命令行格式 | `--initial-start` |
    | --- | --- |

    此选项用于执行集群的部分初始启动。每个节点都应该使用此选项启动，以及`--nowait-nodes`。

    假设您有一个 4 节点集群，其数据节点的 ID 分别为 2、3、4 和 5，您希望仅使用节点 2、4 和 5 执行部分初始启动，即省略节点 3：

    ```sql
    $> ndbd --ndb-nodeid=2 --nowait-nodes=3 --initial-start
    $> ndbd --ndb-nodeid=4 --nowait-nodes=3 --initial-start
    $> ndbd --ndb-nodeid=5 --nowait-nodes=3 --initial-start
    ```

    使用此选项时，您还必须使用`--ndb-nodeid`选项指定要启动的数据节点的节点 ID。

    重要

    不要将此选项与**ndb_mgmd**的`--nowait-nodes`选项混淆，该选项可用于启用配置有多个管理服务器的集群在不所有管理服务器在线的情况下启动。

+   [`--install[=*`name`*]`](mysql-cluster-programs-ndbd.html#option_ndbd_install)

    | 命令行格式 | `--install[=name]` |
    | --- | --- |
    | 平台特定 | Windows |
    | 类型 | 字符串 |
    | 默认值 | `ndbd` |

    安装**ndbd**作为 Windows 服务。可选地，您可以为服务指定一个名称；如果未设置，则服务名称默认为`ndbd`。虽然最好在`my.ini`或`my.cnf`配置文件中指定其他**ndbd**程序选项，但也可以与`--install`一起使用。但是，在这种情况下，必须首先指定`--install`选项，然后再给出任何其他选项，以确保 Windows 服务安装成功。

    通常不建议将此选项与`--initial`选项一起使用，因为这会导致每次停止和启动服务时数据节点文件系统被擦除并重建。如果您打算与其他影响数据节点启动的**ndbd**选项一起使用`--initial-start`、`--nostart`和`--nowait-nodes`一起使用`--install`，则应该非常小心，并确保您充分理解并考虑可能的后果。

    `--install`选项对非 Windows 平台无效。

+   `--logbuffer-size=*`#`*`

    | 命令行格式 | `--logbuffer-size=#` |
    | --- | --- |
    | 类型 | 整数 |
    | 默认值 | `32768` |
    | 最小值 | `2048` |
    | 最大值 | `4294967295` |

    设置数据节点日志缓冲区的大小。在进行大量额外日志记录的调试时，如果有太多日志消息，日志缓冲区可能会耗尽空间，从而可能会丢失一些日志消息。在正常操作中不应发生这种情况。

+   `--login-path`

    | 命令行格式 | `--login-path=path` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `[none]` |

    从登录文件中读取给定路径。

+   `--ndb-connectstring`

    | 命令行格式 | `--ndb-connectstring=connection_string` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `[无]` |

    设置连接到 ndb_mgmd 的连接字符串。语法："[nodeid=id;][host=]hostname[:port]"。覆盖 NDB_CONNECTSTRING 和 my.cnf 中的条目。

+   `--ndb-mgmd-host`

    | 命令行格式 | `--ndb-mgmd-host=connection_string` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `[无]` |

    与`--ndb-connectstring`相同。

+   `--ndb-nodeid`

    | 命令行格式 | `--ndb-nodeid=#` |
    | --- | --- |
    | 类型 | 整数 |
    | 默认值 | `[无]` |

    设置此节点的节点 ID，覆盖由--ndb-connectstring 设置的任何 ID。

+   `--ndb-optimized-node-selection`

    | 命令行格式 | `--ndb-optimized-node-selection` |
    | --- | --- |
    | 已移除 | 8.0.31 |

    启用优化以选择事务节点。默认情况下启用；使用`--skip-ndb-optimized-node-selection`来禁用。

+   `--nodaemon`

    | 命令行格式 | `--nodaemon` |
    | --- | --- |

    防止**ndbd**或**ndbmtd**")作为守护进程执行。此选项覆盖`--daemon`选项。在调试二进制文件时，将输出重定向到屏幕非常有用。

    在 Windows 上，**ndbd**和**ndbmtd**")的默认行为是在前台运行，使得此选项在 Windows 平台上不必要，对 Windows 平台没有影响。

+   `--no-defaults`

    | 命令行格式 | `--no-defaults` |
    | --- | --- |

    不要从登录文件以外的任何选项文件中读取默认选项。

+   `--nostart`, `-n`

    | 命令行格式 | `--nostart` |
    | --- | --- |

    指示**ndbd** 不自动启动。当使用此选项时，**ndbd** 连接到管理服务器，从中获取配置数据，并初始化通信对象。但是，直到管理服务器明确要求执行引擎启动为止，它才不会实际启动执行引擎。这可以通过在管理客户端中发出适当的`START` 命令来实现（参见第 25.6.1 节，“NDB 集群管理客户端中的命令”）。

+   [`--nowait-nodes=*`node_id_1`*[, *`node_id_2`*[, ...]]`](mysql-cluster-programs-ndbd.html#option_ndbd_nowait-nodes)

    | 命令行格式 | `--nowait-nodes=list` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 |  |

    此选项接受一个数据节点列表，集群在启动之前不等待这些节点。

    这可以用于以分区状态启动集群。例如，在一个 4 节点集群中只有一半数据节点（节点 2、3、4 和 5）运行时，可以使用`--nowait-nodes=3,5`启动每个**ndbd** 进程。在这种情况下，集群在节点 2 和 4 连接后立即启动，并且不会等待`StartPartitionedTimeout` 毫秒，以等待节点 3 和 5 连接，否则会等待。

    如果您想要启动与前面示例中相同的集群，但不包括一个**ndbd**（例如，假设节点 3 的主机机器发生了硬件故障），那么使用`--nowait-nodes=3`启动��点 2、4 和 5。然后，集群在节点 2、4 和 5 连接后立即启动，并且不等待节点 3 启动。

+   `--print-defaults`

    | 命令行格式 | `--print-defaults` |
    | --- | --- |

    打印程序参数列表并退出。

+   [`--remove[=*`name`*]`](mysql-cluster-programs-ndbd.html#option_ndbd_remove)

    | 命令行格式 | `--remove[=name]` |
    | --- | --- |
    | 特定平台 | Windows |
    | 类型 | 字符串 |
    | 默认值 | `ndbd` |

    导致之前安装为 Windows 服务的**ndbd** 进程被移除。可选地，您可以指定要卸载的服务的名称；如果未设置，则服务名称默认为`ndbd`。

    `--remove` 选项在非 Windows 平台上没有效果。

+   `--usage`

    | 命令行格式 | `--usage` |
    | --- | --- |

    显示帮助文本并退出；与--help 相同。

+   `--verbose`，`-v`

    导致额外的调试输出写入节点日志。

    您还可以在数据节点运行时使用`NODELOG DEBUG ON`和`NODELOG DEBUG OFF`来启用和禁用此额外的日志记录。

+   `--version`

    | 命令行格式 | `--version` |
    | --- | --- |

    显示版本信息并退出。

**ndbd**生成一组日志文件，这些文件放置在`config.ini`配置文件中的`DataDir`指定的目录中。

下面列出了这些日志文件。*`node_id`*是并代表节点的唯一标识符。例如，`ndb_2_error.log`是由节点 ID 为`2`的数据节点��成的错误日志。

+   `ndb_*`node_id`*_error.log`是一个文件，其中包含引用的**ndbd**进程遇到的所有崩溃的记录。此文件中的每个记录都包含一个简短的错误字符串和此崩溃的跟踪文件的引用。此文件中的典型条目可能如下所示：

    ```sql
    Date/Time: Saturday 30 July 2004 - 00:20:01
    Type of error: error
    Message: Internal program error (failed ndbrequire)
    Fault ID: 2341
    Problem data: DbtupFixAlloc.cpp
    Object of reference: DBTUP (Line: 173)
    ProgramName: NDB Kernel
    ProcessID: 14909
    TraceFile: ndb_2_trace.log.2
    ***EOM***
    ```

    可在**ndbd**退出代码和数据节点进程意外关闭时生成的消息列表中找到可能的列表。

    重要

    *错误日志文件中的最后一条记录不一定是最新的*（也不太可能是）。错误日志中的条目*不*按时间顺序列出；而是对应于`ndb_*`node_id`*_trace.log.next`文件中确定的跟踪文件顺序。因此，错误日志条目以循环而非顺序方式被覆盖。

+   `ndb_*`node_id`*_trace.log.*`trace_id`*`是描述错误发生前发生的情况的跟踪文件。这些信息对 NDB Cluster 开发团队进行分析很有用。

    可以配置在旧文件被覆盖之前创建的这些跟踪文件的数量。*`trace_id`*是一个数字，每个连续的跟踪文件都会递增。

+   `ndb_*`node_id`*_trace.log.next`是跟踪下一个要分配的跟踪文件编号的文件。

+   `ndb_*`node_id`*_out.log`是一个文件，其中包含**ndbd**进程输出的任何数据。仅当**ndbd**作为守护进程启动时才会创建此文件，这是默认行为。

+   `ndb_*`node_id`*.pid`是一个文件，包含了作为守护进程启动时**ndbd**进程的进程 ID。它还充当锁文件，以避免使用相同标识符启动节点。

+   `ndb_*`node_id`*_signal.log`是仅在调试版本的**ndbd**中使用的文件，在这种版本中，可以跟踪所有传入、传出和内部消息及其数据在**ndbd**进程中的情况。

不建议使用通过 NFS 挂载的目录，因为在某些环境中，这可能会导致问题，即使进程终止后，`.pid`文件上的锁仍然有效。

要启动**ndbd**，可能还需要指定管理服务器的主机名和其监听的端口。还可以选择指定进程要使用的节点 ID。

```sql
$> ndbd --connect-string="nodeid=2;host=ndb_mgmd.mysql.com:1186"
```

有关此问题的更多信息，请参见第 25.4.3.3 节，“NDB 集群连接字符串”。有关数据节点配置参数的更多信息，请参见第 25.4.3.6 节，“定义 NDB 集群数据节点”。

当**ndbd**启动时，实际上启动了两个进程。第一个称为“angel 进程”，它的唯一工作是发现执行进程何时完成，然后重新启动**ndbd**进程（如果配置为这样做）。因此，如果尝试使用 Unix 的**kill**命令杀死**ndbd**，则需要杀死这两个进程，首先是 angel 进程。终止**ndbd**进程的首选方法是使用管理客户端并从那里停止进程。

执行过程使用一个线程来读取、写入和扫描数据，以及进行所有其他活动。该线程是异步实现的，因此可以轻松处理成千上万个并发操作。此外，一个看门狗线程监督执行线程，以确保它不会陷入无限循环。一组线程处理文件 I/O，每个线程可以处理一个打开的文件。线程还可以被用于**ndbd**进程中的传输连接。在执行大量操作（包括更新）的多处理器系统中，**ndbd**进程可以使用高达 2 个 CPU。

对于具有多个 CPU 的机器，可以使用几个属于不同节点组的**ndbd**进程；然而，这样的配置仍被视为实验性的，在生产环境中不支持 MySQL 8.0。请参阅第 25.2.7 节，“NDB 集群的已知限制”。
