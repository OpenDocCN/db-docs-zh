> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-schema-table-statistics.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-schema-table-statistics.html)

#### 30.4.3.29 `schema_table_statistics` 和 `x$schema_table_statistics` 视图

这些视图总结了表的统计信息。默认情况下，按照总等待时间降序排序（具有最多争用的表排在前面）。

这些视图使用一个辅助视图，`x$ps_schema_table_statistics_io`。

`schema_table_statistics` 和 `x$schema_table_statistics` 视图具有以下列：

+   `table_schema`

    包含表的模式。

+   `table_name`

    表名。

+   `total_latency`

    对表的定时 I/O 事件的总等待时间。

+   `rows_fetched`

    从表中读取的总行数。

+   `fetch_latency`

    对表的定时读取 I/O 事件的总等待时间。

+   `rows_inserted`

    向表中插入的总行数。

+   `insert_latency`

    对表的定时插入 I/O 事件的总等待时间。

+   `rows_updated`

    在表中更新的总行数。

+   `update_latency`

    对表的定时更新 I/O 事件的总等待时间。

+   `rows_deleted`

    从表中删除的总行数。

+   `delete_latency`

    对表的定时删除 I/O 事件的总等待时间。

+   `io_read_requests`

    对表的总读取请求次数。

+   `io_read`

    从表中读取的总字节数。

+   `io_read_latency`

    从表中读取的总等待时间。

+   `io_write_requests`

    对表的总写入请求次数。

+   `io_write`

    向表中写入的总字节数。

+   `io_write_latency`

    对表的写入的总等待时间。

+   `io_misc_requests`

    对表的杂项 I/O 请求的总数。

+   `io_misc_latency`

    对表的杂项 I/O 请求的总等待时间。
