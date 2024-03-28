# 28.4.13 `INFORMATION_SCHEMA` `INNODB_FOREIGN_COLS` 表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/information-schema-innodb-foreign-cols-table.html`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-innodb-foreign-cols-table.html)

`INNODB_FOREIGN_COLS` 表提供关于`InnoDB`外键列的状态信息。

有关使用信息和示例，请参见第 17.15.3 节，“InnoDB INFORMATION_SCHEMA 模式对象表”。

`INNODB_FOREIGN_COLS` 表具有以下列：

+   `ID`

    与此索引键字段关联的外键索引；与`INNODB_FOREIGN.ID`相同的值。

+   `FOR_COL_NAME`

    子表中关联列的名称。

+   `REF_COL_NAME`

    父表中关联列的名称。

+   `POS`

    此键字段在外键索引中的顺序位置，从 0 开始。

#### 示例

```sql
mysql> SELECT * FROM INFORMATION_SCHEMA.INNODB_FOREIGN_COLS WHERE ID = 'test/fk1'\G
*************************** 1\. row ***************************
          ID: test/fk1
FOR_COL_NAME: parent_id
REF_COL_NAME: id
         POS: 0
```

#### 注意

+   您必须具有`PROCESS`权限才能查询此表。

+   使用`INFORMATION_SCHEMA` `COLUMNS` 表或`SHOW COLUMNS`语句查看有关此表的列的其他信息，包括数据类型和默认值。
