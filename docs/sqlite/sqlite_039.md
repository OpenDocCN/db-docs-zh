# 1\. 概述

> 原文：[`sqlite.com/series.html`](https://sqlite.com/series.html)

generate_series(START,STOP,STEP) 表值函数 是 SQLite 源代码树中包含的一个 可加载扩展，并编译到 命令行 shell 中。generate_series() 表有一个名为 "value" 的可见结果列，其中保存整数值，并且由参数 START、STOP 和 STEP 决定的行数。表的第一行具有 START 的值。随后的行逐步增加 STEP 到不超过 STOP 的值。

generate_series() 表还有额外的隐藏列，名为 "start"、"stop" 和 "step"，其值为提供或默认的 START、STOP 和 STEP 的有效值。它还有一个 rowid，可以通过其通常的名称访问。

省略参数采用默认值。STEP 默认为 1。STOP 默认为 4294967295。从版本 3.37.0（2021-11-27）开始，START 参数是必需的，如果省略或具有自引用或其他不可计算的值，则会引发错误。旧版本中的默认值为 START 的 0。通过使用 -DZERO_ARGUMENT_GENERATE_SERIES 编译，可以从最新代码中获取传统行为。

## 1.1\. 等效递归公共表达式

可以使用 递归公共表达式 模拟为正步幅值生成系列表。如果三个参数分别为 $start、$end 和 $step，则等效的公共表达式为：

```sql
WITH RECURSIVE generate_series(value) AS (
  SELECT $start
  UNION ALL
  SELECT value+$step FROM generate_series
   WHERE value+$step<=$end
) ...

```

公共表达式在不加载扩展的情况下工作。另一方面，扩展更容易编程且更快。

# 2\. 使用示例

生成所有不大于 100 的 5 的倍数：

```sql
SELECT value FROM generate_series(5,100,5);

```

生成 20 个随机整数值：

```sql
SELECT random() FROM generate_series(1,20);

```

查找每位客户的姓名，其帐号号码是在 10000 到 20000 之间的偶数倍数。

```sql
SELECT customer.name
  FROM customer, generate_series(10000,20000,100)
 WHERE customer.id=value;
/* or */
SELECT name FROM customer
 WHERE id IN (SELECT value
                FROM generate_series(10000,20000,200));

```
