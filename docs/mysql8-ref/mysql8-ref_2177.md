> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-statements-with-temp-tables.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-statements-with-temp-tables.html)

#### 30.4.3.40 语句 _with_temp_tables 和 x$statements_with_temp_tables 视图

这些视图列出了使用临时表的规范化语句。默认情况下，按照使用的磁盘临时表数量和内存临时表数量降序排序行。

`statements_with_temp_tables` 和 `x$statements_with_temp_tables` 视图具有以下列：

+   `query`

    规范化的语句字符串。

+   `db`

    语句的默认数据库，如果没有则为`NULL`。

+   `exec_count`

    语句执行的总次数。

+   `total_latency`

    语句定时发生的总等待时间。

+   `memory_tmp_tables`

    由语句发生创建的内存中临时表的总数。

+   `disk_tmp_tables`

    由该语句发生创建的内部磁盘临时表的总数。

+   `avg_tmp_tables_per_query`

    每次语句发生时创建的内部临时表的平均数量。

+   `tmp_tables_to_disk_pct`

    将内存中临时表转换为磁盘表的百分比。

+   `first_seen`

    语句首次出现的时间。

+   `last_seen`

    语句最近出现的时间。

+   `digest`

    陈述摘要。
