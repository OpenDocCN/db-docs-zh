> 原文：[`dev.mysql.com/doc/refman/8.0/en/replication-options-replica.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-options-replica.html)

#### 19.1.6.3 副本服务器选项和变量

本节介绍了适用于副本服务器的服务器选项和系统变量，包括以下内容：

+   副本服务器的启动选项

+   副本服务器上使用的系统变量

在命令行上或在选项文件中指定选项。许多选项可以在服务器运行时通过使用`CHANGE REPLICATION SOURCE TO`语句（从 MySQL 8.0.23 开始）或`CHANGE MASTER TO`语句（在 MySQL 8.0.23 之前）进行设置。使用`SET`语法指定系统变量值。

**服务器 ID。** 在源和每个副本上，您必须设置`server_id`系统变量以建立范围从 1 到 2³² − 1 的唯一复制 ID。 “唯一”意味着每个 ID 必须与复制拓扑中任何其他源或副本正在使用的任何其他 ID 不同。示例`my.cnf`文件：

```sql
[mysqld]
server-id=3
```

##### 副本服务器的启动选项

本节介绍了控制副本服务器启动选项的内容。其中许多选项可以在服务器运行时通过使用`CHANGE REPLICATION SOURCE TO`语句（从 MySQL 8.0.23 开始）或`CHANGE MASTER TO`语句（在 MySQL 8.0.23 之前）进行设置。其他选项，如`--replicate-*`选项，只能在副本服务器启动时设置。有关复制相关的系统变量将在本节后面讨论。

+   `--master-info-file=*`file_name`*`

    | 命令行格式 | `--master-info-file=file_name` |
    | --- | --- |
    | 已弃用 | 8.0.18 |
    | 类型 | 文件名 |
    | 默认值 | `master.info` |

    此选项的使用现已不推荐。如果设置了`master_info_repository=FILE`，则用于设置复制品连接元数据存储库的文件名。`--master-info-file`和`master_info_repository`系统变量的使用已不推荐，因为使用文件作为连接元数据存储库已被崩溃安全表取代。有关连接元数据存储库的信息，请参阅 Section 19.2.4.2, “Replication Metadata Repositories”。

+   `--master-retry-count=*`count`*`

    | 命令行格式 | `--master-retry-count=#` |
    | --- | --- |
    | 已弃用 | 是 |
    | 类型 | 整数 |
    | 默认值 | `86400` |
    | 最小值 | `0` |
    | 最大值（64 位平台） | `18446744073709551615` |
    | 最大值（32 位平台） | `4294967295` |

    复制品在放弃之前尝试重新连接到源的次数。默认值为 86400 次。值为 0 表示“无限”，复制品会永远尝试连接。当复制品在未从源接收数据或心跳信号的情况下达到连接超时（由`replica_net_timeout`或`slave_net_timeout`系统变量指定）时，会触发重新连接尝试。重新连接尝试的间隔由`CHANGE REPLICATION SOURCE TO` | `CHANGE MASTER TO`语句的`SOURCE_CONNECT_RETRY` | `MASTER_CONNECT_RETRY`选项设置（默认为每 60 秒一次）。

    此选项已弃用；预计将在未来的 MySQL 版本中删除。请改用`CHANGE REPLICATION SOURCE TO` | `CHANGE MASTER TO`语句的`SOURCE_RETRY_COUNT` | `MASTER_RETRY_COUNT`选项。

+   `--max-relay-log-size=*`size`*`

    | 命令行格式 | `--max-relay-log-size=#` |
    | --- | --- |
    | 系统变量 | `max_relay_log_size` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `0` |
    | 最小值 | `0` |
    | 最大值 | `1073741824` |
    | 单位 | 字节 |
    | 块大小 | `4096` |

    服务器自动轮换中继日志文件的大小。如果此值非零，则当中继日志大小超过此值时，中继日志会自动轮换。如果此值为零（默认值），则中继日志轮换发生的大小由`max_binlog_size`的值确定。更多信息，请参阅 Section 19.2.4.1, “The Relay Log”。

+   `--relay-log-purge={0|1}`

    | 命令行格式 | `--relay-log-purge[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `relay_log_purge` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `ON` |

    禁用或启用中继日志在不再需要时自动清除。默认值为 1（启用）。这是一个可以通过`SET GLOBAL relay_log_purge = *`N`*`动态更改的全局变量。在启用`--relay-log-recovery`选项时禁用中继日志清除会存在数据一致性风险，因此不具有崩溃安全性。

+   `--relay-log-space-limit=*`size`*`

    | 命令行格式 | `--relay-log-space-limit=#` |
    | --- | --- |
    | 系统变量 | `relay_log_space_limit` |
    | 作用范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `0` |
    | 最小值 | `0` |
    | 最大值 | `18446744073709551615` |
    | 单位 | 字节 |

    此选项将在副本上的所有中继日志的总大小上设置一个上限，以字节为单位。值为 0 表示“无限制”。这对于具有有限磁盘空间的副本服务器主机非常有用。当达到限制时，I/O（接收器）线程停止从源服务器读取二进制日志事件，直到 SQL 线程赶上并删除一些未使用的中继日志。请注意，此限制并非绝对：在某些情况下，SQL（应用程序）线程需要更多事件才能删除中继日志。在这种情况下，接收器线程超过限制，直到应用程序线程能够删除一些中继日志，因为不这样做会导致死锁。不应将`--relay-log-space-limit`设置为小于`--max-relay-log-size`的两倍（或`--max-binlog-size`如果`--max-relay-log-size`为 0）。在这种情况下，接收器线程可能会等待空闲空间，因为`--relay-log-space-limit`已超过，但应用程序线程没有中继日志可清除，无法满足接收器线程。这会迫使接收器线程暂时忽略`--relay-log-space-limit`。

+   `--replicate-do-db=*`db_name`*`

    | 命令行格式 | `--replicate-do-db=name` |
    | --- | --- |
    | 类型 | 字符串 |

    使用数据库名称创建一个复制过滤器。也可以使用`CHANGE REPLICATION FILTER REPLICATE_DO_DB`来创建这样的过滤器。

    此选项支持通道特定的复制过滤器，使多源副本可以为不同来源使用特定的过滤器。要在名为*`channel_1`*的通道上配置特定于通道的复制过滤器，请使用`--replicate-do-db:*`channel_1`*:*`db_name`*`。在这种情况下，第一个冒号被解释为分隔符，后续冒号为字面冒号。有关更多信息，请参见第 19.2.5.4 节，“基于通道的复制过滤器”。

    注意

    全局复制过滤器不能用于配置为组复制的 MySQL 服务器实例，因为在某些服务器上过滤事务会使组无法达成一致状态。通道特定的复制过滤器可以用于与组复制无直接关系的复制通道，例如，其中一个组成员同时充当到组外源的副本。它们不能用于 `group_replication_applier` 或 `group_replication_recovery` 通道。

    这个复制过滤器的精确效果取决于是使用基于语句还是基于行的复制。

    **基于语句的复制。** 告诉复制 SQL 线程将复制限制在默认数据库（即由 `USE` 选择的数据库）为 *`db_name`* 的语句上。要指定多个数据库，请多次使用此选项，每个数据库使用一次；但是，这样做 *不会* 复制跨数据库语句，例如 `UPDATE *`some_db.some_table`* SET foo='bar'`，而选择了不同的数据库（或没有选择数据库）。

    警告

    要指定多个数据库，*必须* 使用此选项的多个实例。因为数据库名称可以包含逗号，如果提供逗号分隔的列表，则该列表被视为单个数据库的名称。

    使用基于语句的复制时不像您期望的那样工作的示例：如果副本是使用 `--replicate-do-db=sales` 启动的，并且您在源上发出以下语句，则 `UPDATE` 语句 *不会* 被复制：

    ```sql
    USE prices;
    UPDATE sales.january SET amount=amount+1000;
    ```

    这种“仅检查默认数据库”行为的主要原因是，仅从语句本身很难知道是否应该复制它（例如，如果您正在使用多表 `DELETE` 语句或跨多个数据库操作的多表 `UPDATE` 语句）。如果没有必要，仅检查默认数据库而不是所有数据库会更快。

    **基于行的复制。** 告诉复制 SQL 线程将复制限制在数据库 *`db_name`* 上。只有属于 *`db_name`* 的表会被更改；当前数据库对此没有影响。假设副本是使用 `--replicate-do-db=sales` 启动的，并且基于行的复制生效，然后在源上运行以下语句：

    ```sql
    USE prices;
    UPDATE sales.february SET amount=amount+100;
    ```

    副本上的`sales`数据库中的`february`表根据`UPDATE`语句进行更改；无论是否发出了`USE`语句都会发生这种情况。但是，在源上发出以下语句时，当使用行复制和`--replicate-do-db=sales`时，副本上没有影响：

    ```sql
    USE prices;
    UPDATE prices.march SET amount=amount-25;
    ```

    即使语句`USE prices`被更改为`USE sales`，`UPDATE`语句的效果仍不会被复制。

    另一个在语句复制和行复制中处理`--replicate-do-db`的重要区别涉及到涉及多个数据库的语句。假设副本使用`--replicate-do-db=db1`启动，并且在源上执行以下语句：

    ```sql
    USE db1;
    UPDATE db1.table1, db2.table2 SET db1.table1.col1 = 10, db2.table2.col2 = 20;
    ```

    如果您使用语句复制，那么副本上的两个表都会被更新。但是，当使用行复制时，副本上只有`table1`受到影响；因为`table2`位于不同的数据库中，所以副本上的`table2`不会被`UPDATE`更改。现在假设，而不是`USE db1`语句，使用了`USE db4`语句：

    ```sql
    USE db4;
    UPDATE db1.table1, db2.table2 SET db1.table1.col1 = 10, db2.table2.col2 = 20;
    ```

    在这种情况下，当使用语句复制时，`UPDATE`语句对副本没有影响。但是，如果使用行复制，则`UPDATE`会更改副本上的`table1`，但不会更改`table2`—换句话说，只有由`--replicate-do-db`命名的数据库中的表会被更改，默认数据库的选择对此行为没有影响。

    如果需要跨数据库更新生效，请改用`--replicate-wild-do-table=*`db_name`*.%`。参见第 19.2.5 节，“服务器如何评估复制过滤规则”。

    注意

    此选项对复制的影响与`--binlog-do-db`对二进制日志记录的影响方式相同，并且复制格式对`--replicate-do-db`对复制行为的影响与日志格式对`--binlog-do-db`行为的影响相同。

    此选项对`BEGIN`、`COMMIT`或`ROLLBACK`语句无效。

+   `--replicate-ignore-db=*`db_name`*`

    | 命令行格式 | `--replicate-ignore-db=name` |
    | --- | --- |
    | 类型 | 字符串 |

    使用数据库名称创建一个复制过滤器。也可以使用`CHANGE REPLICATION FILTER REPLICATE_IGNORE_DB`来创建这样的过滤器。

    此选项支持通道特定的复制过滤器，使多源复制副本可以为不同源使用特定过滤器。要在名为*`channel_1`*的通道上配置通道特定的复制过滤器，请使用`--replicate-ignore-db:*`channel_1`*:*`db_name`*`。在这种情况下，第一个冒号被解释为分隔符，后续冒号为字面冒号。有关更多信息，请参见 Section 19.2.5.4, “Replication Channel Based Filters”。

    注意

    全局复制过滤器不能在配置为组复制的 MySQL 服务器实例上使用，因为在某些服务器上过滤事务会导致组无法达成一致状态。通道特定的复制过滤器可以用于与组复制无直接关系的复制通道，例如，其中一个组成员同时充当了组外源的副本。它们不能用于`group_replication_applier`或`group_replication_recovery`通道。

    要忽略多个数据库，请多次使用此选项，每个数据库一次。因为数据库名称可以包含逗号，如果提供逗号分隔的列表，则将其视为单个数据库的名称。

    与`--replicate-do-db`一样，此过滤的精确效果取决于是否使用基于语句或基于行的复制，并在接下来的几段中描述。

    **基于语句的复制。** 告诉复制 SQL 线程不要复制默认数据库（即由`USE`选择的数据库）为*`db_name`*的任何语句。

    **基于行的复制。** 告诉复制 SQL 线程不要更新数据库*`db_name`*中的任何表。默认数据库不受影响。

    在使用基于语句的复制时，以下示例不会按您期望的方式工作。假设副本是使用`--replicate-ignore-db=sales`启动的，并且您在源上发出以下语句：

    ```sql
    USE prices;
    UPDATE sales.january SET amount=amount+1000;
    ```

    在这种情况下，`UPDATE`语句会被复制，因为`--replicate-ignore-db`仅适用于默认数据库（由`USE`语句确定）。由于`sales`数据库在语句中被明确指定，因此该语句未被过滤。但是，在使用基于行的复制时，`UPDATE`语句的效果不会传播到副本，副本中的`sales.january`表的副本保持不变；在这种情况下，`--replicate-ignore-db=sales`导致源数据库中`sales`数据库中的所有表的更改都被副本忽略。

    如果您正在使用跨数据库更新并且不希望这些更新被复制，请不要使用此选项。请参阅 Section 19.2.5, “How Servers Evaluate Replication Filtering Rules”。

    如果需要跨数据库更新正常工作，请改用`--replicate-wild-ignore-table=*`db_name`*.%`。请参阅 Section 19.2.5, “How Servers Evaluate Replication Filtering Rules”。

    注意

    此选项对复制的影响方式与`--binlog-ignore-db`对二进制日志的影响方式相同，并且复制格式对`--replicate-ignore-db`对复制行为的影响与日志格式对`--binlog-ignore-db`行为的影响相同。

    此选项对`BEGIN`、`COMMIT`或`ROLLBACK`语句没有影响。

+   `--replicate-do-table=*`db_name.tbl_name`*`

    | 命令行格式 | `--replicate-do-table=name` |
    | --- | --- |
    | 类型 | 字符串 |

    通过告诉复制 SQL 线程限制复制到给定表来创建复制过滤器。要指定多个表，多次使用此选项，每次为一个表。与`--replicate-do-db`不同，这适用于跨数据库更新和默认数据库更新。参见第 19.2.5 节，“服务器如何评估复制过滤规则”。您还可以通过发出`CHANGE REPLICATION FILTER REPLICATE_DO_TABLE`语句来创建这样的过滤器。

    此选项支持通道特定的复制过滤器，使多源复制可以为不同源使用特定过滤器。要在名为*`channel_1`*的通道上配置通道特定的复制过滤器，请使用`--replicate-do-table:*`channel_1`*:*`db_name.tbl_name`*`。在这种情况下，第一个冒号被解释为分隔符，后续冒号为字面冒号。有关更多信息，请参见第 19.2.5.4 节，“基于复制通道的过滤器”。

    注意

    全局复制过滤器不能用于配置为组复制的 MySQL 服务器实例，因为在某些服务器上过滤事务会导致组无法达成一致状态。通道特定的复制过滤器可以用于与组复制无直接关系的复制通道，例如，其中一个组成员同时充当一个在组外的源的复制。它们不能用于`group_replication_applier`或`group_replication_recovery`通道。

    此选项仅影响适用于表的语句。它不影响仅适用于其他数据库对象（如存储过程）的语句。要过滤操作存储过程的语句，请使用一个或多个`--replicate-*-db`选项。

+   `--replicate-ignore-table=*`db_name.tbl_name`*`

    | 命令行格式 | `--replicate-ignore-table=name` |
    | --- | --- |
    | 类型 | 字符串 |

    通过告诉复制 SQL 线程不复制更新指定表的任何语句来创建一个复制过滤器，即使同一语句可能更新其他表。要忽略多个表，请多次使用此选项，每次针对一个表。这适用于跨数据库更新，与`--replicate-ignore-db`相反。请参见第 19.2.5 节，“服务器如何评估复制过滤规则”。您还可以通过发出`CHANGE REPLICATION FILTER REPLICATE_IGNORE_TABLE`语句来创建这样的过滤器。

    此选项支持通道特定的复制过滤器，使得多源复制可以为不同源使用特定的过滤器。要在名为*`channel_1`*的通道上配置通道特定的复制过滤器，请使用`--replicate-ignore-table:*`channel_1`*:*`db_name.tbl_name`*`。在这种情况下，第一个冒号被解释为分隔符，后续的冒号是字面冒号。有关更多信息，请参见第 19.2.5.4 节，“基于通道的复制过滤器”。

    注意

    全局复制过滤器不能用于配置为组复制的 MySQL 服务器实例，因为在某些服务器上过滤事务会导致组无法达成一致状态。通道特定的复制过滤器可以用于与组复制无直接关系的复制通道，例如，其中一个组成员同时充当组外源的副本。它们不能用于`group_replication_applier`或`group_replication_recovery`通道。

    此选项仅影响适用于表的语句。它不影响仅适用于其他数据库对象的语句，如存储过程。要过滤操作存储过程的语句，请使用一个或多个`--replicate-*-db`选项。

+   `--replicate-rewrite-db=*`from_name`*->*`to_name`*`

    | 命令行格式 | `--replicate-rewrite-db=old_name->new_name` |
    | --- | --- |
    | 类型 | 字符串 |

    告诉副本创建一个复制过滤器，如果源上的指定数据库是*`from_name`*，则将其转换为*`to_name`*。只有涉及表的语句会受到影响，不包括`CREATE DATABASE`、`DROP DATABASE`和`ALTER DATABASE`等语句。

    要指定多个重写，请多次使用此选项。服务器将使用第一个匹配*`from_name`*值的重写。数据库名称转换是在测试`--replicate-*`规则之前完成的。您还可以通过发出`CHANGE REPLICATION FILTER REPLICATE_REWRITE_DB`语句来创建这样的过滤器。

    如果您在命令行上使用`--replicate-rewrite-db`选项，并且`>`字符对您的命令解释器很重要，请引用选项值。例如：

    ```sql
    $> mysqld --replicate-rewrite-db="*olddb*->*newdb*"
    ```

    `--replicate-rewrite-db`选项的效果取决于查询使用的基于语句还是基于行的二进制日志格式而有所不同。对于基于语句的格式，DML 语句基于由`USE`语句指定的当前数据库进行转换。对于基于行的格式，DML 语句基于修改表所在的数据库进行转换。DDL 语句始终基于由`USE`语句指定的当前数据库进行过滤，而不考虑二进制日志格式。

    为确保重写产生预期结果，特别是与其他复制过滤选项结合使用时，请在使用`--replicate-rewrite-db`选项时遵循以下建议：

    +   在源和副本上手动创建*`from_name`*和*`to_name`*数据库，名称不同。

    +   如果您使用基于语句或混合二进制日志格式，请不要使用跨数据库查询，并且不要在查询中指定数据库名称。对于 DDL 和 DML 语句，依赖`USE`语句来指定当前数据库，并且在查询中只使用表名。

    +   如果您仅使用基于行的二进制日志格式，对于 DDL 语句，依赖`USE`语句来指定当前数据库，并且在查询中只使用表名。对于 DML 语句，如果需要，可以使用完全限定的表名（*`db`*.*`table`*）。

    如果遵循这些建议，结合表级复制过滤选项（如`--replicate-do-table`），使用`--replicate-rewrite-db`选项是安全的。

    此选项支持通道特定的复制过滤器，使多源副本能够为不同源使用特定的过滤器。指定通道名称，后跟冒号，再跟过滤器规范。第一个冒号被解释为分隔符，任何后续的冒号都被解释为字面冒号。例如，要在名为*`channel_1`*的通道上配置通道特定的复制过滤器，请使用：

    ```sql
    $> mysqld --replicate-rewrite-db=*channel_1*:*db_name1*->*db_name2*
    ```

    如果使用冒号但未指定通道名称，则该选项会为默认复制通道配置复制过滤器。有关更多信息，请参见第 19.2.5.4 节，“基于复制通道的过滤器”。

    注意

    全局复制过滤器不能在配置为组复制的 MySQL 服务器实例上使用，因为在某些服务器上过滤事务会导致组无法达成一致状态。通道特定的复制过滤器可以用于与组复制无直接关系的复制通道，例如，其中一个组成员同时充当了组外源的副本。它们不能用于`group_replication_applier`或`group_replication_recovery`通道。

+   `--replicate-same-server-id`

    | 命令行格式 | `--replicate-same-server-id[={OFF&#124;ON}]` |
    | --- | --- |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    此选项用于副本。默认值为 0（`FALSE`）。将此选项设置为 1（`TRUE`）时，副本不会跳过具有自己服务器 ID 的事件。这个设置通常只在罕见的配置中有用。

    当在副本上启用二进制日志记录时，副本上的`--replicate-same-server-id`和`--log-slave-updates`选项的组合可能导致在服务器是循环复制拓扑的一部分时出现复制中的无限循环。（在 MySQL 8.0 中，默认情况下启用二进制日志记录，并且在启用二进制日志记录时，默认情况下启用副本更新日志记录。）然而，全局事务标识符（GTID）的使用通过跳过已应用的事务的执行来防止这种情况。如果在副本上设置了`gtid_mode=ON`，则可以使用这些选项的组合启动服务器，但在服务器运行时不能更改为任何其他 GTID 模式。如果设置了任何其他 GTID 模式，则服务器不会以这些选项的组合启动。

    默认情况下，复制 I/O（接收器）线程不会将具有副本服务器 ID 的二进制日志事件写入中继日志（此优化有助于节省磁盘使用量）。如果要使用`--replicate-same-server-id`，请确保在使副本读取要复制 SQL（applier）线程执行的自己事件之前，使用此选项启动副本。

+   `--replicate-wild-do-table=*`db_name.tbl_name`*`

    | 命令行格式 | `--replicate-wild-do-table=name` |
    | --- | --- |
    | 类型 | 字符串 |

    通过告知复制 SQL（applier）线程，创建一个复制过滤器，限制复制到更新表匹配指定数据库和表名模式的语句。模式可以包含`%`和`_`通配符，其含义与`LIKE`模式匹配运算符相同。要指定多个表，需要多次使用此选项，每个表使用一次。这适用于跨数据库更新。参见 Section 19.2.5, “服务器如何评估复制过滤规则”。您也可以通过发出`CHANGE REPLICATION FILTER REPLICATE_WILD_DO_TABLE`语句来创建这样的过滤器。

    此选项支持通道特定的复制过滤器，使多源副本能够为不同源使用特定过滤器。要在名为*`channel_1`*的通道上配置通道特定的复制过滤器，请使用`--replicate-wild-do-table:*`channel_1`*:*`db_name.tbl_name`*`。在这种情况下，第一个冒号被解释为分隔符，后续冒号为字面冒号。有关更多信息，请参见 Section 19.2.5.4, “基于复制通道的过滤器”。

    重要

    全局复制过滤器不能在配置为组复制的 MySQL 服务器实例上使用，因为在某些服务器上过滤事务会导致组无法达成一致状态。通道特定的复制过滤器可以用于与组复制无直接关系的复制通道，例如，组成员同时充当源站外组的副本。它们不能用于`group_replication_applier`或`group_replication_recovery`通道。

    由`--replicate-wild-do-table`选项指定的复制过滤器适用于表、视图和触发器。它不适用于存储过程和函数，或事件。要过滤操作后者对象的语句，使用一个或多个`--replicate-*-db`选项。

    例如，`--replicate-wild-do-table=foo%.bar%`仅复制使用数据库名称以`foo`开头且表名以`bar`开头的表的更新。

    如果表名模式为`%`，它将匹配任何表名，并且该选项也适用于数据库级语句（`CREATE DATABASE`，`DROP DATABASE`和`ALTER DATABASE`）。例如，如果您使用`--replicate-wild-do-table=foo%.%`，如果数据库名称与模式`foo%`匹配，则数据库级语句将被复制。

    重要

    表级复制过滤器仅应用于在查询中明确提及并操作的表。它们不适用于查询隐式更新的表。例如，一个`GRANT`语句，更新`mysql.user`系统表但未提及该表，不受指定`mysql.%`作为通配符模式的过滤器的影响。

    要在数据库或表名模式中包含字面通配符字符，请使用反斜杠进行转义。例如，要复制名为`my_own%db`的数据库的所有表，但不复制`my1ownAABCdb`数据库的表，您应该像这样转义`_`和`%`字符：`--replicate-wild-do-table=my\_own\%db`。如果您在命令行上使用该选项，可能需要双倍反斜杠或引用选项值，具体取决于您的命令解释器。例如，在**bash** shell 中，您需要键入`--replicate-wild-do-table=my\\_own\\%db`。

+   `--replicate-wild-ignore-table=*`db_name.tbl_name`*`

    | 命令行格式 | `--replicate-wild-ignore-table=name` |
    | --- | --- |
    | 类型 | 字符串 |

    创建一个复制过滤器，使复制 SQL 线程不会复制任何表匹配给定通配符模式的语句。要指定要忽略的多个表，请多次使用此选项，每个表使用一次。这适用于跨数据库更新。请参阅第 19.2.5 节，“服务器如何评估复制过滤规则”。您还可以通过发出`CHANGE REPLICATION FILTER REPLICATE_WILD_IGNORE_TABLE`语句来创建此类过滤器。

    此选项支持通道特定的复制过滤器，使多源副本可以为不同源使用特定过滤器。要在名为*`channel_1`*的通道上配置通道特定的复制过滤器，请使用`--replicate-wild-ignore:*`channel_1`*:*`db_name.tbl_name`*`。在这种情况下，第一个冒号被解释为分隔符，后续冒号为字面冒号。有关更多信息，请参阅第 19.2.5.4 节，“基于复制通道的过滤器”。

    重要

    全局复制过滤器不能用于配置了组复制的 MySQL 服务器实例，因为在某些服务器上过滤事务会导致组无法达成一致状态。通道特定的复制过滤器可以用于与组复制无直接关系的复制通道，例如，其中一个组成员同时充当源站外组的副本。它们不能用于`group_replication_applier`或`group_replication_recovery`通道。

    例如，`--replicate-wild-ignore-table=foo%.bar%` 不会复制使用以`foo`开头的数据库和以`bar`开头的表的更新。有关匹配原理的信息，请参阅`--replicate-wild-do-table`选项的描述。在选项值中包含字面通配符的规则与`--replicate-wild-ignore-table`相同。

    重要

    表级复制过滤器仅适用于在查询中明确提及和操作的表。它们不适用于查询隐式更新的表。例如，一个`GRANT`语句，更新`mysql.user`系统表但未提及该表，不受指定`mysql.%`作为通配符模式的过滤器影响。

    如果需要过滤掉`GRANT`语句或其他管理语句，一个可能的解决方法是使用`--replicate-ignore-db`过滤器。该过滤器作用于当前生效的默认数据库，由`USE`语句确定。因此，您可以创建一个过滤器来忽略未复制的数据库的语句，然后在要忽略的任何管理语句之前立即发出`USE`语句以切换默认数据库到该数据库。在管理语句中，命名应用语句的实际数据库。

    例如，如果在复制服务器上配置了 `--replicate-ignore-db=nonreplicated`，则以下语句序列会导致 `GRANT` 语句被忽略，因为默认数据库 `nonreplicated` 生效：

    ```sql
    USE nonreplicated;
    GRANT SELECT, INSERT ON replicated.t1 TO 'someuser'@'somehost';
    ```

+   `--skip-replica-start`

    | 命令行格式 | `--skip-replica-start[={OFF&#124;ON}]` |
    | --- | --- |
    | 引入版本 | 8.0.26 |
    | 系统变量 | `skip_replica_start` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔 |
    | 默认值 | `OFF` |

    从 MySQL 8.0.26 开始，请使用 `--skip-replica-start` 替代 `--skip-slave-start`，后者在该版本中已被弃用。在 MySQL 8.0.26 之前的版本中，请使用 `--skip-slave-start`。

    `--skip-replica-start` 告诉复制服务器在启动时不要启动复制 I/O（接收器）和 SQL（应用程序）线程。要稍后启动线程，请使用 `START REPLICA` 语句。

    您可以使用 `skip_replica_start` 系统变量代替命令行选项，以允许通过 MySQL 服务器的权限结构访问此功能，从而数据库管理员无需任何操作系统的特权访问。

+   `--skip-slave-start`

    | 命令行格式 | `--skip-slave-start[={OFF&#124;ON}]` |
    | --- | --- |
    | 已弃用 | 8.0.26 |
    | 系统变量 | `skip_slave_start` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔 |
    | 默认值 | `OFF` |

    从 MySQL 8.0.26 开始，`--skip-slave-start` 已被弃用，应改用别名 `--skip-replica-start`。在 MySQL 8.0.26 之前的版本中，请使用 `--skip-slave-start`。

    告诉复制服务器在启动时不要启动复制 I/O（接收器）和 SQL（应用程序）线程。要稍后启动线程，请使用 `START REPLICA` 语句。

    从 MySQL 8.0.24 开始，您可以使用`skip_slave_start`系统变量来代替命令行选项，以允许使用 MySQL 服务器的权限结构访问此功能，这样数据库管理员就不需要对操作系统拥有任何特权访问。

+   [`--slave-skip-errors=[*`err_code1`*,*`err_code2`*,...|all|ddl_exist_errors]`](replication-options-replica.html#option_mysqld_slave-skip-errors)

    | 命令行格式 | `--slave-skip-errors=name` |
    | --- | --- |
    | 已弃用 | 8.0.26 |
    | 系统变量 | `slave_skip_errors` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 字符串 |
    | 默认值 | `OFF` |
    | 有效值 | `OFF``[错误代码列表]``all``ddl_exist_errors` |

    通常，当副本发生错误时，复制会停止，这为您提供了手动解决数据不一致性的机会。此选项使复制 SQL 线程在语句返回选项值中列出的任何错误时继续复制。

    除非您完全理解为什么会出现错误，否则不要使用此选项。如果您的复制设置和客户端程序中没有错误，MySQL 本身也没有错误，那么应该永远不会发生导致复制停止的错误。滥用此选项会导致副本与源头彻底失去同步，而您却不知道为什么会发生这种情况。

    对于错误代码，您应该使用副本错误日志和`SHOW REPLICA STATUS`输出中提供的错误消息中的数字。附录 B，*错误消息和常见问题*列出了服务器错误代码。

    简写值`ddl_exist_errors`等同于错误代码列表`1007,1008,1050,1051,1054,1060,1061,1068,1094,1146`。

    您也可以（但不应该）使用非常不推荐的值`all`，使副本忽略所有错误消息并继续进行，无论发生什么情况。不用说，如果您使用`all`，则关于数据完整性就没有任何保证。如果副本的数据与源头的数据差距很大，请不要在这种情况下抱怨（或提交错误报告）。*您已经被警告*。

    在复制 NDB 集群之间时，此选项的工作方式与复制内部`NDB`机制检查时期序列号不同；通常，一旦`NDB`检测到缺失或其他序列不正确的时期号，它立即停止副本应用程序线程。从 NDB 8.0.28 开始，您可以通过同时指定`--ndb-applier-allow-skip-epoch`和`--slave-skip-errors`来覆盖此行为；这样做会导致`NDB`忽略跳过的时期事务。

    示例:

    ```sql
    --slave-skip-errors=1062,1053
    --slave-skip-errors=all
    --slave-skip-errors=ddl_exist_errors
    ```

+   `--slave-sql-verify-checksum={0|1}`

    | 命令行格式 | `--slave-sql-verify-checksum[={OFF&#124;ON}]` |
    | --- | --- |
    | 类型 | 布尔值 |
    | 默认值 | `ON` |

    启用此选项时，副本会检查从中继日志中读取的校验和。如果不匹配，则副本会出现错误。

以下选项仅在 MySQL 测试套件内部用于复制测试和调试。不适用于生产环境。

+   `--abort-slave-event-count`

    | 命令行格式 | `--abort-slave-event-count=#` |
    | --- | --- |
    | 已弃用 | 8.0.29 |
    | 类型 | 整数 |
    | 默认值 | `0` |
    | 最小值 | `0` |

    当将此选项设置为除 0（默认值）之外的某个正整数*`value`*时，它会影响复制行为如下：在复制 SQL 线程启动后，允许执行*`value`*个日志事件；之后，复制 SQL 线程不再接收任何事件，就好像源端的网络连接被切断一样。复制 SQL 线程继续运行，并且`SHOW REPLICA STATUS`的输出在`Replica_IO_Running`和`Replica_SQL_Running`列中都显示`Yes`，但不再从中继日志中读取更多事件。

    此选项仅在 MySQL 测试套件内部用于复制测试和调试。不适用于生产环境。从 MySQL 8.0.29 开始，已弃用，并可能在将来的 MySQL 版本中移除。

+   `--disconnect-slave-event-count`

    | 命令行格式 | `--disconnect-slave-event-count=#` |
    | --- | --- |
    | 已弃用 | 8.0.29 |
    | 类型 | 整数 |
    | 默认值 | `0` |

    此选项仅在 MySQL 测试套件内部用于复制测试和调试。不适用于生产环境。从 MySQL 8.0.29 开始，已弃用，并可能在将来的 MySQL 版本中移除。

##### 用于副本服务器的系统变量

以下列表描述了用于控制复制服务器的系统变量。它们可以在服务器启动时设置，其中一些可以在运行时使用 `SET` 进行更改。用于复制的服务器选项在本节的前面列出。

+   `init_replica`

    | 命令行格式 | `--init-replica=name` |
    | --- | --- |
    | 引入 | 8.0.26 |
    | 系统变量 | `init_replica` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 字符串 |

    从 MySQL 8.0.26 开始，请使用 `init_replica` 代替已在该版本中弃用的 `init_slave`。在 MySQL 8.0.26 之前的版本中，请使用 `init_slave`。

    `init_replica` 类似于 `init_connect`，但是是每次复制 SQL 线程启动时由复制服务器执行的字符串。该字符串的格式与 `init_connect` 变量相同。此变量的设置将对后续的 `START REPLICA` 生效。

    注意

    复制 SQL 线程在执行 `init_replica` 之前向客户端发送确认。因此，在 `START REPLICA` 返回之前，不能保证已执行 `init_replica`。有关更多信息，请参见 Section 15.4.2.6, “START REPLICA 语句”。

+   `init_slave`

    | 命令行格式 | `--init-slave=name` |
    | --- | --- |
    | 已弃用 | 8.0.26 |
    | 系统变量 | `init_slave` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 字符串 |

    从 MySQL 8.0.26 开始，`init_slave` 已被弃用，应改用别名 `init_replica`。在 MySQL 8.0.26 之前的版本中，请使用 `init_slave`。

    `init_slave` 类似于 `init_connect`，但是是一个字符串，每次复制 SQL 线程启动时由副本服务器执行。字符串的格式与 `init_connect` 变量相同。此变量的设置将对后续的 `START REPLICA` 语句生效。

    注意

    复制 SQL 线程在执行 `init_slave` 之前向客户端发送确认。因此，在 `START REPLICA` 返回之前，不能保证 `init_slave` 已执行。有关更多信息，请参见 Section 15.4.2.6, “START REPLICA Statement”。

+   `log_slow_replica_statements`

    | 命令行格式 | `--log-slow-replica-statements[={OFF&#124;ON}]` |
    | --- | --- |
    | 引入版本 | 8.0.26 |
    | 系统变量 | `log_slow_replica_statements` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    从 MySQL 8.0.26 开始，使用 `log_slow_replica_statements` 替代从该版本开始弃用的 `log_slow_slave_statements`。在 MySQL 8.0.26 之前的版本中，请使用 `log_slow_slave_statements`。

    当慢查询日志启用时，`log_slow_replica_statements` 启用记录在副本上执行时间超过 `long_query_time` 秒的查询。请注意，如果使用基于行的复制（`binlog_format=ROW`），则 `log_slow_replica_statements` 不起作用。仅当以语句格式在二进制日志中记录查询时，即设置 `binlog_format=STATEMENT` 或设置 `binlog_format=MIXED` 且语句以语句格式记录时，才将查询添加到副本的慢查询日志中。即使启用了 `log_slow_replica_statements`，当以行格式记录慢查询时，当设置 `binlog_format=MIXED` 时，或者当设置 `binlog_format=ROW` 时记录时，这些慢查询不会添加到副本的慢查询日志中。

    设置 `log_slow_replica_statements` 没有立即效果。该变量的状态适用于所有后续的 `START REPLICA` 语句。还要注意，全局设置的 `long_query_time` 适用于 SQL 线程的生命周期。如果更改了该设置，必须停止并重新启动复制 SQL 线程以实施更改（例如，通过使用带有 `SQL_THREAD` 选项的 `STOP REPLICA` 和 `START REPLICA` 语句）。

+   `log_slow_slave_statements`

    | 命令行格式 | `--log-slow-slave-statements[={OFF&#124;ON}]` |
    | --- | --- |
    | 已弃用 | 8.0.26 |
    | 系统变量 | `log_slow_slave_statements` |
    | 作用域 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    从 MySQL 8.0.26 开始，`log_slow_slave_statements` 已被弃用，应改用别名 `log_slow_replica_statements`。在 MySQL 8.0.26 之前的版本中，请使用 `log_slow_slave_statements`。

    当慢查询日志被启用时，`log_slow_slave_statements` 会记录在副本上执行时间超过`long_query_time`秒的查询。请注意，如果使用基于行的复制（`binlog_format=ROW`），`log_slow_slave_statements` 不会生效。只有当查询以语句格式在二进制日志中记录时，即当设置`binlog_format=STATEMENT`，或者当设置`binlog_format=MIXED`并且语句以语句格式记录时，查询才会被添加到副本的慢查询日志中。即使启用了`log_slow_slave_statements`，当以行格式记录慢查询时，当设置`binlog_format=MIXED`时，或者当设置`binlog_format=ROW`时记录时，这些慢查询也不会被添加到副本的慢查询日志中。

    设置`log_slow_slave_statements` 没有立即生效。该变量的状态适用于所有后续的`START REPLICA`语句。还要注意，全局设置的`long_query_time`适用于 SQL 线程的生命周期。如果更改了该设置，必须停止并重新启动复制 SQL 线程以在那里实施更改（例如，通过使用带有`SQL_THREAD`选项的`STOP REPLICA`和`START REPLICA`语句）。

+   `master_info_repository`

    | 命令行格式 | `--master-info-repository={FILE&#124;TABLE}` |
    | --- | --- |
    | 已弃用 | 8.0.23 |
    | 系统变量 | `master_info_repository` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 字符串 |
    | 默认值 | `TABLE` |
    | 有效值 | `FILE``TABLE` |

    现在已弃用此系统变量的使用。设置`TABLE`是默认值，并且在配置多个复制通道时是必需的。曾经弃用的替代设置`FILE`。

    默认设置下，副本将关于源的元数据（包括状态和连接信息）记录到 `mysql` 系统数据库中的 `mysql.slave_master_info` 表中的 `InnoDB` 表中。有关连接元数据存储库的更多信息，请参见 第 19.2.4 节，“中继日志和复制元数据存储库”。

    `FILE` 设置将副本的连接元数据存储库写入一个文件，默认情况下命名为 `master.info`。可以使用 `--master-info-file` 选项更改名称。

+   `max_relay_log_size`

    | 命令行格式 | `--max-relay-log-size=#` |
    | --- | --- |
    | 系统变量 | `max_relay_log_size` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `0` |
    | 最小值 | `0` |
    | 最大值 | `1073741824` |
    | 单位 | 字节 |
    | 块大小 | `4096` |

    如果副本对其中继日志的写入导致当前日志文件大小超过此变量的值，则副本会旋转中继日志（关闭当前文件并打开下一个文件）。如果 `max_relay_log_size` 为 0，则服务器对二进制日志和中继日志均使用 `max_binlog_size`。如果 `max_relay_log_size` 大于 0，则限制中继日志的大小，这使您可以为两个日志设置不同的大小。必须将 `max_relay_log_size` 设置为介于 4096 字节和 1GB（包括）之间，或为 0。默认值为 0。请参阅 第 19.2.3 节，“复制线程”。

+   `relay_log`

    | 命令行格式 | `--relay-log=file_name` |
    | --- | --- |
    | 系统变量 | `relay_log` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 文件名 |

    中继日志文件的基本名称。对于默认复制通道，中继日志的默认基本名称为 `*`host_name`*-relay-bin`。对于非默认复制通道，中继日志的默认基本名称为 `*`host_name`*-relay-bin-*`channel`*`，其中 *`channel`* 是此中继日志中记录的复制通道的名称。

    服务器将文件写入数据目录，除非使用具有前导绝对路径名的基本名称来指定不同目录。服务器通过向基本名称添加数字后缀按顺序创建中继日志文件。

    在复制服务器上，中继日志和中继日志索引不能与由`--log-bin`和`--log-bin-index`选项指定的二进制日志和二进制日志索引具有相同的名称。如果二��制日志和中继日志文件基本名称相同，服务器会发出错误消息并且不会启动。

    由于 MySQL 解析服务器选项的方式，如果您在服务器启动时指定了这个变量，您必须提供一个值；*只有在实际未指定该选项时才使用默认基本名称*。如果在服务器启动时指定`relay_log`系统变量而没有指定值，可能会导致意外行为；这取决于使用的其他选项、指定它们的顺序以及它们是在命令行中指定还是在选项文件中指定。有关 MySQL 如何处理服务器选项的更多信息，请参见第 6.2.2 节，“指定程序选项”。

    如果您指定了这个变量，那么指定的值也将用作中继日志索引文件的基本名称。您可以通过使用`relay_log_index`系统变量指定不同的中继日志索引文件基本名称来覆盖此行为。

    当服务器从索引文件中读取条目时，它会检查条目是否包含相对路径。如果包含，路径的相对部分将被使用`relay_log`系统变量设置的绝对路径替换。绝对路径保持不变；在这种情况下，必须手动编辑索引以启用新路径或路径的使用。

    您可能会发现`relay_log`系统变量在执行以下任务时很有用：

    +   创建与主机名无关的中继日志的名称。

    +   如果您需要将中继日志放在数据目录之外的某个区域，因为您的中继日志往往非常大，并且您不想减少`max_relay_log_size`。

    +   通过在磁盘之间使用负载平衡来提高速度。

    您可以从`relay_log_basename`系统变量中获取中继日志文件名（和路径）。

+   `relay_log_basename`

    | 系统变量 | `relay_log_basename` |
    | --- | --- |
    | 作用域 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 文件名 |
    | 默认值 | `datadir + '/' + hostname + '-relay-bin'` |

    保存中继日志文件的基本名称和完整路径。最大变量长度为 256。此变量由服务器设置，只读。

+   `relay_log_index`

    | 命令行格式 | `--relay-log-index=file_name` |
    | --- | --- |
    | 系统变量 | `relay_log_index` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 文件名 |
    | 默认值 | `*host_name*-relay-bin.index` |

    中继日志索引文件的名称。最大变量长度为 256。如果未指定此变量，但指定了`relay_log`系统变量，则其值将用作中继日志索引文件的默认基本名称。如果`relay_log`也未指定，则对于默认复制通道，默认名称为`*`host_name`*-relay-bin.index`，使用主机机器的名称。对于非默认复制通道，默认名称为`*`host_name`*-relay-bin-*`channel`*.index`，其中*`channel`*是在此中继日志索引中记录的复制通道的名称。

    中继日志文件的默认位置是数据目录，或者使用`relay_log`系统变量指定的任何其他位置。您可以使用`relay_log_index`系统变量指定替代位置，通过在基本名称前添加一个绝对路径名来指定不同的目录。

    复制服务器上的中继日志和中继日志索引不能与二进制日志和二进制日志索引具有相同的名称，其名称由`--log-bin`和`--log-bin-index`选项指定。如果二进制日志和中继日志文件的基本名称相同，服务器会发出错误消息并且不会启动。

    由于 MySQL 解析服务器选项的方式，如果在服务器启动时指定此变量，必须提供一个值；*只有在实际未指定该选项时才使用默认基本名称*。 如果在服务器启动时指定 `relay_log_index` 系统变量而没有指定值，可能会导致意外行为；此行为取决于使用的其他选项、指定它们的顺序以及它们是在命令行中指定还是在选项文件中指定。 有关 MySQL 如何处理服务器选项的更多信息，请参见 第 6.2.2 节，“指定程序选项”。

+   `relay_log_info_file`

    | 命令行格式 | `--relay-log-info-file=file_name` |
    | --- | --- |
    | 弃用 | 8.0.18 |
    | 系统变量 | `relay_log_info_file` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 文件名 |
    | 默认值 | `relay-log.info` |

    此系统变量的使用现已弃用。 如果设置了 `relay_log_info_repository=FILE`，则用于设置复制品的应用程序元数据存储库的文件名。 `relay_log_info_file` 和 `relay_log_info_repository` 系统变量的使用已被弃用，因为使用文件作为应用程序元数据存储库已被崩溃安全表取代。 有关应用程序元数据存储库的信息，请参见 第 19.2.4.2 节，“复制元数据存储库”。

+   `relay_log_info_repository`

    | 命令行格式 | `--relay-log-info-repository=value` |
    | --- | --- |
    | 弃用 | 8.0.23 |
    | 系统变量 | `relay_log_info_repository` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 字符串 |
    | 默认值 | `TABLE` |
    | 有效值 | `FILE``TABLE` |

    此系统变量的使用现已不推荐。`TABLE` 是默认设置，当配置多个复制通道时，必须使用 `TABLE` 设置。副本的应用程序元数据存储库也需要 `TABLE` 设置，以使复制对意外停止具有弹性。有关更多信息，请参见 Section 19.4.2, “Handling an Unexpected Halt of a Replica”。曾经不推荐使用替代设置 `FILE`。

    默认设置下，副本将其应用程序元数据存储库存储为 `InnoDB` 表，存储在名为 `mysql.slave_relay_log_info` 的 `mysql` 系统数据库中。有关应用程序元数据存储库的更多信息，请参见 Section 19.2.4, “Relay Log and Replication Metadata Repositories”。

    默认情况下，`FILE` 设置将副本的应用程序元数据存储库写入文件，默认情况下命名为 `relay-log.info`。可以使用 `relay_log_info_file` 系统变量更改名称。

+   `relay_log_purge`

    | 命令行格式 | `--relay-log-purge[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `relay_log_purge` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `ON` |

    禁用或启用中继日志文件在不再需要时立即清除。默认值为 1（`ON`）。

+   `relay_log_recovery`

    | 命令行格式 | `--relay-log-recovery[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `relay_log_recovery` |
    | 作用范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    如果启用此变量，它将在服务器启动后立即启用自动中继日志恢复。恢复过程将创建一个新的中继日志文件，将 SQL（应用程序）线程位置初始化为此新中继日志，并将 I/O（接收器）线程初始化为应用程序线程位置。然后继续从源读取中继日志。如果为使用 `CHANGE REPLICATION SOURCE TO` 选项为复制通道设置了 `SOURCE_AUTO_POSITION=1`，则用于启动复制的源位置可能是在连接中接收到的位置，而不是在此过程中分配的位置。

    此全局变量在运行时是只读的。可以在复制服务器启动时使用`--relay-log-recovery`选项设置其值，该选项应在复制品意外停止后使用，以确保不处理可能损坏的中继日志，并且必须使用以确保崩溃安全的复制品。默认值为 0（禁用）。有关在复制品上设置的组合设置，以使其对意外停止最具弹性，请参见第 19.4.2 节，“处理复制品意外停止”。

    对于多线程复制品（其中`replica_parallel_workers`或`slave_parallel_workers`大于 0），在启动时设置`--relay-log-recovery`会自动处理已从中继日志执行的事务序列中的任何不一致和间隙。当使用基于文件位置的复制时，这些间隙可能会发生。（有关更多详细信息，请参见第 19.5.1.34 节，“复制和事务不一致性”。）中继日志恢复过程使用与`START REPLICA UNTIL SQL_AFTER_MTS_GAPS`语句相同的方法处理间隙。当复制品达到一致的无间隙状态时，中继日志恢复过程继续从源处获取进一步的事务，从 SQL（应用程序）线程位置开始。当使用基于 GTID 的复制时，从 MySQL 8.0.18 开始，多线程复制品首先检查`MASTER_AUTO_POSITION`是否设置为`ON`，如果是，则省略计算应跳过或不跳过的事务的步骤，因此不需要旧的中继日志进行恢复过程。

    注意

    此变量不会影响以下 Group Replication 通道：

    +   `group_replication_applier`

    +   `group_replication_recovery`

    任何其他在组上运行的通道都会受到影响，例如从外部来源或另一个组复制的通道。

+   `relay_log_space_limit`

    | 命令行格式 | `--relay-log-space-limit=#` |
    | --- | --- |
    | 系统变量 | `relay_log_space_limit` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `0` |
    | 最小值 | `0` |
    | 最大值 | `18446744073709551615` |
    | 单位 | 字节 |

    用于所有中继日志的最大空间量。

+   `replica_checkpoint_group`

    | 命令行格式 | `--replica-checkpoint-group=#` |
    | --- | --- |
    | 引入版本 | 8.0.26 |
    | 系统变量 | `replica_checkpoint_group` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `512` |
    | 最小值 | `32` |
    | 最大值 | `524280` |
    | 块大小 | `8` |

    从 MySQL 8.0.26 开始，使用`replica_checkpoint_group`代替`slave_checkpoint_group`，后者从该版本开始已被弃用。在 MySQL 8.0.26 之前的版本中，请使用`slave_checkpoint_group`。

    `replica_checkpoint_group` 设置多线程副本在调用检查点操作之前可以处理的最大事务数，如`SHOW REPLICA STATUS`所示。设置此变量对未启用多线程的副本没有影响。设置此变量没有立即效果。该变量的状态适用于所有后续的`START REPLICA`语句。

    以前，NDB Cluster 不支持多线程副本，对于此变量的设置会被静默忽略。这个限制在 MySQL 8.0.33 中被取消。

    此变量与`replica_checkpoint_period`系统变量结合使用，当超过任一限制时，将执行检查点并重置跟踪事务数量和自上次检查点以来经过的时间的计数器。

    此变量的最小允许值为 32，除非服务器是使用`-DWITH_DEBUG`构建的，在这种情况下，最小值为 1。有效值始终是 8 的倍数；您可以将其设置为不是这种倍数的值，但服务器会在存储值之前将其向下舍入到下一个较低的 8 的倍数。(*例外*: 调试服务器不执行此舍入。)无论服务器是如何构建的，默认值都是 512，最大允许值为 524280。

+   `replica_checkpoint_period`

    | 命令行格式 | `--replica-checkpoint-period=#` |
    | --- | --- |
    | 引入版本 | 8.0.26 |
    | 系统变量 | `replica_checkpoint_period` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `300` |
    | 最小值 | `1` |
    | 最大值 | `4294967295` |
    | 单位 | 毫秒 |

    在 MySQL 8.0.26 及更高版本中，使用`replica_checkpoint_period`代替`slave_checkpoint_period`，后者从该版本开始被弃用；在 MySQL 8.0.26 之前，请使用`slave_checkpoint_period`。

    `replica_checkpoint_period`设置了允许经过的最长时间（以毫秒为单位），在调用检查点操作以更新多线程副本状态之前。设置此变量对于未启用多线程的副本没有影响。设置此变量立即对所有复制通道生效，包括正在运行的通道。

    以前，NDB Cluster 不支持多线程副本，对于这个变量的设置被静默忽略。这个限制在 MySQL 8.0.33 中被取消。

    此变量与`replica_checkpoint_group`系统变量结合使用，当超过任一限制时，将执行检查点并重置跟踪事务数量和自上次检查点以来经过的时间的计数器。

    除非服务器是使用`-DWITH_DEBUG`构建的，否则此变量的最小允许值为 1，此时最小值为 0。无论服务器是如何构建的，默认值为 300 毫秒，最大可能值为 4294967295 毫秒（约 49.7 天）。

+   `replica_compressed_protocol`

    | 命令行格式 | `--replica-compressed-protocol[={OFF&#124;ON}]` |
    | --- | --- |
    | 引入版本 | 8.0.26 |
    | 系统变量 | `replica_compressed_protocol` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    从 MySQL 8.0.26 开始，请使用`replica_compressed_protocol`代替已弃用的`slave_compressed_protocol`。在 MySQL 8.0.26 之前的版本中，请使用`slave_compressed_protocol`。

    `replica_compressed_protocol`指定是否在源和副本都支持时使用源/副本连接协议的压缩。如果禁用此变量（默认值），则连接是未压缩的。对此变量的更改将在后续的连接尝试中生效；这包括发出`START REPLICA`语句后，以及由正在运行的复制 I/O（接收器）线程进行的重新连接。

    二进制日志事务压缩（自 MySQL 8.0.20 起可用），由`binlog_transaction_compression`系统变量激活，也可用于节省带宽。如果将二进制日志事务压缩与协议压缩结合使用，则协议压缩的作用数据减少，但仍可压缩标头以及未压缩的事件和事务有效载荷。有关二进制日志事务压缩的更多信息，请参见第 7.4.4.5 节，“二进制日志事务压缩”。

    如果启用了`replica_compressed_protocol`，则优先于为`CHANGE REPLICATION SOURCE TO`语句指定的任何`SOURCE_COMPRESSION_ALGORITHMS`选项。在这种情况下，如果源和副本都支持该算法，则源到副本的连接使用`zlib`压缩。如果禁用了`replica_compressed_protocol`，则应用`SOURCE_COMPRESSION_ALGORITHMS`的值。有关更多信息，请参见第 6.2.8 节，“连接压缩控制”。

+   `replica_exec_mode`

    | 命令行格式 | `--replica-exec-mode=mode` |
    | --- | --- |
    | 引入版本 | 8.0.26 |
    | 系统变量 | `replica_exec_mode` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 枚举 |
    | 默认值 | `幂等`（NDB）`严格`（其他） |
    | 有效值 | `严格``幂等` |

    从 MySQL 8.0.26 开始，使用 `replica_exec_mode` 替代 `slave_exec_mode`，后者从该版本开始已被弃用。在 MySQL 8.0.26 之前的版本中，请使用 `slave_exec_mode`。

    `replica_exec_mode` 控制复制线程在复制过程中如何解决冲突和错误。`IDEMPOTENT` 模式会导致重复键和未找到键的错误被抑制；`STRICT` 表示不会发生此类抑制。

    `IDEMPOTENT` 模式旨在用于多源复制、循环复制以及一些其他特殊的 NDB 集群复制场景。（详见 第 25.7.10 节，“NDB 集群复制：双向和循环复制”，以及 第 25.7.12 节，“NDB 集群复制冲突解决”，获取更多信息。）NDB 集群会忽略为 `replica_exec_mode` 明确设置的任何值，并始终将其视为 `IDEMPOTENT`。

    在 MySQL Server 8.0 中，`STRICT` 模式是默认值。

    设置此变量会立即对所有复制通道生效，包括正在运行的通道。

    对于除了 `NDB` 外的存储引擎，*只有在您绝对确定重复键错误和未找到键错误可以安全忽略时才应使用 `IDEMPOTENT` 模式*。它旨在用于 NDB 集群的故障转移场景，其中使用了多源复制或循环复制，并不建议在其他情况下使用。

+   `replica_load_tmpdir`

    | 命令行格式 | `--replica-load-tmpdir=dir_name` |
    | --- | --- |
    | 引入版本 | 8.0.26 |
    | 系统变量 | `replica_load_tmpdir` |
    | 作用域 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 目录名称 |
    | 默认值 | `--tmpdir` 的值 |

    从 MySQL 8.0.26 开始，使用 `replica_load_tmpdir` 替代 `slave_load_tmpdir`，后者从该版本开始已被弃用。在 MySQL 8.0.26 之前的版本中，请使用 `slave_load_tmpdir`。

    `replica_load_tmpdir` 指定了副本创建临时文件的目录名称。设置此变量会立即对所有复制通道生效，包括正在运行的通道。该变量的值默认等于`tmpdir`系统变量的值，或者在未指定该系统变量时适用的默认值。

    当复制 SQL 线程复制`LOAD DATA`语句时，它会将要加载的文件从中继日志提取到临时文件中，然后将这些文件加载到表中。如果源上加载的文件很大，副本上的临时文件也很大。因此，建议使用此选项告诉副本将临时文件放在某个具有大量可用空间的文件系统中的目录中。在这种情况下，中继日志也很大，因此您可能还想设置`relay_log`系统变量将中继日志放在该文件系统中。

    此选项指定的目录应位于基于磁盘的文件系统中（而非基于内存的文件系统），以便用于复制`LOAD DATA`语句的临时文件可以在机器重新启动时保留。该目录也不应该是在系统启动过程中被操作系统清除的目录。然而，如果临时文件已被删除，复制现在可以在重新启动后继续。

+   `replica_max_allowed_packet`

    | 命令行格式 | `--replica-max-allowed-packet=#` |
    | --- | --- |
    | 引入版本 | 8.0.26 |
    | 系统变量 | `replica_max_allowed_packet` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `1073741824` |
    | 最小值 | `1024` |
    | 最大值 | `1073741824` |
    | 单位 | 字节 |
    | 块大小 | `1024` |

    从 MySQL 8.0.26 开始，请使用`replica_max_allowed_packet`代替从该版本开始已弃用的`slave_max_allowed_packet`。在 MySQL 8.0.26 之前的版本中，请使用`slave_max_allowed_packet`。

    `replica_max_allowed_packet`设置了复制 SQL（应用程序）和 I/O（接收器）线程可以处理的最大数据包大小（以字节为单位）。设置此变量立即对所有复制通道生效，包括正在运行的通道。一旦添加了事件头，源端可以写入比其`max_allowed_packet`设置更长的二进制日志事件。`replica_max_allowed_packet`的设置必须大于源端的`max_allowed_packet`设置，以确保使用基于行的复制进行大型更新时不会导致复制失败。

    此全局变量始终具有值，该值是 1024 的正整数倍；如果将其设置为非 1024 的值，则该值将向上舍入为下一个最高的 1024 的倍数以进行存储或使用；将`replica_max_allowed_packet`设置为 0 会导致使用 1024。（在所有这些情况下都会发出截断警告。）默认值和最大值为 1073741824（1 GB）；最小值�� 1024。

+   `replica_net_timeout`

    | 命令行格式 | `--replica-net-timeout=#` |
    | --- | --- |
    | 引入版本 | 8.0.26 |
    | 系统变量 | `replica_net_timeout` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `60` |
    | 最小值 | `1` |
    | 最大值 | `31536000` |
    | 单位 | 秒 |

    从 MySQL 8.0.26 开始，使用`replica_net_timeout`代替从该版本开始已弃用的`slave_net_timeout`。在 MySQL 8.0.26 之前的版本中，请使用`slave_net_timeout`。

    `replica_net_timeout`指定了复制品在从源端等待更多数据或心跳信号之前的秒数，如果复制品认为连接中断，则中止读取并尝试重新连接。设置此变量不会立即生效。变量的状态适用于所有后续的`START REPLICA`命令。

    默认值为 60 秒（一分钟）。超时后立即进行第一次重试。重试间隔由`CHANGE REPLICATION SOURCE TO`���句的`SOURCE_CONNECT_RETRY`选项控制，重新连接尝试次数由`SOURCE_RETRY_COUNT`选项限制。

    心跳间隔由 `CHANGE REPLICATION SOURCE TO` 语句的 `SOURCE_HEARTBEAT_PERIOD` 选项控制，该选项防止在连接仍然良好的情况下在没有数据的情况下发生连接超时。心跳间隔默认为 `replica_net_timeout` 值的一半，并记录在副本的连接元数据存储库中，并显示在 `replication_connection_configuration` 性能模式表中。请注意，对于 `replica_net_timeout` 的值或默认设置的更改不会自动更改心跳间隔，无论是显式设置还是使用先前计算的默认值。如果更改连接超时时间，您还必须发出 `CHANGE REPLICATION SOURCE TO` 来调整心跳间隔到一个适当的值，以便在连接超时之前发生。

+   `replica_parallel_type`

    | 命令行格式 | `--replica-parallel-type=value` |
    | --- | --- |
    | 引入版本 | 8.0.26 |
    | 已弃用 | 8.0.29 |
    | 系统变量 | `replica_parallel_type` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 枚举 |
    | 默认值（≥ 8.0.27） | `LOGICAL_CLOCK` |
    | 默认值（8.0.26） | `DATABASE` |
    | 有效值 | `DATABASE``LOGICAL_CLOCK` |

    从 MySQL 8.0.26 开始，使用 `replica_parallel_type` 替代从该版本开始已弃用的 `slave_parallel_type`。在 MySQL 8.0.26 之前的版本中，请使用 `slave_parallel_type`。

    对于多线程副本（`replica_parallel_workers` 或 `slave_parallel_workers` 设置为大于 0 的值的副本），`replica_parallel_type` 指定了在副本上决定哪些事务可以并行执行的策略。该变量对于未启用多线程的副本没有影响。可能的值包括：

    +   `LOGICAL_CLOCK`: 事务在副本上并行应用，基于复制源写入二进制日志的时间戳。根据时间戳跟踪事务之间的依赖关系，以在可能的情况下提供额外的并行化。

    +   `DATABASE`: 更新不同数据库的事务并行应用。只有在数据被分区到多个独立并且同时在源上更新的数据库时才适用此值。不能有跨数据库约束，因为这样的约束可能在副本上被违反。

    当启用`replica_preserve_commit_order`或`slave_preserve_commit_order`时，必须使用`LOGICAL_CLOCK`。在 MySQL 8.0.27 之前，`DATABASE`是默认值。从 MySQL 8.0.27 开始，默认情况下为副本服务器启用多线程（`replica_parallel_workers=4`为默认值），并且`LOGICAL_CLOCK`是默认值。（在 MySQL 8.0.27 及更高版本中，默认情况下还启用了`replica_preserve_commit_order`。）

    当复制拓扑结构使用多级副本时，`LOGICAL_CLOCK`可能会使每个副本距源的级别的并行化减少。为了补偿这种影响，您应该在源上以及每个中间副本上将`binlog_transaction_dependency_tracking`设置为`WRITESET`或`WRITESET_SESSION`，以指定在可能的情况下使用写入集而不是时间戳进行并行化。

    当使用`binlog_transaction_compression`系统变量启用二进制日志事务压缩时，如果将`replica_parallel_type`设置为`DATABASE`，则在调度事务之前会映射受事务影响的所有数据库。使用`DATABASE`策略的二进制日志事务压缩可能会减少与未压缩事务相比的并行性，后者会为每个事件映射并调度。

    `replica_parallel_type`从 MySQL 8.0.29 开始已弃用，使用数据库分区的事务并行化也已弃用。预计在未来的版本中将删除对这些的支持，并且之后将专门使用`LOGICAL_CLOCK`。

+   `replica_parallel_workers`

    | 命令行格式 | `--replica-parallel-workers=#` |
    | --- | --- |
    | 引入版本 | 8.0.26 |
    | 系统变量 | `replica_parallel_workers` |
    | 作用域 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值（≥ 8.0.27） | `4` |
    | 默认值（8.0.26） | `0` |
    | 最小值 | `0` |
    | 最大值 | `1024` |

    从 MySQL 8.0.26 开始，`slave_parallel_workers` 已不推荐使用，应改为使用 `replica_parallel_workers`。 （在 MySQL 8.0.26 之前，必须使用 `slave_parallel_workers` 来设置应用程序线程的数量。）

    `replica_parallel_workers` 启用副本的多线程，并设置用于并行执行复制事务的应用程序线程数。当值大于或等于 1 时，副本使用指定数量的工作线程来执行事务，还有一个协调器线程从中继日志中读取事务并将其调度给工作线程。当值为 0 时，只有一个线程按顺序读取和应用事务。如果使用多个复制通道，则此变量的值适用于每个通道使用的线程。

    在 MySQL 8.0.27 之前，此系统变量的默认值为 0，因此副本默认使用单个工作线程。从 MySQL 8.0.27 开始，默认值为 4，这意味着副本默认为多线程。

    截至 MySQL 8.0.30，将此变量设置为 0 已不推荐使用，会引发警告，并可能在将来的 MySQL 版本中删除。对于单个工作线程，请将 `replica_parallel_workers` 设置为 1。

    当 `replica_preserve_commit_order`（或 `slave_preserve_commit_order`）设置为 `ON`（MySQL 8.0.27 及更高版本的默认设置），副本上的事务按照在副本的中继日志中出现的顺序在副本上外部化。事务在应用程序线程之间的分配方式由 `replica_parallel_type`（MySQL 8.0.26 及更高版本）或 `slave_parallel_type`（MySQL 8.0.26 之前）确定。从 MySQL 8.0.27 开始，这些系统变量也具有适当的多线程默认值。

    要禁用并行执行，请将`replica_parallel_workers`设置为 1，这样副本将使用一个协调线程读取事务，一个工作线程应用事务，这意味着事务按顺序应用。当`replica_parallel_workers`等于 1 时，`replica_parallel_type`（`slave_parallel_type`）和`replica_preserve_commit_order`（`slave_preserve_commit_order`）系统变量不起作用，会被忽略。如果`replica_parallel_workers`等于 0，同时启用`CHANGE REPLICATION SOURCE TO`选项`GTID_ONLY`，副本将有一个协调线程和一个工作线程，就像`replica_parallel_workers`被设置为 1 一样。（`GTID_ONLY`在 MySQL 8.0.27 及更高版本中可用。）有一个并行工作者时，`replica_preserve_commit_order`（`slave_preserve_commit_order`）系统变量也不起作用。

    设置`replica_parallel_workers`没有立即效果，而是适用于所有后续的`START REPLICA`语句。

    多线程副本从 NDB Cluster 8.0.33 版本开始受支持。（之前，`NDB`会默默忽略`replica_parallel_workers`的任何设置。）更多信息请参见第 25.7.11 节，“使用多线程应用程序的 NDB Cluster 复制”。

    增加工作者数量可以提高并行性的潜力。通常情况下，这会提高性能，直到某个点，之后增加工作者数量会由于并发效应（如锁争用）而降低性能。理想数量取决于硬件和工作负载；很难预测，通常需要通过测试找到。没有主键的表总是会损害性能，在`replica_parallel_workers` > 1 的副本上可能会产生更大的负面性能影响；因此，在启用此选项之前，请确保所有表都有主键。

+   `replica_pending_jobs_size_max`

    | 命令行格式 | `--replica-pending-jobs-size-max=#` |
    | --- | --- |
    | 引入版本 | 8.0.26 |
    | 系统变量 | `replica_pending_jobs_size_max`` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `128M` |
    | 最小值 | `1024` |
    | 最大值 | `16EiB` |
    | 单位 | 字节 |
    | 块大小 | `1024` |

    从 MySQL 8.0.26 开始，使用`replica_pending_jobs_size_max`代替从该版本开始已弃用的`slave_pending_jobs_size_max`。在 MySQL 8.0.26 之前的版本中，使用`slave_pending_jobs_size_max`。

    对于多线程复制品，此变量设置了可用于尚未应用的事件的 applier 队列的最大内存量（以字节为单位）。设置此变量对未启用多线程的复制品没有影响。设置此变量没有立即效果。变量的状态适用于所有后续的`START REPLICA`语句。

    此变量的最小可能值为 1024 字节；默认值为 128MB。最大可能值为 18446744073709551615（16 exbibytes）。不是 1024 字节的精确倍数的值在存储之前会被舍入为下一个较低的 1024 字节的倍数。

    此变量的值是一个软限制，可以设置为匹配正常工作负载。如果异常大的事件超过此大小，事务将被暂停，直到所有工作线程的队列为空，然后再处理。所有后续事务将被暂停，直到大事务完成。

+   `replica_preserve_commit_order`

    | 命令行格式 | `--replica-preserve-commit-order[={OFF&#124;ON}]` |
    | --- | --- |
    | 引入 | 8.0.26 |
    | 系统变量 | `replica_preserve_commit_order` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 (≥ 8.0.27) | `ON` |
    | 默认值 (8.0.26) | `OFF` |

    从 MySQL 8.0.26 开始，使用`replica_preserve_commit_order`代替从该版本开始已弃用的`slave_preserve_commit_order`。在 MySQL 8.0.26 之前的版本中，使用`slave_preserve_commit_order`。

    对于多线程复制（`replica_parallel_workers` 设置为大于 0 的值的复制品），设置 `replica_preserve_commit_order=ON` 确保事务在复制品上按照在复制品的中继日志中出现的顺序执行和提交。这可以防止在已从复制品的中继日志执行的事务序列中出现间隙，并保留与源相同的事务历史（以下列出了限制）。此变量对未启用多线程的复制品没有影响。

    在 MySQL 8.0.27 之前，此系统变量的默认值为 `OFF`，意味着事务可能会无序提交。从 MySQL 8.0.27 开始，默认情况下为复制品服务器启用多线程（默认为 `replica_parallel_workers=4`），因此 `replica_preserve_commit_order=ON` 是默认值，并且设置 `replica_parallel_type=LOGICAL_CLOCK` 也是默认值。同样从 MySQL 8.0.27 开始，如果 `replica_parallel_workers` 设置为 1，则 `replica_preserve_commit_order` 的设置将被忽略，因为在这种情况下事务的顺序已经被保留。

    二进制日志和复制品更新日志不需要在复制品上设置 `replica_preserve_commit_order=ON`，如果需要，可以禁用。设置 `replica_preserve_commit_order=ON` 要求将 `replica_parallel_type` 设置为 `LOGICAL_CLOCK`，这在 MySQL 8.0.27 之前 *不* 是默认设置。在更改 `replica_preserve_commit_order` 和 `replica_parallel_type` 的值之前，必须停止复制 SQL 线程（如果使用多个复制通道，则对所有复制通道都要停止）。

    当`replica_preserve_commit_order=OFF`被设置时，多线程复制并行应用的事务可能会无序提交。因此，检查最近执行的事务并不能保证源数据库上的所有先前事务已在复制中执行。在复制中继日志中执行的事务序列可能存在间隙的可能性。这对于使用多线程复制时的日志记录和恢复有影响。有关更多信息，请参见第 19.5.1.34 节，“复制和事务不一致性”。

    当`replica_preserve_commit_order=ON`被设置时，执行的工作线程会等待所有先前的事务提交后再进行提交。当一个线程在等待其他工作线程提交它们的事务时，它会报告其状态为`等待前一个事务提交`。在这种模式下，多线程复制永远不会进入源数据库不在的状态。这支持将复制用于读取扩展。参见第 19.4.5 节，“使用复制进行扩展”。

    注意

    +   `replica_preserve_commit_order=ON`不能防止源数据库二进制日志位置滞后，其中`Exec_master_log_pos`落后于已执行事务的位置。请参见第 19.5.1.34 节，“复制和事务不一致性”。

    +   `replica_preserve_commit_order=ON`不保留如果复制使用二进制日志过滤器，如`--binlog-do-db`，则不保留提交顺序和事务历史。

    +   `replica_preserve_commit_order=ON`不保留非事务性 DML 更新的顺序。这些更新可能在在中继日志中先于它们之前的事务提交，这可能导致在复制中继日志中执行的事务序列中存在间隙。

    +   如果使用基于语句的复制，并且事务性和非事务性存储引擎参与在源上回滚的非 XA 事务，可能会出现在副本上保留提交顺序的限制。通常，在源上回滚的非 XA 事务不会被复制到副本，但在这种特殊情况下，该事务可能会被复制到副本。如果发生这种情况，没有二进制日志记录的多线程副本无法处理事务回滚，因此在这种情况下，副本上的提交顺序会与中继日志中事务的顺序不同。

+   `replica_sql_verify_checksum`

    | 命令行格式 | `--replica-sql-verify-checksum[={OFF&#124;ON}]` |
    | --- | --- |
    | 引入版本 | 8.0.26 |
    | 系统变量 | `replica_sql_verify_checksum` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `ON` |

    从 MySQL 8.0.26 开始，请使用`replica_sql_verify_checksum`替代`slave_sql_verify_checksum`，后者从该版本开始已被弃用。在 MySQL 8.0.26 之前的版本中，请使用`slave_sql_verify_checksum`。

    `slave_sql_verify_checksum`导致复制 SQL（应用程序）线程使用从中继日志读取的校验和来验证数据。如果出现不匹配，副本将停止并显示错误。设置此变量立即对所有复制通道生效，包括正在运行的通道。

    注意

    复制 I/O（接收器）线程在从网络接收事件时，始终尽可能读取校验和。

+   `replica_transaction_retries`

    | 命令行格式 | `--replica-transaction-retries=#` |
    | --- | --- |
    | 引入版本 | 8.0.26 |
    | 系统变量 | `replica_transaction_retries` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `10` |
    | 最小值 | `0` |
    | 最大值 | `18446744073709551615` |

    从 MySQL 8.0.26 开始，使用`replica_transaction_retries`代替从该版本开始已弃用的`slave_transaction_retries`。在 MySQL 8.0.26 之前的版本中，请使用`slave_transaction_retries`。

    `replica_transaction_retries` 设置了单线程或多线程副本上的复制 SQL 线程在停止之前自动重试失败事务的最大次数。设置此变量立即对所有复制通道生效，包括正在运行的通道。默认值为 10。将变量设置为 0 会禁用事务的自动重试。

    如果复制 SQL 线程由于`InnoDB`死锁或事务执行时间超过`InnoDB`的`innodb_lock_wait_timeout`或`NDB`的`TransactionDeadlockDetectionTimeout`或`TransactionInactiveTimeout`而无法执行事务时，它会在停止之前自动重试`replica_transaction_retries`次数。非临时错误的事务不会重试。

    Performance Schema 表`replication_applier_status`显示了每个复制通道上发生的重试次数，在`COUNT_TRANSACTIONS_RETRIES`列中。Performance Schema 表`replication_applier_status_by_worker`显示了单线程或多线程副本上每个应用程序线程对事务重试的详细信息，并标识导致最后一个事务和当前正在进行的事务重新尝试的错误。

+   `replica_type_conversions`

    | 命令行格式 | `--replica-type-conversions=set` |
    | --- | --- |
    | 引入版本 | 8.0.26 |
    | 系统变量 | `replica_type_conversions` |
    | 作用域 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 设置 |
    | 默认值 |  |
    | 有效值 | `ALL_LOSSY``ALL_NON_LOSSY``ALL_SIGNED``ALL_UNSIGNED` |

    从 MySQL 8.0.26 开始，使用 `replica_type_conversions` 替代 `slave_type_conversions`，后者从该版本开始已被弃用。在 MySQL 8.0.26 之前的版本中，请使用 `slave_type_conversions`。

    `replica_type_conversions` 控制在使用基于行的复制时副本上生效的类型转换模式。其值是一个由列表中零个或多个元素组成的逗号分隔集合：`ALL_LOSSY`、`ALL_NON_LOSSY`、`ALL_SIGNED`、`ALL_UNSIGNED`。将此变量设置为空字符串以禁止源和副本之间的类型转换。设置此变量立即对所有复制通道生效，包括正在运行的通道。

    有关基于行复制中适用于属性提升和降级的类型转换模式的更多信息，请参阅基于行复制：属性提升和降级。

+   `replication_optimize_for_static_plugin_config`

    | 命令行格式 | `--replication-optimize-for-static-plugin-config[={OFF&#124;ON}]` |
    | --- | --- |
    | 引入版本 | 8.0.23 |
    | 系统变量 | `replication_optimize_for_static_plugin_config` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    使用共享锁，并避免不必要的锁获取，以提高半同步复制的性能。这个设置和 `replication_sender_observe_commit_only` 在副本数量增加时有帮助，因为锁争用会降低性能。在启用此系统变量时，半同步复制插件无法卸载，因此必须在卸载完成之前禁用该系统变量。

    可以在安装半同步复制插件之前或之后启用此系统变量，并且可以在复制运行时启用。半同步复制源服务器也可以通过启用此系统变量获得性能优势，因为它们使用与副本相同的锁定机制。

    当服务器上使用组复制时，可以启用`replication_optimize_for_static_plugin_config`。在这种情况下，当由于高工作负载而导致锁争用时，可能会提高性能。

+   `replication_sender_observe_commit_only`

    | 命令行格式 | `--replication-sender-observe-commit-only[={OFF&#124;ON}]` |
    | --- | --- |
    | 引入版本 | 8.0.23 |
    | 系统变量 | `replication_sender_observe_commit_only` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    限制回调以提高半同步复制的性能。这个设置和`replication_optimize_for_static_plugin_config`在副本数量增加时有帮助，因为锁争用可能会降低性能。

    这个系统变量可以在安装半同步复制插件之前或之后启用，并且可以在复制运行时启用。半同步复制源服务器也可以通过启用这个系统变量获得性能优势，因为它们使用与副本相同的锁定机制。

+   `report_host`

    | 命令行格式 | `--report-host=host_name` |
    | --- | --- |
    | 系统变量 | `report_host` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 字符串 |

    在副本注册期间向源报告的副本的主机名或 IP 地址。此值将出现在源服务器的`SHOW REPLICAS`输出中。如果不希望副本向源注册自己，请将值保持未设置。

    注意

    仅仅从副本连接后的 TCP/IP 套接字中读取副本服务器的 IP 地址是不够的。由于 NAT 和其他路由问题，该 IP 可能无法用于从源或其他主机连接到副本。

+   `report_password`

    | 命令行格式 | `--report-password=name` |
    | --- | --- |
    | 系统变量 | `report_password` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 字符串 |

    在复制品注册期间向源报告的复制品的帐户密码。如果源使用 `--show-replica-auth-info` 或 `--show-slave-auth-info` 启动，则此值将出现在源服务器上 `SHOW REPLICAS` 的输出中。

    尽管此变量的名称可能暗示相反，`report_password` 与 MySQL 用户权限系统无关，因此不一��（甚至可能不是）与 MySQL 复制用户帐户的密码相同。

+   `report_port`

    | 命令行格式 | `--report-port=port_num` |
    | --- | --- |
    | 系统变量 | `report_port` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `[slave_port]` |
    | 最小值 | `0` |
    | 最大值 | `65535` |

    用于连接到复制品的 TCP/IP 端口号，在复制品注册期间向源报告。仅在复制品侦听非默认端口或源或其他客户端到复制品有特殊隧道时设置此选项。如果不确定，请不要使用此选项。

    此选项的默认值是实际使用的复制品端口号。这也是 `SHOW REPLICAS` 显示的默认值。

+   `report_user`

    | 命令行格式 | `--report-user=name` |
    | --- | --- |
    | 系统变量 | `report_user` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 字符串 |

    在复制品注册期间向源报告的复制品的帐户用户名。如果源使用 `--show-replica-auth-info` 或 `--show-slave-auth-info` 启动，则此值将出现在源服务器上 `SHOW REPLICAS` 的输出中。

    尽管此变量的名称可能暗示相反，`report_user` 与 MySQL 用户权限系统无关，因此不一定（甚至可能不是）与 MySQL 复制用户帐户的名称相同。

+   `rpl_read_size`

    | 命令行格式 | `--rpl-read-size=#` |
    | --- | --- |
    | 系统变量 | `rpl_read_size` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `8192` |
    | 最小值 | `8192` |
    | 最大值 | `4294959104` |
    | 单位 | 字节 |
    | 块大小 | `8192` |

    `rpl_read_size` 系统变量控制从二进制日志文件和中继日志文件中读取的最小数据量（以字节为单位）。如果这些文件的大量磁盘 I/O 活动影响了数据库的性能，增加读取大小可能会减少文件读取和 I/O 停顿，尤其是当文件数据当前未被操作系统缓存时。

    `rpl_read_size` 的最小和默认值为 8192 字节。该值必须是 4KB 的倍数。请注意，为从二进制日志和中继日志文件读取的每个线程分配了此值大小的缓冲区，包括源上的转储线程和副本上的协调器线程。因此，设置一个较大的值可能会影响服务器的内存消耗。

+   `rpl_semi_sync_replica_enabled`

    | 命令行格式 | `--rpl-semi-sync-replica-enabled[={OFF&#124;ON}]` |
    | --- | --- |
    | 引入版本 | 8.0.26 |
    | 系统变量 | `rpl_semi_sync_replica_enabled` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    当在副本上安装了`rpl_semi_sync_replica`（`semisync_replica.so`库）插件以设置半同步复制时，`rpl_semi_sync_replica_enabled` 可用。如果安装了`rpl_semi_sync_slave`插件（`semisync_slave.so`库），则可以使用`rpl_semi_sync_slave_enabled`。

    `rpl_semi_sync_replica_enabled` 控制着复制服务器上是否启用半同步复制。要启用或禁用插件，请将此变量分别设置为`ON`或`OFF`（或 1 或 0）。默认值为`OFF`。

    仅当安装了副本端半同步复制插件时才可用此变量。

+   `rpl_semi_sync_replica_trace_level`

    | 命令���格式 | `--rpl-semi-sync-replica-trace-level=#` |
    | --- | --- |
    | 引入版本 | 8.0.26 |
    | 系统变量 | `rpl_semi_sync_replica_trace_level` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `32` |
    | 最小值 | `0` |
    | 最大值 | `4294967295` |

    当安装了`rpl_semi_sync_replica`（`semisync_replica.so`库）插件以设置半同步复制时，`rpl_semi_sync_replica_trace_level` 可用。如果安装了`rpl_semi_sync_slave`插件（`semisync_slave.so`库），则可用`rpl_semi_sync_slave_trace_level`。

    `rpl_semi_sync_replica_trace_level` 控制副本服务器上半同步复制的调试跟踪级别。有关允许的值，请参阅`rpl_semi_sync_master_trace_level`。

    仅当安装了副本端半同步复制插件时才可用此变量。

+   `rpl_semi_sync_slave_enabled`

    | 命令行格式 | `--rpl-semi-sync-slave-enabled[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `rpl_semi_sync_slave_enabled` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔 |
    | 默认值 | `OFF` |

    当安装了`rpl_semi_sync_slave`（`semisync_slave.so`库）插件以设置半同步复制时，`rpl_semi_sync_slave_enabled` 可用。如果安装了`rpl_semi_sync_replica`插件（`semisync_replica.so`库），则可用`rpl_semi_sync_replica_enabled`。

    `rpl_semi_sync_slave_enabled` 控制副本服务器上是否启用半同步复制。要启用或禁用插件，请将此变量设置为`ON`或`OFF`（或分别为 1 或 0）。默认值为`OFF`。

    仅当安装了副本端半同步复制插件时才可用此变量。

+   `rpl_semi_sync_slave_trace_level`

    | 命令行格式 | `--rpl-semi-sync-slave-trace-level=#` |
    | --- | --- |
    | 系统变量 | `rpl_semi_sync_slave_trace_level` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `32` |
    | 最小值 | `0` |
    | 最大值 | `4294967295` |

    `rpl_semi_sync_slave_trace_level` 在安装了`rpl_semi_sync_slave`（`semisync_slave.so`库）插件的复制品上可用，用于设置半同步复制。如果安装了`rpl_semi_sync_replica`插件（`semisync_replica.so`库），则可用`rpl_semi_sync_replica_trace_level`。

    `rpl_semi_sync_slave_trace_level` 控制复制品服务器上的半同步复制调试跟踪级别。有关允许的值，请参阅`rpl_semi_sync_master_trace_level`。

    仅当安装了复制品端半同步复制插件时才可用。

+   `rpl_stop_replica_timeout`

    | 命令行格式 | `--rpl-stop-replica-timeout=#` |
    | --- | --- |
    | 引入版本 | 8.0.26 |
    | 系统变量 | `rpl_stop_replica_timeout` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `31536000` |
    | 最小值 | `2` |
    | 最大值 | `31536000` |
    | 单位 | 秒 |

    从 MySQL 8.0.26 开始，使用`rpl_stop_replica_timeout`代替`rpl_stop_slave_timeout`，后者从该版本开始已被弃用。在 MySQL 8.0.26 之前的版本中，请使用`rpl_stop_slave_timeout`。

    通过设置此变量，您可以控制`STOP REPLICA`在超时之前等待的时间长度（以秒为单位）。这可用于避免`STOP REPLICA`与使用不同客户端连接到复制品的其他 SQL 语句之间的死锁。

    `rpl_stop_replica_timeout`的最大值和默认值为 31536000 秒（1 年）。最小值为 2 秒。对此变量的更改将对后续的`STOP REPLICA`语句生效。

    此变量仅影响发出 `STOP REPLICA` 命令的客户端。当超时时，发出命令的客户端将返回一条错误消息，指示命令执行不完整。客户端停止等待复制 I/O（接收器）和 SQL（应用程序）线程停止，但复制线程继续尝试停止，并且 `STOP REPLICA` 命令仍然有效。一旦复制线程不再忙碌，`STOP REPLICA` 命令将被执行，副本将停止。

+   `rpl_stop_slave_timeout`

    | 命令行格式 | `--rpl-stop-slave-timeout=#` |
    | --- | --- |
    | 已弃用 | 8.0.26 |
    | 系统变量 | `rpl_stop_slave_timeout` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `31536000` |
    | 最小值 | `2` |
    | 最大值 | `31536000` |
    | 单位 | 秒 |

    从 MySQL 8.0.26 开始，`rpl_stop_slave_timeout` 已被弃用，应改用别名 `rpl_stop_replica_timeout`。在 MySQL 8.0.26 之前的版本中，请使用 `rpl_stop_slave_timeout`。

    通过设置此变量，您可以控制 `STOP REPLICA` 在超时前等待的时间长度（以秒为单位）。这可用于避免在使用不同客户端连接到副本的其他 SQL 语句和 `STOP REPLICA` 之间发生死锁。

    `rpl_stop_slave_timeout` 的最大和默认值为 31536000 秒（1 年）。最小值为 2 秒。对此变量的更改将对后续的 `STOP REPLICA` 命令生效。

    此变量仅影响发出 `STOP REPLICA` 命令的客户端。当超时时，发出命令的客户端将返回一条错误消息，指示命令执行不完整。客户端停止等待复制 I/O（接收器）和 SQL（应用程序）线程停止，但复制线程继续尝试停止，并且 `STOP REPLICA` 指令仍然有效。一旦复制线程不再忙碌，`STOP REPLICA` 命令将被执行，副本将停止。

+   `skip_replica_start`

    | 命令行格式 | `--skip-replica-start[={OFF&#124;ON}]` |
    | --- | --- |
    | 引入 | 8.0.26 |
    | 系统变量 | `skip_replica_start` |
    | 作用范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    从 MySQL 8.0.26 开始，应使用 `skip_replica_start` 替代 `skip_slave_start`，因为从该版本开始已弃用。在 MySQL 8.0.26 之前的版本中，请使用 `skip_slave_start`。

    `skip_replica_start` 告诉复制服务器在启动服务器时不要启动复制 I/O（接收器）和 SQL（应用程序）线程。要稍后启动线程，请使用 `START REPLICA` 语句。

    此系统变量为只读，可以使用 `PERSIST_ONLY` 关键字或 `@@persist_only` 修饰符与 `SET` 语句一起设置。`--skip-replica-start` 命令行选项也设置此系统变量。您可以使用系统变量代替命令行选项，以允许通过 MySQL Server 的权限结构访问此功能，这样数据库管理员就不需要对操作系统有任何特权访问。

+   `skip_slave_start`

    | 命令行格式 | `--skip-slave-start[={OFF&#124;ON}]` |
    | --- | --- |
    | 已弃用 | 8.0.26 |
    | 系统变量 | `skip_slave_start` |
    | 作用范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    从 MySQL 8.0.26 开始，`skip_slave_start` 已被弃用，应改用别名 `skip_replica_start`。在 MySQL 8.0.26 之前的版本中，请使用 `skip_slave_start`。

    告诉复制服务器在启动服务器时不要启动复制 I/O（接收器）和 SQL（应用程序）线程。要稍后启动线程，请使用 `START REPLICA` 语句。

    此系统变量从 MySQL 8.0.24 版本开始可用。它是只读的，可以使用`PERSIST_ONLY`关键字或在`SET`语句中使用`@@persist_only`限定符来设置。`--skip-slave-start`命令行选项也会设置此系统变量。您可以使用系统变量代替命令行选项，以允许通过 MySQL 服务器的权限结构访问此功能，这样数据库管理员就不需要对操作系统具有任何特权访问。

+   `slave_checkpoint_group`

    | 命令行格式 | `--slave-checkpoint-group=#` |
    | --- | --- |
    | 弃用 | 8.0.26 |
    | 系统变量 | `slave_checkpoint_group` |
    | 作用域 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `512` |
    | 最小值 | `32` |
    | 最大值 | `524280` |
    | 块大小 | `8` |

    从 MySQL 8.0.26 开始，`slave_checkpoint_group`已被弃用，应改用别名`replica_checkpoint_group`。在 MySQL 8.0.26 之前的版本中，请使用`slave_checkpoint_group`。

    `slave_checkpoint_group`设置了多线程副本在调用检查点操作以更新其状态之前可以处理的最大事务数量，如`SHOW REPLICA STATUS`所示。设置此变量对未启用多线程的副本没有影响。设置此变量没有立即效果。变量的状态适用于所有后续的`START REPLICA`语句。

    以前，NDB Cluster 不支持多线程副本，对于此变量的设置被静默忽略。此限制在 MySQL 8.0.33 中解除。

    此变量与`slave_checkpoint_period`系统变量结合使用，当超过任一限制时，将执行检查点，并重置跟踪事务数量和自上次检查点以来经过的时间的计数器。

    该变量的最小允许值为 32，除非服务器是使用`-DWITH_DEBUG`构建的，此时最小值为 1。有效值始终是 8 的倍数；您可以将其设置为非 8 的倍数，但服务器在存储值之前会将其向下舍入到下一个较低的 8 的倍数。(*例外*: 调试服务器不执行此舍入操作。) 无论服务器是如何构建的，默认值为 512，最大允许值为 524280。

+   `slave_checkpoint_period`

    | 命令行格式 | `--slave-checkpoint-period=#` |
    | --- | --- |
    | 已弃用 | 8.0.26 |
    | 系统变量 | `slave_checkpoint_period` |
    | 作用域 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `300` |
    | 最小值 | `1` |
    | 最大值 | `4294967295` |
    | 单位 | 毫秒 |

    截至 MySQL 8.0.26，`slave_checkpoint_period`已弃用，应改用`replica_checkpoint_period`；在 MySQL 8.0.26 之前，请使用`slave_checkpoint_period`。

    `slave_checkpoint_period`设置了允许经过的最长时间（以毫秒为单位），在此时间内将调用检查点操作来更新多线程副本的状态，如`SHOW REPLICA STATUS`所示。设置此变量对未启用多线程的副本没有影响。设置此变量立即对所有复制通道生效，包括正在运行的通道。

    以前，NDB Cluster 不支持多线程副本，对于这个变量的设置被静默忽略。这个限制在 MySQL 8.0.33 中被取消。

    此变量与`slave_checkpoint_group`系统变量结合使用，当超过任一限制时，将执行检查点并重置跟踪事务数量和自上次检查点以来经过的时间的计数器。

    该变量的最小允许值为 1，除非服务器是使用`-DWITH_DEBUG`构建的，此时最小值为 0。无论服务器是如何构建的，默认值为 300 毫秒，最大可能值为 4294967295 毫秒（约 49.7 天）。

+   `slave_compressed_protocol`

    | 命令行格式 | `--slave-compressed-protocol[={OFF&#124;ON}]` |
    | --- | --- |
    | 已弃用 | 8.0.18 |
    | 系统变量 | `slave_compressed_protocol` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    `slave_compressed_protocol` 已被弃用，从 MySQL 8.0.26 开始，应改用别名 `replica_compressed_protocol`。在 MySQL 8.0.26 之前的版本中，请使用 `slave_compressed_protocol`。

    `slave_compressed_protocol` 控制是否在源/副本连接协议上使用压缩，如果源和副本都支持。如果禁用此变量（默认情况下），连接将不被压缩。对此变量的更改将在后续连接尝试中生效；这包括在发出 `START REPLICA` 语句后，以及由正在运行的复制 I/O（接收器）线程进行的重新连接。

    二进制日志事务压缩（自 MySQL 8.0.20 起可用），由 `binlog_transaction_compression` 系统变量激活，也可用于节省带宽。如果将二进制日志事务压缩与协议压缩结合使用，协议压缩的作用机会较少，但仍可压缩标头以及未压缩的事件和事务有效载荷。有关二进制日志事务压缩的更多信息，请参见 Section 7.4.4.5, “Binary Log Transaction Compression”。

    自 MySQL 8.0.18 起，如果启用了 `slave_compressed_protocol`，它将优先于为 `CHANGE REPLICATION SOURCE TO` | `CHANGE MASTER TO` 语句指定的任何 `SOURCE_COMPRESSION_ALGORITHMS` | `MASTER_COMPRESSION_ALGORITHMS` 选项。在这种情况下，如果源和副本都支持该算法，则连接到源使用 `zlib` 压缩。如果禁用了 `slave_compressed_protocol`，则 `SOURCE_COMPRESSION_ALGORITHMS` | `MASTER_COMPRESSION_ALGORITHMS` 的值适用。有关更多信息，请参见 Section 6.2.8, “Connection Compression Control”。

    从 MySQL 8.0.18 开始，此系统变量已被弃用。您应该期望它在将来的 MySQL 版本中被移除。请参阅配置传统连接压缩。

+   `slave_exec_mode`

    | 命令行格式 | `--slave-exec-mode=mode` |
    | --- | --- |
    | 系统变量 | `slave_exec_mode` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 枚举 |
    | 默认值 | `IDEMPOTENT`（NDB）`STRICT`（其他） |
    | 有效值 | `STRICT``IDEMPOTENT` |

    从 MySQL 8.0.26 开始，`slave_exec_mode` 已被弃用，应改用别名 `replica_exec_mode`。在 MySQL 8.0.26 之前的版本中，请使用 `slave_exec_mode`。

    `slave_exec_mode` 控制复制线程在复制过程中如何解决冲突和错误。`IDEMPOTENT` 模式导致重复键和找不到键错误的抑制；`STRICT` 表示不会发生这种抑制。

    `IDEMPOTENT` 模式适用于多源复制、循环复制和 NDB Cluster 复制的一些其他特殊复制场景。（有关更多信息，请参阅第 25.7.10 节，“NDB Cluster 复制：双向和循环复制”，以及第 25.7.12 节，“NDB Cluster 复制冲突解决”。）NDB Cluster 忽略为 `slave_exec_mode` 明确设置的任何值，并始终将其视为 `IDEMPOTENT`。

    在 MySQL Server 8.0 中，`STRICT` 模式是默认值。

    设置此变量立即对所有复制通道生效，包括正在运行的通道。

    对于除了`NDB`之外的存储引擎，*只有在您绝对确定可以安全忽略重复键错误和找不到键错误时，才应使用`IDEMPOTENT`模式*。它旨在用于 NDB Cluster 的故障转移场景，其中使用了多源复制或循环复制，并不建议在其他情况下使用。

+   `slave_load_tmpdir`

    | 命令行格式 | `--slave-load-tmpdir=dir_name` |
    | --- | --- |
    | 已弃用 | 8.0.26 |
    | 系统变量 | `slave_load_tmpdir` |
    | 范围 | ��局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 目录名称 |
    | 默认值 | `--tmpdir` 的值 |

    从 MySQL 8.0.26 开始，`slave_load_tmpdir` 已弃用，应改用别名`replica_load_tmpdir`。在 MySQL 8.0.26 之前的版本中，请使用`slave_load_tmpdir`。

    `slave_load_tmpdir` 指定副本创建临时文件的目录名称。设置此变量立即对所有复制通道生效，包括正在运行的通道。变量值默认等于`tmpdir`系统变量的值，或者在未指定该系统变量时适用的默认值。

    当复制 SQL 线程复制`LOAD DATA`语句时，它会将要加载的文件从中继日志中提取到临时文件中，然后将这些文件加载到表中。如果源上加载的文件很大，则副本上的临时文件也很大。因此，建议使用此选项告诉副本将临时文件放在某个具有大量可用空间的文件系统中的目录中。在这种情况下，中继日志也很大，因此您可能还想设置`relay_log`系统变量以将中继日志放在该文件系统中。

    此选项指定的目录应位于基于磁盘的文件系统中（而不是基于内存的文件系统），以便用于复制`LOAD DATA`语句的临时文件可以在机器重新启动时保留。该目录也不应该是在系统启动过程中被操作系统清除的目录。但是，如果临时文件已被删除，则复制现在可以在重新启动后继续。

+   `slave_max_allowed_packet`

    | 命令行格式 | `--slave-max-allowed-packet=#` |
    | --- | --- |
    | 已弃用 | 8.0.26 |
    | 系统变量 | `slave_max_allowed_packet` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `1073741824` |
    | 最小值 | `1024` |
    | 最大值 | `1073741824` |
    | 单位 | 字节 |
    | 块大小 | `1024` |

    从 MySQL 8.0.26 开始，`slave_max_allowed_packet` 已被弃用，应改用别名 `replica_max_allowed_packet`。在 MySQL 8.0.26 之前的版本中，请使用 `slave_max_allowed_packet`。

    `slave_max_allowed_packet` 设置复制 SQL（应用程序）和 I/O（接收器）线程可以处理的最大数据包大小（以字节为单位）。设置此变量立即对所有复制通道生效，包括正在运行的通道。源可能写入比其 `max_allowed_packet` 设置更长的二进制日志事件，一旦添加事件头。`slave_max_allowed_packet` 的设置必须大于源上的 `max_allowed_packet` 设置，以确保使用基于行的复制进行大型更新时不会导致复制失败。

    此全局变量始终具有值，该值是 1024 的正整数倍；如果将其设置为非 1024 的值，则将其舍入到下一个最高的 1024 的倍数以进行存储或使用；将 `slave_max_allowed_packet` 设置为 0 会���致使用 1024。（在所有这些情况下都会发出截断警告。）默认值和最大值为 1073741824（1 GB）；最小值为 1024。

+   `slave_net_timeout`

    | 命令行格式 | `--slave-net-timeout=#` |
    | --- | --- |
    | 已弃用 | 8.0.26 |
    | 系统变量 | `slave_net_timeout` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `60` |
    | 最小值 | `1` |
    | 最大值 | `31536000` |
    | 单位 | 秒 |

    从 MySQL 8.0.26 开始，`slave_net_timeout` 已被弃用，应改用别名 `replica_net_timeout`。在 MySQL 8.0.26 之前的版本中，请使用 `slave_net_timeout`。

    `slave_net_timeout` 指定等待来自源的更多数据或心跳信号的秒数，然后副本认为连接已中断，中止读取，并尝试重新连接。设置此变量没有立即效果。变量的状态适用于所有后续的 `START REPLICA` 命令。

    默认值为 60 秒（一分钟）。超时后立即进行第一次重试。重试间隔由 `CHANGE REPLICATION SOURCE TO` | `CHANGE MASTER TO` 语句的 `SOURCE_CONNECT_RETRY` | `MASTER_CONNECT_RETRY` 选项控制，重连尝试次数由 `SOURCE_RETRY_COUNT` | `MASTER_RETRY_COUNT` 选项限制。

    心跳间隔，用于防止在没有数据的情况下发生连接超时，由 `CHANGE REPLICATION SOURCE TO` | `CHANGE MASTER TO` 语句的 `SOURCE_HEARTBEAT_PERIOD` | `MASTER_HEARTBEAT_PERIOD` 选项控制。心跳间隔默认为 `slave_net_timeout` 值的一半，并记录在副本的连接元数据存储库中，并显示在 `replication_connection_configuration` 性能模式表中。请注意，对于 `slave_net_timeout` 的值或默认设置的更改不会自动更改心跳间隔，无论是显式设置还是使用先前计算的默认值。如果更改了连接超时时间，您还必须发出 `CHANGE REPLICATION SOURCE TO` | `CHANGE MASTER TO` 来调整心跳间隔至适当值，以确保它在连接超时之前发生。

+   `slave_parallel_type`

    | 命令行格式 | `--slave-parallel-type=value` |
    | --- | --- |
    | 已弃用 | 8.0.26 |
    | 系统变量 | `slave_parallel_type` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 枚举 |
    | 默认值（≥ 8.0.27） | `LOGICAL_CLOCK` |
    | 默认值（≤ 8.0.26） | `DATABASE` |
    | 有效值 | `DATABASE``LOGICAL_CLOCK` |

    从 MySQL 8.0.26 开始，`slave_parallel_type` 已被弃用，应改用别名 `replica_parallel_type`。在 MySQL 8.0.26 之前的版本中，请使用 `slave_parallel_type`。

    对于多线程副本（将`replica_parallel_workers`或`slave_parallel_workers`设置为大于 0 的值的副本），`slave_parallel_type`指定了用于决定在副本上哪些事务允许并行执行的策略。该变量对于未启用多线程的副本没有影响。可能的值包括：

    +   `LOGICAL_CLOCK`：在源头上属于同一二进制日志组提交的事务在副本上并行应用。事务之间的依赖关系基于它们的时间戳进行跟踪，以在可能的情况下提供额外的并行化。当设置了这个值时，可以在源头上使用`binlog_transaction_dependency_tracking`系统变量来指定在写入集可用的情况下，使用写入集代替时间戳进行并行化，如果写入集对事务提供了改进的结果。

    +   `DATABASE`：更新不同数据库的事务并行应用。只有在数据被分区到多个独立并同时在源头上更新的数据库时，此值才合适。在副本上不得存在跨数据库约束，因为这样的约束可能会在副本上被违反。

    当`replica_preserve_commit_order=ON`或`slave_preserve_commit_order=ON`被设置时，只能使用`LOGICAL_CLOCK`。在 MySQL 8.0.27 之前，默认为`DATABASE`。从 MySQL 8.0.27 开始，默认情况下为副本服务器启用了多线程（默认为`replica_parallel_workers=4`），因此`LOGICAL_CLOCK`是默认值，并且设置`replica_preserve_commit_order=ON`也是默认值。

    当您的复制拓扑结构使用多级副本时，`LOGICAL_CLOCK`可能会导致每个副本距离源头越远时并行化效果减少。您可以通过在源头使用`binlog_transaction_dependency_tracking`来减少这种影响，以指定在可能的情况下使用写入集而不是时间戳进行并行化。

    当使用 `binlog_transaction_compression` 系统变量启用二进制日志事务压缩时，如果 `replica_parallel_type` 或 `slave_parallel_type` 设置为 `DATABASE`，则在调度事务之前将映射受事务影响的所有数据库。与未压缩的事务相比，使用 `DATABASE` 策略的二进制日志事务压缩可能会减少并行性，因为未压缩的事务会为每个事件映射并调度。

+   `slave_parallel_workers`

    | 命令行格式 | `--slave-parallel-workers=#` |
    | --- | --- |
    | 弃用 | 8.0.26 |
    | 系统变量 | `slave_parallel_workers` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值（≥ 8.0.27） | `4` |
    | 默认值（≤ 8.0.26） | `0` |
    | 最小值 | `0` |
    | 最大值 | `1024` |

    从 MySQL 8.0.26 开始，`slave_parallel_workers` 已弃用，应改用别名 `replica_parallel_workers`。在 MySQL 8.0.26 之前的版本中，请使用 `slave_parallel_workers`。

    `slave_parallel_workers` 启用副本上的多线程，并设置用于并行执行复制事务的应用程序线程数。当值大于 0 时，副本是具有指定数量的应用程序线程的多线程副本，还有一个协调器线程来管理它们。如果使用多个复制通道，则每个通道都有这个数量的线程。

    在 MySQL 8.0.27 之前，此系统变量的默认值为 0，因此默认情况下副本不是多线程的。从 MySQL 8.0.27 开始，默认值为 4，因此默认情况下副本是多线程的。

    当启用多线程时，支持事务的重试。当设置`replica_preserve_commit_order=ON`或`slave_preserve_commit_order=ON`时，副本上的事务以与副本的中继日志中出现的顺序相同的顺序在副本上外部化。事务在应用程序线程之间分配的方式由`replica_parallel_type`（从 MySQL 8.0.26 开始）或`slave_parallel_type`（在 MySQL 8.0.26 之前）配置。从 MySQL 8.0.27 开始，这些系统变量也具有适用于多线程的默认值。

    要禁用并行执行，将`replica_parallel_workers`设置为 0，这将为副本提供一个单独的应用程序线程和没有协调器线程。在这种设置下，`replica_parallel_type`或`slave_parallel_type`以及`replica_preserve_commit_order`或`slave_preserve_commit_order`系统变量没有效果，并且被忽略。从 MySQL 8.0.27 开始，如果在副本上启用`GTID_ONLY`选项时禁用并行执行，副本实际上会使用一个并行工作者来利用无需访问文件位置即可重试事务的方法。使用一个并行工作者，`replica_preserve_commit_order`（`slave_preserve_commit_order`）系统变量也没有效果。

    设置`replica_parallel_workers`没有立即效果。变量的状态适用于所有后续的`START REPLICA`语句。

    以前，NDB Cluster 不支持多线程副本，静默地忽略了此变量的设置。这个限制在 MySQL 8.0.33 中解除。

+   `slave_pending_jobs_size_max`

    | 命令行格式 | `--slave-pending-jobs-size-max=#` |
    | --- | --- |
    | 已弃用 | 8.0.26 |
    | 系统变量 | `slave_pending_jobs_size_max` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值（≥ 8.0.12） | `128M` |
    | 默认值（8.0.11） | `16M` |
    | 最小值 | `1024` |
    | 最大值 | `16EiB` |
    | 单位 | 字节 |
    | 块大小 | `1024` |

    从 MySQL 8.0.26 开始，`slave_pending_jobs_size_max` 已被弃用，应改用别名 `replica_pending_jobs_size_max`。在 MySQL 8.0.26 之前的版本中，请使用 `slave_pending_jobs_size_max`。

    对于多线程复制，此变量设置了尚未应用的事件的接收器队列可用的最大内存量（以字节为单位）。设置此变量对未启用多线程的复制品没有影响。设置此变量不会立即生效。变量的状态适用于所有后续的 `START REPLICA` 命令。

    此变量的最小可能值为 1024 字节；默认值为 128MB。最大可能值为 18446744073709551615（16 exbibytes）。不是 1024 字节的精确倍数的值在存储之前会被舍入到下一个较低的 1024 字节的倍数。

    此变量的值是一个软限制，可以设置为匹配正常工作负载。如果异常大的事件超过此大小，事务将被保持，直到所有工作线程的队列为空，然后再处理。所有后续事务将被保持，直到大事务完成。

+   `slave_preserve_commit_order`

    | 命令行格式 | `--slave-preserve-commit-order[={OFF&#124;ON}]` |
    | --- | --- |
    | 弃用 | 8.0.26 |
    | 系统变量 | `slave_preserve_commit_order` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值（≥ 8.0.27） | `ON` |
    | 默认值（≤ 8.0.26） | `OFF` |

    从 MySQL 8.0.26 开始，`slave_preserve_commit_order` 已被弃用，应改用别名 `replica_preserve_commit_order`。在 MySQL 8.0.26 之前的版本中，请使用 `slave_preserve_commit_order`。

    对于多线程副本（将`replica_parallel_workers`或`slave_parallel_workers`设置为大于 0 的值的副本），设置`slave_preserve_commit_order=1`确保事务在副本上以与它们在副本的中继日志中出现的顺序相同的顺序执行和提交。这可以防止在已从副本的中继日志执行的事务序列中出现间隙，并保留与源相同的事务历史（具有下面列出的限制）。此变量对未启用多线程的副本无效。

    在 MySQL 8.0.27 之前，此系统变量的默认值为`OFF`，意味着事务可能会按顺序提交。从 MySQL 8.0.27 开始，默认情况下为副本服务器启用了多线程（默认为`replica_parallel_workers=4`），因此`slave_preserve_commit_order=ON`是默认设置，设置`slave_parallel_type=LOGICAL_CLOCK`也是默认设置。同样从 MySQL 8.0.27 开始，如果将`slave_parallel_workers`设置为 1，则`slave_preserve_commit_order`的设置将被忽略，因为在这种情况下，事务的顺序仍然会被保留。

    截至 MySQL 8.0.18，设置`slave_preserve_commit_order=ON`要求在副本上启用二进制日志（`log_bin`）和副本更新日志（`log_slave_updates`），这是从 MySQL 8.0 开始的默认设置。从 MySQL 8.0.19 开始，在副本上设置`slave_preserve_commit_order=ON`不再需要启用二进制日志和副本更新日志，并且如果需要的话可以禁用。在所有版本中，设置`slave_preserve_commit_order=ON`要求将`slave_parallel_type`设置为`LOGICAL_CLOCK`，这在 MySQL 8.0.27 之前*不*是默认设置。在更改`slave_preserve_commit_order`和`slave_parallel_type`的值之前，必须停止复制 SQL 线程（如果使用多个复制通道，则对所有复制通道的复制 SQL 线程都要停止）。

    当`slave_preserve_commit_order=OFF`被设置时，这是默认设置，多线程复制的副本并行应用的事务可能会无序提交。因此，检查最近执行的事务并不能保证源端的所有先前事务在副本上已执行。副本的中继日志中执行的事务序列可能存在间隙的可能性。这对于使用多线程复制时的日志记录和恢复有影响。更多信息请参见 Section 19.5.1.34, “复制和事务不一致性”。

    当`slave_preserve_commit_order=ON`被设置时，执行的工作线程会等待所有先前的事务提交后再进行提交。当一个线程在等待其他工作线程提交事务时，它会报告其状态为`等待前面的事务提交`。使用这种模式，多线程复制永远不会进入源端不在的状态。这支持将复制用于读取扩展。更多信息请参见 Section 19.4.5, “使用复制进行扩展”。

    注意

    +   `slave_preserve_commit_order=ON`不会阻止源二进制日志位置滞后，其中`Exec_master_log_pos`落后于已执行事务的位置。请参阅 Section 19.5.1.34, “Replication and Transaction Inconsistencies”。

    +   `slave_preserve_commit_order=ON`不会保留副本在其二进制日志上使用过滤器时的提交顺序和事务历史，例如`--binlog-do-db`。

    +   `slave_preserve_commit_order=ON`不会保留非事务性 DML 更新的顺序。这些更新可能会在中继日志中的先前事务之前提交，这可能导致从副本的中继日志中执行的事务序列中出现间隙。

    +   在 MySQL 8.0.19 之前的版本中，`slave_preserve_commit_order=ON`在对象不存在时不会保留带有`IF EXISTS`子句的语句的顺序。这些语句可能会在中继日志中的先前事务之前提交，这可能导致从副本的中继日志中执行的事务序列中出现间隙。

    +   如果使用基于语句的复制，并且事务性和非事务性存储引擎参与在源上回滚的非 XA 事务，可能会出现保留副本上提交顺序的限制。通常，在源上回滚的非 XA 事务不会被复制到副本，但在这种特殊情况下，该事务可能会被复制到副本。如果发生这种情况，没有二进制日志记录的多线程副本无法处理事务回滚，因此在这种情况下，副本上的提交顺序会与事务的中继日志顺序发生分歧。

+   `slave_rows_search_algorithms`

    | 命令行格式 | `--slave-rows-search-algorithms=value` |
    | --- | --- |
    | 弃用 | 8.0.18 |
    | 系统变量 | `slave_rows_search_algorithms` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 设置 |
    | 默认值 | `INDEX_SCAN,HASH_SCAN` |
    | 有效取值 | `TABLE_SCAN,INDEX_SCAN``INDEX_SCAN,HASH_SCAN``TABLE_SCAN,HASH_SCAN``TABLE_SCAN,INDEX_SCAN,HASH_SCAN`（等同于 INDEX_SCAN,HASH_SCAN） |

    在为基于行的日志记录和复制准备行批次时，此系统变量控制如何搜索匹配的行，特别是是否使用哈希扫描。现在已弃用此系统变量。默认设置 `INDEX_SCAN,HASH_SCAN` 对性能最佳，并在所有情况下都能正常工作。请参阅 Section 19.5.1.27, “Replication and Row Searches”。

+   `slave_skip_errors`

    | 命令行格式 | `--slave-skip-errors=name` |
    | --- | --- |
    | 已弃用 | 8.0.26 |
    | 系统变量 | `slave_skip_errors` |
    | 作用范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 字符串 |
    | 默认值 | `OFF` |
    | 有效值 | `OFF``[错误代码列表]``all``ddl_exist_errors` |

    从 MySQL 8.0.26 开始，`slave_skip_errors` 已弃用，应使用别名 `replica_skip_errors`。在 MySQL 8.0.26 之前的版本中，请使用 `slave_skip_errors`。

    通常情况下，当副本出现错误时，复制过程会停止，这给了你机会手动解决数据不一致性。此变量使得复制 SQL 线程在语句返回变量值中列出的任何错误时继续复制。

+   `replica_skip_errors`

    | 命令行格式 | `--replica-skip-errors=name` |
    | --- | --- |
    | 引入版本 | 8.0.26 |
    | 系统变量 | `replica_skip_errors` |
    | 作用范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 字符串 |
    | 默认值 | `OFF` |
    | 有效值 | `OFF``[错误代码列表]``all``ddl_exist_errors` |

    从 MySQL 8.0.26 开始，应使用 `replica_skip_errors` 替代 `slave_skip_errors`，后者从该版本开始已被弃用。在 MySQL 8.0.26 之前的版本中，请使用 `slave_skip_errors`。

    通常情况下，当副本���现错误时，复制过程会停止，这给了你机会手动解决数据不一致性。此变量使得复制 SQL 线程在语句返回变量值中列出的任何错误时继续复制。

+   `slave_sql_verify_checksum`

    | 命令行格式 | `--slave-sql-verify-checksum[={OFF&#124;ON}]` |
    | --- | --- |
    | 已弃用 | 8.0.26 |
    | 系统变量 | `slave_sql_verify_checksum` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` Hint Applies | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `ON` |

    从 MySQL 8.0.26 开始，`slave_sql_verify_checksum` 已弃用，应改用别名 `replica_sql_verify_checksum`。在 MySQL 8.0.26 之前的版本中，请使用 `slave_sql_verify_checksum`。

    `slave_sql_verify_checksum` 导致复制 SQL 线程使用从中继日志读取的校验和来验证数据。在出现不匹配时，副本将停止并显示错误。设置此变量立即对所有复制通道生效，包括正在运行的通道。

    注意

    复制 I/O（接收器）线程在从网络接收事件时始终尽可能读取校验和。

+   `slave_transaction_retries`

    | 命令行格式 | `--slave-transaction-retries=#` |
    | --- | --- |
    | 已弃用 | 8.0.26 |
    | 系统变量 | `slave_transaction_retries` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` Hint Applies | 否 |
    | 类型 | 整数 |
    | 默认值 | `10` |
    | 最小值 | `0` |
    | 最大值（64 位平台） | `18446744073709551615` |
    | 最大值（32 位平台） | `4294967295` |

    从 MySQL 8.0.26 开始，`slave_transaction_retries` 已弃用，应改用别名 `replica_transaction_retries`。在 MySQL 8.0.26 之前的版本中，请使用 `slave_transaction_retries`。

    `slave_transaction_retries` 设置单线程或多线程副本上的复制 SQL 线程在停止之前自动重试失败事务的最大次数。设置此变量立即对所有复制通道生效，包括正在运行的通道。默认值为 10。将变量设置为 0 会禁用事务的自动重试。

    如果复制 SQL 线程由于 `InnoDB` 死锁或因事务执行时间超过 `InnoDB` 的 `innodb_lock_wait_timeout` 或 `NDB` 的 `TransactionDeadlockDetectionTimeout` 或 `TransactionInactiveTimeout` 而无法执行事务时，它会在停止并报错之前自动重试 `slave_transaction_retries` 次。具有非临时错误的事务不会被重试。

    Performance Schema 表 `replication_applier_status` 显示了每个复制通道上发生的重试次数，在 `COUNT_TRANSACTIONS_RETRIES` 列中。Performance Schema 表 `replication_applier_status_by_worker` 显示了单线程或多线程副本上各个应用程序线程对事务重试的详细信息，并标识导致最后一个事务和当前正在进行的事务重新尝试的错误。

+   `slave_type_conversions`

    | 命令行格式 | `--slave-type-conversions=set` |
    | --- | --- |
    | 弃用 | 8.0.26 |
    | 系统变量 | `slave_type_conversions` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 集合 |
    | 默认值 |  |
    | 有效值 | `ALL_LOSSY``ALL_NON_LOSSY``ALL_SIGNED``ALL_UNSIGNED` |

    从 MySQL 8.0.26 开始，`slave_type_conversions` 已被弃用，应改用别名 `replica_type_conversions`。在 MySQL 8.0.26 之前的版本中，请使用 `slave_type_conversions`。

    `slave_type_conversions` 控制着在使用基于行的复制时副本上生效的类型转换模式。其值是一个由逗号分隔的零个或多个元素的集合：`ALL_LOSSY`、`ALL_NON_LOSSY`、`ALL_SIGNED`、`ALL_UNSIGNED`。将此变量设置为空字符串以禁止源和副本之间的类型转换。设置此变量立即对所有复制通道生效，包括正在运行的通道。

    有关适用于基于行的复制中属性提升和降级的类型转换模式的更多信息，请参阅基于行的复制：属性提升和降级。

+   `sql_replica_skip_counter`

    | 引入版本 | 8.0.26 |
    | --- | --- |
    | 系统变量 | `sql_replica_skip_counter` |
    | 作用域 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `0` |
    | 最小值 | `0` |
    | 最大值 | `4294967295` |

    从 MySQL 8.0.26 开始，使用 `sql_replica_skip_counter` 代替 `sql_slave_skip_counter`，后者从该版本开始已被弃用。在 MySQL 8.0.26 之前的版本中，请使用 `sql_slave_skip_counter`。

    `sql_replica_skip_counter` 指定副本应跳过的源事件数。设置该选项没有立即效果。该变量适用于下一个 `START REPLICA` 语句；下一个 `START REPLICA` 语句还会将值更改回 0。当此变量设置为非零值且配置了多个复制通道时，`START REPLICA` 语句只能与 `FOR CHANNEL *`channel`*` 子句一起使用。

    该选项与基于 GTID 的复制不兼容，在设置`gtid_mode=ON`时不能将其设置为非零值。如果在使用 GTIDs 时需要跳过事务，请改用源端的`gtid_executed`。如果已经在复制通道上启用了 GTID 分配，使用`CHANGE REPLICATION SOURCE TO`语句的`ASSIGN_GTIDS_TO_ANONYMOUS_TRANSACTIONS`选项，`sql_replica_skip_counter`可用。请参见第 19.1.7.3 节，“跳过事务”。

    重要

    如果跳过设置该变量指定的事件数量会导致复制品从事件组中间开始，那么复制品将继续跳过，直到找到下一个事件组的开头并从那一点开始。更多信息，请参见第 19.1.7.3 节，“跳过事务”。

+   `sql_slave_skip_counter`

    | 已弃用 | 8.0.26 |
    | --- | --- |
    | 系统变量 | `sql_slave_skip_counter` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `0` |
    | 最小值 | `0` |
    | 最大值 | `4294967295` |

    从 MySQL 8.0.26 开始，`sql_slave_skip_counter`已弃用，应改用别名`sql_replica_skip_counter`。在 MySQL 8.0.26 之前的版本中，请使用`sql_slave_skip_counter`。

    `sql_slave_skip_counter`指定复制品应该跳过的源事件数量。设置该选项没有立即效果。该变量适用于下一个`START REPLICA`语句；下一个`START REPLICA`语句也会将值更改回 0。当该变量设置为非零值且配置了多个复制通道时，`START REPLICA`语句只能与`FOR CHANNEL *`channel`*`子句一起使用。

    此选项与基于 GTID 的复制不兼容，在设置`gtid_mode=ON`时不能设置为非零值。如果在使用 GTIDs 时需要跳过事务，请改用源端的`gtid_executed`。如果已经在复制通道上启用了 GTID 分配，使用`CHANGE REPLICATION SOURCE TO`语句的`ASSIGN_GTIDS_TO_ANONYMOUS_TRANSACTIONS`选项，可以使用`sql_slave_skip_counter`。参见第 19.1.7.3 节，“跳过事务”。

    重要提示

    如果通过设置此变量跳过指定数量的事件会导致副本在事件组中间开始，副本将继续跳过，直到找到下一个事件组的开头并从那里开始。有关更多信息，请参见第 19.1.7.3 节，“跳过事务”。

+   `sync_master_info`

    | 命令行格式 | `--sync-master-info=#` |
    | --- | --- |
    | 已弃用 | 8.0.26 |
    | 系统变量 | `sync_master_info` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `10000` |
    | 最小值 | `0` |
    | 最大值 | `4294967295` |

    从 MySQL 8.0.26 开始，`sync_master_info`已被弃用，应改用别名`sync_source_info`。在 MySQL 8.0.26 之前的版本中，请使用`sync_master_info`。

    `sync_master_info`指定副本在更新连接元数据存储库之前的事件数量。当连接元数据存储库存储为`InnoDB`表时（从 MySQL 8.0 开始默认），在此事件数量之后进行更新。如果连接元数据存储库存储为文件（从 MySQL 8.0 开始不推荐使用），副本在此事件数量之后将其`master.info`文件同步到磁盘（使用`fdatasync()`）。默认值为 10000，零值表示存储库永远不会更新。设置此变量立即对所有复制通道生效，包括正在运行的通道。

+   `sync_relay_log`

    | 命令行格式 | `--sync-relay-log=#` |
    | --- | --- |
    | 系统变量 | `sync_relay_log` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` Hint Applies | 否 |
    | 类型 | 整数 |
    | 默认值 | `10000` |
    | 最小值 | `0` |
    | 最大值 | `4294967295` |

    如果此变量的值大于 0，则 MySQL 服务器在每写入`sync_relay_log`事件到中继日志后将其中继日志同步到磁盘（使用`fdatasync()`）。设置此变量立即对所有复制通道生效，包括正在运行的通道。

    将`sync_relay_log`设置为 0 会导致不对磁盘进行同步；在这种情况下，服务器依赖操作系统定期刷新中继日志的内容，就像对任何其他文件一样。

    选择值为 1 是最安全的选择，因为在意外停机时，您最多只会丢失一个中继日志事件。然而，这也是最慢的选择（除非磁盘具有带电池备份缓存，这样同步会非常快）。有关在复制品上设置的组合对于意外停机最具弹性的信息，请参阅第 19.4.2 节，“处理复制品的意外停机”。

+   `sync_relay_log_info`

    | 命令行格式 | `--sync-relay-log-info=#` |
    | --- | --- |
    | 已弃用 | 8.0.34 |
    | 系统变量 | `sync_relay_log_info` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` Hint Applies | 否 |
    | 类型 | 整数 |
    | 默认值 | `10000` |
    | 最小值 | `0` |
    | 最大值 | `4294967295` |

    复制品更新应用程序元数据存储库之后的事务数。当应用程序元数据存储库存储为一个`InnoDB`表（MySQL 8.0 及以后版本的默认值），它在每个事务之后更新，此系统变量将被忽略。如果应用程序元数据存储库存储为一个文件（MySQL 8.0 中已弃用），则复制品在此数量的事务之后将其`relay-log.info`文件同步到磁盘（使用`fdatasync()`）。`0`（零）表示文件内容仅由操作系统刷新。设置此变量立即对所有复制通道生效，包括正在运行的通道。

    由于存储应用程序元数据为文件已被弃用，此变量也已被弃用；从 MySQL 8.0.34 开始，服务器在设置或读取其值时会发出警告。您应该期望`sync_relay_log_info`在将来的 MySQL 版本中被移除，并立即迁移可能依赖它的应用程序。

+   `sync_source_info`

    | 命令行格式 | `--sync-source-info=#` |
    | --- | --- |
    | 引入版本 | 8.0.26 |
    | 系统变量 | `sync_source_info` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `10000` |
    | 最小值 | `0` |
    | 最大值 | `4294967295` |

    从 MySQL 8.0.26 开始，使用 `sync_source_info` 替代从该版本开始已弃用的 `sync_master_info`。在 MySQL 8.0.26 之前的版本中，请使用 `sync_source_info`。

    `sync_source_info` 指定了副本更新连接元数据存储库之后的事件数量。当连接元数据存储库存储为 `InnoDB` 表时（从 MySQL 8.0 开始默认），在此事件数量之后进行更新。如果连接元数据存储库存储为文件（从 MySQL 8.0 开始已弃用），则副本在此事件数量之后将其 `master.info` 文件同步到磁盘（使用 `fdatasync()`）。默认值为 10000，零值表示存储库永远不会更新。设置此变量立即对所有复制通道生效，包括正在运行的通道。

+   `terminology_use_previous`

    | 命令行格式 | `--terminology-use-previous=#` |
    | --- | --- |
    | 引入版本 | 8.0.26 |
    | 系统变量 | `terminology_use_previous` |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 枚举 |
    | 默认值 | `NONE` |
    | 有效值 | `NONE``BEFORE_8_0_26` |

    在 MySQL 8.0.26 中，对包含术语 `master`、`slave` 和 `mts`（代表“多线程从属”）的仪表命名进行了不兼容的更改，分别更改为 `source`、`replica` 和 `mta`（代表“多线程应用程序”）。如果这些不兼容的更改影响到您的应用程序，请将 `terminology_use_previous` 系统变量设置为 `BEFORE_8_0_26`，以便 MySQL Server 使用前述列表中指定对象的旧版本名称。这样一来，依赖于旧名称的监控工具将继续工作，直到它们可以更新为使用新名称。

    将`terminology_use_previous`系统变量设置为会话范围以支持个别用户，或者设置为全局范围以成为所有新会话的默认值。当使用全局范围时，慢查询日志包含旧版本的名称。

    受影响的仪表化名称列在以下列表中。`terminology_use_previous`系统变量仅影响这些项目。它不影响在 MySQL 8.0.26 中引入的系统变量、状态变量和命令行选项的新别名，当设置时仍然可以使用这些别名。

    +   仪表化锁（互斥锁），在具有前缀`wait/synch/mutex/`的`mutex_instances`和`events_waits_*`性能模式表中可见

    +   读/写锁，在具有前缀`wait/synch/rwlock/`的`rwlock_instances`和`events_waits_*`性能模式表中可见

    +   仪表化条件变量，在具有前缀`wait/synch/cond/`的`cond_instances`和`events_waits_*`性能模式表中可见

    +   仪表化内存分配，在具有前缀`memory/sql/`的`memory_summary_*`性能模式表中可见

    +   线程名称，在具有前缀`thread/sql/`的`threads`性能模式表中可见

    +   线程阶段，在具有前缀`stage/sql/`的`events_stages_*`性能模式表中可见，在`threads`和`processlist`性能模式表中没有前缀，在`SHOW PROCESSLIST`语句的输出中，在信息模式`processlist`表中，在慢查询日志中

    +   线程命令，在具有前缀`statement/com/`的`events_statements_history*`和`events_statements_summary_*_by_event_name`性能模式表中可见，在`threads`和`processlist`性能模式表中没有前缀，在`SHOW PROCESSLIST`语句的输出中，在信息模式`processlist`表中，在`SHOW REPLICA STATUS`语句的输出中
