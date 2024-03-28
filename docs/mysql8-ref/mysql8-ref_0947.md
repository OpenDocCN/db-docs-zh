# 15.2.8 INTERSECT 子句

> 原文：[`dev.mysql.com/doc/refman/8.0/en/intersect.html`](https://dev.mysql.com/doc/refman/8.0/en/intersect.html)

```sql
*query_expression_body* INTERSECT [ALL | DISTINCT] *query_expression_body*
    [INTERSECT [ALL | DISTINCT] *query_expression_body*]
    [...]

*query_expression_body*:
    *See Section 15.2.14, “Set Operations with UNION, INTERSECT, and EXCEPT”*
```

`INTERSECT` 限制了来自多个查询块的结果，使其仅包含所有查询块中共同的行。示例：

```sql
mysql> TABLE a;
+------+------+
| m    | n    |
+------+------+
|    1 |    2 |
|    2 |    3 |
|    3 |    4 |
+------+------+
3 rows in set (0.00 sec)

mysql> TABLE b;
+------+------+
| m    | n    |
+------+------+
|    1 |    2 |
|    1 |    3 |
|    3 |    4 |
+------+------+
3 rows in set (0.00 sec)

mysql> TABLE c;
+------+------+
| m    | n    |
+------+------+
|    1 |    3 |
|    1 |    3 |
|    3 |    4 |
+------+------+
3 rows in set (0.00 sec)

mysql> TABLE a INTERSECT TABLE b;
+------+------+
| m    | n    |
+------+------+
|    1 |    2 |
|    3 |    4 |
+------+------+
2 rows in set (0.00 sec)

mysql> TABLE a INTERSECT TABLE c;
+------+------+
| m    | n    |
+------+------+
|    3 |    4 |
+------+------+
1 row in set (0.00 sec)
```

与 `UNION` 和 `EXCEPT` 一样，如果未指定 `DISTINCT` 或 `ALL`，则默认为 `DISTINCT`。

`DISTINCT` 可以从交集的任一侧去除重复项，如下所示：

```sql
mysql> TABLE c INTERSECT DISTINCT TABLE c;
+------+------+
| m    | n    |
+------+------+
|    1 |    3 |
|    3 |    4 |
+------+------+
2 rows in set (0.00 sec)

mysql> TABLE c INTERSECT ALL TABLE c;
+------+------+
| m    | n    |
+------+------+
|    1 |    3 |
|    1 |    3 |
|    3 |    4 |
+------+------+
3 rows in set (0.00 sec)
```

（`TABLE c INTERSECT TABLE c` 相当于刚刚显示的两个语句中的第一个。）

与 `UNION` 一样，操作数必须具有相同数量的列。结果集列类型也与 `UNION` 相同。

`INTERSECT` 的优先级高于 `UNION` 和 `EXCEPT`，因此这两个语句是等效的：

```sql
TABLE r EXCEPT TABLE s INTERSECT TABLE t;

TABLE r EXCEPT (TABLE s INTERSECT TABLE t);
```

对于 `INTERSECT ALL`，左侧表中任何唯一行的最大支持重复次数为 `4294967295`。

`INTERSECT` 在 MySQL 8.0.31 中添加。
