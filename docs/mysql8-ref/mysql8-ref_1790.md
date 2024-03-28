> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-index-columns.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-index-columns.html)

#### 25.6.16.39 `ndbinfo index_columns`表

该表提供关于`NDB`表上的索引的信息。`index_columns`表的列在此处列出，并附有简要描述：

+   `table_id`

    `NDB`表的唯一 ID，定义了该索引

+   包含此表的数据库的名称

    `varchar(64)`

+   `table_name`

    表的名称

+   `index_object_id`

    此索引的对象 ID

+   `index_name`

    索引的名称；如果索引没有命名，则使用索引中的第一列的名称。

+   `index_type`

    索引类型；通常为 3（唯一哈希索引）或 6（有序索引）；这些值与`dict_obj_types`表中的`type_id`列中的值相同

+   `status`

    其中之一是`new`、`changed`、`retrieved`、`invalid`或`altered`

+   `columns`

    由构成索引的列组成的逗号分隔列表

`index_columns`表在 NDB 8.0.29 中添加。
