# 25.5.27 ndb_show_tables — 显示 NDB 表的列表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-show-tables.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-show-tables.html)

**ndb_show_tables** 显示集群中所有`NDB`数据库对象的列表。默认情况下，这不仅包括用户创建的表和`NDB`系统表，还包括`NDB`特定的索引、内部触发器和 NDB 集群磁盘数据对象。

可与 **ndb_show_tables** 一起使用的选项显示在下表中。表后面是附加描述。

**表 25.48 与程序 ndb_show_tables 一起使用的命令行选项**

| 格式 | 描述 | 添加、弃用或移除 |
| --- | --- | --- |
| `--character-sets-dir=path` | 包含字符集的目录 | 移除：8.0.31 |
| `--connect-retries=#` | 放弃之前重试连接的次数 | (在基于 MySQL 8.0 的所有 NDB 发行版中支持) |
| `--connect-retry-delay=#` | 尝试联系管理服务器之间等待的秒数 | (在基于 MySQL 8.0 的所有 NDB 发行版中支持) |
| `--connect-string=connection_string`,`-c connection_string` | 与 --ndb-connectstring 相同 | (在基于 MySQL 8.0 的所有 NDB 发行版中支持) |
| `--core-file` | 在错误时写入核心文件；用于调试 | 移除：8.0.31 |
| `--database=name`,`-d name` | 指定表所在的数据库；数据库名称后必须跟表名 | (在基于 MySQL 8.0 的所有 NDB 发行版中支持) |
| `--defaults-extra-file=path` | 在读取全局文件后读取给定文件 | (在基于 MySQL 8.0 的所有 NDB 发行版中支持) |
| `--defaults-file=path` | 仅从给定文件中读取默认选项 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--defaults-group-suffix=string` | 还读取具有 concat(group, suffix) 的组 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--login-path=path` | 从登录文件中读取给定路径 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--loops=#`,`-l #` | 重复输出的次数 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--help`,`-?` | 显示帮助文本并退出 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--ndb-connectstring=connection_string`,`-c connection_string` | 设置连接到 ndb_mgmd 的连接字符串。语法：“[nodeid=id;][host=]hostname[:port]”。覆盖 NDB_CONNECTSTRING 和 my.cnf 中的条目 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--ndb-mgmd-host=connection_string`,`-c connection_string` | 与 --ndb-connectstring 相同 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--ndb-nodeid=#` | 设置此节点的节点 ID，覆盖 --ndb-connectstring 设置的任何 ID | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--ndb-optimized-node-selection` | 启用用于事务节点选择的优化。默认情况下启用；使用 --skip-ndb-optimized-node-selection 来禁用 | 已移除: 8.0.31 |
| `--no-defaults` | 不从登录文件以外的任何选项文件中读取默认选项 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--parsable`,`-p` | 返回适用于 MySQL LOAD DATA 语句的输出 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--print-defaults` | 打印程序参数列表并退出 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--show-temp-status` | 显示表临时标志 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--type=#`,`-t #` | 限制输出到此类型的对象 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--unqualified`,`-u` | 不限定表名 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--usage`,`-?` | 显示帮助文本并退出；与 --help 相同 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--version`,`-V` | 显示版本信息并退出 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| 格式 | 描述 | 添加、弃用或移除 |

#### 使用

```sql
ndb_show_tables [-c *connection_string*]
```

+   `--character-sets-dir`

    | 命令行格式 | `--character-sets-dir=path` |
    | --- | --- |
    | 已移除 | 8.0.31 |

    包含字符集的目录。

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

    | 命令行格式 | `--connect-string=connection_string` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `[无]` |

    与`--ndb-connectstring`相同。

+   `--core-file`

    | 命令行格式 | `--core-file` |
    | --- | --- |
    | 已移除 | 8.0.31 |

    在错误时写入核心文件；用于调试。

+   `--database`, `-d`

    指定所找到所需表的数据库的名称。如果给定此选项，则表的名称必须跟在数据库名称后面。

    如果未指定此选项，并且`TEST_DB`数据库中找不到任何表，则**ndb_show_tables**会发出警告。

+   `--defaults-extra-file`

    | 命令行格式 | `--defaults-extra-file=path` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `[无]` |

    在读取全局文件后读取给定文件。

+   `--defaults-file`

    | 命令行格式 | `--defaults-file=path` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `[无]` |

    仅从给定文件中读取默认选项。

+   `--defaults-group-suffix`

    | 命令行格式 | `--defaults-group-suffix=string` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `[无]` |

    还读取具有 concat(group, suffix)的组。

+   `--help`

    | 命令行格式 | `--help` |
    | --- | --- |

    显示帮助文本并退出。

+   `--login-path`

    | 命令行格式 | `--login-path=path` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `[无]` |

    从登录文件中读取给定路径。

+   `--loops`, `-l`

    指定实用程序应执行的次数。当未指定此选项时，此值为 1，但如果使用该选项，则必须为其提供整数参数。

+   `--ndb-connectstring`

    | 命令行格式 | `--ndb-connectstring=connection_string` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `[无]` |

    设置用于连接到 ndb_mgmd 的连接字符串。语法："[nodeid=id;][host=]hostname[:port]"。覆盖 NDB_CONNECTSTRING 和 my.cnf 中的条目。

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

    为此节点设置节点 ID，覆盖`--ndb-connectstring`设置的任何 ID。

+   `--ndb-optimized-node-selection`

    | 命令行格式 | `--ndb-optimized-node-selection` |
    | --- | --- |
    | 已移除 | 8.0.31 |

    启用用于事务节点选择的优化。默认情况下启用；使用`--skip-ndb-optimized-node-selection`来禁用。

+   `--no-defaults`

    | 命令行格式 | `--no-defaults` |
    | --- | --- |

    不要从除登录文件以外的任何选项文件中读取默认选项。

+   `--parsable`, `-p`

    使用此选项会使输出格式适合与`LOAD DATA`一起使用。

+   `--print-defaults`

    | 命令行格式 | `--print-defaults` |
    | --- | --- |

    打印程序参数列表并退出。

+   `--show-temp-status`

    如果指定，将显示临时表。

+   `--type`, `-t`

    可用于将输出限制为此处显示的整数类型代码指定的一种对象类型：

    +   `1`: 系统表

    +   `2`: 用户创建的表

    +   `3`: 唯一哈希索引

    任何其他值会导致列出所有`NDB`数据库对象（默认）。

+   `--unqualified`, `-u`

    如果指定，将显示未经限定的对象名称。

+   `--usage`

    | 命令行格式 | `--usage` |
    | --- | --- |

    显示帮助文本并退出；与`--help`相同。

+   `--version`

    | 命令行格式 | `--version` |
    | --- | --- |

    显示版本信息并退出。

注意

只有用户创建的 NDB Cluster 表可以从 MySQL 访问；系统表如`SYSTAB_0`对**mysqld**不可见。但是，您可以使用`NDB` API 应用程序（如**ndb_select_all**）来检查系统表的内容（请参阅 Section 25.5.25, “ndb_select_all — Print Rows from an NDB Table”）。

在 NDB 8.0.20 之前，该程序在运行完成时打印`NDBT_ProgramExit - *`status`*`，因为不必要地依赖于`NDBT`测试库。已删除此依赖项，消除了多余的输出。
