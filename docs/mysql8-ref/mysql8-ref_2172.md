> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-statement-analysis.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-statement-analysis.html)

#### 30.4.3.35 语句分析和 x$statement_analysis 视图

这些视图列出了带有聚合统计信息的规范化语句。内容模仿了 MySQL Enterprise Monitor 查询分析视图。默认情况下，按总延迟降序排序行。

`statement_analysis` 和 `x$statement_analysis` 视图具有以下列：

+   `query`

    规范化的语句字符串。

+   `db`

    语句的默认数据库，如果没有则为 `NULL`。

+   `full_scan`

    由语句出现执行的全表扫描总数。

+   `exec_count`

    语句执行的总次数。

+   `err_count`

    由语句出现产生的错误总数。

+   `warn_count`

    由语句出现产生的警告总数。

+   `total_latency`

    语句定时出现的总等待时间。

+   `max_latency`

    定时出现的语句的最大单次等待时间。

+   `avg_latency`

    每次语句出现的平均等待时间。

+   `lock_latency`

    定时出现的语句等待锁的总时间。

+   `cpu_latency`

    当前线程在 CPU 上花费的时间。

+   `rows_sent`

    由语句出现返回的总行数。

+   `rows_sent_avg`

    每次语句出现平均返回的行数。

+   `rows_examined`

    由语句出现从存储引擎读取的总行数。

+   `rows_examined_avg`

    从存储引擎读取的平均行数。

+   `rows_affected`

    由语句出现影响的总行数。

+   `rows_affected_avg`

    每次语句出现的平均影响行数。

+   `tmp_tables`

    由语句出现创建的内部内存临时表的总数。

+   `tmp_disk_tables`

    由语句出现创建的内部磁盘临时表的总数。

+   `rows_sorted`

    由语句出现排序的总行数。

+   `sort_merge_passes`

    由语句出现的排序合并总数。

+   `max_controlled_memory`

    语句使用的最大受控内存量（字节）。

    该列在 MySQL 8.0.31 中添加。

+   `max_total_memory`

    语句使用的最大内存量（字节）。

    该列在 MySQL 8.0.31 中添加。

+   `digest`

    语句摘要。

+   `first_seen`

    语句首次出现的时间。

+   `last_seen`

    语句最近一次出现的时间。
