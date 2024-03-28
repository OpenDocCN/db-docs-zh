> 原文：[`dev.mysql.com/doc/refman/8.0/en/scalar-subqueries.html`](https://dev.mysql.com/doc/refman/8.0/en/scalar-subqueries.html)

#### 15.2.15.1 标量操作数的子查询

在其最简单的形式中，子查询是返回单个值的标量子查询。标量子查询是一个简单的操作数，你几乎可以在任何地方使用它，只要单列值或字面值是合法的，并且你可以期望它具有所有操作数具有的特征：数据类型、长度、可以为`NULL`的指示等。例如：

```sql
CREATE TABLE t1 (s1 INT, s2 CHAR(5) NOT NULL);
INSERT INTO t1 VALUES(100, 'abcde');
SELECT (SELECT s2 FROM t1);
```

这个`SELECT`中的子查询返回一个单一值（`'abcde'`），其数据类型为`CHAR`，长度为 5，字符集和排序规则等于`CREATE TABLE`时生效的默认值，并指示列中的值可以为`NULL`。标量子查询选择的值的可空性不会被复制，因为如果子查询结果为空，结果就是`NULL`。对于刚刚显示的子查询，如果`t1`为空，结果将是`NULL`，即使`s2`是`NOT NULL`。

在一些情况下，标量子查询无法使用。如果语句只允许使用字面值，你就不能使用子查询。例如，`LIMIT`需要字面整数参数，而`LOAD DATA`需要字面字符串文件名。你不能使用子查询来提供这些值。

当你在以下部分看到包含相当简陋结构`(SELECT column1 FROM t1)`的示例时，请想象你自己的代码包含更加多样化和复杂的结构。

假设我们创建了两个表：

```sql
CREATE TABLE t1 (s1 INT);
INSERT INTO t1 VALUES (1);
CREATE TABLE t2 (s1 INT);
INSERT INTO t2 VALUES (2);
```

然后执行一个`SELECT`：

```sql
SELECT (SELECT s1 FROM t2) FROM t1;
```

结果是`2`，因为`t2`中有一行包含一个值为`2`的列`s1`。

在 MySQL 8.0.19 及更高版本中，前面的查询也可以这样写，使用`TABLE`：

```sql
SELECT (TABLE t2) FROM t1;
```

标量子查询可以是表达式的一部分，但记住括号，即使子查询是为函数提供参数的操作数。例如：

```sql
SELECT UPPER((SELECT s1 FROM t1)) FROM t2;
```

在 MySQL 8.0.19 及更高版本中，可以使用`SELECT UPPER((TABLE t1)) FROM t2`获得相同的结果。
