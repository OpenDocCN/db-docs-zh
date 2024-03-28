> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-schema-redundant-indexes.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-schema-redundant-indexes.html)

#### 30.4.3.27 模式冗余索引和 x$schema_flattened_keys 视图

`schema_redundant_indexes` 视图显示重复其他索引或被其他索引冗余的索引。`x$schema_flattened_keys` 视图是`schema_redundant_indexes` 的辅助视图。

在以下列描述中，主导索引是使冗余索引冗余的索引。

`schema_redundant_indexes` 视图包含以下列：

+   `table_schema`

    包含表的模式。

+   `table_name`

    包含索引的表。

+   `redundant_index_name`

    冗余索引的名称。

+   `redundant_index_columns`

    冗余索引中列的名称。

+   `redundant_index_non_unique`

    冗余索引中非唯一列的数量。

+   `dominant_index_name`

    主导索引的名称。

+   `dominant_index_columns`

    主导索引中列的名称。

+   `dominant_index_non_unique`

    主导索引中非唯一列的数量。

+   `subpart_exists`

    索引是否仅索引列的一部分。

+   `sql_drop_index`

    执行以删除冗余索引的语句。

`x$schema_flattened_keys` 视图包含以下列：

+   `table_schema`

    包含表的模式。

+   `table_name`

    包含索引的表。

+   `index_name`

    索引名称。

+   `non_unique`

    索引中非唯一列的数量。

+   `subpart_exists`

    索引是否仅索引列的一部分。

+   `index_columns`

    索引中列的名称。
