> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-innodb-buffer-stats-by-schema.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-innodb-buffer-stats-by-schema.html)

#### 30.4.3.7 innodb_buffer_stats_by_schema 和 x$innodb_buffer_stats_by_schema 视图

这些视图总结了 `INFORMATION_SCHEMA` `INNODB_BUFFER_PAGE` 表中按模式分组的信息。默认情况下，按降序缓冲区大小排序行。

警告

查询访问 `INNODB_BUFFER_PAGE` 表的视图可能会影响性能。除非您了解性能影响并确定其可接受，否则不要在生产系统上查询这些视图。为避免影响生产系统的性能，请在测试实例上重现要调查的问题并查询缓冲池统计信息。

`innodb_buffer_stats_by_schema` 和 `x$innodb_buffer_stats_by_schema` 视图具有以下列：

+   `object_schema`

    对象的模式名称，如果表属于 `InnoDB` 存储引擎，则为 `InnoDB System`。

+   `allocated`

    为模式分配的字节的总数。

+   `data`

    为模式分配的数据字节的总数。

+   `pages`

    为模式分配的页的总数。

+   `pages_hashed`

    为模式分配的散列页的总数。

+   `pages_old`

    为模式分配的旧页的总数。

+   `rows_cached`

    为模式缓存的行的总数。
