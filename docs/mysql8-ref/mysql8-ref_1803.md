> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-restart-info.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-restart-info.html)

#### 25.6.16.52 ndbinfo restart_info 表

`restart_info`表包含有关节点重新启动操作的信息。表中的每个条目对应于具有给定节点 ID 的数据节点实时的节点重新启动状态报告。仅显示任何给定节点的最新报告。

`restart_info`表包含以下列：

+   `node_id`

    集群中的节点 ID

+   `node_restart_status`

    节点状态；请参阅文本以获取值。每个值对应于`node_restart_status_int`的可能值。

+   `node_restart_status_int`

    节点状态代码；请参阅文本以获取值。

+   `secs_to_complete_node_failure`

    完成节点故障处理所需的秒数

+   `secs_to_allocate_node_id`

    从节点故障完成到分配节点 ID 所需的秒数

+   `secs_to_include_in_heartbeat_protocol`

    从分配节点 ID 到包含在心跳协议中所花费的秒数

+   `secs_until_wait_for_ndbcntr_master`

    从包含在心跳协议中到等待`NDBCNTR`主节点开始等待的秒数

+   `secs_wait_for_ndbcntr_master`

    等待被`NDBCNTR`主节点接受开始的秒数

+   `secs_to_get_start_permitted`

    从主节点接收启动许可到所有节点接受此节点启动所花费的秒数

+   `secs_to_wait_for_lcp_for_copy_meta_data`

    等待复制元数据之前等待 LCP 完成所花费的秒数

+   `secs_to_copy_meta_data`

    从主节点复制元数据到新开始节点所需的秒数

+   `secs_to_include_node`

    等待 GCP 和所有节点加入协议的秒数

+   `secs_starting_node_to_request_local_recovery`

    刚开始的节点等待请求本地恢复所花费的秒数

+   `secs_for_local_recovery`

    刚开始的节点本地恢复所需的秒数

+   `secs_restore_fragments`

    从 LCP 文件中恢复片段所需的秒数

+   `secs_undo_disk_data`

    执行在磁盘数据部分上的撤销日志所需的秒数

+   `secs_exec_redo_log`

    执行重做日志在所有恢复片段上所需的秒数

+   `secs_index_rebuild`

    重建恢复片段上的索引所需的秒数

+   `secs_to_synchronize_starting_node`

    从活动节点同步开始节点所需的秒数

+   `secs_wait_lcp_for_restart`

    完成重新启动之前开始和完成 LCP 所需的秒数

+   `secs_wait_subscription_handover`

    等待复制元数据的秒数

+   `total_restart_secs`

    从节点故障到节点再次启动的总秒数

##### 备注

以下列表包含了`node_restart_status_int`列中定义的值及其内部状态名称（括号中），以及在`node_restart_status`列中显示的相应消息：

+   `0` (`ALLOCATED_NODE_ID`)

    `分配的节点 ID`

+   `1` (`INCLUDED_IN_HB_PROTOCOL`)

    `包含在心跳协议中`

+   `2` (`NDBCNTR_START_WAIT`)

    `等待 NDBCNTR 主节点允许我们启动`

+   `3` (`NDBCNTR_STARTED`)

    `NDBCNTR 主节点允许我们启动`

+   `4` (`START_PERMITTED`)

    `所有节点允许我们启动`

+   `5` (`WAIT_LCP_TO_COPY_DICT`)

    `等待 LCP 完成以开始复制元数据`

+   `6` (`COPY_DICT_TO_STARTING_NODE`)

    `正在将元数据复制到起始节点`

+   `7` (`INCLUDE_NODE_IN_LCP_AND_GCP`)

    `将节点包含在 LCP 和 GCP 协议中`

+   `8` (`LOCAL_RECOVERY_STARTED`)

    `正在恢复片段`

+   `9` (`COPY_FRAGMENTS_STARTED`)

    `正在将起始节点与活动节点同步`

+   `10` (`WAIT_LCP_FOR_RESTART`)

    `等待 LCP 以确保持久性`

+   `11` (`WAIT_SUMA_HANDOVER`)

    `等待订阅切换`

+   `12` (`RESTART_COMPLETED`)

    `重启已完成`

+   `13` (`NODE_FAILED`)

    `节点失败，故障处理正在进行中`

+   `14` (`NODE_FAILURE_COMPLETED`)

    `节点故障处理已完成`

+   `15` (`NODE_GETTING_PERMIT`)

    `所有节点允许我们启动`

+   `16` (`NODE_GETTING_INCLUDED`)

    `将节点包含在 LCP 和 GCP 协议中`

+   `17` (`NODE_GETTING_SYNCHED`)

    `正在将起始节点与活动节点同步`

+   `18` (`NODE_GETTING_LCP_WAITED`)

    [无]

+   `19` (`NODE_ACTIVE`)

    `重启已完成`

+   `20` (`NOT_DEFINED_IN_CLUSTER`)

    [无]

+   `21` (`NODE_NOT_RESTARTED_YET`)

    `初始状态`

状态编号 0 到 12 仅适用于主节点；表中显示的其余状态编号适用于所有重新启动的数据节点。状态编号 13 和 14 定义了节点故障状态；20 和 21 发生在没有关于给定节点重新启动的信息时。

另请参阅 Section 25.6.4, “NDB Cluster 启动阶段总结”。
