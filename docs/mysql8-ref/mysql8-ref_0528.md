> 原文：[`dev.mysql.com/doc/refman/8.0/en/is-null-optimization.html`](https://dev.mysql.com/doc/refman/8.0/en/is-null-optimization.html)

#### 10.2.1.15 `IS NULL` 优化

MySQL 可以对 *`col_name`* `IS NULL` 执行与 *`col_name`* `=` *`constant_value`* 相同的优化。例如，MySQL 可以使用索引和范围来搜索带有 `IS NULL` 的 `NULL`。

示例:

```sql
SELECT * FROM *tbl_name* WHERE *key_col* IS NULL;

SELECT * FROM *tbl_name* WHERE *key_col* <=> NULL;

SELECT * FROM *tbl_name*
  WHERE *key_col*=*const1* OR *key_col*=*const2* OR *key_col* IS NULL;
```

如果 `WHERE` 子句包含对声明为 `NOT NULL` 的列的 *`col_name`* `IS NULL` 条件，则该表达式会被优化掉。在列可能产生 `NULL` 的情况下（例如，如果它来自 `LEFT JOIN` 的右侧表），此优化不会发生。

MySQL 也可以优化组合 `*`col_name`* = *`expr`* OR *`col_name`* IS NULL`，这种形式在解析子查询中很常见。`EXPLAIN` 在使用此优化时显示 `ref_or_null`。

此优化可以处理任何关键部分的一个 `IS NULL`。

一些查询示例，假设在表 `t2` 的列 `a` 和 `b` 上有索引：

```sql
SELECT * FROM t1 WHERE t1.a=*expr* OR t1.a IS NULL;

SELECT * FROM t1, t2 WHERE t1.a=t2.a OR t2.a IS NULL;

SELECT * FROM t1, t2
  WHERE (t1.a=t2.a OR t2.a IS NULL) AND t2.b=t1.b;

SELECT * FROM t1, t2
  WHERE t1.a=t2.a AND (t2.b=t1.b OR t2.b IS NULL);

SELECT * FROM t1, t2
  WHERE (t1.a=t2.a AND t2.a IS NULL AND ...)
  OR (t1.a=t2.a AND t2.a IS NULL AND ...);
```

`ref_or_null` 首先对参考键进行读取，然后单独搜索具有 `NULL` 键值的行。

该优化只能处理一个 `IS NULL` 级别。在下面的查询中，MySQL 仅在表达式 `(t1.a=t2.a AND t2.a IS NULL)` 上使用关键查找，并不能使用列 `b` 上的关键部分：

```sql
SELECT * FROM t1, t2
  WHERE (t1.a=t2.a AND t2.a IS NULL)
  OR (t1.b=t2.b AND t2.b IS NULL);
```
