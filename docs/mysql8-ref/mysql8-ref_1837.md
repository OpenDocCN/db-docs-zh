# 25.7.11 NDB 集群使用多线程应用程序的复制

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-replication-mta.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-replication-mta.html)

+   [需求](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-replication-mta.html#cluster-replication-mta-reqs "需求")

+   [MTA 配置：源](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-replication-mta.html#cluster-replication-mta-config-source "MTA 配置：源")

+   [MTA 配置：副本](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-replication-mta.html#cluster-replication-mta-config-replica "MTA 配置：副本")

+   [事务依赖和写集处理](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-replication-mta.html#cluster-replication-mta-transaction-deps "事务依赖和写集处理")

+   [写集跟踪内存使用](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-replication-mta.html#cluster-replication-mta-writeset-tracking "写集跟踪内存使用")

+   [已知限制](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-replication-mta.html#cluster-replication-mta-limitations "已知限制")

从 NDB 8.0.33 开始，NDB 复制支持使用通用的 MySQL 服务器多线程应用程序机制（MTA），允许在副本上并行应用独立的二进制日志事务，从而增加了峰值复制吞吐量。

#### 需求

MySQL 服务器 MTA 实现将单独的二进制日志事务的处理委托给一组工作线程（其大小是可配置的），并协调工作线程以确保二进制日志中编码的事务依赖关系得到尊重，并在必要时保持提交顺序（参见[第 19.2.3 节，“复制线程”](https://dev.mysql.com/doc/refman/8.0/en/replication-threads.html "19.2.3 复制线程")）。要在 NDB 集群中使用此功能，必须满足以下三个条件：

1.  *二进制日志事务依赖关系在源端确定*。

    为了实现这一点，必须在源端将[`binlog_transaction_dependency_tracking`](https://dev.mysql.com/doc/refman/8.0/en/replication-options-binary-log.html#sysvar_binlog_transaction_dependency_tracking)服务器系统变量设置为`WRITESET`。这在 NDB 8.0.33 及更高版本中受支持。（默认值为`COMMIT_ORDER`。）

    `NDB`中的写集维护工作由 MySQL 二进制日志注入线程执行，作为准备和提交每个时代事务到二进制日志的一部分。这需要额外的资源，并可能降低峰值吞吐量。

1.  *事务依赖关系被编码到二进制日志中*。

    NDB 8.0.33 及更高版本支持[`--ndb-log-transaction-dependency`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-options-variables.html#option_mysqld_ndb-log-transaction-dependency)启动选项用于[**mysqld**](https://dev.mysql.com/doc/refman/8.0/en/mysqld.html "6.3.1 mysqld — MySQL 服务器"); 将此选项设置为`ON`以启用将`NDB`事务依赖关系写入二进制日志。

1.  *副本配置为使用多个工作线程*。

    NDB 8.0.33 及更高版本支持将`replica_parallel_workers`设置为非零值，以控制副本上的工作线程数。默认值为 4。

#### MTA 配置：源

`NDB` MTA 的源**mysqld**配置必须包括以下显式设置：

+   `binlog_transaction_dependency_tracking`必须设置为`WRITESET`。

+   复制源 mysqld 必须使用`--ndb-log-transaction-dependency=ON`启动。

如果设置了，`replica_parallel_type`必须为`LOGICAL_CLOCK`（默认值）。

注意

`NDB`不支持`replica_parallel_type=DATABASE`。

此外，建议您将用于跟踪二进制日志事务写入集的内存量设置为源上的`binlog_transaction_dependency_history_size`为`*`E`* * *`P`*`，其中*`E`*是平均时代大小（即每个时代的操作数），*`P`*是最大预期并行性。有关更多信息，请参见写集跟踪内存使用情况。

#### MTA 配置：副本

`NDB` MTA 的副本**mysqld**配置要求`replica_parallel_workers`大于 1。首次启用 MTA 时的推荐起始值为 4，这是默认值。

此外，`replica_preserve_commit_order`必须为`ON`。这也是默认值。

#### 事务依赖性和写集处理

通过分析每个事务的写集（即事务写入的行（表、键值）集）来检测事务依赖关系。如果两个事务修改相同的行，则它们被视为依赖关系，并且必须按顺序（换句话说，串行）应用，以避免死锁或不正确的结果。如果表具有辅助唯一键，这些值也会添加到事务的写集中，以检测由不同事务影响相同唯一键值而暗示事务依赖关系的情况，因此需要排序。无法有效确定依赖关系的情况下，**mysqld**会退回到考虑出于安全原因而依赖于事务的情况。

事务依赖关系通过源**mysqld**在二进制日志中编码。依赖关系通过使用称为“逻辑时钟”的方案在`ANONYMOUS_GTID`事件中编码。（参见 Section 19.1.4.1, “Replication Mode Concepts”.）

MySQL（和 NDB Cluster）采用的写集实现使用基于哈希的冲突检测，基于匹配的相关表和索引值的 64 位行哈希。这可靠地检测到当相同键被看到两次时，但如果不同的表和索引值哈希到相同的 64 位值，则可能产生误报，这可能导致人为依赖关系，从而降低可用的并行性。

事务依赖关系由以下任一方式强制：

+   DDL 语句

+   二进制日志轮换或遇到二进制日志文件边界

+   写集历史大小限制

+   写入引用目标表中的父外键

    更具体地说，对外键*父*表执行插入、更新和删除的事务相对于所有前后事务进行序列化，而不仅仅是与涉及约束关系的表相关的事务。相反，对外键*子*表（引用）执行插入、更新和删除的事务与彼此之间并没有特别的序列化。

MySQL MTA 实现尝试并行应用独立的二进制日志事务。`NDB`记录在一个二进制日志事务中发生的所有用户事务在一个时期内提交的所有更改（`TimeBetweenEpochs`，默认为 100 毫秒）。因此，为了使两个连续的时期事务独立且可能并行应用，需要确保在两个时期中没有任何行被修改。如果任何单行在两个时期中都被修改，则它们是依赖的，并且按顺序应用，这可能限制可利用的并行性。

基于在时期内在源集群上修改的行集，但不包括传达时期元数据的生成的`mysql.ndb_apply_status` `WRITE_ROW`事件，时期事务被视为独立的。这避免了每个时期事务都简单地依赖于前一个时期，但需要在保留提交顺序的情况下在副本上应用 binlog。这也意味着具有写集依赖关系的 NDB 二进制日志不适合由使用不同 MySQL 存储引擎的副本数据库使用。

可能或有必要修改应用程序事务行为，以避免在短时间内通过单独的事务重复修改相同行的模式，以增加可利用的应用并行性。

#### 写集跟踪内存使用

用于跟踪二进制日志事务写入集的内存使用量可以使用`binlog_transaction_dependency_history_size`服务器系统变量进行设置，默认为 25000 行哈希。

如果平均二进制日志事务修改了*`N`*行，则为了能够识别独立（可并行化）事务达到并行级别*`P`*，我们需要`binlog_transaction_dependency_history_size`至少为`*`N`* * *`P`*`。（最大值为 1000000。）

历史记录的有限大小导致可靠确定的有限最大依赖长度，从而给出可以表达的有限并行性。在历史记录中找不到的任何行可能依赖于从历史记录中清除的最后一个事务。

写入集历史不像是对最后*`N`*个事务的滑动窗口；相反，它是一个允许完全填满的有限缓冲区，当其完全填满时，其内容完全丢弃。这意味着历史大小随时间呈锯齿状变化，因此最大可检测的依赖长度也随时间呈锯齿状变化，这样，如果写入集历史缓冲区在它们被处理之间被重置，独立事务仍可能被标记为依赖。

在这个方案中，二进制日志文件中的每个事务都带有一个`sequence_number`（1、2、3，...），以及它依赖的最近二进制日志事务的序列号，我们称之为`last_committed`。

在给定的二进制日志文件中，第一个事务的`sequence_number`为 1，`last_committed`为 0。

如果一个二进制日志事务依赖于其直接前任，则其应用是串行的。如果依赖于较早的事务，则可能可以与前面的独立事务并行应用该事务。

`ANONYMOUS_GTID`事件的内容，包括`sequence_number`和`last_committed`（因此事务依赖关系），可以使用**mysqlbinlog**查看。

在源上生成的`ANONYMOUS_GTID`事件与批量`BEGIN`、`TABLE_MAP*`、`WRITE_ROW*`、`UPDATE_ROW*`、`DELETE_ROW*`和`COMMIT`事件的压缩事务有效载荷分开处理，允许在解压缩之前确定依赖关系。这意味着副本协调器线程可以将事务有效载荷解压缩委托给工作线程，从而在副本上自动并行解压缩独立事务。

#### 已知限制

**次要唯一列。** 具有次要唯一列（即主键以外的唯一键）的表将所有列发送到源，以便可以检测到与唯一键相关的冲突。

当当前的二进制日志模式不包括所有列，而只包括更改的列（`--ndb-log-updated-only=OFF`, `--ndb-log-update-minimal=ON`, `--ndb-log-update-as-write=OFF`)时，这可能会增加从数据节点发送到 SQL 节点的数据量。

影响取决于这些表中行的修改（更新或删除）速率以及实际未修改的列中的数据量。

**将 NDB 复制到 InnoDB。** `NDB` 二进制日志注入器事务依赖跟踪有意忽略由生成的 `mysql.ndb_apply_status` 元数据事件创建的事务间依赖关系，这些事件作为副本应用程序上的时代事务提交的一部分单独处理。对于复制到`InnoDB`，没有特殊处理；当使用`InnoDB`多线程应用程序来消耗`NDB` MTA 二进制日志时，这可能导致性能降低或其他问题。
