> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-analyze-table-complexity.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-analyze-table-complexity.html)

#### 17.8.10.3 估计 InnoDB 表的 ANALYZE TABLE 复杂性

`ANALYZE TABLE`复杂性对`InnoDB`表取决于：

+   采样的页面数，由`innodb_stats_persistent_sample_pages`定义。

+   表中索引列的数量

+   分区的数量。如果表没有分区，则分区的数量被视为 1。

使用这些参数，估计`ANALYZE TABLE`复杂性的近似公式将是：

`innodb_stats_persistent_sample_pages`的值 * 表中索引列的数量 * 表中分区的数量

通常，结果值越大，执行`ANALYZE TABLE`所需的时间越长。

注意

`innodb_stats_persistent_sample_pages`定义了在全局级别采样的页面数。要为单个表设置采样页面数，请在`CREATE TABLE`或`ALTER TABLE`中使用`STATS_SAMPLE_PAGES`选项。更多信息，请参见第 17.8.10.1 节，“配置持久性优化器统计参数”。

如果`innodb_stats_persistent=OFF`，则采样的页面数由`innodb_stats_transient_sample_pages`定义。有关更多信息，请参见第 17.8.10.2 节，“配置非持久性优化器统计参数”。

要更深入地估计`ANALYZE TABLE`复杂性，请考虑以下示例。

在[大 O 符号](http://en.wikipedia.org/wiki/Big_O_notation)中，`ANALYZE TABLE`的复杂性描述为：

```sql
 O(n_sample
  * (n_cols_in_uniq_i
     + n_cols_in_non_uniq_i
     + n_cols_in_pk * (1 + n_non_uniq_i))
  * n_part)
```

其中：

+   `n_sample`是采样的页面数（由`innodb_stats_persistent_sample_pages`定义）

+   `n_cols_in_uniq_i`是所有唯一索引中所有列的总数（不包括主键列）

+   `n_cols_in_non_uniq_i`是所有非唯一索引中所有列的总数

+   `n_cols_in_pk`是主键中的列数（如果未定义主键，则`InnoDB`在内部创建单列主键）

+   `n_non_uniq_i`是表中非唯一索引的数量

+   `n_part`是分区的数量。如果未定义分区，则认为表是单个分区。

现在，考虑以下表（表`t`），它具有主键（2 列），唯一索引（2 列）和两个非唯一索引（每个两列）：

```sql
CREATE TABLE t (
  a INT,
  b INT,
  c INT,
  d INT,
  e INT,
  f INT,
  g INT,
  h INT,
  PRIMARY KEY (a, b),
  UNIQUE KEY i1uniq (c, d),
  KEY i2nonuniq (e, f),
  KEY i3nonuniq (g, h)
);
```

对于上述算法所需的列和索引数据，请查询表`t`的`mysql.innodb_index_stats`持久性索引统计表。`n_diff_pfx%`统计数据显示了为每个索引计算的列。例如，主键索引计算了列`a`和`b`。对于非唯一索引，除了用户定义的列外，还计算了主键列（a，b）。

注意

有关`InnoDB`持久性统计表的其他信息，请参见第 17.8.10.1 节，“配置持久性优化器统计参数”

```sql
mysql> SELECT index_name, stat_name, stat_description
       FROM mysql.innodb_index_stats WHERE
       database_name='test' AND
       table_name='t' AND
       stat_name like 'n_diff_pfx%';
 +------------+--------------+------------------+
 | index_name | stat_name    | stat_description |
 +------------+--------------+------------------+
 | PRIMARY    | n_diff_pfx01 | a                |
 | PRIMARY    | n_diff_pfx02 | a,b              |
 | i1uniq     | n_diff_pfx01 | c                |
 | i1uniq     | n_diff_pfx02 | c,d              |
 | i2nonuniq  | n_diff_pfx01 | e                |
 | i2nonuniq  | n_diff_pfx02 | e,f              |
 | i2nonuniq  | n_diff_pfx03 | e,f,a            |
 | i2nonuniq  | n_diff_pfx04 | e,f,a,b          |
 | i3nonuniq  | n_diff_pfx01 | g                |
 | i3nonuniq  | n_diff_pfx02 | g,h              |
 | i3nonuniq  | n_diff_pfx03 | g,h,a            |
 | i3nonuniq  | n_diff_pfx04 | g,h,a,b          |
 +------------+--------------+------------------+
```

根据上述索引统计数据和表定义，可以确定以下值：

+   `n_cols_in_uniq_i`，所有唯一索引中所有列的总数，不包括主键列，为 2（`c`和`d`）

+   `n_cols_in_non_uniq_i`，所有非唯一索引中所有列的总数为 4（`e`，`f`，`g`和`h`）

+   `n_cols_in_pk`，主键中的列数为 2（`a`和`b`）

+   `n_non_uniq_i`，表中非唯一索引的数量为 2（`i2nonuniq`和`i3nonuniq`）

+   `n_part`，分区的数量为 1。

现在，您可以计算`innodb_stats_persistent_sample_pages` *（2 + 4 + 2 *（1 + 2））* 1 来确定扫描的叶页数。假设`innodb_stats_persistent_sample_pages`设置为默认值`20`，默认页面大小为 16 `KiB`（`innodb_page_size`=16384），则可以估计对表`t`读取了 20 * 12 * 16384 `bytes`，约为 4 `MiB`。

注意

可能并非从磁盘读取全部 4 `MiB`，因为一些叶页可能已经缓存在缓冲池中。
