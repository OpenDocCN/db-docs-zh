# 25.5.26 ndb_select_count — 打印 NDB 表的行数

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-select-count.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-select-count.html)

**ndb_select_count** 打印一个或多个 `NDB` 表中的行数。对于单个表，结果等同于使用 MySQL 语句 `SELECT COUNT(*) FROM *`tbl_name`*` 获得的结果。

#### 用法

```sql
ndb_select_count [-c *connection_string*] -d*db_name* *tbl_name*[, *tbl_name2*[, ...]]
```

可与**ndb_select_count**一起使用的选项如下表所示。表后面会有额外的描述。

**表 25.47 与程序 ndb_select_count 一起使用的命令行选项**

| 格式 | 描述 | 添加、弃用或移除 |
| --- | --- | --- |
| `--character-sets-dir=path` | 包含字符集的目录 | 移除：8.0.31 |
| `--connect-retries=#` | 放弃之前重试连接的次数 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--connect-retry-delay=#` | 尝试联系管理服务器之间等待的秒数 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--connect-string=connection_string`,`-c connection_string` | 与 --ndb-connectstring 相同 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--core-file` | 在错误时写入核心文件；用于调试 | 移除：8.0.31 |
| `--database=name`,`-d name` | 表所在数据库的名称 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--defaults-extra-file=path` | 在全局文件读取后读取给定文件 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--defaults-file=path` | 仅从给定文件中读取默认选项 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--defaults-group-suffix=string` | 也读取带有 concat(group, suffix) 的组 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--help`,`-?` | 显示帮助文本并退出 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--lock=#`,`-l #` | 锁类型 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--login-path=path` | 从登录文件中读取给定路径 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--ndb-connectstring=connection_string`,`-c connection_string` | 设置连接到 ndb_mgmd 的连接字符串。语法: "[nodeid=id;][host=]hostname[:port]"。覆盖 NDB_CONNECTSTRING 和 my.cnf 中的条目 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--ndb-mgmd-host=connection_string`,`-c connection_string` | 与 --ndb-connectstring 相同 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--ndb-nodeid=#` | 为此节点设置节点 ID，覆盖由 --ndb-connectstring 设置的任何 ID | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--ndb-optimized-node-selection` | 启用用于事务节点选择的优化。默认启用；使用 --skip-ndb-optimized-node-selection 来禁用 | 已移除: 8.0.31 |
| `--no-defaults` | 不从登录文件以外的任何选项文件中读取默认选项 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--parallelism=#`,`-p #` | 并行度 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--print-defaults` | 打印程序参数列表并退出 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--usage`,`-?` | 显示帮助文本并退出；与 --help 相同 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--version`,`-V` | 显示版本信息并退出 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| 格式 | 描述 | 添加、弃用或移除 |

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
    | 默认值 | `[none]` |

    与 `--ndb-connectstring` 相同。

+   `--core-file`

    | 命令行格式 | `--core-file` |
    | --- | --- |
    | 移除 | 8.0.31 |

    在错误时写入核心文件；用于调试。

+   `--defaults-file`

    | 命令行格式 | `--defaults-file=path` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `[none]` |

    仅从给定文件中读取默认选项。

+   `--defaults-extra-file`

    | 命令行格式 | `--defaults-extra-file=path` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `[none]` |

    在全局文件读取后读取给定文件。

+   `--defaults-group-suffix`

    | 命令行格式 | `--defaults-group-suffix=string` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `[none]` |

    同时读取带有 concat(group, suffix) 的组。

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

    设置连接到 ndb_mgmd 的连接字符串。语法："[nodeid=id;][host=]hostname[:port]"。覆盖 NDB_CONNECTSTRING 和 my.cnf 中的条目。

+   `--ndb-mgmd-host`

    | 命令行格式 | `--ndb-mgmd-host=connection_string` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `[none]` |

    与`--ndb-connectstring`相同。

+   `--ndb-optimized-node-selection`

    | 命令行格式 | `--ndb-optimized-node-selection` |
    | --- | --- |
    | 已移除 | 8.0.31 |

    启用优化以选择事务节点。默认启用；使用`--skip-ndb-optimized-node-selection`来禁用。

+   `--ndb-nodeid`

    | 命令行格式 | `--ndb-nodeid=#` |
    | --- | --- |
    | 类型 | 整数 |
    | 默认值 | `[none]` |

    为此节点设置节点 ID，覆盖由`--ndb-connectstring`设置的任何 ID。

+   `--no-defaults`

    | 命令行格式 | `--no-defaults` |
    | --- | --- |

    不要从除登录文件以外的任何选项文件中读取默认选项。

+   `--print-defaults`

    | 命令行格式 | `--print-defaults` |
    | --- | --- |

    打印程序参数列表并退出。

+   `--usage`

    | 命令行格式 | `--usage` |
    | --- | --- |

    显示帮助文本并退出；与`--help`相同。

+   `--version`

    | 命令行格式 | `--version` |
    | --- | --- |

    显示版本信息并退出。

通过在调用此命令时用空格分隔的表名列表，可以从同一数据库中获取多个表的行数，如**示例输出**所示。

#### 示例输出

```sql
$> ./ndb_select_count -c localhost -d ctest1 fish dogs
6 records in table fish
4 records in table dogs
```
