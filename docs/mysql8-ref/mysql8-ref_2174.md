> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-statements-with-full-table-scans.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-statements-with-full-table-scans.html)

#### 30.4.3.37 statements_with_full_table_scans 和 x$statements_with_full_table_scans 视图

这些视图显示了执行全表扫描的规范化语句。默认情况下，按照全表扫描所占时间的百分比和总延迟时间降序排序。

`statements_with_full_table_scans` 和 `x$statements_with_full_table_scans` 视图具有以下列：

+   `query`

    规范化的语句字符串。

+   `db`

    语句的默认数据库，如果没有则为 `NULL`。

+   `exec_count`

    语句执行的总次数。

+   `total_latency`

    语句事件的计时等待时间。

+   `no_index_used_count`

    未使用索引扫描表的总次数。

+   `no_good_index_used_count`

    未使用良好索引扫描表的总次数。

+   `no_index_used_pct`

    未使用索引扫描表的时间百分比。

+   `rows_sent`

    从表返回的总行数。

+   `rows_examined`

    从存储引擎读取的表的总行数。

+   `rows_sent_avg`

    从表返回的平均行数。

+   `rows_examined_avg`

    从存储引擎读取的表的平均行数。

+   `first_seen`

    语句首次出现的时间。

+   `last_seen`

    语句最近出现的时间。

+   `digest`

    语句摘要。
