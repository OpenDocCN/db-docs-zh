# 25.5.28 ndb_size.pl — NDBCLUSTER 大小需求估算器

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-size-pl.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-size-pl.html)

这是一个 Perl 脚本，可用于估算如果将 MySQL 数据库转换为使用`NDBCLUSTER`存储引擎所需的空间量。与本节讨论的其他实用程序不同，它不需要访问 NDB 集群（实际上，它没有理由这样做）。但是，它确实需要访问要测试的数据库所在的 MySQL 服务器。

#### 要求

+   运行中的 MySQL 服务器。服务器实例不必提供对 NDB 集群的支持。

+   需要安装 Perl。

+   `DBI` 模块，如果它不是您的 Perl 安装的一部分，可以从 CPAN 获取。（许多 Linux 和其他操作系统发行版为此库提供了自己的软件包。）

+   具有必要权限的 MySQL 用户帐户。如果您不希望使用现有帐户，则使用 `GRANT USAGE ON *`db_name`*.*`—其中 *`db_name`* 是要检查的数据库的名称—对于此目的已经足够。

`ndb_size.pl` 也可以在 MySQL 源代码中的 `storage/ndb/tools` 中找到。

可与**ndb_size.pl**一起使用的选项显示在以下表中。表后面会有额外的描述。

**表 25.49 与程序 ndb_size.pl 一起使用的命令行选项**

| 格式 | 描述 | 已添加、已弃用或已移除 |
| --- | --- | --- |
| `--database=string` | 要检查的数据库或数据库；逗号分隔的列表；默认为 ALL（使用在服务器上找到的所有数据库） | （在基于 MySQL 8.0 的所有 NDB 版本中支持） |
| `--hostname=string` | 指定主机和可选端口的主机[:端口]格式 | （在基于 MySQL 8.0 的所有 NDB 版本中支持） |
| `--socket=path` | 指定要连接的套接字 | （在基于 MySQL 8.0 的所有 NDB 版本中支持） |
| `--user=string` | 指定 MySQL 用户名 | （在基于 MySQL 8.0 的所有 NDB 版本中支持） |
| `--password=password` | 指定 MySQL 用户密码 | （在基于 MySQL 8.0 的所有 NDB 版本中支持） |
| `--format=string` | 设置输出格式（文本或 HTML） | （在基于 MySQL 8.0 的所有 NDB 版本中支持） |
| `--excludetables=list` | 跳过逗号分隔列表中的任何表 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--excludedbs=list` | 跳过逗号分隔列表中的任何数据库 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--savequeries=path` | 将数据库中的所有查询保存到指定文件中 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--loadqueries=path` | 从指定文件加载所有查询；不连接到数据库 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--real_table_name=string` | 指定处理唯一索引大小计算的表 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| 格式 | 描述 | 添加、弃用或删除 |

#### 用法

```sql
perl ndb_size.pl [--database={*db_name*|ALL}] [--hostname=*host*[:*port*]] [--socket=*socket*] \
      [--user=*user*] [--password=*password*]  \
      [--help|-h] [--format={html|text}] \
      [--loadqueries=*file_name*] [--savequeries=*file_name*]
```

默认情况下，此实用程序尝试分析服务器上的所有数据库。您可以使用`--database`选项指定单个数据库；使用`ALL`作为数据库名称可以明确指定默认行为。您还可以使用逗号分隔的数据库名称列表使用`--excludedbs`选项排除一个或多个数据库。类似地，您可以通过在可选的`--excludetables`选项后列出其名称（用逗号分隔）来跳过特定表。可以使用`--hostname`指定主机名；默认值为`localhost`。除了主机，还可以使用*`host`*:*`port`*格式为`--hostname`的值指定端口。默认端口号为 3306。必要时，还可以指定套接字；默认值为`/var/lib/mysql.sock`。可以使用相应的选项指定 MySQL 用户名和密码。还可以使用`--format`选项控制输出的格式；可以使用`html`或`text`中的任一值，`text`为默认值。这里展示了文本输出的示例：

```sql
$> ndb_size.pl --database=test --socket=/tmp/mysql.sock
ndb_size.pl report for database: 'test' (1 tables)
--------------------------------------------------
Connected to: DBI:mysql:host=localhost;mysql_socket=/tmp/mysql.sock

Including information for versions: 4.1, 5.0, 5.1

test.t1
-------

DataMemory for Columns (* means varsized DataMemory):
         Column Name            Type  Varsized   Key  4.1  5.0   5.1
     HIDDEN_NDB_PKEY          bigint             PRI    8    8     8
                  c2     varchar(50)         Y         52   52    4*
                  c1         int(11)                    4    4     4
                                                       --   --    --
Fixed Size Columns DM/Row                              64   64    12
   Varsize Columns DM/Row                               0    0     4

DataMemory for Indexes:
   Index Name                 Type        4.1        5.0        5.1
      PRIMARY                BTREE         16         16         16
                                           --         --         --
       Total Index DM/Row                  16         16         16

IndexMemory for Indexes:
               Index Name        4.1        5.0        5.1
                  PRIMARY         33         16         16
                                  --         --         --
           Indexes IM/Row         33         16         16

Summary (for THIS table):
                                 4.1        5.0        5.1
    Fixed Overhead DM/Row         12         12         16
           NULL Bytes/Row          4          4          4
           DataMemory/Row         96         96         48
                    (Includes overhead, bitmap and indexes)

  Varsize Overhead DM/Row          0          0          8
   Varsize NULL Bytes/Row          0          0          4
       Avg Varside DM/Row          0          0         16

                 No. Rows          0          0          0

        Rows/32kb DM Page        340        340        680
Fixedsize DataMemory (KB)          0          0          0

Rows/32kb Varsize DM Page          0          0       2040
  Varsize DataMemory (KB)          0          0          0

         Rows/8kb IM Page        248        512        512
         IndexMemory (KB)          0          0          0

Parameter Minimum Requirements
------------------------------
* indicates greater than default

                Parameter     Default        4.1         5.0         5.1
          DataMemory (KB)       81920          0           0           0
       NoOfOrderedIndexes         128          1           1           1
               NoOfTables         128          1           1           1
         IndexMemory (KB)       18432          0           0           0
    NoOfUniqueHashIndexes          64          0           0           0
           NoOfAttributes        1000          3           3           3
             NoOfTriggers         768          5           5           5
```

为了调试目的，可以从指定文件中读取此脚本运行的 Perl 数组包含的查询，并使用`--savequeries`将其保存到文件中；可以使用`--loadqueries`指定包含在脚本执行期间读取的数组的文件。这两个选项都没有默认值。

要以 HTML 格式生成输出，请使用`--format`选项并将输出重定向到文件，如下所示：

```sql
$> ndb_size.pl --database=test --socket=/tmp/mysql.sock --format=html > ndb_size.html
```

(如果没有重定向，则输出将发送到`stdout`。)

此脚本的输出包括以下信息：

+   为了容纳分析的表所需的`DataMemory`、`IndexMemory`、`MaxNoOfTables`、`MaxNoOfAttributes`、`MaxNoOfOrderedIndexes`和`MaxNoOfTriggers`配置参数的最小值。

+   数据库中定义的所有表、属性、有序索引和唯一哈希索引的内存需求。

+   每个表和表行所需的`IndexMemory`和`DataMemory`。
