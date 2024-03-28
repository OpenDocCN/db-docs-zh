# 28.4.5 INFORMATION_SCHEMA INNODB_CACHED_INDEXES 表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/information-schema-innodb-cached-indexes-table.html`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-innodb-cached-indexes-table.html)

`INNODB_CACHED_INDEXES` 表报告了每个索引在 `InnoDB` 缓冲池中缓存的索引页数。

有关相关用法信息和示例，请参见 Section 17.15.5, “InnoDB INFORMATION_SCHEMA Buffer Pool Tables”。

`INNODB_CACHED_INDEXES` 表具有以下列：

+   `SPACE_ID`

    表空间 ID。

+   `INDEX_ID`

    索引的标识符。索引标识符在实例中的所有数据库中是唯一的。

+   `N_CACHED_PAGES`

    在 `InnoDB` 缓冲池中缓存的索引页数。

#### 示例

此查询返回特定索引在 `InnoDB` 缓冲池中缓存的索引页数：

```sql
mysql> SELECT * FROM INFORMATION_SCHEMA.INNODB_CACHED_INDEXES WHERE INDEX_ID=65\G
*************************** 1\. row ***************************
      SPACE_ID: 4294967294
      INDEX_ID: 65
N_CACHED_PAGES: 45
```

此查询使用 `INNODB_INDEXES` 和 `INNODB_TABLES` 表来解析每个 `INDEX_ID` 值的表名和索引名，返回在 `InnoDB` 缓冲池中缓存的每个索引的索引页数。

```sql
SELECT
  tables.NAME AS table_name,
  indexes.NAME AS index_name,
  cached.N_CACHED_PAGES AS n_cached_pages
FROM
  INFORMATION_SCHEMA.INNODB_CACHED_INDEXES AS cached,
  INFORMATION_SCHEMA.INNODB_INDEXES AS indexes,
  INFORMATION_SCHEMA.INNODB_TABLES AS tables
WHERE
  cached.INDEX_ID = indexes.INDEX_ID
  AND indexes.TABLE_ID = tables.TABLE_ID;
```

#### 注意

+   您必须具有 `PROCESS` 权限才能查询此表。

+   使用 `INFORMATION_SCHEMA` `COLUMNS` 表或 `SHOW COLUMNS` 语句查看有关此表的列的其他信息，包括数据类型和默认值。
