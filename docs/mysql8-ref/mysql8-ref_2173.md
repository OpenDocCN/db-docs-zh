> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-statements-with-errors-or-warnings.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-statements-with-errors-or-warnings.html)

#### 30.4.3.36 包含错误或警告的语句和 x$statements_with_errors_or_warnings 视图

这些视图显示产生错误或警告的规范化语句。默认情况下，按照错误和警告计数降序排序。

`statements_with_errors_or_warnings` 和 `x$statements_with_errors_or_warnings` 视图具有以下列：

+   `query`

    规范化的语句字符串。

+   `db`

    语句的默认数据库，如果没有则为 `NULL`。

+   `exec_count`

    语句执行的总次数。

+   `errors`

    语句产生的错误总数。

+   `error_pct`

    产生错误的语句出现次数的百分比。

+   `warnings`

    语句产生的警告总数。

+   `warning_pct`

    产生警告的语句出现次数的百分比。

+   `first_seen`

    语句首次被看到的时间。

+   `last_seen`

    语句最近被看到的时间。

+   `digest`

    语句摘要。
