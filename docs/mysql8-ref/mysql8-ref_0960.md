> 原文：[`dev.mysql.com/doc/refman/8.0/en/all-subqueries.html`](https://dev.mysql.com/doc/refman/8.0/en/all-subqueries.html)

#### 15.2.15.4 带有 ALL 的子查询

语法：

```sql
*operand* *comparison_operator* ALL (*subquery*)
```

单词 `ALL` 必须跟在比较运算符后面，意思是“如果比较对子查询返回的列中的所有值都为 `TRUE`，则返回 `TRUE`”。例如：

```sql
SELECT s1 FROM t1 WHERE s1 > ALL (SELECT s1 FROM t2);
```

假设表 `t1` 中包含一行 `(10)`。如果表 `t2` 包含 `(-5,0,+5)`，表达式为 `TRUE`，因为 `10` 大于 `t2` 中的所有三个值。如果表 `t2` 包含 `(12,6,NULL,-100)`，表达式为 `FALSE`，因为 `t2` 中有一个值 `12` 大于 `10`。如果表 `t2` 包含 `(0,NULL,1)`，表达式为 *unknown*（即 `NULL`）。

最后，如果表 `t2` 是空的，表达式为 `TRUE`。因此，当表 `t2` 为空时，以下表达式为 `TRUE`：

```sql
SELECT * FROM t1 WHERE 1 > ALL (SELECT s1 FROM t2);
```

但是当表 `t2` 为空时，此表达式为 `NULL`：

```sql
SELECT * FROM t1 WHERE 1 > (SELECT s1 FROM t2);
```

此外，当表 `t2` 为空时，以下表达式为 `NULL`：

```sql
SELECT * FROM t1 WHERE 1 > ALL (SELECT MAX(s1) FROM t2);
```

一般来说，*包含 `NULL` 值的表* 和 *空表* 是“边缘情况”。在编写子查询时，始终考虑是否考虑了这两种可能性。

`NOT IN` 是 `<> ALL` 的别名。因此，这两个语句是相同的：

```sql
SELECT s1 FROM t1 WHERE s1 <> ALL (SELECT s1 FROM t2);
SELECT s1 FROM t1 WHERE s1 NOT IN (SELECT s1 FROM t2);
```

MySQL 8.0.19 支持 `TABLE` 语句。与 `IN`、`ANY` 和 `SOME` 一样，你可以在 `TABLE` 中使用 `ALL` 和 `NOT IN`，前提是满足以下两个条件：

+   子查询中只包含一列

+   子查询不依赖于列表达式

例如，假设表 `t2` 只包含一列，前面显示的最后两个语句可以这样使用 `TABLE t2` 编写：

```sql
SELECT s1 FROM t1 WHERE s1 <> ALL (TABLE t2);
SELECT s1 FROM t1 WHERE s1 NOT IN (TABLE t2);
```

无法使用 `TABLE t2` 编写诸如 `SELECT * FROM t1 WHERE 1 > ALL (SELECT MAX(s1) FROM t2);` 这样的查询，因为子查询依赖于列表达式。
