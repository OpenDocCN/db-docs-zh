# 25.5.30 ndb_waiter — 等待 NDB 集群达到给定状态

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-waiter.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-waiter.html)

**ndb_waiter** 每隔 100 毫秒重复打印出所有集群数据节点的状态，直到集群达到给定状态或超过`--timeout`限制，然后退出。默认情况下，它等待集群达到`STARTED`状态，其中所有节点已启动并连接到集群。可以使用`--no-contact`和`--not-started`选项覆盖此设置。

此实用程序报告的节点状态如下：

+   `NO_CONTACT`: 无法联系节点。

+   `UNKNOWN`: 可以联系节点，但其状态尚不清楚。通常，这意味着节点已收到管理服务器的`START` 或 `RESTART` 命令，但尚未执行。

+   `NOT_STARTED`: 节点已停止，但仍与集群保持联系。在使用管理客户端的 `RESTART` 命令重���启动节点时会出现此状态。

+   `STARTING`: 节点的**ndbd** 进程已启动，但节点尚未加入集群。

+   `STARTED`: 节点正在运行，并已加入集群。

+   `SHUTTING_DOWN`: 节点正在关闭。

+   `SINGLE USER MODE`: 当集群处于单用户模式时，所有集群数据节点都会显示此状态。

可与**ndb_waiter**一起使用的选项显示在以下表中。表后面是附加描述。

**表 25.51 与程序 ndb_waiter 一起使用的命令行选项**

| 格式 | 描述 | 添加、弃用或移除 |
| --- | --- | --- |
| `--character-sets-dir=path` | 包含字符集的目录 | 移除：8.0.31 |
| `--connect-retries=#` | 放弃之前重试连接的次数 | (在基于 MySQL 8.0 的所有 NDB 发行版中支持) |
| `--connect-retry-delay=#` | 尝试联系管理服务器之间等待的秒数 | (在基于 MySQL 8.0 的所有 NDB 发行版中支持) |
| `--connect-string=connection_string`,`-c connection_string` | 与 --ndb-connectstring 相同 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--core-file` | 在错误时写入核心文件；用于调试 | 已移除：8.0.31 |
| `--defaults-extra-file=path` | 在读取全局文件后读取给定文件 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--defaults-file=path` | 仅从给定文件中读取默认选项 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--defaults-group-suffix=string` | 还读取带有 concat(group, suffix) 的组 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--help`,`-?` | 显示帮助文本并退出 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--login-path=path` | 从登录文件中读取给定路径 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--ndb-connectstring=connection_string`,`-c connection_string` | 设置连接到 ndb_mgmd 的连接字符串。语法：“[nodeid=id;][host=]hostname[:port]”。覆盖 NDB_CONNECTSTRING 和 my.cnf 中的条目 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--ndb-mgmd-host=connection_string`,`-c connection_string` | 与 --ndb-connectstring 相同 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--ndb-nodeid=#` | 为此节点设置节点 ID，覆盖 --ndb-connectstring 设置的任何 ID | 已移除：8.0.31 |
| `--ndb-optimized-node-selection` | 启用用于事务节点选择的优化。默认启用；使用 --skip-ndb-optimized-node-selection 禁用 | 已移除：8.0.31 |
| `--no-contact`,`-n` | 等待集群达到无联系状态 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--no-defaults` | 不从登录文件以外的任何选项文件中读取默认选项 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--not-started` | 等待集群达到未启动状态 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--nowait-nodes=list` | 不等待的节点列表 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--print-defaults` | 打印程序参数列表并退出 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--single-user` | 等待集群进入单用户模式 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--timeout=#`,`-t #` | 等待这么多秒，然后退出，无论集群是否达到所需状态 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--usage`,`-?` | 显示帮助文本并退出；与 --help 相同 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--verbose=#`,`-v` | 设置输出详细程度；查看文本以获取输入和返回值 | 添加: 8.0.37 |
| `--version`,`-V` | 显示版本信息并退出 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--wait-nodes=list`,`-w list` | 等待的节点列表 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| 格式 | 描述 | 添加、废弃或移除 |

#### 用法

```sql
ndb_waiter [-c *connection_string*]
```

#### 附加选项

+   `--character-sets-dir`

    | 命令行格式 | `--character-sets-dir=path` |
    | --- | --- |
    | 移除 | 8.0.31 |

    包含字符集的目录。

+   `--connect-retries`

    | 命令行格式 | `--connect-retries=#` |
    | --- | --- |
    | 类型 | 整数 |
    | 默认值 | `12` |
    | 最小值 | `0` |
    | 最大值 | `12` |

    连接重试次数上限。

+   `--connect-retry-delay`

    | 命令行格式 | `--connect-retry-delay=#` |
    | --- | --- |
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

    与`--ndb-connectstring`相同。

+   `--core-file`

    | 命令行格式 | `--core-file` |
    | --- | --- |
    | 已移除 | 8.0.31 |

    在错误时写入核心文件；用于调试。

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

    还读取具有 concat(group, suffix)的组。

+   `--login-path`

    | 命令行格式 | `--login-path=path` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `[none]` |

    从登录文件中读取给定路径。

+   `--help`

    | 命令行格式 | `--help` |
    | --- | --- |

    显示帮助文本并退出。

+   `--ndb-connectstring`

    | 命令行格式 | `--ndb-connectstring=connection_string` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `[none]` |

    设置用于连接到 ndb_mgmd 的连接字符串。语法："[nodeid=id;][host=]hostname[:port]"。覆盖 NDB_CONNECTSTRING 和 my.cnf 中的条目。

+   `--ndb-mgmd-host`

    | 命令行格式 | `--ndb-mgmd-host=connection_string` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `[none]` |

    与--`ndb-connectstring`相同。

+   `--ndb-nodeid`

    | 命令行格式 | `--ndb-nodeid=#` |
    | --- | --- |
    | 已移除 | 8.0.31 |
    | 类型 | 整数 |
    | 默认值 | `[none]` |

    设置此节点的节点 ID，覆盖由`--ndb-connectstring`设置的任何 ID。

+   `--ndb-optimized-node-selection`

    | 命令行格式 | `--ndb-optimized-node-selection` |
    | --- | --- |
    | 移除 | 8.0.31 |

    启用优化以选择事务节点。默认情况下启用；使用`--skip-ndb-optimized-node-selection`来禁用。

+   `--no-contact`, `-n`

    而不是等待`STARTED`状态，**ndb_waiter**在退出之前继续运行，直到集群达到`NO_CONTACT`状态。

+   `--no-defaults`

    | 命令行格式 | `--no-defaults` |
    | --- | --- |

    不要从登录文件以外的任何选项文件中读取默认选项。

+   `--not-started`

    而不是等待`STARTED`状态，**ndb_waiter**在退出之前继续运行，直到集群达到`NOT_STARTED`状态。

+   `--nowait-nodes=*`list`*`

    当使用此选项时，**ndb_waiter**不会等待列出的节点 ID。列表以逗号分隔；范围可以用破折号表示，如下所示：

    ```sql
    $> ndb_waiter --nowait-nodes=1,3,7-9
    ```

    重要

    不要与`--wait-nodes`选项一起使用。

+   `--print-defaults`

    | 命令行格式 | `--print-defaults` |
    | --- | --- |

    打印程序参数列表并退出。

+   `--timeout=*`seconds`*`, `-t *`seconds`*`

    等待时间。如果在此秒数内未达到所需状态，则程序将退出。默认值为 120 秒（1200 个报告周期）。

+   `--single-user`

    该程序等待集群进入单用户��式。

+   `--usage`

    | 命令行格式 | `--usage` |
    | --- | --- |

    显示帮助文本并退出；与`--help`相同。

+   `--verbose`

    | 命令行格式 | `--verbose=#` |
    | --- | --- |
    | 引入 | 8.0.37 |
    | 类型 | 整数 |
    | 默认值 | `2` |
    | 最小值 | `0` |
    | 最大值 | `2` |

    控制打印输出的详细程度。可能的级别及其效果在此处列出：

    +   `0`: 仅返回退出代码，不打印（请参阅后续的退出代码）。

    +   `1`: 仅打印最终连接状态。

    +   `2`: 每次检查时打印状态。

        这与 8.4 之前的 NDB Cluster 版本的行为相同。

    **ndb_waiter** 返回的退出代码在此处列出，及其含义：

    +   `0`: 成功。

    +   `1`: 等待超时。

    +   `2`: 参数错误，如无效的节点 ID。

    +   `3`: 连接到管理服务器失败。

+   `--version`

    | 命令行格式 | `--version` |
    | --- | --- |

    显示版本信息并退出。

+   `--wait-nodes=*`list`*`, `-w *`list`*`

    当使用此选项时，**ndb_waiter** 仅等待列出的节点。列表以逗号分隔；范围可以用破折号表示，如下所示：

    ```sql
    $> ndb_waiter --wait-nodes=2,4-6,10
    ```

    重要

    不要与`--nowait-nodes`选项一起使用。

**示例输出。** 这里显示了对一个 4 节点集群运行**ndb_waiter**的输出，其中两个节点已手动关闭然后重新启动。重复报告（由`...`表示）被省略。

```sql
$> ./ndb_waiter -c localhost

Connecting to mgmsrv at (localhost)
State node 1 STARTED
State node 2 NO_CONTACT
State node 3 STARTED
State node 4 NO_CONTACT
Waiting for cluster enter state STARTED

...

State node 1 STARTED
State node 2 UNKNOWN
State node 3 STARTED
State node 4 NO_CONTACT
Waiting for cluster enter state STARTED

...

State node 1 STARTED
State node 2 STARTING
State node 3 STARTED
State node 4 NO_CONTACT
Waiting for cluster enter state STARTED

...

State node 1 STARTED
State node 2 STARTING
State node 3 STARTED
State node 4 UNKNOWN
Waiting for cluster enter state STARTED

...

State node 1 STARTED
State node 2 STARTING
State node 3 STARTED
State node 4 STARTING
Waiting for cluster enter state STARTED

...

State node 1 STARTED
State node 2 STARTED
State node 3 STARTED
State node 4 STARTING
Waiting for cluster enter state STARTED

...

State node 1 STARTED
State node 2 STARTED
State node 3 STARTED
State node 4 STARTED
Waiting for cluster enter state STARTED
```

注意

如果未指定连接字符串，则**ndb_waiter** 尝试连接到`localhost`上的管理服务器，并报告`Connecting to mgmsrv at (null)`。

在 NDB 8.0.20 之前，该程序在运行完成后打印`NDBT_ProgramExit - *`status`*`，这是由于对`NDBT`测试库的不必要依赖。已经移除了这种依赖，消除了多余的输出。
