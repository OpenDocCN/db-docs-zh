# 25.5.24 ndb_secretsfile_reader — 从加密的 NDB 数据文件中获取密钥信息

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-secretsfile-reader.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-secretsfile-reader.html)

**ndb_secretsfile_reader** 从 `NDB` 加密秘密文件中获取加密密钥，给定密码。

#### 用法

```sql
ndb_secretsfile_reader *options* *file*
```

*`选项`* 必须包括 `--filesystem-password` 或 `--filesystem-password-from-stdin` 之一，并且必须提供加密密码，如下所示：

```sql
> ndb_secretsfile_reader --filesystem-password=54kl14 ndb_5_fs/D1/NDBCNTR/S0.sysfile
ndb_secretsfile_reader: [Warning] Using a password on the command line interface can be insecure.
cac256e18b2ddf6b5ef82d99a72f18e864b78453cc7fa40bfaf0c40b91122d18
```

可与 **ndb_secretsfile_reader** 一起使用的其他选项显示在下表中。表后面有附加描述。

**表 25.45 与程序 ndb_secretsfile_reader 一起使用的命令行选项**

| 格式 | 描述 | 添加、弃用或移除 |
| --- | --- | --- |
| `--defaults-extra-file=path` | 在读取全局文件后读取给定文件 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--defaults-file=path` | 仅从给定文件中读取默认选项 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--defaults-group-suffix=string` | 还读取具有 concat(group, suffix) 的组 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--filesystem-password=password` | 节点文件系统加密的密码；可以从 stdin、tty 或 my.cnf 文件传递 | 添加: 8.0.31 |
| `--filesystem-password-from-stdin={TRUE&#124;FALSE}` | 从 stdin 获取加密密码 | 添加: 8.0.31 |
| `--help`,`-?` | 显示帮助文本并退出 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--login-path=path` | 从登录文件中读取���定路径 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--no-defaults` | 不从登录文件以外的任何选项文件中读取默认选项 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--print-defaults` | 打印程序参数列表并退出 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--usage`,`-?` | 显示帮助文本并退出；与 --help 相同 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--version`,`-V` | 显示版本信息并退出 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| 格式 | 描述 | 添加、弃用或移除 |

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

    还读取带有 concat(group, suffix) 的组。

+   `--filesystem-password`

    | 命令行格式 | `--filesystem-password=password` |
    | --- | --- |
    | 引入版本 | 8.0.31 |

    通过 `stdin`、`tty` 或 `my.cnf` 文件将文件系统加密和解密密码传递给**ndb_secretsfile_reader**。

+   `--filesystem-password-from-stdin`

    | 命令行格式 | `--filesystem-password-from-stdin={TRUE&#124;FALSE}` |
    | --- | --- |
    | 引入版本 | 8.0.31 |

    从 `stdin`（仅限）向 **ndb_secretsfile_reader** 传递文件系统加密和解密密码。

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

    显示帮助文本并退出；与 --help 相同。

+   `--version`

    | 命令行格式 | `--version` |
    | --- | --- |

    显示版本信息并退出。

**ndb_secretsfile_reader**是在 NDB 8.0.31 中添加的。
