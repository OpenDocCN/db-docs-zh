# 25.5.6 ndb_blob_tool — 检查和修复 NDB Cluster 表的 BLOB 和 TEXT 列

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-blob-tool.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-blob-tool.html)

这个工具可用于检查和删除`NDB`表中的孤立的 BLOB 列部分，以及生成列出任何孤立部分的文件。在诊断和修复包含 `BLOB` 或 `TEXT` 列的损坏或受损 `NDB` 表时，有时会很有用。

**ndb_blob_tool** 的基本语法如下所示：

```sql
ndb_blob_tool [*options*] *table* [*column*, ...]
```

除非使用 `--help` 选项，否则必须指定要执行的操作，包括一个或多个选项 `--check-orphans`、`--delete-orphans` 或 `--dump-file`。这些选项会导致 **ndb_blob_tool** 检查孤立��� BLOB 部分、删除任何孤立的 BLOB 部分，并生成列出孤立的 BLOB 部分的转储文件，分别在本节后面有更详细的描述。

在调用 **ndb_blob_tool** 时，您还必须指定表的名称。此外，您还可以选择在表名后面（逗号分隔）跟随该表的一个或多个 `BLOB` 或 `TEXT` 列的名称。如果未列出列，则该工具将处理表的所有 `BLOB` 和 `TEXT` 列。如果需要指定数据库，请使用 `--database` (`-d`) 选项。

`--verbose` 选项在输出中提供有关工具进度的附加信息。

所有可与 **ndb_mgmd** 一起使用的选项都显示在下表中。表后面会有附加描述。

**表 25.28 与程序 ndb_blob_tool 一起使用的命令行选项**

| 格式 | 描述 | 添加、弃用或删除 |
| --- | --- | --- |
| `--add-missing` | 写入虚拟 blob 部分以取代缺失的部分 | 添加: NDB 8.0.20 |
| `--character-sets-dir=path` | 包含字符集的目录 | 移除: 8.0.31 |
| `--check-missing` | 检查具有内联部分但从部分表中缺少一个或多个部分的 blob | 添加: NDB 8.0.20 |
| `--check-orphans` | 检查没有对应内联部分的 blob 部分 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--connect-retries=#` | 放弃之前重试连接的次数 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--connect-retry-delay=#` | 重试联系管理服务器之间等待的秒数 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--connect-string=connection_string`,`-c connection_string` | 与 --ndb-connectstring 相同 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--core-file` | 在错误时写入核心文件；用于调试 | 移除: 8.0.31 |
| `--database=name`,`-d name` | 要在其中查找表的数据库 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--defaults-extra-file=path` | 在全局文件读取后读取给定文件 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--defaults-file=path` | 仅从给定文件中读取默认选项 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--defaults-group-suffix=string` | 也读取具有 concat(group, suffix) 的组 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--delete-orphans` | 删除没有对应内联部分的 blob 部分 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--dump-file=file` | 将孤立键写入指定文件 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--help`,`-?` | 显示帮助文本并退出 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--login-path=path` | 从登录文件中读取给定路径 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--ndb-connectstring=connection_string`,`-c connection_string` | 设置用于连接到 ndb_mgmd 的连接字符串。语法: "[nodeid=id;][host=]hostname[:port]"。覆盖 NDB_CONNECTSTRING 和 my.cnf 中的条目 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--ndb-mgmd-host=connection_string`,`-c connection_string` | 与 --ndb-connectstring 相同 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--ndb-nodeid=#` | 为此节点设置节点 ID，覆盖 --ndb-connectstring 设置的任何 ID | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--ndb-optimized-node-selection` | 启用用于事务节点选择的优化。默认启用；使用 --skip-ndb-optimized-node-selection 来禁用 | 已移除: 8.0.31 |
| `--no-defaults` | 不从除登录文件之外的任何选项文件中读取默认选项 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--print-defaults` | 打印程序参数列表并退出 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--usage`,`-?` | 显示帮助文本并退出；与 --help 相同 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--verbose`,`-v` | 详细输出 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--version`,`-V` | 显示版本信息并退出 | (支持所有基于 MySQL 8.0 的 NDB 版本) |
| 格式 | 描述 | 添加、弃用或移除 |

+   `--add-missing`

    | 命令行格式 | `--add-missing` |
    | --- | --- |
    | 引入版本 | 8.0.20-ndb-8.0.20 |

    对于 NDB Cluster 表中没有对应 BLOB 部分的每个内联部分，写入所需长度的虚拟 BLOB 部分，由空格组成。

+   `--character-sets-dir`

    | 命令行格式 | `--character-sets-dir=path` |
    | --- | --- |
    | 移除版本 | 8.0.31 |

    包含字符集的目录。

+   `--check-missing`

    | 命令行格式 | `--check-missing` |
    | --- | --- |
    | 引入版本 | 8.0.20-ndb-8.0.20 |

    检查 NDB Cluster 表中没有对应 BLOB 部分的内联部分。

+   `--check-orphans`

    | 命令行格式 | `--check-orphans` |
    | --- | --- |

    检查 NDB Cluster 表中没有对应内联部分的 BLOB 部分。

+   `--connect-retries`

    | 命令行格式 | `--connect-retries=#` |
    | --- | --- |
    | 类型 | 整数 |
    | 默认值 | `12` |
    | 最小值 | `0` |
    | 最大值 | `12` |

    放弃之前重试连接的次数。

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
    | 移除版本 | 8.0.31 |

    在出错时写入核心文件；用于调试。

+   `--database=*`db_name`*`, `-d`

    | 命令行格式 | `--database=name` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `[none]` |

    指定要在其中查找表的数据库。

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
    | 默认数值 | `[none]` |

    仅从给定文件中读取默认选项。

+   `--defaults-group-suffix`

    | 命令行格式 | `--defaults-group-suffix=string` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认数值 | `[none]` |

    也读取具有 concat(group, suffix) 的组。

+   `--delete-orphans`

    | 命令行格式 | `--delete-orphans` |
    | --- | --- |

    从 NDB Cluster 表中删除没有对应内联部分的 BLOB 部分。

+   `--dump-file=*`file`*`

    | 命令行格式 | `--dump-file=file` |
    | --- | --- |
    | 类型 | 文件名 |
    | 默认数值 | `[none]` |

    将孤立的 BLOB 列部分的列表写入 *`file`*。写入文件的信息包括每个孤立的 BLOB 部分的表键和部分编号。

+   `--help`

    | 命令行格式 | `--help` |
    | --- | --- |

    显示帮助文本并退出。

+   `--login-path`

    | 命令行格式 | `--login-path=path` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认数值 | `[none]` |

    从登录文件中读取给定路径。

+   `--ndb-connectstring`

    | 命令行格式 | `--ndb-connectstring=connection_string` |
    | --- | --- |
    | 类型 | 字符串 |
    | ��认数值 | `[none]` |

    设置连接字符串以连接到 ndb_mgmd。语法："[nodeid=id;][host=]hostname[:port]"。覆盖 NDB_CONNECTSTRING 和 my.cnf 中的条目。

+   `--ndb-mgmd-host`

    | 命令行格式 | `--ndb-mgmd-host=connection_string` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认数值 | `[none]` |

    与 `--ndb-connectstring` 相同。

+   `--ndb-nodeid`

    | 命令行格式 | `--ndb-nodeid=#` |
    | --- | --- |
    | 类型 | 整数 |
    | 默认数值 | `[none]` |

    设置此节点的节点 ID，覆盖由 --ndb-connectstring 设置的任何 ID。

+   `--ndb-optimized-node-selection`

    | 命令行格式 | `--ndb-optimized-node-selection` |
    | --- | --- |
    | 已移除 | 8.0.31 |

    启用用于事务节点选择的优化。默认情况下启用；使用 `--skip-ndb-optimized-node-selection` 来禁用。

+   `--no-defaults`

    | 命令行格式 | `--no-defaults` |
    | --- | --- |

    不要从登录文件以外的任何选项文件中读取默认选项。

+   `--print-defaults`

    | 命令行格式 | `--print-defaults` |
    | --- | --- |

    打印程序参数列表并退出。

+   `--usage`

    | 命令行格式 | `--usage` |
    | --- | --- |

    显示帮助文本并退出；与--help 相同。

+   `--verbose`

    | 命令行格式 | `--verbose` |
    | --- | --- |

    在工具的输出中提供关于其进度的额外信息。

+   `--version`

    | 命令行格式 | `--version` |
    | --- | --- |

    显示版本信息并退出。

#### 示例

首先，我们在`test`数据库中创建一个`NDB`表，使用这里显示的`CREATE TABLE`语句：

```sql
USE test;

CREATE TABLE btest (
    c0 BIGINT UNSIGNED NOT NULL AUTO_INCREMENT PRIMARY KEY,
    c1 TEXT,
    c2 BLOB
)   ENGINE=NDB;
```

然后，我们向这个表中插入几行数据，使用类似于这个语句的一系列语句：

```sql
INSERT INTO btest VALUES (NULL, 'x', REPEAT('x', 1000));
```

当针对这个表运行`--check-orphans`时，**ndb_blob_tool** 生成以下输出：

```sql
$> ndb_blob_tool --check-orphans --verbose -d test btest
connected
processing 2 blobs
processing blob #0 c1 NDB$BLOB_19_1
NDB$BLOB_19_1: nextResult: res=1
total parts: 0
orphan parts: 0
processing blob #1 c2 NDB$BLOB_19_2
NDB$BLOB_19_2: nextResult: res=0
NDB$BLOB_19_2: nextResult: res=0
NDB$BLOB_19_2: nextResult: res=0
NDB$BLOB_19_2: nextResult: res=0
NDB$BLOB_19_2: nextResult: res=0
NDB$BLOB_19_2: nextResult: res=0
NDB$BLOB_19_2: nextResult: res=0
NDB$BLOB_19_2: nextResult: res=0
NDB$BLOB_19_2: nextResult: res=0
NDB$BLOB_19_2: nextResult: res=0
NDB$BLOB_19_2: nextResult: res=1
total parts: 10
orphan parts: 0
disconnected
```

该工具报告称，与列`c1`关联的`NDB` BLOB 列部分不存在，即使`c1`是一个`TEXT`列。 这是因为，在`NDB`表中，只有`BLOB`或`TEXT`列值的前 256 个字节被内联存储，多余的部分（如果有的话）被单独存储；因此，如果给定类型的列中没有使用超过 256 字节的值，则`NDB`不会为此列创建`BLOB`列部分。 更多信息请参见第 13.7 节，“数据类型存储要求”。
