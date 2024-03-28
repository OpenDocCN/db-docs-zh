# 25.5.22 ndb_redo_log_reader — 检查和打印集群重做日志的内容

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-redo-log-reader.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-redo-log-reader.html)

读取重做日志文件，检查其中是否有错误，以人类可读的格式打印其内容，或两者兼而有之。**ndb_redo_log_reader**主要供 NDB Cluster 开发人员和支持人员在调试和诊断问题时使用。

该实用程序仍在开发中，其语法和行为可能会在未来的 NDB Cluster 版本中发生变化。

**ndb_redo_log_reader**的 C++源文件可以在目录`/storage/ndb/src/kernel/blocks/dblqh/redoLogReader`中找到。

可用于**ndb_redo_log_reader**的选项如下表所示。表后附有附加描述。

**表 25.41 与程序 ndb_redo_log_reader 一起使用的命令行选项**

| 格式 | 描述 | 新增、弃用或移除 |
| --- | --- | --- |
| `-dump` | 打印转储信息 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--file-key=key`,`-K key` | 提供解密密钥 | 新增：NDB 8.0.31 |
| `--file-key-from-stdin` | 使用 stdin 提供解密密钥 | 新增：NDB 8.0.31 |
| `-filedescriptors` | 仅打印文件描述符 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--help` | 打印使用信息（无简短形式） | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `-lap` | 提供 lap 信息，包括最大 GCI 的开始和完成 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `-mbyte #` | 起始兆字节 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `-mbyteheaders` | 仅显示文件中每兆字节的第一页标题 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `-nocheck` | 不检查记录错误 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `-noprint` | 不打印记录 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `-page #` | 从此页开始 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `-pageheaders` | 仅显示页头 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `-pageindex #` | 从此页索引开始 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `-twiddle` | 位移转储 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| 格式 | 描述 | 添加、弃用或移除 |

#### 用法

```sql
ndb_redo_log_reader *file_name* [*options*]
```

*`file_name`* 是集群重做日志文件的名称。重做日志文件位于数据节点数据目录（`DataDir`）下的编号目录中；在此目录下到重做日志文件的路径与模式匹配为 `ndb_*`nodeid`*_fs/D*`#`*/DBLQH/S*`#`*.FragLog`。*`nodeid`* 是数据节点的节点 ID。*`#`* 的两个实例分别代表一个数字（不一定相同的数字）；`D` 后面的数字范围为 8-39 包括 8 和 39；`S` 后面的数字范围根据 `NoOfFragmentLogFiles` 配置参数的值而变化，其默认值为 16；因此，文件名中数字的默认范围为 0-15 包括 0 和 15。更多信息，请参阅 NDB Cluster Data Node File System Directory。

要读取的文件名后面可以跟随此处列出的一个或多个选项：

+   `-dump`

    | 命令行格式 | `-dump` |
    | --- | --- |

    打印转储信息。

+   `--file-key`, `-K`

    | 命令行格式 | `--file-key=key` |
    | --- | --- |
    | 引入版本 | 8.0.31-ndb-8.0.31 |

    使用`stdin`、`tty`或`my.cnf`文件提供文件解密密钥。

+   `--file-key-from-stdin`

    | 命令行格式 | `--file-key-from-stdin` |
    | --- | --- |
    | 引入版本 | 8.0.31-ndb-8.0.31 |

    使用`stdin`提供文件解密密钥。

+   | 命令行格式 | `-filedescriptors` |
+   | --- | --- |

    `-filedescriptors`: 仅打印文件描述符。

+   | 命令行格式 | `--help` |
+   | --- | --- |

    `--help`: 打印使用信息。

+   `-lap`

    | 命令行格式 | `-lap` |
    | --- | --- |

    提供 lap 信息，包括最大 GCI 的开始和完成。

+   | 命令行格式 | `-mbyte #` |
+   | --- | --- |
    | 类型 | 数值型 |
    | 默认值 | `0` |
    | 最小值 | `0` |
    | 最大值 | `15` |

    `-mbyte *`#`*`: 起始兆字节。

    *`#`* 是一个范围在 0 到 15 之间的整数。

+   | 命令行格式 | `-mbyteheaders` |
+   | --- | --- |

    `-mbyteheaders`: 仅显示文件中每个兆字节的第一页页眉。

+   | 命令行格式 | `-noprint` |
+   | --- | --- |

    `-noprint`: 不打印日志文件的内容。

+   | 命令行格式 | `-nocheck` |
+   | --- | --- |

    `-nocheck`: 不检查日志文件中的错误。

+   | 命令行格式 | `-page #` |
+   | --- | --- |
    | 类型 | 整数型 |
    | 默认值 | `0` |
    | 最小值 | `0` |
    | 最大值 | `31` |

    `-page *`#`*`: 从这一页开始。

    *`#`* 是一个范围在 0 到 31 之间的整数。

+   | 命令行格式 | `-pageheaders` |
+   | --- | --- |

    `-pageheaders`: 仅显示页眉。

+   | 命令行格式 | `-pageindex #` |
+   | --- | --- |
    | 类型 | 整数型 |
    | 默认值 | `12` |
    | 最小值 | `12` |
    | 最大值 | `8191` |

    `-pageindex *`#`*`: 从这个页面索引开始。

    *`#`* 是一个范围在 12 到 8191 之间的整数。

+   `-twiddle`

    | 命令行格式 | `-twiddle` |
    | --- | --- |

    位移转储。

像**ndb_print_backup_file**和**ndb_print_schema_file**（与大多数`NDB`实用程序不同，这些实用程序旨在在管理服务器主机上运行或连接到管理服务器）**ndb_redo_log_reader**必须在集群数据节点上运行，因为它直接访问数据节点文件系统。由于它不使用管理服务器，因此即使管理服务器未运行，甚至在集群完全关闭时也可以使用此实用程序。
