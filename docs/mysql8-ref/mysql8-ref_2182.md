> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-user-summary-by-statement-latency.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-user-summary-by-statement-latency.html)

#### 30.4.3.45 用户 _summary_by_statement_latency 和 x$user_summary_by_statement_latency 视图

这些视图总结了按用户分组的整体语句统计信息。默认情况下，按总延迟降序排序行。

`user_summary_by_statement_latency` 和 `x$user_summary_by_statement_latency` 视图具有以下列：

+   `user`

    客户端用户名。假定底层性能模式表中的 `USER` 列为 `NULL` 的行是后台线程的行，并报告为 `background` 主机名。

+   `total`

    用户的语句总数。

+   `total_latency`

    用户的定时语句的总等待时间。

+   `max_latency`

    用户的定时语句的最大单个等待时间。

+   `lock_latency`

    用户的定时语句等待锁的总时间。

+   `cpu_latency`

    当前线程在 CPU 上花费的时间。

+   `rows_sent`

    用户语句返回的行数总数。

+   `rows_examined`

    用户语句从存储引擎读取的行数总数。

+   `rows_affected`

    用户语句影响的行数总数。

+   `full_scans`

    用户语句的全表扫描总数。
