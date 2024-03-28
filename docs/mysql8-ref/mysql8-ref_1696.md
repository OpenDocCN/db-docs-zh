# 25.5.16 ndb_perror — Obtain NDB Error Message Information

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-perror.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-perror.html)

**ndb_perror**显示有关 NDB 错误的信息，给定其错误代码。这包括错误消息、错误类型以及错误是永久的还是临时的。这旨在作为**perror** `--ndb`的替代品，后者不再受支持。

#### 用法

```sql
ndb_perror [*options*] *error_code*
```

**ndb_perror**不需要访问运行中的 NDB 集群或任何节点（包括 SQL 节点）。要查看有关给定 NDB 错误的信息，请调用该程序，并将错误代码作为参数，如下所示：

```sql
$> ndb_perror 323
NDB error code 323: Invalid nodegroup id, nodegroup already existing: Permanent error: Application error
```

为了仅显示错误消息，请使用带有`--silent`选项（简写为`-s`）调用**ndb_perror**，如下所示：

```sql
$> ndb_perror -s 323
Invalid nodegroup id, nodegroup already existing: Permanent error: Application error
```

与**perror**类似，**ndb_perror**接受多个错误代码：

```sql
$> ndb_perror 321 1001
NDB error code 321: Invalid nodegroup id: Permanent error: Application error
NDB error code 1001: Illegal connect string
```

**ndb_perror**的其他程序选项将在本节后面描述。

**ndb_perror**替换了不再受 NDB Cluster 支持的**perror** `--ndb`。为了在脚本和其他可能依赖于**perror**获取 NDB 错误信息的应用程序中更容易进行替换，**ndb_perror**支持自己的“虚拟”`--ndb`选项，不执行任何操作。

以下表格包括所有特定于 NDB Cluster 程序**ndb_perror**的选项。表格后面是附加描述。

**表 25.38 与程序 ndb_perror 一起使用的命令行选项**

| 格式 | 描述 | 添加、弃用或移除 |
| --- | --- | --- |
| `--defaults-extra-file=path` | 在读取全局文件后读取给定文件 | (适用于基于 MySQL 8.0 的所有 NDB 版本) |
| `--defaults-file=path` | 仅从给定文件中读取默认选项 | (适用于基于 MySQL 8.0 的所有 NDB 版本) |
| `--defaults-group-suffix=string` | 也读取带有连接（group, suffix）的组 | (适用于基于 MySQL 8.0 的所有 NDB 版本) |
| `--help`,`-?` | 显示帮助文本 | (适用于基于 MySQL 8.0 的所有 NDB 版本) |
| `--login-path=path` | 从登录文件中读取给定路径 | (适用于基于 MySQL 8.0 的所有 NDB 版本) |
| `--ndb` | 为依赖于旧版本 perror 的应用程序兼容性而存在；无任何作用 | (适用于基于 MySQL 8.0 的所有 NDB 版本) |
| `--no-defaults` | 除登录文件外不从任何选项文件中读取默认选项 | (适用于基于 MySQL 8.0 的所有 NDB 版本) |
| `--print-defaults` | 打印程序参数列表并退出 | (适用于基于 MySQL 8.0 的所有 NDB 版本) |
| `--silent`,`-s` | 仅显示错误消息 | (适用于基于 MySQL 8.0 的所有 NDB 版本) |
| `--version`,`-V` | 打印程序版本信息并退出 | (适用于基于 MySQL 8.0 的所有 NDB 版本) |
| `--verbose`,`-v` | 冗长输出；使用 --silent 禁用 | (适用于基于 MySQL 8.0 的所有 NDB 版本) |
| 格式 | 描述 | 添加、弃用或移除 |

#### 附加选项

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

    还读取连接(group, suffix)的组。

+   `--help`, `-?`

    | 命令行格式 | `--help` |
    | --- | --- |

    显示程序帮助文本并退出。

+   `--login-path`

    | 命令行格式 | `--login-path=path` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `[none]` |

    从登录文件中读取给定路径。

+   `--ndb`

    | 命令行格式 | `--ndb` |
    | --- | --- |

    为了与依赖于旧版本**perror**的应用程序兼容，该程序使用了该程序的`--ndb`选项。当与**ndb_perror**一起使用时，该选项不起作用，并被忽略。

+   `--no-defaults`

    | 命令行格式 | `--no-defaults` |
    | --- | --- |

    不从登录文件以外的任何选项文件中读取默认选项。

+   `--print-defaults`

    | 命令行格式 | `--print-defaults` |
    | --- | --- |

    打印程序参数列表并退出。

+   `--silent`, `-s`

    | 命令行格式 | `--silent` |
    | --- | --- |

    仅显示错误消息。

+   `--version`, `-V`

    | 命令行格式 | `--version` |
    | --- | --- |

    打印程序版本信息并退出。

+   `--verbose`, `-v`

    | 命令行格式 | `--verbose` |
    | --- | --- |

    冗长输出；使用`--silent`禁用。
