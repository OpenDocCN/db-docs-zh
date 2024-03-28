# 26.4 分区修剪

> 原文：[`dev.mysql.com/doc/refman/8.0/en/partitioning-pruning.html`](https://dev.mysql.com/doc/refman/8.0/en/partitioning-pruning.html)

分区修剪优化基于一个相对简单的概念，可以描述为“不要扫描那些不可能有匹配值的分区”。假设通过以下语句创建了一个分区表`t1`：

```sql
CREATE TABLE t1 (
    fname VARCHAR(50) NOT NULL,
    lname VARCHAR(50) NOT NULL,
    region_code TINYINT UNSIGNED NOT NULL,
    dob DATE NOT NULL
)
PARTITION BY RANGE( region_code ) (
    PARTITION p0 VALUES LESS THAN (64),
    PARTITION p1 VALUES LESS THAN (128),
    PARTITION p2 VALUES LESS THAN (192),
    PARTITION p3 VALUES LESS THAN MAXVALUE
);
```

假设您希望从类似于以下语句的`SELECT`语句中获取结果：

```sql
SELECT fname, lname, region_code, dob
    FROM t1
    WHERE region_code > 125 AND region_code < 130;
```

很容易看出，应该返回的行中没有任何位于分区`p0`或`p3`中的行；也就是说，我们只需要在分区`p1`和`p2`中搜索匹配的行。通过限制搜索范围，可以比扫描表中的所有分区更少的时间和精力来找到匹配的行。这种“剪掉”不需要的分区的操作称为修剪。当优化器在执行此查询时可以利用分区修剪时，查询的执行速度可能比针对包含相同列定义和数据的非分区表的相同查询快一个数量级。

优化器可以在`WHERE`条件可以简化为以下两种情况之一时执行修剪：

+   `*`partition_column`* = *`constant`*`

+   `*`partition_column`* IN (*`constant1`*, *`constant2`*, ..., *`constantN`*)`

在第一种情况下，优化器简单地对给定值评估分区表达式，确定包含该值的分区，并仅扫描此分区。在许多情况下，等号可以替换为另一个算术比较，包括`<`、`>`、`<=`、`>=`和`<>`。一些在`WHERE`子句中使用`BETWEEN`的查询也可以利用分区修剪。请参见本节后面的示例。

在第二种情况下，优化器对列表中的每个值评估分区表达式，创建一个匹配分区的列表，然后仅扫描此分区列表中的分区。

`SELECT`、`DELETE`和`UPDATE`语句支持分区修剪。`INSERT`语句也仅访问每个插入行的一个分区；即使对于使用`HASH`或`KEY`进行分区的表，这也是正确的，尽管目前在`EXPLAIN`的输出中没有显示。

修剪也可以应用于短范围，优化器可以将其转换为等效值列表。例如，在前面的例子中，`WHERE` 子句可以转换为 `WHERE region_code IN (126, 127, 128, 129)`。然后优化器可以确定列表中的前两个值位于分区 `p1` 中，剩下的两个值位于分区 `p2` 中，其他分区不包含相关值，因此不需要搜索匹配行。

优化器还可以对使用 `RANGE COLUMNS` 或 `LIST COLUMNS` 分区的表上涉及多列比较的 `WHERE` 条件执行修剪。

只要分区表达式由相等性或可减少为一组相等性的范围组成，或者分区表达式表示递增或递减关系，就可以应用这种优化。当分区表达式使用 `YEAR()` 或 `TO_DAYS()` 函数时，也可以应用修剪到基于 `DATE` 或 `DATETIME` 列分区的表。当分区表达式使用 `TO_SECONDS()` 函数时，也可以应用修剪到这样的表。

假设表 `t2`，根据一个 `DATE` 列进行分区，是使用以下语句创建的：

```sql
CREATE TABLE t2 (
    fname VARCHAR(50) NOT NULL,
    lname VARCHAR(50) NOT NULL,
    region_code TINYINT UNSIGNED NOT NULL,
    dob DATE NOT NULL
)
PARTITION BY RANGE( YEAR(dob) ) (
    PARTITION d0 VALUES LESS THAN (1970),
    PARTITION d1 VALUES LESS THAN (1975),
    PARTITION d2 VALUES LESS THAN (1980),
    PARTITION d3 VALUES LESS THAN (1985),
    PARTITION d4 VALUES LESS THAN (1990),
    PARTITION d5 VALUES LESS THAN (2000),
    PARTITION d6 VALUES LESS THAN (2005),
    PARTITION d7 VALUES LESS THAN MAXVALUE
);
```

以下使用 `t2` 的语句可以利用分区修剪：

```sql
SELECT * FROM t2 WHERE dob = '1982-06-23';

UPDATE t2 SET region_code = 8 WHERE dob BETWEEN '1991-02-15' AND '1997-04-25';

DELETE FROM t2 WHERE dob >= '1984-06-21' AND dob <= '1999-06-21'
```

对于最后一条语句，优化器还可以执行以下操作：

1.  *找到包含范围低端的分区*。

    `YEAR('1984-06-21')` 返回值为 `1984`，该值位于分区 `d3` 中。

1.  *找到包含范围高端的分区*。

    `YEAR('1999-06-21')` 计算结果为 `1999`，该值位于分区 `d5` 中。

1.  *仅扫描这两个分区以及可能位于它们之间的任何分区*。

    在这种情况下，这意味着只有分区 `d3`、`d4` 和 `d5` 被扫描。其余分区可以安全地被忽略（并且被忽略）。

重要提示

对于针对分区表的语句中 `WHERE` 条件引用的无效 `DATE` 和 `DATETIME` 值，将被视为 `NULL`。这意味着像 `SELECT * FROM *`partitioned_table`* WHERE *`date_column`* < '2008-12-00'` 这样的查询不会返回任何值（参见 Bug #40972）。

到目前为止，我们只看过使用 `RANGE` 分区的示例，但修剪也可以应用于其他分区类型。

考虑一个由`LIST`分区的表，其中分区表达式是递增或递减的，例如这里显示的表`t3`。（在本例中，为简洁起见，我们假设`region_code`列的值限定在 1 到 10 之间。）

```sql
CREATE TABLE t3 (
    fname VARCHAR(50) NOT NULL,
    lname VARCHAR(50) NOT NULL,
    region_code TINYINT UNSIGNED NOT NULL,
    dob DATE NOT NULL
)
PARTITION BY LIST(region_code) (
    PARTITION r0 VALUES IN (1, 3),
    PARTITION r1 VALUES IN (2, 5, 8),
    PARTITION r2 VALUES IN (4, 9),
    PARTITION r3 VALUES IN (6, 7, 10)
);
```

对于像`SELECT * FROM t3 WHERE region_code BETWEEN 1 AND 3`这样的语句，优化器确定值 1、2 和 3 在哪些分区中找到（`r0`和`r1`），并跳过其余的分区（`r2`和`r3`）。

对于由`HASH`或`[LINEAR] KEY`分区的表，如果`WHERE`子句使用简单的`=`关系针对分区表达式中使用的列，则也可以进行分区修剪。考虑创建如下表：

```sql
CREATE TABLE t4 (
    fname VARCHAR(50) NOT NULL,
    lname VARCHAR(50) NOT NULL,
    region_code TINYINT UNSIGNED NOT NULL,
    dob DATE NOT NULL
)
PARTITION BY KEY(region_code)
PARTITIONS 8;
```

将列值与常量进行比较的语句可以被修剪：

```sql
UPDATE t4 WHERE region_code = 7;
```

对于短范围，修剪也可以用于，因为优化器可以将这种条件转换为`IN`关系。例如，使用之前定义的相同表`t4`，可以修剪以下查询：

```sql
SELECT * FROM t4 WHERE region_code > 2 AND region_code < 6;

SELECT * FROM t4 WHERE region_code BETWEEN 3 AND 5;
```

在这两种情况下，优化器将`WHERE`子句转换为`WHERE region_code IN (3, 4, 5)`。

重要

只有当范围大小小于分区数时才使用这种优化。考虑这个语句：

```sql
DELETE FROM t4 WHERE region_code BETWEEN 4 AND 12;
```

`WHERE`子句中的范围涵盖了 9 个值（4, 5, 6, 7, 8, 9, 10, 11, 12），但`t4`只有 8 个分区。这意味着`DELETE`无法被修剪。

当表由`HASH`或`[LINEAR] KEY`分区时，修剪只能用于整数列。例如，这个语句不能使用修剪，因为`dob`是一个`DATE`列：

```sql
SELECT * FROM t4 WHERE dob >= '2001-04-14' AND dob <= '2005-10-15';
```

然而，如果表在`INT`列中存储年份值，则查询`WHERE year_col >= 2001 AND year_col <= 2005`可以被修剪。

使用提供自动分区的存储引擎的表，例如 MySQL Cluster 使用的`NDB`存储引擎，如果它们被显式分区，则可以被修剪。
