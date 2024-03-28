# 10.13.1 测量表达式和函数的速度

> 原文：[`dev.mysql.com/doc/refman/8.0/en/select-benchmarking.html`](https://dev.mysql.com/doc/refman/8.0/en/select-benchmarking.html)

要测量特定 MySQL 表达式或函数的速度，请使用 **mysql** 客户端程序调用 `BENCHMARK()` 函数。其语法为 `BENCHMARK(*`loop_count`*,*`expr`*)`。返回值始终为零，但 **mysql** 会打印一行显示语句执行大约花费的时间。例如：

```sql
mysql> SELECT BENCHMARK(1000000,1+1);
+------------------------+
| BENCHMARK(1000000,1+1) |
+------------------------+
|                      0 |
+------------------------+
1 row in set (0.32 sec)
```

这个结果是在一台 Pentium II 400MHz 系统上获得的。它显示 MySQL 在该系统上可以在 0.32 秒内执行 1,000,000 个简单的加法表达式。

内置的 MySQL 函数通常经过高度优化，但也可能有一些例外。`BENCHMARK()` 是一个很好的工具，可以找出某个函数是否影响了你的查询。
