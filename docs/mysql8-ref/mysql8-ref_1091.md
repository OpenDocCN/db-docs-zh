> 原文：[`dev.mysql.com/doc/refman/8.0/en/show-engine.html`](https://dev.mysql.com/doc/refman/8.0/en/show-engine.html)

#### 15.7.7.15 SHOW ENGINE Statement

```sql
SHOW ENGINE *engine_name* {STATUS | MUTEX}
```

`SHOW ENGINE`显示有关存储引擎的操作信息。它需要`PROCESS`权限。该语句有以下变体：

```sql
SHOW ENGINE INNODB STATUS
SHOW ENGINE INNODB MUTEX
SHOW ENGINE PERFORMANCE_SCHEMA STATUS
```

`SHOW ENGINE INNODB STATUS`显示有关`InnoDB`存储引擎状态的标准`InnoDB`监视器的详细信息。有关标准监视器和提供有关`InnoDB`处理信息的其他`InnoDB`监视器的信息，请参见第 17.17 节，“InnoDB 监视器”。

`SHOW ENGINE INNODB MUTEX`显示`InnoDB` mutex 和 rw-lock 统计信息。

注意

也可以使用性能模式表监视`InnoDB`互斥锁和读写锁。请参见第 17.16.2 节，“使用性能模式监视 InnoDB 互斥锁等待”。

互斥锁统计信息的收集通过以下选项动态配置：

+   要启用互斥锁统计信息的收集，请运行：

    ```sql
    SET GLOBAL innodb_monitor_enable='latch';
    ```

+   要重置互斥锁统计信息，请运行：

    ```sql
    SET GLOBAL innodb_monitor_reset='latch';
    ```

+   要禁用互斥锁统计信息的收集，请运行：

    ```sql
    SET GLOBAL innodb_monitor_disable='latch';
    ```

通过设置`innodb_monitor_enable='all'`可以启用对`SHOW ENGINE INNODB MUTEX`的互斥锁统计信息的收集，通过设置`innodb_monitor_disable='all'`可以禁用。

`SHOW ENGINE INNODB MUTEX`输出具有以下列：

+   `Type`

    始终使用`InnoDB`。

+   `Name`

    对于互斥锁，`Name`字段仅报告互斥锁名称。对于读写锁，`Name`字段报告实现读写锁的源文件，以及创建读写锁的文件中的行号。行号特定于您的 MySQL 版本。

+   `Status`

    互斥锁状态。此字段报告旋转次数、等待次数和调用次数。不报告在`InnoDB`之外实现的低级操作系统互斥锁的统计信息。

    +   `spins`表示旋转次数。

    +   `waits`表示互斥锁等待次数。

    +   `calls`表示请求互斥锁的次数。

`SHOW ENGINE INNODB MUTEX` 不会列出每个缓冲池块的互斥锁和读写锁，因为在具有大缓冲池的系统上，输出量会非常庞大。然而，`SHOW ENGINE INNODB MUTEX` 会打印缓冲池块互斥锁和读写锁的聚合 `BUF_BLOCK_MUTEX` 自旋、等待和调用值。`SHOW ENGINE INNODB MUTEX` 也不会列出任何从未等待过的互斥锁或读写锁（`os_waits=0`）。因此，`SHOW ENGINE INNODB MUTEX` 仅显示导致至少一个操作系统级 等待 的互斥锁和读写锁的信息。

使用 `SHOW ENGINE PERFORMANCE_SCHEMA STATUS` 来检查性能模式代码的内部操作：

```sql
mysql> SHOW ENGINE PERFORMANCE_SCHEMA STATUS\G
...
*************************** 3\. row ***************************
  Type: performance_schema
  Name: events_waits_history.size
Status: 76
*************************** 4\. row ***************************
  Type: performance_schema
  Name: events_waits_history.count
Status: 10000
*************************** 5\. row ***************************
  Type: performance_schema
  Name: events_waits_history.memory
Status: 760000
...
*************************** 57\. row ***************************
  Type: performance_schema
  Name: performance_schema.memory
Status: 26459600
...
```

该语句旨在帮助数据库管理员了解不同性能模式选项对内存需求的影响。

`Name` 值由两部分组成，分别命名内部缓冲区和缓冲属性。解释缓冲区名称如下：

+   未公开为表的内部缓冲区在括号内命名。例如：`(pfs_cond_class).size`，`(pfs_mutex_class).memory`。

+   在 `performance_schema` 数据库中作为表公开的内部缓冲区以表名命名，不带括号。例如：`events_waits_history.size`，`mutex_instances.count`。

+   适用于整个性能模式的值以 `performance_schema` 开头。例如：`performance_schema.memory`。

缓冲属性具有以下含义：

+   `size` 是实现中使用的内部记录的大小，比如表中行的大小。`size` 值无法更改。

+   `count` 是内部记录的数量，比如表中的行数。`count` 值可以通过性能模式配置选项进行更改。

+   对于表，`*tbl_name`*.memory` 是 `size` 和 `count` 的乘积。对于整个性能模式，`performance_schema.memory` 是所有内存使用量的总和（所有其他 `memory` 值的总和）。

在某些情况下，性能模式配置参数与 `SHOW ENGINE` 值之间存在直接关系。例如，`events_waits_history_long.count` 对应于 `performance_schema_events_waits_history_long_size`。在其他情况下，关系更为复杂。例如，`events_waits_history.count` 对应于 `performance_schema_events_waits_history_size`（每个线程的行数）乘以 `performance_schema_max_thread_instances`（线程数）。

**SHOW ENGINE NDB STATUS. ** 如果服务器启用了`NDB`存储引擎，`SHOW ENGINE NDB STATUS`显示集群状态信息，如连接的数据节点数量、集群连接字符串和集群二进制日志时代，以及 MySQL 服务器连接到集群时创建的各种 Cluster API 对象的计数。此语句的示例输出如下：

```sql
mysql> SHOW ENGINE NDB STATUS;
+------------+-----------------------+--------------------------------------------------+
| Type       | Name                  | Status                                           |
+------------+-----------------------+--------------------------------------------------+
| ndbcluster | connection            | cluster_node_id=7,
  connected_host=198.51.100.103, connected_port=1186, number_of_data_nodes=4,
  number_of_ready_data_nodes=3, connect_count=0                                         |
| ndbcluster | NdbTransaction        | created=6, free=0, sizeof=212                    |
| ndbcluster | NdbOperation          | created=8, free=8, sizeof=660                    |
| ndbcluster | NdbIndexScanOperation | created=1, free=1, sizeof=744                    |
| ndbcluster | NdbIndexOperation     | created=0, free=0, sizeof=664                    |
| ndbcluster | NdbRecAttr            | created=1285, free=1285, sizeof=60               |
| ndbcluster | NdbApiSignal          | created=16, free=16, sizeof=136                  |
| ndbcluster | NdbLabel              | created=0, free=0, sizeof=196                    |
| ndbcluster | NdbBranch             | created=0, free=0, sizeof=24                     |
| ndbcluster | NdbSubroutine         | created=0, free=0, sizeof=68                     |
| ndbcluster | NdbCall               | created=0, free=0, sizeof=16                     |
| ndbcluster | NdbBlob               | created=1, free=1, sizeof=264                    |
| ndbcluster | NdbReceiver           | created=4, free=0, sizeof=68                     |
| ndbcluster | binlog                | latest_epoch=155467, latest_trans_epoch=148126,
  latest_received_binlog_epoch=0, latest_handled_binlog_epoch=0,
  latest_applied_binlog_epoch=0                                                         |
+------------+-----------------------+--------------------------------------------------+
```

这些行中的`Status`列提供有关 MySQL 服务器与集群的连接以及集群二进制日志状态的信息。`Status`信息以逗号分隔的名称/值对形式呈现。

`connection`行的`Status`列包含以下表中描述的名称/值对。

| 名称 | 值 |
| --- | --- |
| `cluster_node_id` | 集群中 MySQL 服务器的节点 ID |
| `connected_host` | MySQL 服务器连接的集群管理服务器的主机名或 IP 地址 |
| `connected_port` | MySQL 服务器用于连接管理服务器(`connected_host`)的端口号 |
| `number_of_data_nodes` | 集群配置的数据节点数量（即，集群`config.ini`文件中`[ndbd]`部分的数量） |
| `number_of_ready_data_nodes` | 集群中实际运行的数据节点数量 |
| `connect_count` | 此**mysqld**连接或重新连接到集群数据节点的次数 |

`binlog`行的`Status`列包含与 NDB 集群复制相关的信息。它包含的名称/值对在以下表中描述。

| 名称 | 值 |
| --- | --- |
| `latest_epoch` | 此 MySQL 服务器上最近运行的最新时代（即，服务器上最近运行的最新事务的序列号） |
| `latest_trans_epoch` | 集群数据节点处理的最新时代 |
| `latest_received_binlog_epoch` | 二进制日志线程接收的最新时代 |
| `latest_handled_binlog_epoch` | 二进制日志线程处理的最新时代（用于写入二进制日志） |
| `latest_applied_binlog_epoch` | 实际写入二进制日志的最新时代 |

更多信息请参阅第 25.7 节，“NDB 集群复制”。

`SHOW ENGINE NDB STATUS`输出中剩余的行最有可能在监视集群时证明有用，按`Name`列在此处列出：

+   `NdbTransaction`：已创建的`NdbTransaction`对象的数量和大小。每次在`NDB`表上执行表模式操作（如`CREATE TABLE`或`ALTER TABLE`）时都会创建一个`NdbTransaction`。

+   `NdbOperation`: 已创建的`NdbOperation`对象的数量和大小。

+   `NdbIndexScanOperation`: 已创建的`NdbIndexScanOperation`对象的数量和大小。

+   `NdbIndexOperation`: 已创建的`NdbIndexOperation`对象的数量和大小。

+   `NdbRecAttr`: 已创建的`NdbRecAttr`对象的数量和大小。通常，每当由 SQL 节点执行数据操作语句时，就会创建一个`NdbRecAttr`。

+   `NdbBlob`: 已创建的`NdbBlob`对象的数量和大小。每当涉及`NDB`表中的`BLOB`列的新操作时，就会创建一个`NdbBlob`。

+   `NdbReceiver`: 已创建的任何`NdbReceiver`对象的数量和大小。`created`列中的数字与 MySQL 服务器连接的集群中的数据节点数量相同。

注意

如果在当前会话中，通过访问运行此语句的 SQL 节点的 MySQL 客户端执行了涉及`NDB`表的操作，则`SHOW ENGINE NDB STATUS`将返回空结果。
