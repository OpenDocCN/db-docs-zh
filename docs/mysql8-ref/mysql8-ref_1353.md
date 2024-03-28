> 原文：[`dev.mysql.com/doc/refman/8.0/en/replication-mode-change-online-concepts.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-mode-change-online-concepts.html)

#### 19.1.4.1 复制模式概念

在设置在线服务器的复制模式之前，了解一些复制的关键概念非常重要。本节解释了这些概念，在尝试修改在线服务器的复制模式之前，这是必读的内容。

MySQL 中可用的复制模式依赖于不同的技术来识别已记录的事务。复制使用的事务类型在此处列出：

+   GTID 事务由全局事务标识符（GTID）标识，其形式为`UUID:NUMBER`。二进制日志中的每个 GTID 事务都以`Gtid_log_event`开头。GTID 事务可以通过其 GTID 或记录在其中的文件的名称及其在该文件中的位置来寻址。

+   匿名事务没有 GTID；MySQL 8.0 确保日志中的每个匿名事务都以`Anonymous_gtid_log_event`开头。（在 MySQL 的先前版本中，匿名事务不会以任何特定事件开头。）匿名事务只能通过文件名和位置来寻址。

使用 GTID 时，您可以利用 GTID 自动定位和自动故障转移，并使用`WAIT_FOR_EXECUTED_GTID_SET()`、`session_track_gtids`和性能模式表来监视复制事务（请参阅第 29.12.11 节，“性能模式复制表”）。

来自运行先前版本的 MySQL 的源的中继日志中的事务可能不会以任何特定事件开头，但在被重放并记录在副本的二进制日志中之后，它将以`Anonymous_gtid_log_event`开头。

要在线更改复制模式，需要使用具有足够权限设置全局系统变量的帐户来设置`gtid_mode`和`enforce_gtid_consistency`变量；请参阅第 7.1.9.1 节，“系统变量权限”。`gtid_mode`的允许值按顺序列在此处，并附有其含义：

+   `OFF`：只有匿名事务可以被复制。

+   `OFF_PERMISSIVE`：新事务是匿名的；复制的事务可以是 GTID 或匿名的。

+   `ON_PERMISSIVE`：新事务使用 GTID；复制的事务可以是 GTID 或匿名的。

+   `ON`：所有事务必须具有 GTID；匿名事务无法被复制。

可以在同一复制拓扑中同时使用匿名事务和 GTID 事务的服务器。例如，一个源，其中`gtid_mode=ON`可以复制到一个副本，其中`gtid_mode=ON_PERMISSIVE`。

`gtid_mode`只能一次更改一步，根据前面列表中显示的值的顺序。例如，如果`gtid_mode`设置为`OFF_PERMISSIVE`，则可以将其更改为`OFF`或`ON_PERMISSIVE`，但不能更改为`ON`。这是为了确保服务器正确处理从匿名事务到 GTID 事务在线更改的过程；GTID 状态（即`gtid_executed`的值）是持久的。这确保了服务器应用的 GTID 设置始终保留并正确，而不受`gtid_mode`值的任何更改影响。

显示 GTID 集的系统变量，如`gtid_executed`和`gtid_purged`，性能模式`replication_connection_status`表的`RECEIVED_TRANSACTION_SET`列，以及`SHOW REPLICA STATUS`输出中与 GTID 相关的结果，当没有 GTID 时都返回空字符串。关于单个 GTID 的信息来源，例如性能模式`replication_applier_status_by_worker`表中显示的`CURRENT_TRANSACTION`列中显示`ANONYMOUS`，当未使用 GTID 事务时。

使用`gtid_mode=ON`从源复制提供了使用 GTID 自动定位的能力，通过`CHANGE REPLICATION SOURCE TO`语句的`SOURCE_AUTO_POSITION`选项进行配置。正在使用的复制拓扑结构会影响是否可以启用自动定位，因为此功能依赖于 GTIDs，并且与匿名事务不兼容。强烈建议在启用自动定位之前确保拓扑结构中没有匿名事务；参见第 19.1.4.2 节，“在线启用 GTID 事务”。

源和副本上 `gtid_mode` 和自动定位的有效组合如下表所示。每个条目的含义如下：

+   `Y`: 源和副本上的 `gtid_mode` 的值兼容。

+   `N`: 源和副本上的 `gtid_mode` 的值不兼容。

+   `*`: 可以使用此值组合进行自动定位。

**表 19.1 源和副本 gtid_mode 的有效组合**

| `gtid_mode` | 源 `OFF` | 源 `OFF_PERMISSIVE` | 源 `ON_PERMISSIVE` | 源 `ON` |
| --- | --- | --- | --- | --- |
| 副本 `OFF` | Y | Y | N | N |
| 副本 `OFF_PERMISSIVE` | Y | Y | Y | Y* |
| 副本 `ON_PERMISSIVE` | Y | Y | Y | Y* |
| 副本 `ON` | N | N | Y | Y* |

当前的 `gtid_mode` 值也会影响 `gtid_next`。下表显示了服务器对不同值的 `gtid_mode` 和 `gtid_next` 组合的行为。每个条目的含义如下：

+   `匿名`: 生成匿名事务。

+   `错误`: 生成错误，并且不执行 `SET GTID_NEXT`。

+   `UUID:NUMBER`: 生成具有指定 UUID:NUMBER 的 GTID。

+   `新 GTID`: 生成带有自动生成编号的 GTID。

**表 19.2 gtid_mode 和 gtid_next 的有效组合**

|  | `gtid_next` 自动二进制日志开启 | `gtid_next` 自动二进制日志关闭 | `gtid_next` 匿名 | `gtid_next` UUID:NUMBER |
| --- | --- | --- | --- | --- |
| `gtid_mode` `OFF` | 匿名 | 匿名 | 匿名 | 错误 |
| `gtid_mode` `OFF_PERMISSIVE` | 匿名 | 匿名 | 匿名 | UUID:NUMBER |
| `gtid_mode` `ON_PERMISSIVE` | 新 GTID | 匿名 | 匿名 | UUID:NUMBER |
| `gtid_mode` `ON` | 新 GTID | 匿名 | 错误 | UUID:NUMBER |

当未使用二进制日志记录且 `gtid_next` 为 `AUTOMATIC` 时，不会生成任何 GTID，这与 MySQL 之前版本的行为一致。
