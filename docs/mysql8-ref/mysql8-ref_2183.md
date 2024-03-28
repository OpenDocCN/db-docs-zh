> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-user-summary-by-statement-type.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-user-summary-by-statement-type.html)

#### 30.4.3.46 用户汇总按语句类型和 x$user_summary_by_statement_type 视图

这些视图总结了按用户和语句类型分组执行的语句信息。默认情况下，按用户和降序总延迟排序行。

`user_summary_by_statement_type` 和 `x$user_summary_by_statement_type` 视图具有以下列：

+   `user`

    客户端用户名。在底层性能模式表中 `USER` 列为 `NULL` 的行被假定为后台线程，并报告为主机名 `background`。

+   `statement`

    语句事件名称的最终组成部分。

+   `total`

    用户语句事件的总发生次数。

+   `total_latency`

    用户语句事件的总等待时间。

+   `max_latency`

    用户语句事件的最大单个等待时间。

+   `lock_latency`

    用户语句事件的总等待锁时间。

+   `cpu_latency`

    当前线程在 CPU 上花费的时间。

+   `rows_sent`

    用户语句事件返回的总行数。

+   `rows_examined`

    用户语句事件从存储引擎读取的总行数。

+   `rows_affected`

    语句事件对用户影响的总行数。

+   `full_scans`

    用户语句事件的总表全表扫描次数。
