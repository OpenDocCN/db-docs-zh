> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-dictionary-columns.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-dictionary-columns.html)

#### 25.6.16.22 `ndbinfo`字典列表

该表提供有关`NDB`表列的`NDB`字典信息。`dictionary_columns`列在此处列出（带有简要描述）：

+   `table_id`

    包含列的表的 ID

+   `column_id`

    列的唯一 ID

+   `name`

    列名

+   `column_type`

    来自 NDB API 的列的数据类型；请参阅 Column::Type，了解可能的值

+   `default_value`

    列的默认值，如果有的话

+   `nullable`

    `NULL`或`NOT NULL`中的任一值

+   `array_type`

    列的内部属性存储格式；其中之一为`FIXED`、`SHORT_VAR`或`MEDIUM_VAR`；有关更多信息，请参阅 NDB API 文档中的`Column::ArrayType`

+   `storage_type`

    表使用的存储类型；`MEMORY`或`DISK`中的任一值

+   `primary_key`

    如果这是一个主键列，则为`1`，否则为`0`

+   `partition_key`

    如果这是一个分区键列，则为`1`，否则为`0`

+   `dynamic`

    如果列是动态的，则为`1`，否则为`0`

+   `auto_inc`

    如果这是一个`AUTO_INCREMENT`列，则为`1`，否则为`0`

通过将`dictionary_columns`与`dictionary_tables`表连接，您可以获取给定表中所有列的信息，就像这样：

```sql
SELECT dc.*
  FROM dictionary_columns dc
JOIN dictionary_tables dt
  ON dc.table_id=dt.table_id
WHERE dt.table_name='t1'
  AND dt.database_name='mydb';
```

`dictionary_columns`表在 NDB 8.0.29 中添加。

注意

Blob 列不会显示在此表中。这是一个已知问题。
