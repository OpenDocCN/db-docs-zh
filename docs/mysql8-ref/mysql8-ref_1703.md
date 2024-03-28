# 25.5.23 ndb_restore — 恢复 NDB 集群备份

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-restore.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-restore.html)

25.5.23.1 将 NDB 备份恢复到不同版本的 NDB 集群

25.5.23.2 恢复到不同数量的数据节点

25.5.23.3 从并行备份中恢复

NDB 集群恢复程序实现为一个独立的命令行实用程序**ndb_restore**，通常可以在 MySQL `bin` 目录中找到。该程序读取备份生成的文件，并将存储的信息插入数据库。

在 NDB 7.6 及更早版本中，由于对 `NDBT` 测试库的不必要依赖，该程序在运行完成时打印了 `NDBT_ProgramExit - *`status`*`，在 NDB 8.0 中已经移除了这个依赖，消除了多余的输出。

**ndb_restore**必须针对每个由 `START BACKUP` 命令创建的备份文件执行一次。这等于在创建备份时集群中的数据节点数量。

注意

在使用**ndb_restore**之前，建议集群在单用户模式下运行，除非您要并行恢复多个数据节点。有关更多信息，请参见第 25.6.6 节，“NDB 集群单用户模式”。

可与**ndb_restore**一起使用的选项显示在下表中。表后面会有额外的描述。

**表 25.42 与程序 ndb_restore 一起使用的命令行选项**

| 格式 | 描述 | 添加、弃用或移除 |
| --- | --- | --- |
| `[--allow-pk-changes[=0&#124;1]](mysql-cluster-programs-ndb-restore.html#option_ndb_restore_allow-pk-changes)` | 允许更改构成表主键的列集 | 添加：NDB 8.0.21 |
| `--append` | 追加数据到制表符文件 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--backup-password=password` | 为使用--decrypt 解密的加密备份提供密码; 请参阅允许的值的文档 | 新增: NDB 8.0.22 |
| `--backup-password-from-stdin` | 从 STDIN 安全获取解密密码; 与--decrypt 选项一起使用 | 新增: NDB 8.0.24 |
| `--backup-path=path` | 备份文件目录的路径 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--backupid=#`,`-b #` | 恢复具有此 ID 的备份 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--character-sets-dir=path` | 包含字符集的目录 | 移除: 8.0.31 |
| `--connect=connection_string`,`-c connection_string` | --connectstring 的别名 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--connect-retries=#` | 放弃之前重试连接的次数 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--connect-retry-delay=#` | 尝试联系管理服务器之间等待的秒数 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--connect-string=connection_string`,`-c connection_string` | 与--ndb-connectstring 相同 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--core-file` | 在错误时写入核心文件; 用于调试 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--decrypt` | 解密加密备份; 需要--backup-password | 新增: NDB 8.0.22 |
| `--defaults-extra-file=path` | 在读取全局文件后读取给定文件 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--defaults-file=path` | 仅从给定文件中读取默认选项 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--defaults-group-suffix=string` | 还读取具有 concat(group, suffix) 的组 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--disable-indexes` | 导致备份中的索引被忽略；可能减少恢复数据所需的时间 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--dont-ignore-systab-0`,`-f` | 在恢复过程中不要忽略系统表；仅用于实验；不适用于生产环境 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--exclude-databases=list` | 要排除的一个或多个数据库的列表（包括未命名的数据库） | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `[--exclude-intermediate-sql-tables[=TRUE&#124;FALSE]](mysql-cluster-programs-ndb-restore.html#option_ndb_restore_exclude-intermediate-sql-tables)` | 不恢复任何中间表（名称以'#sql-'为前缀）；这些表是从复制 ALTER TABLE 操作中留下的；指定 FALSE 来恢复这些表 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--exclude-missing-columns` | 导致备份版本中缺失于数据库中表版本的列被忽略 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--exclude-missing-tables` | 导致备份中缺失于数据库中的表被忽略 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--exclude-tables=list` | 要排除的一个或多个表的列表（包括未命名的同一数据库中的表）；每个表引用必须包括数据库名称 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--fields-enclosed-by=char` | 字段由此字符包围 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--fields-optionally-enclosed-by` | 字段可选择由此字符包围 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--fields-terminated-by=char` | 字段由此字符终止 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--help`,`-?` | 显示帮助文本并退出 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--hex` | 以十六进制格式打印二进制类型 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `[--ignore-extended-pk-updates[=0&#124;1]](mysql-cluster-programs-ndb-restore.html#option_ndb_restore_ignore-extended-pk-updates)` | 忽略包含对现在包含在扩展主键中的列的更新的日志条目 | 新增：NDB 8.0.21 |
| `--include-databases=list` | 要恢复的一个或多个数据库的列表（排除未命名的数据库） | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--include-stored-grants` | 将共享用户和授权恢复到 ndb_sql_metadata 表 | 新增：NDB 8.0.19 |
| `--include-tables=list` | 要恢复的一个或多个表的列表（排除同一数据库中未命名的表）；每个表引用必须包括数据库名称 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--lines-terminated-by=char` | 行以此字符结尾 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--login-path=path` | 从登录文件中读取给定路径 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--lossy-conversions`,`-L` | 在从备份中恢复数据时允许���值的有损转换（类型降级或符号更改） | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--no-binlog` | 如果 mysqld 已连接并使用二进制日志记录，则不记录恢复的数据 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--no-defaults` | 不从登录文件以外的任何选项文件中读取默认选项 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--no-restore-disk-objects`,`-d` | 不恢复与磁盘数据相关的对象 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--no-upgrade`,`-u` | 不升级尚未调整 VAR 数据大小的 varsize 属性的数组类型，并且不更改列属性 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--ndb-connectstring=connection_string`,`-c connection_string` | 设置用于连接到 ndb_mgmd 的连接字符串。语法："[nodeid=id;][host=]hostname[:port]"。覆盖 NDB_CONNECTSTRING 和 my.cnf 中的条目 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--ndb-mgmd-host=connection_string`,`-c connection_string` | 与 --ndb-connectstring 相同 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--ndb-nodegroup-map=map`,`-z` | 指定节点组映射；未使用，不支持 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--ndb-nodeid=#` | 为此节点设置节点 ID，覆盖 --ndb-connectstring 设置的任何 ID | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--ndb-optimized-node-selection` | 启用节点选择优化以进行事务。默认启用；使用 --skip-ndb-optimized-node-selection 来禁用 | 移除：8.0.31 |
| `--nodeid=#`,`-n #` | 备份所在节点的 ID | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--num-slices=#` | 恢复时应用的切片数量 | 添加：NDB 8.0.20 |
| `--parallelism=#`,`-p #` | 并行事务的数量，用于恢复数据 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--preserve-trailing-spaces`,`-P` | 允许在将固定宽度字符串类型提升为可变宽度类型时保留尾随空格（包括填充） | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--print` | 将元数据、数据和日志打印到标准输出（等同于 --print-meta --print-data --print-log） | (支持所有基于 MySQL 8.0 的 NDB 版本) |
| `--print-data` | 将数据打印到标准输出 | (支持所有基于 MySQL 8.0 的 NDB 版本) |
| `--print-defaults` | 打印程序参数列表并退出 | (支持所有基于 MySQL 8.0 的 NDB 版本) |
| `--print-log` | 将日志打印到标准输出 | (支持所有基于 MySQL 8.0 的 NDB 版本) |
| `--print-meta` | 将元数据打印到��准输出 | (支持所有基于 MySQL 8.0 的 NDB 版本) |
| `--print-sql-log` | 将 SQL 日志写入标准输出 | (支持所有基于 MySQL 8.0 的 NDB 版本) |
| `--progress-frequency=#` | 每隔指定秒数打印还原进度状态 | (支持所有基于 MySQL 8.0 的 NDB 版本) |
| `--promote-attributes`,`-A` | 在从备份还原数据时允许属性提升 | (支持所有基于 MySQL 8.0 的 NDB 版本) |
| `--rebuild-indexes` | 导致在备份中找到的有序索引进行多线程重建；使用的线程数由设置 BuildIndexThreads 决定 | (支持所有基于 MySQL 8.0 的 NDB 版本) |
| `--remap-column=string` | 对指定列的值应用偏移量，使用指定的函数和参数。格式为[db].[tbl].[col]:[fn]:[args]；详细信息请参阅文档 | 新增功能：NDB 8.0.21 |
| `--restore-data`,`-r` | 使用 NDB API 将表数据和日志还原到 NDB 集群 | (支持所有基于 MySQL 8.0 的 NDB 版本) |
| `--restore-epoch`,`-e` | 将 epoch 信息还原到状态表；在副本集群上用于启动复制；更新或插入具有 ID 0 的行到 mysql.ndb_apply_status | (支持所有基于 MySQL 8.0 的 NDB 版本) |
| `--restore-meta`,`-m` | 使用 NDB API 将元数据恢复到 NDB 集群 | (适用于基于 MySQL 8.0 的所有 NDB 版本) |
| `--restore-privilege-tables` | 恢复之前移至 NDB 的 MySQL 权限表 | 废弃：NDB 8.0.16 |
| `--rewrite-database=string` | 恢复到不同命名的数据库；格式为 olddb,newdb | (适用于基于 MySQL 8.0 的所有 NDB 版本) |
| `--skip-broken-objects` | 忽略备份文件中缺失的 blob 表 | (适用于基于 MySQL 8.0 的所有 NDB 版本) |
| `--skip-table-check`,`-s` | 在恢复过程中跳过表结构检查 | (适用于基于 MySQL 8.0 的所有 NDB 版本) |
| `--skip-unknown-objects` | 导致 ndb_restore 不识别的模式对象在从较新 NDB 版本备份恢复到较旧版本时被忽略 | (适用于基于 MySQL 8.0 的所有 NDB 版本) |
| `--slice-id=#` | 在按片段恢复时的片段 ID | 新增：NDB 8.0.20 |
| `--tab=path`,`-T path` | 为提供的路径中的每个表创建一个制表符分隔的.txt 文件 | (适用于基于 MySQL 8.0 的所有 NDB 版本) |
| `--timestamp-printouts{=true&#124;false}` | 在所有信息、错误和调试日志消息前加上时间戳 | 新增：NDB 8.0.33 |
| `--usage`,`-?` | 显示帮助文本并退出；与--help 相同 | (适用于基于 MySQL 8.0 的所有 NDB 版本) |
| `--verbose=#` | 输出中的详细程度 | (适用于基于 MySQL 8.0 的所有 NDB 版本) |
| `--version`,`-V` | 显示版本信息并退出 | (适用于基于 MySQL 8.0 的所有 NDB 版本) |
| `[--with-apply-status](https://wiki.example.org/mysql-cluster-programs-ndb-restore.html#option_ndb_restore_with-apply-status)` | 还原 ndb_apply_status 表。需要--restore-data | 新增：NDB 8.0.29 |
| 格式 | 描述 | 添加、弃用或移除 |

+   [`--allow-pk-changes`](https://wiki.example.org/mysql-cluster-programs-ndb-restore.html#option_ndb_restore_allow-pk-changes)

    | 命令行格式 | `--allow-pk-changes[=0&#124;1]` |
    | --- | --- |
    | 引入版本 | 8.0.21-ndb-8.0.21 |
    | 类型 | 整数 |
    | 默认值 | `0` |
    | 最小值 | `0` |
    | 最大值 | `1` |

    当此选项设置为`1`时，[**ndb_restore**](https://wiki.example.org/mysql-cluster-programs-ndb-restore.html)允许表定义中的主键与备份中相同表的主键不同。当在不同模式版本之间备份和还原时，可能希望在一个或多个表上进行主键更改，而且执行还原操作使用 ndb_restore 比在还原表模式和数据后发出许多[`ALTER TABLE`](https://wiki.example.org/alter-table.html)语句更简单或更有效。

    `--allow-pk-changes`支持以下主键定义的更改：

    +   **扩展主键**：备份中表模式中存在的非空列将成为数据库中表的主键的一部分。

        重要提示

        在扩展表的主键时，任何成为主键一部分的列在备份时不能被更新；任何由[**ndb_restore**](https://wiki.example.org/mysql-cluster-programs-ndb-restore.html)发现的此类更新会导致还原操作失败，即使值没有发生变化。在某些情况下，可能可以使用[`--ignore-extended-pk-updates`](https://wiki.example.org/mysql-cluster-programs-ndb-restore.html#option_ndb_restore_ignore-extended-pk-updates)选项覆盖此行为；有关此选项的更多信息，请参阅该选项的描述。

    +   **缩小主键（1）**：备份模式中已经是表主键一部分的列不再是主键的一部分，但仍然保留在表中。

    +   **缩小主键（2）**：备份模式中已经是表主键一部分的列将被完全从表中移除。

    这些差异可以与[**ndb_restore**](https://wiki.example.org/mysql-cluster-programs-ndb-restore.html)支持的其他模式差异结合使用，包括需要使用分段表的 blob 和 text 列的更改。

    典型情景中使用主键模式更改的基本步骤如下：

    1.  使用[**ndb_restore**](https://wiki.example.org/mysql-cluster-programs-ndb-restore.html)还原表模式[`--restore-meta`](https://wiki.example.org/mysql-cluster-programs-ndb-restore.html#option_ndb_restore_restore-meta)

    1.  更改模式为所需的模式，或创建它

    1.  备份所需的模式

    1.  运行**ndb_restore** `--disable-indexes`使用上一步的备份，以删除索引和约束

    1.  运行**ndb_restore** `--allow-pk-changes`（可能还包括`--ignore-extended-pk-updates`、`--disable-indexes`以及可能需要的其他选项）以恢复所有数据

    1.  运行**ndb_restore** `--rebuild-indexes`使用具有所需模式的备份，以重建索引和约束

    在扩展主键时，可能需要**ndb_restore**在恢复操作期间使用临时的次要唯一索引，以将旧主键映射到新主键。只有在需要将备份日志中的事件应用于具有扩展主键的表时，才会创建这样的索引。此索引命名为`NDB$RESTORE_PK_MAPPING`，并且在每个需要它的表上创建；如果需要，多个运行在并行的**ndb_restore**实例可以共享它。（在恢复过程结束时运行**ndb_restore** `--rebuild-indexes`会导致此索引被删除。）

+   `--append`

    | 命令行格式 | `--append` |
    | --- | --- |

    当与`--tab`和`--print-data`选项一起使用时，这会导致数据附加到具有相同名称的任何现有文件中。

+   `--backup-path`=*`dir_name`*

    | 命令行格式 | `--backup-path=path` |
    | --- | --- |
    | 类型 | 目录名称 |
    | 默认值 | `./` |

    必须提供备份目录的路径；这通过 `--backup-path` 选项提供给**ndb_restore**，并且必须包括与要恢复的备份的 ID 相对应的子目录。例如，如果数据节点的 `DataDir` 是 `/var/lib/mysql-cluster`，那么备份目录是 `/var/lib/mysql-cluster/BACKUP`，备份 ID 为 3 的备份文件可以在 `/var/lib/mysql-cluster/BACKUP/BACKUP-3` 中找到。路径可以是绝对路径，也可以是相对于**ndb_restore**可执行文件所在目录的路径，并且可以选择性地加上 `backup-path=` 前缀。

    可以将备份恢复到与创建时不同配置的数据库。例如，假设在具有节点 ID `2` 和 `3` 的两个存储节点的集群中创建的备份 ID 为 `12` 的备份，需要恢复到具有四个节点的集群中。那么必须分别在备份所在集群的每个存储节点上运行**ndb_restore**两次。然而，**ndb_restore** 不能总是将从运行不同 MySQL 版本的集群中制作的备份恢复到运行不同 MySQL 版本的集群中。有关更多信息，请参见第 25.3.7 节，“NDB 集群升级和降级”。

    重要

    无法使用较旧版本的**ndb_restore**恢复从较新版本的 NDB Cluster 制作的备份。可以将从较新版本的 MySQL 制作的备份恢复到较旧的集群，但必须使用较新 NDB Cluster 版本的**ndb_restore**的副本来执行此操作。

    例如，要将从运行 NDB Cluster 8.0.35 的集群中获取的集群备份恢复到运行 NDB Cluster 7.6.28 的集群中，必须使用随 NDB Cluster 7.6.28 发行的**ndb_restore**。

    为了更快地恢复，可以并行恢复数据，前提是有足够数量的集群连接可用。也就是说，当并行恢复到多个节点时，必须为每个并发的**ndb_restore**进程在集群`config.ini`文件中有一个`[api]`或`[mysqld]`部分可用。但是，数据文件必须始终先于日志应用。

+   `--backup-password=*`password`*`

    | 命令行格式 | `--backup-password=password` |
    | --- | --- |
    | 引入版本 | 8.0.22-ndb-8.0.22 |
    | 类型 | 字符串 |
    | 默认值 | `[none]` |

    此选项指定在使用`--decrypt`选项解密加密备份时要使用的密码。这必须是用于加密备份的相同密码。

    密码必须为 1 到 256 个字符，并且必须用单引号或双引号括起来。它可以包含 ASCII 字符代码为 32、35、38、40-91、93、95 和 97-126 的任何 ASCII 字符；换句话说，它可以使用除了`!`、`'`、`"`、`$`、`%`、`\`和`^`之外的任何可打印 ASCII 字符。

    在 MySQL 8.0.24 及更高版本中，可以省略密码，此时**ndb_restore**将等待从`stdin`提供密码，就像在使用`--backup-password-from-stdin`时一样。

+   [`--backup-password-from-stdin[=TRUE|FALSE]`](mysql-cluster-programs-ndb-restore.html#option_ndb_restore_backup-password-from-stdin)

    | 命令行格式 | `--backup-password-from-stdin` |
    | --- | --- |
    | 引入版本 | 8.0.24-ndb-8.0.24 |

    当用于替代`--backup-password`时，此选项允许从系统 shell（`stdin`）输入备份密码，类似于在使用`--password`时交互地向**mysql**提供密码而不在命令行上提供密码时的操作。

+   `--backupid`=*`#`*, `-b`

    | 命令行格式 | `--backupid=#` |
    | --- | --- |
    | 类型 | 数字 |
    | 默认值 | `none` |

    此选项用于指定备份的 ID 或序列号，与管理客户端在完成备份时显示的`Backup *`backup_id`* completed`消息中显示的数字相同（请参阅第 25.6.8.2 节，“使用 NDB 集群管理客户端创建备份”）。

    重要

    在恢复集群备份时，必须确保从具有相同备份 ID 的备份中恢复所有数据节点。使用来自不同备份的文件最多会导致将集群恢复到不一致状态，并且很可能完全失败。

    在 NDB 8.0 中，此选项是必需的。

+   `--character-sets-dir`

    | 命令行格式 | `--character-sets-dir=path` |
    | --- | --- |
    | 已移除 | 8.0.31 |

    包含字符集的目录。

+   `--connect`, `-c`

    | 命令行格式 | `--connect=connection_string` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `localhost:1186` |

    `--ndb-connectstring` 的别名。

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

    在错误时写入核心文件；用于调试。

+   `--decrypt`

    | 命令行格式 | `--decrypt` |
    | --- | --- |
    | 引入 | 8.0.22-ndb-8.0.22 |

    使用由 `--backup-password` 选项提供的密码解密加密备份。

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

    也读取带有 concat(group, suffix) 的组。

+   `--disable-indexes`

    | 命令行格式 | `--disable-indexes` |
    | --- | --- |

    在从本机`NDB`备份中恢复数据时禁用索引的恢复。之后，您可以使用多线程构建索引一次性为所有表恢复索引，使用`--rebuild-indexes`，这应该比为非常大的表同时重建索引更快。

    在 NDB 8.0.27 及更高版本中，此选项还会删除备份中指定的任何外键。

    在 NDB 8.0.29 之前，尝试从 MySQL 访问一个`NDB`表，其中一个或多个索引找不到时，总是会被拒绝并显示错误`4243` 索引未找到。从 NDB 8.0.29 开始，MySQL 可以打开这样的表，前提是查询不使用任何受影响的索引；否则查询将被拒绝并显示`ER_NOT_KEYFILE`。在后一种情况下，您可以通过执行类似于以下语句的`ALTER TABLE`语句来暂时解决问题：

    ```sql
    ALTER TABLE tbl ALTER INDEX idx INVISIBLE;
    ```

    这会导致 MySQL 忽略表`tbl`上的索引`idx`。有关更多信息，请参阅主键和索引。

+   `--dont-ignore-systab-0`, `-f`

    | 命令行格式 | `--dont-ignore-systab-0` |
    | --- | --- |

    通常，在恢复表数据和元数据时，**ndb_restore**会忽略备份中存在的`NDB`系统表的副本。`--dont-ignore-systab-0`会导致系统表被恢复。*此选项仅供实验和开发使用，不建议在生产环境中使用*。

+   `--exclude-databases`=*`db-list`*

    | 命令行格式 | `--exclude-databases=list` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 |  |

    逗号分隔的一个或多个不应恢复的数据库列表。

    这个选项通常与`--exclude-tables`结合使用；请参阅该选项的描述以获取更多信息和示例。

+   `--exclude-intermediate-sql-tables[`=*`TRUE|FALSE]`*

    | 命令行格式 | `--exclude-intermediate-sql-tables[=TRUE&#124;FALSE]` |
    | --- | --- |
    | 类型 | 布尔值 |
    | 默认值 | `TRUE` |

    在执行复制`ALTER TABLE`操作时，**mysqld**会创建中间表（其名称以`#sql-`为前缀）。当为`TRUE`时，`--exclude-intermediate-sql-tables`选项会阻止**ndb_restore**从恢复这些可能残留的表。此选项默认为`TRUE`。

+   `--exclude-missing-columns`

    | 命令行格式 | `--exclude-missing-columns` |
    | --- | --- |

    可以使用此选项仅恢复选定的表列，这会导致**ndb_restore**忽略备份中与正在恢复的表版本中缺失的任何列。此选项适用于所有正在恢复的表。如果希望仅将此选项应用于选定的表或数据库，可以将其与本节其他地方描述的一个或多个`--include-*`或`--exclude-*`选项结合使用，然后使用一组补充选项将数据恢复到其余表中。

+   `--exclude-missing-tables`

    | 命令行格式 | `--exclude-missing-tables` |
    | --- | --- |

    可以使用此选项仅恢复选定的表，这会导致**ndb_restore**忽略备份中未在目标数据库中找到的任何表。

+   `--exclude-tables`=*`table-list`*

    | 命令行格式 | `--exclude-tables=list` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 |  |

    列出一个或多个要排除的表；每个表引用必须包括数据库名称。通常与`--exclude-databases`一起使用。

    当使用`--exclude-databases`或`--exclude-tables`时，仅排除选项命名的那些数据库或表；所有其他数据库和表都会被**ndb_restore**恢复。

    本表显示了使用`--exclude-*`选项（为清晰起见，可能需要的其他选项已被省略）调用**ndb_restore**的几种情况，以及这些选项对从 NDB Cluster 备份中恢复的影响：

    **表 25.43 使用--exclude-*选项进行多次调用 ndb_restore，并这些选项对从 NDB Cluster 备份中恢复的影响。**

    | 选项 | 结果 |
    | --- | --- |
    | `--exclude-databases=db1` | 所有数据库中除了`db1`的所有表都会被恢复；`db1`中的表不会被恢复 |
    | `--exclude-databases=db1,db2`（或`--exclude-databases=db1` `--exclude-databases=db2`） | 所有数据库中除了`db1`和`db2`的所有表都会被恢复；`db1`或`db2`中的表不会被恢复 |
    | `--exclude-tables=db1.t1` | 除了`db1`中的`t1`之外的所有表都会被恢复；`db1`中的所有其他表都会被恢复；所有其他数据库中的表都会被恢复 |
    | `--exclude-tables=db1.t2,db2.t1`（或`--exclude-tables=db1.t2` `--exclude-tables=db2.t1)` | 除了数据库`db1`中的`t2`和数据库`db2`中的表`t1`之外的所有表都会被恢复；`db1`或`db2`中的其他表不会被恢复；所有其他数据库中的表都会被恢复 |

    您可以一起使用这两个选项。例如，以下操作会导致所有数据库中的所有表*除了*数据库`db1`和`db2`以及数据库`db3`中的表`t1`和`t2`被恢复：

    ```sql
    $> ndb_restore [...] --exclude-databases=db1,db2 --exclude-tables=db3.t1,db3.t2
    ```

    （同样，为了清晰和简洁起见，我们已经省略了示例中可能需要的其他选项。）

    您可以一起使用`--include-*`和`--exclude-*`选项，但必须遵守以下规则：

    +   所有`--include-*`和`--exclude-*`选项的操作是累积的。

    +   所有`--include-*`和`--exclude-*`选项都按照传递给 ndb_restore 的顺序从右到左进行评估。

    +   在存在冲突选项的情况下，第一个（最右侧）选项优先。换句话说，从右到左的第一个选项（匹配给定数据库或表）“获胜”。

    例如，以下选项集导致**ndb_restore**从数据库`db1`中恢复所有表，除了`db1.t1`，同时不恢复来自任何其他数据库的表：

    ```sql
    --include-databases=db1 --exclude-tables=db1.t1
    ```

    然而，简单地颠倒给定选项的顺序会导致所有来自数据库`db1`的表被恢复（包括`db1.t1`，但不包括来自任何其他数据库的表），因为`--include-databases`选项位于最右侧，首先匹配数据库`db1`，因此优先于匹配`db1`或`db1`中任何表的任何其他选项：

    ```sql
    --exclude-tables=db1.t1 --include-databases=db1
    ```

+   `--fields-enclosed-by`=*`char`*

    | 命令行格式 | `--fields-enclosed-by=char` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 |  |

    每个列值都由传递给此选项的字符串括起来（不管数据类型如何；请参阅`--fields-optionally-enclosed-by`的描述）。

+   `--fields-optionally-enclosed-by`

    | 命令行格式 | `--fields-optionally-enclosed-by` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 |  |

    传递给此选项的字符串用于包含包含字符数据（如`CHAR`、`VARCHAR`、`BINARY`、`TEXT`或`ENUM`）的列值。

+   `--fields-terminated-by`=*`char`*

    | 命令行格式 | `--fields-terminated-by=char` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `\t (制表符)` |

    传递给此选项的字符串用于分隔列值。默认值是制表符（`\t`）。

+   `--help`

    | 命令行格式 | `--help` |
    | --- | --- |

    显示帮助文本并退出。

+   `--hex`

    | 命令行格式 | `--hex` |
    | --- | --- |

    如果使用了此选项，则所有二进制值都以十六进制格式输出。

+   `--ignore-extended-pk-updates`

    | 命令行格式 | `--ignore-extended-pk-updates[=0&#124;1]` |
    | --- | --- |
    | 引入版本 | 8.0.21-ndb-8.0.21 |
    | 类型 | 整数 |
    | 默认值 | `0` |
    | 最小值 | `0` |
    | 最大值 | `1` |

    在使用`--allow-pk-changes`时，成为表主键一部分的列在备份过程中不能被更新；这些列应该保持与插入值相同的值，直到包含这些值的行被删除。如果**ndb_restore**在恢复备份时遇到这些列的更新，恢复将失败。因为一些应用程序在更新行时可能为所有列设置值，即使有些列的值没有改变，备份可能包含看似更新未实际修改的列的日志事件。在这种情况下，您可以将`--ignore-extended-pk-updates`设置为`1`，强制**ndb_restore**忽略这样的更新。

    重要

    当导致这些更新被忽略时，用户有责任确保任何成为主键一部分的列的值没有更新。

    更多信息，请参阅`--allow-pk-changes`的描述。

+   `--include-databases`=*`db-list`*

    | 命令行格式 | `--include-databases=list` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 |  |

    逗号分隔的一个或多个要恢复的数据库列表。通常与`--include-tables`一起使用；有关更多信息和示例，请参阅该选项的描述。

+   `--include-stored-grants`

    | 命令行格式 | `--include-stored-grants` |
    | --- | --- |
    | 引入版本 | 8.0.19-ndb-8.0.19 |

    在 NDB 8.0 中，**ndb_restore**默认不恢复共享用户和授权（请参阅第 25.6.13 节，“权限同步和 NDB_STORED_USER”）到`ndb_sql_metadata`表。指定此选项会导致它执行此操作。

+   `--include-tables`=*`table-list`*

    | 命令行格式 | `--include-tables=list` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 |  |

    逗号分隔的要恢复的表列表；每个表引用必须包括数据库名称。

    当使用`--include-databases`或`--include-tables`时，只有由该选项命名的数据库或表会被恢复；所有其他数据库和表都会被**ndb_restore**排除，并且不会被恢复。

    以下表显示了使用`--include-*`选项调用**ndb_restore**的几种情况（为清晰起见，省略了可能需要的其他选项），以及这些选项对从 NDB Cluster 备份中恢复的影响：

    **表 25.44 使用--include-*选项调用 ndb_restore 的几种情况，以及它们对从 NDB Cluster 备份中恢复的影响。**

    | 选项 | 结果 |
    | --- | --- |
    | `--include-databases=db1` | 仅恢复数据库`db1`中的表；忽略所有其他数据库中的所有表 |
    | `--include-databases=db1,db2`（或`--include-databases=db1` `--include-databases=db2`） | 仅恢复数据库`db1`和`db2`中的表；忽略所有其他数据库中的所有表 |
    | `--include-tables=db1.t1` | 仅恢复数据库`db1`中的表`t1`；不恢复`db1`中的其他表或任何其他数据库中的表 |
    | `--include-tables=db1.t2,db2.t1`（或`--include-tables=db1.t2` `--include-tables=db2.t1`） | 仅恢复数据库`db1`中的表`t2`和数据库`db2`中的表`t1`；不恢复`db1`、`db2`或任何其他数据库中的其他表 |

    您还可以同时使用这两个选项。例如，以下内容将导致恢复数据库 `db1` 和 `db2` 中的所有表，以及数据库 `db3` 中的表 `t1` 和 `t2`（而不���其他数据库或表）：

    ```sql
    $> ndb_restore [...] --include-databases=db1,db2 --include-tables=db3.t1,db3.t2
    ```

    （在刚刚显示的示例中，我们再次省略了其他可能需要的选项。）

    也可以仅恢复选定的数据库或来自单个数据库的选定表，而不使用任何 `--include-*`（或 `--exclude-*`）选项，使用此处显示的语法：

    ```sql
    ndb_restore *other_options* *db_name*,[*db_name*[,...] | *tbl_name*[,*tbl_name*][,...]]
    ```

    换句话说，您可以指定要恢复的以下内容之一：

    +   一个或多个数据库的所有表

    +   来自单个数据库的一个或多个表

+   `--lines-terminated-by`=*`char`*

    | 命令行格式 | `--lines-terminated-by=char` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `\n (换行)` |

    指定用于结束每行输出的字符串。默认值是换行符 (`\n`)。

+   `--login-path`

    | 命令行格式 | `--login-path=path` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `[none]` |

    从登录文件中读取给定路径。

+   `--lossy-conversions`, `-L`

    | 命令行格式 | `--lossy-conversions` |
    | --- | --- |

    此选项旨在补充 `--promote-attributes` 选项。使用 `--lossy-conversions` 允许在从备份恢复数据时进行列值的有损转换（类型降级或符号更改）。除了一些例外情况外，规定降级的规则与 MySQL 复制相同；有关当前受支持的属性降级的具体类型转换信息，请参见 Section 19.5.1.9.2, “Replication of Columns Having Different Data Types”。

    从 NDB 8.0.26 开始，此选项还使得将 `NULL` 列恢复为 `NOT NULL` 成为可能。该列不得包含任何 `NULL` 条目；否则 **ndb_restore** 将停止并显示错误。

    **ndb_restore** 在进行有损转换期间执行的任何数据截断都会报告一次，每个属性和列一次。

+   `--no-binlog`

    | 命令行格式 | `--no-binlog` |
    | --- | --- |

    此选项防止任何连接的 SQL 节点将由 **ndb_restore** 恢复的数据写入其二进制日志。

+   `--no-restore-disk-objects`, `-d`

    | 命令行格式 | `--no-restore-disk-objects` |
    | --- | --- |

    此选项阻止**ndb_restore**恢复任何 NDB 集群磁盘数据对象，如表空间和日志文件组；有关这些内容的更多信息，请参见第 25.6.11 节，“NDB 集群磁盘数据表”。

+   `--no-upgrade`, `-u`

    | 命令行格式 | `--no-upgrade` |
    | --- | --- |

    使用**ndb_restore**恢复备份时，使用旧的固定格式创建的`VARCHAR`列将被调整大小并重新创建为现在使用的可变宽度格式。可以通过指定`--no-upgrade`来覆盖此行为。

+   `--ndb-connectstring`

    | 命令行格式 | `--ndb-connectstring=connection_string` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `[none]` |

    为连接到 ndb_mgmd 设置连接字符串。语法：“[nodeid=id;][host=]hostname[:port]”。覆盖 NDB_CONNECTSTRING 和 my.cnf 中的条目。

+   `--ndb-mgmd-host`

    | 命令行格式 | `--ndb-mgmd-host=connection_string` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `[none]` |

    与`--ndb-connectstring`相同。

+   `--ndb-nodegroup-map`=*`map`*, `-z`

    | 命令行格式 | `--ndb-nodegroup-map=map` |
    | --- | --- |

    用于将从一个节点组获取的备份还原到另一个节点组，但从未完全实现；不受支持。

    所有支持此选项的代码在 NDB 8.0.27 中已删除；在此版本及更高版本中，对其设置的任何值都将被忽略，该选项本身不起作用。

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

    启用优化以选择事务节点。默认启用；使用`--skip-ndb-optimized-node-selection`来禁用。

+   `--no-defaults`

    | 命令行格式 | `--no-defaults` |
    | --- | --- |

    不要从除登录文件以外的任何选项文件中读取默认选项。

+   `--nodeid`=*`#`*, `-n`

    | 命令行格式 | `--nodeid=#` |
    | --- | --- |
    | 类型 | 数字 |
    | 默认值 | `none` |

    指定取备份的数据节点的节点 ID。

    在将备份恢复到数据节点数量与取备份的节点不同的集群时，此信息有助于识别要恢复到给定节点的正确文件集或文件集。 （在这种情况下，通常需要将多个文件恢复到单个数据节点。）有关更多信息和示例，请参见 Section 25.5.23.2, “Restoring to a different number of data nodes”。

    在 NDB 8.0 中，此选项是必需的。

+   `--num-slices`=*`#`*

    | 命令行格式 | `--num-slices=#` |
    | --- | --- |
    | 引入版本 | 8.0.20-ndb-8.0.20 |
    | 类型 | 整数 |
    | 默认值 | `1` |
    | 最小值 | `1` |
    | 最大值 | `1024` |

    在按片段恢复备份时，此选项设置备份分成的片段数。这允许多个**ndb_restore**实例并行恢复不相交的子集，可能减少执行恢复操作所需的时间。

    *片段* 是给定备份中数据的子集；也就是说，它是具有相同片段 ID 的一组片段，使用 `--slice-id` 选项指定。这两个选项必须始终一起使用，并且由 `--slice-id` 设置的值必须始终小于片段数。

    **ndb_restore** 遇到片段并为每个片段分配一个片段计数器。在按片段恢复时，每个片段被分配一个片段 ID；此片段 ID 在 0 到片段数减 1 的范围内。对于不是`BLOB`表的表，确定给定片段属于哪个片段的公式如下：

    ```sql
    [*slice_ID*] = [*fragment_counter*] % [*number_of_slices*]
    ```

    对于`BLOB`表，不使用片段计数器；而是使用片段号，以及`BLOB`表的主表 ID（回想一下，`NDB`在内部将 *`BLOB`* 值存储在单独的表中）。在这种情况下，给定片段的片段 ID 计算如下：

    ```sql
    [*slice_ID*] =
    ([*main_table_ID*] + [*fragment_ID*]) % [*number_of_slices*]
    ```

    因此，通过*`N`*切片进行恢复意味着运行*`N`*个**ndb_restore**实例，每个实例都带有`--num-slices=*`N`*`（以及任何其他必要选项），并且每个实例分别带有`--slice-id=1`、`--slice-id=2`、`--slice-id=3`，依此类推直到`slice-id=*`N`*-1`。

    **示例。** 假设您想要恢复名为`BACKUP-1`的备份，在每个数据节点的默认目录`/var/lib/mysql-cluster/BACKUP/BACKUP-3`中找到，到具有节点 ID 为 1、2、3 和 4 的四个数据节点的集群。要使用五个切片执行此操作，请执行以下列表中显示的命令集：

    1.  使用**ndb_restore**恢复集群元数据如下所示：

        ```sql
        $> ndb_restore -b 1 -n 1 -m --disable-indexes --backup-path=/home/ndbuser/backups
        ```

    1.  恢复集群数据到数据节点，调用**ndb_restore**如下所示：

        ```sql
        $> ndb_restore -b 1 -n 1 -r --num-slices=5 --slice-id=0 --backup-path=/var/lib/mysql-cluster/BACKUP/BACKUP-1
        $> ndb_restore -b 1 -n 1 -r --num-slices=5 --slice-id=1 --backup-path=/var/lib/mysql-cluster/BACKUP/BACKUP-1
        $> ndb_restore -b 1 -n 1 -r --num-slices=5 --slice-id=2 --backup-path=/var/lib/mysql-cluster/BACKUP/BACKUP-1
        $> ndb_restore -b 1 -n 1 -r --num-slices=5 --slice-id=3 --backup-path=/var/lib/mysql-cluster/BACKUP/BACKUP-1
        $> ndb_restore -b 1 -n 1 -r --num-slices=5 --slice-id=4 --backup-path=/var/lib/mysql-cluster/BACKUP/BACKUP-1

        $> ndb_restore -b 1 -n 2 -r --num-slices=5 --slice-id=0 --backup-path=/var/lib/mysql-cluster/BACKUP/BACKUP-1
        $> ndb_restore -b 1 -n 2 -r --num-slices=5 --slice-id=1 --backup-path=/var/lib/mysql-cluster/BACKUP/BACKUP-1
        $> ndb_restore -b 1 -n 2 -r --num-slices=5 --slice-id=2 --backup-path=/var/lib/mysql-cluster/BACKUP/BACKUP-1
        $> ndb_restore -b 1 -n 2 -r --num-slices=5 --slice-id=3 --backup-path=/var/lib/mysql-cluster/BACKUP/BACKUP-1
        $> ndb_restore -b 1 -n 2 -r --num-slices=5 --slice-id=4 --backup-path=/var/lib/mysql-cluster/BACKUP/BACKUP-1

        $> ndb_restore -b 1 -n 3 -r --num-slices=5 --slice-id=0 --backup-path=/var/lib/mysql-cluster/BACKUP/BACKUP-1
        $> ndb_restore -b 1 -n 3 -r --num-slices=5 --slice-id=1 --backup-path=/var/lib/mysql-cluster/BACKUP/BACKUP-1
        $> ndb_restore -b 1 -n 3 -r --num-slices=5 --slice-id=2 --backup-path=/var/lib/mysql-cluster/BACKUP/BACKUP-1
        $> ndb_restore -b 1 -n 3 -r --num-slices=5 --slice-id=3 --backup-path=/var/lib/mysql-cluster/BACKUP/BACKUP-1
        $> ndb_restore -b 1 -n 3 -r --num-slices=5 --slice-id=4 --backup-path=/var/lib/mysql-cluster/BACKUP/BACKUP-1

        $> ndb_restore -b 1 -n 4 -r --num-slices=5 --slice-id=0 --backup-path=/var/lib/mysql-cluster/BACKUP/BACKUP-1
        $> ndb_restore -b 1 -n 4 -r --num-slices=5 --slice-id=1 --backup-path=/var/lib/mysql-cluster/BACKUP/BACKUP-1
        $> ndb_restore -b 1 -n 4 -r --num-slices=5 --slice-id=2 --backup-path=/var/lib/mysql-cluster/BACKUP/BACKUP-1
        $> ndb_restore -b 1 -n 4 -r --num-slices=5 --slice-id=3 --backup-path=/var/lib/mysql-cluster/BACKUP/BACKUP-1
        $> ndb_restore -b 1 -n 4 -r --num-slices=5 --slice-id=4 --backup-path=/var/lib/mysql-cluster/BACKUP/BACKUP-1
        ```

        所有刚刚在这一步中展示的命令可以并行执行，只要集群中有足够的连接槽位（参见`--backup-path`选项的描述）。

    1.  恢复索引如常，如下所示：

        ```sql
        $> ndb_restore -b 1 -n 1 --rebuild-indexes --backup-path=/var/lib/mysql-cluster/BACKUP/BACKUP-1
        ```

    1.  最后，使用以下命令恢复时代：

        ```sql
        $> ndb_restore -b 1 -n 1 --restore-epoch --backup-path=/var/lib/mysql-cluster/BACKUP/BACKUP-1
        ```

    恢复集群数据时应使用切片，只需恢复集群数据，不需要在恢复元数据、索引或时代信息时使用`--num-slices`或`--slice-id`。如果这两个选项中的任何一个或两个与控制这些内容恢复的**ndb_restore**选项一起使用，程序会忽略它们。

    使用`--parallelism`选项对恢复速度的影响与使用切片或使用多个**ndb_restore**实例进行并行恢复（`--parallelism`指定单个**ndb_restore**线程执行的并行事务数）是独立的，但可以与其中一个或两者一起使用。您应该注意增加`--parallelism`会导致**ndb_restore**��集群施加更大的负载；如果系统可以处理这一点，恢复应该会更快完成。

    `--num-slices`的值不直接取决于与硬件相关的值，如 CPU 数量或 CPU 核心数，RAM 容量等，也不取决于 LDM 的数量。

    可以在同一恢复过程中在不同的数据节点上使用不同的值来使用此选项；这样做本身不会产生任何不良影响。

+   `--parallelism`=*`#`*, `-p`

    | 命令行格式 | `--parallelism=#` |
    | --- | --- |
    | 类型 | 数值 |
    | 默认值 | `128` |
    | 最小值 | `1` |
    | 最大值 | `1024` |

    **ndb_restore**使用单行事务并发应用多行。此参数确定**ndb_restore**实例尝试使用的并行事务（并发行数）的数量。默认值为 128；最小值为 1，最大值为 1024。

    执行插入操作的工作在涉及的数据节点的线程之间并行化。这种机制用于从`.Data`文件中恢复大量数据，即数据的模糊快照；它不用于构建或重建索引。更改日志是串行应用的；索引删除和构建是 DDL 操作，单独处理。在恢复的客户端端没有线程级并行性。

+   `--preserve-trailing-spaces`, `-P`

    | 命令行格式 | `--preserve-trailing-spaces` |
    | --- | --- |

    当将固定宽度字符数据类型提升为其可变宽度等效类型时，即将`CHAR`列值提升为`VARCHAR`，或将`BINARY`列值提升为`VARBINARY`时，导致尾随空格被保留。否则，当将这些列值插入新列时，任何尾随空格都将从这些列值中删除。

    注意

    虽然你可以将`CHAR`列提升为`VARCHAR`，将`BINARY`列提升为`VARBINARY`，但不能将`VARCHAR`列提升为`CHAR`或`VARBINARY`列提升为`BINARY`。

+   `--print`

    | 命令行格式 | `--print` |
    | --- | --- |

    使**ndb_restore**打印所有数据、元数据和日志到`stdout`。等同于同时使用`--print-data`、`--print-meta`和`--print-log`选项。

    注意

    使用`--print`或任何`--print_*`选项都是执行干预运行。包括其中一个或多个这些选项会导致任何输出重定向到`stdout`；在这种情况下，**ndb_restore** 不会尝试将数据或元数据恢复到 NDB Cluster。

+   `--print-data`

    | 命令行格式 | `--print-data` |
    | --- | --- |

    使**ndb_restore**将其输出定向到`stdout`。通常与`--tab`、`--fields-enclosed-by`、`--fields-optionally-enclosed-by`、`--fields-terminated-by`、`--hex`和`--append`中的一个或多个一起使用。

    `TEXT`和`BLOB`列值总是被截断。���些值在输出中被截断为前 256 个字节。目前在使用`--print-data`时无法覆盖此行为。

+   `--print-defaults`

    | 命令行格式 | `--print-defaults` |
    | --- | --- |

    打印程序参数列表并退出。

+   `--print-log`

    | 命令行格式 | `--print-log` |
    | --- | --- |

    使**ndb_restore**将其日志输出到`stdout`。

+   `--print-meta`

    | 命令行格式 | `--print-meta` |
    | --- | --- |

    将所有元数据打印到`stdout`。

+   `print-sql-log`

    | 命令行格式 | `--print-sql-log` |
    | --- | --- |

    将 SQL 语句记录到 `stdout`。使用该选项启用；通常此行为已禁用。在尝试记录之前，该选项会检查是否所有正在恢复的表都有明确定义的主键；只有具有 `NDB` 实现的��藏主键的表上的查询无法转换为有效的 SQL。

    此选项不适用于具有 `BLOB` 列的表。

+   `--progress-frequency`=*`N`*

    | 命令行格式 | `--progress-frequency=#` |
    | --- | --- |
    | 类型 | 数值 |
    | 默认值 | `0` |
    | 最小值 | `0` |
    | 最大值 | `65535` |

    在备份进行中每 *`N`* 秒打印一次状态报告。0（默认值）不会打印任何状态报告。最大值为 65535。

+   `--promote-attributes`, `-A`

    | 命令行格式 | `--promote-attributes` |
    | --- | --- |

    **ndb_restore** 支持有限的属性提升，方式与 MySQL 复制支持的方式类似；也就是说，从给定类型的列备份的数据通常可以恢复到使用“更大、类似”的类型的列。例如，从 `CHAR(20)` 列的数据可以恢复到声明为 `VARCHAR(20)`、`VARCHAR(30)` 或 `CHAR(30)` 的列；从 `MEDIUMINT` 列的数据可以恢复到类型为 `INT` 或 `BIGINT` 的列。请参阅 19.5.1.9.2 节，“具有不同数据类型的列的复制”，了解目前由属性提升支持的类型转换表。

    从 NDB 8.0.26 开始，此选项还可以将 `NOT NULL` 列恢复为 `NULL`。

    **ndb_restore** 的属性提升必须显式启用，如下所示：

    1.  准备要恢复备份的表。**ndb_restore** 不能用于使用与原始表定义不同的定义重新创建表；这意味着您必须手动创建表，或在恢复表元数据后但在恢复数据之前使用 `ALTER TABLE` 修改要提升的列。

    1.  在恢复表数据时，使用`--promote-attributes`选项（简写为`-A`）调用**ndb_restore**。如果不使用此选项，则不会发生属性提升；相反，恢复操作将失败并显示错误。

    在字符数据类型和`TEXT`或`BLOB`之间进行转换时，只能同时执行字符类型（`CHAR`和`VARCHAR`）和二进制类型（`BINARY`和`VARBINARY`）之间的转换。例如，你不能在同一次调用**ndb_restore**时将一个`INT`列提升为`BIGINT`同时将一个`VARCHAR`列提升为`TEXT`。

    不支持在不同字符集之间转换`TEXT`列，并且明确禁止。

    在使用**ndb_restore**进行字符或二进制类型到`TEXT`或`BLOB`的转换时，你可能会注意到它创建并使用一个或多个名为`*`table_name`*$ST*`node_id`*`的临时表。这些表在之后不再需要，并且通常在成功恢复后由**ndb_restore**删除。

+   `--rebuild-indexes`

    | 命令行格式 | `--rebuild-indexes` |
    | --- | --- |

    在恢复本地`NDB`备份时启用有序索引的多线程重建。使用此选项由**ndb_restore**用于构建有序索引的线程数受`BuildIndexThreads`数据节点配置参数和 LDM 数的控制。

    只有在首次运行**ndb_restore**时才需要使用此选项；这将导致所有有序索引在恢复后续节点时重新构建，而无需再次使用`--rebuild-indexes`。在向数据库插入新行之前，应使用此选项；否则，在尝试重建索引时可能会插入导致唯一约束冲突的行。

    默认情况下，有序索引的构建与 LDM 的数量并行进行。在节点和系统重新启动期间执行的离线索引构建可以通过`BuildIndexThreads`数据节点配置参数加快速度；此参数对由**ndb_restore**在线执行的索引删除和重建没有影响。

    唯一索引的重建使用磁盘写入带宽进行重做日志记录和本地检查点。带宽不足可能导致重做缓冲区超载或日志超载错误。在这种情况下，您可以再次运行**ndb_restore** `--rebuild-indexes`；进程将在发生错误的地方恢复。在遇到临时错误时，也可以这样做。您可以无限次重复执行**ndb_restore** `--rebuild-indexes`；通过减少`--parallelism`的值，您可能能够停止此类错误。如果问题是空间不足，您可以增加重做日志的大小（`FragmentLogFileSize`节点配置参数），或者增加执行 LCP 的速度（`MaxDiskWriteSpeed`和相关参数），以便更快地释放空间。

+   `--remap-column=*`db`*.*`tbl`*.*`col`*:*`fn`*:*`args`*`

    | 命令行格式 | `--remap-column=string` |
    | --- | --- |
    | 引入版本 | 8.0.21-ndb-8.0.21 |
    | 类型 | 字符串 |
    | 默认值 | `[none]` |

    与`--restore-data`一起使用时，此选项将对指定列的值应用函数。参数字符串中的值在此处列出：

    +   *`db`*: 数据库名称，在`--rewrite-database`执行的任何重命名之后。

    +   *`tbl`*: 表名。

    +   *`col`*: 要更新的列的名称。此列必须是`INT`或`BIGINT`类型。该列也可以是但不需要是`UNSIGNED`。

    +   *`fn`*: 函数名称；目前，唯一支持的名称是`offset`。

    +   *`args`*: 提供给函数的参数。目前，仅支持一个参数，即`offset`函数要添加的偏移量的大小。支持负值。参数的大小不能超过列类型的有符号变体；例如，如果*`col`*是一个`INT`列，则传递给`offset`函数的参数的允许范围为`-2147483648`到`2147483647`（参见第 13.1.2 节，“整数类型（精确值） - INTEGER、INT、SMALLINT、TINYINT、MEDIUMINT、BIGINT”）。

        如果将偏移值应用于列会导致溢出或下溢，则恢复操作将失败。例如，如果列是`BIGINT`，并且选项尝试在列值为 4294967291 的行上应用偏移值 8，则会发生这种情况，因为`4294967291 + 8 = 4294967299 > 4294967295`。

    当您希望将存储在多个 NDB Cluster 源实例中的数据（均使用相同模式）合并到单个目标 NDB Cluster 中时，可以使用此选项，使用 NDB 本机备份（参见第 25.6.8.2 节，“使用 NDB Cluster 管理客户端创建备份”）和**ndb_restore**来合并数据，其中源集群之间的主键和唯一键值重叠，并且在过程中有必要将这些值重新映射到不重叠的范围。可能还需要保留表之间的其他关系。为了满足这些要求，可以在同一次调用**ndb_restore**中多次使用该选项来重新映射不同表的列，如下所示：

    ```sql
    $> ndb_restore --restore-data --remap-column=hr.employee.id:offset:1000 \
        --remap-column=hr.manager.id:offset:1000 --remap-column=hr.firstaiders.id:offset:1000
    ```

    （此处未显示其他选项也可以使用。）

    `--remap-column`也可用于更新同一表的多个列。可以组合多个表和列。不同的偏移值也可以用于同一表的不同列，如下所示：

    ```sql
    $> ndb_restore --restore-data --remap-column=hr.employee.salary:offset:10000 \
        --remap-column=hr.employee.hours:offset:-10
    ```

    当源备份包含不应合并的重复表时，您可以通过使用`--exclude-tables`，`--exclude-databases`，或者通过应用程序中的其他方式来处理这个问题。

    可以使用`SHOW CREATE TABLE`；**ndb_desc**工具；以及`MAX()`，`MIN()`，`LAST_INSERT_ID()`和其他 MySQL 函数来获取要合并的表的结构和其他特征的信息。

    不支持在 NDB Cluster 的不同实例之间从合并到未合并表或从未合并到合并表的更改复制。

+   `--restore-data`, `-r`

    | 命令行格式 | `--restore-data` |
    | --- | --- |

    输出`NDB`表数据和日志。

+   `--restore-epoch`, `-e`

    | 命令行格式 | `--restore-epoch` |
    | --- | --- |

    向集群复制状态表添加（或恢复）时代信息。这对于在 NDB Cluster 副本上启动复制非常有用。当使用此选项时，如果`mysql.ndb_apply_status`中具有`id`列中的`0`已经存在，则更新该行；如果尚不存在，则插入这样的行。（参见第 25.7.9 节，“带有 NDB Cluster 复制的 NDB Cluster 备份”。）

+   `--restore-meta`, `-m`

    | 命令行格式 | `--restore-meta` |
    | --- | --- |

    此选项导致**ndb_restore**打印`NDB`表元数据。

    第一次运行**ndb_restore**恢复程序时，您还需要恢复元数据。换句话说，您必须重新创建数据库表 - 这可以通过使用`--restore-meta`（`-m`）选项来完成。只需要在单个数据节点上恢复元数据；这就足以将其恢复到整个集群。

    在旧版本的 NDB 集群中，使用此选项恢复模式的表使用与新集群中的数据节点数量不同的情况下，仍然使用与原始集群相同数量的分区。在 NDB 8.0 中，当恢复元数据时，这不再是一个问题；**ndb_restore**现在使用目标集群的默认分区数，除非本地数据管理器线程的数量也与原始集群中的数据节点不同。

    在 NDB 8.0 中使用此选项时，建议通过设置`ndb_metadata_check=OFF`来禁用自动同步，直到**ndb_restore**完成恢复元数据，然后可以再次打开以同步在 NDB 字典中新创建的对象。

    注意

    在开始恢复备份时，集群应该有一个空数据库。（换句话说，在执行恢复操作之前，应该使用`--initial`启动数据节点。）

+   `--restore-privilege-tables`

    | 命令行格式 | `--restore-privilege-tables` |
    | --- | --- |
    | 已弃用 | 8.0.16-ndb-8.0.16 |

    **ndb_restore**默认情况下不会恢复在 NDB 集群版本 8.0 之前创建的分布式 MySQL 权限表，该版本不支持 NDB 7.6 及更早版本中实现的分布式权限。此选项导致**ndb_restore**恢复这些表。

    在 NDB 8.0 中，这些表不用于访问控制；作为 MySQL 服务器升级过程的一部分，服务器在本地创建`InnoDB`表的副本。有关更多信息，请参阅 Section 25.3.7, “Upgrading and Downgrading NDB Cluster”以及 Section 8.2.3, “Grant Tables”。

+   `--rewrite-database`=*`olddb,newdb`*

    | 命令行格式 | `--rewrite-database=string` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `none` |

    此选项使得可以将数据恢复到与备份中使用的名称不同的数据库。例如，如果备份的数据库名为`products`，则可以将其包含的数据恢复到名为`inventory`的数据库中，使用如下所示的选项（省略可能需要的其他选项）：

    ```sql
    $> ndb_restore --rewrite-database=product,inventory
    ```

    可以在单个**ndb_restore**调用中多次使用该选项。因此，可以同时从名为`db1`的数据库恢复到名为`db2`的数据库，以及从名为`db3`的数据库恢复到名为`db4`的数据库，使用`--rewrite-database=db1,db2 --rewrite-database=db3,db4`。在多次出现`--rewrite-database`之间可以使用其他**ndb_restore**选项。

    在多个`--rewrite-database`选项之间发生冲突时，从左到右读取的最后一个`--rewrite-database`选项生效。例如，如果使用`--rewrite-database=db1,db2 --rewrite-database=db1,db3`，只有`--rewrite-database=db1,db3`会被采纳，而`--rewrite-database=db1,db2`会被忽略。也可以从多个数据库恢复到单个数据库，因此`--rewrite-database=db1,db3 --rewrite-database=db2,db3`会将数据库`db1`和`db2`的所有表和数据恢复到数据库`db3`中。

    重要

    使用`--rewrite-database`从多个备份数据库恢复到单个目标数据库时，不会检查表或其他对象名称之间的冲突，并且不保证恢复行的顺序。这意味着在这种情况下，可能会覆盖行并丢失更新。

+   `--skip-broken-objects`

    | 命令行格式 | `--skip-broken-objects` |
    | --- | --- |

    此选项导致**ndb_restore**在读取本地`NDB`备份时忽略损坏的表，并继续恢复任何剩余的表（未损坏的）。目前，`--skip-broken-objects`选项仅在缺少 blob 部分表的情况下有效。

+   `--skip-table-check`, `-s`

    | 命令行格式 | `--skip-table-check` |
    | --- | --- |

    可以在不恢复表元数据的情况下恢复数据。默认情况下，当这样做时，**ndb_restore**在表数据和表模式之间发现不匹配时会失败并显示错误；此选项覆盖了该行为。

    在使用**ndb_restore**还原数据时，对列定义不匹配的限制有所放宽；当遇到这些类型的不匹配时，**ndb_restore**不像以前那样停止并报错，而是接受数据并将其插入目标表中，同时向用户发出警告，说明正在执行此操作。无论是否使用选项`--skip-table-check`或`--promote-attributes`，这种列定义的差异都是以下类型：

    +   不同的`COLUMN_FORMAT`设置（`FIXED`，`DYNAMIC`，`DEFAULT`）

    +   不同的`STORAGE`设置（`MEMORY`，`DISK`）

    +   不同的默认值

    +   不同的分发键设置

+   `--skip-unknown-objects`

    | 命令行格式 | `--skip-unknown-objects` |
    | --- | --- |

    此选项导致**ndb_restore**在读取本地`NDB`备份时忽略任何它不认识的模式对象。这可用于将从运行（例如）NDB 7.6 的集群制作的备份还原到运行 NDB Cluster 7.5 的集群。

+   `--slice-id`=*`#`*

    | 命令行格式 | `--slice-id=#` |
    | --- | --- |
    | 引入版本 | 8.0.20-ndb-8.0.20 |
    | 类型 | 整数 |
    | 默认值 | `0` |
    | 最小值 | `0` |
    | 最大值 | `1023` |

    在按片还原时，这是要还原的片的 ID。此选项始终与`--num-slices`一起使用，其值必须始终小于`--num-slices`的值。

    更多信息，请参阅本节其他地方的`--num-slices`的描述。

+   `--tab`=*`dir_name`*, `-T` *`dir_name`*

    | 命令行格式 | `--tab=path` |
    | --- | --- |
    | 类型 | 目录名称 |

    导致`--print-data`创建转储文件，每个表一个文件，每个文件名为`*`tbl_name`*.txt`。它需要作为参数的是文件应该保存的目录路径；对于当前目录使用`.`。

+   `--timestamp-printouts`

    | 命令行格式 | `--timestamp-printouts{=true&#124;false}` |
    | --- | --- |
    | 引入版本 | 8.0.33-ndb-8.0.33 |
    | 类型 | 布尔值 |
    | 默认值 | `true` |

    导致信息、错误和调试日志消息以时间戳为前缀。

    在 NDB 8.0 中默认启用此选项。使用`--timestamp-printouts=false` 禁用它。

+   `--usage`

    | 命令行格式 | `--usage` |
    | --- | --- |

    显示帮助文本并退出；与`--help`相同。

+   `--verbose`=*`#`*

    | 命令行格式 | `--verbose=#` |
    | --- | --- |
    | 类型 | 数值 |
    | 默认值 | `1` |
    | 最小值 | `0` |
    | 最大值 | `255` |

    设置输出的详细程度级别。最小值为 0；最大值为 255。默认值为 1。

+   `--version`

    | 命令行格式 | `--version` |
    | --- | --- |

    显示版本信息并退出。

+   `--with-apply-status`

    | 命令行格式 | `--with-apply-status` |
    | --- | --- |
    | 引入版本 | 8.0.29-ndb-8.0.29 |

    从备份的`ndb_apply_status`表中恢复所有行（除了具有`server_id = 0`的行，该行是使用`--restore-epoch`生成的）。此选项要求还必须使用`--restore-data`。

    如果备份中的`ndb_apply_status`表已经包含`server_id = 0`的行，**ndb_restore** `--with-apply-status` 将其删除。因此，我们建议在使用`--with-apply-status`选项调用**ndb_restore**后使用**ndb_restore** `--restore-epoch`。您还可以在最后一次调用**ndb_restore** `--with-apply-status` 用于恢复集群时同时使用`--restore-epoch`。

    有关更多信息，请参阅 ndb_apply_status Table。

此实用程序的典型选项如下所示：

```sql
ndb_restore [-c *connection_string*] -n *node_id* -b *backup_id* \
      [-m] -r --backup-path=*/path/to/backup/files*
```

通常，在从 NDB 集群备份恢复时，**ndb_restore** 至少需要`--nodeid`（简写：`-n`）、`--backupid`（简写：`-b`）和`--backup-path`选项。

`-c` 选项用于指定连接字符串，告诉 `ndb_restore` 在哪里找到集群管理服务器（参见第 25.4.3.3 节，“NDB 集群连接字符串”）。如果不使用此选项，则**ndb_restore** 将尝试连接到 `localhost:1186` 上的管理服务器。此实用程序充当集群 API 节点，因此需要一个可用的连接“槽”来连接到集群管理服务器。这意味着在集群 `config.ini` 文件中必须至少有一个可供其使用的 `[api]` 或 `[mysqld]` 部分。出于这个原因，最好保留至少一个未被用于 MySQL 服务器或其他应用程序的空的 `[api]` 或 `[mysqld]` 部分（请参阅第 25.4.3.7 节，“在 NDB 集群中定义 SQL 和其他 API 节点”）。

在 NDB 8.0.22 及更高版本中，**ndb_restore** 可以使用 `--decrypt` 和 `--backup-password` 解密加密备份。必须同时指定这两个选项才能执行解密操作。有关创建加密备份的信息，请参阅`START BACKUP`管理客户端命令的文档。

您可以通过在**ndb_mgm**管理客户端中使用`SHOW`命令来验证**ndb_restore**是否连接到集群。您也可以从系统 shell 中执行此操作，如下所示：

```sql
$> ndb_mgm -e "SHOW"
```

**错误报告。** **ndb_restore** 报告临时和永久错误。在临时错误的情况下，它可能能够从中恢复，并在这种情况下报告`恢复成功，但遇到临时错误，请查看配置`。

重要

使用**ndb_restore**来初始化用于循环复制的 NDB 集群后，作为副本的 SQL 节点上不会自动创建二进制日志，您必须手动创建。要创建二进制日志，请在运行`START SLAVE`之前在该 SQL 节点上发出一个`SHOW TABLES`语句。这是 NDB 集群中已知的问题。
