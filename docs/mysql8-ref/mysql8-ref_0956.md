# 15.2.15 子查询

> 原文：[`dev.mysql.com/doc/refman/8.0/en/subqueries.html`](https://dev.mysql.com/doc/refman/8.0/en/subqueries.html)

15.2.15.1 子查询作为标量操作数

15.2.15.2 使用子查询进行比较

15.2.15.3 使用 ANY、IN 或 SOME 的子查询

15.2.15.4 使用 ALL 的子查询

15.2.15.5 行子查询

15.2.15.6 使用 EXISTS 或 NOT EXISTS 的子查询

15.2.15.7 相关子查询

15.2.15.8 派生表

15.2.15.9 横向派生表

15.2.15.10 子查询错误

15.2.15.11 优化子查询

15.2.15.12 子查询的限制

子查询是另一个语句内的`SELECT`语句。

所有 SQL 标准要求的子查询形式和操作都得到支持，以及一些 MySQL 特有的功能。

这里是一个子查询的示例：

```sql
SELECT * FROM t1 WHERE column1 = (SELECT column1 FROM t2);
```

在这个示例中，`SELECT * FROM t1 ...`是*外部查询*（或*外部语句*），`(SELECT column1 FROM t2)`是*子查询*。我们说子查询*嵌套*在外部查询中，实际上可以在其他子查询中嵌套子查询，深度相当大。子查询必须始终出现在括号内。

子查询的主要优点是：

+   它们允许查询*结构化*，以便可以隔离语句的每个部分。

+   它们提供了执行通常需要复杂连接和联合的操作的替代方法。

+   许多人发现子查询比复杂的连接或联合更易读。事实上，正是子查询的创新给人们最初的想法，称早期的 SQL 为“结构化查询语言”。

这里是一个示例语句，展示了 SQL 标准规定的子查询语法的主要要点，并在 MySQL 中得到支持：

```sql
DELETE FROM t1
WHERE s11 > ANY
 (SELECT COUNT(*) /* no hint */ FROM t2
  WHERE NOT EXISTS
   (SELECT * FROM t3
    WHERE ROW(5*t2.s1,77)=
     (SELECT 50,11*s1 FROM t4 UNION SELECT 50,77 FROM
      (SELECT * FROM t5) AS t5)));
```

子查询可以返回标量（单个值）、单行、单列或表（一个或多个列的一个或多行）。这些称为标量、列、行和表子查询。通常只能在特定上下文中使用返回特定类型结果的子查询，如下节所述。

子查询可以在哪些类型的语句中使用没有太多限制。子查询可以包含许多普通`SELECT`可以包含的关键字或子句：`DISTINCT`、`GROUP BY`、`ORDER BY`、`LIMIT`、连接、索引提示、`UNION`构造、注释、函数等等。

从 MySQL 8.0.19 开始，`TABLE`和`VALUES`语句可以在子查询中使用。使用`VALUES`的子查询通常是更冗长的子查询版本，可以使用集合表示法更简洁地重写，或者使用`SELECT`或`TABLE`语法；假设表`ts`是使用语句`CREATE TABLE ts VALUES ROW(2), ROW(4), ROW(6)`创建的，这里显示的语句都是等效的：

```sql
SELECT * FROM tt
    WHERE b > ANY (VALUES ROW(2), ROW(4), ROW(6));

SELECT * FROM tt
    WHERE b > ANY (SELECT * FROM ts);

SELECT * FROM tt
    WHERE b > ANY (TABLE ts);
```

`TABLE`子查询的示例将在接下来的章节中展示。

子查询的外部语句可以是任何一个：`SELECT`, `INSERT`, `UPDATE`, `DELETE`, `SET`, 或 `DO`。

有关优化器如何处理子查询的信息，请参阅第 10.2.2 节，“优化子查询、派生表、视图引用和公共表达式”。有关子查询使用的限制讨论，包括某些形式子查询语法的性能问题，请参阅第 15.2.15.12 节，“子查询的限制”。
