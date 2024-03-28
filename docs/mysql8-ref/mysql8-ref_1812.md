> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-threadblocks.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-threadblocks.html)

#### 25.6.16.61 `ndbinfo threadblocks` 表

`threadblocks` 表关联数据节点、线程和 `NDB` 内核块的实例。

`threadblocks` 表包含以下列：

+   `node_id`

    节点 ID

+   `thr_no`

    线程 ID

+   `block_name`

    区块名称

+   `block_instance`

    块实例编号

##### 备注

在这个表中，`block_name` 的值是从 `ndbinfo.blocks` 表中选择时在 `block_name` 列中找到的值之一。虽然在给定的 NDB Cluster 版本中可能值的列表是静态的，但在不同版本之间可能会有所不同。

`block_instance` 列提供了内核块实例编号。
