> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-schema-table-statistics-with-buffer.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-schema-table-statistics-with-buffer.html)

#### 30.4.3.30 `schema_table_statistics_with_buffer`和`x$schema_table_statistics_with_buffer`视图

这些视图总结了表的统计信息，包括`InnoDB`缓冲池统计信息。默认情况下，按照总等待时间降序排序（最有争议的表排在前面）。

这些视图使用一个辅助视图，`x$ps_schema_table_statistics_io`。

`schema_table_statistics_with_buffer`和`x$schema_table_statistics_with_buffer`视图具有以下列：

+   `table_schema`

    包含表的模式。

+   `table_name`

    表名。

+   `rows_fetched`

    从表中读取的总行数。

+   `fetch_latency`

    表的定时读取 I/O 事件的总等待时间。

+   `rows_inserted`

    插入表的总行数。

+   `insert_latency`

    表的定时插入 I/O 事件的总等待时间。

+   `rows_updated`

    更新表的总行数。

+   `update_latency`

    表的定时更新 I/O 事件的总等待时间。

+   `rows_deleted`

    从表中删除的总行数。

+   `delete_latency`

    表的定时删除 I/O 事件的总等待时间。

+   `io_read_requests`

    表的读取请求总数。

+   `io_read`

    从表中读取的总字节数。

+   `io_read_latency`

    从表中读取的总等待时间。

+   `io_write_requests`

    为表的写入请求总数。

+   `io_write`

    写入表的总字节数。

+   `io_write_latency`

    写入表的总等待时间。

+   `io_misc_requests`

    表的杂项 I/O 请求总数。

+   `io_misc_latency`

    表的杂项 I/O 请求的总等待时间。

+   `innodb_buffer_allocated`

    为表分配的总`InnoDB`缓冲字节数。

+   `innodb_buffer_data`

    为表分配的总`InnoDB`数据字节数。

+   `innodb_buffer_free`

    为表分配的总`InnoDB`非数据字节数（`innodb_buffer_allocated` - `innodb_buffer_data`）。

+   `innodb_buffer_pages`

    为表分配的总`InnoDB`页面数。

+   `innodb_buffer_pages_hashed`

    为表分配的总`InnoDB`哈希页面数。

+   `innodb_buffer_pages_old`

    为表分配的总`InnoDB`旧页面数。

+   `innodb_buffer_rows_cached`

    表的总`InnoDB`缓存行数。
