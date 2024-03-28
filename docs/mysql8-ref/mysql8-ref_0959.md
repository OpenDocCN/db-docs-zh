> 原文：[`dev.mysql.com/doc/refman/8.0/en/any-in-some-subqueries.html`](https://dev.mysql.com/doc/refman/8.0/en/any-in-some-subqueries.html)

#### 15.2.15.3 带有 ANY、IN 或 SOME 的子查询

语法：

```sql
*operand* *comparison_operator* ANY (*subquery*)
*operand* IN (*subquery*)
*operand* *comparison_operator* SOME (*subquery*)
```

其中*`comparison_operator`*是以下这些运算符之一：

```sql
=  >  <  >=  <=  <>  !=
```

`ANY`关键字必须跟在比较运算符后面，意思是“如果子查询返回的列中的任何值对比较为`TRUE`，则返回`TRUE`”。例如：

```sql
SELECT s1 FROM t1 WHERE s1 > ANY (SELECT s1 FROM t2);
```

假设表`t1`中有一行包含`(10)`。如果表`t2`包含`(21,14,7)`，则表达式为`TRUE`，因为`t2`中有一个值`7`小于`10`。如果表`t2`包含`(20,10)`，或者表`t2`为空，则表达式为`FALSE`。如果表`t2`包含`(NULL,NULL,NULL)`，则表达式为*unknown*（即`NULL`）。

在与子查询一起使用时，`IN`是`= ANY`的别名。因此，这两个语句是相同的：

```sql
SELECT s1 FROM t1 WHERE s1 = ANY (SELECT s1 FROM t2);
SELECT s1 FROM t1 WHERE s1 IN    (SELECT s1 FROM t2);
```

当与表达式列表一起使用时，`IN`和`= ANY`不是同义词。`IN`可以接受表达式列表，但`= ANY`不能。参见 Section 14.4.2, “Comparison Functions and Operators”。

`NOT IN`不是`<> ANY`的别名，而是`<> ALL`的别名。参见 Section 15.2.15.4, “Subqueries with ALL”。

`SOME`这个词是`ANY`的别名。因此，这两个语句是相同的：

```sql
SELECT s1 FROM t1 WHERE s1 <> ANY  (SELECT s1 FROM t2);
SELECT s1 FROM t1 WHERE s1 <> SOME (SELECT s1 FROM t2);
```

使用`SOME`这个词的情况很少，但这个例子说明了为什么它可能有用。对大多数人来说，英语短语“a is not equal to any b”意味着“没有 b 等于 a”，但这并不是 SQL 语法的意思。该语法的含义是“有一些 b 不等于 a”。使用`<> SOME`可以确保每个人都理解查询的真正含义。

从 MySQL 8.0.19 开始，您可以在标量`IN`、`ANY`或`SOME`子查询中使用`TABLE`，前提是表只包含一列。如果`t2`只有一列，那么本节中先前显示的语句可以写成这样，在每种情况下用`TABLE t2`替换`SELECT s1 FROM t2`：

```sql
SELECT s1 FROM t1 WHERE s1 > ANY (TABLE t2);

SELECT s1 FROM t1 WHERE s1 = ANY (TABLE t2);

SELECT s1 FROM t1 WHERE s1 IN (TABLE t2);

SELECT s1 FROM t1 WHERE s1 <> ANY  (TABLE t2);

SELECT s1 FROM t1 WHERE s1 <> SOME (TABLE t2);
```
