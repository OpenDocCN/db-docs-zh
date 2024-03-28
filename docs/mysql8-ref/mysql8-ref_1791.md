> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-index-stats.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-index-stats.html)

#### 25.6.16.40 `ndbinfo index_stats` 表

`index_stats` 表提供关于 `NDB` 索引统计的基本信息。

使用 **ndb_index_stat** 实用程序可以获得更完整的索引统计信息。

`index_stats` 表包含以下列：

+   `index_id`

    索引 ID

+   `index_version`

    索引版本

+   `sample_version`

    样本版本

##### 注意

该表在 NDB 8.0.28 版本中添加。
