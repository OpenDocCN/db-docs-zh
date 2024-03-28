> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-host-summary-by-statement-type.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-host-summary-by-statement-type.html)

#### 30.4.3.6 主机按语句类型和 x$host_summary_by_statement_type 视图

这些视图总结了按主机和语句类型分组执行的语句的信息。默认情况下，按主机和降序总延迟排序行。

`host_summary_by_statement_type` 和 `x$host_summary_by_statement_type` 视图具有以下列：

+   `host`

    客户端连接的主机。在底层性能模式表中`HOST`列为`NULL`的行被假定为后台线程，并以`background`作为主机名报告。

+   `statement`

    语句事件名称的最终组成部分。

+   `total`

    主机的语句事件的总发生次数。

+   `total_latency`

    主机的语句事件的总等待时间。

+   `max_latency`

    主机的语句事件的最大单个等待时间。

+   `lock_latency`

    由主机的语句事件等待锁的总时间。

+   `cpu_latency`

    当前线程在 CPU 上花费的时间。

+   `rows_sent`

    由主机的语句事件返回的行总数。

+   `rows_examined`

    由主机的语句事件从存储引擎读取的行总数。

+   `rows_affected`

    由主机的语句事件影响的行总数。

+   `full_scans`

    由主机的语句事件执行的全表扫描的总数。
