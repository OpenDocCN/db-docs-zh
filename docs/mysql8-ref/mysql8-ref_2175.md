> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-statements-with-runtimes-in-95th-percentile.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-statements-with-runtimes-in-95th-percentile.html)

#### 30.4.3.38 `statements_with_runtimes_in_95th_percentile` 和 `x$statements_with_runtimes_in_95th_percentile` 视图

这些视图列出了运行时间处于第 95 百分位数的语句。默认情况下，按照平均延迟降序排序行。

这两个视图使用两个辅助视图，`x$ps_digest_avg_latency_distribution` 和 `x$ps_digest_95th_percentile_by_avg_us`。

`statements_with_runtimes_in_95th_percentile` 和 `x$statements_with_runtimes_in_95th_percentile` 视图具有以下列：

+   `query`

    规范化的语句字符串。

+   `db`

    语句的默认数据库，如果没有则为`NULL`。

+   `full_scan`

    语句执行的全表扫描总次数。

+   `exec_count`

    语句执行的总次数。

+   `err_count`

    语句执行产生的错误总数。

+   `warn_count`

    语句执行产生的警告总数。

+   `total_latency`

    语句定时执行的总等待时间。

+   `max_latency`

    语句定时执行的最大单次等待时间。

+   `avg_latency`

    每次定时执行语句的平均等待时间。

+   `rows_sent`

    语句执行返回的总行数。

+   `rows_sent_avg`

    每次语句执行返回的平均行数。

+   `rows_examined`

    语句执行从存储引擎读取的总行数。

+   `rows_examined_avg`

    每次语句执行从存储引擎读取的平均行数。

+   `first_seen`

    首次出现语句的时间。

+   `last_seen`

    语句最近一次出现的时间。

+   `digest`

    语句摘要。
