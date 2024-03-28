> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-counters.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-counters.html)

#### 25.6.16.12 ndbinfo 计数器表

`counters`表提供了特定内核块和数据节点的事件（如读取和写入）的累计总数。计数从最近的节点启动或重新启动开始保持；节点的启动或重新启动会重置该节点上的所有计数器。并非所有内核块都具有所有类型的计数器。

`counters`表包含以下列：

+   `node_id`

    数据节点 ID

+   `block_name`

    关联的 NDB 内核块的名称（请参阅 NDB 内核块）。

+   `block_instance`

    区块实例

+   `counter_id`

    计数器的内部 ID 号码；通常为 1 到 10 之间的整数。

+   `counter_name`

    计数器的名称。请参阅各个计数器的名称和每个计数器关联的 NDB 内核块的文本。

+   `val`

    计数器的值

##### 注意事项

每个计数器都与特定的 NDB 内核块相关联。

`OPERATIONS`计数器与`DBLQH`（本地查询处理程序）内核块相关联。主键读取计为一个操作，主键更新也是如此。对于读取，每个`DBTC`中的操作在`DBLQH`中有一个操作。对于写入，每个分片副本计为一个操作。

`ATTRINFO`、`TRANSACTIONS`、`COMMITS`、`READS`、`LOCAL_READS`、`SIMPLE_READS`、`WRITES`、`LOCAL_WRITES`、`ABORTS`、`TABLE_SCANS`和`RANGE_SCANS`计数器与`DBTC`（事务协调器）内核块相关联。

`LOCAL_WRITES`和`LOCAL_READS`是在一个节点中使用事务协调器的主键操作，该节点还持有记录的主分片副本。

`READS`计数器包括所有读取。`LOCAL_READS`仅包括与此事务协调器相同节点上的主分片副本的读取。`SIMPLE_READS`仅包括那些读取，其中读取操作是给定事务的开始和结束操作。简单读取不持有锁，但是作为事务的一部分，它们观察由包含它们的事务进行的未提交更改，而不是任何其他未提交事务。从 TC 块的角度来看，这样的读取是“简单的”；由于它们不持有锁，它们不是持久的，一旦`DBTC`将它们路由到相关的 LQH 块后，它不为它们保留状态。

`ATTRINFO` 记录将解释程序发送到数据节点的次数。有关`NDB`内核中`ATTRINFO`消息的更多信息，请参阅 NDB 协议消息。

`LOCAL_TABLE_SCANS_SENT`、`READS_RECEIVED`、`PRUNED_RANGE_SCANS_RECEIVED`、`RANGE_SCANS_RECEIVED`、`LOCAL_READS_SENT`、`CONST_PRUNED_RANGE_SCANS_RECEIVED`、`LOCAL_RANGE_SCANS_SENT`、`REMOTE_READS_SENT`、`REMOTE_RANGE_SCANS_SENT`、`READS_NOT_FOUND`、`SCAN_BATCHES_RETURNED`、`TABLE_SCANS_RECEIVED` 和 `SCAN_ROWS_RETURNED` 计数器与`DBSPJ`（选择推送连接）内核块相关联。

`block_name` 和 `block_instance` 列分别提供适用的 NDB 内核块名称和实例编号。您可以使用这些信息从`threadblocks`表中获取有关特定线程的信息。

一些计数器提供了有关传输器过载和发送缓冲区大小的信息，用于排除此类问题。对于每个 LQH 实例，在以下列表中的每个计数器中都有一个实例：

+   `LQHKEY_OVERLOAD`: 由于传输器过载而导致 LQH 块实例拒绝主键请求的次数。

+   `LQHKEY_OVERLOAD_TC`: 计算 TC 节点传输器过载的实例数。

+   `LQHKEY_OVERLOAD_READER`: 计算出现 API 读取节点（仅读取）过载的`LQHKEY_OVERLOAD`实例数。

+   `LQHKEY_OVERLOAD_NODE_PEER`: 计算下一个备份数据节点（仅写入）过载的`LQHKEY_OVERLOAD`实例数。

+   `LQHKEY_OVERLOAD_SUBSCRIBER`: 计算出现事件订阅者（仅写入）过载的`LQHKEY_OVERLOAD`实例数。

+   `LQHSCAN_SLOWDOWNS`: 计算由于扫描 API 传输器过载而导致片段扫描批量大小减少的实例数。
