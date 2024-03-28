> 原文：[`dev.mysql.com/doc/refman/8.0/en/distinct-optimization.html`](https://dev.mysql.com/doc/refman/8.0/en/distinct-optimization.html)

#### 10.2.1.18 **DISTINCT** 优化

`DISTINCT` 与 `ORDER BY` 结合在许多情况下需要一个临时表。

由于 `DISTINCT` 可能使用 `GROUP BY`，了解 MySQL 如何处理 `ORDER BY` 或 `HAVING` 子句中不属于所选列的列。请参见 Section 14.19.3, “MySQL Handling of GROUP BY”。

在大多数情况下，`DISTINCT` 子句可以被视为 `GROUP BY` 的特例。例如，以下两个查询是等效的：

```sql
SELECT DISTINCT c1, c2, c3 FROM t1
WHERE c1 > *const*;

SELECT c1, c2, c3 FROM t1
WHERE c1 > *const* GROUP BY c1, c2, c3;
```

由于这种等价性，适用于 `GROUP BY` 查询的优化也可以应用于带有 `DISTINCT` 子句的查询。因此，有关 `DISTINCT` 查询的优化可能性的更多详细信息，请参见 Section 10.2.1.17, “GROUP BY Optimization”。

当将 `LIMIT *`row_count`*` 与 `DISTINCT` 结合时，MySQL 会在找到 *`row_count`* 个唯一行后停止。

如果在查询中没有使用所有表中的列，MySQL 会在找到第一个匹配项后停止扫描任何未使用的表。在以下情况下，假设 `t1` 在 `t2` 之前被使用（您可以通过 `EXPLAIN` 进行检查），MySQL 在找到 `t2` 的第一行后（对于 `t1` 中的任何特定行）停止从 `t2` 中读取：

```sql
SELECT DISTINCT t1.a FROM t1, t2 where t1.a=t2.a;
```
