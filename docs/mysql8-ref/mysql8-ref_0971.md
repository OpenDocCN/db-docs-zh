# 15.2.18 UNION 子句

> 原文：[`dev.mysql.com/doc/refman/8.0/en/union.html`](https://dev.mysql.com/doc/refman/8.0/en/union.html)

```sql
*query_expression_body* UNION [ALL | DISTINCT] *query_block*
    [UNION [ALL | DISTINCT] *query_expression_body*]
    [...]

*query_expression_body*:
    *See Section 15.2.14, “Set Operations with UNION, INTERSECT, and EXCEPT”*
```

`UNION` 将多个查询块的结果合并为单个结果集。此示例使用 `SELECT` 语句：

```sql
mysql> SELECT 1, 2;
+---+---+
| 1 | 2 |
+---+---+
| 1 | 2 |
+---+---+
mysql> SELECT 'a', 'b';
+---+---+
| a | b |
+---+---+
| a | b |
+---+---+
mysql> SELECT 1, 2 UNION SELECT 'a', 'b';
+---+---+
| 1 | 2 |
+---+---+
| 1 | 2 |
| a | b |
+---+---+
```

#### MySQL 8.0 中的 UNION 处理与 MySQL 5.7 相比

在 MySQL 8.0 中，`SELECT` 和 `UNION` 的解析器规则进行了重构，以使其更一致（在每个上下文中统一应用相同的 `SELECT` 语法）并减少重复。与 MySQL 5.7 相比，这项工作产生了几个用户可见的效果，可能需要重写某些语句：

+   `NATURAL JOIN` 允许可选的 `INNER` 关键字（`NATURAL INNER JOIN`），符合标准 SQL。

+   允许不带括号的右深度连接（例如，`... JOIN ... JOIN ... ON ... ON`），符合标准 SQL。

+   `STRAIGHT_JOIN` 现在允许 `USING` 子句，类似于其他内连接。

+   解析器接受围绕查询表达式的括号。例如，`(SELECT ... UNION SELECT ...)` 是允许的。另请参阅 第 15.2.11 节，“带括号的查询表达式”。

+   解析器更好地符合文档中允许放置 `SQL_CACHE` 和 `SQL_NO_CACHE` 查询修饰符的规定。

+   左侧嵌套联合，以前仅在子查询中允许，现在在顶层语句中也允许。例如，此语句现在被接受为有效：

    ```sql
    (SELECT 1 UNION SELECT 1) UNION SELECT 1;
    ```

+   锁定子句（`FOR UPDATE`，`LOCK IN SHARE MODE`）仅允许在非 `UNION` 查询中使用。这意味着必须对包含锁定子句的 `SELECT` 语句使用括号。此语句不再被接受为有效：

    ```sql
    SELECT 1 FOR UPDATE UNION SELECT 1 FOR UPDATE;
    ```

    相反，应该这样写陈述：

    ```sql
    (SELECT 1 FOR UPDATE) UNION (SELECT 1 FOR UPDATE);
    ```
