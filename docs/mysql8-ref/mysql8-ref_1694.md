# 25.5.14 ndb_index_stat — NDB 索引统计实用程序

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-index-stat.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-index-stat.html)

**ndb_index_stat**提供了关于`NDB`表上索引的每个片段的统计信息。这包括缓存版本和年龄，每个分区的索引条目数，以及索引的内存消耗。

#### 用法

要获取关于给定`NDB`表的基本索引统计信息，请按照这里显示的方式调用**ndb_index_stat**，将表名作为第一个参数，并紧接着指定包含此表的数据库的名称，使用`--database`（`-d`）选项：

```sql
ndb_index_stat *table* -d *database*
```

在这个例子中，我们使用**ndb_index_stat**来获取关于`test`数据库中名为`mytable`的`NDB`表的信息：

```sql
$> ndb_index_stat -d test mytable
table:City index:PRIMARY fragCount:2
sampleVersion:3 loadTime:1399585986 sampleCount:1994 keyBytes:7976
query cache: valid:1 sampleCount:1994 totalBytes:27916
times in ms: save: 7.133 sort: 1.974 sort per sample: 0.000
```

`sampleVersion`是取得统计数据的缓存的版本号。使用`--update`选项运行**ndb_index_stat**会导致 sampleVersion 递增。

`loadTime`显示了缓存上次更新的时间。这是自 Unix 纪元以来的秒数。

`sampleCount`是每个分区找到的索引条目数。您可以通过将此乘以片段数（显示为`fragCount`）来估算总条目数。

`sampleCount`可以与`SHOW INDEX`或`INFORMATION_SCHEMA.STATISTICS`的基数进行比较，尽管后两者提供了整个表的视图，而**ndb_index_stat**提供了每个片段的平均值。

`keyBytes`是索引使用的字节数。在这个例子中，主键是一个整数，每个索引需要四个字节，所以在这种情况下可以按照这里显示的方式计算`keyBytes`：

```sql
 keyBytes = sampleCount * (4 bytes per index) = 1994 * 4 = 7976
```

这些信息也可以使用`INFORMATION_SCHEMA.COLUMNS`中的相应列定义来获取（这需要一个 MySQL 服务器和一个 MySQL 客户端应用程序）。

`totalBytes`是表上所有索引消耗的总内存，以字节为单位。

前面示例中显示的时间是每次调用**ndb_index_stat**的特定时间。

`--verbose`选项提供一些额外的输出，如下所示：

```sql
$> ndb_index_stat -d test mytable --verbose
random seed 1337010518
connected
loop 1 of 1
table:mytable index:PRIMARY fragCount:4
sampleVersion:2 loadTime:1336751773 sampleCount:0 keyBytes:0
read stats
query cache created
query cache: valid:1 sampleCount:0 totalBytes:0
times in ms: save: 20.766 sort: 0.001
disconnected

$>
```

如果程序的输出为空，这可能表示尚无统计数据。要强制创建它们（或更新如果它们已经存在），请使用`--update`选项调用**ndb_index_stat**，或在**mysql**客户端中对表执行`ANALYZE TABLE`。

#### 选项

以下表格包括特定于 NDB Cluster **ndb_index_stat** 实用程序的选项。表后列出了附加描述。

**表 25.36 与程序 ndb_index_stat 一起使用的命令行选项**

| 格式 | 描述 | 添加、弃用���移除 |
| --- | --- | --- |
| `--character-sets-dir=path` | 包含字符集的目录 | 移除：8.0.31 |
| `--connect-retries=#` | 放弃之前重试连接的次数 | （支持所有基于 MySQL 8.0 的 NDB 版本） |
| `--connect-retry-delay=#` | 尝试联系管理服务器之间等待的秒数 | （支持所有基于 MySQL 8.0 的 NDB 版本） |
| `--connect-string=connection_string`,`-c connection_string` | 与--ndb-connectstring 相同 | （支持所有基于 MySQL 8.0 的 NDB 版本） |
| `--core-file` | 在错误时写入核心文件；用于调试 | 移除：8.0.31 |
| `--database=name`,`-d name` | 包含表的数据库名称 | （支持所有基于 MySQL 8.0 的 NDB 版本） |
| `--defaults-extra-file=path` | 在全局文件读取后读取给定文件 | （支持所有基于 MySQL 8.0 的 NDB 版本） |
| `--defaults-file=path` | 仅从给定文件中读取默认选项 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--defaults-group-suffix=string` | 也读取 concat(group, suffix) 组 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--delete` | 删除表的索引统计信息，停止之前配置的任何自动更新 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--dump` | 打印查询缓存 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--help`,`-?` | 显示帮助文本并退出 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--login-path=path` | 从登录文件中读取给定路径 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--loops=#` | 设置执行给定命令的次数；默认为 0 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--ndb-connectstring=connection_string`,`-c connection_string` | 设置连接到 ndb_mgmd 的��接字符串。语法："[nodeid=id;][host=]hostname[:port]"。覆盖 NDB_CONNECTSTRING 和 my.cnf 中的条目 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--ndb-mgmd-host=connection_string`,`-c connection_string` | 与 --ndb-connectstring 相同 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--ndb-nodeid=#` | 为此节点设置节点 ID，覆盖 --ndb-connectstring 设置的任何 ID | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--ndb-optimized-node-selection` | 启用对事务节点选择的优化。默认启用；使用 --skip-ndb-optimized-node-selection 来禁用 | 移除：8.0.31 |
| `--no-defaults` | 不从登录文件以外的任何选项文件中读取默认选项 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--print-defaults` | 打印程序参数列表并退出 | (适用于基于 MySQL 8.0 的所有 NDB 版本) |
| `--query=#` | 在第一个键属性上执行随机范围查询（必须为无符号整数） | (适用于基于 MySQL 8.0 的所有 NDB 版本) |
| `--sys-drop` | 删除 NDB 内核中的所有统计表和事件（所有统计数据都将丢失） | (适用于基于 MySQL 8.0 的所有 NDB 版本) |
| `--sys-create` | 在 NDB 内核中创建所有统计表和事件，如果它们尚不存在 | (适用于基于 MySQL 8.0 的所有 NDB 版本) |
| `--sys-create-if-not-exist` | 在 NDB 内核中创建任何尚不存在的统计表和事件 | (适用于基于 MySQL 8.0 的所有 NDB 版本) |
| `--sys-create-if-not-valid` | 在 NDB 内核中创建任何尚不存在的统计表或事件，之前会删除任何无效的表或事件 | (适用于基于 MySQL 8.0 的所有 NDB 版本) |
| `--sys-check` | 验证 NDB 系统索引统计和事件表是否存在 | (适用于基于 MySQL 8.0 的所有 NDB 版本) |
| `--sys-skip-tables` | 不对表应用 sys-* 选项 | (适用于基于 MySQL 8.0 的所有 NDB 版本) |
| `--sys-skip-events` | 不对事件应用 sys-* 选项 | (适用于基于 MySQL 8.0 的所有 NDB 版本) |
| `--update` | 更新表的索引统计信息，重新启动之前配置的任何自动更新 | (适用于基于 MySQL 8.0 的所有 NDB 版本) |
| `--usage`,`-?` | 显示帮助文本并退出；与 --help 相同 | (适用于基于 MySQL 8.0 的所有 NDB 版本) |
| `--verbose`,`-v` | 打开详细输出 | (适用于基于 MySQL 8.0 的所有 NDB 版本) |
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

+   `--database=*`name`*`, `-d *`name`*`

    | 命令行格式 | `--database=name` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `[none]` |
    | 最小值 |  |
    | 最大值 |  |

    包含被查询表的数据库的名称。

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

    还读取连接组与连接后缀的连接。

+   `--delete`

    | 命令行格式 | `--delete` |
    | --- | --- |

    删除给定表的索引统计信息，停止先前配置的任何自动更新。

+   `--dump`

    | 命令行格式 | `--dump` |
    | --- | --- |

    转储查询缓存的内容。

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

+   `--loops=*`#`*`

    | 命令行格式 | `--loops=#` |
    | --- | --- |
    | 类型 | 数字 |
    | 默认值 | `0` |
    | 最小值 | `0` |
    | 最大值 | `MAX_INT` |

    重复命令此次数（用于测试）。

+   `--ndb-connectstring`

    | 命令行格式 | `--ndb-connectstring=connection_string` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `[none]` |

    设置用于连接到 ndb_mgmd 的连接字符串。语法："[nodeid=id;][host=]hostname[:port]"。覆盖 NDB_CONNECTSTRING 和 my.cnf 中的条目。

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

    设置此节点的节点 ID，覆盖由`--ndb-connectstring`设置的任何 ID。

+   `--ndb-optimized-node-selection`

    | 命令行格式 | `--ndb-optimized-node-selection` |
    | --- | --- |
    | 已移除 | 8.0.31 |

    启用优化以选择事务节点。默认启用；使用`--skip-ndb-optimized-node-selection`来禁用。

+   `--no-defaults`

    | 命令��格式 | `--no-defaults` |
    | --- | --- |

    不要从除登录文件以外的任何选项文件中读取默认选项。

+   `--print-defaults`

    | 命令行格式 | `--print-defaults` |
    | --- | --- |

    打印程序参数列表并退出。

+   `--query=*`#`*`

    | 命令行格式 | `--query=#` |
    | --- | --- |
    | 类型 | 数字 |
    | 默认值 | `0` |
    | 最小值 | `0` |
    | 最大值 | `MAX_INT` |

    对第一个键属性执行随机范围查询（必须为 int unsigned）。

+   `--sys-drop`

    | 命令行格式 | `--sys-drop` |
    | --- | --- |

    删除 NDB 内核中的所有统计表和事件。*这将导致所有统计数据丢失*。

+   `--sys-create`

    | 命令行格式 | `--sys-create` |
    | --- | --- |

    在 NDB 内核中创建所有统计表和事件。仅当它们之前不存在时才有效。

+   `--sys-create-if-not-exist`

    | 命令行格式 | `--sys-create-if-not-exist` |
    | --- | --- |

    在调用程序时创建任何尚不存在的 NDB 系统统计表或事件（或两者）。

+   `--sys-create-if-not-valid`

    | 命令行格式 | `--sys-create-if-not-valid` |
    | --- | --- |

    在删除任何无效的统计表后，创建任何尚不存在的 NDB 系统统计表或事件。

+   `--sys-check`

    | 命令行格式 | `--sys-check` |
    | --- | --- |

    验证 NDB 内核中是否存在所有必需的系统统计表和事件。

+   `--sys-skip-tables`

    | 命令行格式 | `--sys-skip-tables` |
    | --- | --- |

    不要将任何`--sys-*`选项应用于任何统计表。

+   `--sys-skip-events`

    | 命令行格式 | `--sys-skip-events` |
    | --- | --- |

    不要将任何`--sys-*`选项应用于任何事件。

+   `--update`

    | 命令行格式 | `--update` |
    | --- | --- |

    更新给定表的索引统计信息，并重新启动先前配置的任何自动更新。

+   `--usage`

    | 命令行格式 | `--usage` |
    | --- | --- |

    显示帮助文本并退出；与`--help`相同。

+   `--verbose`

    | 命令行格式 | `--verbose` |
    | --- | --- |

    打开详细输出。

+   `--version`

    | 命令行格式 | `--version` |
    | --- | --- |

    显示版本信息并退出。

**ndb_index_stat 系统选项。** 以下选项用于在 NDB 内核中生成和更新统计表。这些选项中的任何一个都不能与统计选项混合使用（参见 ndb_index_stat 统计选项）。

+   `--sys-drop`

+   `--sys-create`

+   `--sys-create-if-not-exist`

+   `--sys-create-if-not-valid`

+   `--sys-check`

+   `--sys-skip-tables`

+   `--sys-skip-events`

**ndb_index_stat 统计选项。** 这里列出的选项用于生成索引统计信息。它们与给定的表和数据库一起工作。它们不能与系统选项混合使用（参见 ndb_index_stat 系统选项）。

+   `--database`

+   `--delete`

+   `--update`

+   `--dump`

+   `--query`
