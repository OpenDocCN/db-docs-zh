# 14.19.2 GROUP BY 修饰符

> 原文：[`dev.mysql.com/doc/refman/8.0/en/group-by-modifiers.html`](https://dev.mysql.com/doc/refman/8.0/en/group-by-modifiers.html)

`GROUP BY`子句允许使用`WITH ROLLUP`修饰符，导致摘要输出包括代表更高级别（即超级聚合）摘要操作的额外行。因此，`ROLLUP`使您能够使用单个查询回答多个分析级别的问题。例如，`ROLLUP`可用于支持 OLAP（在线分析处理）操作。

假设一个`sales`表有`year`、`country`、`product`和`profit`列用于记录销售利润：

```sql
CREATE TABLE sales
(
    year    INT,
    country VARCHAR(20),
    product VARCHAR(32),
    profit  INT
);
```

要按年份总结表内容，请使用简单的`GROUP BY`，如下所示：

```sql
mysql> SELECT year, SUM(profit) AS profit
       FROM sales
       GROUP BY year;
+------+--------+
| year | profit |
+------+--------+
| 2000 |   4525 |
| 2001 |   3010 |
+------+--------+
```

输出显示每年的总利润。要确定所有年份总利润的总和，您必须自己加总各个值或运行另一个查询。或者您可以使用`ROLLUP`，它通过单个查询提供了两个级别的分析。在`GROUP BY`子句中添加`WITH ROLLUP`修饰符会导致查询生成另一行（超级聚合行），显示所有年份值的总计：

```sql
mysql> SELECT year, SUM(profit) AS profit
       FROM sales
       GROUP BY year WITH ROLLUP;
+------+--------+
| year | profit |
+------+--------+
| 2000 |   4525 |
| 2001 |   3010 |
| NULL |   7535 |
+------+--------+
```

`year`列中的`NULL`值标识总计超级聚合行。

`ROLLUP`在有多个`GROUP BY`列时具有更复杂的效果。在这种情况下，每当除最后一个分组列之外的任何列的值发生变化时，查询都会生成一个额外的超级聚合摘要行。

例如，没有`ROLLUP`，基于`year`、`country`和`product`的`sales`表摘要可能如下所示，其中输出仅指示年/国家/产品分析级别的摘要值：

```sql
mysql> SELECT year, country, product, SUM(profit) AS profit
       FROM sales
       GROUP BY year, country, product;
+------+---------+------------+--------+
| year | country | product    | profit |
+------+---------+------------+--------+
| 2000 | Finland | Computer   |   1500 |
| 2000 | Finland | Phone      |    100 |
| 2000 | India   | Calculator |    150 |
| 2000 | India   | Computer   |   1200 |
| 2000 | USA     | Calculator |     75 |
| 2000 | USA     | Computer   |   1500 |
| 2001 | Finland | Phone      |     10 |
| 2001 | USA     | Calculator |     50 |
| 2001 | USA     | Computer   |   2700 |
| 2001 | USA     | TV         |    250 |
+------+---------+------------+--------+
```

添加了`ROLLUP`后，查询会生成几行额外的行：

```sql
mysql> SELECT year, country, product, SUM(profit) AS profit
       FROM sales
       GROUP BY year, country, product WITH ROLLUP;
+------+---------+------------+--------+
| year | country | product    | profit |
+------+---------+------------+--------+
| 2000 | Finland | Computer   |   1500 |
| 2000 | Finland | Phone      |    100 |
| 2000 | Finland | NULL       |   1600 |
| 2000 | India   | Calculator |    150 |
| 2000 | India   | Computer   |   1200 |
| 2000 | India   | NULL       |   1350 |
| 2000 | USA     | Calculator |     75 |
| 2000 | USA     | Computer   |   1500 |
| 2000 | USA     | NULL       |   1575 |
| 2000 | NULL    | NULL       |   4525 |
| 2001 | Finland | Phone      |     10 |
| 2001 | Finland | NULL       |     10 |
| 2001 | USA     | Calculator |     50 |
| 2001 | USA     | Computer   |   2700 |
| 2001 | USA     | TV         |    250 |
| 2001 | USA     | NULL       |   3000 |
| 2001 | NULL    | NULL       |   3010 |
| NULL | NULL    | NULL       |   7535 |
+------+---------+------------+--------+
```

现在输出包括四个层次的分析摘要信息，而不仅仅是一个：

+   对于给定年份和国家的每组产品行之后，会出现一个额外的超级聚合摘要行，显示所有产品的总计。这些行的`product`列设置为`NULL`。

+   对于给定年份的每组行之后，会出现一个额外的超级聚合摘要行，显示所有国家和产品的总计。这些行的`country`和`products`列设置为`NULL`。

+   最后，在所有其他行之后，会出现一个额外的超级聚合摘要行，显示所有年份、国家和产品的总计。此行的`year`、`country`和`products`列设置为`NULL`。

每个超级聚合行中的`NULL`指示符在将行发送到客户端时生成。服务器查看在第一个发生值变化的最左边的`GROUP BY`子句中命名的列。对于结果集中的任何列，其名称与这些名称中的任何一个匹配，其值都设置为`NULL`。（如果按列位置指定分组列，则服务器通过位置确定要设置为`NULL`的列。）

因为在超级聚合行中的`NULL`值是在查询处理的最后阶段放入结果集中的，所以你只能在选择列表或`HAVING`子句中将它们作为`NULL`值进行测试。你不能在连接条件或`WHERE`子句中将它们作为`NULL`值进行测试，以确定选择哪些行。例如，你不能在查询中添加`WHERE product IS NULL`来从输出中消除除了超级聚合行之外的所有行。

`NULL`值在客户端显示为`NULL`，可以使用任何 MySQL 客户端编程接口进行测试。然而，在这一点上，你无法区分`NULL`是代表常规分组值还是超级聚合值。要测试区分，使用稍后描述的`GROUPING()`函数。

以前，MySQL 不允许在具有`WITH ROLLUP`选项的查询中使用`DISTINCT`或`ORDER BY`。这个限制在 MySQL 8.0.12 及更高版本中被取消。（Bug #87450，Bug #86311，Bug #26640100，Bug #26073513）

对于`GROUP BY ... WITH ROLLUP`查询，为了测试结果中的`NULL`值是否代表超级聚合值，可以在选择列表、`HAVING`子句和（从 MySQL 8.0.12 开始）`ORDER BY`子句中使用`GROUPING()`函数。例如，`GROUPING(year)`在`year`列中的`NULL`出现在超级聚合行时返回 1，否则返回 0。类似地，`GROUPING(country)`和`GROUPING(product)`分别在`country`和`product`列中的超级聚合`NULL`值时返回 1：

```sql
mysql> SELECT
         year, country, product, SUM(profit) AS profit,
         GROUPING(year) AS grp_year,
         GROUPING(country) AS grp_country,
         GROUPING(product) AS grp_product
       FROM sales
       GROUP BY year, country, product WITH ROLLUP;
+------+---------+------------+--------+----------+-------------+-------------+
| year | country | product    | profit | grp_year | grp_country | grp_product |
+------+---------+------------+--------+----------+-------------+-------------+
| 2000 | Finland | Computer   |   1500 |        0 |           0 |           0 |
| 2000 | Finland | Phone      |    100 |        0 |           0 |           0 |
| 2000 | Finland | NULL       |   1600 |        0 |           0 |           1 |
| 2000 | India   | Calculator |    150 |        0 |           0 |           0 |
| 2000 | India   | Computer   |   1200 |        0 |           0 |           0 |
| 2000 | India   | NULL       |   1350 |        0 |           0 |           1 |
| 2000 | USA     | Calculator |     75 |        0 |           0 |           0 |
| 2000 | USA     | Computer   |   1500 |        0 |           0 |           0 |
| 2000 | USA     | NULL       |   1575 |        0 |           0 |           1 |
| 2000 | NULL    | NULL       |   4525 |        0 |           1 |           1 |
| 2001 | Finland | Phone      |     10 |        0 |           0 |           0 |
| 2001 | Finland | NULL       |     10 |        0 |           0 |           1 |
| 2001 | USA     | Calculator |     50 |        0 |           0 |           0 |
| 2001 | USA     | Computer   |   2700 |        0 |           0 |           0 |
| 2001 | USA     | TV         |    250 |        0 |           0 |           0 |
| 2001 | USA     | NULL       |   3000 |        0 |           0 |           1 |
| 2001 | NULL    | NULL       |   3010 |        0 |           1 |           1 |
| NULL | NULL    | NULL       |   7535 |        1 |           1 |           1 |
+------+---------+------------+--------+----------+-------------+-------------+
```

你可以使用`GROUPING()`来替换超级聚合`NULL`值的标签，而不是直接显示`GROUPING()`的结果：

```sql
mysql> SELECT
         IF(GROUPING(year), 'All years', year) AS year,
         IF(GROUPING(country), 'All countries', country) AS country,
         IF(GROUPING(product), 'All products', product) AS product,
         SUM(profit) AS profit
       FROM sales
       GROUP BY year, country, product WITH ROLLUP;
+-----------+---------------+--------------+--------+
| year      | country       | product      | profit |
+-----------+---------------+--------------+--------+
| 2000      | Finland       | Computer     |   1500 |
| 2000      | Finland       | Phone        |    100 |
| 2000      | Finland       | All products |   1600 |
| 2000      | India         | Calculator   |    150 |
| 2000      | India         | Computer     |   1200 |
| 2000      | India         | All products |   1350 |
| 2000      | USA           | Calculator   |     75 |
| 2000      | USA           | Computer     |   1500 |
| 2000      | USA           | All products |   1575 |
| 2000      | All countries | All products |   4525 |
| 2001      | Finland       | Phone        |     10 |
| 2001      | Finland       | All products |     10 |
| 2001      | USA           | Calculator   |     50 |
| 2001      | USA           | Computer     |   2700 |
| 2001      | USA           | TV           |    250 |
| 2001      | USA           | All products |   3000 |
| 2001      | All countries | All products |   3010 |
| All years | All countries | All products |   7535 |
+-----------+---------------+--------------+--------+
```

带有多个表达式参数的`GROUPING()`函数返回一个结果，表示将每个表达式的结果组合在一起的位掩码，最低位对应最右边表达式的结果。例如，`GROUPING(year, country, product)`的计算如下：

```sql
 result for GROUPING(*product*)
+ result for GROUPING(*country*) << 1
+ result for GROUPING(*year*) << 2
```

这样的`GROUPING()`的结果非零，如果任何表达式代表超级聚合`NULL`，那么你可以只返回超级聚合行并过滤掉常规分组行，如下所示：

```sql
mysql> SELECT year, country, product, SUM(profit) AS profit
       FROM sales
       GROUP BY year, country, product WITH ROLLUP
       HAVING GROUPING(year, country, product) <> 0;
+------+---------+---------+--------+
| year | country | product | profit |
+------+---------+---------+--------+
| 2000 | Finland | NULL    |   1600 |
| 2000 | India   | NULL    |   1350 |
| 2000 | USA     | NULL    |   1575 |
| 2000 | NULL    | NULL    |   4525 |
| 2001 | Finland | NULL    |     10 |
| 2001 | USA     | NULL    |   3000 |
| 2001 | NULL    | NULL    |   3010 |
| NULL | NULL    | NULL    |   7535 |
+------+---------+---------+--------+
```

`sales` 表中不包含 `NULL` 值，因此 `ROLLUP` 结果中的所有 `NULL` 值都代表超级聚合值。当数据集包含 `NULL` 值时，`ROLLUP` 汇总可能不仅在超级聚合行中包含 `NULL` 值，还可能在常规分组行中包含 `NULL` 值。`GROUPING()` 可以帮助区分它们。假设表 `t1` 包含一个简单的数据集，其中有两个用于一组数量值的分组因素，其中 `NULL` 表示类似于“其他”或“未知”：

```sql
mysql> SELECT * FROM t1;
+------+-------+----------+
| name | size  | quantity |
+------+-------+----------+
| ball | small |       10 |
| ball | large |       20 |
| ball | NULL  |        5 |
| hoop | small |       15 |
| hoop | large |        5 |
| hoop | NULL  |        3 |
+------+-------+----------+
```

简单的 `ROLLUP` 操作会产生这些结果，在其中很难区分超级聚合行中的 `NULL` 值和常规分组行中的 `NULL` 值：

```sql
mysql> SELECT name, size, SUM(quantity) AS quantity
       FROM t1
       GROUP BY name, size WITH ROLLUP;
+------+-------+----------+
| name | size  | quantity |
+------+-------+----------+
| ball | NULL  |        5 |
| ball | large |       20 |
| ball | small |       10 |
| ball | NULL  |       35 |
| hoop | NULL  |        3 |
| hoop | large |        5 |
| hoop | small |       15 |
| hoop | NULL  |       23 |
| NULL | NULL  |       58 |
+------+-------+----------+
```

使用 `GROUPING()` 来替换超级聚合 `NULL` 值的标签使结果更容易解释：

```sql
mysql> SELECT
         IF(GROUPING(name) = 1, 'All items', name) AS name,
         IF(GROUPING(size) = 1, 'All sizes', size) AS size,
         SUM(quantity) AS quantity
       FROM t1
       GROUP BY name, size WITH ROLLUP;
+-----------+-----------+----------+
| name      | size      | quantity |
+-----------+-----------+----------+
| ball      | NULL      |        5 |
| ball      | large     |       20 |
| ball      | small     |       10 |
| ball      | All sizes |       35 |
| hoop      | NULL      |        3 |
| hoop      | large     |        5 |
| hoop      | small     |       15 |
| hoop      | All sizes |       23 |
| All items | All sizes |       58 |
+-----------+-----------+----------+
```

#### 使用 ROLLUP 时的其他考虑事项

以下讨论列出了 MySQL 实现 `ROLLUP` 的一些特定行为。

在 MySQL 8.0.12 之前，当使用 `ROLLUP` 时，不能同时使用 `ORDER BY` 子句对结果进行排序。换句话说，在 MySQL 中，`ROLLUP` 和 `ORDER BY` 是互斥的。然而，你仍然可以在排序顺序上有一定的控制。为了绕过不能将 `ROLLUP` 与 `ORDER BY` 结合使用的限制，并实现对分组结果的特定排序顺序，可以将分组结果集生成为派生表，并对其应用 `ORDER BY`。例如：

```sql
mysql> SELECT * FROM
         (SELECT year, SUM(profit) AS profit
         FROM sales GROUP BY year WITH ROLLUP) AS dt
       ORDER BY year DESC;
+------+--------+
| year | profit |
+------+--------+
| 2001 |   3010 |
| 2000 |   4525 |
| NULL |   7535 |
+------+--------+
```

截至 MySQL 8.0.12，`ORDER BY` 和 `ROLLUP` 可以一起使用，这使得可以使用 `ORDER BY` 和 `GROUPING()` 来实现对分组结果的特定排序顺序。例如：

```sql
mysql> SELECT year, SUM(profit) AS profit
       FROM sales
       GROUP BY year WITH ROLLUP
       ORDER BY GROUPING(year) DESC;
+------+--------+
| year | profit |
+------+--------+
| NULL |   7535 |
| 2000 |   4525 |
| 2001 |   3010 |
+------+--------+
```

在这两种情况下，超级聚合摘要行与它们计算出的行一起排序，并且它们的放置取决于排序顺序（升序排序时在末尾，降序排序时在开头）。

`LIMIT` 可以用于限制返回给客户端的行数。`LIMIT` 应用于 `ROLLUP` 之后，因此限制适用于 `ROLLUP` 添加的额外行。例如：

```sql
mysql> SELECT year, country, product, SUM(profit) AS profit
       FROM sales
       GROUP BY year, country, product WITH ROLLUP
       LIMIT 5;
+------+---------+------------+--------+
| year | country | product    | profit |
+------+---------+------------+--------+
| 2000 | Finland | Computer   |   1500 |
| 2000 | Finland | Phone      |    100 |
| 2000 | Finland | NULL       |   1600 |
| 2000 | India   | Calculator |    150 |
| 2000 | India   | Computer   |   1200 |
+------+---------+------------+--------+
```

使用 `LIMIT` 与 `ROLLUP` 可能会产生更难解释的结果，因为对于理解超级聚合行来说，上下文更少。

MySQL 的一个扩展允许在选择列表中命名不在 `GROUP BY` 列表中出现的列。（有关非聚合列和 `GROUP BY` 的信息，请参阅 第 14.19.3 节，“MySQL 对 GROUP BY 的处理”。）在这种情况下，服务器可以自由选择摘要行中来自此非聚合列的任何值，包括 `WITH ROLLUP` 添加的额外行。例如，在以下查询中，`country` 是一个非聚合列，不在 `GROUP BY` 列表中出现，为此列选择的值是不确定的：

```sql
mysql> SELECT year, country, SUM(profit) AS profit
       FROM sales
       GROUP BY year WITH ROLLUP;
+------+---------+--------+
| year | country | profit |
+------+---------+--------+
| 2000 | India   |   4525 |
| 2001 | USA     |   3010 |
| NULL | USA     |   7535 |
+------+---------+--------+
```

当未启用`ONLY_FULL_GROUP_BY` SQL 模式时，允许这种行为。如果启用了该模式，服务器会因为`country`未在`GROUP BY`子句中列出而拒绝查询。启用`ONLY_FULL_GROUP_BY`后，您仍可以通过对非确定性值列使用`ANY_VALUE()`函数来执行查询：

```sql
mysql> SELECT year, ANY_VALUE(country) AS country, SUM(profit) AS profit
       FROM sales
       GROUP BY year WITH ROLLUP;
+------+---------+--------+
| year | country | profit |
+------+---------+--------+
| 2000 | India   |   4525 |
| 2001 | USA     |   3010 |
| NULL | USA     |   7535 |
+------+---------+--------+
```

在 MySQL 8.0.28 及更高版本中，`rollup` 列不能作为`MATCH()`的参数（并将被拒绝并显示错误），除非在`WHERE`子句中调用。有关更多信息，请参见第 14.9 节，“全文搜索函数”。
