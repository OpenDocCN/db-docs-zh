# 17.8.11 配置索引页面的合并阈值

> 原文：[`dev.mysql.com/doc/refman/8.0/en/index-page-merge-threshold.html`](https://dev.mysql.com/doc/refman/8.0/en/index-page-merge-threshold.html)

您可以配置索引页面的`MERGE_THRESHOLD`值。如果索引页面的“page-full”百分比低于`MERGE_THRESHOLD`值，当删除行或通过`UPDATE`操作缩短行时，`InnoDB`会尝试将索引页面与相邻的索引页面合并。默认的`MERGE_THRESHOLD`值为 50，这是先前硬编码的值。最小的`MERGE_THRESHOLD`值为 1，最大值为 50。

当索引页面的“page-full”百分比低于默认的`MERGE_THRESHOLD`设置值 50%时，`InnoDB`会尝试将索引页面与相邻页面合并。如果两个页面都接近 50%满，页面合并后很快可能会发生页面分裂。如果此合并-分裂行为频繁发生，可能会对性能产生不利影响。为避免频繁的合并-分裂，您可以降低`MERGE_THRESHOLD`值，以便`InnoDB`在较低的“page-full”百分比下尝试页面合并。在较低的页面满百分比下合并页面会在索引页面中留下更多空间，并有助于减少合并-分裂行为。

可以为表或单个索引定义索引页面的`MERGE_THRESHOLD`。为单个索引定义的`MERGE_THRESHOLD`值优先于为表定义的`MERGE_THRESHOLD`值。如果未定义，`MERGE_THRESHOLD`值默认为 50。

#### 为表设置`MERGE_THRESHOLD`

您可以使用`CREATE TABLE`语句的*`table_option`* `COMMENT`子句为表设置`MERGE_THRESHOLD`值。例如：

```sql
CREATE TABLE t1 (
   id INT,
  KEY id_index (id)
) COMMENT='MERGE_THRESHOLD=45';
```

您还可以使用`ALTER TABLE`的*`table_option`* `COMMENT`子句为现有表设置`MERGE_THRESHOLD`值：

```sql
CREATE TABLE t1 (
   id INT,
  KEY id_index (id)
);

ALTER TABLE t1 COMMENT='MERGE_THRESHOLD=40';
```

#### 为单个索引设置`MERGE_THRESHOLD`

要为单个索引设置`MERGE_THRESHOLD`值，可以使用`CREATE TABLE`、`ALTER TABLE`或`CREATE INDEX`中的*`index_option`* `COMMENT`子句，如下例所示：

+   使用`CREATE TABLE`为单个索引设置`MERGE_THRESHOLD`：

    ```sql
    CREATE TABLE t1 (
       id INT,
      KEY id_index (id) COMMENT 'MERGE_THRESHOLD=40'
    );
    ```

+   使用`ALTER TABLE`为单个索引设置`MERGE_THRESHOLD`：

    ```sql
    CREATE TABLE t1 (
       id INT,
      KEY id_index (id)
    );

    ALTER TABLE t1 DROP KEY id_index;
    ALTER TABLE t1 ADD KEY id_index (id) COMMENT 'MERGE_THRESHOLD=40';
    ```

+   使用`CREATE INDEX`为单个索引设置`MERGE_THRESHOLD`：

    ```sql
    CREATE TABLE t1 (id INT);
    CREATE INDEX id_index ON t1 (id) COMMENT 'MERGE_THRESHOLD=40';
    ```

注意

无法修改`GEN_CLUST_INDEX`的索引级别的`MERGE_THRESHOLD`值，`GEN_CLUST_INDEX`是`InnoDB`在创建没有主键或唯一键索引的`InnoDB`表时创建的聚簇索引。只能通过为表设置`MERGE_THRESHOLD`来修改`GEN_CLUST_INDEX`的`MERGE_THRESHOLD`值。

#### 查询索引的`MERGE_THRESHOLD`值

可通过查询`INNODB_INDEXES`表来获取索引的当前`MERGE_THRESHOLD`值。例如：

```sql
mysql> SELECT * FROM INFORMATION_SCHEMA.INNODB_INDEXES WHERE NAME='id_index' \G
*************************** 1\. row ***************************
       INDEX_ID: 91
           NAME: id_index
       TABLE_ID: 68
           TYPE: 0
       N_FIELDS: 1
        PAGE_NO: 4
          SPACE: 57
MERGE_THRESHOLD: 40
```

您可以使用`SHOW CREATE TABLE`来查看表的`MERGE_THRESHOLD`值，如果使用*`table_option`* `COMMENT`子句明确定义：

```sql
mysql> SHOW CREATE TABLE t2 \G
*************************** 1\. row ***************************
       Table: t2
Create Table: CREATE TABLE `t2` (
  `id` int(11) DEFAULT NULL,
  KEY `id_index` (`id`) COMMENT 'MERGE_THRESHOLD=40'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci
```

注意

在索引级别定义的`MERGE_THRESHOLD`值优先于为表定义的`MERGE_THRESHOLD`值。如果未定义，`MERGE_THRESHOLD`默认为 50%（`MERGE_THRESHOLD=50`，这是先前硬编码的值。

同样，您可以使用`SHOW INDEX`来查看索引的`MERGE_THRESHOLD`值，如果使用*`index_option`* `COMMENT`子句明确定义：

```sql
mysql> SHOW INDEX FROM t2 \G
*************************** 1\. row ***************************
        Table: t2
   Non_unique: 1
     Key_name: id_index
 Seq_in_index: 1
  Column_name: id
    Collation: A
  Cardinality: 0
     Sub_part: NULL
       Packed: NULL
         Null: YES
   Index_type: BTREE
      Comment:
Index_comment: MERGE_THRESHOLD=40
```

#### 测量`MERGE_THRESHOLD`设置的效果

`INNODB_METRICS`表提供两个计数器，可用于衡量`MERGE_THRESHOLD`设置对索引页面合并的影响。

```sql
mysql> SELECT NAME, COMMENT FROM INFORMATION_SCHEMA.INNODB_METRICS
       WHERE NAME like '%index_page_merge%';
+-----------------------------+----------------------------------------+
| NAME                        | COMMENT                                |
+-----------------------------+----------------------------------------+
| index_page_merge_attempts   | Number of index page merge attempts    |
| index_page_merge_successful | Number of successful index page merges |
+-----------------------------+----------------------------------------+
```

降低`MERGE_THRESHOLD`值时的目标是：

+   较少数量的页面合并尝试和成功的页面合并

+   一样数量的页面合并尝试和成功的页面合并

如果`MERGE_THRESHOLD`设置过小，可能会导致数据文件过大，因为空页面空间过多。

有关使用`INNODB_METRICS`计数器的信息，请参阅第 17.15.6 节，“InnoDB INFORMATION_SCHEMA Metrics Table”。
