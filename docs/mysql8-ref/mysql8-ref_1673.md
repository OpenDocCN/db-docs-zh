> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-options-variables.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-options-variables.html)

#### 25.4.3.9 NDB 集群的 MySQL 服务器选项和变量

本节提供了关于 MySQL 服务器选项、服务器和状态变量的信息，这些信息特定于 NDB 集群。有关如何使用这些信息以及不特定于 NDB 集群的其他选项和变量的一般信息，请参见第 7.1 节，“MySQL 服务器”。

有关在集群配置文件中使用的 NDB 集群配置参数（通常命名为 `config.ini`）的信息，请参见第 25.4 节，“NDB 集群的配置”。

##### 25.4.3.9.1 NDB 集群的 MySQL 服务器选项

本节提供了关于 NDB 集群相关的 **mysqld** 服务器选项的描述。有关不特定于 NDB 集群的 **mysqld** 选项的信息，以及有关在 **mysqld** 中使用选项的一般信息，请参见第 7.1.7 节，“服务器命令选项”。

有关与其他 NDB 集群进程一起使用的命令行选项的信息，请参见第 25.5 节，“NDB 集群程序”。

+   `--ndbcluster`

    | 命令行格式 | `--ndbcluster[=value]` |
    | --- | --- |
    | 被 `skip-ndbcluster` 禁用 | 是 |
    | 类型 | 枚举 |
    | 默认值 | `ON` |
    | 有效数值 | `OFF``FORCE` |

    使用 NDB 集群必须使用 `NDBCLUSTER` 存储引擎。如果 **mysqld** 二进制文件包含对 `NDBCLUSTER` 存储引擎的支持，则该引擎默认处于禁用状态。使用 `--ndbcluster` 选项来启用它。使用 `--skip-ndbcluster` 明确禁用该引擎。

    如果同时使用 `--initialize`，则 `--ndbcluster` 选项将被忽略（并且 `NDB` 存储引擎 *不* 会被启用）。（使用这个选项与 `--initialize` 一起既不必要也不可取。）

+   `--ndb-allow-copying-alter-table=[ON|OFF]`

    | 命令行格式 | `--ndb-allow-copying-alter-table[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `ndb_allow_copying_alter_table` |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔 |
    | 默认值 | `ON` |

    让`ALTER TABLE`和其他 DDL 语句在`NDB`表上使用复制操作。将其设置为`OFF`可防止此操作发生；这样做可能会提高关键应用程序的性能。

+   `--ndb-applier-allow-skip-epoch`

    | 命令行格式 | `--ndb-applier-allow-skip-epoch` |
    | --- | --- |
    | 引入版本 | 8.0.28-ndb-8.0.28 |
    | 系统变量 | `ndb_applier_allow_skip_epoch` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |

    与`--slave-skip-errors`一起使用，使`NDB`忽略跳过的 epoch 事务。单独使用时没有效果。

+   `--ndb-batch-size=*`#`*`

    | 命令行格式 | `--ndb-batch-size` |
    | --- | --- |
    | 系统变量 | `ndb_batch_size` |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `32768` |
    | 最小值 | `0` |
    | 最大值 (≥ 8.0.29-ndb-8.0.29) | `2147483648` |
    | 最大值 | `2147483648` |
    | 最大值 | `2147483648` |
    | 最大值 (≤ 8.0.28-ndb-8.0.28) | `31536000` |
    | 单位 | 字节 |

    这设置了用于 NDB 事务批处理的字节大小。

+   `--ndb-cluster-connection-pool=*`#`*`

    | 命令行格式 | `--ndb-cluster-connection-pool` |
    | --- | --- |
    | 系统变量 | `ndb_cluster_connection_pool` |
    | 系统变量 | `ndb_cluster_connection_pool` |
    | 范围 | 全局 |
    | 范围 | 全局 |
    | 动态 | 否 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `1` |
    | 最小值 | `1` |
    | 最大值 | `63` |

    通过将此选项设置为大于 1 的值（默认值），**mysqld**进程可以使用多个连接到集群，有效地模拟多个 SQL 节点。每个连接都需要在集群配置（`config.ini`）文件中拥有自己的`[api]`或`[mysqld]`部分，并计入集群支持的最大 API 连接数。

    假设您有 2 个集群主机计算机，每个计算机都运行一个 SQL 节点，其**mysqld**进程是使用`--ndb-cluster-connection-pool=4`启动的；这意味着集群必须为这些连接提供 8 个 API 插槽（而不是 2 个）。当 SQL 节点连接到集群时，所有这些连接都会建立，并以循环方式分配给线程。

    当在具有多个 CPU、多个核心或两者都有的主机上运行 **mysqld** 时，此选项才有用。为了获得最佳效果，该值应小于主机上可用的总核心数。将其设置为大于此值的值可能会严重降低性能。

    重要

    因为每个使用连接池的 SQL 节点占用多个 API 节点槽位——每个槽位在集群中都有自己的节点 ID，所以在启动任何使用连接池的 **mysqld** 进程时，不要在集群连接字符串中使用节点 ID。

    在使用 `--ndb-cluster-connection-pool` 选项时在连接字符串中设置节点 ID 会导致 SQL 节点尝试连接到集群时出现节点 ID 分配错误。

+   `--ndb-cluster-connection-pool-nodeids=*`列表`*`

    | 命令行格式 | `--ndb-cluster-connection-pool-nodeids` |
    | --- | --- |
    | 系统变量 | `ndb_cluster_connection_pool_nodeids` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 集合 |
    | 默认值 |  |

    指定用于连接到 SQL 节点使用的集群的节点 ID 的逗号分隔列表。此列表中的节点数必须与为 `--ndb-cluster-connection-pool` 选项设置的值相同。

+   `--ndb-blob-read-batch-bytes=*`字节`*`

    | 命令行格式 | `--ndb-blob-read-batch-bytes` |
    | --- | --- |
    | 系统变量 | `ndb_blob_read_batch_bytes` |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `65536` |
    | 最小值 | `0` |
    | 最大值 | `4294967295` |

    此选项可用于设置 NDB 集群应用程序中 `BLOB` 数据读取的批处理大小（以字节为单位）。当当前事务中要读取的 `BLOB` 数据量超过此批处理大小时，任何待处理的 `BLOB` 读取操作将立即执行。

    此选项的最大值为 4294967295；默认值为 65536。将其设置为 0 会禁用 `BLOB` 读取批处理。

    注意

    在 NDB API 应用程序中，您可以使用 `setMaxPendingBlobReadBytes()` 和 `getMaxPendingBlobReadBytes()` 方法来控制 `BLOB` 写入批处理。

+   `--ndb-blob-write-batch-bytes=*`字节`*`

    | 命令行格式 | `--ndb-blob-write-batch-bytes` |
    | --- | --- |
    | 系统变量 | `ndb_blob_write_batch_bytes` |
    | 作用范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `65536` |
    | 最小值 | `0` |
    | 最大值 | `4294967295` |
    | 单位 | 字节 |

    此选项可用于设置 NDB 集群应用程序中`BLOB`数据写入的批处理大小（以字节为单位）。当当前事务中要写入的`BLOB`数据量超过此批处理大小时，任何待处理的`BLOB`写入操作将立即执行。

    此选项的最大值为 4294967295；默认值为 65536。将其设置为 0 将禁用`BLOB`写入批处理。

    注意

    在 NDB API 应用程序中，您可以使用`setMaxPendingBlobWriteBytes()`和`getMaxPendingBlobWriteBytes()`方法来控制`BLOB`写入批处理。

+   `--ndb-connectstring=*`connection_string`*`

    | 命令行格式 | `--ndb-connectstring` |
    | --- | --- |
    | 类型 | 字符串 |

    在使用`NDBCLUSTER`存储引擎时，此选项指定分发集群配置数据的管理服务器。有关语法，请参见第 25.4.3.3 节，“NDB 集群连接字符串”。

+   `--ndb-default-column-format=[FIXED|DYNAMIC]`

    | 命令行格式 | `--ndb-default-column-format={FIXED&#124;DYNAMIC}` |
    | --- | --- |
    | 系统变量 | `ndb_default_column_format` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 枚举 |
    | 默认值 | `FIXED` |
    | 有效值 | `FIXED``DYNAMIC` |

    设置新表的默认`COLUMN_FORMAT`和`ROW_FORMAT`（参见第 15.1.20 节，“CREATE TABLE 语句”）。默认值为`FIXED`。

+   `--ndb-deferred-constraints=[0|1]`

    | 命令行格式 | `--ndb-deferred-constraints` |
    | --- | --- |
    | 系统变量 | `ndb_deferred_constraints` |
    | 作用范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `0` |
    | 最小值 | `0` |
    | 最大值 | `1` |

    控制是否将唯一索引的约束检查推迟到提交时间，支持这种检查。 `0`是默认值。

    此选项通常不需要用于 NDB 集群或 NDB 集群复制的操作，并主要用于测试。

+   `--ndb-schema-dist-timeout=#`

    | 命令行格式 | `--ndb-schema-dist-timeout=#` |
    | --- | --- |
    | 引入版本 | 8.0.17-ndb-8.0.17 |
    | 系统变量 | `ndb_schema_dist_timeout` |
    | 作用范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `120` |
    | 最小值 | `5` |
    | 最大值 | `1200` |
    | 单位 | 秒 |

    指定此**mysqld**等待模式操作完成的最长时间（以秒为单位），然后将其标记为已超时。

+   `--ndb-distribution=[KEYHASH|LINHASH]`

    | 命令行格式 | `--ndb-distribution={KEYHASH&#124;LINHASH}` |
    | --- | --- |
    | 系统变量 | `ndb_distribution` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 枚举 |
    | 默认值 | `KEYHASH` |
    | 有效值 | `LINHASH``KEYHASH` |

    控制`NDB`表的默认分发方法。可以设置为`KEYHASH`（键哈希）或`LINHASH`（线性哈希）之一。 `KEYHASH`是默认值。

+   `--ndb-log-apply-status`

    | 命令行格式 | `--ndb-log-apply-status[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `ndb_log_apply_status` |
    | 作用范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    导致副本**mysqld**记录从其直接来源接收的任何更新到`mysql.ndb_apply_status`表中的二进制日志中，使用自己的服务器 ID 而不是来源的服务器 ID。在循环或链式复制设置中，这允许这些更新传播到任何配置为当前**mysqld**的副本的`mysql.ndb_apply_status`表中的 MySQL 服务器。

    在链式复制设置中，使用此选项允许下游（副本）集群了解其相对于所有上游贡献者（源）的位置。

    在循环复制设置中，此选项导致对`ndb_apply_status`表的更改完成整个电路，最终传播回原始的 NDB 集群。这也允许充当复制源的集群看到其更改（时代）何时应用到圈中的其他集群。

    除非 MySQL 服务器使用`--ndbcluster`选项启动，否则此选项无效。

+   `--ndb-log-empty-epochs=[ON|OFF]`

    | 命令行格式 | `--ndb-log-empty-epochs[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `ndb_log_empty_epochs` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    导致没有更改的时代被写入`ndb_apply_status`和`ndb_binlog_index`表，即使启用了`log_replica_updates`或`log_slave_updates`。

    默认情况下，此选项已禁用。禁用`--ndb-log-empty-epochs`会导致没有更改的时代事务不会被写入二进制日志，尽管在`ndb_binlog_index`中仍会写入一行。

    因为`--ndb-log-empty-epochs=1`会导致`ndb_binlog_index`表的大小独立于二进制日志的大小增加，用户应准备好管理此表的增长，即使他们期望集群大部分时间处于空闲状态。

+   `--ndb-log-empty-update=[ON|OFF]`

    | 命令行格式 | `--ndb-log-empty-update[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `ndb_log_empty_update` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    导致没有产生更改的更新被写入`ndb_apply_status`和`ndb_binlog_index`表，即使启用了`log_replica_updates`或`log_slave_updates`。

    默认情况下，此选项已禁用（`OFF`）。禁用`--ndb-log-empty-update`会导致没有更改的更新不会被写入二进制日志。

+   `--ndb-log-exclusive-reads=[0|1]`

    | 命令行格式 | `--ndb-log-exclusive-reads[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `ndb_log_exclusive_reads` |
    | 作用范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `0` |

    使用此选项启动服务器会导致主键读取被记录为独占锁，从而允许基于读冲突的 NDB 集群复制冲突检测和解决。您还可以通过将`ndb_log_exclusive_reads`系统变量的值分别设置为 1 或 0 来在运行时启用和禁用这些锁。0（禁用锁定）是默认值。

    更多信息，请参见读冲突检测和解决。

+   `--ndb-log-fail-terminate`

    | 命令行格式 | `--ndb-log-fail-terminate` |
    | --- | --- |
    | 引入版本 | 8.0.21-ndb-8.0.21 |
    | 系统变量 | `ndb_log_fail_terminate` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `FALSE` |

    当指定此选项且无法完全记录所有找到的行事件时，**mysqld** 进程将被终止。

+   `--ndb-log-orig`

    | 命令行格式 | `--ndb-log-orig[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `ndb_log_orig` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    记录`ndb_binlog_index`表中的起始服务器 ID 和时代。

    注意

    这使得一个给定的时代可以在`ndb_binlog_index`中有多行，每行对应一个起始时代。

    更多信息，请参见第 25.7.4 节，“NDB 集群复制模式和表”。

+   `--ndb-log-transaction-dependency`

    | ���令行格式 | `--ndb-log-transaction-dependency={true&#124;false}` |
    | --- | --- |
    | 引入版本 | 8.0.33-ndb-8.0.33 |
    | 系统变量 | `ndb_log_transaction_dependency` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `false` |

    使`NDB`二进制日志线程为其写入二进制日志的每个事务计算事务依赖关系。默认值为`FALSE`。

    此选项无法在运行时设置；相应的`ndb_log_transaction_dependency`系统变量是只读的。

+   `--ndb-log-transaction-id`

    | 命令行格式 | `--ndb-log-transaction-id[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `ndb_log_transaction_id` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    导致副本**mysqld**在二进制日志的每一行中写入 NDB 事务 ID。默认值为`FALSE`。

    `--ndb-log-transaction-id`需要启用 NDB 集群复制冲突检测和解决，使用`NDB$EPOCH_TRANS()`函数（参见 NDB$EPOCH_TRANS()")）。更多信息，请参见第 25.7.12 节，“NDB 集群复制冲突解决”。

    废弃的`log_bin_use_v1_row_events`系统变量，默认值为`OFF`，在使用`--ndb-log-transaction-id=ON`时不能设置为`ON`。

+   `--ndb-log-update-as-write`

    | 命令行格式 | `--ndb-log-update-as-write[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `ndb_log_update_as_write` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `ON` |

    源上的更新是否作为更新(`OFF`)或写入(`ON`)写入二进制日志。当启用此选项，并且`--ndb-log-updated-only`和`--ndb-log-update-minimal`都被禁用时，不同类型的操作将按照以下列表描述的方式记录：

    +   `INSERT`：作为`WRITE_ROW`事件记录，没有前置图像；所有列都记录了后置图像。

        `UPDATE`：作为`WRITE_ROW`事件记录，没有前置图像；所有列都记录了后置图像。

        `DELETE`：作为`DELETE_ROW`事件记录，所有列都在前置图像中记录；后置图像未记录。

    此选项可与前面提到的其他两个 NDB 日志选项一起用于 NDB 复制冲突解决；有关更多信息，请参见 ndb_replication 表。

+   `--ndb-log-updated-only`

    | 命令行格式 | `--ndb-log-updated-only[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `ndb_log_updated_only` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `ON` |

    控制**mysqld**是否仅将更新(`ON`)或完整行(`OFF`)写入二进制日志。当启用此选项，并且同时禁用`--ndb-log-update-as-write`和`--ndb-log-update-minimal`时，不同类型的操作将按照以下列表中描述的方式记录：

    +   `INSERT`: 记录为`WRITE_ROW`事件，没有前置图像；所有列都记录在后置图像中。

    +   `UPDATE`: 记录为`UPDATE_ROW`事件，主键列和更新列同时出现在前置和后置图像中。

    +   `DELETE`: 记录为`DELETE_ROW`事件，主键列包含在前置图像中；后置图像不记录。

    此选项可用于 NDB 复制冲突解决，与之前提到的其他两个 NDB 日志选项结合使用；有关这些选项如何相互作用的更多信息，请参阅 ndb_replication Table。

+   `--ndb-log-update-minimal`

    | 命令行格式 | `--ndb-log-update-minimal[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `ndb_log_update_minimal` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    以最小方式记录更新，只在前置图像中写入主键值，在后置图像中只写入更改的列。如果复制到除`NDB`之外的存储引擎可能会导致兼容性问题。当启用此选项，并且同时禁用`--ndb-log-updated-only`和`--ndb-log-update-as-write`时，不同类型的操作将按照以下列表中描述的方式记录：

    +   `INSERT`: 记录为`WRITE_ROW`事件，没有前置图像；所有列都记录在后置图像中。

    +   `UPDATE`: 记录为`UPDATE_ROW`事件，主键列在前置图像中；除主键列外的所有列都在后置图像中记录。

    +   `DELETE`: 记录为`DELETE_ROW`事件，所有列都在前置图像中；后置图像不记录。

    此选项可用于 NDB 复制冲突解决，与之前提到的其他两个 NDB 日志选项结合使用；有关更多信息，请参阅 ndb_replication Table。

+   `--ndb-mgmd-host=*`host`*[:*`port`*]`

    | 命令行格式 | `--ndb-mgmd-host=host_name[:port_num]` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `localhost:1186` |

    可用于设置程序连接到的单个管理服务器的主机和端口号。如果程序在其连接信息中需要节点 ID 或对多个管理服务器的引用（或两者都需要），请改用`--ndb-connectstring`选项。

+   `--ndb-nodeid=#`

    | 命令行格式 | `--ndb-nodeid=#` |
    | --- | --- |
    | 状态变量 | `Ndb_cluster_node_id` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | 类型 | 整数 |
    | 默认值 | `N/A` |
    | 最小值 | `1` |
    | 最大值 | `255` |
    | 最大值 | `63` |

    在 NDB Cluster 中设置此 MySQL 服务器的节点 ID。

    `--ndb-nodeid`选项会覆盖使用`--ndb-connectstring`设置的任何节点 ID，无论这两个选项的使用顺序如何。

    此外，如果使用了`--ndb-nodeid`，则必须在`config.ini`的`[mysqld]`或`[api]`部分中找到匹配的节点 ID，或者文件中必须有一个“开放”的`[mysqld]`或`[api]`部分（即，一个没有指定`NodeId`或`Id`参数的部分）。如果节点 ID 作为连接字符串的一部分指定，这也是正确的。

    无论节点 ID 如何确定，它都显示为全局状态变量`Ndb_cluster_node_id`的值在`SHOW STATUS`输出中，并且在`SHOW ENGINE NDBCLUSTER STATUS`的`connection`行中显示为`cluster_node_id`。

    有关 NDB Cluster SQL 节点的节点 ID 的更多信息，请参见第 25.4.3.7 节，“在 NDB Cluster 中定义 SQL 和其他 API 节点”。

+   `--ndbinfo={ON|OFF|FORCE}`

    | 命令行格式 | `--ndbinfo[=value]` (≥ 8.0.13-ndb-8.0.13) |
    | --- | --- |
    | 引入版本 | 8.0.13-ndb-8.0.13 |
    | 类型 | 枚举 |
    | 默认值 | `ON` |
    | 有效值 | `ON``OFF``FORCE` |

    启用`ndbinfo`信息数据库的插件。默认情况下，只要启用`NDBCLUSTER`，此选项就为 ON。

+   `--ndb-optimization-delay=*`毫秒`*`

    | 命令行格式 | `--ndb-optimization-delay=#` |
    | --- | --- |
    | 系统变量 | `ndb_optimization_delay` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `10` |
    | 最小值 | `0` |
    | 最大值 | `100000` |
    | 单位 | 毫秒 |

    设置在`NDB`表上通过`OPTIMIZE TABLE`语句设置行之间等待的毫秒数。默认值为 10。

+   `--ndb-optimized-node-selection`

    | 命令行格式 | `--ndb-optimized-node-selection` |
    | --- | --- |

    启用用于选择事务节点的优化。默认情况下启用；使用`--skip-ndb-optimized-node-selection`来禁用。

+   `--ndb-transid-mysql-connection-map=*`state`*`

    | 命令行格式 | `--ndb-transid-mysql-connection-map[=state]` |
    | --- | --- |
    | 类型 | 枚举 |
    | 默认值 | `ON` |
    | 有效值 | `ON``OFF``FORCE` |

    启用或禁用处理`INFORMATION_SCHEMA`数据库中`ndb_transid_mysql_connection_map`表的插件。接受`ON`、`OFF`或`FORCE`值。`ON`（默认）启用插件。`OFF`禁用插件，使`ndb_transid_mysql_connection_map`不可访问。`FORCE`使 MySQL 服务器在插件加载和启动失败时无法启动。

    通过检查`SHOW PLUGINS`的输出，您可以查看`ndb_transid_mysql_connection_map`表插件是否正在运行。

+   `--ndb-wait-connected=*`seconds`*`

    | 命令行格式 | `--ndb-wait-connected=#` |
    | --- | --- |
    | 系统变量 | `ndb_wait_connected` |
    | 作用范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值（≥ 8.0.27-ndb-8.0.27） | `120` |
    | 默认值（≤ 8.0.26-ndb-8.0.26） | `30` |
    | 默认值 | `30` |
    | 最小值 | `0` |
    | 最大值 | `31536000` |
    | 单位 | 秒 |

    此选项设置 MySQL 服务器在接受 MySQL 客户端连接之前等待与 NDB Cluster 管理和数据节点建立连接的时间段。时间以秒为单位。默认值为`30`。

+   `--ndb-wait-setup=*`seconds`*`

    | 命令行格式 | `--ndb-wait-setup=#` |
    | --- | --- |
    | 系统变量 | `ndb_wait_setup` |
    | 作用范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值（≥ 8.0.27-ndb-8.0.27） | `120` |
    | 默认值（≤ 8.0.26-ndb-8.0.26） | `30` |
    | 默认值 | `30` |
    | 默认值 | `15` |
    | 默认值 | `15` |
    | 最小值 | `0` |
    | 最大值 | `31536000` |
    | 单位 | 秒 |

    此变量显示 MySQL 服务器在超时并将`NDB`视为不可用之前等待`NDB`存储引擎完成设置的时间段。时间以秒为单位。默认值为`30`。

+   `--skip-ndbcluster`

    | 命令行格式 | `--skip-ndbcluster` |
    | --- | --- |

    禁用`NDBCLUSTER`存储引擎。这是使用支持`NDBCLUSTER`存储引擎构建的二进制文件的默认设置；只有在显式给出`--ndbcluster`选项时，服务器才为该存储引擎分配内存和其他资源。有关示例，请参见第 25.4.1 节，“NDB Cluster 的快速测试设置”。

##### 25.4.3.9.2 NDB Cluster 系统变量

本节提供了关于 MySQL 服务器特定于 NDB Cluster 和`NDB`存储引擎的系统变量的详细信息。有关不特定于 NDB Cluster 的系统变量，请参见第 7.1.8 节，“服务器系统变量”。有关使用系统变量的一般信息，请参见第 7.1.9 节，“使用系统变量”。

+   `ndb_autoincrement_prefetch_sz`

    | 命令行格式 | `--ndb-autoincrement-prefetch-sz=#` |
    | --- | --- |
    | 系统变量 | `ndb_autoincrement_prefetch_sz` |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 (≥ 8.0.19-ndb-8.0.19) | `512` |
    | 默认值 (≤ 8.0.18-ndb-8.0.18) | `1` |
    | 最小值 | `1` |
    | 最大值 | `65536` |

    确定自增列中间隔的概率。将其设置为`1`以最小化这种情况。将其设置为较高值以进行优化可以加快插入速度，但会降低在一批插入中使用连续自增编号的可能性。

    此变量仅影响在语句之间获取的`AUTO_INCREMENT` ID 的数量；在给定语句中，每次至少获取 32 个 ID。

    重要

    此变量不影响使用`INSERT ... SELECT`执行的插入操作。

+   `ndb_clear_apply_status`

    | 命令行格式 | `--ndb-clear-apply-status[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `ndb_clear_apply_status` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `ON` |

    默认情况下，执行`RESET SLAVE`会导致 NDB 集群副本从其`ndb_apply_status`表中清除所有行。您可以通过设置`ndb_clear_apply_status=OFF`来禁用此功能。

+   `ndb_conflict_role`

    | 命令行格式 | `--ndb-conflict-role=value` |
    | --- | --- |
    | 引入 | 8.0.23-ndb-8.0.23 |
    | 系统变量 | `ndb_conflict_role` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 枚举 |
    | 默认值 | `NONE` |
    | 有效值 | `NONE``PRIMARY``SECONDARY``PASS` |

    确定此 SQL 节点（和 NDB 集群）在循环（“主动-主动”）复制设置中的角色。`ndb_slave_conflict_role`可以取`PRIMARY`、`SECONDARY`、`PASS`或`NULL`（默认）中的任何一个值。在更改`ndb_slave_conflict_role`之前，必须停止副本 SQL 线程。此外，不可能直接在`PASS`和`PRIMARY`或`SECONDARY`之间直接更改；在这种情况下，您必须确保 SQL 线程已停止，然后首先执行`SET @@GLOBAL.ndb_slave_conflict_role = 'NONE'`。

    此变量取代了`ndb_slave_conflict_role`，自 NDB 8.0.23 起已被弃用。

    有关更多信息，请参见第 25.7.12 节，“NDB 集群复制冲突解决”。

+   `ndb_data_node_neighbour`

    | 命令行格式 | `--ndb-data-node-neighbour=#` |
    | --- | --- |
    | 系统变量 | `ndb_data_node_neighbour` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `0` |
    | 最小值 | `0` |
    | 最大值 | `255` |

    设置“最近”数据节点的 ID——即，选择一个首选的非本地数据节点来执行事务，而不是在与 SQL 或 API 节点相同主机上运行的节点上执行。这用于确保在访问完全复制的表时，我们在此数据节点上访问它，以确保尽可能始终使用表的本地副本。这也可用于为事务提供提示。

    这可以提高数据访问时间，如果一个节点比其他同一主机上的节点更接近，从而具有更高的网络吞吐量。

    有关更多信息，请参见第 15.1.20.12 节，“设置 NDB 注释选项”。

    注意

    NDB API 应用程序提供了一个等效的方法`set_data_node_neighbour()`。

+   `ndb_dbg_check_shares`

    | 命令行格式 | `--ndb-dbg-check-shares=#` |
    | --- | --- |
    | 引入版本 | 8.0.13-ndb-8.0.13 |
    | 系统变量 | `ndb_dbg_check_shares` |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `0` |
    | 最小值 | `0` |
    | 最大值 | `1` |

    当设置为 1 时，检查是否有未完成的共享。仅在调试构建中可用。

+   `ndb_default_column_format`

    | 命令行格式 | `--ndb-default-column-format={FIXED | DYNAMIC}` |
    | --- | --- | --- |
    | 系统变量 | `ndb_default_column_format` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 枚举 |
    | 默认值 | `FIXED` |
    | 有效值 | `FIXED``DYNAMIC` |

    设置新表的默认`COLUMN_FORMAT`和`ROW_FORMAT`（参见第 15.1.20 节，“CREATE TABLE Statement”）。默认值为`FIXED`。

+   `ndb_deferred_constraints`

    | 命令行格式 | `--ndb-deferred-constraints=#` |
    | --- | --- |
    | 系统变量 | `ndb_deferred_constraints` |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `0` |
    | 最小值 | `0` |
    | 最大值 | `1` |

    控制是否延迟约束检查，如果支持的话。`0`是默认值。

    此变量通常不需要用于 NDB Cluster 或 NDB Cluster Replication 的操作，主要用于测试目的。

+   `ndb_distribution`

    | 命令行格式 | `--ndb-distribution={KEYHASH | LINHASH}` |
    | --- | --- | --- |
    | 系统变量 | `ndb_distribution` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 枚举 |
    | 默认值 | `KEYHASH` |
    | 有效值 | `LINHASH``KEYHASH` |

    控制`NDB`表的默认分布方法。可以设置为`KEYHASH`（键哈希）或`LINHASH`（线性哈希）。`KEYHASH` 是默认值。

+   `ndb_eventbuffer_free_percent`

    | 命令行格式 | `--ndb-eventbuffer-free-percent=#` |
    | --- | --- |
    | 系统变量 | `ndb_eventbuffer_free_percent` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `20` |
    | 最小值 | `1` |
    | 最大值 | `99` |

    设置事件缓冲区（ndb_eventbuffer_max_alloc）在达到最大值后，在重新开始缓冲之前应该保留的最大内存百分比。

+   `ndb_eventbuffer_max_alloc`

    | 命令行格式 | `--ndb-eventbuffer-max-alloc=#` |
    | --- | --- |
    | 系统变量 | `ndb_eventbuffer_max_alloc` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `0` |
    | 最小值 | `0` |
    | 最大值（≥ 8.0.26-ndb-8.0.26） | `9223372036854775807` |
    | 最大值 | `9223372036854775807` |
    | 最大值 | `9223372036854775807` |
    | 最大值（≤ 8.0.25-ndb-8.0.25） | `4294967295` |

    设置 NDB API 缓冲事件的最大内存量（以字节为单位）。0 表示没有限制，这也是默认值。

+   `ndb_extra_logging`

    | 命令行格式 | `ndb_extra_logging=#` |
    | --- | --- |
    | 系统变量 | `ndb_extra_logging` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `1` |
    | 最小值 | `0` |
    | 最大值 | `1` |

    此变量使得可以在 MySQL 错误日志中记录特定于`NDB`存储引擎的信息。

    当此变量设置为 0 时，写入 MySQL 错误日志的唯一与`NDB`相关的信息与事务处理有关。如果设置为大于 0 但小于 10 的值，还会记录`NDB`表模式和连接事件，以及是否正在使用冲突解决，以及其他`NDB`错误和信息。如果值设置为 10 或更高，则还会将有关`NDB`内部的信息，例如数据在集群节点之间的分发进度，写入 MySQL 错误日志。默认值为 1。

+   `ndb_force_send`

    | 命令行格式 | `--ndb-force-send[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `ndb_force_send` |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔 |
    | 默认值 | `ON` |

    立即将缓冲区发送到`NDB` ，无需等待其他线程。默认为`ON`。

+   `ndb_fully_replicated`

    | 命令行格式 | `--ndb-fully-replicated[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `ndb_fully_replicated` |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔 |
    | 默认值 | `OFF` |

    确定新的`NDB`表是否完全复制。可以使用`CREATE TABLE`或`ALTER TABLE`语句中的`COMMENT="NDB_TABLE=FULLY_REPLICATED=..."`来为单个表覆盖此设置；有关语法和其他信息，请参见第 15.1.20.12 节，“设置 NDB 注释选项”。

+   `ndb_index_stat_enable`

    | 命令行格式 | `--ndb-index-stat-enable[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `ndb_index_stat_enable` |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔 |
    | 默认值 | `ON` |

    在查询优化中使用`NDB` 索引统计信息。默认为`ON`。

    在 NDB 8.0.27 之前，使用`--ndb-index-stat-enable`设置为`OFF`启动服务器会阻止创建索引统计表。在 NDB 8.0.27 及更高版本中，无论此选项的值如何，这些表在服务器启动时都会被创建。

+   `ndb_index_stat_option`

    | 命令行格式 | `--ndb-index-stat-option=value` |
    | --- | --- |
    | 系统变量 | `ndb_index_stat_option` |
    | 范围 | 全局、会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 字符串 |
    | 默认值 | `loop_checkon=1000ms,loop_idle=1000ms,loop_busy=100ms, update_batch=1,read_batch=4,idle_batch=32,check_batch=32, check_delay=1m,delete_batch=8,clean_delay=0,error_batch=4, error_delay=1m,evict_batch=8,evict_delay=1m,cache_limit=32M, cache_lowpct=90` |

    此变量用于为 NDB 索引统计生成提供调整选项。列表由逗号分隔的名称-值对组成，此列表不得包含任何空格字符。

    在设置`ndb_index_stat_option`时未使用的选项不会更改其默认值。例如，您可以设置`ndb_index_stat_option = 'loop_idle=1000ms,cache_limit=32M'`。

    时间值可以选择性地加上`h`（小时）、`m`（分钟）或`s`（秒）后缀。毫秒值可以选择性地使用`ms`指定；不能使用`h`、`m`或`s`指定毫秒值。整数值可以加上`K`、`M`或`G`后缀。

    可以使用此变量设置的选项名称在接下来的表中显示。该表还提供了这些选项的简要描述、默认值以及（如果适用）它们的最小和最大值。

    **表 25.20 ndb_index_stat_option 选项和值**

    | 名称 | 描述 | 默认/单位 | 最小/最大 |
    | --- | --- | --- | --- |
    | `loop_enable` |  | 1000 ms | 0/4G |
    | `loop_idle` | 空闲时的休眠时间 | 1000 ms | 0/4G |
    | `loop_busy` | 等待更多工作时的休眠时间 | 100 ms | 0/4G |
    | `update_batch` |  | 1 | 0/4G |
    | `read_batch` |  | 4 | 1/4G |
    | `idle_batch` |  | 32 | 1/4G |
    | `check_batch` |  | 8 | 1/4G |
    | `check_delay` | 多久检查新统计数据 | 10 m | 1/4G |
    | `delete_batch` |  | 8 | 0/4G |
    | `clean_delay` |  | 1 m | 0/4G |
    | `error_batch` |  | 4 | 1/4G |
    | `error_delay` |  | 1 m | 1/4G |
    | `evict_batch` |  | 8 | 1/4G |
    | `evict_delay` | 从读取时间开始清除 LRU 缓存 | 1 m | 0/4G |
    | `cache_limit` | 由此**mysqld**用于缓存的索引统计的最大内存量（以字节为单位）；当超过此限制时清除缓存。 | 32 M | 0/4G |
    | `cache_lowpct` |  | 90 | 0/100 |
    | `zero_total` | 将其设置为 1 会将`ndb_index_stat_status`中所有累积计数器重置为 0。当执行此操作时，此选项值也会重置为 0。 | 0 | 0/1 |
    | 名称 | 描述 | 默认/单位 | 最小/最大 |

+   `ndb_join_pushdown`

    | 系统变量 | `ndb_join_pushdown` |
    | --- | --- |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `ON` |

    此变量控制是否将对`NDB`表的连接下推到 NDB 内核（数据节点）。以前，SQL 节点通过多次访问`NDB`来处理连接；然而，当启用`ndb_join_pushdown`时，可推送的连接将完整地发送到数据节点，其中可以在数据节点之间分发并在多个数据副本上并行执行，最终将单个合并结果返回给**mysqld**。这可以大大减少处理此类连接所需的 SQL 节点和数据节点之间的往返次数。

    默认情况下，`ndb_join_pushdown`已启用。

    **NDB 下推连接的条件。** 为了使连接可下推，必须满足以下条件：

    1.  只能比较列，并且所有要连接的列必须使用*完全*相同的数据类型。这意味着（例如）在`INT`列和`BIGINT`列上的连接也无法被下推。

        以前，诸如 `t1.a = t2.a + *`constant`*` 的表达式无法被下推。在 NDB 8.0 中取消了此限制。任何要比较的列上的任何操作的结果必须与列本身产生相同的类型。

        比较同一表中列的表达式也可以被下推。这些列（或对这些列进行的任何操作的结果）必须完全相同，包括相同的符号、长度、字符集和排序规则、精度和比例，如果适用的话。

    1.  查询引用`BLOB`或`TEXT`列不受支持。

    1.  不支持显式锁定；但是，强制执行`NDB`存储引擎的特征隐式基于行的锁定。

        这意味着使用 `FOR UPDATE` 的连接无法被下推。

    1.  为了使连接被下推，连接中的子表必须使用`ref`、`eq_ref`或`const`访问方法之一进行访问，或者这些方法的某种组合。

        外连接的子表只能使用`eq_ref`进行推送。

        如果推送连接的根是 `eq_ref` 或 `const`，只有通过 `eq_ref` 连接的子表才能被附加。（通过 `ref` 连接的表可能会成为另一个推送连接的根。）

        如果查询优化器决定对候选子表使用 `Using join cache`，那么该表无法作为子表推送。但是，它可能成为另一组推送表的根。

    1.  目前无法推送显式分区为 `[LINEAR] HASH`、`LIST` 或 `RANGE` 的表的连接。

    通过使用 `EXPLAIN` 可以查看给定连接是否可以被推送；当连接可以被推送时，可以在输出的 `Extra` 列中看到对 `pushed join` 的引用，如下例所示：

    ```sql
    mysql> EXPLAIN
     ->     SELECT e.first_name, e.last_name, t.title, d.dept_name
     ->         FROM employees e
     ->         JOIN dept_emp de ON e.emp_no=de.emp_no
     ->         JOIN departments d ON d.dept_no=de.dept_no
     ->         JOIN titles t ON e.emp_no=t.emp_no\G
    *************************** 1\. row ***************************
               id: 1
      select_type: SIMPLE
            table: d
             type: ALL
    possible_keys: PRIMARY
              key: NULL
          key_len: NULL
              ref: NULL
             rows: 9
            Extra: Parent of 4 pushed join@1
    *************************** 2\. row ***************************
               id: 1
      select_type: SIMPLE
            table: de
             type: ref
    possible_keys: PRIMARY,emp_no,dept_no
              key: dept_no
          key_len: 4
              ref: employees.d.dept_no
             rows: 5305
            Extra: Child of 'd' in pushed join@1
    *************************** 3\. row ***************************
               id: 1
      select_type: SIMPLE
            table: e
             type: eq_ref
    possible_keys: PRIMARY
              key: PRIMARY
          key_len: 4
              ref: employees.de.emp_no
             rows: 1
            Extra: Child of 'de' in pushed join@1
    *************************** 4\. row ***************************
               id: 1
      select_type: SIMPLE
            table: t
             type: ref
    possible_keys: PRIMARY,emp_no
              key: emp_no
          key_len: 4
              ref: employees.de.emp_no
             rows: 19
            Extra: Child of 'e' in pushed join@1 4 rows in set (0.00 sec)
    ```

    注意

    如果内连接的子表通过 `ref` 连接，并且结果按排序索引排序或分组，该索引无法提供排序行，这将强制写入到排序临时文件。

    还有两个关于推送连接性能的额外信息来源：

    1.  状态变量 `Ndb_pushed_queries_defined`、`Ndb_pushed_queries_dropped`、`Ndb_pushed_queries_executed` 和 `Ndb_pushed_reads`。

    1.  属于 `DBSPJ` 内核块的 `ndbinfo.counters` 表中的计数器。

+   `ndb_log_apply_status`

    | 命令行格式 | `--ndb-log-apply-status[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `ndb_log_apply_status` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    一个只读变量，显示服务器是否使用 `--ndb-log-apply-status` 选项启动。

+   `ndb_log_bin`

    | 命令行格式 | `--ndb-log-bin[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `ndb_log_bin` |
    | 范围 | 全局，会话 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值（≥ 8.0.16-ndb-8.0.16） | `OFF` |
    | 默认值（≤ 8.0.15-ndb-8.0.15） | `ON` |

    导致`NDB`表的更新写入二进制日志。如果服务器尚未使用`log_bin`启用二进制日志记录，则此变量的设置不起作用。在 NDB 8.0 中，`ndb_log_bin`默认为 0（FALSE）。

+   `ndb_log_binlog_index`

    | 命令行格式 | `--ndb-log-binlog-index[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `ndb_log_binlog_index` |
    | 作用域 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `ON` |

    导致将时代映射到二进制日志中的位置插入`ndb_binlog_index`表中。如果服务器尚未使用`log_bin`启用二进制日志记录，则设置此变量不起作用。（此外，`ndb_log_bin`不能被禁用。）`ndb_log_binlog_index`默认为`1`（`ON`）；通常，在生产环境中永远不需要更改此值。

+   `ndb_log_empty_epochs`

    | 命令行格式 | `--ndb-log-empty-epochs[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `ndb_log_empty_epochs` |
    | 作用域 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    当此变量设置为 0 时，没有更改的时代事务不会写入二进制日志，尽管在`ndb_binlog_index`中仍会为空时代写入一行。

+   `ndb_log_empty_update`

    | 命令行格式 | `--ndb-log-empty-update[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `ndb_log_empty_update` |
    | 作用域 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    当此变量设置为`ON`（`1`）时，即使启用了`log_replica_updates`或`log_slave_updates`，没有更改的更新事务也会写入二进制日志。

+   `ndb_log_exclusive_reads`

    | 命令行格式 | `--ndb-log-exclusive-reads[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `ndb_log_exclusive_reads` |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔 |
    | 默认值 | `0` |

    此变量确定是否使用独占锁记录主键读取，从而允许基于读取冲突的 NDB 集群复制冲突检测和解决。要启用这些锁，请将 `ndb_log_exclusive_reads` 的值设置为 1。默认情况下，禁用此类锁定的值为 0。

    有关更多信息，请参见 读取冲突检测和解决。

+   `ndb_log_orig`

    | 命令行格式 | `--ndb-log-orig[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `ndb_log_orig` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔 |
    | 默认值 | `OFF` |

    显示原始服务器 ID 和时代是否记录在 `ndb_binlog_index` 表中。使用 `--ndb-log-orig` 服务器选项设置。

+   `ndb_log_transaction_id`

    | 系统变量 | `ndb_log_transaction_id` |
    | --- | --- |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔 |
    | 默认值 | `OFF` |

    此只读的布尔系统变量显示一个副本 **mysqld** 是否在二进制日志中写入 NDB 事务 ID（使用 `NDB$EPOCH_TRANS()` 冲突检测来使用“主动-主动”NDB 集群复制所需）。要更改设置，请使用 `--ndb-log-transaction-id` 选项。

    `ndb_log_transaction_id` 在主流 MySQL Server 8.0 中不受支持。

    有关更多信息，请参见 第 25.7.12 节，“NDB 集群复制冲突解决”。

+   `ndb_log_transaction_compression`

    | 命令行格式 | `--ndb-log-transaction-compression` |
    | --- | --- |
    | 引入 | 8.0.31-ndb-8.0.31 |
    | 系统变量 | `ndb_log_transaction_compression` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    副本**mysqld**是否在二进制日志中写入压缩事务；仅在**mysqld**编译支持`NDB`时存在。

    使用`--binlog-transaction-compression`启动 MySQL 服务器会强制启用此变量（`ON`），这将覆盖在命令行或`my.cnf`文件中设置的`--ndb-log-transaction-compression`，如下所示：

    ```sql
    $> mysqld_safe --ndbcluster --ndb-connectstring=127.0.0.1 \
      --binlog-transaction-compression=ON --ndb-log-transaction-compression=OFF &
    [1] 27667
    $> 2022-07-07T12:29:20.459937Z mysqld_safe Logging to '/usr/local/mysql/data/myhost.err'.
    2022-07-07T12:29:20.509873Z mysqld_safe Starting mysqld daemon with databases from /usr/local/mysql/data

    $> mysql -e 'SHOW VARIABLES LIKE "%transaction_compression%"'
    +--------------------------------------------+-------+
    | Variable_name                              | Value |
    +--------------------------------------------+-------+
    | binlog_transaction_compression             | ON    |
    | binlog_transaction_compression_level_zstd  | 3     |
    | ndb_log_transaction_compression            | ON    |
    | ndb_log_transaction_compression_level_zstd | 3     |
    +--------------------------------------------+-------+
    ```

    要仅为`NDB`表禁用二进制日志事务压缩，请在启动**mysqld**后在**mysql**或其他客户端会话中将`ndb_log_transaction_compression`系统变量设置为`OFF`。

    在启动后设置`binlog_transaction_compression`变量对`ndb_log_transaction_compression`的值没有影响。

    有关二进制日志事务压缩的更多信息，例如哪些事件被压缩或未压缩以及在使用此功能时需要注意的行为更改，请参阅第 7.4.4.5 节，“二进制日志事务压缩”。

+   `ndb_log_transaction_compression_level_zstd`

    | 命令行格式 | `--ndb-log-transaction-compression-level-zstd=#` |
    | --- | --- |
    | 引入 | 8.0.31-ndb-8.0.31 |
    | 系统变量 | `ndb_log_transaction_compression_level_zstd` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `3` |
    | 最小值 | `1` |
    | 最大值 | `22` |

    如果由`ndb_log_transaction_compression`启用，则用于将压缩事务写入副本二进制日志的`ZSTD`压缩级别。如果**mysqld**未编译支持`NDB`存储引擎，则不支持。

    更多信息请参阅第 7.4.4.5 节，“二进制日志事务压缩”。

+   `ndb_metadata_check`

    | 命令行格式 | `--ndb-metadata-check[={OFF&#124;ON}]` |
    | --- | --- |
    | 引入 | 8.0.16-ndb-8.0.16 |
    | 系统变量 | `ndb_metadata_check` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `ON` |

    `NDB` 使用一个后台线程每隔 `ndb_metadata_check_interval` 秒检查一次元数据更改，与 MySQL 数据字典进行比较。可以通过将 `ndb_metadata_check` 设置为 `OFF` 来禁用此元数据更改检测线程。该线程默认情况下是启用的。

+   `ndb_metadata_check_interval`

    | 命令行格式 | `--ndb-metadata-check-interval=#` |
    | --- | --- |
    | 引入版本 | 8.0.16-ndb-8.0.16 |
    | 系统变量 | `ndb_metadata_check_interval` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `60` |
    | 最小值 | `0` |
    | 最大值 | `31536000` |
    | 单位 | 秒 |

    `NDB` 在后台运行一个元数据更改检测线程，以确定 NDB 字典相对于 MySQL 数据字典的更改。默认情况下，这些检查之间的间隔为 60 秒；可以通过设置 `ndb_metadata_check_interval` 的值来调整。要启用或禁用线程，请使用 `ndb_metadata_check`。

+   `ndb_metadata_sync`

    | 引入版本 | 8.0.19-ndb-8.0.19 |
    | --- | --- |
    | 系统变量 | `ndb_metadata_sync` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `false` |

    设置此变量会导致更改监视线程覆盖为 `ndb_metadata_check` 或 `ndb_metadata_check_interval` 设置的任何值，并进入持续更改检测期。当线程确定没有更多要检测的更改时，它会停滞���直到二进制日志线程完成所有检测到的对象的同步。然后 `ndb_metadata_sync` 设置为 `false`，更改监视线程将恢复到由 `ndb_metadata_check` 和 `ndb_metadata_check_interval` 设置确定的行为。

    在 NDB 8.0.22 及更高版本中，将此变量设置为 `true` 会导致排除对象列表被清除，将其设置为 `false` 会清除要重试的对象列表。

+   `ndb_optimized_node_selection`

    | 命令行格式 | `--ndb-optimized-node-selection=#` |
    | --- | --- |
    | 系统变量 | `ndb_optimized_node_selection` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `3` |
    | 最小值 | `0` |
    | 最大值 | `3` |

    优化节点选择有两种形式，如下所述：

    1.  SQL 节点使用接近性来确定事务协调器；也就是说，SQL 节点“最近”的数据节点被选择为事务协调器。为此，具有与 SQL 节点共享内存连接的数据节点被认为是与 SQL 节点“最近”的；接下来最接近的（按照减少接近性的顺序）是：从`localhost`的 TCP 连接，然后是从`localhost`以外的主机的 TCP 连接。

    1.  SQL 线程使用分布感知来选择数据节点。也就是说，由给定事务的第一个语句访问的集群分区所在的数据节点被用作整个事务的事务协调器。（仅当事务的第一个语句访问不超过一个集群分区时有效。）

    此选项接受整数值`0`、`1`、`2`或`3`中的一个。`3`是默认值。这些值影响节点选择如下：

    +   `0`：节点选择未经优化。在 SQL 线程继续到下一个数据节点之前，每个数据节点被用作事务协调器 8 次。

    +   `1`：接近 SQL 节点用于确定事务协调器。

    +   `2`：使用分布感知来选择事务协调器。但是，如果事务的第一个语句访问多个集群分区，则 SQL 节点会在将此选项设置为`0`时恢复到循环轮询行为。

    +   `3`：如果可以使用分布感知来确定事务协调器，则使用它；否则使用接近性来选择事务协调器。（这是默认行为。）

    接近性的确定如下：

    1.  从为`Group`参数设置的值开始（默认为 55）。

    1.  对于与其他 API 节点共享同一主机的 API 节点，将值减 1。假设`Group`的默认值，与 API 节点在同一主机上的数据节点的有效值为 54，远程数据节点为 55。

    1.  设置`ndb_data_node_neighbour`会进一步减少有效的`Group`值 50，使得这个节点被视为最近的节点。只有当所有数据节点都在与 API 节点不同的主机上，并且希望将其中一个专用于 API 节点时才需要这样做。在正常情况下，前面描述的默认调整已经足够。

    频繁更改`ndb_data_node_neighbour`是不明智的，因为这会改变集群连接的状态，从而可能破坏每个线程对新事务的选择算法，直到它稳定下来。

+   `ndb_read_backup`

    | 命令行格式 | `--ndb-read-backup[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `ndb_read_backup` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` Hint Applies | 否 |
    | 类型 | 布尔值 |
    | 默认值 (≥ 8.0.19-ndb-8.0.19) | `ON` |
    | 默认值 (≤ 8.0.18-ndb-8.0.18) | `OFF` |

    启用随后创建的任何`NDB`表的任何片段副本读取；这样做可以极大地提高表的读取性能，对写入的成本相对较小。

    如果 SQL 节点和数据节点使用相同的主机名或 IP 地址，则会自动检测到这一事实，因此首选将读取发送到同一主机。如果这些节点位于同一主机上但使用不同的 IP 地址，则可以通过将 SQL 节点上的`ndb_data_node_neighbour`的值��置为数据节点的节点 ID 来告诉 SQL 节点使用正确的数据节点。

    要为单个表启用或禁用从任何片段副本读取，您可以相应地为表设置`NDB_TABLE`选项`READ_BACKUP`，在`CREATE TABLE`或`ALTER TABLE`语句中；有关更多信息，请参见第 15.1.20.12 节，“设置 NDB 注释选项”。

+   `ndb_recv_thread_activation_threshold`

    | 命令行格式 | `--ndb-recv-thread-activation-threshold=#` |
    | --- | --- |
    | 系统变量 | `ndb_recv_thread_activation_threshold` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` Hint Applies | 否 |
    | 类型 | 整数 |
    | 默认值 | `8` |
    | 最小值 | `0 (MIN_ACTIVATION_THRESHOLD)` |
    | 最大值 | `16 (MAX_ACTIVATION_THRESHOLD)` |

    当达到此数量的并发活动线程时，接收线程接管集群连接的轮询。

    此变量的作用范围是全局的。它也可以在启动时设置。

+   `ndb_recv_thread_cpu_mask`

    | 命令行格式 | `--ndb-recv-thread-cpu-mask=mask` |
    | --- | --- |
    | 系统变量 | `ndb_recv_thread_cpu_mask` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 位图 |
    | 默认值 | `[empty]` |

    用于将接收线程锁定到特定 CPU 的 CPU 掩码。这是指定为十六进制位掩码的。例如，`0x33` 表示每个接收线程使用一个 CPU。空字符串是默认值；将 `ndb_recv_thread_cpu_mask` 设置为此值会删除先前设置的任何接收线程锁定。

    此变量的作用范围是全局的。也可以在启动时设置。

+   `ndb_report_thresh_binlog_epoch_slip`

    | 命令行格式 | `--ndb-report-thresh-binlog-epoch-slip=#` |
    | --- | --- |
    | 系统变量 | `ndb_report_thresh_binlog_epoch_slip` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `10` |
    | 最小值 | `0` |
    | 最大值 | `256` |

    这代表事件缓冲区中完全缓冲的时代数量的阈值，但尚未被二进制日志注入器线程消耗。当超过这种滑动（滞后）程度时，将报告事件缓冲区状态消息，原因为`BUFFERED_EPOCHS_OVER_THRESHOLD`（请参阅第 25.6.2.3 节，“集群日志中的事件缓冲区报告”）。当从数据节点接收并完全缓冲一个时代时，滑动会增加；当二进制日志注入器线程消耗一个时代时，滑动会减少。仅当使用 NDB API 中的 `Ndb::setEventBufferQueueEmptyEpoch()` 方法启用时，空时代才会被缓冲和排队，因此仅在此计算中包括。

+   `ndb_report_thresh_binlog_mem_usage`

    | 命令行格式 | `--ndb-report-thresh-binlog-mem-usage=#` |
    | --- | --- |
    | 系统变量 | `ndb_report_thresh_binlog_mem_usage` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `10` |
    | 最小值 | `0` |
    | 最大值 | `10` |

    这是在报告二进制日志状态之前剩余可用内存百分比的阈值。例如，默认值为`10`，这意味着如果用于从数据节点接收二进制日志数据的可用内存量低于 10%，则会向集群日志发送状态消息。

+   `ndb_row_checksum`

    | 系统变量 | `ndb_row_checksum` |
    | --- | --- |
    | 作用范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `1` |
    | 最小值 | `0` |
    | 最大值 | `1` |

    传统上，`NDB` 创建带有行校验和的表，这会检查硬件问题，但会牺牲性能。将 `ndb_row_checksum` 设置为 0 意味着新建或更改表时不使用行校验和，这对所有类型的查询性能都有显著影响。此变量默认设置为 1，以提供向后兼容的行为。

+   `ndb_schema_dist_lock_wait_timeout`

    | 命令行格式 | `--ndb-schema-dist-lock-wait-timeout=value` |
    | --- | --- |
    | 引入版本 | 8.0.18-ndb-8.0.18 |
    | 系统变量 | `ndb_schema_dist_lock_wait_timeout` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `30` |
    | 最小值 | `0` |
    | 最大值 | `1200` |
    | 单位 | 秒 |

    在模式分发期间等待的秒数，以获取每个 SQL 节点上获取的元数据锁，以便更改其本地数据字典以反映 DDL 语句更改。在经过此时间后，将返回警告，指出给定 SQL 节点的数据字典未更新。这样可以避免二进制日志线程在处理模式操作时等待过长时间。

+   `ndb_schema_dist_timeout`

    | 命令行格式 | `--ndb-schema-dist-timeout=value` |
    | --- | --- |
    | 引入版本 | 8.0.16-ndb-8.0.16 |
    | 系统变量 | `ndb_schema_dist_timeout` |
    | 作用范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `120` |
    | 最小值 | `5` |
    | 最大值 | `1200` |
    | 单位 | 秒 |

    在模式分发期间检测超时之前等待的秒数。这可能表明其他 SQL 节点正在经历过多的活动，或者它们在某种程度上被阻止获取必要资源。

+   `ndb_schema_dist_upgrade_allowed`

    | 命令行格式 | `--ndb-schema-dist-upgrade-allowed=value` |
    | --- | --- |
    | 引入版本 | 8.0.17-ndb-8.0.17 |
    | 系统变量 | `ndb_schema_dist_upgrade_allowed` |
    | 作用范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `true` |

    允许在连接到`NDB`时升级模式分发表。当为真（默认值），此更改被推迟，直到所有 SQL 节点已升级到相同版本的 NDB Cluster 软件。

    注意

    在升级执行之前，模式分发的性能可能会有所下降。

+   `ndb_show_foreign_key_mock_tables`

    | 命令行格式 | `--ndb-show-foreign-key-mock-tables[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `ndb_show_foreign_key_mock_tables` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    显示`NDB`用于支持`foreign_key_checks=0`的模拟表。启用此功能时，在创建和删除表时会显示额外警告。表的真实（内部）名称可以在`SHOW CREATE TABLE`的输出中看到。

+   `ndb_slave_conflict_role`

    | 命令行格式 | `--ndb-slave-conflict-role=value` |
    | --- | --- |
    | 已弃用 | 8.0.23-ndb-8.0.23 |
    | 系统变量 | `ndb_slave_conflict_role` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 枚举 |
    | 默认值 | `NONE` |
    | 有效值 | `NONE``PRIMARY``SECONDARY``PASS` |

    在 NDB 8.0.23 中已弃用，并可能在将来的版本中被移除。请改用`ndb_conflict_role`。

+   `ndb_table_no_logging`

    | 系统变量 | `ndb_table_no_logging` |
    | --- | --- |
    | 作用范围 | 会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    当此变量设置为`ON`或`1`时，导致使用`ENGINE NDB`创建或更改的所有表都是非记录日志的；也就是说，此表的数据更改不会写入重做日志或写入磁盘检查点，就好像使用`CREATE TABLE`或`ALTER TABLE`时使用`NOLOGGING`选项创建或更改表一样。

    有关非记录`NDB`表的更多信息，请参阅 NDB_TABLE 选项。

    `ndb_table_no_logging`对`NDB`表模式文件的创建没有影响；要抑制这些文件，请改用`ndb_table_temporary`。

+   `ndb_table_temporary`

    | 系统变量 | `ndb_table_temporary` |
    | --- | --- |
    | 作用范围 | 会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    当设置为`ON`或`1`时，此变量导致`NDB`表不写入磁盘：这意味着不会创建表模式文件，也不会记录表。

    注意

    设置此变量目前没有效果。这是一个已知问题；请参阅 Bug #34036。

+   `ndb_use_copying_alter_table`

    | 系统变量 | `ndb_use_copying_alter_table` |
    | --- | --- |
    | 作用范围 | 全局, 会话 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |

    强制`NDB`在在线`ALTER TABLE`操作出现问题时使用表的复制。默认值为`OFF`。

+   `ndb_use_exact_count`

    | 系统变量 | `ndb_use_exact_count` |
    | --- | --- |
    | 作用范围 | 全局, 会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    强制`NDB`在`SELECT COUNT(*)`查询规划期间使用记录计数以加快此类查询的速度。默认值为`OFF`，这样可以加快整体查询速度。

+   `ndb_use_transactions`

    | 命令行格式 | `--ndb-use-transactions[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `ndb_use_transactions` |
    | 作用范围 | 全局, 会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `ON` |

    通过将此变量的值设置为`OFF`，您可以禁用`NDB`事务支持。一般情况下不建议这样做，尽管在某些情况下可能会有用，比如在导入一个或多个大事务的转储文件时，可以在给定的客户端会话中禁用事务支持；这样可以将多行插入分批执行，而不是作为单个事务。在这种情况下，一旦导入完成，您应该将此会话的变量值重置为`ON`，或者直接终止会话。

+   `ndb_version`

    | 系统变量 | `ndb_version` |
    | --- | --- |
    | 作用范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 字符串 |
    | 默认值 |  |

    `NDB`引擎版本，作为一个复合整数。

+   `ndb_version_string`

    | 系统变量 | `ndb_version_string` |
    | --- | --- |
    | 作用范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 字符串 |
    | 默认值 |  |

    `NDB`引擎版本以`ndb-*`x.y.z`*`格式。

+   `replica_allow_batching`

    | 命令行格式 | `--replica-allow-batching[={OFF&#124;ON}]` |
    | --- | --- |
    | 引入版本 | 8.0.26-ndb-8.0.26 |
    | 系统变量 | `replica_allow_batching` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 (≥ 8.0.30-ndb-8.0.30) | `ON` |
    | 默认值 (≤ 8.0.29-ndb-8.0.29) | `OFF` |

    NDB 集群复制品上是否启用了批量更新。从 NDB 8.0.26 开始，您应该使用`replica_allow_batching`来替代在该版本中已弃用的`slave_allow_batching`。

    在复制品上允许批量更新极大地提高了性能，特别是在复制`TEXT`、`BLOB`和`JSON`列时。因此，在 NDB 8.0.30 及更高版本中，默认情况下启用了`replica_allow_batching`。

    设置此变量仅在使用`NDB`存储引擎的复制时才会生效；在 MySQL Server 8.0 中，它存在但不起作用。有关更多信息，请参见第 25.7.6 节，“启动 NDB 集群复制（单个复制通道）”")。

+   `ndb_replica_batch_size`

    | 命令行格式 | `--ndb-replica-batch-size=#` |
    | --- | --- |
    | 引入版本 | 8.0.30-ndb-8.0.30 |
    | 系统变量 | `ndb_replica_batch_size` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `2097152` |
    | 最小值 | `0` |
    | 最大值 | `2147483648` |
    | 单位 | 字节 |

    确定复制应用程序线程使用的批量大小（以字节为单位）。在 NDB 8.0.30 及更高版本中，设置��变量而不是`--ndb-batch-size`选项，将此设置应用于副本，不包括任何其他会话。

    如果未设置此变量（默认为 2 MB），其有效值为`--ndb-batch-size`的值和 2 MB 中较大的一个。

+   `ndb_replica_blob_write_batch_bytes`

    | 命令行格式 | `--ndb-replica-blob-write-batch-bytes=#` |
    | --- | --- |
    | 引入版本 | 8.0.30-ndb-8.0.30 |
    | 系统变量 | `ndb_replica_blob_write_batch_bytes` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `2097152` |
    | 最小值 | `0` |
    | 最大值 | `2147483648` |
    | 单位 | 字节 |

    控制复制应用程序线程用于 blob 数据的批量写入大小。

    从 NDB 8.0.30 开始，您应该设置这个变量，而不是`--ndb-blob-write-batch-bytes`选项来控制副本上的 blob 批量写入大小，不包括任何其他会话。这样做的原因是，当未设置`ndb_replica_blob_write_batch_bytes`时，有效的 blob 批量大小（即，blob 列待写入的最大字节数）由`--ndb-blob-write-batch-bytes`的值和 2 MB（`ndb_replica_blob_write_batch_bytes`的默认值）中较大的一个确定。

    将`ndb_replica_blob_write_batch_bytes`设置为 0 意味着`NDB`在副本上对 blob 批量写入大小不设限。

+   `server_id_bits`

    | 命令行格式 | `--server-id-bits=#` |
    | --- | --- |
    | 系统变量 | `server_id_bits` |
    | 作用范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `32` |
    | 最小值 | `7` |
    | 最大值 | `32` |

    此变量指示在 32 位 `server_id` 中，实际标识服务器的最低有效位数。指示服务器实际上由少于 32 位标识使得一些剩余位可以用于其他目的，例如通过使用 NDB API 的事件 API 生成的用户数据存储在 `OperationOptions` 结构的 `AnyValue` 中（NDB 集群使用 `AnyValue` 存储服务器 ID）。

    在提取用于检测复制循环等目的的 `server_id` 的有效服务器 ID 时，服务器会忽略剩余的位。在决定是否应根据服务器 ID 忽略事件时，I/O 和 SQL 线程中使用 `server_id_bits` 变量来屏蔽 `server_id` 的任何无关位。

    此数据可以通过 **mysqlbinlog** 从二进制日志中读取，前提是它以自己的 `server_id_bits` 变量设置为 32（默认值）运行。

    如果 `server_id` 的值大于或等于 2 的 `server_id_bits` 次方；否则，**mysqld** 拒绝启动。

    此系统变量仅受 NDB 集群支持。标准 MySQL 8.0 服务器不支持。

+   `slave_allow_batching`

    | 命令行格式 | `--slave-allow-batching[={OFF&#124;ON}]` |
    | --- | --- |
    | 已弃用 | 8.0.26-ndb-8.0.26 |
    | 系统变量 | `slave_allow_batching` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值（≥ 8.0.30-ndb-8.0.30） | `ON` |
    | 默认值（≤ 8.0.29-ndb-8.0.29） | `OFF` |

    NDB 集群副本是否启用批量更新。从 NDB 8.0.26 开始，此变量已弃用，应改用 `replica_allow_batching`。

    在副本上允许批量更新可以极大地提高性能，特别是在复制`TEXT`、`BLOB`和`JSON`列时。因此，在 NDB 8.0.30 及更高版本中，默认情况下`replica_allow_batching`为`ON`。从 NDB 8.0.30 开始，每当将此变量设置为`OFF`时都会发出警告。

    设置此变量仅在使用`NDB`存储引擎的复制时才会生效；在 MySQL Server 8.0 中，它存在但不起作用。更多信息，请参见第 25.7.6 节，“启动 NDB 集群复制（单一复制通道）”")。

+   `transaction_allow_batching`

    | 系统变量 | `transaction_allow_batching` |
    | --- | --- |
    | 作用范围 | 会话 |
    | 动态 | 是 |
    | `SET_VAR` Hint Applies | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    当设置为`1`或`ON`时，此变量允许在同一事务中批处理语句。要使用此变量，必须先通过将`autocommit`设置为`0`或`OFF`来禁用自动提交；否则，设置`transaction_allow_batching`不会生效。

    在执行仅进行写操作的事务时使用此变量是安全的，因为启用它可能导致从“之前”图像中读取。在发出`SELECT`之前，应确保任何待处理的事务已提交（如果需要，使用显式的`COMMIT`）。

    重要

    `transaction_allow_batching`在同一事务中的给定语句的效果取决于先前语句的结果时不应使用。

    当前仅支持 NDB Cluster 的此变量。

下面列表中的系统变量都与`ndbinfo`信息数据库相关。

+   `ndbinfo_database`

    | 系统变量 | `ndbinfo_database` |
    | --- | --- |
    | 作用范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` Hint Applies | 否 |
    | 类型 | 字符串 |
    | 默认值 | `ndbinfo` |

    显示用于`NDB`信息数据库的名称；默认值为`ndbinfo`。这是一个只读变量，其值在编译时确定。

+   `ndbinfo_max_bytes`

    | 命令行格式 | `--ndbinfo-max-bytes=#` |
    | --- | --- |
    | 系统变量 | `ndbinfo_max_bytes` |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `0` |
    | 最小值 | `0` |
    | 最大值 | `65535` |

    仅用于测试和调试。

+   `ndbinfo_max_rows`

    | 命令行格式 | `--ndbinfo-max-rows=#` |
    | --- | --- |
    | 系统变量 | `ndbinfo_max_rows` |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `10` |
    | 最小值 | `1` |
    | 最大值 | `256` |

    仅用于测试和调试。

+   `ndbinfo_offline`

    | 系统变量 | `ndbinfo_offline` |
    | --- | --- |
    | 范围 | 全��� |  |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    将`ndbinfo`数据库置于离线模式，即使表和视图实际上不存在，或者存在但在`NDB`中具有不同的定义时也可以打开这些表（或视图）。不会从这些表（或视图）返回任何行。

+   `ndbinfo_show_hidden`

    | 命令行格式 | `--ndbinfo-show-hidden[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `ndbinfo_show_hidden` |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |
    | 有效值 | `ON``OFF` |

    是否在**mysql**客户端中显示`ndbinfo`数据库的底层内部表。默认值为`OFF`。

    注意

    当启用`ndbinfo_show_hidden`时，内部表仅在`ndbinfo`数据库中显示；无论变量设置如何，它们都不会在`TABLES`或其他`INFORMATION_SCHEMA`表中可见。

+   `ndbinfo_table_prefix`

    | 系统变量 | `ndbinfo_table_prefix` |
    | --- | --- |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 字符串 |
    | 默认值 | `ndb$` |

    用于命名 ndbinfo 数据库基本表的前缀（通常隐藏，除非通过设置`ndbinfo_show_hidden`公开）。这是一个只读变量，默认值为`ndb$`；前缀本身在编译时确定。

+   `ndbinfo_version`

    | 系统变量 | `ndbinfo_version` |
    | --- | --- |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 字符串 |
    | 默认值 |  |

    显示正在使用的`ndbinfo`引擎的版本；只读。

##### 25.4.3.9.3 NDB Cluster 状态变量

本节提供了与 NDB Cluster 和`NDB`存储引擎相关的 MySQL 服务器状态变量的详细信息。有关不特定于 NDB Cluster 的状态变量以及使用状态变量的一般信息，请参见第 7.1.10 节，“服务器状态变量”。

+   `Handler_discover`

    MySQL 服务器可以询问`NDBCLUSTER`存储引擎是否知道具有给定名称的表。这称为发现。`Handler_discover`指示使用此机制发现表的次数。

+   `Ndb_api_adaptive_send_deferred_count`

    实际未发送的自适应发送调用次数。

    更多信息，请参见第 25.6.15 节，“NDB API 统计计数器和变量”。

+   `Ndb_api_adaptive_send_deferred_count_session`

    实际未发送的自适应发送调用次数。

    更多信息，请参阅第 25.6.15 节，“NDB API 统计计数器和变量”。

+   `Ndb_api_adaptive_send_deferred_count_replica`

    该副本实际未发送的自适应发送调用次数。

    更多信息，请参阅第 25.6.15 节，“NDB API 统计计数器和变量”。

+   `Ndb_api_adaptive_send_deferred_count_slave`

    注意

    在 NDB 8.0.23 中已弃用；请改用`Ndb_api_adaptive_send_deferred_count_replica`。

    该副本实际未发送的自适应发送调用次数。

    更多信息，请参阅第 25.6.15 节，“NDB API 统计计数器和变量”。

+   `Ndb_api_adaptive_send_forced_count`

    由此 MySQL 服务器（SQL 节点）发送的强制发送调用的自适应发送次数。

    更多信息，请参阅第 25.6.15 节，“NDB API 统计计数器和变量”。

+   `Ndb_api_adaptive_send_forced_count_session`

    在此客户端会话中发送的强制发送调用的自适应发送次数。

    更多信息，请参阅第 25.6.15 节，“NDB API 统计计数器和变量”。

+   `Ndb_api_adaptive_send_forced_count_replica`

    该副本发送的强制发送调用的自适应发送次数。

    更多信息，请参阅第 25.6.15 节，“NDB API 统计计数器和变量”。

+   `Ndb_api_adaptive_send_forced_count_slave`

    注意

    在 NDB 8.0.23 中已弃用；请改用`Ndb_api_adaptive_send_forced_count_replica`。

    该副本发送的强制发送调用的自适应发送次数。

    更多信息，请参阅第 25.6.15 节，“NDB API 统计计数器和变量”。

+   `Ndb_api_adaptive_send_unforced_count`

    由此 MySQL 服务器（SQL 节点）发送的无强制发送的自适应发送调用次数。

    有关更多信息，请参见第 25.6.15 节，“NDB API 统计计数器和变量”。

+   `Ndb_api_adaptive_send_unforced_count_session`

    在此客户端会话中发送的无强制发送的自适应发送调用次数。

    有关更多信息，请参见第 25.6.15 节，“NDB API 统计计数器和变量”。

+   `Ndb_api_adaptive_send_unforced_count_replica`

    由此副本发送的无强制发送的自适应发送调用次数。

    有关更多信息，请参见第 25.6.15 节，“NDB API 统计计数器和变量”。

+   `Ndb_api_adaptive_send_unforced_count_slave`

    注意

    在 NDB 8.0.23 中已弃用；请改用`Ndb_api_adaptive_send_unforced_count_replica`。

    由此副本发送的无强制发送的自适应发送调用次数。

    有关更多信息，请参见第 25.6.15 节，“NDB API 统计计数器和变量”。

+   `Ndb_api_bytes_sent_count_session`

    在此客户端会话中发送到数据节点的数据量（以字节为单位）。

    尽管可以使用`SHOW GLOBAL STATUS`或`SHOW SESSION STATUS`读取此变量，但它仅与当前会话相关，并不受此**mysqld**的任何其他客户端影响。

    有关更多信息，请参见第 25.6.15 节，“NDB API 统计计数器和变量”。

+   `Ndb_api_bytes_sent_count_replica`

    由此副本发送到数据节点的数据量（以字节为单位）。

    尽管可以使用`SHOW GLOBAL STATUS`或`SHOW SESSION STATUS`读取此变量，但其范围实际上是全局的。 如果此 MySQL 服务器不充当副本，或不使用 NDB 表，则此值始终为 0。

    更多信息，请参见第 25.6.15 节，“NDB API 统计计数器和变量”。

+   `Ndb_api_bytes_sent_count_slave`

    注意

    在 NDB 8.0.23 中已弃用；请改用`Ndb_api_bytes_sent_count_replica`。

    发送到数据节点的数据量（以字节为单位）。

    虽然这个变量可以通过`SHOW GLOBAL STATUS`或`SHOW SESSION STATUS`来读取，但实际上它在全局范围内有效。如果这个 MySQL 服务器不充当副本，或者不使用 NDB 表，这个值始终为 0。

    更多信息，请参见第 25.6.15 节，“NDB API 统计计数器和变量”。

+   `Ndb_api_bytes_sent_count`

    由此 MySQL 服务器（SQL 节点）发送到数据节点的数据量（以字节为单位）。

    虽然这个变量可以通过`SHOW GLOBAL STATUS`或`SHOW SESSION STATUS`来读取，但实际上它在全局范围内有效。

    更多信息，请参见第 25.6.15 节，“NDB API 统计计数器和变量”。

+   `Ndb_api_bytes_received_count_session`

    从数据节点在此客户端会话中接收到的数据量（以字节为单位）。

    虽然这个变量可以通过`SHOW GLOBAL STATUS`或`SHOW SESSION STATUS`来读取，但它只与当前会话相关，并不受这个**mysqld**的任何其他客户端影响。

    更多信息，请参见第 25.6.15 节，“NDB API 统计计数器和变量”。

+   `Ndb_api_bytes_received_count_replica`

    从数据节点接收到的数据量（以字节为单位）。

    虽然这个变量可以通过`SHOW GLOBAL STATUS`或`SHOW SESSION STATUS`来读取，但实际上它在全局范围内有效。如果这个 MySQL 服务器不充当副本，或者不使用 NDB 表，这个值始终为 0。

    更多信息，请参见第 25.6.15 节，“NDB API 统计计数器和变量”。

+   `Ndb_api_bytes_received_count_slave`

    注意

    在 NDB 8.0.23 中已弃用；请改用`Ndb_api_bytes_received_count_replica`。

    此副本从数据节点接收的数据量（以字节为单位）。

    尽管可以使用`SHOW GLOBAL STATUS`或`SHOW SESSION STATUS`读取此变量，但其实际范围是全局的。如果此 MySQL 服务器不充当副本，或者不使用 NDB 表，此值始终为 0。

    更多信息，请参见第 25.6.15 节，“NDB API 统计计数器和变量”。

+   `Ndb_api_bytes_received_count`

    此 MySQL 服务器（SQL 节点）从数据节点接收的数据量（以字节为单位）。

    尽管可以使用`SHOW GLOBAL STATUS`或`SHOW SESSION STATUS`读取此变量，但其实际范围是全局的。

    更多信息，请参见第 25.6.15 节，“NDB API 统计计数器和变量”。

+   `Ndb_api_event_data_count_injector`

    NDB 二进制日志注入器线程接收的行更改事件数量。

    尽管可以使用`SHOW GLOBAL STATUS`或`SHOW SESSION STATUS`读取此变量，但其实际范围是全局的。

    更多信息，请参见第 25.6.15 节，“NDB API 统计计数器和变量”。

+   `Ndb_api_event_data_count`

    此 MySQL 服务器（SQL 节点）接收的行更改事件数量。

    尽管可以使用`SHOW GLOBAL STATUS`或`SHOW SESSION STATUS`读取此变量，但其实际范围是全局的。

    更多信息，请参见第 25.6.15 节，“NDB API 统计计数器和变量”。

+   `Ndb_api_event_nondata_count_injector`

    NDB 二进制日志注入线程接收的除行更改事件之外的事件数量。

    尽管可以使用`SHOW GLOBAL STATUS`或`SHOW SESSION STATUS`读取此变量，但其实际范围是全局的。

    查看更多信息，请参见第 25.6.15 节，“NDB API 统计计数器和变量”。

+   `Ndb_api_event_nondata_count`

    MySQL 服务器（SQL 节点）接收的除行更改事件之外的事件数量。

    尽管可以使用`SHOW GLOBAL STATUS`或`SHOW SESSION STATUS`读取此变量，但其实际范围是全局的。

    查看更多信息，请参见第 25.6.15 节，“NDB API 统计计数器和变量”。

+   `Ndb_api_event_bytes_count_injector`

    NDB 二进制日志注入线程接收的事件字节数。

    尽管可以使用`SHOW GLOBAL STATUS`或`SHOW SESSION STATUS`读取此变量，但其实际范围是全局的。

    查看更多信息，请参见第 25.6.15 节，“NDB API 统计计数器和变量”。

+   `Ndb_api_event_bytes_count`

    MySQL 服务器（SQL 节点）接收的事件字节数。

    尽管可以使用`SHOW GLOBAL STATUS`或`SHOW SESSION STATUS`读取此变量，但其实际范围是全局的。

    查看更多信息，请参见第 25.6.15 节，“NDB API 统计计数器和变量”。

+   `Ndb_api_pk_op_count_session`

    基于或使用主键的客户端会话中的操作数量。这包括对 blob 表的操作、隐式解锁操作、自增操作，以及用户可见的主键操作。

    尽管可以使用`SHOW GLOBAL STATUS`或`SHOW SESSION STATUS`读取此变量，但它仅与当前会话相关，并不受此**mysqld**的任何其他客户端影响。

    更多信息，请参见第 25.6.15 节，“NDB API 统计计数器和变量”。

+   `Ndb_api_pk_op_count_replica`

    这个复制品基于或使用主键的操作次数。这包括对 blob 表的操作、隐式解锁操作、自增操作，以及用户可见的主键操作。

    尽管可以使用`SHOW GLOBAL STATUS`或`SHOW SESSION STATUS`读取此变量，但其实际范围是全局的。如果此 MySQL 服务器不充当复制品，或不使用 NDB 表，则此值始终为 0。

    更多信息，请参见第 25.6.15 节，“NDB API 统计计数器和变量”。

+   `Ndb_api_pk_op_count_slave`

    注意

    在 NDB 8.0.23 中已弃用；请使用`Ndb_api_pk_op_count_replica`代替。

    这个复制品基于或使用主键的操作次数。这包括对 blob 表的操作、隐式解锁操作、自增操作，以及用户可见的主键操作。

    尽管可以使用`SHOW GLOBAL STATUS`或`SHOW SESSION STATUS`读取此变量，但其实际范围是全局的。如果此 MySQL 服务器不充当复制品，或不使用 NDB 表，则此值始终为 0。

    更多信息，请参见第 25.6.15 节，“NDB API 统计计数器和变量”。

+   `Ndb_api_pk_op_count`

    这个 MySQL 服务器（SQL 节点）基于或使用主键的操作次数。这包括对 blob 表的操作、隐式解锁操作、自增操作，以及用户可见的主键操作。

    尽管可以使用`SHOW GLOBAL STATUS`或`SHOW SESSION STATUS`读取此变量，但其实际范围是全局的。

    更多信息，请参见第 25.6.15 节，“NDB API 统计计数器和变量”。

+   `Ndb_api_pruned_scan_count_session`

    此客户端会话中已被修剪为单个分区的扫描次数。

    尽管可以使用`SHOW GLOBAL STATUS`或`SHOW SESSION STATUS`读取此变量，但它与当前会话相关，不受此**mysqld**的任何其他客户端影响。

    更多信息，请参见第 25.6.15 节，“NDB API 统计计数器和变量”。

+   `Ndb_api_pruned_scan_count_replica`

    此副本进行的已被修剪为单个分区的扫描次数。

    尽管可以使用`SHOW GLOBAL STATUS`或`SHOW SESSION STATUS`读取此变量，但其实际范围是全局的。如果此 MySQL 服务器不充当副本，或不使用 NDB 表，此值始终为 0。

    更多信息，请参见第 25.6.15 节，“NDB API 统计计数器和变量”。

+   `Ndb_api_pruned_scan_count_slave`

    注意

    在 NDB 8.0.23 中已弃用；请改用`Ndb_api_pruned_scan_count_replica`。

    此副本进行的已被修剪为单个分区的扫描次数。

    尽管可以使用`SHOW GLOBAL STATUS`或`SHOW SESSION STATUS`读取此变量，但其实际范围���全局的。如果此 MySQL 服务器不充当副本，或不使用 NDB 表，此值始终为 0。

    更多信息，请参见第 25.6.15 节，“NDB API 统计计数器和变量”。

+   `Ndb_api_pruned_scan_count`

    此 MySQL 服务器（SQL 节点）进行的已被修剪为单个分区的扫描次数。

    尽管可以使用`SHOW GLOBAL STATUS`或`SHOW SESSION STATUS`读取此变量，但其实际范围是全局的。

    更多信息，请参见第 25.6.15 节，“NDB API 统计计数器和变量”。

+   `Ndb_api_range_scan_count_session`

    在此客户端会话中启动的范围扫描次数。

    尽管可以使用`SHOW GLOBAL STATUS`或`SHOW SESSION STATUS`读取此变量，但它仅与当前会话相关，并不受此**mysqld**的任何其他客户端影响。

    更多信息，请参见第 25.6.15 节，“NDB API 统计计数器和变量”。

+   `Ndb_api_range_scan_count_replica`

    由此副本启动的范围扫描次数。

    尽管可以使用`SHOW GLOBAL STATUS`或`SHOW SESSION STATUS`读取此变量，但其实际上是全局范围的。如果此 MySQL 服务器不充当副本，或不使用 NDB 表，则此值始终为 0。

    更多信息，请参见第 25.6.15 节，“NDB API 统计计数器和变量”。

+   `Ndb_api_range_scan_count_slave`

    注意

    在 NDB 8.0.23 中已弃用；请改用`Ndb_api_range_scan_count_replica`。

    由此副本启动的范围扫描次数。

    尽管可以使用`SHOW GLOBAL STATUS`或`SHOW SESSION STATUS`读取此变量，但其实际上是全局范围的。如果此 MySQL 服务器不充当副本，或不使用 NDB 表，则此值始终为 0。

    更多信息，请参见第 25.6.15 节，“NDB API 统计计���器和变量”。

+   `Ndb_api_range_scan_count`

    由此 MySQL 服务器（SQL 节点）启动的范围扫描次数。

    尽管可以使用`SHOW GLOBAL STATUS`或`SHOW SESSION STATUS`读取此变量，但其实际上是全局范围的。

    更多信息，请参见第 25.6.15 节，“NDB API 统计计数器和变量”。

+   `Ndb_api_read_row_count_session`

    在此客户端会话中已读取的总行数。这包括此客户端会话中通过任何主键、唯一键或扫描操作读取的所有行。

    尽管此变量可以使用`SHOW GLOBAL STATUS`或`SHOW SESSION STATUS`读取，但它仅与当前会话相关，并不受此**mysqld**的任何其他客户端影响。

    更多信息，请参见第 25.6.15 节，“NDB API 统计计数器和变量”。

+   `Ndb_api_read_row_count_replica`

    此副本已读取的总行数。这包括此副本通过任何主键、唯一键或扫描操作读取的所有行。

    尽管此变量可以使用`SHOW GLOBAL STATUS`或`SHOW SESSION STATUS`读取，但其实质上是全局范围的。如果此 MySQL 服务器不充当副本，或不使用 NDB 表，此值始终为 0。

    更多信息，请参见第 25.6.15 节，“NDB API 统计计数器和变量”。

+   `Ndb_api_read_row_count_slave`

    注意

    在 NDB 8.0.23 中已弃用；请改用`Ndb_api_read_row_count_replica`。

    此副本已读取的总行数。这包括此副本通过任何主键、唯一键或扫描操作读取的所有行。

    尽管此变量可以使用`SHOW GLOBAL STATUS`或`SHOW SESSION STATUS`读取，但其实质上是全局范围的。如果此 MySQL 服务器不充当副本，或不使用 NDB 表，此值始终为 0。

    更多信息，请参见第 25.6.15 节，“NDB API 统计计数器和变量”。

+   `Ndb_api_read_row_count`

    此 MySQL 服务器（SQL 节点）已读取的总行数。这包括此 MySQL 服务器（SQL 节点）通过任何主键、唯一键或扫描操作读取的所有行。

    你应该意识到，对于由`SELECT` `COUNT(*)`查询读取的行，这个值可能不完全准确，因为在这种情况下，MySQL 服务器实际上读取伪行，形式为`[*`表片段 ID`*]:[*`片段中的行数`*]`，并对表中所有片段的行进行求和，以推导出所有行的估计计数。`Ndb_api_read_row_count`使用这个估计值，而不是表中实际的行数。

    尽管可以使用`SHOW GLOBAL STATUS`或`SHOW SESSION STATUS`读取此变量，但其实质上是全局范围的。

    更多信息，请参见第 25.6.15 节，“NDB API 统计计数器和变量”。

+   `Ndb_api_scan_batch_count_session`

    此客户端会话中接收的行批次数。1 批次定义为来自单个片段的扫描结果集。

    尽管可以使用`SHOW GLOBAL STATUS`或`SHOW SESSION STATUS`读取此变量，但它仅与当前会话相关，并不受此**mysqld**的任何其他客户端影响。

    更多信息，请参见第 25.6.15 节，“NDB API 统计计数器和变量”。

+   `Ndb_api_scan_batch_count_replica`

    此副本接收的行批次数。1 批次定义为来自单个片段的扫描结果集。

    尽管可以使用`SHOW GLOBAL STATUS`或`SHOW SESSION STATUS`读取此变量，但其实质上是全局范围的。如果此 MySQL 服务器不充当副本，或不使用 NDB 表，此值始终为 0。

    更多信息，请参见第 25.6.15 节，“NDB API 统计计数器和变量”。

+   `Ndb_api_scan_batch_count_slave`

    注意

    在 NDB 8.0.23 中已弃用；请改用`Ndb_api_scan_batch_count_replica`。

    此副本接收的行批次数。1 批次定义为来自单个片段的扫描结果集。

    虽然可以使用`SHOW GLOBAL STATUS`或`SHOW SESSION STATUS`读取此变量，但其实际范围是全局的。如果此 MySQL 服务器不充当副本，或不使用 NDB 表，则此值始终为 0。

    更多信息，请参见第 25.6.15 节，“NDB API 统计计数器和变量”。

+   `Ndb_api_scan_batch_count`

    MySQL 服务器（SQL 节点）接收的行批次数。1 批次定义为来自单个片段的扫描结果集。

    虽然可以使用`SHOW GLOBAL STATUS`或`SHOW SESSION STATUS`读取此变量，但其实际范围是全局的。

    更多信息，请参见第 25.6.15 节，“NDB API 统计计数器和变量”。

+   `Ndb_api_table_scan_count_session`

    在此客户端会话中已启动的表扫描次数，包括对内部表的扫描。

    虽然可以使用`SHOW GLOBAL STATUS`或`SHOW SESSION STATUS`读取此变量，但它仅与当前会话相关，并不受此**mysqld**的任何其他客户端影响。

    更多信息，请参见第 25.6.15 节，“NDB API 统计计数器和变量”。

+   `Ndb_api_table_scan_count_replica`

    此副本已启动的表扫描次数，包括对内部表的扫描。

    虽然可以使用`SHOW GLOBAL STATUS`或`SHOW SESSION STATUS`读取此变量，但其实际范围是全局的。如果此 MySQL 服务器不充当副本，或不使用 NDB 表，则此值始终为 0。

    更多信息，请参见第 25.6.15 节，“NDB API 统计计数器和变量”。

+   `Ndb_api_table_scan_count_slave`

    注意

    在 NDB 8.0.23 中已弃用；请改用`Ndb_api_table_scan_count_replica`。

    由此副本启动的表扫描次数，包括内部表的扫描。

    尽管可以使用`SHOW GLOBAL STATUS`或`SHOW SESSION STATUS`读取此变量，但其实际范围是全局的。如果此 MySQL 服务器不充当副本，或不使用 NDB 表，则此值始终为 0。

    更多信息，请参阅第 25.6.15 节，“NDB API 统计计数器和变量”。

+   `Ndb_api_table_scan_count`

    由此 MySQL 服务器（SQL 节点）启动的表扫描次数，包括内部表的扫描。

    尽管可以使用`SHOW GLOBAL STATUS`或`SHOW SESSION STATUS`读取此变量，但其实际范围是全局的。

    更多信息，请参阅第 25.6.15 节，“NDB API 统计计数器和变量”。

+   `Ndb_api_trans_abort_count_session`

    在此客户端会话中中止的事务数量。

    尽管可以使用`SHOW GLOBAL STATUS`或`SHOW SESSION STATUS`读取此变量，但它仅与当前会话相关，并不受此**mysqld**的任何其他客户端影响。

    更多信息，请参阅第 25.6.15 节，“NDB API 统计计数器和变量”。

+   `Ndb_api_trans_abort_count_replica`

    该副本中被中止的事务数量。

    尽管可以使用`SHOW GLOBAL STATUS`或`SHOW SESSION STATUS`读取此变量，但其实际范围是全局的。如果此 MySQL 服务器不充当副本，或不使用 NDB 表，则此值始终为 0。

    更多信息，请参阅第 25.6.15 节，“NDB API 统计计数器和变量”。

+   `Ndb_api_trans_abort_count_slave`

    注意

    在 NDB 8.0.23 中已弃用；请改用`Ndb_api_trans_abort_count_replica`。

    由此副本中止的事务数量。

    尽管可以使用`SHOW GLOBAL STATUS`或`SHOW SESSION STATUS`来读取此变量，但其实质上是全局范围的。如果此 MySQL 服务器不充当副本，或不使用 NDB 表，此值始终为 0。

    更多信息，请参见第 25.6.15 节，“NDB API 统计计数器和变量”。

+   `Ndb_api_trans_abort_count`

    由此 MySQL 服务器（SQL 节点）中止的事务数量。

    尽管可以使用`SHOW GLOBAL STATUS`或`SHOW SESSION STATUS`来读取此变量，但其实质上是全局范围的。

    更多信息，请参见第 25.6.15 节，“NDB API 统计计数器和变量”。

+   `Ndb_api_trans_close_count_session`

    在此客户端会话中关闭的事务数量。此值可能大于`Ndb_api_trans_commit_count_session`和`Ndb_api_trans_abort_count_session`的总和，因为某些事务可能已被回滚。

    尽管可以使用`SHOW GLOBAL STATUS`或`SHOW SESSION STATUS`来读取此变量，但它仅与当前会话相关，并不受此**mysqld**的任何其他客户端影响。

    更多信息，请参见第 25.6.15 节，“NDB API 统计计数器和变量”。

+   `Ndb_api_trans_close_count_replica`

    由此副本关闭的事务数量。此值可能大于`Ndb_api_trans_commit_count_replica`和`Ndb_api_trans_abort_count_replica`的总和，因为某些事务可能已被回滚。

    尽管可以使用`SHOW GLOBAL STATUS`或`SHOW SESSION STATUS`读取此变量，但其实际范围是全局的。如果此 MySQL 服务器不充当副本，或不使用 NDB 表，则此值始终为 0。

    更多信息，请参见第 25.6.15 节，“NDB API 统计计数器和变量”。

+   `Ndb_api_trans_close_count_slave`

    注意

    在 NDB 8.0.23 中已弃用；请改用`Ndb_api_trans_close_count_replica`。

    由此副本关闭的事务数。此值可能大于`Ndb_api_trans_commit_count_replica`和`Ndb_api_trans_abort_count_replica`的总和，因为某些事务可能已被回滚。

    尽管可以使用`SHOW GLOBAL STATUS`或`SHOW SESSION STATUS`读取此变量，但其实际范围是全局的。如果此 MySQL 服务器不充当副本，或不使用 NDB 表，则此值始终为 0。

    更多信息，请参见第 25.6.15 节，“NDB API 统计计数器和变量”。

+   `Ndb_api_trans_close_count`

    由此 MySQL 服务器（SQL 节点）关闭的事务数。此值可能大于`Ndb_api_trans_commit_count`和`Ndb_api_trans_abort_count`的总和，因为某些事务可能已被回滚。

    尽管可以使用`SHOW GLOBAL STATUS`或`SHOW SESSION STATUS`读取此变量，但其实际范围是全局的。

    更多信息，请参见第 25.6.15 节，“NDB API 统计计数器和变量”。

+   `Ndb_api_trans_commit_count_session`

    在此客户端会话中提交的事务数。

    尽管可以使用 `SHOW GLOBAL STATUS` 或 `SHOW SESSION STATUS` 读取此变量，但它仅与当前会话相关，并不受此 **mysqld** 的任何其他客户端影响。

    更多信息，请参见 第 25.6.15 节，“NDB API 统计计数器和变量”。

+   `Ndb_api_trans_commit_count_replica`

    此副本提交的事务数量。

    尽管可以使用 `SHOW GLOBAL STATUS` 或 `SHOW SESSION STATUS` 读取此变量，但其实际范围是全局的。如果此 MySQL 服务器不充当副本，或不使用 NDB 表，则此值始终为 0。

    更多信息，请参见 第 25.6.15 节，“NDB API 统计计数器和变量”。

+   `Ndb_api_trans_commit_count_slave`

    注意

    在 NDB 8.0.23 版本中已弃用；请使用 `Ndb_api_trans_commit_count_replica` 代替。

    此副本提交的事务数量。

    尽管可以使用 `SHOW GLOBAL STATUS` 或 `SHOW SESSION STATUS` 读取此变量，但其实际范围是全局的。如果此 MySQL 服务器不充当副本，或不使用 NDB 表，则此值始终为 0。

    更多信息，请参见 第 25.6.15 节，“NDB API 统计计数器和变量”。

+   `Ndb_api_trans_commit_count`

    此 MySQL 服务器（SQL 节点）提交的事务数量。

    尽管可以使用 `SHOW GLOBAL STATUS` 或 `SHOW SESSION STATUS` 读取此变量，但其实际范围是全局的。

    更多信息，请参见 第 25.6.15 节，“NDB API 统计计数器和变量”。

+   `Ndb_api_trans_local_read_row_count_session`

    在此客户端会话中已读取的总行数。这包括在此客户端会话中进行的任何主键、唯一键或扫描操作读取的所有行。

    虽然这个变量可以使用`SHOW GLOBAL STATUS`或`SHOW SESSION STATUS`来读取，但它仅与当前会话相关，并不受此**mysqld**的任何其他客户端影响。

    更多信息，请参见 Section 25.6.15，“NDB API Statistics Counters and Variables”。

+   `Ndb_api_trans_local_read_row_count_replica`

    这个副本已读取的总行数。这包括此副本执行的任何主键、唯一键或扫描操作读取的所有行。

    虽然这个变量可以使用`SHOW GLOBAL STATUS`或`SHOW SESSION STATUS`来读取，但它实际上是全局范围的。如果此 MySQL 服务器不充当副本，或不使用 NDB 表，此值始终为 0。

    更多信息，请参见 Section 25.6.15，“NDB API Statistics Counters and Variables”。

+   `Ndb_api_trans_local_read_row_count_slave`

    注意

    在 NDB 8.0.23 中已弃用；请改用`Ndb_api_trans_local_read_row_count_replica`。

    这个副本已读取的总行数。这包括此副本执行的任何主键、唯一键或扫描操作读取的所有行。

    虽然这个变量可以使用`SHOW GLOBAL STATUS`或`SHOW SESSION STATUS`来读取，但它实际上是全局范围的。如果此 MySQL 服务器不充当副本，或不使用 NDB 表，此值始终为 0。

    更多信息，请参见 Section 25.6.15，“NDB API Statistics Counters and Variables”。

+   `Ndb_api_trans_local_read_row_count`

    这个 MySQL 服务器（SQL 节点）已读取的总行数。这包括此 MySQL 服务器（SQL 节点）执行的任何主键、唯一键或扫描操作读取的所有行。

    虽然这个变量可以使用`SHOW GLOBAL STATUS`或`SHOW SESSION STATUS`来读取，但它实际上是全局范围的。

    查看更多信息，请参见第 25.6.15 节，“NDB API 统计计数器和变量”。

+   `Ndb_api_trans_start_count_session`

    此客户端会话中启动的事务数量。

    尽管可以使用`SHOW GLOBAL STATUS`或`SHOW SESSION STATUS`来读取此变量，但它与当前会话相关，并不受此**mysqld**的任何其他客户端影响。

    查看更多信息，请参见第 25.6.15 节，“NDB API 统计计数器和变量”。

+   `Ndb_api_trans_start_count_replica`

    此副本启动的事务数量。

    尽管可以使用`SHOW GLOBAL STATUS`或`SHOW SESSION STATUS`来读取此变量，但其实际范围是全局的。如果此 MySQL 服务器不充当副本，或不使用 NDB 表，则此值始终为 0。

    查看更多信息，请参见第 25.6.15 节，“NDB API 统计计数器和变量”。

+   `Ndb_api_trans_start_count_slave`

    注意

    在 NDB 8.0.23 中已弃用；请改用`Ndb_api_trans_start_count_replica`。

    此副本启动的事务数量。

    尽管可以使用`SHOW GLOBAL STATUS`或`SHOW SESSION STATUS`来读取此变量，但其实际范围是全局的。如果此 MySQL 服务器不充当副本，或不使用 NDB 表，则此值始终为 0。

    查看更多信息，请参见第 25.6.15 节，“NDB API 统计计数器和变量”。

+   `Ndb_api_trans_start_count`

    此 MySQL 服务器（SQL 节点）启动的事务数量。

    尽管可以使用`SHOW GLOBAL STATUS`或`SHOW SESSION STATUS`来读取此变量，但其实际范围是全局的。

    更多信息，请参见第 25.6.15 节，“NDB API 统计计数器和变量”。

+   `Ndb_api_uk_op_count_session`

    这个客户端会话中基于或使用唯一键的操作次数。

    虽然这个变量可以通过`SHOW GLOBAL STATUS`或`SHOW SESSION STATUS`来读取，但它只与当前会话相关，并不受这个**mysqld**的任何其他客户端影响。

    更多信息，请参见第 25.6.15 节，“NDB API 统计计数器和变量”。

+   `Ndb_api_uk_op_count_replica`

    这个副本基于或使用唯一键的操作次数。

    虽然这个变量可以通过`SHOW GLOBAL STATUS`或`SHOW SESSION STATUS`来读取，但实际上它在全局范围内有效。如果这个 MySQL 服务器不充当副本，或者不使用 NDB 表，这个值始终为 0。

    更多信息，请参见第 25.6.15 节，“NDB API 统计计数器和变量”。

+   `Ndb_api_uk_op_count_slave`

    注意

    在 NDB 8.0.23 中已弃用；请使用`Ndb_api_uk_op_count_replica`代替。

    这个副本基于或使用唯一键的操作次数。

    虽然这个变量可以通过`SHOW GLOBAL STATUS`或`SHOW SESSION STATUS`来读取，但实际上它在全局范围内有效。如果这个 MySQL 服务器不充当副本，或者不使用 NDB 表，这个值始终为 0。

    更多信息，请参见第 25.6.15 节，“NDB API 统计计数器和变量”。

+   `Ndb_api_uk_op_count`

    这个 MySQL 服务器（SQL 节点）基于或使用唯一键的操作次数。

    虽然这个变量可以通过`SHOW GLOBAL STATUS`或`SHOW SESSION STATUS`来读取，但实际上它在全局范围内有效。

    更多信息，请参见第 25.6.15 节，“NDB API 统计计数器和变量”。

+   `Ndb_api_wait_exec_complete_count_session`

    线程在此客户端会话中由于等待操作完成而被阻塞的次数。这包括所有`execute()`调用，以及对客户端不可见的 blob 和自增操作的隐式执行。

    尽管可以使用`SHOW GLOBAL STATUS`或`SHOW SESSION STATUS`读取此变量，但它仅与当前会话相关，并不受此**mysqld**的任何其他客户端影响。

    更多信息，请参见第 25.6.15 节，“NDB API 统计计数器和变量”。

+   `Ndb_api_wait_exec_complete_count_replica`

    线程由于等待操作完成而被此副本阻塞的次数。这包括所有`execute()`调用，以及对客户端不可见的 blob 和自增操作的隐式执行。

    尽管可以使用`SHOW GLOBAL STATUS`或`SHOW SESSION STATUS`读取此变量，但其实际范围是全局的。如果此 MySQL 服务器不充当副本，或不使用 NDB 表，此值始终为 0。

    更多信息，请参见第 25.6.15 节，“NDB API 统计计数器和变量”。

+   `Ndb_api_wait_exec_complete_count_slave`

    注意

    在 NDB 8.0.23 中已弃用；请改用`Ndb_api_wait_exec_complete_count_replica`。

    线程由于等待操作完成而被此副本阻塞的次数。这包括所有`execute()`调用，以及对客户端不可见的 blob 和自增操作的隐式执行。

    尽管可以使用`SHOW GLOBAL STATUS`或`SHOW SESSION STATUS`读取此变量，但其实是全局范围的。如果此 MySQL 服务器不充当复制品，或不使用 NDB 表，此值始终为 0。

    更多信息，请参见第 25.6.15 节，“NDB API 统计计数器和变量”。

+   `Ndb_api_wait_exec_complete_count`

    一个线程在等待操作完成时被此 MySQL 服务器（SQL 节点）阻塞的次数。这包括所有`execute()`调用，以及对客户端不可见的 blob 和自增操作的隐式执行。

    尽管可以使用`SHOW GLOBAL STATUS`或`SHOW SESSION STATUS`读取此变量，但其实是全局范围的。

    更多信息，请参见第 25.6.15 节，“NDB API 统计计数器和变量”。

+   `Ndb_api_wait_meta_request_count_session`

    一个线程在此客户端会话中被阻塞等待基于元数据的信号的次数，例如 DDL 请求、新纪元和事务记录的占用。

    尽管可以使用`SHOW GLOBAL STATUS`或`SHOW SESSION STATUS`读取此变量，但它仅与当前会话相关，并不受此**mysqld**的任何其他客户端影响。

    更多信息，请参见第 25.6.15 节，“NDB API 统计计数器和变量”。

+   `Ndb_api_wait_meta_request_count_replica`

    一个线程在此复制品中被阻塞等待基于元数据的信号的次数，例如 DDL 请求、新纪元和事务记录的占用。

    尽管可以使用`SHOW GLOBAL STATUS`或`SHOW SESSION STATUS`读取此变量，但其实是全局范围的。如果此 MySQL 服务器不充当复制品，或不使用 NDB 表，此值始终为 0。

    查看更多信息，请参阅第 25.6.15 节，“NDB API 统计计数器和变量”。

+   `Ndb_api_wait_meta_request_count_slave`

    注意

    在 NDB 8.0.23 中已弃用；请改用`Ndb_api_wait_meta_request_count_replica`。

    线程由于等待基于元数据的信号（例如 DDL 请求、新纪元和事务记录的占用）而被此复制品阻塞的次数。

    尽管可以使用`SHOW GLOBAL STATUS`或`SHOW SESSION STATUS`读取此变量，但其实际上是全局范围的。如果此 MySQL 服务器不充当复制品，或不使用 NDB 表，则此值始终为 0。

    查看更多信息，请参阅第 25.6.15 节，“NDB API 统计计数器和变量”。

+   `Ndb_api_wait_meta_request_count`

    线程由于等待基于元数据的信号（例如 DDL 请求、新纪元和事务记录的占用）而被此 MySQL 服务器（SQL 节点）阻塞的次数。

    尽管可以使用`SHOW GLOBAL STATUS`或`SHOW SESSION STATUS`读取此变量，但其实际上是全局范围的。

    查看更多信息，请参阅第 25.6.15 节，“NDB API 统计计数器和变量”。

+   `Ndb_api_wait_nanos_count_session`

    在此客户端会话中等待来自数据节点的任何类型信号所花费的总时间（以纳秒为单位）。

    尽管可以使用`SHOW GLOBAL STATUS`或`SHOW SESSION STATUS`读取此变量，但它仅与当前会话相关，并且不受此**mysqld**的任何其他客户端影响。

    查看更多信息，请参阅第 25.6.15 节，“NDB API 统计计数器和变量”。

+   `Ndb_api_wait_nanos_count_replica`

    复制品等待来自数据节点的任何类型信号所花费的总时间（以纳秒为单位）。

    尽管可以使用`SHOW GLOBAL STATUS`或`SHOW SESSION STATUS`读取此变量，但其实际范围是全局的。如果此 MySQL 服务器不充当副本，或不使用 NDB 表，则此值始终为 0。

    查看更多信息，请参阅第 25.6.15 节，“NDB API 统计计数器和变量”。

+   `Ndb_api_wait_nanos_count_slave`

    注意

    在 NDB 8.0.23 中已弃用；请改用`Ndb_api_wait_nanos_count_replica`。

    该副本等待来自数据节点的任何类型信号所花费的总时间（以纳秒为单位）。

    尽管可以使用`SHOW GLOBAL STATUS`或`SHOW SESSION STATUS`读取此变量，但其实际范围是全局的。如果此 MySQL 服务器不充当副本，或不使用 NDB 表，则此值始终为 0。

    查看更多信息，请参阅第 25.6.15 节，“NDB API 统计计数器和变量”。

+   `Ndb_api_wait_nanos_count`

    该 MySQL 服务器（SQL 节点）等待来自数据节点的任何类型信号所花费的总时间（以纳秒为单位）。

    尽管可以使用`SHOW GLOBAL STATUS`或`SHOW SESSION STATUS`读取此变量，但其实际范围是全局的。

    查看更多信息，请参阅第 25.6.15 节，“NDB API 统计计数器和变量”。

+   `Ndb_api_wait_scan_result_count_session`

    在此客户端会话中，线程由于等待基于扫描的信号而被阻塞的次数，例如等待扫描结果，或等待扫描关闭时。

    尽管可以使用`SHOW GLOBAL STATUS`或`SHOW SESSION STATUS`读取此变量，但它仅与当前会话相关，并不受此**mysqld**的任何其他客户端影响。

    查看更多信息，请参阅第 25.6.15 节，“NDB API 统计计数器和变量”。

+   `Ndb_api_wait_scan_result_count_replica`

    线程由于等待扫描信号（例如等待扫描结果或等待扫描关闭）而被此副本阻塞的次数。

    尽管可以使用`SHOW GLOBAL STATUS`或`SHOW SESSION STATUS`读取此变量，但其实际范围是全局的。如果此 MySQL 服务器不充当副本，或不使用 NDB 表，则此值始终为 0。

    查看更多信息，请参见第 25.6.15 节，“NDB API 统计计数器和变量”。

+   `Ndb_api_wait_scan_result_count_slave`

    注意

    在 NDB 8.0.23 中已弃用；请改用`Ndb_api_wait_scan_result_count_replica`。

    线程由于等待扫描信号（例如等待扫描结果或等待扫描关闭）而被此副本阻塞的次数。

    尽管可以使用`SHOW GLOBAL STATUS`或`SHOW SESSION STATUS`读取此变量，但其实际范围是全局的。如果此 MySQL 服务器不充当副本，或不使用 NDB 表，则此值始终为 0。

    查看更多信息，请参见第 25.6.15 节，“NDB API 统计计数器和变量”。

+   `Ndb_api_wait_scan_result_count`

    线程由于等待扫描信号（例如等待扫描结果或等待扫描关闭）而被此 MySQL 服务器（SQL 节点）阻塞的次数。

    尽管可以使用`SHOW GLOBAL STATUS`或`SHOW SESSION STATUS`读取此变量��但其实际范围是全局的。

    查看更多信息，请参见第 25.6.15 节，“NDB API 统计计数器和变量”。

+   `Ndb_cluster_node_id`

    如果服务器充当 NDB 集群节点，则此变量的值为其在集群中的节点 ID。

    如果服务器不是 NDB 集群的一部分，则此变量的值为 0。

+   `Ndb_config_from_host`

    如果服务器是 NDB 集群的一部分，则此变量的值是从其获取配置数据的集群管理服务器的主机名或 IP 地址。

    如果服务器不是 NDB 集群的一部分，则此变量的值为空字符串。

+   `Ndb_config_from_port`

    如果服务器是 NDB 集群的一部分，则此变量的值是通过其连接到的用于获取配置数据的集群管理服务器的端口号。

    如果服务器不是 NDB 集群的一部分，则此变量的值为 0。

+   `Ndb_config_generation`

    显示集群当前配置的生成编号。这可用作指示器，用于确定自此 SQL 节点上次连接到集群以来集群的配置是否发生了更改。

+   `Ndb_conflict_fn_epoch`

    在 NDB 集群复制冲突解决中使用，此变量显示使用`NDB$EPOCH()`冲突解决在给定的**mysqld**上找到的冲突行数，自上次重新启动以来。

    欲了解更多信息，请参阅第 25.7.12 节，“NDB 集群复制冲突解决”。

+   `Ndb_conflict_fn_epoch_trans`

    在 NDB 集群复制冲突解决中使用，此变量显示使用`NDB$EPOCH_TRANS()`冲突解决在给定的**mysqld**上找到的冲突行数，自上次重新启动以来。

    欲了解更多信息，请参阅第 25.7.12 节，“NDB 集群复制冲突解决”。

+   `Ndb_conflict_fn_epoch2`

    使用`NDB$EPOCH2()`时，在 NDB 集群复制冲突解决中找到的冲突行数，源自上次重新启动以来被指定为主服务器的源。

    欲了解更多信息，请参阅 NDB$EPOCH2()")。

+   `Ndb_conflict_fn_epoch2_trans`

    在 NDB 集群复制冲突解决中使用，此变量显示使用`NDB$EPOCH_TRANS2()`冲突解决在给定的**mysqld**上找到的冲突行数，自上次重新启动以来。

    查看更多信息，请参见 NDB$EPOCH2_TRANS()")。

+   `Ndb_conflict_fn_max`

    用于 NDB 集群复制冲突解决，此变量显示由于“最大时间戳获胜”冲突解决而在当前 SQL 节点上未应用行的次数，自上次启动此**mysqld**以来。

    查看更多信息，请参见第 25.7.12 节，“NDB 集群复制冲突解决”。

+   `Ndb_conflict_fn_max_del_win`

    显示由于 NDB 集群复制冲突解决使用`NDB$MAX_DELETE_WIN()`而在当前 SQL 节点上拒绝行的次数，自上次启动此**mysqld**以来。

    查看更多信息，请参见第 25.7.12 节，“NDB 集群复制冲突解决”。

+   `Ndb_conflict_fn_max_del_win_ins`

    显示由于 NDB 集群复制冲突解决使用`NDB$MAX_DEL_WIN_INS()`而在当前 SQL 节点上拒绝插入行的次数，自上次启动此**mysqld**以来。

    查看更多信息，请参见第 25.7.12 节，“NDB 集群复制冲突解决”。

+   `Ndb_conflict_fn_max_ins`

    用于 NDB 集群复制冲突解决，此变量显示自上次启动此**mysqld**以来，由于“最大时间戳获胜”冲突解决而未在当前 SQL 节点上插入行的次数。

    查看更多信息，请参见第 25.7.12 节，“NDB 集群复制冲突解决”。

+   `Ndb_conflict_fn_old`

    用于 NDB 集群复制冲突解决，此变量显示由于“相同时间戳获胜”冲突解决而在给定的**mysqld**上未应用行的次数，自上次重新启动以来。

    更多信息，请参见第 25.7.12 节，“NDB 集群复制冲突解决”。

+   `Ndb_conflict_last_conflict_epoch`

    在此副本上检测到冲突的最新时代。您可以将此值与`Ndb_replica_max_replicated_epoch`进行比较；如果`Ndb_replica_max_replicated_epoch`大于`Ndb_conflict_last_conflict_epoch`，则尚未检测到冲突。

    查看第 25.7.12 节，“NDB 集群复制冲突解决”，获取更多信息。

+   `Ndb_conflict_reflected_op_discard_count`

    当使用 NDB 集群复制冲突解决时，这是未在辅助节点上应用的反射操作数量，因为在执行过程中遇到错误。

    查看第 25.7.12 节，“NDB 集群复制冲突解决”，获取更多信息。

+   `Ndb_conflict_reflected_op_prepare_count`

    当使用 NDB 集群复制的冲突解决时，此状态变量包含已定义的反射操作数量（即在辅助节点上准备执行的操作）。

    查看第 25.7.12 节，“NDB 集群复制冲突解决”，获取更多信息。

+   `Ndb_conflict_refresh_op_count`

    当使用 NDB 集群复制的冲突解决时，这是已准备在辅助节点上执行的刷新操作数量。

    查看第 25.7.12 节，“NDB 集群复制冲突解决”，获取更多信息。

+   `Ndb_conflict_last_stable_epoch`

    事务冲突函数发现的冲突行数

    查看第 25.7.12 节，“NDB 集群复制冲突解决”，获取更多信息。

+   `Ndb_conflict_trans_row_conflict_count`

    用于 NDB Cluster 复制冲突解决，该状态变量显示自上次重启以来，由事务冲突函数直接确定为冲突的行数。

    目前，NDB Cluster 支持的唯一事务冲突检测函数是 NDB$EPOCH_TRANS()，因此该状态变量实际上与 `Ndb_conflict_fn_epoch_trans` 相同。

    欲了解更多信息，请参阅 第 25.7.12 节，“NDB Cluster 复制冲突解决”。

+   `Ndb_conflict_trans_row_reject_count`

    用于 NDB Cluster 复制冲突解决，该状态变量显示由于被事务冲突检测函数确定为冲突而重新调整的行总数。这不仅包括 `Ndb_conflict_trans_row_conflict_count`，还包括任何在冲突事务中或依赖于冲突事务的行。

    欲了解更多信息，请参阅 第 25.7.12 节，“NDB Cluster 复制冲突解决”。

+   `Ndb_conflict_trans_reject_count`

    用于 NDB Cluster 复制冲突解决，该状态变量显示通过事务冲突检测函数发现的冲突事务数量。

    欲了解更多信息，请参阅 第 25.7.12 节，“NDB Cluster 复制冲突解决”。

+   `Ndb_conflict_trans_detect_iter_count`

    用于 NDB Cluster 复制冲突解决，显示提交时需要的内部迭代次数。应该（略微）大于或等于 `Ndb_conflict_trans_conflict_commit_count`。

    欲了解更多信息，请参阅 第 25.7.12 节，“NDB Cluster 复制冲突解决”。

+   `Ndb_conflict_trans_conflict_commit_count`

    用于 NDB Cluster 复制冲突解决，显示需要事务冲突处理后提交的时代事务数量。

    更多信息，请参见 Section 25.7.12, “NDB Cluster Replication Conflict Resolution”。

+   `Ndb_epoch_delete_delete_count`

    在使用删除-删除冲突检测时，检测到的删除-删除冲突数量，其中应用了删除操作，但指定的行不存在。

+   `Ndb_execute_count`

    提供由操作向`NDB`内核进行的往返次数。

+   `Ndb_last_commit_epoch_server`

    最近由`NDB`提交的时代。

+   `Ndb_last_commit_epoch_session`

    最近由此`NDB`客户端提交的时代。

+   `Ndb_metadata_detected_count`

    自此服务器上次启动以来，NDB 元数据更改检测线程发现与 MySQL 数据字典相关的更改次数。

+   `Ndb_metadata_excluded_count`

    自上次重新启动以来，NDB 二进制日志线程无法在此 SQL 节点上同步的元数据对象数量。

    如果对象被排除，则直到用户手动纠正不匹配为止，不再考虑自动同步。可以通过尝试使用类似`SHOW CREATE TABLE *`table`*`，`SELECT * FROM *`table`*`或任何触发表发现的语句来执行此操作。

    在 NDB 8.0.22 之前，此变量名为`Ndb_metadata_blacklist_size`。

+   `Ndb_metadata_synced_count`

    自上次重新启动以来，在此 SQL 节点上已同步的 NDB 元数据对象的数量。

+   `Ndb_number_of_data_nodes`

    如果服务器是 NDB 集群的一部分，则此变量的值是集群中数据节点的数量。

    如果服务器不是 NDB 集群的一部分，则此变量的值为 0。

+   `Ndb_pushed_queries_defined`

    将联接下推到 NDB 内核以进行数据节点上的分布式处理的总次数。

    注意

    使用`EXPLAIN`测试可以下推的联接会对此数字产生影响。

+   `Ndb_pushed_queries_dropped`

    被推送到 NDB 内核但无法在那里处理的联接数量。

+   `Ndb_pushed_queries_executed`

    成功推送到`NDB`并在那里执行的连接数。

+   `Ndb_pushed_reads`

    通过被推送的连接从 NDB 内核返回给**mysqld**的行数。

    注意

    对可以推送到`NDB`的连接执行`EXPLAIN`不会增加此数字。

+   `Ndb_pruned_scan_count`

    此变量保存自 NDB 集群上次启动以来执行的扫描总数，其中 NDB 集群能够使用分区修剪。

    将此变量与`Ndb_scan_count`一起使用，有助于在模式设计中最大化服务器修剪扫描到单个表分区的能力，从而仅涉及复制一个数据节点。

+   `Ndb_replica_max_replicated_epoch`

    此副本上最近提交的时代。您可以将此值与`Ndb_conflict_last_conflict_epoch`进行比较；如果`Ndb_replica_max_replicated_epoch`是两者中较大的值，则尚未检测到冲突。

    更多信息，请参见第 25.7.12 节，“NDB 集群复制冲突解决”。

+   `Ndb_scan_count`

    此变量保存自 NDB 集群上次启动以来执行的扫描总数。

+   `Ndb_slave_max_replicated_epoch`

    注意

    在 NDB 8.0.23 中已弃用；请改用`Ndb_slave_max_replicated_epoch`。

    此副本上最近提交的时代。您可以将此值与`Ndb_conflict_last_conflict_epoch`进行比较；如果`Ndb_slave_max_replicated_epoch`是两者中较大的值，则尚未检测到冲突。

    更多信息，请参见第 25.7.12 节，“NDB 集群复制冲突解决”。

+   `Ndb_system_name`

    如果这个 MySQL 服务器连接到一个 NDB 集群，这个只读变量显示集群系统名称。否则，该值为空字符串。

+   `Ndb_trans_hint_count_session`

    在当前会话中已经启动的使用提示的事务数量。与`Ndb_api_trans_start_count_session`进行比较，以获得所有能够使用提示的 NDB 事务的比例。
