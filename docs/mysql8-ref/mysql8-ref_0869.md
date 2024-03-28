# 14.20.2 窗口函数概念和语法

> 原文：[`dev.mysql.com/doc/refman/8.0/en/window-functions-usage.html`](https://dev.mysql.com/doc/refman/8.0/en/window-functions-usage.html)

本节描述了如何使用窗口函数。示例使用与第 14.19.2 节，“GROUP BY 修饰符”中的`GROUPING()`函数讨论中找到的相同销售信息数据集：

```sql
mysql> SELECT * FROM sales ORDER BY country, year, product;
+------+---------+------------+--------+
| year | country | product    | profit |
+------+---------+------------+--------+
| 2000 | Finland | Computer   |   1500 |
| 2000 | Finland | Phone      |    100 |
| 2001 | Finland | Phone      |     10 |
| 2000 | India   | Calculator |     75 |
| 2000 | India   | Calculator |     75 |
| 2000 | India   | Computer   |   1200 |
| 2000 | USA     | Calculator |     75 |
| 2000 | USA     | Computer   |   1500 |
| 2001 | USA     | Calculator |     50 |
| 2001 | USA     | Computer   |   1500 |
| 2001 | USA     | Computer   |   1200 |
| 2001 | USA     | TV         |    150 |
| 2001 | USA     | TV         |    100 |
+------+---------+------------+--------+
```

窗口函数在一组查询行上执行类似聚合的操作。然而，聚合操作将查询行分组为单个结果行，而窗口函数为每个查询行产生一个结果：

+   函数评估发生的行称为当前行。

+   与函数评估相关的当前行的查询行构成了当前行的窗口。

例如，使用销售信息表，这两个查询执行产生所有行作为一组的单个全局总和的聚合操作，以及按国家分组的总和：

```sql
mysql> SELECT SUM(profit) AS total_profit
       FROM sales;
+--------------+
| total_profit |
+--------------+
|         7535 |
+--------------+
mysql> SELECT country, SUM(profit) AS country_profit
       FROM sales
       GROUP BY country
       ORDER BY country;
+---------+----------------+
| country | country_profit |
+---------+----------------+
| Finland |           1610 |
| India   |           1350 |
| USA     |           4575 |
+---------+----------------+
```

相比之下，窗口操作不会将查询行的组合折叠为单个输出行。相反，它们为每一行产生一个结果。与前面的查询一样，以下查询使用`SUM()`，但这次作为一个窗口函数：

```sql
mysql> SELECT
         year, country, product, profit,
         SUM(profit) OVER() AS total_profit,
         SUM(profit) OVER(PARTITION BY country) AS country_profit
       FROM sales
       ORDER BY country, year, product, profit;
+------+---------+------------+--------+--------------+----------------+
| year | country | product    | profit | total_profit | country_profit |
+------+---------+------------+--------+--------------+----------------+
| 2000 | Finland | Computer   |   1500 |         7535 |           1610 |
| 2000 | Finland | Phone      |    100 |         7535 |           1610 |
| 2001 | Finland | Phone      |     10 |         7535 |           1610 |
| 2000 | India   | Calculator |     75 |         7535 |           1350 |
| 2000 | India   | Calculator |     75 |         7535 |           1350 |
| 2000 | India   | Computer   |   1200 |         7535 |           1350 |
| 2000 | USA     | Calculator |     75 |         7535 |           4575 |
| 2000 | USA     | Computer   |   1500 |         7535 |           4575 |
| 2001 | USA     | Calculator |     50 |         7535 |           4575 |
| 2001 | USA     | Computer   |   1200 |         7535 |           4575 |
| 2001 | USA     | Computer   |   1500 |         7535 |           4575 |
| 2001 | USA     | TV         |    100 |         7535 |           4575 |
| 2001 | USA     | TV         |    150 |         7535 |           4575 |
+------+---------+------------+--------+--------------+----------------+
```

查询中的每个窗口操作都通过包含指定如何将查询行分组以供窗口函数处理的`OVER`子句来表示：

+   第一个`OVER`子句为空，这将整个查询行集视为单个分区。因此，窗口函数为每一行产生一个全局总和。

+   第二个`OVER`子句按国家对行进行分区，为每个分区（每个国家）产生一个总和。该函数为每个分区行产生这个总和。

窗口函数仅允许在选择列表和`ORDER BY`子句中使用。查询结果行是从`FROM`子句中确定的，在`WHERE`、`GROUP BY`和`HAVING`处理之后，窗口执行发生在`ORDER BY`、`LIMIT`和`SELECT DISTINCT`之前。

`OVER`子句允许许多聚合函数，因此可以根据`OVER`子句的存在与否将其用作窗口函数或非窗口函数：

```sql
AVG()
BIT_AND()
BIT_OR()
BIT_XOR()
COUNT()
JSON_ARRAYAGG()
JSON_OBJECTAGG()
MAX()
MIN()
STDDEV_POP(), STDDEV(), STD()
STDDEV_SAMP()
SUM()
VAR_POP(), VARIANCE()
VAR_SAMP()
```

对于每个聚合函数的详细信息，请参见第 14.19.1 节，“聚合函数描述”。

MySQL 还支持仅用作窗口函数的非聚合函数。对于这些函数，`OVER`子句是强制的：

```sql
CUME_DIST()
DENSE_RANK()
FIRST_VALUE()
LAG()
LAST_VALUE()
LEAD()
NTH_VALUE()
NTILE()
PERCENT_RANK()
RANK()
ROW_NUMBER()
```

对于每个非聚合函数的详细信息，请参见第 14.20.1 节，“窗口函数描述”。

作为那些非聚合窗口函数之一的示例，此查询使用`ROW_NUMBER()`，它生成每个分区内每行的行号。在本例中，行按国家编号。默认情况下，分区行是无序的，行编号是不确定的。要对分区行进行排序，请在窗口定义中包含一个`ORDER BY`子句。查询使用无序和有序分区（`row_num1`和`row_num2`列）来说明省略和包含`ORDER BY`之间的差异：

```sql
mysql> SELECT
         year, country, product, profit,
         ROW_NUMBER() OVER(PARTITION BY country) AS row_num1,
         ROW_NUMBER() OVER(PARTITION BY country ORDER BY year, product) AS row_num2
       FROM sales;
+------+---------+------------+--------+----------+----------+
| year | country | product    | profit | row_num1 | row_num2 |
+------+---------+------------+--------+----------+----------+
| 2000 | Finland | Computer   |   1500 |        2 |        1 |
| 2000 | Finland | Phone      |    100 |        1 |        2 |
| 2001 | Finland | Phone      |     10 |        3 |        3 |
| 2000 | India   | Calculator |     75 |        2 |        1 |
| 2000 | India   | Calculator |     75 |        3 |        2 |
| 2000 | India   | Computer   |   1200 |        1 |        3 |
| 2000 | USA     | Calculator |     75 |        5 |        1 |
| 2000 | USA     | Computer   |   1500 |        4 |        2 |
| 2001 | USA     | Calculator |     50 |        2 |        3 |
| 2001 | USA     | Computer   |   1500 |        3 |        4 |
| 2001 | USA     | Computer   |   1200 |        7 |        5 |
| 2001 | USA     | TV         |    150 |        1 |        6 |
| 2001 | USA     | TV         |    100 |        6 |        7 |
+------+---------+------------+--------+----------+----------+
```

如前所述，要使用窗口函数（或将聚合函数视为窗口函数），请在函数调用后包含一个`OVER`子句。`OVER`子句有两种形式：

```sql
*over_clause*:
    {OVER (*window_spec*) | OVER *window_name*}
```

这两种形式定义了窗口函数如何处理查询行。它们的区别在于窗口是直接在`OVER`子句中定义，还是通过引用在查询中其他地方定义的命名窗口提供：

+   在第一种情况下，窗口规范直接出现在括号之间的`OVER`子句中。

+   在第二种情况下，*`window_name`*是查询中其他地方由`WINDOW`子句定义的窗口规范的名称。有关详细信息，请参见第 14.20.4 节，“命名窗口”。

对于`OVER (*`window_spec`*)`语法，窗口规范有几个部分，都是可选的：

```sql
*window_spec*:
    [*window_name*] [*partition_clause*] [*order_clause*] [*frame_clause*]
```

如果`OVER()`为空，则窗口包含所有查询行，窗口函数使用所有行计算结果。否则，括号内的子句确定用于计算函数结果的查询行以及它们如何分区和排序：

+   *`window_name`*：查询中其他地方由`WINDOW`子句定义的窗口的名称。如果*`window_name`*单独出现在`OVER`子句中，它完全定义了窗口。如果还提供了分区、排序或帧子句，则它们修改了命名窗口的解释。有关详细信息，请参见第 14.20.4 节，“命名窗口”。

+   *`partition_clause`*：`PARTITION BY`子句指示如何将查询行分成组。给定行的窗口函数结果基于包含该行的分区的行。如果省略`PARTITION BY`，则有一个包含所有查询行的单个分区。

    注意

    窗口函数的分区与表分区不同。有关表分区的信息，请参见第二十六章，*分区*。

    *`partition_clause`*的语法如下：

    ```sql
    *partition_clause*:
        PARTITION BY *expr* [, *expr*] ...
    ```

    标准 SQL 要求 `PARTITION BY` 后面只能跟列名。MySQL 的扩展允许表达式，而不仅仅是列名。例如，如果一个表包含名为 `ts` 的 `TIMESTAMP` 列，标准 SQL 允许 `PARTITION BY ts`，但不允许 `PARTITION BY HOUR(ts)`，而 MySQL 允许两者。

+   *`order_clause`*: `ORDER BY` 子句指示如何对每个分区的行进行排序。根据 `ORDER BY` 子句相等的分区行被视为对等。如果省略 `ORDER BY`，分区行是无序的，没有暗示任何处理顺序，并且所有分区行都是对等的。

    *`order_clause`* 的语法如下：

    ```sql
    *order_clause*:
        ORDER BY *expr* [ASC|DESC] [, *expr* [ASC|DESC]] ...
    ```

    每个 `ORDER BY` 表达式可选择跟随 `ASC` 或 `DESC` 表示排序方向。如果未指定方向，则默认为 `ASC`。对于升序排序，`NULL` 值排在最前面，对于降序排序，排在最后面。

    窗口定义中的 `ORDER BY` 适用于各个分区。要对整个结果集进行排序，请在查询顶层包含一个 `ORDER BY`。

+   *`frame_clause`*: 一个框架是当前分区的子集，框架子句指定如何定义这个子集。框架子句有许多自己的子句。详情请参见 Section 14.20.3, “窗口函数框架规范”。
