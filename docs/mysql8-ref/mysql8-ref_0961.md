> 原文：[`dev.mysql.com/doc/refman/8.0/en/row-subqueries.html`](https://dev.mysql.com/doc/refman/8.0/en/row-subqueries.html)

#### 15.2.15.5 行子查询

标量或列子查询返回单个值或一列值。*行子查询*是一种返回单行的子查询变体，因此可以返回多个列值。行子查询比较的合法运算符有：

```sql
=  >  <  >=  <=  <>  !=  <=>
```

以下是两个示例：

```sql
SELECT * FROM t1
  WHERE (col1,col2) = (SELECT col3, col4 FROM t2 WHERE id = 10);
SELECT * FROM t1
  WHERE ROW(col1,col2) = (SELECT col3, col4 FROM t2 WHERE id = 10);
```

对于这两个查询，如果表`t2`包含一个`id = 10`的单行，子查询返回一个单行。如果此行的`col3`和`col4`值等于任何`t1`行的`col1`和`col2`值，则`WHERE`表达式为`TRUE`，每个查询都返回这些`t1`行。如果`t2`行的`col3`和`col4`值不等于任何`t1`行的`col1`和`col2`值，则表达式为`FALSE`，查询返回一个空结果集。如果子查询未产生行，则表达式为*未知*（即`NULL`）。如果子查询产生多行，则会出现错误，因为行子查询最多只能返回一行。

有关每个运算符如何用于行比较的信息，请参阅第 14.4.2 节，“比较函数和运算符”。

表达式`(1,2)`和`ROW(1,2)`有时被称为行构造器。这两者是等价的。子查询返回的行构造器和行必须包含相同数量的值。

行构造器用于与返回两个或更多列的子查询进行比较。当子查询返回单列时，这被视为标量值而不是行，因此不能将行构造器与不返回至少两列的子查询一起使用。因此，以下查询由于语法错误而失败：

```sql
SELECT * FROM t1 WHERE ROW(1) = (SELECT column1 FROM t2)
```

行构造器在其他情境下也是合法的。例如，以下两个语句在语义上是等价的（并且由优化器以相同方式处理）：

```sql
SELECT * FROM t1 WHERE (column1,column2) = (1,1);
SELECT * FROM t1 WHERE column1 = 1 AND column2 = 1;
```

以下查询回答了请求，“找到表`t1`中存在于表`t2`中的所有行”：

```sql
SELECT column1,column2,column3
  FROM t1
  WHERE (column1,column2,column3) IN
         (SELECT column1,column2,column3 FROM t2);
```

有关优化器和行构造器的更多信息，请参阅第 10.2.1.22 节，“行构造器表达式优化”
