> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-schema-unused-indexes.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-schema-unused-indexes.html)

#### 30.4.3.32 The schema_unused_indexes View

这些视图显示没有事件的索引，这表明它们没有被使用。默认情况下，按模式和表对行进行排序。

当服务器运行时间足够长且其工作负载具有代表性时，此视图最有用。否则，此视图中的索引存在可能没有意义。

`schema_unused_indexes` 视图包含以下列：

+   `object_schema`

    模式名称。

+   `object_name`

    表名称。

+   `index_name`

    未使用的索引名称。
