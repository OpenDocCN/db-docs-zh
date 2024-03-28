# 25.5.12 ndb_error_reporter — NDB Error-Reporting Utility

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-error-reporter.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-error-reporter.html)

**ndb_error_reporter** 从数据节点和管理节点日志文件创建一个归档，可用于帮助诊断集群中的错误或其他问题。*强烈建议在提交 NDB Cluster 错误报告时使用此实用程序*。

可用于**ndb_error_reporter**的选项如下表所示。表后面是附加描述。

**表 25.34 与程序 ndb_error_reporter 一起使用的命令行选项**

| 格式 | 描述 | 添加、弃用或移除 |
| --- | --- | --- |
| `--connection-timeout=#` | 连接到节点时等待的秒数，超时前 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--dry-scp` | 禁用与远程主机的 scp；仅用于测试 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--fs` | 在错误报告中包含文件系统数据；可能使用大量磁盘空间 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--help`,`-?` | 显示帮助文本并退出 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--skip-nodegroup=#` | 跳过具有此 ID 的节点组中的所有节点 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |

#### 用法

```sql
ndb_error_reporter *path/to/config-file* [*username*] [*options*]
```

此实用程序旨在在管理节点主机上使用，并需要管理主机配置文件的路径（通常命名为`config.ini`）。可选地，您可以提供能够使用 SSH 访问集群数据节点的用户的名称，以复制数据节点日志文件。**ndb_error_reporter** 然后将所有这些文件包含在创建的归档中，该归档位于运行它的相同目录中。归档的名称为`ndb_error_report_*`YYYYMMDDhhmmss`*.tar.bz2`，其中*`YYYYMMDDhhmmss`*是日期时间字符串。

**ndb_error_reporter** 还接受以下列出的选项：

+   `--connection-timeout=*`timeout`*`

    | 命令行格式 | `--connection-timeout=#` |
    | --- | --- |
    | 类型 | 整数 |
    | 默认值 | `0` |

    在尝试连接节点时等待这么多秒，然后超时。

+   `--dry-scp`

    | 命令行格式 | `--dry-scp` |
    | --- | --- |

    运行**ndb_error_reporter**，不使用 scp 从远程主机。仅用于测试。

+   `--help`

    | 命令行格式 | `--help` |
    | --- | --- |

    显示帮助文本并退出。

+   `--fs`

    | 命令行格式 | `--fs` |
    | --- | --- |

    将数据节点文件系统复制到管理主机并包含在归档中。

    因为数据节点文件系统可能非常庞大，即使经过压缩，我们要求您请*不要*将使用此选项创建的归档发送给 Oracle，除非有明确要求。

+   `--skip-nodegroup=*`nodegroup_id`*`

    | 命令行格式 | `--connection-timeout=#` |
    | --- | --- |
    | 类型 | 整数 |
    | 默认值 | `0` |

    跳过所有属于具有提供的节点组 ID 的节点组的节点。
