# 25.5.17 ndb_print_backup_file — 打印 NDB 备份文件内容

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-print-backup-file.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-print-backup-file.html)

**ndb_print_backup_file** 从集群备份文件中获取诊断信息。

**表 25.39 与程序 ndb_print_backup_file 一起使用的命令行选项**

| 格式 | 描述 | 新增、弃用或移除 |
| --- | --- | --- |
| `--backup-key=key`,`-K password` | 使用此密码解密文件 | 新增: NDB 8.0.31 |
| `--backup-key-from-stdin` | 从 STDIN 安全获取解密密钥 | 新增: NDB 8.0.31 |
| `--backup-password=password`,`-P password` | 使用此密码解密文件 | 新增: NDB 8.0.22 |
| `--backup-password-from-stdin` | 从 STDIN 安全获取解密密码 | 新增: NDB 8.0.24 |
| `--control-directory-number=#`,`-c #` | 控制目录编号 | 新增: NDB 8.0.24 |
| `--defaults-extra-file=path` | 在全局文件读取后读取给定文件 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--defaults-file=path` | 仅从给定文件中读取默认选项 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--defaults-group-suffix=string` | 也读取 concat(group, suffix) 组 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--fragment-id=#`,`-f #` | 片段 ID | 新增: NDB 8.0.24 |
| `--help`,`--usage`,`-h`,`-?` | 打印使用信息 | 新增：NDB 8.0.24 |
| `--login-path=path` | 从登录文件中读取给定路径 | (所有基于 MySQL 8.0 的 NDB 版本都支持) |
| `--no-defaults` | 不从登录文件以外的任何选项文件中读取默认选项 | (所有基于 MySQL 8.0 的 NDB 版本都支持) |
| `--no-print-rows`,`-u` | 不打印行 | 新增：NDB 8.0.24 |
| `--print-defaults` | 打印程序参数列表并退出 | (所有基于 MySQL 8.0 的 NDB 版本都支持) |
| `--print-header-words`,`-h` | 打印标题词 | 新增：NDB 8.0.24 |
| `--print-restored-rows` | 打印恢复的行 | 新增：NDB 8.0.24 |
| `--print-rows`,`-U` | 打印行。默认启用；使用 --no-print-rows 禁用 | 新增：NDB 8.0.24 |
| `--print-rows-per-page` | 每页打印行数 | 新增：NDB 8.0.24 |
| `--rowid-file=path`,`-n path` | 包含要检查的行 ID 的文件 | 新增：NDB 8.0.24 |
| `--show-ignored-rows`,`-i` | 显示被忽略的行 | 新增：NDB 8.0.24 |
| `--table-id=#`,`-t #` | 表 ID；与--print-restored rows 一起使用 | 添加：NDB 8.0.24 |
| `--usage`,`-?` | 显示帮助文本并退出；与--help 相同 | （基于 MySQL 8.0 的所有 NDB 版本都支持） |
| `[--verbose[=#]](mysql-cluster-programs-ndb-print-backup-file.html#option_ndb_print_backup_file_verbose)`,`-v` | 详细级别 | 添加：NDB 8.0.24 |
| `--version`,`-V` | 显示版本信息并退出 | （基于 MySQL 8.0 的所有 NDB 版本都支持） |
| 格式 | 描述 | 添加、弃用或移除 |

#### 用法

```sql
ndb_print_backup_file [-P *password*] *file_name*
```

*`file_name`* 是集群备份文件的名称。这可以是集群备份目录中找到的任何文件（`.Data`、`.ctl`或`.log`文件）。这些文件位于数据节点的备份目录中的子目录`BACKUP-*`#`*`下，其中*`#`*是备份的序列号。有关集群备份文件及其内容的更多信息，请参见第 25.6.8.1 节，“NDB 集群备份概念”。

与**ndb_print_schema_file**和**ndb_print_sys_file**（以及大多数旨在在管理服务器主机上运行或连接到管理服务器的其他`NDB`实用程序不同）**ndb_print_backup_file**必须在集群数据节点上运行，因为它直接访问数据节点文件系统。由于它不使用管理服务器，因此即使管理服务器未运行，甚至当集群完全关闭时，也可以使用此实用程序。

在 NDB 8.0 中，此程序还可用于读取撤销日志文件。

#### 选项

在 NDB 8.0.24 之前，**ndb_print_backup_file**仅支持`-P`选项。从 NDB 8.0.24���始，该程序支持许多选项，这些选项在以下列表中描述。

+   `--backup-key`, `-K`

    | 命令行格式 | `--backup-key=key` |
    | --- | --- |
    | 引入版本 | 8.0.31-ndb-8.0.31 |

    指定解密加密备份所需的密钥。

+   `--backup-key-from-stdin`

    | 命令行格式 | `--backup-key-from-stdin` |
    | --- | --- |
    | 引入版本 | 8.0.31-ndb-8.0.31 |

    允许从标准输入输入解密密钥，类似于在调用**mysql**时不提供密码后输入密码。

+   `--backup-password`

    | 命令行格式 | `--backup-password=password` |
    | --- | --- |
    | 引入版本 | 8.0.22-ndb-8.0.22 |
    | 类型 | 字符串 |
    | 默认值 | `[无]` |

    指定解密加密备份所需的密码。

    从 NDB 8.0.24 开始提供此选项的长格式。

+   `--backup-password-from-stdin`

    | 命令行格式 | `--backup-password-from-stdin` |
    | --- | --- |
    | 引入版本 | 8.0.24-ndb-8.0.24 |

    允许从标准输入输入密码，类似于在调用**mysql**时不提供密码后输入密码。

+   `--control-directory-number`

    | 命令行格式 | `--control-directory-number=#` |
    | --- | --- |
    | 引入版本 | 8.0.24-ndb-8.0.24 |
    | 类型 | 整数 |
    | 默认值 | `0` |

    控制文件目录编号。与`--print-restored-rows`一起使用。

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

    仅从给定文件读取默认选项。

+   `--defaults-group-suffix`

    | 命令行格式 | `--defaults-group-suffix=string` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `[无]` |

    还读取带有 concat(group, suffix)的组。

+   `--fragment-id`

    | 命令行格式 | `--fragment-id=#` |
    | --- | --- |
    | 引入版本 | 8.0.24-ndb-8.0.24 |
    | 类型 | 整数 |
    | 默认值 | `0` |

    片段 ID。与`--print-restored-rows`一起使用。

+   `--help`

    | 命令行格式 | `--help``--usage` |
    | --- | --- |
    | 引入版本 | 8.0.24-ndb-8.0.24 |

    打印程序使用信息。

+   `--login-path`

    | 命令行格式 | `--login-path=path` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `[无]` |

    从登录文件中读取给定路径。

+   `--no-defaults`

    | 命令行格式 | `--no-defaults` |
    | --- | --- |

    不要从除登录文件之外的任何选项文件中读取默认选项。

+   `--no-print-rows`

    | 命令行格式 | `--no-print-rows` |
    | --- | --- |
    | 引入版本 | 8.0.24-ndb-8.0.24 |

    不在输出中包含行。

+   `--print-defaults`

    | 命令行格式 | `--print-defaults` |
    | --- | --- |

    打印程序参数列表并退出。

+   `--print-header-words`

    | 命令行格式 | `--print-header-words` |
    | --- | --- |
    | 引入版本 | 8.0.24-ndb-8.0.24 |

    在输出中包含标题词。

+   `--print-restored-rows`

    | 命令行格式 | `--print-restored-rows` |
    | --- | --- |
    | 引入版本 | 8.0.24-ndb-8.0.24 |

    在输出中包含恢复的行，使用文件 `LCP/*`c`*/T*`t`*F*`f`*.ctl`，其值设置如下：

    +   *`c`* 是使用`--control-directory-number`设置的控制文件编号。

    +   *`t`* 是使用`--table-id`设置的表 ID。

    +   *`f`* 是使用`--fragment-id`设置的片段 ID。

+   `--print-rows`

    | 命令行格式 | `--print-rows` |
    | --- | --- |
    | 引入版本 | 8.0.24-ndb-8.0.24 |

    打印行。此选项默认启用；要禁用它，请使用`--no-print-rows`。

+   `--print-rows-per-page`

    | 命令行格式 | `--print-rows-per-page` |
    | --- | --- |
    | 引入版本 | 8.0.24-ndb-8.0.24 |

    打印每页行数。

+   `--rowid-file`

    | 命令行格式 | `--rowid-file=path` |
    | --- | --- |
    | 引入版本 | 8.0.24-ndb-8.0.24 |
    | 类型 | 文件名 |
    | 默认值 | `[none]` |

    用于检查行 ID 的文件。

+   `--show-ignored-rows`

    | 命令行格式 | `--show-ignored-rows` |
    | --- | --- |
    | 引入版本 | 8.0.24-ndb-8.0.24 |

    显示被忽略的行。

+   `--table-id`

    | 命令行格式 | `--table-id=#` |
    | --- | --- |
    | 引入版本 | 8.0.24-ndb-8.0.24 |
    | 类型 | 整数 |
    | 默认值 | `[none]` |

    表 ID。与`--print-restored-rows`一起使用。

+   `--usage`

    | 命令行格式 | `--usage` |
    | --- | --- |

    显示帮助文本并退出；与`--help`相同。

+   `--verbose`

    | 命令行格式 | `--verbose[=#]` |
    | --- | --- |
    | 引入版本 | 8.0.24-ndb-8.0.24 |
    | 类型 | 整数 |
    | 默认值 | `0` |

    输出的详细程度。较大的值表示增加的详细程度。

+   `--version`

    | 命令行格式 | `--version` |
    | --- | --- |

    显示版本信息并退出。
