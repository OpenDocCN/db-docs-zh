# 25.5.10 ndb_drop_index — 从 NDB 表中删除索引

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-drop-index.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-drop-index.html)

[**ndb_drop_index**](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-drop-index.html "25.5.10 ndb_drop_index — Drop Index from an NDB Table") 从[`NDB`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster.html "第二十五章 MySQL NDB Cluster 8.0")表中删除指定的索引。*建议您仅将此实用程序用作编写 NDB API 应用程序的示例*—有关详细信息，请参见本节后面的警告。

#### 用法

```sql
ndb_drop_index -c *connection_string* *table_name* *index* -d *db_name*
```

上面显示的语句从*`database`*中的*`table`*中删除名为*`index`*的索引。

可与[**ndb_drop_index**](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-drop-index.html "25.5.10 ndb_drop_index — Drop Index from an NDB Table")一起使用的选项如下表所示。表后面会有附加描述。

**表 25.32 与程序 ndb_drop_index 一起使用的命令行选项**

| 格式 | 描述 | 添加、弃用或移除 |
| --- | --- | --- |
| `[--character-sets-dir=path](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-drop-index.html#option_ndb_drop_index_character-sets-dir)` | 包含字符集的目录 | 已移除：8.0.31 |
| `[--connect-retries=#](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-drop-index.html#option_ndb_drop_index_connect-retries)` | 放弃之前重试连接的次数 | (在基于 MySQL 8.0 的所有 NDB 发行版中支持) |
| `[--connect-retry-delay=#](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-drop-index.html#option_ndb_drop_index_connect-retry-delay)` | 尝试联系管理服务器之间等待的秒数 | (在基于 MySQL 8.0 的所有 NDB 发行版中支持) |
| `[--connect-string=connection_string](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-drop-index.html#option_ndb_drop_index_connect-string)`,`[-c connection_string](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-drop-index.html#option_ndb_drop_index_connect-string)` | 与 --ndb-connectstring 相同 | (在基于 MySQL 8.0 的所有 NDB 发行版中支持) |
| `[--core-file](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-drop-index.html#option_ndb_drop_index_core-file)` | 在错误时写入核心文件；用于调试 | 已移除：8.0.31 |
| `--database=name`,`-d name` | 表所在数据库的名称 | (在基于 MySQL 8.0 的所有 NDB 发行版中支持) |
| `[--defaults-extra-file=path](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-drop-index.html#option_ndb_drop_index_defaults-extra-file)` | 在读取全局文件后读取给定文件 | (在基于 MySQL 8.0 的所有 NDB 发行版中支持) |
| `[--defaults-file=path](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-drop-index.html#option_ndb_drop_index_defaults-file)` | 仅从给定文件中读取默认选项 | (在基于 MySQL 8.0 的所有 NDB 发行版中支持) |
| `[--defaults-group-suffix=string](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-drop-index.html#option_ndb_drop_index_defaults-group-suffix)` | 还读取连接组与连接后缀的连接 | (在基于 MySQL 8.0 的所有 NDB 发行版中支持) |
| `--help`,`-?` | 显示帮助文本并退出 | （支持基于 MySQL 8.0 的所有 NDB 版本） |
| `--login-path=path` | 从登录文件中读取给定路径 | （支持基于 MySQL 8.0 的所有 NDB 版本） |
| `--ndb-connectstring=connection_string`,`-c connection_string` | 设置用于连接到 ndb_mgmd 的连接字符串。语法：“[nodeid=id;][host=]hostname[:port]”。覆盖 NDB_CONNECTSTRING 和 my.cnf 中的条目 | （支持基于 MySQL 8.0 的所有 NDB 版本） |
| `--ndb-mgmd-host=connection_string`,`-c connection_string` | 与--ndb-connectstring 相同 | （支持基于 MySQL 8.0 的所有 NDB 版本） |
| `--ndb-nodeid=#` | 设置此节点的节点 ID，覆盖--ndb-connectstring 设置的任何 ID | （支持基于 MySQL 8.0 的所有 NDB 版本） |
| `--ndb-optimized-node-selection` | 启用节点选择优化以进行事务。默认启用；使用--skip-ndb-optimized-node-selection 来禁用 | 移除：8.0.31 |
| `--no-defaults` | 不要从除登录文件之外的任何选项文件中读取默认选项 | （支持基于 MySQL 8.0 的所有 NDB 版本） |
| `--print-defaults` | 打印程序参数列表并退出 | （支持基于 MySQL 8.0 的所有 NDB 版本） |
| `--usage`,`-?` | 显示帮助文本并退出；与--help 相同 | （支持基于 MySQL 8.0 的所有 NDB 版本） |
| `--version`,`-V` | 显示版本信息并退出 | （支持基于 MySQL 8.0 的所有 NDB 版本） |
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

    重试连���之前放弃的次数。

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
    | 已移除 | 8.0.31 |

    在错误时写入核心文件；用于调试。

+   `--database`, `-d`

    | 命令行格式 | `--database=name` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `TEST_DB` |

    表所在的数据库的名称。

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

    也读取具有连接组的组合（group, suffix）。

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

    设置连接到 ndb_mgmd 的连接字符串。语法：“[nodeid=id;][host=]hostname[:port]”。覆盖 NDB_CONNECTSTRING 和 my.cnf 中的条目。

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

    为此节点设置节点 ID，覆盖`--ndb-connectstring`设置的任何 ID。

+   `--ndb-optimized-node-selection`

    | 命令行格式 | `--ndb-optimized-node-selection` |
    | --- | --- |
    | 已移除 | 8.0.31 |

    启用用于事务节点选择的优化。默认情况下启用；使用`--skip-ndb-optimized-node-selection`来禁用。

+   `--no-defaults`

    | 命令行格式 | `--no-defaults` |
    | --- | --- |

    不要从除登录文件以外的任何选项文件中读取默认选项。

+   `--print-defaults`

    | 命令行格式 | `--print-defaults` |
    | --- | --- |

    打印程序参数列表并退出。

+   `--usage`

    | 命令行格式 | `--usage` |
    | --- | --- |

    显示帮助文本并退出；与`--help`相同。

+   `--version`

    | 命令行格式 | `--version` |
    | --- | --- |

    显示版本信息并退出。

警告

使用 NDB API 对集群表索引执行的操作对 MySQL 不可见，并使 MySQL 服务器无法使用该表。如果您使用此程序删除索引，然后尝试从 SQL 节点访问表，将会出现错误，如下所示：

```sql
$> ./ndb_drop_index -c localhost dogs ix -d ctest1
Dropping index dogs/idx...OK

$> ./mysql -u jon -p ctest1
Enter password: *******
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 7 to server version: 5.7.44-ndb-7.5.32

Type 'help;' or '\h' for help. Type '\c' to clear the buffer.

mysql> SHOW TABLES;
+------------------+
| Tables_in_ctest1 |
+------------------+
| a                |
| bt1              |
| bt2              |
| dogs             |
| employees        |
| fish             |
+------------------+
6 rows in set (0.00 sec)

mysql> SELECT * FROM dogs;
ERROR 1296 (HY000): Got error 4243 'Index not found' from NDBCLUSTER
```

在这种情况下，使表再次对 MySQL 可用的*唯一*选项是删除表并重新创建它。您可以使用 SQL 语句`DROP TABLE`或**ndb_drop_table**实用程序（请参阅 Section 25.5.11, “ndb_drop_table — Drop an NDB Table”）来删除表。
