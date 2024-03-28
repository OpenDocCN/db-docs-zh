# 25.5.9 ndb_desc — 描述 NDB 表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-desc.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-desc.html)

**ndb_desc**提供了一个或多个`NDB`表的详细描述。

#### 用法

```sql
ndb_desc -c *connection_string* *tbl_name* -d *db_name* [*options*]

ndb_desc -c *connection_string* *index_name* -d *db_name* -t *tbl_name*
```

可以在本节后面列出可与**ndb_desc**一起使用的其他选项。

#### 示例输出

创建和填充 MySQL 表的语句：

```sql
USE test;

CREATE TABLE fish (
    id INT NOT NULL AUTO_INCREMENT,
    name VARCHAR(20) NOT NULL,
    length_mm INT NOT NULL,
    weight_gm INT NOT NULL,

    PRIMARY KEY pk (id),
    UNIQUE KEY uk (name)
) ENGINE=NDB;

INSERT INTO fish VALUES
    (NULL, 'guppy', 35, 2), (NULL, 'tuna', 2500, 150000),
    (NULL, 'shark', 3000, 110000), (NULL, 'manta ray', 1500, 50000),
    (NULL, 'grouper', 900, 125000), (NULL ,'puffer', 250, 2500);
```

**ndb_desc**的输出：

```sql
$> ./ndb_desc -c localhost fish -d test -p
-- fish --
Version: 2
Fragment type: HashMapPartition
K Value: 6
Min load factor: 78
Max load factor: 80
Temporary table: no
Number of attributes: 4
Number of primary keys: 1
Length of frm data: 337
Max Rows: 0
Row Checksum: 1
Row GCI: 1
SingleUserMode: 0
ForceVarPart: 1
PartitionCount: 2
FragmentCount: 2
PartitionBalance: FOR_RP_BY_LDM
ExtraRowGciBits: 0
ExtraRowAuthorBits: 0
TableStatus: Retrieved
Table options:
HashMap: DEFAULT-HASHMAP-3840-2
-- Attributes --
id Int PRIMARY KEY DISTRIBUTION KEY AT=FIXED ST=MEMORY AUTO_INCR
name Varchar(20;latin1_swedish_ci) NOT NULL AT=SHORT_VAR ST=MEMORY DYNAMIC
length_mm Int NOT NULL AT=FIXED ST=MEMORY DYNAMIC
weight_gm Int NOT NULL AT=FIXED ST=MEMORY DYNAMIC
-- Indexes --
PRIMARY KEY(id) - UniqueHashIndex
PRIMARY(id) - OrderedIndex
uk(name) - OrderedIndex
uk$unique(name) - UniqueHashIndex
-- Per partition info --
Partition       Row count       Commit count    Frag fixed memory       Frag varsized memory    Extent_space    Free extent_space
0               2               2               32768                   32768                   0               0
1               4               4               32768                   32768                   0               0
```

可以通过使用它们的名称（用空格分隔）在单个调用中获取多个表的信息**ndb_desc**。所有表必须在同一个数据库中。

您可以使用`--table`（简写形式：`-t`）选项并将索引名称作为**ndb_desc**的第一个参数来获取有关特定索引的其他信息，如下所示：

```sql
$> ./ndb_desc uk -d test -t fish
-- uk --
Version: 2
Base table: fish
Number of attributes: 1
Logging: 0
Index type: OrderedIndex
Index status: Retrieved
-- Attributes --
name Varchar(20;latin1_swedish_ci) NOT NULL AT=SHORT_VAR ST=MEMORY
-- IndexTable 10/uk --
Version: 2
Fragment type: FragUndefined
K Value: 6
Min load factor: 78
Max load factor: 80
Temporary table: yes
Number of attributes: 2
Number of primary keys: 1
Length of frm data: 0
Max Rows: 0
Row Checksum: 1
Row GCI: 1
SingleUserMode: 2
ForceVarPart: 0
PartitionCount: 2
FragmentCount: 2
FragmentCountType: ONE_PER_LDM_PER_NODE
ExtraRowGciBits: 0
ExtraRowAuthorBits: 0
TableStatus: Retrieved
Table options:
-- Attributes --
name Varchar(20;latin1_swedish_ci) NOT NULL AT=SHORT_VAR ST=MEMORY
NDB$TNODE Unsigned [64] PRIMARY KEY DISTRIBUTION KEY AT=FIXED ST=MEMORY
-- Indexes --
PRIMARY KEY(NDB$TNODE) - UniqueHashIndex
```

当以这种方式指定索引时，`--extra-partition-info`和`--extra-node-info`选项不起作用。

输出中的`Version`列包含表的模式对象版本。有关解释此值的信息，请参阅 NDB ��式对象版本。

可以使用嵌入在`CREATE TABLE`和`ALTER TABLE`语句中的`NDB_TABLE`注释设置的三个表属性也可在**ndb_desc**输出中看到。表的`FRAGMENT_COUNT_TYPE`始终显示在`FragmentCountType`列中。如果将`READ_ONLY`和`FULLY_REPLICATED`设置为 1，则会显示在`Table options`列中。您可以在**mysql**客户端中执行以下`ALTER TABLE`语句后查看此信息：

```sql
mysql> ALTER TABLE fish COMMENT='NDB_TABLE=READ_ONLY=1,FULLY_REPLICATED=1';
1 row in set, 1 warning (0.00 sec)

mysql> SHOW WARNINGS\G
+---------+------+---------------------------------------------------------------------------------------------------------+
| Level   | Code | Message                                                                                                 |
+---------+------+---------------------------------------------------------------------------------------------------------+
| Warning | 1296 | Got error 4503 'Table property is FRAGMENT_COUNT_TYPE=ONE_PER_LDM_PER_NODE but not in comment' from NDB |
+---------+------+---------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)
```

发出警告是因为`READ_ONLY=1`要求表的片段计数类型为（或设置为）`ONE_PER_LDM_PER_NODE_GROUP`；在这种情况下，`NDB`会自动设置这一点。您可以使用`SHOW CREATE TABLE`检查`ALTER TABLE`语句是否产生了预期效果：

```sql
mysql> SHOW CREATE TABLE fish\G
*************************** 1\. row ***************************
       Table: fish
Create Table: CREATE TABLE `fish` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(20) NOT NULL,
  `length_mm` int(11) NOT NULL,
  `weight_gm` int(11) NOT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk` (`name`)
) ENGINE=ndbcluster DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci
COMMENT='NDB_TABLE=READ_BACKUP=1,FULLY_REPLICATED=1' 1 row in set (0.01 sec)
```

因为`FRAGMENT_COUNT_TYPE`没有被显式设置，所以在`SHOW CREATE TABLE`打印的注释文本中不显示其值。然而，**ndb_desc**显示了此属性的更新值。`Table options`列显示了刚刚启用的二进制属性。您可以在这里显示的输出中看到这一点（加粗的文本）：

```sql
$> ./ndb_desc -c localhost fish -d test -p
-- fish --
Version: 4
Fragment type: HashMapPartition
K Value: 6
Min load factor: 78
Max load factor: 80
Temporary table: no
Number of attributes: 4
Number of primary keys: 1
Length of frm data: 380
Max Rows: 0
Row Checksum: 1
Row GCI: 1
SingleUserMode: 0
ForceVarPart: 1
PartitionCount: 1
FragmentCount: 1
*FragmentCountType: ONE_PER_LDM_PER_NODE_GROUP*
ExtraRowGciBits: 0
ExtraRowAuthorBits: 0
TableStatus: Retrieved
*Table options: readbackup, fullyreplicated*
HashMap: DEFAULT-HASHMAP-3840-1
-- Attributes --
id Int PRIMARY KEY DISTRIBUTION KEY AT=FIXED ST=MEMORY AUTO_INCR
name Varchar(20;latin1_swedish_ci) NOT NULL AT=SHORT_VAR ST=MEMORY DYNAMIC
length_mm Int NOT NULL AT=FIXED ST=MEMORY DYNAMIC
weight_gm Int NOT NULL AT=FIXED ST=MEMORY DYNAMIC
-- Indexes --
PRIMARY KEY(id) - UniqueHashIndex
PRIMARY(id) - OrderedIndex
uk(name) - OrderedIndex
uk$unique(name) - UniqueHashIndex
-- Per partition info --
Partition       Row count       Commit count    Frag fixed memory       Frag varsized memory    Extent_space    Free extent_space
```

有关这些表属性的更多信息，请参见第 15.1.20.12 节，“设置 NDB 注释选项”。

`Extent_space`和`Free extent_space`列仅适用于具有磁盘列的`NDB`表；对于仅具有内存列的表，这些列始终包含值`0`。

为了说明它们的用途，我们修改了上一个示例。首先，我们必须创建必要的磁盘数据对象，如下所示：

```sql
CREATE LOGFILE GROUP lg_1
    ADD UNDOFILE 'undo_1.log'
    INITIAL_SIZE 16M
    UNDO_BUFFER_SIZE 2M
    ENGINE NDB;

ALTER LOGFILE GROUP lg_1
    ADD UNDOFILE 'undo_2.log'
    INITIAL_SIZE 12M
    ENGINE NDB;

CREATE TABLESPACE ts_1
    ADD DATAFILE 'data_1.dat'
    USE LOGFILE GROUP lg_1
    INITIAL_SIZE 32M
    ENGINE NDB;

ALTER TABLESPACE ts_1
    ADD DATAFILE 'data_2.dat'
    INITIAL_SIZE 48M
    ENGINE NDB;
```

（有关刚刚显示的语句和由它们创建的对象的更多信息，请参见第 25.6.11.1 节，“NDB 集群磁盘数据对象”，以及第 15.1.16 节，“CREATE LOGFILE GROUP 语句”和第 15.1.21 节，“CREATE TABLESPACE 语句”。）

现在我们可以创建并填充一个存储了`fish`表中 2 列在磁盘上的版本（如果该表已存在，则首先删除之前的版本）：

```sql
DROP TABLE IF EXISTS fish;

CREATE TABLE fish (
    id INT NOT NULL AUTO_INCREMENT,
    name VARCHAR(20) NOT NULL,
    length_mm INT NOT NULL,
    weight_gm INT NOT NULL,

    PRIMARY KEY pk (id),
    UNIQUE KEY uk (name)
) TABLESPACE ts_1 STORAGE DISK
ENGINE=NDB;

INSERT INTO fish VALUES
    (NULL, 'guppy', 35, 2), (NULL, 'tuna', 2500, 150000),
    (NULL, 'shark', 3000, 110000), (NULL, 'manta ray', 1500, 50000),
    (NULL, 'grouper', 900, 125000), (NULL ,'puffer', 250, 2500);
```

当针对该表的此版本运行时，**ndb_desc**显示以下输出：

```sql
$> ./ndb_desc -c localhost fish -d test -p
-- fish --
Version: 1
Fragment type: HashMapPartition
K Value: 6
Min load factor: 78
Max load factor: 80
Temporary table: no
Number of attributes: 4
Number of primary keys: 1
Length of frm data: 1001
Max Rows: 0
Row Checksum: 1
Row GCI: 1
SingleUserMode: 0
ForceVarPart: 1
PartitionCount: 2
FragmentCount: 2
PartitionBalance: FOR_RP_BY_LDM
ExtraRowGciBits: 0
ExtraRowAuthorBits: 0
TableStatus: Retrieved
Table options: readbackup
HashMap: DEFAULT-HASHMAP-3840-2
Tablespace id: 16
Tablespace: ts_1
-- Attributes --
id Int PRIMARY KEY DISTRIBUTION KEY AT=FIXED ST=MEMORY AUTO_INCR
name Varchar(80;utf8mb4_0900_ai_ci) NOT NULL AT=SHORT_VAR ST=MEMORY
length_mm Int NOT NULL AT=FIXED ST=DISK
weight_gm Int NOT NULL AT=FIXED ST=DISK
-- Indexes --
PRIMARY KEY(id) - UniqueHashIndex
PRIMARY(id) - OrderedIndex
uk(name) - OrderedIndex
uk$unique(name) - UniqueHashIndex
-- Per partition info --
Partition       Row count       Commit count    Frag fixed memory       Frag varsized memory    Extent_space    Free extent_space
0               2               2               32768                   32768                   1048576         1044440
1               4               4               32768                   32768                   1048576         1044400
```

这意味着每个分区为该表从表空间分配了 1048576 字节，其中有 1044440 字节可用于额外的存储空间。换句话说，每个分区当前使用 4136 字节来存储来自该表的磁盘列的数据。`Free extent_space`显示的字节数仅用于存储`fish`表的磁盘列数据；因此，在从 Information Schema `FILES`表中进行选择时，它是不可见的。

从 NDB 8.0.21 开始，对于具有磁盘数据的表，`Tablespace id`和`Tablespace`将被显示。

对于完全复制的表，**ndb_desc**仅显示持有主分区片段副本的节点；仅具有副本分区副本的节点将被忽略。您可以使用**mysql**客户端从`table_distribution_status`、`table_fragments`、`table_info`和`table_replicas`表中的信息来获取这样的信息，这些表位于`ndbinfo`数据库中。

所有可以与**ndb_desc**一起使用的选项都显示在下表中。表后面还有附加描述。

**表 25.31 与程序 ndb_desc 一起使用的命令行选项**

| 格式 | 描述 | 新增、弃用或移除 |
| --- | --- | --- |
| `--auto-inc`,`-a` | 如果表有 AUTO_INCREMENT 列，则显示下一个值 | 新增：NDB 8.0.21 |
| `--blob-info`,`-b` | 在输出中包含 BLOB 表的分区信息。需要同时使用-p 选项 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--character-sets-dir=path` | 包含字符集的目录 | 移除：8.0.31 |
| `--connect-retries=#` | 放弃之前重试连接的次数 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--connect-retry-delay=#` | 尝试联系管理服务器之间等待的秒数 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--connect-string=connection_string`,`-c connection_string` | 与--ndb-connectstring 相同 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--context`,`-x` | 显示表的额外信息，如数据库、模式、名称和内部 ID | 新增：NDB 8.0.21 |
| `--core-file` | 在错误时写入核心文件；用于调试 | 已移除：8.0.31 |
| `--database=name`,`-d name` | 包含表的数据库名称 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--defaults-extra-file=path` | 在全局文件读取后读取给定文件 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--defaults-file=path` | 仅从给定文件中读取默认选项 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--defaults-group-suffix=string` | 也读取连接组与后缀连接 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--extra-node-info`,`-n` | 在输出中包含分区到数据节点的映射；需要 --extra-partition-info | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--extra-partition-info`,`-p` | 显示有关分区的信息 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--help`,`-?` | 显示帮助文本并退出 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--login-path=path` | 从登录文件中读取给定路径 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--ndb-connectstring=connection_string`,`-c connection_string` | 设置连接到 ndb_mgmd 的连接字符串。语法："[nodeid=id;][host=]hostname[:port]"。覆盖 NDB_CONNECTSTRING 和 my.cnf 中的条目 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--ndb-mgmd-host=connection_string`,`-c connection_string` | 与 --ndb-connectstring 相同 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--ndb-nodeid=#` | 为此节点设置节点 ID，覆盖 --ndb-connectstring 设置的任何 ID | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--ndb-optimized-node-selection` | 启用优化以选择事务节点。默认启用；使用 --skip-ndb-optimized-node-selection 来禁用 | 移除: 8.0.31 |
| `--no-defaults` | 不从登录文件以外的任何选项文件中读取默认选项 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--print-defaults` | 打印程序参数列表并退出 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--retries=#`,`-r #` | 重试连接的次数（每秒一次） | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--table=name`,`-t name` | 指定要查找索引的表。使用此选项时，-p 和 -n 没有效果且被忽略 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--unqualified`,`-u` | 使用未限定的表名 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--usage`,`-?` | 显示帮助文本并退出；与 --help 相同 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--version`,`-V` | 显示版本信息并退出 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| 格式 | 描述 | 添加、弃用或移除 |

+   `--auto-inc`, `-a`

    显示表的 `AUTO_INCREMENT` 列的下一个值，如果有的话。

+   `--blob-info`, `-b`

    包括有关从属 `BLOB` 和 `TEXT` 列的信息。

    使用此选项还需要使用 `--extra-partition-info` (`-p`) 选项。

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
    | 默认值 | `[无]` |

    与`--ndb-connectstring`相同。

+   `--context`, `-x`

    显示有关表的其他上下文信息，例如模式、数据库名称、表名称和表的内部 ID。

+   `--core-file`

    | 命令行格式 | `--core-file` |
    | --- | --- |
    | 已移除 | 8.0.31 |

    在错误时写入核心文件；用于调试。

+   `--database=*`db_name`*`, `-d`

    指定应在其中找到表的数据库。

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

    仅从给定文件中读取默认选项。

+   `--defaults-group-suffix`

    | 命令行格式 | `--defaults-group-suffix=string` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `[无]` |

    还读取具有 concat(group, suffix)的组。

+   `--extra-node-info`, `-n`

    包括有关表分区与它们所驻留的数据节点之间映射的信息。此信息对于验证分布感知机制和支持更有效的应用程序访问存储在 NDB 集群中的数据非常有用。

    使用此选项还需要使用`--extra-partition-info` (`-p`) 选项。

+   `--extra-partition-info`, `-p`

    打印有关表分区的附加信息。

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

    设置用于连接到 ndb_mgmd 的连接字符串。语法：“[nodeid=id;][host=]hostname[:port]”。覆盖 NDB_CONNECTSTRING 和 my.cnf 中的条目。

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

    为此节点设置节点 ID，覆盖由`--ndb-connectstring`设置的任何 ID。

+   `--ndb-optimized-node-selection`

    | 命令行格式 | `--ndb-optimized-node-selection` |
    | --- | --- |
    | 已移除 | 8.0.31 |

    启用优化以选择事务节点。默认情况下启用；使用`--skip-ndb-optimized-node-selection`来禁用。

+   `--no-defaults`

    | 命令行格式 | `--no-defaults` |
    | --- | --- |

    不要从登录文件以外的任何选项文件中读取默认选项。

+   `--print-defaults`

    | 命令行格式 | `--print-defaults` |
    | --- | --- |

    打印程序参数列表并退出。

+   `--retries=*`#`*`, `-r`

    在放弃之前尝试连接这么多次。每秒进行一次连接尝试。

+   `--table=*`tbl_name`*`, `-t`

    指定要查找索引的表。

+   `--unqualified`, `-u`

    使用未限定的表名。

+   `--usage`

    | 命令行格式 | `--usage` |
    | --- | --- |

    显示帮助文本并退出；与`--help`相同。

+   `--version`

    | 命令行格式 | `--version` |
    | --- | --- |

    显示版本信息并退出。

输出中列出的表索引按 ID 排序。
