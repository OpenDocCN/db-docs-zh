# 15.2.11 带括号的查询表达式

> 译文：[`dev.mysql.com/doc/refman/8.0/en/parenthesized-query-expressions.html`](https://dev.mysql.com/doc/refman/8.0/en/parenthesized-query-expressions.html)

```sql
*parenthesized_query_expression*:
    ( *query_expression* [*order_by_clause*] [*limit_clause*] )
      [*order_by_clause*]
      [*limit_clause*]
      [*into_clause*]

*query_expression*:
    *query_block* [*set_op* *query_block* [*set_op* *query_block* ...]]
      [*order_by_clause*]
      [*limit_clause*]
      [*into_clause*]

*query_block*:
    SELECT ... | TABLE | VALUES

*order_by_clause*:
    ORDER BY as for SELECT

*limit_clause*:
    LIMIT as for SELECT

*into_clause*:
    INTO as for SELECT

*set_op*:
    UNION | INTERSECT | EXCEPT
```

MySQL 8.0.22 及更高版本支持根据前述语法的带括号的查询表达式。在其最简单形式下，带括号的查询表达式包含一个返回结果集的单个`SELECT`或其他语句，没有后续的可选子句：

```sql
(SELECT 1);
(SELECT * FROM INFORMATION_SCHEMA.SCHEMATA WHERE SCHEMA_NAME = 'mysql');

TABLE t;

VALUES ROW(2, 3, 4), ROW(1, -2, 3);
```

(支持从 MySQL 8.0.19 开始提供的`TABLE`和`VALUES`语句。)

一个带括号的查询表达式也可以包含通过一个或多个集合操作连接的查询，例如`UNION`，并以任意或所有可选子句结束：

```sql
mysql> (SELECT 1 AS result UNION SELECT 2);
+--------+
| result |
+--------+
|      1 |
|      2 |
+--------+
mysql> (SELECT 1 AS result UNION SELECT 2) LIMIT 1;
+--------+
| result |
+--------+
|      1 |
+--------+
mysql> (SELECT 1 AS result UNION SELECT 2) LIMIT 1 OFFSET 1;
+--------+
| result |
+--------+
|      2 |
+--------+
mysql> (SELECT 1 AS result UNION SELECT 2)
       ORDER BY result DESC LIMIT 1;
+--------+
| result |
+--------+
|      2 |
+--------+
mysql> (SELECT 1 AS result UNION SELECT 2)
       ORDER BY result DESC LIMIT 1 OFFSET 1;
+--------+
| result |
+--------+
|      1 |
+--------+
mysql> (SELECT 1 AS result UNION SELECT 3 UNION SELECT 2)
       ORDER BY result LIMIT 1 OFFSET 1 INTO @var;
mysql> SELECT @var;
+------+
| @var |
+------+
|    2 |
+------+
```

除了`UNION`之外，`INTERSECT`和`EXCEPT`集合操作符从 MySQL 8.0.31 开始提供。`INTERSECT`在`UNION`和`EXCEPT`之前起作用，因此以下两个语句是等效的：

```sql
SELECT a FROM t1 EXCEPT SELECT b FROM t2 INTERSECT SELECT c FROM t3;

SELECT a FROM t1 EXCEPT (SELECT b FROM t2 INTERSECT SELECT c FROM t3);
```

带括号的查询表达式也用作查询表达式，因此查询表达式通常由查询块组成，也可以由带括号的查询表达式组成：

```sql
(TABLE t1 ORDER BY a) UNION (TABLE t2 ORDER BY b) ORDER BY z;
```

查询块可以有尾随的`ORDER BY`和`LIMIT`子句，在外部集合操作、`ORDER BY`和`LIMIT`之前应用。

不能有带有尾随`ORDER BY`或`LIMIT`的查询块，而不将其包装在括号中，但可以以各种方式使用括号进行强制执行：

+   要在每个查询块上强制执行`LIMIT`：

    ```sql
    (SELECT 1 LIMIT 1) UNION (VALUES ROW(2) LIMIT 1);

    (VALUES ROW(1), ROW(2) LIMIT 2) EXCEPT (SELECT 2 LIMIT 1);
    ```

+   要在查询块和整个查询表达式上强制执行`LIMIT`：

    ```sql
    (SELECT 1 LIMIT 1) UNION (SELECT 2 LIMIT 1) LIMIT 1;
    ```

+   要在整个查询表达式上强制执行`LIMIT`（不带括号）：

    ```sql
    VALUES ROW(1), ROW(2) INTERSECT VALUES ROW(2), ROW(1) LIMIT 1;
    ```

+   混合强制：在第一个查询块和整个查询表达式上强制执行`LIMIT`：

    ```sql
    (SELECT 1 LIMIT 1) UNION SELECT 2 LIMIT 1;
    ```

本节中描述的语法受到一定的限制：

+   如果括号内有另一个`INTO`子句，则不允许为查询表达式添加尾随的`INTO`子句。

+   在 MySQL 8.0.31 之前，当`ORDER BY`或`LIMIT`出现在带括号的查询表达式中并且也应用于外部查询时，结果是未定义的。在 MySQL 8.0.31 及更高版本中，这将按照 SQL 标准处理。

    在 MySQL 8.0.31 之前，带括号的查询表达式不允许多层次的`ORDER BY`或`LIMIT`操作，并且包含这些操作的语句将被拒绝，并显示`ER_NOT_SUPPORTED_YET`。在 MySQL 8.0.31 及更高版本中，此限制已被取消，并允许嵌套的带括号的查询表达式。支持的最大嵌套级别为 63；这是在解析器执行任何简化或合并之后。

    这里展示了这种语句的一个示例：

    ```sql
    mysql> (SELECT 'a' UNION SELECT 'b' LIMIT 2) LIMIT 3;
    +---+
    | a |
    +---+
    | a |
    | b |
    +---+
    2 rows in set (0.00 sec)
    ```

    你应该注意，在 MySQL 8.0.31 及更高版本中，当折叠括号表达式体时，MySQL 遵循 SQL 标准语义，因此更高的外部限制不能覆盖更低的内部限制。例如，`(SELECT ... LIMIT 5) LIMIT 10` 最多只能返回五行。
