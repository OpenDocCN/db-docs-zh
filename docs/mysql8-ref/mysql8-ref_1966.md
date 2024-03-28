# 28.4.20 The INFORMATION_SCHEMA INNODB_INDEXES Table

> 原文：[`dev.mysql.com/doc/refman/8.0/en/information-schema-innodb-indexes-table.html`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-innodb-indexes-table.html)

`INNODB_INDEXES` 表提供关于 `InnoDB` 索引的元数据。

有关使用信息和示例，请参见 第 17.15.3 节，“InnoDB INFORMATION_SCHEMA Schema Object Tables”。

`INNODB_INDEXES` 表具有以下列：

+   `INDEX_ID`

    索引的标识符。索引标识符在实例中的所有数据库中是唯一的。

+   `NAME`

    索引的名称。大多数由 `InnoDB` 隐式创建的索引具有一致的名称，但索引名称不一定是唯一的。例如：主键索引的 `PRIMARY`，表示主键的索引的 `GEN_CLUST_INDEX`，以及外键约束的 `ID_IND`、`FOR_IND` 和 `REF_IND`。

+   `TABLE_ID`

    表示与索引关联的表的标识符；与 `INNODB_TABLES.TABLE_ID` 相同的值。

+   `TYPE`

    从位级信息派生的数字值，用于标识索引类型。0 = 非唯一二级索引；1 = 自动生成的聚簇索引 (`GEN_CLUST_INDEX`)；2 = 唯一非聚簇索引；3 = 聚簇索引；32 = 全文索引；64 = 空间索引；128 = 虚拟生成列 上的二级索引。

+   `N_FIELDS`

    索引键中的列数。对于 `GEN_CLUST_INDEX` 索引，此值为 0，因为该索引是使用人工值而不是真实表列创建的。

+   `PAGE_NO`

    索引 B 树的根页码。对于全文索引，`PAGE_NO` 列未使用，并设置为 -1 (`FIL_NULL`)，因为全文索引是在几个 B 树（辅助表）中布局的。

+   `SPACE`

    表所在表空间的标识符。0 表示 `InnoDB` 系统表空间。任何其他数字表示以 每表一个文件 模式创建的单独 `.ibd` 文件的表。在 `TRUNCATE TABLE` 语句之后，此标识符保持不变。因为表的所有索引都位于与表相同的表空间中，所以此值不一定是唯一的。

+   `MERGE_THRESHOLD`

    索引页面的合并阈值值。如果索引页面中的数据量低于`MERGE_THRESHOLD`值，当删除一行或通过更新操作缩短一行时，`InnoDB`会尝试将索引页面与相邻的索引页面合并。默认阈值值为 50%。有关更多信息，请参见第 17.8.11 节，“配置索引页面合并阈值”。

#### 示例

```sql
mysql> SELECT * FROM INFORMATION_SCHEMA.INNODB_INDEXES WHERE TABLE_ID = 34\G
*************************** 1\. row ***************************
       INDEX_ID: 39
           NAME: GEN_CLUST_INDEX
       TABLE_ID: 34
           TYPE: 1
       N_FIELDS: 0
        PAGE_NO: 3
          SPACE: 23
MERGE_THRESHOLD: 50
*************************** 2\. row ***************************
       INDEX_ID: 40
           NAME: i1
       TABLE_ID: 34
           TYPE: 0
       N_FIELDS: 1
        PAGE_NO: 4
          SPACE: 23
MERGE_THRESHOLD: 50
```

#### 注意

+   您必须具有`PROCESS`权限才能查询此表。

+   使用`INFORMATION_SCHEMA` `COLUMNS`表或`SHOW COLUMNS`语句查看有关此表的列的其他信息，包括数据类型和默认值。
