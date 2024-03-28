# 25.5.2 ndbinfo_select_all — 从 ndbinfo 表中选择

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndbinfo-select-all.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndbinfo-select-all.html)

**ndbinfo_select_all** 是一个客户端程序，从 `ndbinfo` 数据库中的一个或多个表中选择所有行和列

并非所有在 **mysql** 客户端中可用的 `ndbinfo` 表都可以被此程序读取（请参见本节后面）。此外，**ndbinfo_select_all** 可以显示一些无法使用 SQL 访问的 `ndbinfo` 内部表的信息，包括 `tables` 和 `columns` 元数据表。

要使用 **ndbinfo_select_all** 从一个或多个 `ndbinfo` 表中选择，需要在调用程序时提供表的名称，如下所示：

```sql
$> ndbinfo_select_all *table_name1*  [*table_name2*] [...]
```

例如：

```sql
$> ndbinfo_select_all logbuffers logspaces
== logbuffers ==
node_id log_type        log_id  log_part        total   used    high
5       0       0       0       33554432        262144  0
6       0       0       0       33554432        262144  0
7       0       0       0       33554432        262144  0
8       0       0       0       33554432        262144  0
== logspaces ==
node_id log_type        log_id  log_part        total   used    high
5       0       0       0       268435456       0       0
5       0       0       1       268435456       0       0
5       0       0       2       268435456       0       0
5       0       0       3       268435456       0       0
6       0       0       0       268435456       0       0
6       0       0       1       268435456       0       0
6       0       0       2       268435456       0       0
6       0       0       3       268435456       0       0
7       0       0       0       268435456       0       0
7       0       0       1       268435456       0       0
7       0       0       2       268435456       0       0
7       0       0       3       268435456       0       0
8       0       0       0       268435456       0       0
8       0       0       1       268435456       0       0
8       0       0       2       268435456       0       0
8       0       0       3       268435456       0       0
$>
```

可用于 **ndbinfo_select_all** 的选项显示在下表中。表后面会有额外的描述。

**表 25.25 与程序 ndbinfo_select_all 一起使用的命令行选项**

| 格式 | 描述 | 添加、弃用或移除 |
| --- | --- | --- |
| `--character-sets-dir=path` | 包含字符集的目录 | 移除：8.0.31 |
| `--connect-retries=#` | 放弃之前重试连接的次数 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--connect-retry-delay=#` | 尝试联系管理服务器之间等待的秒数 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--connect-string=connection-string`,`-c connection_string` | 与 --ndb-connectstring 相同 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--core-file` | 在错误时写入核心文件；用于调试 | 移除：8.0.31 |
| `--database=db_name`,`-d` | 表所在数据库的名称 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--defaults-extra-file=path` | 在读取全局文件后读取给定文件 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--defaults-file=path` | 仅从给定文件中读取默认选项 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--defaults-group-suffix=string` | 还读取带有 `concat(group, suffix)` 的组 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--delay=#` | 设置循环之间的延迟时间（秒） | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--help`,`-?` | 显示帮助文本并退出 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--login-path=path` | 从登录文件中读取给定路径 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--loops=#`,`-l` | 设置执行选择操作的次数 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--ndb-connectstring=connection-string`,`-c` | 设置连接到 `ndb_mgmd` 的连接字符串。语法: "[nodeid=id;][host=]hostname[:port]"。覆盖 `NDB_CONNECTSTRING` 和 `my.cnf` 中的条目 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--ndb-mgmd-host=connection-string`,`-c` | 与 `--ndb-connectstring` 相同 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--ndb-nodeid=#` | 为此节点设置节点 ID，覆盖 `--ndb-connectstring` 设置的任何 ID | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--no-defaults` | 不要从登录文件以外的任何选项文件中读取默认选项 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--ndb-optimized-node-selection` | 启用优化以选择事务节点。默认启用；使用 --skip-ndb-optimized-node-selection 来禁用 | 移除：8.0.31 |
| `--parallelism=#`,`-p` | 设置并行度 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--print-defaults` | 打印程序参数列表并退出 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--usage`,`-?` | 显示帮助文本并退出；与 --help 相同 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--version`,`-V` | 显示版本信息并退出 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| 格式 | 描述 | 添加、弃用或移除 |

+   `--character-sets-dir`

    | 命令行格式 | `--character-sets-dir=path` |
    | --- | --- |
    | 移除 | 8.0.31 |

    包含字符集的目录。

+   `--core-file`

    | 命令行格式 | `--core-file` |
    | --- | --- |
    | 移除 | 8.0.31 |

    在错误时写入核心文件；用于调试。

+   `--connect-retries`

    | 命令行格式 | `--connect-retries=#` |
    | --- | --- |
    | 类型 | 整数 |
    | 默认值 | `12` |
    | 最小值 | `0` |
    | 最大值 | `12` |

    在放弃之前重试连接的次数。

+   `--connect-retry-delay`

    | 命令行格式 | `--connect-retry-delay=#` |
    | --- | --- |
    | 类型 | 整数 |
    | 默认值 | `5` |
    | 最小值 | `0` |
    | 最大值 | `5` |

    尝试联系管理服务器之间等待的秒数。

+   `--connect-string`

    | 命令行格式 | `--connect-string=connection-string` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `[none]` |

    与 `--ndb-connectstring` 相同。

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

    仅从给定文件中读取默认选项。

+   `--defaults-group-suffix`

    | 命令行格式 | `--defaults-group-suffix=string` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `[none]` |

    同时读取使用 concat(group, suffix) 的组。

+   `--delay=`seconds``

    | 命令行格式 | `--delay=#` |
    | --- | --- |
    | 类型 | 数字 |
    | 默认值 | `5` |
    | 最小值 | `0` |
    | 最大值 | `MAX_INT` |

    此选项设置执行循环之间等待的秒数。如果 `--loops` 设置为 0 或 1，则无效。

+   `--help`

    | 命令行格式 | `--help` |
    | --- | --- |

    显示帮助文本并退出。

+   `--login-path`

    | 命令行格式 | `--login-path=path` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `[none]` |

    从登录文件中读取给定路径。

+   `--loops=`number``, `-l *`number`*`

    | 命令行格式 | `--loops=#` |
    | --- | --- |
    | 类型 | 数字 |
    | 默认值 | `1` |
    | 最小值 | `0` |
    | 最大值 | `MAX_INT` |

    此选项设置执行选择的次数。使用 `--delay` 设置循环之间的时间。

+   `--ndb-connectstring`

    | 命令行格式 | `--ndb-connectstring=connection-string` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `[none]` |

    设置用于连接到 ndb_mgmd 的连接字符串。语法："[nodeid=id;][host=]hostname[:port]"。覆盖 NDB_CONNECTSTRING 和 my.cnf 中的条目。

+   `--ndb-mgmd-host`

    | 命令行格式 | `--ndb-mgmd-host=connection-string` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `[none]` |

    与 `--ndb-connectstring` 相同。

+   `--ndb-nodeid`

    | 命令行格式 | `--ndb-nodeid=#` |
    | --- | --- |
    | 类型 | 整数 |
    | 默认值 | `[无]` |

    为此节点设置节点 ID，覆盖--ndb-connectstring 设置的任何 ID。

+   `--ndb-optimized-node-selection`

    | 命令行格式 | `--ndb-optimized-node-selection` |
    | --- | --- |
    | 移除 | 8.0.31 |

    启用节点选择优化以进行事务。默认情况下启用；使用`--skip-ndb-optimized-node-selection`来禁用。

+   `--no-defaults`

    | 命令行格式 | `--no-defaults` |
    | --- | --- |

    不要从除登录文件之外的任何选项文件中读取默认选项。

+   `--print-defaults`

    | 命令行格式 | `--print-defaults` |
    | --- | --- |

    打印程序参数列表并退出。

+   `--usage`

    | 命令行格式 | `--usage` |
    | --- | --- |

    显示帮助文本并退出；与--help 相同。

+   `--version`

    | 命令行格式 | `--version` |
    | --- | --- |

    显示版本信息并退出。

**ndbinfo_select_all** 无法读取以下表格：

+   `arbitrator_validity_detail`

+   `arbitrator_validity_summary`

+   `cluster_locks`

+   `cluster_operations`

+   `cluster_transactions`

+   `disk_write_speed_aggregate_node`

+   `locks_per_fragment`

+   `memory_per_fragment`

+   `memoryusage`

+   `operations_per_fragment`

+   `server_locks`

+   `server_operations`

+   `server_transactions`

+   `table_info`
