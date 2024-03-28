> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-schema-index-statistics.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-schema-index-statistics.html)

#### 30.4.3.25 `schema_index_statistics` 和 `x$schema_index_statistics` 视图

这些视图提供索引统计信息。默认情况下，按照总索引延迟降序排序行。

`schema_index_statistics` 和 `x$schema_index_statistics` 视图具有以下列：

+   `table_schema`

    包含表的模式。

+   `table_name`

    包含索引的表。

+   `index_name`

    索引的名称。

+   `rows_selected`

    使用索引读取的总行数。

+   `select_latency`

    使用索引进行定时读取的总等待时间。

+   `rows_inserted`

    插入到索引中的总行数。

+   `insert_latency`

    插入到索引中的定时等待总时间。

+   `rows_updated`

    在索引中更新的总行数。

+   `update_latency`

    在索引中进行定时更新的总等待时间。

+   `rows_deleted`

    从索引中删除的总行数。

+   `delete_latency`

    从索引中删除的定时等待总时间。
