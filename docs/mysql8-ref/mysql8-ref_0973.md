# 15.2.20 WITH（公共表达式）

> 原文：[`dev.mysql.com/doc/refman/8.0/en/with.html`](https://dev.mysql.com/doc/refman/8.0/en/with.html)

公共表达式（CTE）是存在于单个语句范围内的命名临时结果集，可以在该语句中稍后引用，可能多次。以下讨论描述了如何编写使用 CTE 的语句。

+   公共表达式

+   递归公共表达式

+   限制公共表达式递归

+   递归公共表达式示例

+   与类似结构比较的公共表达式

有关 CTE 优化的信息，请参见第 10.2.2.4 节，“使用合并或材料化优化派生表、视图引用和公共表达式”。

#### 其他资源

这些文章包含有关在 MySQL 中使用 CTEs 的其他信息，包括许多示例：

+   [MySQL 8.0 实验室：MySQL 中的[递归]公共表达式（CTEs）](https://dev.mysql.com/blog-archive/mysql-8-0-labs-recursive-common-table-expressions-in-mysql-ctes/)

+   [MySQL 8.0 实验室：MySQL 中的[递归]公共表达式（CTEs）第二部分 - 如何生成系列](https://dev.mysql.com/blog-archive/mysql-8-0-labs-recursive-common-table-expressions-in-mysql-ctes-part-two-how-to-generate-series/)

+   [MySQL 8.0 实验室：MySQL 中的[递归]公共表达式（CTEs）第三部分 - 层次结构](https://dev.mysql.com/blog-archive/mysql-8-0-labs-recursive-common-table-expressions-in-mysql-ctes-part-three-hierarchies/)

+   [MySQL 8.0.1：MySQL 中的[递归]公共表达式（CTEs）第四部分 - 深度优先或广度优先遍历，传递闭包，循环避免](https://dev.mysql.com/blog-archive/mysql-8-0-1-recursive-common-table-expressions-in-mysql-ctes-part-four-depth-first-or-breadth-first-traversal-transitive-closure-cycle-avoidance/)

#### 公共表达式

要指定公共表达式，请使用具有一个或多个逗号分隔子句的`WITH`")子句。每个子句提供一个生成结果集的子查询，并将一个名称与子查询关联。以下示例在`WITH`")子句中定义了名为`cte1`和`cte2`的 CTE，并在跟随`WITH`")子句的顶层`SELECT`中引用它们：

```sql
WITH
  cte1 AS (SELECT a, b FROM table1),
  cte2 AS (SELECT c, d FROM table2)
SELECT b, d FROM cte1 JOIN cte2
WHERE cte1.a = cte2.c;
```

在包含`WITH`")子句的语句中，每个 CTE 名称都可以被引用以访问相应的 CTE 结果集。

CTE 名称可以在其他 CTE 中引用，使得 CTE 可以基于其他 CTE 定义。

CTE 可以引用自身来定义递归 CTE。递归 CTE 的常见应用包括序列生成和遍历分层或树形数据。

公共表达式是 DML 语句语法的可选部分。它们使用`WITH`")子句定义：

```sql
*with_clause*:
    WITH [RECURSIVE]
        *cte_name* [(*col_name* [, *col_name*] ...)] AS (*subquery*)
        [, *cte_name* [(*col_name* [, *col_name*] ...)] AS (*subquery*)] ...
```

*`cte_name`*指定一个单个公共表达式，并且可以在包含`WITH`")子句的语句中作为表引用。

`AS (*`子查询`*)`部分中的*`子查询`*称为“CTE 的子查询”，并生成 CTE 结果集。`AS`后面的括号是必需的。

如果 CTE 的子查询引用其自身的名称，则该公共表达式是递归的。如果`WITH`")子句中的任何 CTE 是递归的，则必须包含`RECURSIVE`关键字。有关更多信息，请参见递归公共表达式。

给定 CTE 的列名确定如下：

+   如果 CTE 名称后跟着一个括号括起的名称列表，则这些名称是列名：

    ```sql
    WITH cte (col1, col2) AS
    (
      SELECT 1, 2
      UNION ALL
      SELECT 3, 4
    )
    SELECT col1, col2 FROM cte;
    ```

    名称列表中的名称数量必须与结果集中的列数相同。

+   否则，列名来自`AS (*`子查询`*)`部分中第一个`SELECT`的选择列表：

    ```sql
    WITH cte AS
    (
      SELECT 1 AS col1, 2 AS col2
      UNION ALL
      SELECT 3, 4
    )
    SELECT col1, col2 FROM cte;
    ```

在以下情况下允许使用`WITH`")子句：

+   在`SELECT`、`UPDATE`和`DELETE`语句的开头。

    ```sql
    WITH ... SELECT ...
    WITH ... UPDATE ...
    WITH ... DELETE ...
    ```

+   在子查询（包括派生表子查询）的开头：

    ```sql
    SELECT ... WHERE id IN (WITH ... SELECT ...) ...
    SELECT * FROM (WITH ... SELECT ...) AS dt ...
    ```

+   在包含`SELECT`语句的语句之前：

    ```sql
    INSERT ... WITH ... SELECT ...
    REPLACE ... WITH ... SELECT ...
    CREATE TABLE ... WITH ... SELECT ...
    CREATE VIEW ... WITH ... SELECT ...
    DECLARE CURSOR ... WITH ... SELECT ...
    EXPLAIN ... WITH ... SELECT ...
    ```

同一级别只允许一个`WITH`")子句。不允许在同一级别后跟`WITH`")再跟`WITH`")，因此这是不合法的：

```sql
WITH cte1 AS (...) WITH cte2 AS (...) SELECT ...
```

要使语句合法，使用一个单独的`WITH`")子句，通过逗号分隔子句：

```sql
WITH cte1 AS (...), cte2 AS (...) SELECT ...
```

但是，如果它们出现在不同级别，则语句可以包含多个`WITH`")子句：

```sql
WITH cte1 AS (SELECT 1)
SELECT * FROM (WITH cte2 AS (SELECT 2) SELECT * FROM cte2 JOIN cte1) AS dt;
```

`WITH`")子句可以定义一个或多个公共表达式，但每个 CTE 名称必须对该子句唯一。这是不合法的：

```sql
WITH cte1 AS (...), cte1 AS (...) SELECT ...
```

要使语句合法，定义具有唯一名称的 CTE：

```sql
WITH cte1 AS (...), cte2 AS (...) SELECT ...
```

CTE 可以引用自身或其他 CTE：

+   自引用 CTE 是递归的。

+   CTE 可以引用在同一`WITH`")子句中较早定义的 CTE，但不能引用稍后定义的 CTE。

    这个约束排除了相互递归的 CTE，其中`cte1`引用`cte2`，而`cte2`引用`cte1`。这些引用中的一个必须是对稍后定义的 CTE 的引用，这是不允许的。

+   在给定查询块中，CTE 可以引用更外层级别的查询块中定义的 CTE，但不能引用更内层级别的查询块中定义的 CTE。

对于解析具有相同名称的对象引用，派生表隐藏 CTE；而 CTE 隐藏基本表、`TEMPORARY`表和视图。名称解析通过在同一查询块中搜索对象，然后依次在外部块中查找，直到找到具有该名称的对象为止。

与派生表类似，在 MySQL 8.0.14 之前，CTE 不能包含外部引用。这是 MySQL 在 MySQL 8.0.14 中解除的限制，而不是 SQL 标准的限制。有关递归 CTE 的特定语法考虑事项，请参见递归公共表达式。

#### 递归公共表达式

递归公共表达式是具有引用其自身名称的子查询的表达式。例如：

```sql
WITH RECURSIVE cte (n) AS
(
  SELECT 1
  UNION ALL
  SELECT n + 1 FROM cte WHERE n < 5
)
SELECT * FROM cte;
```

当执行时，该语句产生这样的结果，一个包含简单线性序列的单列：

```sql
+------+
| n    |
+------+
|    1 |
|    2 |
|    3 |
|    4 |
|    5 |
+------+
```

递归 CTE 具有以下结构：

+   如果`WITH`子句中的任何 CTE 引用自身，则`WITH`子句必须以`WITH RECURSIVE`开头。（如果没有 CTE 引用自身，则允许使用`RECURSIVE`但不是必需的。）

    如果忘记为递归 CTE 添加`RECURSIVE`，则可能会出现以下错误：

    ```sql
    ERROR 1146 (42S02): Table '*cte_name*' doesn't exist
    ```

+   递归 CTE 子查询有两部分，由`UNION ALL`或[`UNION [DISTINCT]`](union.html "15.2.18 UNION Clause")分隔：

    ```sql
    SELECT ...      -- return initial row set
    UNION ALL
    SELECT ...      -- return additional row sets
    ```

    第一个 `SELECT` 生成 CTE 的初始行或行，并不引用 CTE 名称。第二个 `SELECT` 生成额外的行并通过在其 `FROM` 子句中引用 CTE 名称进行递归。当这部分不再生成新行时，递归结束。因此，递归 CTE 由一个非递归的 `SELECT` 部分后跟一个递归的 `SELECT` 部分组成。

    每个 `SELECT` 部分本身可以是多个 `SELECT` 语句的联合。

+   CTE 结果列的类型是从非递归 `SELECT` 部分的列类型推断出来的，所有列都是可空的。对于类型确定，递归 `SELECT` 部分将被忽略。

+   如果非递归部分和递归部分由 `UNION DISTINCT` 分隔，重复行将被消除。这对执行传递闭包的查询很有用，以避免无限循环。

+   递归部分的每次迭代仅在前一次迭代生成的行上操作。如果递归部分有多个查询块，每个查询块的迭代按照未指定的顺序进行安排，并且每个查询块操作的行是由其前一次迭代或自其前一次迭代结束以来其他查询块生成的行。

之前显示的递归 CTE 子查询有这个非递归部分，用于检索单行以生成初始行集合：

```sql
SELECT 1
```

CTE 子查询也有这个递归部分：

```sql
SELECT n + 1 FROM cte WHERE n < 5
```

在每次迭代中，`SELECT` 会生成一个新值，比上一行集合中的 `n` 的值大 `1`。第一次迭代操作初始行集合（`1`）并生成 `1+1=2`；第二次迭代操作第一次迭代的行集合（`2`）并生成 `2+1=3`；依此类推。直到递归结束，即当 `n` 不再小于 `5` 时。

如果 CTE 的递归部分为某列生成了比非递归部分更宽的值，可能需要扩展非递归部分中的列以避免数据截断。考虑以下语句：

```sql
WITH RECURSIVE cte AS
(
  SELECT 1 AS n, 'abc' AS str
  UNION ALL
  SELECT n + 1, CONCAT(str, str) FROM cte WHERE n < 3
)
SELECT * FROM cte;
```

在非严格 SQL 模式下，该语句会产生如下输出：

```sql
+------+------+
| n    | str  |
+------+------+
|    1 | abc  |
|    2 | abc  |
|    3 | abc  |
+------+------+
```

`str` 列的值都是 `'abc'`，因为非递归 `SELECT` 确定了列宽度。因此，递归 `SELECT` 生成的更宽的 `str` 值会被截断。

在严格 SQL 模式下，该语句会产生错误：

```sql
ERROR 1406 (22001): Data too long for column 'str' at row 1
```

为了解决这个问题，使语句不产生截断或错误，可以在非递归的`SELECT`中使用`CAST()`来使`str`列变宽：

```sql
WITH RECURSIVE cte AS
(
  SELECT 1 AS n, CAST('abc' AS CHAR(20)) AS str
  UNION ALL
  SELECT n + 1, CONCAT(str, str) FROM cte WHERE n < 3
)
SELECT * FROM cte;
```

现在该语句产生了这个结果，没有截断：

```sql
+------+--------------+
| n    | str          |
+------+--------------+
|    1 | abc          |
|    2 | abcabc       |
|    3 | abcabcabcabc |
+------+--------------+
```

列通过名称访问，而不是位置，这意味着递归部分中的列可以访问非递归部分中位置不同的列，正如这个 CTE 所示：

```sql
WITH RECURSIVE cte AS
(
  SELECT 1 AS n, 1 AS p, -1 AS q
  UNION ALL
  SELECT n + 1, q * 2, p * 2 FROM cte WHERE n < 5
)
SELECT * FROM cte;
```

因为一行中的`p`是从前一行中的`q`派生的，反之亦然，所以输出的每一行中正负值交换位置：

```sql
+------+------+------+
| n    | p    | q    |
+------+------+------+
|    1 |    1 |   -1 |
|    2 |   -2 |    2 |
|    3 |    4 |   -4 |
|    4 |   -8 |    8 |
|    5 |   16 |  -16 |
+------+------+------+
```

一些语法约束适用于递归 CTE 子查询：

+   递归`SELECT`部分不能包含以下结构：

    +   聚合函数如`SUM()`

    +   窗口函数

    +   `GROUP BY`

    +   `ORDER BY`

    +   `DISTINCT`

    在 MySQL 8.0.19 之前，递归 CTE 的递归`SELECT`部分也不能使用`LIMIT`子句。这个限制在 MySQL 8.0.19 中被解除，现在在这种情况下支持`LIMIT`，还有一个可选的`OFFSET`子句。结果集的效果与在最外层`SELECT`中使用`LIMIT`时相同，但更有效，因为在递归`SELECT`中使用它会在生成请求的行数后立即停止生成行。

    这些约束不适用于递归 CTE 的非递归`SELECT`部分。对`DISTINCT`的禁止仅适用于`UNION`成员；允许`UNION DISTINCT`。

+   递归的`SELECT`部分必须仅在其`FROM`子句中引用 CTE 一次，而且不能在任何子查询中引用。它可以引用除 CTE 之外的其他表，并将它们与 CTE 连接起来。如果在这样的连接中使用，CTE 不能位于`LEFT JOIN`的右侧。

这些约束来自 SQL 标准，除了 MySQL 特定的排除`ORDER BY`、`LIMIT`（MySQL 8.0.18 及更早版本）和`DISTINCT`之外。

对于递归 CTE，`EXPLAIN`输出的递归`SELECT`部分在`Extra`列中显示`Recursive`。

`EXPLAIN`显示的成本估计表示每次迭代的成本，这可能与总成本有很大差异。优化器无法预测迭代次数，因为它无法预测`WHERE`子句何时变为 false。

CTE 的实际成本也可能受到结果集大小的影响。产生许多行的 CTE 可能需要一个足够大的内部临时表，以便从内存转换为磁盘格式，并可能遭受性能损失。如果是这样，增加允许的内存临时表大小可能会提高性能；参见第 10.4.4 节，“MySQL 中的内部临时表使用”。

#### 限制公共表达式递归

对于递归 CTE 非常重要的是，递归的`SELECT`部分包含一个终止递归的条件。作为一种开发技术，防止递归 CTE 失控的方法是通过设置执行时间限制来强制终止：

+   `cte_max_recursion_depth`系统变量为 CTE 的递归级别数量设置了限制。服务器会终止任何递归超过此变量值的 CTE 的执行。

+   `max_execution_time`系统变量强制执行当前会话中执行的`SELECT`语句的执行超时。

+   `MAX_EXECUTION_TIME`优化提示为其中出现的`SELECT`语句强制执行每个查询的执行超时。

假设递归 CTE 被错误地编写为没有递归执行终止条件：

```sql
WITH RECURSIVE cte (n) AS
(
  SELECT 1
  UNION ALL
  SELECT n + 1 FROM cte
)
SELECT * FROM cte;
```

默认情况下，`cte_max_recursion_depth`的值为 1000，当递归超过 1000 级时，CTE 会终止。应用程序可以更改会话值以满足其需求：

```sql
SET SESSION cte_max_recursion_depth = 10;      -- permit only shallow recursion
SET SESSION cte_max_recursion_depth = 1000000; -- permit deeper recursion
```

你也可以设置全局`cte_max_recursion_depth`值，以影响随后开始的所有会话。

对于执行缓慢且因此递归的查询，或者有理由将`cte_max_recursion_depth`值设置得非常高的情况，另一种防止深度递归的方法是设置每个会话的超时时间。为此，在执行 CTE 语句之前执行类似以下语句：

```sql
SET max_execution_time = 1000; -- impose one second timeout
```

或者，在 CTE 语句本身中包含一个优化提示：

```sql
WITH RECURSIVE cte (n) AS
(
  SELECT 1
  UNION ALL
  SELECT n + 1 FROM cte
)
SELECT /*+ SET_VAR(cte_max_recursion_depth = 1M) */ * FROM cte;

WITH RECURSIVE cte (n) AS
(
  SELECT 1
  UNION ALL
  SELECT n + 1 FROM cte
)
SELECT /*+ MAX_EXECUTION_TIME(1000) */ * FROM cte;
```

从 MySQL 8.0.19 开始，你也可以在递归查询中使用`LIMIT`来对最外层的`SELECT`返回的最大行数进行限制，例如：

```sql
WITH RECURSIVE cte (n) AS
(
  SELECT 1
  UNION ALL
  SELECT n + 1 FROM cte LIMIT 10000
)
SELECT * FROM cte;
```

你可以在设置时间限制之外或代替设置时间限制。因此，以下 CTE 在返回一万行或运行一秒钟（1000 毫秒）后终止：

```sql
WITH RECURSIVE cte (n) AS
(
  SELECT 1
  UNION ALL
  SELECT n + 1 FROM cte LIMIT 10000
)
SELECT /*+ MAX_EXECUTION_TIME(1000) */ * FROM cte;
```

如果没有执行时间限制的递归查询进入无限循环，您可以从另一个会话中使用`KILL QUERY`终止它。在会话本身中，用于运行查询的客户端程序可能提供了终止查询的方法。例如，在**mysql**中，键入**Control+C**会中断当前语句。

#### 递归公共表达式示例

如前所述，递归公共表达式（CTEs）经常用于生成系列和遍历分层或树形结构数据。本节展示了这些技术的一些简单示例。

+   斐波那契数列生成

+   日期系列生成

+   分层数据遍历

##### 斐波那契数列生成

斐波那契数列以数字 0 和 1（或 1 和 1）开始，之后每个数字都是前两个数字的和。如果递归`SELECT`生成的每行都可以访问系列中前两个数字，则递归公共表达式可以生成斐波那契数列。以下 CTE 使用 0 和 1 作为前两个数字生成了一个包含 10 个数字的系列：

```sql
WITH RECURSIVE fibonacci (n, fib_n, next_fib_n) AS
(
  SELECT 1, 0, 1
  UNION ALL
  SELECT n + 1, next_fib_n, fib_n + next_fib_n
    FROM fibonacci WHERE n < 10
)
SELECT * FROM fibonacci;
```

CTE 生成了这个结果：

```sql
+------+-------+------------+
| n    | fib_n | next_fib_n |
+------+-------+------------+
|    1 |     0 |          1 |
|    2 |     1 |          1 |
|    3 |     1 |          2 |
|    4 |     2 |          3 |
|    5 |     3 |          5 |
|    6 |     5 |          8 |
|    7 |     8 |         13 |
|    8 |    13 |         21 |
|    9 |    21 |         34 |
|   10 |    34 |         55 |
+------+-------+------------+
```

CTE 的工作原理如下：

+   `n`是一个显示列，表示该行包含第`n`个斐波那契数。例如，第 8 个斐波那契数是 13。

+   `fib_n`列显示斐波那契数`n`。

+   `next_fib_n`列显示数字`n`后面的下一个斐波那契数。该列为下一行提供了下一个系列值，以便该行可以在其`fib_n`列中生成前两个系列值的和。

+   当`n`达到 10 时，递归结束。这是一个任意选择，以限制输出为一小组行。

前面的输出显示了整个 CTE 结果。要仅选择其中的一部分，请在顶层`SELECT`中添加适当的`WHERE`子句。例如，要选择第 8 个斐波那契数，执行以下操作：

```sql
mysql> WITH RECURSIVE fibonacci ...
       ...
       SELECT fib_n FROM fibonacci WHERE n = 8;
+-------+
| fib_n |
+-------+
|    13 |
+-------+
```

##### 日期系列生成

公共表达式可以生成一系列连续的日期，这对于生成包含系列中所有日期的行的摘要非常有用，包括未在摘要数据中表示的日期。

假设销售数字表包含以下行：

```sql
mysql> SELECT * FROM sales ORDER BY date, price;
+------------+--------+
| date       | price  |
+------------+--------+
| 2017-01-03 | 100.00 |
| 2017-01-03 | 200.00 |
| 2017-01-06 |  50.00 |
| 2017-01-08 |  10.00 |
| 2017-01-08 |  20.00 |
| 2017-01-08 | 150.00 |
| 2017-01-10 |   5.00 |
+------------+--------+
```

此查询总结了每天的销售情况：

```sql
mysql> SELECT date, SUM(price) AS sum_price
       FROM sales
       GROUP BY date
       ORDER BY date;
+------------+-----------+
| date       | sum_price |
+------------+-----------+
| 2017-01-03 |    300.00 |
| 2017-01-06 |     50.00 |
| 2017-01-08 |    180.00 |
| 2017-01-10 |      5.00 |
+------------+-----------+
```

然而，该结果对于表跨越的日期范围中未表示的日期存在“空洞”。可以使用递归 CTE 生成该日期集合，然后与销售数据进行`LEFT JOIN`连接以生成表示范围内所有日期的结果。

这是生成日期范围系列的 CTE：

```sql
WITH RECURSIVE dates (date) AS
(
  SELECT MIN(date) FROM sales
  UNION ALL
  SELECT date + INTERVAL 1 DAY FROM dates
  WHERE date + INTERVAL 1 DAY <= (SELECT MAX(date) FROM sales)
)
SELECT * FROM dates;
```

CTE 生成此结果：

```sql
+------------+
| date       |
+------------+
| 2017-01-03 |
| 2017-01-04 |
| 2017-01-05 |
| 2017-01-06 |
| 2017-01-07 |
| 2017-01-08 |
| 2017-01-09 |
| 2017-01-10 |
+------------+
```

CTE 的工作原理：

+   非递归`SELECT`生成`sales`表跨度日期范围内最早的日期。

+   递归`SELECT`生成的每行将日期增加一天到前一行生成的日期。

+   当日期达到`sales`表跨度的日期范围内的最高日期时，递归结束。

将 CTE 与`sales`表进行`LEFT JOIN`连接，生成每个日期范围内的销售摘要：

```sql
WITH RECURSIVE dates (date) AS
(
  SELECT MIN(date) FROM sales
  UNION ALL
  SELECT date + INTERVAL 1 DAY FROM dates
  WHERE date + INTERVAL 1 DAY <= (SELECT MAX(date) FROM sales)
)
SELECT dates.date, COALESCE(SUM(price), 0) AS sum_price
FROM dates LEFT JOIN sales ON dates.date = sales.date
GROUP BY dates.date
ORDER BY dates.date;
```

输出如下所示：

```sql
+------------+-----------+
| date       | sum_price |
+------------+-----------+
| 2017-01-03 |    300.00 |
| 2017-01-04 |      0.00 |
| 2017-01-05 |      0.00 |
| 2017-01-06 |     50.00 |
| 2017-01-07 |      0.00 |
| 2017-01-08 |    180.00 |
| 2017-01-09 |      0.00 |
| 2017-01-10 |      5.00 |
+------------+-----------+
```

一些需要注意的要点：

+   查询是否低效，特别是对于递归`SELECT`中每行执行的包含`MAX()`子查询？`EXPLAIN`显示包含`MAX()`的子查询仅评估一次，并且结果被缓存。

+   使用`COALESCE()`避免在`sales`表中没有销售数据的日期中在`sum_price`列中显示`NULL`。

##### 分层数据遍历

递归通用表达式对于遍历形成层次结构的数据非常有用。考虑以下创建一个小数据集的语句，该数据集显示公司中每个员工的员工姓名和 ID 号，以及员工的经理的 ID。顶层员工（CEO）的经理 ID 为`NULL`（没有经理）。

```sql
CREATE TABLE employees (
  id         INT PRIMARY KEY NOT NULL,
  name       VARCHAR(100) NOT NULL,
  manager_id INT NULL,
  INDEX (manager_id),
FOREIGN KEY (manager_id) REFERENCES employees (id)
);
INSERT INTO employees VALUES
(333, "Yasmina", NULL),  # Yasmina is the CEO (manager_id is NULL)
(198, "John", 333),      # John has ID 198 and reports to 333 (Yasmina)
(692, "Tarek", 333),
(29, "Pedro", 198),
(4610, "Sarah", 29),
(72, "Pierre", 29),
(123, "Adil", 692);
```

结果数据集如下所示：

```sql
mysql> SELECT * FROM employees ORDER BY id;
+------+---------+------------+
| id   | name    | manager_id |
+------+---------+------------+
|   29 | Pedro   |        198 |
|   72 | Pierre  |         29 |
|  123 | Adil    |        692 |
|  198 | John    |        333 |
|  333 | Yasmina |       NULL |
|  692 | Tarek   |        333 |
| 4610 | Sarah   |         29 |
+------+---------+------------+
```

要为每个员工生成组织结构图和每个员工的管理链（即从 CEO 到员工的路径），请使用递归 CTE：

```sql
WITH RECURSIVE employee_paths (id, name, path) AS
(
  SELECT id, name, CAST(id AS CHAR(200))
    FROM employees
    WHERE manager_id IS NULL
  UNION ALL
  SELECT e.id, e.name, CONCAT(ep.path, ',', e.id)
    FROM employee_paths AS ep JOIN employees AS e
      ON ep.id = e.manager_id
)
SELECT * FROM employee_paths ORDER BY path;
```

CTE 生成此输出：

```sql
+------+---------+-----------------+
| id   | name    | path            |
+------+---------+-----------------+
|  333 | Yasmina | 333             |
|  198 | John    | 333,198         |
|   29 | Pedro   | 333,198,29      |
| 4610 | Sarah   | 333,198,29,4610 |
|   72 | Pierre  | 333,198,29,72   |
|  692 | Tarek   | 333,692         |
|  123 | Adil    | 333,692,123     |
+------+---------+-----------------+
```

CTE 的工作原理：

+   非递归`SELECT`生成 CEO 的行（具有`NULL`管理者 ID 的行）。

    `path`列扩展为`CHAR(200)`，以确保递归`SELECT`生成的较长`path`值有足够的空间。

+   递归`SELECT`生成的每行找到所有直接向前一行生成的员工报告的员工。对于每个这样的员工，行包括员工 ID 和姓名，以及员工的管理链。链是经理的链，员工 ID 添加到末尾。

+   当员工没有其他人向他们报告时，递归结束。

要查找特定员工或员工的路径，请在顶层`SELECT`添加`WHERE`子句。例如，要显示 Tarek 和 Sarah 的结果，请修改`SELECT`如下：

```sql
mysql> WITH RECURSIVE ...
       ...
       SELECT * FROM employees_extended
       WHERE id IN (692, 4610)
       ORDER BY path;
+------+-------+-----------------+
| id   | name  | path            |
+------+-------+-----------------+
| 4610 | Sarah | 333,198,29,4610 |
|  692 | Tarek | 333,692         |
+------+-------+-----------------+
```

#### 与类似结构比较的通用表达式

通用表达式（CTEs）在某些方面类似于派生表：

+   两种结构都有名称。

+   两种结构存在于单个语句的范围内。

由于这些相似之处，CTE 和派生表通常可以互换使用。作为一个简单的例子，以下语句是等价的：

```sql
WITH cte AS (SELECT 1) SELECT * FROM cte;
SELECT * FROM (SELECT 1) AS dt;
```

然而，CTE 相对于派生表有一些优势：

+   派生表只能在查询中引用一次。而 CTE 可以被多次引用。要使用派生表结果的多个实例，必须多次派生结果。

+   一个 CTE 可以是自引用的（递归的）。

+   一个 CTE 可以引用另一个 CTE。

+   一个 CTE 在语句中的定义出现在开头时可能更容易阅读，而不是嵌入其中。

CTE 类似于使用[`CREATE [TEMPORARY] TABLE`](create-table.html "15.1.20 CREATE TABLE Statement")创建的表，但不需要显式定义或删除。对于 CTE，您无需拥有创建表的权限。
