# 25.5.18 ndb_print_file — 打印 NDB 磁盘数据文件内容

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-print-file.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-print-file.html)

**ndb_print_file** 从 NDB Cluster 磁盘数据文件中获取信息。

#### 用法

```sql
ndb_print_file [-v] [-q] *file_name*+
```

*`file_name`* 是 NDB Cluster 磁盘数据文件的名称。可以接受多个文件名，用空格分隔。

与**ndb_print_schema_file**和**ndb_print_sys_file**（与大多数旨在在管理服务器主机上运行或连接到管理服务器的其他`NDB`实用程序不同）不同，**ndb_print_file**必须在 NDB Cluster 数据节点上运行，因为它直接访问数据节点文件系统。由于它不使用管理服务器，因此即使管理服务器未运行，甚至集群已完全关闭，也可以使用此实用程序。

#### 选项

**表 25.40 与程序 ndb_print_file 一起使用的命令行选项**

| 格式 | 描述 | 新增、弃用或移除 |
| --- | --- | --- |
| `--file-key=hex_data`,`-K hex_data` | 通过标准输入、tty 或 my.cnf 文件提供加密密钥 | 新增：NDB 8.0.31 |
| `--file-key-from-stdin` | 通过标准输入提供加密密钥 | 新增：NDB 8.0.31 |
| `--help`,`-?` | 显示帮助文本并退出；与--usage 相同 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--quiet`,`-q` | 减少输出的冗长性 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--usage`,`-?` | 显示帮助文本并退出；与--help 相同 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--verbose`,`-v` | 增加输出的详细程度 | (适用于基于 MySQL 8.0 的所有 NDB 版本) |
| `--version`,`-V` | 显示版本信息并退出 | (适用于基于 MySQL 8.0 的所有 NDB 版本) |

**ndb_print_file** 支持以下选项:

+   `--file-key`, `-K`

    | 命令行格式 | `--file-key=hex_data` |
    | --- | --- |
    | 引入版本 | 8.0.31-ndb-8.0.31 |

    从 `stdin`, `tty`, 或 `my.cnf` 文件提供文件系统加密或解密密钥。

+   `--file-key-from-stdin`

    | 命令行格式 | `--file-key-from-stdin` |
    | --- | --- |
    | 引入版本 | 8.0.31-ndb-8.0.31 |
    | 类型 | 布尔值 |
    | 默认数值 | `FALSE` |
    | 有效数值 | `TRUE` |

    从 `stdin` 提供文件系统加密或解密密钥。

+   `--help`, `-h`, `-?`

    | 命令行格式 | `--help` |
    | --- | --- |

    打印帮助信息并退出。

+   `--quiet`, `-q`

    | 命令行格式 | `--quiet` |
    | --- | --- |

    抑制输出 (静默模式)。

+   `--usage`, `-?`

    | 命令行格式 | `--usage` |
    | --- | --- |

    打印帮助信息并退出。

+   `--verbose`, `-v`

    | 命令行格式 | `--verbose` |
    | --- | --- |

    使输出详细。

+   `--version`, `-v`

    | 命令行格式 | `--version` |
    | --- | --- |

    打印版本信息并退出。

更多信息，请参见 第 25.6.11 节, “NDB 集群磁盘数据表”。
