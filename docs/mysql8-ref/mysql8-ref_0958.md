> 原文：[`dev.mysql.com/doc/refman/8.0/en/comparisons-using-subqueries.html`](https://dev.mysql.com/doc/refman/8.0/en/comparisons-using-subqueries.html)

#### 15.2.15.2 使用子查询进行比较

子查询最常见的用法是形式：

```sql
*non_subquery_operand* *comparison_operator* (*subquery*)
```

其中*`comparison_operator`*是这些运算符之一：

```sql
=  >  <  >=  <=  <>  !=  <=>
```

例如：

```sql
... WHERE 'a' = (SELECT column1 FROM t1)
```

MySQL 也允许这种结构：

```sql
*non_subquery_operand* LIKE (*subquery*)
```

曾经，子查询的唯一合法位置是在比较的右侧，您可能仍然会发现一些坚持这一点的旧 DBMS。

这是一个常见形式的子查询比较的例子，使用连接无法完成。它查找表`t1`中所有`column1`值等于表`t2`中最大值的行：

```sql
SELECT * FROM t1
  WHERE column1 = (SELECT MAX(column2) FROM t2);
```

这里是另一个例子，这个例子再次使用连接是不可能的，因为它涉及对其中一个表进行聚合。它查找表`t1`中包含在给定列中出现两次值的所有行：

```sql
SELECT * FROM t1 AS t
  WHERE 2 = (SELECT COUNT(*) FROM t1 WHERE t1.id = t.id);
```

对于将子查询与标量进行比较，子查询必须返回一个标量。对于将子查询与行构造函数进行比较，子查询必须是返回与行构造函数相同数量值的行子查询。参见第 15.2.15.5 节，“行子查询”。
