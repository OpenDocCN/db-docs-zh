> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-innodb-buffer-stats-by-table.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-innodb-buffer-stats-by-table.html)

#### 30.4.3.8 `innodb_buffer_stats_by_table`和`x$innodb_buffer_stats_by_table`视图

这些视图总结了`INFORMATION_SCHEMA` `INNODB_BUFFER_PAGE`表中的信息，按模式和表进行分组。默认情况下，按照缓冲区大小降序排序行。

警告

查询访问`INNODB_BUFFER_PAGE`表的视图可能会影响性能。除非您意识到性能影响并确定其可接受，否则不要在生产系统上查询这些视图。为了避免在生产系统上影响性能，重现您想要调查的问题，并在测试实例上查询缓冲池统计信息。

`innodb_buffer_stats_by_table`和`x$innodb_buffer_stats_by_table`视图具有以下列：

+   `object_schema`

    对象的模式名称，如果表属于`InnoDB`存储引擎，则为`InnoDB System`。

+   `object_name`

    表名。

+   `allocated`

    表格分配的总字节数。

+   `data`

    为表分配的数据字节数。

+   `pages`

    表格分配的总页数。

+   `pages_hashed`

    为表分配的哈希页数。

+   `pages_old`

    为表分配的旧页数。

+   `rows_cached`

    表的缓存行数。
