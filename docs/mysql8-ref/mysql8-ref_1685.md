# 25.5.5 ndb_mgm — The NDB Cluster Management Client

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-mgm.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-mgm.html)

**ndb_mgm** 管理客户端进程实际上并不需要运行集群。它的价值在于提供一组命令来检查集群的状态、启动备份以及执行其他管理功能。管理客户端使用 C API 访问管理服务器。高级用户还可以使用此 API 编写专用管理进程，执行类似于 **ndb_mgm** 执行的任务。

要启动管理客户端，需要提供管理服务器的主机名和端口号：

```sql
$> ndb_mgm [*host_name* [*port_num*]]
```

例如：

```sql
$> ndb_mgm ndb_mgmd.mysql.com 1186
```

默认主机名和端口号分别为 `localhost` 和 1186。

所有可与 **ndb_mgm** 一起使用的选项显示在以下表中。表后面是附加描述。

**表 25.27 与程序 ndb_mgm 一起使用的命令行选项**

| 格式 | 描述 | 添加、弃用或移除 |
| --- | --- | --- |
| `--backup-password-from-stdin` | 以安全方式从 STDIN 获取解密密码；与 --execute 和 ndb_mgm START BACKUP 命令一起使用 | 添加：NDB 8.0.24 |
| `--character-sets-dir=path` | 包含字符集的目录 | 移除：8.0.31 |
| `--connect-retries=#` | 设置在放弃之前重试连接的次数；0 表示仅尝试 1 次（无重试） | （在基于 MySQL 8.0 的所有 NDB 版本中支持） |
| `--connect-retry-delay=#` | 尝试联系管理服务器之间等待的秒数 | （在基于 MySQL 8.0 的所有 NDB 版本中支持） |
| `--connect-string=connection_string`,`-c connection_string` | 与 --ndb-connectstring 相同 | （在基于 MySQL 8.0 的所有 NDB 版本中支持） |
| `--core-file` | 在错误时写入核心文件；用于调试 | 移除：8.0.31 |
| `--defaults-extra-file=path` | 在读取全局文件后读取给定文件 | （在基于 MySQL 8.0 的所有 NDB 版本中支持） |
| `--defaults-file=path` | 仅从给定文件中读取默认选项 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--defaults-group-suffix=string` | 也读取具有 concat(group, suffix) 的组 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--encrypt-backup` | 导致 START BACKUP 在进行备份时进行加密，如果用户未提供密码，则提示输入 | 添加：NDB 8.0.24 |
| `--execute=command`,`-e command` | 执行命令并退出 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--help`,`-?` | 显示帮助文本并退出 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--login-path=path` | 从登录文件中读取给定路径 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--ndb-connectstring=connection_string`,`-c connection_string` | 设置用于连接到 ndb_mgmd 的连接字符串。语法："[nodeid=id;][host=]hostname[:port]"。覆盖 NDB_CONNECTSTRING 和 my.cnf 中的条目 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--ndb-mgmd-host=connection_string`,`-c connection_string` | 与 --ndb-connectstring 相同 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--ndb-nodeid=#` | 为此节点设置节点 ID，覆盖 --ndb-connectstring 设置的任何 ID | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--ndb-optimized-node-selection` | 启用优化以选择事务节点。默认启用；使用 --skip-ndb-optimized-node-selection 来禁用 | 移除：8.0.31 |
| `--no-defaults` | 不从登录文件以外的任何选项文件中读取默认选项 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--print-defaults` | 打印程序参数列表并退出 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--try-reconnect=#`,`-t #` | 设置在放弃之前重试连接的次数；与 --connect-retries 同义词 | (适用于基于 MySQL 8.0 的所有 NDB 版本) |
| `--usage`,`-?` | 显示帮助文本并退出；与 --help 相同 | (适用于基于 MySQL 8.0 的所有 NDB 版本) |
| `--version`,`-V` | 显示版本信息并退出 | (适用于基于 MySQL 8.0 的所有 NDB 版本) |
| 格式 | 描述 | 添加、弃用或移除 |

+   [`--backup-password-from-stdin[=TRUE|FALSE]`](mysql-cluster-programs-ndb-mgm.html#option_ndb_mgm_backup-password-from-stdin)

    | 命令行格式 | `--backup-password-from-stdin` |
    | --- | --- |
    | 引入 | 8.0.24-ndb-8.0.24 |

    此选项允许在使用 `--execute "START BACKUP"` 或类似操作创建备份时，从系统 shell (`stdin`) 输入备份密码。使用此选项需要同时使用 `--execute`。

+   `--character-sets-dir`

    | 命令行格式 | `--character-sets-dir=path` |
    | --- | --- |
    | 移除 | 8.0.31 |

    包含字符集的目录。

+   `--connect-retries=*`#`*`

    | 命令行格式 | `--connect-retries=#` |
    | --- | --- |
    | 类型 | 数值 |
    | 默认值 | `3` |
    | 最小值 | `0` |
    | 最大值 | `4294967295` |

    此选项指定在第一次尝试后重试连接的次数，直到放弃（客户端始终至少尝试一次连接）。每次尝试等待的时间长度由 `--connect-retry-delay` 设置。

    此选项与现已弃用的 `--try-reconnect` 选项是同义词。

+   `--connect-retry-delay`

    | 命令行格式 | `--connect-retry-delay=#` |
    | --- | --- |
    | 类�� | 整数 |
    | 默认值 | `5` |
    | 最小值 | `0` |
    | 最大值 | `5` |

    尝试联系管理服务器之间等待的秒数。

+   `--connect-string`

    | 命令行格式 | `--connect-string=connection_string` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `[无]` |

    与 `--ndb-connectstring` 相同。

+   `--core-file`

    | 命令行格式 | `--core-file` |
    | --- | --- |
    | 已移除 | 8.0.31 |

    在错误时写入核心文件；用于调试。

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

    还读取带有连接（组，后缀）的组。

+   `--encrypt-backup`

    | 命令行格式 | `--encrypt-backup` |
    | --- | --- |
    | 引入版本 | 8.0.24-ndb-8.0.24 |

    当使用此选项时，将对所有备份进行加密。要使每次运行**ndb_mgm**时都发生这种情况，请将选项放在`my.cnf`文件的`[ndb_mgm]`部分中。

+   `--execute=`command``, `-e `command``

    | 命令行格式 | `--execute=command` |
    | --- | --- |

    此选项可用于从系统 shell 向 NDB Cluster 管理客户端发送命令。例如，以下任一命令等同于在管理客户端中执行`SHOW`：

    ```sql
    $> ndb_mgm -e "SHOW"

    $> ndb_mgm --execute="SHOW"
    ```

    这类似于在**mysql**命令行客户端中使用`--execute`或`-e`选项的方式。请参阅第 6.2.2.1 节，“在命令行上使用选项”。

    注意

    如果使用此选项传递的管理客户端命令包含任何空格字符，则命令*必须*用引号括起来。可以使用单���号或双引号。如果管理客户端命令不包含空格字符，则引号是可选的。

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

+   `--ndb-connectstring`

    | 命令行格式 | `--ndb-connectstring=connection_string` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `[none]` |

    为连接到**ndb_mgmd**设置连接字符串。语法：[`nodeid=*`id`*;`][`host=`]`*`hostname`*`[`:*`port`*`]. 覆盖`NDB_CONNECTSTRING`和`my.cnf`中的条目。

+   `--ndb-nodeid`

    | 命令行格式 | `--ndb-nodeid=#` |
    | --- | --- |
    | 类型 | 整数 |
    | 默认值 | `[none]` |

    为此节点设置节点 ID，覆盖`--ndb-connectstring`设置的任何 ID。

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

    启用事务节点选择的优化。默认情况下启用；使用`--skip-ndb-optimized-node-selection`来禁用。

+   `--no-defaults`

    | 命令行格式 | `--no-defaults` |
    | --- | --- |

    不要从登录文件以外的任何选项文件中读取默认选项。

+   `--print-defaults`

    | 命令行格式 | `--print-defaults` |
    | --- | --- |

    打印程序参数列表并退出。

+   `--try-reconnect=*`number`*`

    | 命令行格式 | `--try-reconnect=#` |
    | --- | --- |
    | 已弃用 | 是 |
    | 类型 | 数字 |
    | 类型 | 整数 |
    | 默认值 | `12` |
    | 默认值 | `3` |
    | 最小值 | `0` |
    | 最大值 | `4294967295` |

    如果与管理服务器的连接中断，节点将每 5 秒尝试重新连接，直到成功。通过使用此选项，可以在放弃并报告错误之前限制尝试次数为*`number`*。

    此选项已弃用，并可能在将来的版本中删除。请改用`--connect-retries`。

+   `--用法`

    | 命令行格式 | `--usage` |
    | --- | --- |

    显示帮助文本并退出；与`--help`相同。

+   `--version`

    | 命令行格式 | `--version` |
    | --- | --- |

    显示版本信息并退出。

有关如何使用**ndb_mgm**的其他信息，请参阅第 25.6.1 节，“NDB Cluster Management Client 中的命令”。
