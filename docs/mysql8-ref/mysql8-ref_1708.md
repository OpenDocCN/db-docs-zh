# 25.5.25 ndb_select_all — 从 NDB 表中打印行

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-select-all.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-select-all.html)

**ndb_select_all** 将所有行从 `NDB` 表打印到 `stdout`。

#### 用法

```sql
ndb_select_all -c *connection_string* *tbl_name* -d *db_name* [> *file_name*]
```

可与 **ndb_select_all** 一起使用的选项显示��下表中。表后面还有附加描述。

**表 25.46 与程序 ndb_select_all 一起使用的命令行选项**

| 格式 | 描述 | 添加、弃用或移除 |
| --- | --- | --- |
| `--character-sets-dir=path` | 包含字符集的目录 | 移除：8.0.31 |
| `--connect-retries=#` | 放弃之前重试连接的次数 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--connect-retry-delay=#` | 重试联系管理服务器之间等待的秒数 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--connect-string=connection_string`,`-c connection_string` | 与 --ndb-connectstring 相同 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--core-file` | 在错误时写入核心文件；用于调试 | 移除：8.0.31 |
| `--database=name`,`-d name` | 表所在数据库的名称 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--defaults-extra-file=path` | 读取全局文件后读取给定文件 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--defaults-file=path` | 仅从给定文件中读取默认选项 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--defaults-group-suffix=string` | 也读取 concat(group, suffix) 组 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--delimiter=char`,`-D char` | 设置列分隔符 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--descending`,`-z` | 按降序对结果集进行排序（需要--order） | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--disk` | 打印磁盘引用（仅对具有未索引列的磁盘数据表有用） | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--gci` | 在输出中包含 GCI | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--gci64` | 在输出中包含 GCI 和行时代 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `[--header[=value]](mysql-cluster-programs-ndb-select-all.html#option_ndb_select_all_header)`,`-h` | 打印标题（设置为 0 或 FALSE 以禁用输出中的标题） | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--lock=#`,`-l #` | 锁类型 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--login-path=path` | 从登录文件中读取给定路径 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--help`,`-?` | 显示帮助文本并退出 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--ndb-connectstring=connection_string`,`-c connection_string` | 设置连接到 ndb_mgmd 的连接字符串。语法："[nodeid=id;][host=]hostname[:port]"。覆盖 NDB_CONNECTSTRING 和 my.cnf 中的条目 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--ndb-mgmd-host=connection_string`,`-c connection_string` | 与--ndb-connectstring 相同 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--ndb-nodeid=#` | 为此节点设置节点 ID，覆盖任何由 --ndb-connectstring 设置的 ID | (在基于 MySQL 8.0 的所有 NDB 发行版中支持) |
| `--ndb-optimized-node-selection` | 启用用于事务节点选择的优化。默认启用；使用 --skip-ndb-optimized-node-selection 禁用 | 移除: 8.0.31 |
| `--no-defaults` | 不从除登录文件之外的任何选项文件中读取默认选项 | (在基于 MySQL 8.0 的所有 NDB 发行版中支持) |
| `--nodata` | 不打印表列数据 | (在基于 MySQL 8.0 的所有 NDB 发行版中支持) |
| `--order=index`,`-o index` | 根据具有此名称的索引对结果集进行排序 | (在基于 MySQL 8.0 的所有 NDB 发行版中支持) |
| `--parallelism=#`,`-p #` | 并行度 | (在基于 MySQL 8.0 的所有 NDB 发行版中支持) |
| `--print-defaults` | 打印程序参数列表并退出 | (在基于 MySQL 8.0 的所有 NDB 发行版中支持) |
| `--rowid` | 打印行 ID | (在基于 MySQL 8.0 的所有 NDB 发行版中支持) |
| `--tupscan`,`-t` | 按照 tup 顺序扫描 | (在基于 MySQL 8.0 的所有 NDB 发行版中支持) |
| `--usage`,`-?` | 显示帮助文本并退出；与 --help 相同 | (在基于 MySQL 8.0 的所有 NDB 发行版中支持) |
| `--useHexFormat`,`-x` | 以十六进制格式输出数字 | (在基于 MySQL 8.0 的所有 NDB 发行版中支持) |
| `--version`,`-V` | 显示版本信息并退出 | (在基于 MySQL 8.0 的所有 NDB 发行版中支持) |
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

    与`--ndb-connectstring`相同。

+   `--core-file`

    | 命令行格式 | `--core-file` |
    | --- | --- |
    | 移除 | 8.0.31 |

    在错误时写入核心文件；用于调试。

+   `--database=*`dbname`*`, `-d` *`dbname`*

    表所在的数据库名称。默认值为`TEST_DB`。

+   `--descending`, `-z`

    将输出按降序排序。此选项只能与`-o` (`--order`) 选项一起使用。

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

    还可以读取使用 concat(group, suffix)的组。

+   `--delimiter=*`character`*`, `-D *`character`*`

    导致*`character`*被用作列分隔符。只有表数据列才会被此分隔符分隔。

    默认分隔符是制表符。

+   `--disk`

    向输出中添加一个磁盘引用列。该列仅对具有非索引列的磁盘数据表非空。

+   `--gci`

    向输出中添加一个`GCI`列，显示每行最后更新时的全局检查点。有关检查点的更多信息，请参见第 25.2 节，“NDB 集群概述”和第 25.6.3.2 节，“NDB 集群日志事件”。

+   `--gci64`

    向输出中添加一个`ROW$GCI64`列，显示每行最后更新时的全局检查点，以及此更新发生的时代编号。

+   `--help`

    | 命令行格式 | `--help` |
    | --- | --- |

    显示帮助文本并退出。

+   `--lock=*`lock_type`*`, `-l *`lock_type`*`

    在读取表时使用锁定。*`lock_type`* 的可能值为：

    +   `0`: 读取锁定

    +   `1`: 读取锁定并保持

    +   `2`: 独占读取锁定

    此选项没有默认值。

+   `--login-path`

    | 命令行格式 | `--login-path=path` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `[none]` |

    从登录文件中读取给定路径。

+   `--header=FALSE`

    从输出中排除列标题。

+   `--nodata`

    导致任何表数据被省略。

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

    启用用于事务节点选择的优化。默认启用；使用`--skip-ndb-optimized-node-selection`来禁用。

+   `--no-defaults`

    | 命令行格式 | `--no-defaults` |
    | --- | --- |

    不要从登录文件以外的任何选项文件中读取默认选项。

+   `--order=*`index_name`*`, `-o *`index_name`*`

    根据名为*`index_name`*的索引对输出进行排序。

    注意

    这是一个索引的名称，而不是列的名称；在创建时必须显式命名索引。

+   `parallelism=*`#`*`, `-p` *`#`*

    指定并行度。

+   `--print-defaults`

    | 命令行格式 | `--print-defaults` |
    | --- | --- |

    打印程序参数列表并退出。

+   `--rowid`

    添加一个`ROWID`列，提供有关存储行的片段的信息。

+   `--tupscan`, `-t`

    按元组的顺序扫描表。

+   `--usage`

    | 命令行格式 | `--usage` |
    | --- | --- |

    显示帮助文本并退出；与`--help`相同。

+   `--useHexFormat` `-x`

    导致所有数字值以十六进制格式显示。这不会影响包含在字符串或日期时间值中的数字的输出。

+   `--version`

    | 命令行格式 | `--version` |
    | --- | --- |

    显示版本信息并退出。

#### 示例输出

来自 MySQL `SELECT` 语句的输出：

```sql
mysql> SELECT * FROM ctest1.fish;
+----+-----------+
| id | name      |
+----+-----------+
|  3 | shark     |
|  6 | puffer    |
|  2 | tuna      |
|  4 | manta ray |
|  5 | grouper   |
|  1 | guppy     |
+----+-----------+
6 rows in set (0.04 sec)
```

与等效调用**ndb_select_all**的输出：

```sql
$> ./ndb_select_all -c localhost fish -d ctest1
id      name
3       [shark]
6       [puffer]
2       [tuna]
4       [manta ray]
5       [grouper]
1       [guppy]
6 rows returned
```

所有字符串值在**ndb_select_all**的输出中都用方括号(`[`...`]`)括起来。另一个示例，请考虑在此处创建和填充的表：

```sql
CREATE TABLE dogs (
    id INT(11) NOT NULL AUTO_INCREMENT,
    name VARCHAR(25) NOT NULL,
    breed VARCHAR(50) NOT NULL,
    PRIMARY KEY pk (id),
    KEY ix (name)
)
TABLESPACE ts STORAGE DISK
ENGINE=NDBCLUSTER;

INSERT INTO dogs VALUES
    ('', 'Lassie', 'collie'),
    ('', 'Scooby-Doo', 'Great Dane'),
    ('', 'Rin-Tin-Tin', 'Alsatian'),
    ('', 'Rosscoe', 'Mutt');
```

这演示了几个额外的**ndb_select_all**选项的使用：

```sql
$> ./ndb_select_all -d ctest1 dogs -o ix -z --gci --disk
GCI     id name          breed        DISK_REF
834461  2  [Scooby-Doo]  [Great Dane] [ m_file_no: 0 m_page: 98 m_page_idx: 0 ]
834878  4  [Rosscoe]     [Mutt]       [ m_file_no: 0 m_page: 98 m_page_idx: 16 ]
834463  3  [Rin-Tin-Tin] [Alsatian]   [ m_file_no: 0 m_page: 34 m_page_idx: 0 ]
835657  1  [Lassie]      [Collie]     [ m_file_no: 0 m_page: 66 m_page_idx: 0 ]
4 rows returned
```
