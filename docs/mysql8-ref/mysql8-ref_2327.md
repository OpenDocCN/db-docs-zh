> 原文：[`dev.mysql.com/doc/refman/8.0/en/problems-with-alias.html`](https://dev.mysql.com/doc/refman/8.0/en/problems-with-alias.html)

#### B.3.4.4 列别名的问题

可以在查询的选择列表中使用别名为列指定不同的名称。您可以在 `GROUP BY`、`ORDER BY` 或 `HAVING` 子句中使用别名来引用该列：

```sql
SELECT SQRT(a*b) AS root FROM *tbl_name*
  GROUP BY root HAVING root > 0;
SELECT id, COUNT(*) AS cnt FROM *tbl_name*
  GROUP BY id HAVING cnt > 0;
SELECT id AS 'Customer identity' FROM *tbl_name*;
```

标准 SQL 不允许在 `WHERE` 子句中引用列别名。这个限制是因为在评估 `WHERE` 子句时，列的值可能尚未确定。例如，以下查询是非法的：

```sql
SELECT id, COUNT(*) AS cnt FROM *tbl_name*
  WHERE cnt > 0 GROUP BY id;
```

`WHERE` 子句确定应包括在 `GROUP BY` 子句中的行，但它引用了一个在选择行并按 `GROUP BY` 分组之后才知道的列值的别名。

在查询的选择列表中，可以使用标识符或字符串引号字符指定带引号的列别名：

```sql
SELECT 1 AS `one`, 2 AS 'two';
```

在语句的其他地方，引用别名的引号必须使用标识符引用，否则引用将被视为字符串字面量。例如，该语句按照使用别名 ``a`` 引用的列 `id` 的值进行分组：

```sql
SELECT id AS 'a', COUNT(*) AS cnt FROM *tbl_name*
  GROUP BY `a`;
```

该语句按照字面字符串 `'a'` 进行分组，并不会按照您的预期工作：

```sql
SELECT id AS 'a', COUNT(*) AS cnt FROM *tbl_name*
  GROUP BY 'a';
```
