> 原文：[`dev.mysql.com/doc/refman/8.0/zh/mysql-cluster-ndbinfo-table-fragments.html`](https://dev.mysql.com/doc/refman/8.0/zh/mysql-cluster-ndbinfo-table-fragments.html)

#### 25.6.16.57 ndbinfo 表 _fragments 表

`table_fragments` 表提供关于 `NDB` 表的分片、分区、分布和（内部）复制的信息。

`table_fragments` 表包含以下列：

+   `node_id`

    节点 ID（`DIH` 主节点）

+   `table_id`

    表 ID

+   `partition_id`

    分区 ID

+   `fragment_id`

    片段 ID（除非表完全复制，否则与分区 ID 相同）

+   `partition_order`

    片段在分区中的顺序

+   `log_part_id`

    片段的日志部分 ID

+   `no_of_replicas`

    片段副本数量

+   `current_primary`

    当前主节点 ID

+   `preferred_primary`

    首选主节点 ID

+   `current_first_backup`

    当前第一备份节点 ID

+   `current_second_backup`

    当前第二备份节点 ID

+   `current_third_backup`

    当前第三备份节点 ID

+   `num_alive_replicas`

    当前活片段副本数量

+   `num_dead_replicas`

    当前死片段副本数量

+   `num_lcp_replicas`

    剩余待检查点的片段副本数量
