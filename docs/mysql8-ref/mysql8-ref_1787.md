> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-foreign-keys.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-foreign-keys.html)

#### 25.6.16.36 ndbinfo foreign_keys 表

`foreign_keys` 表提供关于 `NDB` 表上的外键的信息。该表具有以下列：

+   `object_id`

    外键的对象 ID

+   `name`

    外键的名称

+   `parent_table`

    外键的父表的名称

+   `parent_columns`

    父列的逗号分隔列表

+   `child_table`

    子表的名称

+   `child_columns`

    子列的逗号分隔列表

+   `parent_index`

    父索引的名称

+   `child_index`

    子索引的名称

+   `on_update_action`

    `ON UPDATE` 指定的外键操作；其中之一是 `No Action`、`Restrict`、`Cascade`、`Set Null` 或 `Set Default`

+   `on_delete_action`

    `ON DELETE` 指定的外键操作；其中之一是 `No Action`、`Restrict`、`Cascade`、`Set Null` 或 `Set Default`

`foreign_keys` 表在 NDB 8.0.29 中添加。
