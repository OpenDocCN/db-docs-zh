> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-host-summary-by-statement-latency.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-host-summary-by-statement-latency.html)

#### 30.4.3.5 `host_summary_by_statement_latency` 和 `x$host_summary_by_statement_latency` 视图

这些视图按主机分组总结了整体语句统计信息。默认情况下，按总延迟降序排序行。

`host_summary_by_statement_latency` 和 `x$host_summary_by_statement_latency` 视图具有以下列：

+   `host`

    客户端连接的主机。假定基础性能模式表中`HOST`列为`NULL`的行是后台线程的行，并且报告的主机名为`background`。

+   `total`

    主机的语句总数。

+   `total_latency`

    主机的定时语句的总等待时间。

+   `max_latency`

    主机的定时语句的最大单个等待时间。

+   `lock_latency`

    主机的定时语句等待锁的总时间。

+   `cpu_latency`

    当前线程在 CPU 上花费的时间。

+   `rows_sent`

    语句返回的主机的行总数。

+   `rows_examined`

    语句从存储引擎读取的主机的总行数。

+   `rows_affected`

    语句影响的主机的总行数。

+   `full_scans`

    语句对主机进行的全表扫描总数。
