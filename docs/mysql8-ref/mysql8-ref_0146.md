# 5.6.8 计算每日访问量

> 原文：[`dev.mysql.com/doc/refman/8.0/en/calculating-days.html`](https://dev.mysql.com/doc/refman/8.0/en/calculating-days.html)

以下示例展示了如何使用位组函数来计算用户每月访问网页的天数。

```sql
CREATE TABLE t1 (year YEAR, month INT UNSIGNED,
             day INT UNSIGNED);
INSERT INTO t1 VALUES(2000,1,1),(2000,1,20),(2000,1,30),(2000,2,2),
            (2000,2,23),(2000,2,23);
```

示例表包含表示用户访问页面的年-月-日值。要确定每个月这些访问发生在多少不同的天数，请使用以下查询：

```sql
SELECT year,month,BIT_COUNT(BIT_OR(1<<day)) AS days FROM t1
       GROUP BY year,month;
```

返回结果为：

```sql
+------+-------+------+
| year | month | days |
+------+-------+------+
| 2000 |     1 |    3 |
| 2000 |     2 |    2 |
+------+-------+------+
```

该查询计算表中每个年/月组合出现多少不同的天数，自动去除重复条目。
