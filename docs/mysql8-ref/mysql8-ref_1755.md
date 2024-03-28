> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-blobs.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-blobs.html)

#### 25.6.16.4 ndbinfo blobs 表

此表提供了存储在`NDB`中的 blob 值的信息。`blobs`表包含以下列：

+   `table_id`

    包含列的表的唯一 ID

+   `database_name`

    此表所在的数据库的名称

+   `table_name`

    表的名称

+   `column_id`

    表内此列的唯一 ID

+   `column_name`

    列的名称

+   `inline_size`

    列的内联大小

+   `part_size`

    列的部分大小

+   `stripe_size`

    列的条带大小

+   `blob_table_name`

    包含此列 blob 数据的 blob 表的名称（如果有）

此表中存在行，用于存储超过 255 字节的`BLOB`、`TEXT`值的`NDB`表列，因此需要使用 blob 表。超过 4000 字节的`JSON`值的部分也存储在此表中。有关 NDB 集群如何存储此类类型列的更多信息，请参阅字符串类型存储要求。

可以使用包含`NDB`表列注释的`CREATE TABLE`和`ALTER TABLE`语句设置`NDB` blob 列的部分和（NDB 8.0.30 及更高版本）内联大小（请参阅 NDB_COLUMN 选项）；这也可以在 NDB API 应用程序中完成（请参阅`Column::setPartSize()`和`setInlineSize()`）。

`blobs`表在 NDB 8.0.29 中添加。
