> 原文：[`dev.mysql.com/doc/refman/8.0/en/join.html`](https://dev.mysql.com/doc/refman/8.0/en/join.html)

#### 15.2.13.2 连接子句

MySQL 支持以下`JOIN`语法用于`SELECT`语句的*`table_references`*部分以及多表`DELETE`和`UPDATE`语句：

```sql
*table_references:*
    *escaped_table_reference* [, *escaped_table_reference*] ...

*escaped_table_reference*: {
    *table_reference*
  | { OJ *table_reference* }
}

*table_reference*: {
    *table_factor*
  | *joined_table*
}

*table_factor*: {
    *tbl_name* [PARTITION (*partition_names*)]
        [[AS] *alias*] [*index_hint_list*]
  | [LATERAL] *table_subquery* [AS] *alias* [(*col_list*)]
  | ( *table_references* )
}

*joined_table*: {
    *table_reference* {[INNER | CROSS] JOIN | STRAIGHT_JOIN} *table_factor* [*join_specification*]
  | *table_reference* {LEFT|RIGHT} [OUTER] JOIN *table_reference* *join_specification*
  | *table_reference* NATURAL [INNER | {LEFT|RIGHT} [OUTER]] JOIN *table_factor*
}

*join_specification*: {
    ON *search_condition*
  | USING (*join_column_list*)
}

*join_column_list*:
    *column_name* [, *column_name*] ...

*index_hint_list*:
    *index_hint* [, *index_hint*] ...

*index_hint*: {
    USE {INDEX|KEY}
      [FOR {JOIN|ORDER BY|GROUP BY}] ([*index_list*])
  | {IGNORE|FORCE} {INDEX|KEY}
      [FOR {JOIN|ORDER BY|GROUP BY}] (*index_list*)
}

*index_list*:
    *index_name* [, *index_name*] ...
```

表引用也被称为连接表达式。

表引用（当它引用分区表时）可以包含一个`PARTITION`子句，包括一个逗号分隔的分区、子分区列表，或两者。此选项跟随表名之后，并在任何别名声明之前。此选项的效果是仅从列出的分区或子分区中选择行。未在列表中命名的任何分区或子分区将被忽略。有关更多信息和示例，请参见第 26.5 节，“分区选择”。

与标准 SQL 相比，MySQL 中*`table_factor`*的语法得到了扩展。标准只接受*`table_reference`*，而不是在括号中包含它们的列表。

如果将*`table_reference`*项目列表中的每个逗号视为等同于内连接，则这是一种保守的扩展。例如：

```sql
SELECT * FROM t1 LEFT JOIN (t2, t3, t4)
                 ON (t2.a = t1.a AND t3.b = t1.b AND t4.c = t1.c)
```

等同于：

```sql
SELECT * FROM t1 LEFT JOIN (t2 CROSS JOIN t3 CROSS JOIN t4)
                 ON (t2.a = t1.a AND t3.b = t1.b AND t4.c = t1.c)
```

在 MySQL 中，`JOIN`、`CROSS JOIN`和`INNER JOIN`是语法上的等效（它们可以互相替换）。在标准 SQL 中，它们不是等效的。`INNER JOIN`与`ON`子句一起使用，否则使用`CROSS JOIN`。

通常情况下，在仅包含内连接操作的连接表达式中，括号可以忽略。MySQL 还支持嵌套连接。参见第 10.2.1.8 节，“嵌套连接优化”。

可以指定索引提示以影响 MySQL 优化器如何使用索引。有关更多信息，请参见第 10.9.4 节，“索引提示”。优化器提示和`optimizer_switch`系统变量是影响优化器使用索引的其他方法。请参见第 10.9.3 节，“优化器提示”和第 10.9.2 节，“可切换优化”。

下面的列表描述了编写连接时需要考虑的一般因素：

+   可以使用`*`tbl_name`* AS *`alias_name`*`或*`tbl_name alias_name`*为表引用取别名：

    ```sql
    SELECT t1.name, t2.salary
      FROM employee AS t1 INNER JOIN info AS t2 ON t1.name = t2.name;

    SELECT t1.name, t2.salary
      FROM employee t1 INNER JOIN info t2 ON t1.name = t2.name;
    ```

+   *`table_subquery`*也被称为`FROM`子句中的派生表或子查询。参见第 15.2.15.8 节，“派生表”。这样的子查询*必须*包含一个别名，以给子查询结果一个表名，并且可以选择在括号中包含一个表列名列表。以下是一个简单的示例：

    ```sql
    SELECT * FROM (SELECT 1, 2, 3) AS t1;
    ```

+   在单个连接中引用的最大表数为 61。这包括通过将 `FROM` 子句中的派生表和视图合并到外部查询块中处理的连接（参见 Section 10.2.2.4, “Optimizing Derived Tables, View References, and Common Table Expressions with Merging or Materialization”）。

+   在没有连接条件的情况下，`INNER JOIN` 和 `,`（逗号）在语义上是等效的：两者都会在指定的表之间产生笛卡尔积（即，第一个表中的每一行都与第二个表中的每一行连接）。

    然而，逗号运算符的优先级低于 `INNER JOIN`、`CROSS JOIN`、`LEFT JOIN` 等。如果在存在连接条件时混合使用逗号连接和其他连接类型，则可能会出现类似 `Unknown column '*`col_name`*' in 'on clause'` 的错误。有关处理此问题的信息稍后在本节中给出。

+   与 `ON` 一起使用的 *`search_condition`* 是可以在 `WHERE` 子句中使用的任何条件表达式的形式。通常，`ON` 子句用于指定如何连接表，而 `WHERE` 子句用于限制结果集中包含哪些行。

+   如果在 `LEFT JOIN` 中右表的 `ON` 或 `USING` 部分中没有匹配的行，则会使用所有列均设置为 `NULL` 的行作为右表。您可以利用这一点找到一个表中没有对应的另一个表中的行：

    ```sql
    SELECT left_tbl.*
      FROM left_tbl LEFT JOIN right_tbl ON left_tbl.id = right_tbl.id
      WHERE right_tbl.id IS NULL;
    ```

    此示例查找 `left_tbl` 中所有具有不在 `right_tbl` 中存在的 `id` 值的行（即，所有在 `right_tbl` 中没有对应行的 `left_tbl` 中的所有行）。参见 Section 10.2.1.9, “Outer Join Optimization”。

+   `USING(*`join_column_list`*)` 子句命名了两个表中必须存在的列的列表。如果表 `a` 和 `b` 都包含列 `c1`、`c2` 和 `c3`，则以下连接将比较来自两个表的对应列：

    ```sql
    a LEFT JOIN b USING (c1, c2, c3)
    ```

+   两个表的 `NATURAL [LEFT] JOIN` 被定义为与使用命名了两个表中所有列的 `USING` 子句的 `INNER JOIN` 或 `LEFT JOIN` 在���义上等效。

+   `RIGHT JOIN` 的工作方式类似于 `LEFT JOIN`。为了保持代码在各种数据库中的可移植性，建议您使用 `LEFT JOIN` 而不是 `RIGHT JOIN`。

+   在连接语法描述中显示的 `{ OJ ... }` 语法仅用于与 ODBC 的兼容性。语法中的大括号应该按照字面意义写入；它们不是在其他语法描述中使用的元语法。

    ```sql
    SELECT left_tbl.*
        FROM { OJ left_tbl LEFT OUTER JOIN right_tbl
               ON left_tbl.id = right_tbl.id }
        WHERE right_tbl.id IS NULL;
    ```

    您可以在 `{ OJ ... }` 中使用其他类型的连接，例如 `INNER JOIN` 或 `RIGHT OUTER JOIN`。这有助于与一些第三方应用程序的兼容性，但不是官方的 ODBC 语法。

+   `STRAIGHT_JOIN`类似于`JOIN`，不同之处在于左表始终在右表之前读取。这可以用于那些（少数）情况下，连接优化器以次优顺序处理表的情况。

一些连接示例：

```sql
SELECT * FROM table1, table2;

SELECT * FROM table1 INNER JOIN table2 ON table1.id = table2.id;

SELECT * FROM table1 LEFT JOIN table2 ON table1.id = table2.id;

SELECT * FROM table1 LEFT JOIN table2 USING (id);

SELECT * FROM table1 LEFT JOIN table2 ON table1.id = table2.id
  LEFT JOIN table3 ON table2.id = table3.id;
```

根据 SQL:2003 标准处理`NATURAL`连接和带有`USING`的连接，包括外连接变体：

+   `NATURAL`连接的冗余列不会出现。考虑以下一组语句：

    ```sql
    CREATE TABLE t1 (i INT, j INT);
    CREATE TABLE t2 (k INT, j INT);
    INSERT INTO t1 VALUES(1, 1);
    INSERT INTO t2 VALUES(1, 1);
    SELECT * FROM t1 NATURAL JOIN t2;
    SELECT * FROM t1 JOIN t2 USING (j);
    ```

    在第一个`SELECT`语句中，列`j`出现在两个表中，因此成为连接列，因此，根据标准 SQL，它应该在输出中只出现一次，而不是两次。类似地，在第二个 SELECT 语句中，列`j`在`USING`子句中命名，应该在输出中只出现一次，而不是两次。

    因此，这些语句产生这个输出：

    ```sql
    +------+------+------+
    | j    | i    | k    |
    +------+------+------+
    |    1 |    1 |    1 |
    +------+------+------+
    +------+------+------+
    | j    | i    | k    |
    +------+------+------+
    |    1 |    1 |    1 |
    +------+------+------+
    ```

    根据标准 SQL 进行冗余列消除和列排序，产生这个显示顺序：

    +   首先，按照它们在第一个表中出现的顺序，合并两个连接表的共同列

    +   第二，第一个表中独有的列，按照它们在该表中出现的顺序

    +   第三，第二个表中独有的列，按照它们在该表中出现的顺序

    替换两个共同列的单个结果列是使用合并操作定义的。也就是说，对于两个`t1.a`和`t2.a`，生成的单个连接列`a`被定义为`a = COALESCE(t1.a, t2.a)`，其中：

    ```sql
    COALESCE(x, y) = (CASE WHEN x IS NOT NULL THEN x ELSE y END)
    ```

    如果连接操作是任何其他连接，则连接的结果列由连接表的所有列的串联组成。

    合并列的定义的一个结果是，对于外连接，如果两个列中的一个始终为`NULL`，则合并列包含非`NULL`列的值。如果两个列都不是`NULL`或都是`NULL`，那么两个共同列具有相同的值，因此选择哪个作为合并列的值并不重要。解释这个的一个简单方法是将外连接的合并列表示为`JOIN`的内表的共同列。假设表`t1(a, b)`和表`t2(a, c)`具有以下内容：

    ```sql
    t1    t2
    ----  ----
    1 x   2 z
    2 y   3 w
    ```

    然后，对于这个连接，列`a`包含`t1.a`的值：

    ```sql
    mysql> SELECT * FROM t1 NATURAL LEFT JOIN t2;
    +------+------+------+
    | a    | b    | c    |
    +------+------+------+
    |    1 | x    | NULL |
    |    2 | y    | z    |
    +------+------+------+
    ```

    相比之下，对于这个连接，列`a`包含`t2.a`的值。

    ```sql
    mysql> SELECT * FROM t1 NATURAL RIGHT JOIN t2;
    +------+------+------+
    | a    | c    | b    |
    +------+------+------+
    |    2 | z    | y    |
    |    3 | w    | NULL |
    +------+------+------+
    ```

    将这些结果与使用`JOIN ... ON`的等效查询进行比较：

    ```sql
    mysql> SELECT * FROM t1 LEFT JOIN t2 ON (t1.a = t2.a);
    +------+------+------+------+
    | a    | b    | a    | c    |
    +------+------+------+------+
    |    1 | x    | NULL | NULL |
    |    2 | y    |    2 | z    |
    +------+------+------+------+
    ```

    ```sql
    mysql> SELECT * FROM t1 RIGHT JOIN t2 ON (t1.a = t2.a);
    +------+------+------+------+
    | a    | b    | a    | c    |
    +------+------+------+------+
    |    2 | y    |    2 | z    |
    | NULL | NULL |    3 | w    |
    +------+------+------+------+
    ```

+   `USING`子句可以重写为比较相应列的`ON`子句。然而，尽管`USING`和`ON`类似，但它们并不完全相同。考虑以下两个查询：

    ```sql
    a LEFT JOIN b USING (c1, c2, c3)
    a LEFT JOIN b ON a.c1 = b.c1 AND a.c2 = b.c2 AND a.c3 = b.c3
    ```

    就确定哪些行满足连接条件而言，这两个连接在语义上是相同的。

    关于确定要显示哪些列进行`SELECT *`扩展，这两个连接在语义上并不相同。`USING`连接选择对应列的合并值，而`ON`连接选择所有表中的所有列。对于`USING`连接，`SELECT *`选择这些值：

    ```sql
    COALESCE(a.c1, b.c1), COALESCE(a.c2, b.c2), COALESCE(a.c3, b.c3)
    ```

    对于`ON`连接，`SELECT *`选择这些值：

    ```sql
    a.c1, a.c2, a.c3, b.c1, b.c2, b.c3
    ```

    在内连接中，`COALESCE(a.c1, b.c1)`与`a.c1`或`b.c1`相同，因为两列的值相同。在外连接（如`LEFT JOIN`）中，两列中的一个可以是`NULL`。该列将从结果中省略。

+   `ON`子句只能引用其操作数。

    示例：

    ```sql
    CREATE TABLE t1 (i1 INT);
    CREATE TABLE t2 (i2 INT);
    CREATE TABLE t3 (i3 INT);
    SELECT * FROM t1 JOIN t2 ON (i1 = i3) JOIN t3;
    ```

    该语句因为`i3`是`t3`中的列，而不是`ON`子句的操作数而失败，会出现`Unknown column 'i3' in 'on clause'`错误。要使连接能够被处理，请将语句重写如下：

    ```sql
    SELECT * FROM t1 JOIN t2 JOIN t3 ON (i1 = i3);
    ```

+   `JOIN`比逗号运算符（`,`）具有更高的优先级，因此连接表达式`t1, t2 JOIN t3`被解释为`(t1, (t2 JOIN t3))`，而不是`((t1, t2) JOIN t3)`。这会影响使用`ON`子句的语句，因为该子句只能引用连接操作数中的列，而优先级会影响这些操作数的解释。

    示例：

    ```sql
    CREATE TABLE t1 (i1 INT, j1 INT);
    CREATE TABLE t2 (i2 INT, j2 INT);
    CREATE TABLE t3 (i3 INT, j3 INT);
    INSERT INTO t1 VALUES(1, 1);
    INSERT INTO t2 VALUES(1, 1);
    INSERT INTO t3 VALUES(1, 1);
    SELECT * FROM t1, t2 JOIN t3 ON (t1.i1 = t3.i3);
    ```

    `JOIN`优先于逗号运算符，因此`ON`子句的操作数为`t2`和`t3`。因为`t1.i1`不是任何操作数中的列，结果是一个`Unknown column 't1.i1' in 'on clause'`错误。

    要使连接能够被处理，可以使用以下策略之一：

    +   使用括号明确地将前两个表分组，以便`ON`子句的操作数为`(t1, t2)`和`t3`：

        ```sql
        SELECT * FROM (t1, t2) JOIN t3 ON (t1.i1 = t3.i3);
        ```

    +   避免使用逗号运算符，改用`JOIN`代替：

        ```sql
        SELECT * FROM t1 JOIN t2 JOIN t3 ON (t1.i1 = t3.i3);
        ```

    相同的优先级解释也适用于混合逗号运算符与`INNER JOIN`、`CROSS JOIN`、`LEFT JOIN`和`RIGHT JOIN`的语句，所有这些连接比逗号运算符具有更高的优先级。

+   与 SQL:2003 标准相比，MySQL 的一个扩展是允许您对`NATURAL`或`USING`连接的共同（合并的）列进行限定，而标准则不允许。
