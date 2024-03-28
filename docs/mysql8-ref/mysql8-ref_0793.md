# 14.4.1 运算符优先级

> 原文：[`dev.mysql.com/doc/refman/8.0/en/operator-precedence.html`](https://dev.mysql.com/doc/refman/8.0/en/operator-precedence.html)

运算符优先级按照以下列表从最高优先级到最低优先级显示。在同一行上显示在一起的运算符具有相同的优先级。

```sql
INTERVAL
BINARY, COLLATE
!
- (unary minus), ~ (unary bit inversion)
^
*, /, DIV, %, MOD
-, +
<<, >>
&
|
= (comparison), <=>, >=, >, <=, <, <>, !=, IS, LIKE, REGEXP, IN, MEMBER OF
BETWEEN, CASE, WHEN, THEN, ELSE
NOT
AND, &&
XOR
OR, ||
= (assignment), :=
```

`=`的优先级取决于它是作为比较运算符(`=`)还是作为赋值运算符(`=`)使用。当作为比较运算符使用时，它与`<=>`、`>=`、`>`、`<=`、`<`、`<>`、`!=`、`IS`、`LIKE`、`REGEXP`和`IN()`具有相同的优先级。当作为赋值运算符使用时，它与`:=`具有相同的优先级。“变量赋值的 SET 语法”和“用户定义变量”部分解释了 MySQL 如何确定应该应用哪种`=`的解释。

对于在表达式中具有相同优先级的运算符，计算从左到右进行，但赋值运算符的计算是从右到左进行。

一些运算符的优先级和含义取决于 SQL 模式：

+   默认情况下，`||`是逻辑`OR`运算符。启用`PIPES_AS_CONCAT`后，`||`是字符串连接运算符，优先级介于`^`和一元运算符之间。

+   默认情况下，`!`的优先级高于`NOT`。启用`HIGH_NOT_PRECEDENCE`后，`!`和`NOT`具有相同的优先级。

参见“服务器 SQL 模式”第 7.1.11 节。

运算符的优先级确定了表达式中项的计算顺序。要覆盖这个顺序并显式地分组项，请使用括号。例如：

```sql
mysql> SELECT 1+2*3;
 -> 7
mysql> SELECT (1+2)*3;
 -> 9
```
