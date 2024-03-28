> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-tc-time-track-stats.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-tc-time-track-stats.html)

#### 25.6.16.60 The ndbinfo tc_time_track_stats Table

`tc_time_track_stats` 表提供从数据节点中的 `DBTC` 块（TC）实例中获取的时间跟踪信息，通过 API 节点访问 `NDB`。每个 TC 实例跟踪它代表 API 节点或其他数据节点执行的一组活动的延迟；这些活动包括事务、事务错误、键读取、键写入、唯一索引操作、任何类型的失败键操作、扫描、失败扫描、片段扫描和失败片段扫描。

为每种活动维护一组计数器，每个计数器覆盖小于或等于上限值的延迟范围。在每个活动结束时，确定其延迟并适当地递增计数器。`tc_time_track_stats` 将此信息呈现为行，每个实例对应一行：

+   数据节点，使用其 ID

+   TC 块实例

+   其他通信数据节点或 API 节点，使用其 ID

+   上限值

每行包含每种活动类型的值。这是该活动发生的次数，其延迟在行指定的范围内（即，延迟不超过上限值）。

`tc_time_track_stats` 表包含以下列：

+   `node_id`

    请求节点 ID

+   `block_number`

    TC 块编号

+   `block_instance`

    TC 块实例编号

+   `comm_node_id`

    通信 API 或数据节点的节点 ID

+   `upper_bound`

    区间的上限值（以微秒为单位）

+   `scans`

    基于从打开到关闭的成功扫描持续时间，针对请求它们的 API 或数据节点进行跟踪。

+   `scan_errors`

    基于从打开到关闭的失败扫描持续时间，针对请求它们的 API 或数据节点进行跟踪。

+   `scan_fragments`

    基于从打开到关闭的成功片段扫描持续时间，针对执行它们的数据节点进行跟踪。

+   `scan_fragment_errors`

    基于从打开到关闭的失败片段扫描持续时间，针对执行它们的数据节点进行跟踪。

+   `transactions`

    基于从开始到发送提交`ACK`的成功事务持续时间，针对请求它们的 API 或数据节点进行跟踪。无状态事务不包括在内。

+   `transaction_errors`

    基于从开始到失败点的失败事务持续时间，针对请求它们的 API 或数据节点进行跟踪。

+   `read_key_ops`

    基于带锁的成功主键读取的持续时间。针对请求它们的 API 或数据节点以及执行它们的数据节点进行跟踪。

+   `write_key_ops`

    基于成功主键写入的持续时间，针对请求它们的 API 或数据节点以及执行它们的数据节点进行跟踪。

+   `index_key_ops`

    基于成功唯一索引键操作的持续时间，跟踪 API 或数据节点请求它们以及执行基表读取的数据节点。

+   `key_op_errors`

    基于所有不成功的键读取或写入操作的持续时间，跟踪 API 或数据节点请求它们以及执行它们的数据节点。

##### 注意

`block_instance`列提供了`DBTC`内核块实例编号。您可以将其与块名称一起使用，从`threadblocks`表中获取有关特定线程的信息。
