> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-table-replicas.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-table-replicas.html)

#### 25.6.16.59 ndbinfo 表 _replicas 表

`table_replicas`表提供有关 NDB 表片段和片段副本的复制、分发和检查点的信息。

`table_replicas`表包含以下列：

+   `node_id`

    从中提取数据的节点的 ID（`DIH`主节点）

+   `table_id`

    表 ID

+   `fragment_id`

    片段 ID

+   `initial_gci`

    表的初始 GCI

+   `replica_node_id`

    存储片段副本的节点的 ID

+   `is_lcp_ongoing`

    如果此片段上正在进行 LCP，则为 1，否则为 0

+   `num_crashed_replicas`

    崩溃片段副本实例的数量

+   `last_max_gci_started`

    最近 LCP 中开始的最高 GCI

+   `last_max_gci_completed`

    最近 LCP 中完成的最高 GCI

+   `last_lcp_id`

    最近 LCP 的 ID

+   `prev_lcp_id`

    上一个 LCP 的 ID

+   `prev_max_gci_started`

    上一个 LCP 中开始的最高 GCI

+   `prev_max_gci_completed`

    上一个 LCP 中完成的最高 GCI

+   `last_create_gci`

    上一个崩溃片段副本实例的最后创建 GCI

+   `last_replica_gci`

    上一个崩溃片段副本实例的最后 GCI

+   `is_replica_alive`

    如果此片段副本存活，则为 1，否则为 0
