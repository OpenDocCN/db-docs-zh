> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-table-info.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-table-info.html)

#### 25.6.16.58 ndbinfo 表 table_info

`table_info`表提供了关于个别`NDB`表的日志记录、检查点、分布和存储选项的信息。

`table_info`表包含以下列：

+   `table_id`

    表 ID

+   `logged_table`

    表是否已记录（1）或未记录（0）

+   `row_contains_gci`

    表行是否包含 GCI（1 为真，0 为假）

+   `row_contains_checksum`

    表行是否包含校验和（1 为真，0 为假）

+   `read_backup`

    如果备份片段副本被读取，则为 1，否则为 0

+   `fully_replicated`

    如果表是完全复制的话，这个值为 1，否则为 0

+   `storage_type`

    表存储类型；其中之一为`MEMORY`或`DISK`

+   `hashmap_id`

    哈希映射 ID

+   `partition_balance`

    表的分区平衡（片段计数类型）；其中之一为`FOR_RP_BY_NODE`、`FOR_RA_BY_NODE`、`FOR_RP_BY_LDM`或`FOR_RA_BY_LDM`

+   `create_gci`

    表创建时的 GCI
