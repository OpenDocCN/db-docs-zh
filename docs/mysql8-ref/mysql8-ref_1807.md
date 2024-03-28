> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-table-distribution-status.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-table-distribution-status.html)

#### 25.6.16.56 ndbinfo 表`table_distribution_status`

`table_distribution_status`表提供了关于`NDB`表分布进度的信息。

`table_distribution_status`表包含以下列：

+   `node_id`

    节点 id

+   `table_id`

    表 ID

+   `tab_copy_status`

    将表分布数据复制到磁盘的状态；其中之一为`IDLE`、`SR_PHASE1_READ_PAGES`、`SR_PHASE2_READ_TABLE`、`SR_PHASE3_COPY_TABLE`、`REMOVE_NODE`、`LCP_READ_TABLE`、`COPY_TAB_REQ`、`COPY_NODE_STATE`、`ADD_TABLE_COORDINATOR`（*在 NDB 8.0.23 之前*：`ADD_TABLE_MASTER`）、`ADD_TABLE_PARTICIPANT`（*在 NDB 8.0.23 之前*：`ADD_TABLE_SLAVE`）、`INVALIDATE_NODE_LCP`、`ALTER_TABLE`、`COPY_TO_SAVE`或`GET_TABINFO`

+   `tab_update_status`

    表分布数据更新状态；其中之一为`IDLE`、`LOCAL_CHECKPOINT`、`LOCAL_CHECKPOINT_QUEUED`、`REMOVE_NODE`、`COPY_TAB_REQ`、`ADD_TABLE_MASTER`、`ADD_TABLE_SLAVE`、`INVALIDATE_NODE_LCP`或`CALLBACK`

+   `tab_lcp_status`

    表 LCP 的状态；其中之一为`ACTIVE`（等待执行本地检查点）、`WRITING_TO_FILE`（检查点已执行但尚未写入磁盘）或`COMPLETED`（检查点已执行并持久化到磁盘）

+   `tab_status`

    表内部状态；其中之一为`ACTIVE`（表存在）、`CREATING`（表正在创建）或`DROPPING`（表正在删除）

+   `tab_storage`

    表可恢复性；其中之一为`NORMAL`（通过重做日志和检查点完全可恢复），`NOLOGGING`（从节点崩溃中恢复，空集群崩溃后恢复），或`TEMPORARY`（不可恢复）

+   `tab_partitions`

    表中的分区数

+   `tab_fragments`

    表中的片段数；通常与`tab_partitions`相同；对于完全复制的表，等于`tab_partitions * [节点组数]`

+   `current_scan_count`

    当前活动扫描数

+   `scan_count_wait`

    当前等待执行的扫描数，直到`ALTER TABLE`完成。

+   `is_reorg_ongoing`

    表当前是否正在重新组织（如果是则为 1）
