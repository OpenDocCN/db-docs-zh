# 25.5.15 ndb_move_data — NDB 数据复制实用程序

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-move-data.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-move-data.html)

**ndb_move_data**将数据从一个 NDB 表复制到另一个。

#### 用法

通过指定源表和目标表的名称来调用该程序；这两个表中的任何一个或两者都可以选择性地带有数据库名称。这两个表都必须使用 NDB 存储引擎。

```sql
ndb_move_data *options* *source* *target*
```

可与**ndb_move_data**一起使用的选项显示在下表中。表后面会有附加描述。

**表 25.37 与程序 ndb_move_data 一起使用的命令行选项**

| 格式 | 描述 | 添加、弃用或移除 |
| --- | --- | --- |
| `--abort-on-error` | 在永久错误时转储核心（调试选项） | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--character-sets-dir=path` | 字符集所在目录 | 移除：8.0.31 |
| `--connect-retries=#` | 在放弃之前重试连接的次数 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--connect-retry-delay=#` | 尝试联系管理服务器之间等待的秒数 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--connect-string=connection_string`,`-c connection_string` | 与 --ndb-connectstring 相同 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--core-file` | 在错误时写入核心文件；用于调试 | 移除：8.0.31 |
| `--database=name`,`-d name` | 表所在数据库的名称 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--defaults-extra-file=path` | 在全局文件读取后读取给定文件 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--defaults-file=path` | 仅从给定文件中读取默认选项 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--defaults-group-suffix=string` | 还读取具有 concat(group, suffix) 的组 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--drop-source` | 在移动所有行后删除源表 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--error-insert` | 插入随机临时错误（用于测试） | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--exclude-missing-columns` | 忽略源表或目标表中的额外列 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--help`,`-?` | 显示帮助文本并退出 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--login-path=path` | 从登录文件中读取给定路径 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--lossy-conversions`,`-l` | 允许属性数据在转换为较小类型时被截断 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--ndb-connectstring=connection_string`,`-c connection_string` | 设置用于连接到 ndb_mgmd 的连接字符串。语法："[nodeid=id;][host=]hostname[:port]"。覆盖 NDB_CONNECTSTRING 和 my.cnf 中的条目 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--ndb-mgmd-host=connection_string`,`-c connection_string` | 与 --ndb-connectstring 相同 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--ndb-nodeid=#` | 为此节点设置节点 ID，覆盖由 --ndb-connectstring 设置的任何 ID | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--ndb-optimized-node-selection` | 启用用于事务节点选择的优化。默认启用；使用 --skip-ndb-optimized-node-selection 来禁用 | 已移除：8.0.31 |
| `--no-defaults` | 不从登录文件以外的任何选项文件中读取默认选项 | (适用于基于 MySQL 8.0 的所有 NDB 版本) |
| `--print-defaults` | 打印程序参数列表并退出 | (适用于基于 MySQL 8.0 的所有 NDB 版本) |
| `--promote-attributes`,`-A` | 允许将属性数据转换为更大类型 | (适用于基于 MySQL 8.0 的所有 NDB 版本) |
| `[--staging-tries=x[,y[,z]]](mysql-cluster-programs-ndb-move-data.html#option_ndb_move_data_staging-tries)` | 指定临时错误的尝试次数；格式为 x[,y[,z]]，其中 x=最大尝试次数（0=无限制），y=最小延迟（毫秒），z=最大延迟（毫秒） | (适用于基于 MySQL 8.0 的所有 NDB 版本) |
| `--usage`,`-?` | 显示帮助文本并退出；与 --help 相同 | (适用于基于 MySQL 8.0 的所有 NDB 版本) |
| `--verbose` | 启用详细消息 | (适用于基于 MySQL 8.0 的所有 NDB 版本) |
| `--version`,`-V` | 显示版本信息并退出 | (适用于基于 MySQL 8.0 的所有 NDB 版本) |
| 格式 | 描述 | 添加、弃用或移除 |

+   `--abort-on-error`

    | 命令行格式 | `--abort-on-error` |
    | --- | --- |

    在永久错误时转储核心（调试选项）。

+   `--character-sets-dir`=*`name`*

    | 命令行格式 | `--character-sets-dir=path` |
    | --- | --- |
    | 移除 | 8.0.31 |
    | 类型 | 字符串 |
    | 默认值 | `[none]` |

    存放字符集的目录。

+   `--connect-retry-delay`

    | 命令行格式 | `--connect-retry-delay=#` |
    | --- | --- |
    | 类型 | 整数 |
    | 默认值 | `5` |
    | 最小值 | `0` |
    | 最大值 | `5` |

    尝试联系管理服务器之间等待的秒数。

+   `--connect-retries`

    | 命令行格式 | `--connect-retries=#` |
    | --- | --- |
    | 类型 | 整数 |
    | 默认值 | `12` |
    | 最小值 | `0` |
    | 最大值 | `12` |

    连接失败前重试的次数。

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

+   `--database`=*`dbname`*, `-d`

    | 命令行格式 | `--database=name` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `TEST_DB` |

    表所在的数据库名称。

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

    还读取具有 concat(group, suffix)的组。

+   `--drop-source`

    | 命令行格式 | `--drop-source` |
    | --- | --- |

    在移动所有行后删除源表。

+   `--error-insert`

    | 命令行格式 | `--error-insert` |
    | --- | --- |

    插入随机临时错误（测试选项）。

+   `--exclude-missing-columns`

    | 命令行格式 | `--exclude-missing-columns` |
    | --- | --- |

    忽略源表或目标表中的额外列。

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

+   `--lossy-conversions`, `-l`

    | 命令行格式 | `--lossy-conversions` |
    | --- | --- |

    允许属性数据在转换为较小类型时被截断。

+   `--ndb-connectstring`

    | 命令行格式 | `--ndb-connectstring=connection_string` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `[none]` |

    设置用于连接到 ndb_mgmd 的连接字符串。语法：“[nodeid=id;][host=]hostname[:port]”。覆盖 NDB_CONNECTSTRING 和 my.cnf 中的条目。

+   `--ndb-mgmd-host`

    | 命令行格式 | `--ndb-mgmd-host=connection_string` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `[none]` |

    与`--ndb-connectstring`相同。

+   `--ndb-nodeid`

    | 命令行格式 | `--ndb-nodeid=#` |
    | --- | --- |
    | 类型 | 整数 |
    | 默认值 | `[none]` |

    为此节点设置节点 ID，覆盖由`--ndb-connectstring`设置的任何 ID。

+   `--ndb-optimized-node-selection`

    | 命令行格式 | `--ndb-optimized-node-selection` |
    | --- | --- |
    | 已移除 | 8.0.31 |

    启用优化以选择事务节点。默认启用；使用`--skip-ndb-optimized-node-selection`来禁用。

+   `--no-defaults`

    | 命令行格式 | `--no-defaults` |
    | --- | --- |

    不要从除登录文件以外的任何选项文件中读取默认选项。

+   `--print-defaults`

    | 命令行格式 | `--print-defaults` |
    | --- | --- |

    打印程序参数列表并退出。

+   `--promote-attributes`, `-A`

    | 命令行格式 | `--promote-attributes` |
    | --- | --- |

    允许将属性数据转换为更大的类型。

+   `--staging-tries`=*`x[,y[,z]]`*

    | 命令行格式 | `--staging-tries=x[,y[,z]]` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `0,1000,60000` |

    指定在临时错误上的尝试次数。格式为 x[,y[,z]]，其中 x=最大尝试次数（0=无限制），y=最小延迟（毫秒），z=最大延迟（毫秒）。

+   `--usage`

    | 命令行格式 | `--usage` |
    | --- | --- |

    显示帮助文本并退出；与`--help`相同。

+   `--verbose`

    | 命令行格式 | `--verbose` |
    | --- | --- |

    启用详细消息。

+   `--version`

    | 命令行格式 | `--version` |
    | --- | --- |

    显示版本信息并退出。
