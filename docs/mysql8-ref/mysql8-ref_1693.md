# 25.5.13 ndb_import — 将 CSV 数据导入到 NDB 中

> [`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-import.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-import.html)

[**ndb_import**](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-import.html)是通过 NDB API 直接将 CSV 格式的数据导入到`NDB`中的工具，例如由**mysqldump** `--tab`生成的数据。[**ndb_import**](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-import.html)需要连接到 NDB 管理服务器(**ndb_mgmd**)才能正常工作；它不需要连接到 MySQL 服务器。

#### 用法

```sql
ndb_import *db_name* *file_name* *options*
```

[**ndb_import**](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-import.html)需要两个参数。*`db_name`*是要导入数据的表所在的数据库的名称；*`file_name`*是要读取数据的 CSV 文件的名称；如果文件不在当前目录中，则必须包含该文件的路径。文件的名称必须与表的名称匹配；文件的扩展名（如果有）不会被考虑。[**ndb_import**](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-import.html)支持的选项包括用于指定字段分隔符、转义符和行终止符的选项，在本节的后续部分中进行了描述。

在 NDB 8.0.30 之前，[**ndb_import**](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-import.html)会拒绝从 CSV 文件中读取的任何空行。从 NDB 8.0.30 开始，当导入单列时，可以用作列值的空值，ndb_import 会像`LOAD DATA`语句一样处理。

[**ndb_import**](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-import.html)必须能够连接到 NDB 集群管理服务器；因此，在集群`config.ini`文件中必须有一个未使用的`[api]`插槽。

要复制使用不同存储引擎（如`InnoDB`）的现有表作为`NDB`表，可以使用**mysql**客户端执行`SELECT INTO OUTFILE`语句将现有表导出为 CSV 文件，然后执行`CREATE TABLE LIKE`语句创建具有与现有表相同结构的新表，然后在新表上执行`ALTER TABLE ... ENGINE=NDB`；之后，从系统 shell 中调用**ndb_import**将数据加载到新的`NDB`表中。例如，假设您已经以具有适当权限的 MySQL 用户登录，可以将名为`myinnodb_table`的现有`InnoDB`表从名为`myinnodb`的数据库导出到名为`myndb_table`的`NDB`表中，如下所示：

1.  在**mysql**客户端中：

    ```sql
    mysql> USE myinnodb;

    mysql> SELECT * INTO OUTFILE '/tmp/myndb_table.csv'
         >  FIELDS TERMINATED BY ',' OPTIONALLY ENCLOSED BY '"' ESCAPED BY '\\'
         >  LINES TERMINATED BY '\n'
         >  FROM myinnodbtable;

    mysql> CREATE DATABASE myndb;

    mysql> USE myndb;

    mysql> CREATE TABLE myndb_table LIKE myinnodb.myinnodb_table;

    mysql> ALTER TABLE myndb_table ENGINE=NDB;

    mysql> EXIT;
    Bye
    $>
    ```

    一旦目标数据库和表已创建，运行中的**mysqld**不再需要。如果愿意，可以在继续之前使用**mysqladmin shutdown**或其他方法停止它。

1.  在系统 shell 中：

    ```sql
    # if you are not already in the MySQL bin directory:
    $> cd *path-to-mysql-bin-dir*

    $> ndb_import myndb /tmp/myndb_table.csv --fields-optionally-enclosed-by='"' \
        --fields-terminated-by="," --fields-escaped-by='\\'
    ```

    输出应类似于此处所示：

    ```sql
    job-1 import myndb.myndb_table from /tmp/myndb_table.csv
    job-1 [running] import myndb.myndb_table from /tmp/myndb_table.csv
    job-1 [success] import myndb.myndb_table from /tmp/myndb_table.csv
    job-1 imported 19984 rows in 0h0m9s at 2277 rows/s
    jobs summary: defined: 1 run: 1 with success: 1 with failure: 0
    $>
    ```

所有可与**ndb_import**一起使用的选项均显示在以下表中。表后面是附加描述。

**表 25.35 与程序 ndb_import 一起使用的命令行选项**

| 格式 | 描述 | 添加、弃用或移除 |
| --- | --- | --- |
| `--abort-on-error` | 在任何致命错误时转储核心；用于调试 | (在基于 MySQL 8.0 的所有 NDB 版本中���持) |
| `--ai-increment=#` | 对于具有隐藏 PK 的表，指定自增增量。参见 mysqld | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--ai-offset=#` | 对于具有隐藏 PK 的表，指定自增偏移量。参见 mysqld | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--ai-prefetch-sz=#` | 对于具有隐藏 PK 的表，指定预取的自增值数量。参见 mysqld | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--character-sets-dir=path` | 包含字符集的目录 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--connect-retries=#` | 放弃之前尝试连接的次数 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--connect-retry-delay=#` | 尝试联系管理服务器之间等待的秒数 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--connect-string=connection_string`,`-c connection_string` | 与 --ndb-connectstring 相同 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--connections=#` | 要创建的集群连接数 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--continue` | 作业失败时，继续下一个作业 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--core-file` | 在错误时编写核心文件；用于调试 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--csvopt=opts` | 设置典型 CSV 选项值的简写选项。请参阅语法和其他信息的文档 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--db-workers=#` | 每个数据节点执行数据库操作的线程数 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--defaults-extra-file=path` | 在读取全局文件后读取给定文件 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--defaults-file=path` | 仅从给定文件中读取默认选项 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--defaults-group-suffix=string` | 也读取连接组与连接后缀的连接 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--errins-type=name` | 错误插入类型，用于测试目的；使用 "list" 获取所有可能的值 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--errins-delay=#` | 错误插入延迟（毫秒）；添加随机变化 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--fields-enclosed-by=char` | 与 LOAD DATA 语句的 FIELDS ENCLOSED BY 选项相同。对于 CSV 输入，这与使用--fields-optionally-enclosed-by 相同 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--fields-escaped-by=char` | 与 LOAD DATA 语句的 FIELDS ESCAPED BY 选项相同 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--fields-optionally-enclosed-by=char` | 与 LOAD DATA 语句的 FIELDS OPTIONALLY ENCLOSED BY 选项相同 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--fields-terminated-by=char` | 与 LOAD DATA 语句的 FIELDS TERMINATED BY 选项相同 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--help`,`-?` | 显示帮助文本并退出 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--idlesleep=#` | 等待更多工作的毫秒数 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--idlespin=#` | 在 idlesleep 之前重试的次数 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--ignore-lines=#` | 忽略输入文件中的前#行。用于跳过非数据标题 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--input-type=name` | 输入类型：random 或 csv | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--input-workers=#` | 处理输入的线程数。如果--input-type 是 csv，则必须是 2 或更多 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--keep-state` | 通常在作业完成时删除状态文件（除非是非空的*.rej 文件）。使用此选项会导致保留所有状态文件 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--lines-terminated-by=char` | 与 LOAD DATA 语句的 LINES TERMINATED BY 选项相同 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--login-path=path` | 从登录文件中读取给定路径 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--max-rows=#` | 仅导入此数量的输入数据行；默认为 0，即导入所有行 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--missing-ai-column='name'` | 表示要导入的 CSV 文件中缺少自增值。 | 新增：NDB 8.0.30 |
| `--monitor=#` | 如果有变化（状态、拒绝行、临时错误），定期打印正在运行作业的状态。值 0 禁用。值 1 打印任何看到的变化。较高的值按指数方式减少状态打印，直到达到某个预定义限制 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--ndb-connectstring=connection_string`,`-c connection_string` | 设置用于连接到 ndb_mgmd 的连接字符串。���法：“[nodeid=id;][host=]hostname[:port]”。覆盖 NDB_CONNECTSTRING 和 my.cnf 中的条目 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--ndb-mgmd-host=connection_string`,`-c connection_string` | 与--ndb-connectstring 相同 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--ndb-nodeid=#` | 为此节点设置节点 ID，覆盖--ndb-connectstring 设置的任何 ID | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--ndb-optimized-node-selection` | 启用用于事务节点选择的优化。默认启用；使用--skip-ndb-optimized-node-selection 来禁用 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--no-asynch` | 将数据库操作作为批处理，在单个事务中运行 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--no-defaults` | 不从登录文件以外的任何选项文件中读取默认选项 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--no-hint` | 告诉事务协调器在选择数据节点时不要使用分发键提示 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--opbatch=#` | 数据库执行批处理是一组事务和操作发送到 NDB 内核。此选项限制了数据库执行批处理中的 NDB 操作（包括 blob 操作）。因此，它还限制了异步事务的数量。值 0 无效 | (适用于基于 MySQL 8.0 的所有 NDB 版本) |
| `--opbytes=#` | 限制执行批处理中的字节数（默认值 0 = 无限制） | (适用于基于 MySQL 8.0 的所有 NDB 版本) |
| `--output-type=name` | 输出类型：ndb 为默认值，null 用于测试 | (适用于基于 MySQL 8.0 的所有 NDB 版本) |
| `--output-workers=#` | 处理输出或中继数据库操作的线程数 | (适用于基于 MySQL 8.0 的所有 NDB 版本) |
| `--pagesize=#` | 将 I/O 缓冲区对齐到给定大小 | (适用于基于 MySQL 8.0 的所有 NDB 版本) |
| `--pagecnt=#` | I/O 缓冲区大小，以页面大小的倍数计算。CSV 输入工作程序分配双倍大小的缓冲区 | (适用于基于 MySQL 8.0 的所有 NDB 版本) |
| `--polltimeout=#` | 每个轮询的超时时间用于已完成的异步事务；轮询将继续进行，直到所有轮询完成或发生错误 | (适用于基于 MySQL 8.0 的所有 NDB 版本) |
| `--print-defaults` | 打印程序参数列表并退出 | (适用于基于 MySQL 8.0 的所有 NDB 版本) |
| `--rejects=#` | 限制数据加载中被拒绝的行数（具有永久错误的行）。默认值为 0，表示任何被拒绝的行都会导致致命错误。超过限制的行也会添加到*.rej 文件中 | (适用于基于 MySQL 8.0 的所有 NDB 版本) |
| `--resume` | 如果作业中止（临时错误，用户中断），则恢复尚未处理的行 | (适用于基于 MySQL 8.0 的所有 NDB 版本) |
| `--rowbatch=#` | 限制行队列中的行数（默认值 0 = 无限制）；如果--input-type 为 random，则必须为 1 或更多 | (适用于基于 MySQL 8.0 的所有 NDB 版本) |
| `--rowbytes=#` | 限制行队列中的字节数（0 = 无限制） | (适用于基于 MySQL 8.0 的所有 NDB 版本) |
| `--state-dir=path` | 写入状态文件的位置；默认为当前目录 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--stats` | 将性能相关选项和内部统计信息保存在 *.sto 和 *.stt 文件中。即使未使用 --keep-state，这些文件在成功完成后也会保留 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--table=name`,`-t name` | 导入数据的目标名称；默认为输入文件的基本名称 | 新增：NDB 8.0.28 |
| `--tempdelay=#` | 在临时错误之间休眠的毫秒数 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--temperrors=#` | 每个执行批次中事务由于临时错误而失败的次数；0 表示任何临时错误都是致命的。这些错误不会导致任何行被写入 .rej 文件 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--usage`,`-?` | 显示帮助文本并退出；与 --help 相同 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `[--verbose[=#]](mysql-cluster-programs-ndb-import.html#option_ndb_import_verbose)`,`[-v [#]](mysql-cluster-programs-ndb-import.html#option_ndb_import_verbose)` | 启用详细输出 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--version`,`-V` | 显示版本信息并退出 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| 格式 | 描述 | 添加、弃用或移除 |

+   `--abort-on-error`

    | 命令行格式 | `--abort-on-error` |
    | --- | --- |

    在任何致命错误时转储核心；仅用于调试目的。

+   `--ai-increment`=*`#`*

    | 命令行格式 | `--ai-increment=#` |
    | --- | --- |
    | 类型 | 整数 |
    | 默认值 | `1` |
    | 最小值 | `1` |
    | 最大值 | `4294967295` |

    对于具有隐藏主键的表，指定自增增量，就像 MySQL Server 中的 `auto_increment_increment` 系统变量一样。

+   `--ai-offset`=*`#`*

    | 命令行格式 | `--ai-offset=#` |
    | --- | --- |
    | 类型 | 整数 |
    | 默认值 | `1` |
    | 最小值 | `1` |
    | 最大值 | `4294967295` |

    对于具有隐藏主键的表，指定自增偏移量。类似于`auto_increment_offset`系统变量。

+   `--ai-prefetch-sz`=*`#`*

    | 命令行格式 | `--ai-prefetch-sz=#` |
    | --- | --- |
    | 类型 | 整数 |
    | 默认值 | `1024` |
    | 最小值 | `1` |
    | 最大值 | `4294967295` |

    对于具有隐藏主键的表，指定预取的自增值数量。类似于 MySQL 服务器中的`ndb_autoincrement_prefetch_sz`系统变量的行为。

+   `--character-sets-dir`

    | 命令行格式 | `--character-sets-dir=path` |
    | --- | --- |

    包含字符集的目录。

+   `--connections`=*`#`*

    | 命令行格式 | `--connections=#` |
    | --- | --- |
    | 类型 | 整数 |
    | 默认值 | `1` |
    | 最小值 | `1` |
    | 最大值 | `4294967295` |

    创建的集群连接数。

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

    与`--ndb-connectstring`相同。

+   `--continue`

    | 命令行格式 | `--continue` |
    | --- | --- |

    当作业失败时，继续下一个作业。

+   `--core-file`

    | 命令行格式 | `--core-file` |
    | --- | --- |

    在错误时写入核心文件；用于调试。

+   `--csvopt`=*`string`*

    | 命令行格式 | `--csvopt=opts` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `[none]` |

    提供了一种设置典型 CSV 导入选项的快捷方法。此选项的参数是一个由以下一个或多个参数组成的字符串：

    +   `c`: 字段以逗号终止

    +   `d`: 使用默认值，除非被另一个参数覆盖

    +   `n`: 行以`\n`终止

    +   `q`: 字段可选地由双引号字符(`"`)包围

    +   `r`: 行以`\r`终止

    在 NDB 8.0.28 及更高版本中，处理此选项参数中使用的参数顺序，使得最右边的参数始终优先于已在同一参数值中使用的可能冲突的参数。这也适用于给定参数的任何重复实例。在 NDB 8.0.28 之前，参数的顺序没有任何区别，除了当同时指定 `n` 和 `r` 时，最后出现的（最右边的）参数实际上生效。

    此选项旨在用于在难以传输转义符或引号的条件下进行测试。

+   `--db-workers`=*`#`*

    | 命令行格式 | `--db-workers=#` |
    | --- | --- |
    | 类型 | 整数 |
    | 默认值 | `4` |
    | 最小值 | `1` |
    | 最大值 | `4294967295` |

    每个数据节点执行数据库操作的线程数。

+   `--defaults-file`

    | 命令行格式 | `--defaults-file=path` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `[none]` |

    仅从给定文件中读取默认选项。

+   `--defaults-extra-file`

    | 命令行格式 | `--defaults-extra-file=path` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `[none]` |

    在读取全局文件后读取给定文件。

+   `--defaults-group-suffix`

    | 命令行格式 | `--defaults-group-suffix=string` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `[none]` |

    还读取连接（group，suffix）的组。

+   `--errins-type`=*`name`*

    | 命令行格式 | `--errins-type=name` |
    | --- | --- |
    | 类型 | 枚举 |
    | 默认值 | `[none]` |
    | 有效值 | `stopjob``stopall``sighup``sigint``list` |

    错误插入类型；使用 `list` 作为 *`name`* 值以获取所有可能的值。此选项仅用于测试目的。

+   `--errins-delay`=*`#`*

    | 命令行格式 | `--errins-delay=#` |
    | --- | --- |
    | 类型 | 整数 |
    | 默认值 | `1000` |
    | 最小值 | `0` |
    | 最大值 | `4294967295` |
    | 单位 | 毫秒 |

    错误插入延迟（毫秒）；添加随机变化。此选项仅用于测试目的。

+   `--fields-enclosed-by`=*`char`*

    | 命令行格式 | `--fields-enclosed-by=char` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `[none]` |

    这与`LOAD DATA`语句中的`FIELDS ENCLOSED BY`选项的工作方式相同，指定一个字符作为引用字段值的解释。对于 CSV 输入，这与`--fields-optionally-enclosed-by`相同。

+   `--fields-escaped-by`=*`name`*

    | 命令行格式 | `--fields-escaped-by=char` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `\` |

    指定一个转义字符，与 SQL `LOAD DATA`语句中的`FIELDS ESCAPED BY`选项的工作方式相同。

+   `--fields-optionally-enclosed-by`=*`char`*

    | 命令行格式 | `--fields-optionally-enclosed-by=char` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `[none]` |

    这与`LOAD DATA`语句中的`FIELDS OPTIONALLY ENCLOSED BY`选项的工作方式相同，指定一个字符作为可选引用字段值的解释。对于 CSV 输入，这与`--fields-enclosed-by`相同。

+   `--fields-terminated-by`=*`char`*

    | 命令行格式 | `--fields-terminated-by=char` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `\t` |

    这与`LOAD DATA`语句中的`FIELDS TERMINATED BY`选项的工作方式相同，指定一个字符作为字段分隔符的解释。

+   `--help`

    | 命令行格式 | `--help` |
    | --- | --- |

    显示帮助文本并退出。

+   `--idlesleep`=*`#`*

    | 命令行格式 | `--idlesleep=#` |
    | --- | --- |
    | 类型 | 整数 |
    | 默认值 | `1` |
    | 最小值 | `1` |
    | 最大值 | `4294967295` |
    | 单位 | 毫秒 |

    等待更多工作执行的毫秒数。

+   `--idlespin`=*`#`*

    | 命令行格式 | `--idlespin=#` |
    | --- | --- |
    | 类型 | 整数 |
    | 默认值 | `0` |
    | 最小值 | `0` |
    | 最大值 | `4294967295` |

    重试之前睡眠的次数。

+   `--ignore-lines`=*`#`*

    | 命令行格式 | `--ignore-lines=#` |
    | --- | --- |
    | 类型 | 整数 |
    | 默认值 | `0` |
    | 最小值 | `0` |
    | 最大值 | `4294967295` |

    使 ndb_import 忽略输入文件的前*`#`*行。这可用于跳过不包含任何数据的文件头。

+   `--input-type`=*`name`*

    | 命令行格式 | `--input-type=name` |
    | --- | --- |
    | 类型 | 枚举 |
    | 默认值 | `csv` |
    | 有效值 | `random``csv` |

    设置输入类型的类型。默认为 `csv`；`random` 仅用于测试目的。

+   `--input-workers`=*`#`*

    | 命令行格式 | `--input-workers=#` |
    | --- | --- |
    | 类型 | 整数 |
    | 默认值 | `4` |
    | 最小值 | `1` |
    | 最大值 | `4294967295` |

    设置处理输入的线程数。

+   `--keep-state`

    | 命令行格式 | `--keep-state` |
    | --- | --- |

    默认情况下，ndb_import 在完成作业时会删除所有状态文件（除非是非空的 `*.rej` 文件）。指定此选项（不需要参数）可以强制程序保留所有状态文件。

+   `--lines-terminated-by`=*`name`*

    | 命令行格式 | `--lines-terminated-by=char` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `\n` |

    这与 `LOAD DATA` 语句的 `LINES TERMINATED BY` 选项的工作方式相同，指定要解释为行尾的字符。

+   `--log-level`=*`#`*

    | 命令行格式 | `--log-level=#` |
    | --- | --- |
    | 类型 | 整数 |
    | 默认值 | `0` |
    | 最小值 | `0` |
    | 最大值 | `2` |

    在给定级别执行内部日志记录。此选项主要用于内部和开发目的。

    在 NDB 的调试版本中，可以使用此选项将日志级别设置为最大值 4。

+   `--login-path`

    | 命令行格式 | `--login-path=path` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `[none]` |

    从登录文件中读取给定路径。

+   `--max-rows`=*`#`*

    | 命令行格式 | `--max-rows=#` |
    | --- | --- |
    | 类型 | 整数 |
    | 默认值 | `0` |
    | 最小值 | `0` |
    | 最大值 | `4294967295` |
    | 单位 | 字节 |

    仅导入此数量的输入数据行；默认值为 0，表示导入所有行。

+   `--missing-ai-column`

    | 命令行格式 | `--missing-ai-column='name'` |
    | --- | --- |
    | 引入版本 | 8.0.30-ndb-8.0.30 |
    | 类型 | 布尔值 |
    | 默认值 | `FALSE` |

    当导入单个表或多个表时，可以使用此选项。使用时，表示要导入的 CSV 文件不包含任何 `AUTO_INCREMENT` 列的值，并且 **ndb_import** 应该提供这些值；如果使用了该选项且 `AUTO_INCREMENT` 列包含任何值，则无法继续导入操作。

+   `--monitor`=*`#`*

    | 命令行格式 | `--monitor=#` |
    | --- | --- |
    | 类型 | 整数 |
    | 默认值 | `2` |
    | 最小值 | `0` |
    | 最大值 | `4294967295` |
    | 单位 | 字节 |

    定期打印正在运行的作业的状态，如果有变化（状态、拒绝行、临时错误）。设置为 0 以禁用此报告。设置为 1 会打印任何看到的变化。较高的值会减少此状态报告的频率。

+   `--ndb-connectstring`

    | 命令行格式 | `--ndb-connectstring=connection_string` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `[none]` |

    设置连接到 ndb_mgmd 的连接字符串。语法："[nodeid=id;][host=]hostname[:port]"。覆盖 NDB_CONNECTSTRING 和 my.cnf 中的条目。

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

    启用优化以选择事务节点。默认情况下启用；使用`--skip-ndb-optimized-node-selection`来禁用。

+   `--no-asynch`

    | 命令行格式 | `--no-asynch` |
    | --- | --- |

    将数据库操作作为批处理，在单个事务中运行。

+   `--no-defaults`

    | 命令行格式 | `--no-defaults` |
    | --- | --- |

    不要从除登录文件以外的任何选项文件中读取默认选项。

+   `--no-hint`

    | 命令行格式 | `--no-hint` |
    | --- | --- |

    不要使用分布键提示来选择数据节点。

+   `--opbatch`=*`#`*

    | 命令行格式 | `--opbatch=#` |
    | --- | --- |
    | 类型 | 整数 |
    | 默认值 | `256` |
    | 最小值 | `1` |
    | 最大值 | `4294967295` |
    | 单位 | 字节 |

    设置每个执行批次的操作数限制（包括 blob 操作），从而限制异步事务��数量。

+   `--opbytes`=*`#`*

    | 命令行格式 | `--opbytes=#` |
    | --- | --- |
    | 类型 | 整数 |
    | 默认值 | `0` |
    | 最小值 | `0` |
    | 最大值 | `4294967295` |
    | 单位 | 字节 |

    设置每次执行批处理的字节数限制。使用 0 表示无限制。

+   `--output-type`=*`name`*

    | 命令行格式 | `--output-type=name` |
    | --- | --- |
    | 类型 | 枚举 |
    | 默认值 | `ndb` |
    | 有效值 | `null` |

    设置输出类型。`ndb`是默认值。`null`仅用于测试。

+   `--output-workers`=*`#`*

    | 命令行格式 | `--output-workers=#` |
    | --- | --- |
    | 类型 | 整数 |
    | 默认值 | `2` |
    | 最小值 | `1` |
    | 最大值 | `4294967295` |

    设置处理输出或中继数据库操作的线程数。

+   `--pagesize`=*`#`*

    | 命令行格式 | `--pagesize=#` |
    | --- | --- |
    | 类型 | 整数 |
    | 默认值 | `4096` |
    | 最小值 | `1` |
    | 最大值 | `4294967295` |
    | 单位 | 字节 |

    对齐 I/O 缓冲区到指定大小。

+   `--pagecnt`=*`#`*

    | 命令行格式 | `--pagecnt=#` |
    | --- | --- |
    | 类型 | 整数 |
    | 默认值 | `64` |
    | 最小值 | `1` |
    | 最大值 | `4294967295` |

    设置 I/O 缓冲区的大小，以页大小的倍数计算。CSV 输入工作程序分配的缓冲区大小加倍。

+   `--polltimeout`=*`#`*

    | 命令行格式 | `--polltimeout=#` |
    | --- | --- |
    | 类型 | 整数 |
    | 默认值 | `1000` |
    | 最小值 | `1` |
    | 最大值 | `4294967295` |
    | 单位 | 毫秒 |

    设置每次轮询已完成的异步事务的超时时间；轮询将继续直到所有轮询完成，或发生错误。

+   `--print-defaults`

    | 命令行格式 | `--print-defaults` |
    | --- | --- |

    打印程序参数列表并退出。

+   `--rejects`=*`#`*

    | 命令行格式 | `--rejects=#` |
    | --- | --- |
    | 类型 | 整数 |
    | 默认值 | `0` |
    | 最小值 | `0` |
    | 最大值 | `4294967295` |

    限制数据加载中被拒绝的行数（具有永久错误的行）。默认值为 0，这意味着任何被拒绝的行都会导致致命错误。导致超出限制的行��添加到`.rej`文件中。

    此选项设置的限制在当前运行期间有效。使用`--resume`重新启动的运行被视为此目的上的“新”运行。

+   `--resume`

    | 命令行格式 | `--resume` |
    | --- | --- |

    如果作业中止（由于临时数据库错误或用户中断），则恢复尚未处理的任何行。

+   `--rowbatch`=*`#`*

    | 命令行格式 | `--rowbatch=#` |
    | --- | --- |
    | 类型 | 整数 |
    | 默认值 | `0` |
    | 最小值 | `0` |
    | 最大值 | `4294967295` |
    | 单位 | 行 |

    设置每个行队列的行数限制。使用 0 表示无限制。

+   `--rowbytes`=*`#`*

    | 命令行格式 | `--rowbytes=#` |
    | --- | --- |
    | 类型 | 整数 |
    | 默认值 | `262144` |
    | 最小值 | `0` |
    | 最大值 | `4294967295` |
    | 单位 | 字节 |

    设置每个行队列的字节限制。使用 0 表示无限制。

+   `--stats`

    | 命令行格式 | `--stats` |
    | --- | --- |

    保存与性能和其他内部统计信息相关的选项信息在名为 `*.sto` 和 `*.stt` 的文件中。这些文件始终在成功完成时保留（即使未指定 `--keep-state`）。

+   `--state-dir`=*`name`*

    | 命令行格式 | `--state-dir=path` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `.` |

    写入程序运行产生的状态文件（`*`tbl_name`*.map`、`*`tbl_name`*.rej`、`*`tbl_name`*.res`和`*`tbl_name`*.stt`）的位置；默认为当前目录。

+   `--table=*`name`*`

    | 命令行格式 | `--table=name` |
    | --- | --- |
    | 引入版本 | 8.0.28-ndb-8.0.28 |
    | 类型 | 字符串 |
    | 默认值 | `[输入文件基本名称]` |

    默认情况下，**ndb_import** 尝试将数据导入到与正在读取数据的 CSV 文件的基本名称相同的表中。从 NDB 8.0.28 开始，您可以通过使用 `--table` 选项（简写 `-t`）来覆盖表名的选择。

+   `--tempdelay`=*`#`*

    | 命令行格式 | `--tempdelay=#` |
    | --- | --- |
    | 类型 | 整数 |
    | 默认值 | `10` |
    | 最小值 | `0` |
    | 最大值 | `4294967295` |
    | 单位 | 毫秒 |

    临时错误之间休眠的毫秒数。

+   `--temperrors`=*`#`*

    | 命令行格式 | `--temperrors=#` |
    | --- | --- |
    | 类型 | 整数 |
    | 默认值 | `0` |
    | 最小值 | `0` |
    | 最大值 | `4294967295` |

    每个执行批次中由于临时错误而导致事务失败的次数。默认值为 0，这意味着任何临时错误都是致命的。临时错误不会导致任何行被添加到 `.rej` 文件中。

+   `--verbose`, `-v`

    | 命令行格式 | `--verbose[=#]` |
    | --- | --- |
    | 类型 | 布尔值 |
    | 默认值 | `false` |

    启用详细输出。

+   `--usage`

    | 命令行格式 | `--usage` |
    | --- | --- |

    显示帮助文本并退出；与`--help`相同。

+   `--version`

    | 命令行格式 | `--version` |
    | --- | --- |

    显示版本信息并退出。

与`LOAD DATA`一样，用于字段和行格式化的选项必须与用于创建 CSV 文件的选项相匹配，无论是使用`SELECT INTO ... OUTFILE`还是其他方式完成的。没有相当于`LOAD DATA`语句的`STARTING WITH`选项。
