# 28.4.11 INFORMATION_SCHEMA INNODB_FIELDS 表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/information-schema-innodb-fields-table.html`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-innodb-fields-table.html)

`INNODB_FIELDS`表提供有关`InnoDB`索引的关键列（字段）的元数据。

有关相关用法信息和示例，请参见第 17.15.3 节，“InnoDB INFORMATION_SCHEMA 模式对象表”。

`INNODB_FIELDS`表具有以下列：

+   `INDEX_ID`

    与此关键字段相关联的索引的标识符；与`INNODB_INDEXES.INDEX_ID`相同的值。

+   `NAME`

    表中原始列的名称；与`INNODB_COLUMNS.NAME`相同的值。

+   `POS`

    关键字段在索引中的顺序位置，从 0 开始递增。当删除列时，剩余列将重新排序，以确保序列没有间隙。

#### 示例

```sql
mysql> SELECT * FROM INFORMATION_SCHEMA.INNODB_FIELDS WHERE INDEX_ID = 117\G
*************************** 1\. row ***************************
INDEX_ID: 117
    NAME: col1
     POS: 0
```

#### 注意事项

+   您必须具有`PROCESS`权限才能查询此表。

+   使用`INFORMATION_SCHEMA` `COLUMNS`表或`SHOW COLUMNS`语句查看有关此表的列的其他信息，包括数据类型和默认值。
