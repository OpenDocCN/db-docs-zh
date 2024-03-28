> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-statements-with-sorting.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-statements-with-sorting.html)

#### 30.4.3.39 statements_with_sorting 和 x$statements_with_sorting 视图

这些视图列出了执行排序操作的规范化语句。默认情况下，按总延迟降序排序行。

`statements_with_sorting` 和 `x$statements_with_sorting` 视图具有以下列：

+   `query`

    规范化的语句字符串。

+   `db`

    语句的默认数据库，如果没有则为`NULL`。

+   `exec_count`

    语句执行的总次数。

+   `total_latency`

    该语句定时出现的总等待时间。

+   `sort_merge_passes`

    通过语句出现的总排序合并次数。

+   `avg_sort_merges`

    每次语句出现的平均排序合并次数。

+   `sorts_using_scans`

    通过语句出现的表扫描进行排序的总次数。

+   `sort_using_range`

    通过语句出现的范围访问进行排序的总次数。

+   `rows_sorted`

    通过语句出现的总排序行数。

+   `avg_rows_sorted`

    每次语句出现的平均排序行数。

+   `first_seen`

    该语句首次出现的时间。

+   `last_seen`

    语句最近一次出现的时间。

+   `digest`

    语句摘要。
