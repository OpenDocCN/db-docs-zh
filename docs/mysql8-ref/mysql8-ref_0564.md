# 10.3.13 降序索引

> 原文：[`dev.mysql.com/doc/refman/8.0/en/descending-indexes.html`](https://dev.mysql.com/doc/refman/8.0/en/descending-indexes.html)

MySQL 支持降序索引：在索引定义中的`DESC`不再被忽略，而是导致键值以降序存储。以前，索引可以以相反顺序扫描，但会导致性能损失。降序索引可以以正向顺序扫描，这更有效率。降序索引还使优化器能够在最有效的扫描顺序中混合使用多列索引，其中一些列按升序顺序排列，而其他列按降序顺序排列。

考虑以下表定义，其中包含两列和四个两列索引定义，用于列的各种升序和降序索引组合：

```sql
CREATE TABLE t (
  c1 INT, c2 INT,
  INDEX idx1 (c1 ASC, c2 ASC),
  INDEX idx2 (c1 ASC, c2 DESC),
  INDEX idx3 (c1 DESC, c2 ASC),
  INDEX idx4 (c1 DESC, c2 DESC)
);
```

表定义会产生四个不同的索引。优化器可以对每个`ORDER BY`子句执行正向索引扫描，而无需使用`filesort`操作：

```sql
ORDER BY c1 ASC, c2 ASC    -- optimizer can use idx1
ORDER BY c1 DESC, c2 DESC  -- optimizer can use idx4
ORDER BY c1 ASC, c2 DESC   -- optimizer can use idx2
ORDER BY c1 DESC, c2 ASC   -- optimizer can use idx3
```

使用降序索引需满足以下条件：

+   仅支持`InnoDB`存储引擎的降序索引，具有以下限制：

    +   如果索引包含降序索引键列或主键包含降序索引列，则辅助索引不支持更改缓冲。

    +   `InnoDB` SQL 解析器不使用降序索引。对于`InnoDB`全文搜索，这意味着索引表的`FTS_DOC_ID`列上所需的索引不能定义为降序索引。有关更多信息，请参阅 Section 17.6.2.4, “InnoDB Full-Text Indexes”。

+   支持所有可用升序索引的数据类型的降序索引。

+   支持普通（非生成的）和生成列（`VIRTUAL`和`STORED`）的降序索引。

+   `DISTINCT`可以使用包含匹配列的任何索引，包括降序键部分。

+   具有降序键部分的索引不会用于调用聚合函数但没有`GROUP BY`子句的查询的`MIN()`/`MAX()`优化。

+   支持`BTREE`但不支持`HASH`索引的降序索引。不支持`FULLTEXT`或`SPATIAL`索引的降序索引。

    明确指定`HASH`、`FULLTEXT`和`SPATIAL`索引的`ASC`和`DESC`标识符会导致错误。

您可以在`EXPLAIN`输出的**`Extra`**列中看到优化器能够使用降序索引，如下所示：

```sql
mysql> CREATE TABLE t1 (
 -> a INT, 
 -> b INT, 
 -> INDEX a_desc_b_asc (a DESC, b ASC)
 -> );

mysql> EXPLAIN SELECT * FROM t1 ORDER BY a ASC\G
*************************** 1\. row ***************************
           id: 1
  select_type: SIMPLE
        table: t1
   partitions: NULL
         type: index
possible_keys: NULL
          key: a_desc_b_asc
      key_len: 10
          ref: NULL
         rows: 1
     filtered: 100.00
        Extra: Backward index scan; Using index
```

在`EXPLAIN FORMAT=TREE`输出中，使用降序索引会在索引名称后面添加`(reverse)`，如下所示：

```sql
mysql> EXPLAIN FORMAT=TREE SELECT * FROM t1 ORDER BY a ASC\G 
*************************** 1\. row ***************************
EXPLAIN: -> Index scan on t1 using a_desc_b_asc (reverse)  (cost=0.35 rows=1)
```

另请参阅 EXPLAIN Extra Information。
