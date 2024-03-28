# 26.6.3 与函数相关的分区限制

> 原文：[`dev.mysql.com/doc/refman/8.0/en/partitioning-limitations-functions.html`](https://dev.mysql.com/doc/refman/8.0/en/partitioning-limitations-functions.html)

这一部分讨论了 MySQL 分区中与分区表达式中使用的函数相关的限制。

只有以下列出的 MySQL 函数允许在分区表达式中使用：

+   `ABS()`

+   `CEILING()`（参见 CEILING() 和 FLOOR() and FLOOR()"))

+   `DATEDIFF()`

+   `DAY()`

+   `DAYOFMONTH()`

+   `DAYOFWEEK()`

+   `DAYOFYEAR()`

+   `EXTRACT()`（参见 带有 WEEK 参数的 EXTRACT() 函数 function with WEEK specifier")）

+   `FLOOR()`（参见 CEILING() 和 FLOOR() and FLOOR()")）

+   `HOUR()`

+   `MICROSECOND()`

+   `MINUTE()`

+   `MOD()`

+   `MONTH()`

+   `QUARTER()`

+   `SECOND()`

+   `TIME_TO_SEC()`

+   `TO_DAYS()`

+   `TO_SECONDS()`

+   `UNIX_TIMESTAMP()`（带有 `TIMESTAMP` 列）

+   `WEEKDAY()`

+   `YEAR()`

+   `YEARWEEK()`

在 MySQL 8.0 中，支持对 `TO_DAYS()`、`TO_SECONDS()`、`YEAR()` 和 `UNIX_TIMESTAMP()` 函数进行分区修剪。更多信息请参见 第 26.4 节，“分区修剪”。

**CEILING()和 FLOOR()。** 如果这些函数的参数是精确数值类型，如`INT`类型或`DECIMAL`之一，则每个函数仅返回整数。这意味着，例如，以下`CREATE TABLE`语句将因错误而失败，如下所示：

```sql
mysql> CREATE TABLE t (c FLOAT) PARTITION BY LIST( FLOOR(c) )(
 ->     PARTITION p0 VALUES IN (1,3,5),
 ->     PARTITION p1 VALUES IN (2,4,6)
 -> );
ERROR 1490 (HY000): The PARTITION function returns the wrong type
```

**带有 WEEK 指定符的 EXTRACT()函数。** 当作为`EXTRACT(WEEK FROM *`col`*)`使用时，`EXTRACT()`函数返回的值取决于`default_week_format`系统变量的值。因此，当指定单位为`WEEK`时，`EXTRACT()`不允许作为分区函数。（Bug #54483）

查看第 14.6.2 节，“数学函数”，了解这些函数的返回类型的更多信息，以及第 13.1 节，“数值数据类型”。
