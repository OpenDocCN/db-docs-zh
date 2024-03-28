# 15.2.14 使用 UNION、INTERSECT 和 EXCEPT 的集合操作

> 原文：[`dev.mysql.com/doc/refman/8.0/en/set-operations.html`](https://dev.mysql.com/doc/refman/8.0/en/set-operations.html)

+   结果集列名和数据类型

+   使用 TABLE 和 VALUES 语句进行集合操作

+   使用 DISTINCT 和 ALL 的集合操作

+   使用 ORDER BY 和 LIMIT 的集合操作

+   集合操作的限制

SQL 集合操作将多个查询块的结果合并为单个结果。查询块，有时也称为简单表，是任何返回结果集的 SQL 语句，例如 `SELECT`。MySQL 8.0（8.0.19 及更高版本）还支持 `TABLE` 和 `VALUES` 语句。有关这些语句的详细信息，请参见本章其他部分中的各自描述。

SQL 标准定义了以下三种集合操作：

+   `UNION`：将两个查询块的所有结果合并为单个结果，省略任何重复项。

+   `INTERSECT`：仅合并两个查询块结果中共有的行，省略任何重复项。

+   `EXCEPT`：对于两个查询块 *`A`* 和 *`B`*，返回 *`A`* 中不在 *`B`* 中出现的所有结果，省略任何重复项。

    （一些数据库系统，如 Oracle，使用 `MINUS` 作为此运算符的名称。MySQL 不支持此功能。）

MySQL 长期支持 `UNION`；MySQL 8.0 添加了对 `INTERSECT` 和 `EXCEPT` 的支持（MySQL 8.0.31 及更高版本）。

每个集合运算符都支持 `ALL` 修饰符。当 `ALL` 关键字跟随一个集合运算符时，这会导致结果中包含重复项。有关更多信息和示例，请参阅涵盖各个运算符的以下部分。

所有三个集合运算符还支持 `DISTINCT` 关键字，用于在结果中消除重复项。由于这是集合运算符的默认行为，通常不需要显式指定 `DISTINCT`。

一般来说，查询块和集合操作可以以任意数量和顺序组合。这里展示了一个大大简化的表示：

```sql
*query_block* [*set_op* *query_block*] [*set_op* *query_block*] ...

*query_block*:
    SELECT | TABLE | VALUES

*set_op*:
    UNION | INTERSECT | EXCEPT
```

这可以更准确地表示，并更详细地描述如下：

```sql
*query_expression*:
  [*with_clause*] /* WITH clause */ 
  *query_expression_body*
  [*order_by_clause*] [*limit_clause*] [*into_clause*]

*query_expression_body*:
    *query_term*
 |  *query_expression_body* UNION [ALL | DISTINCT] *query_term*
 |  *query_expression_body* EXCEPT [ALL | DISTINCT] *query_term*

*query_term*:
    *query_primary*
 |  *query_term* INTERSECT [ALL | DISTINCT] *query_primary*

*query_primary*:
    *query_block*
 |  '(' *query_expression_body* [*order_by_clause*] [*limit_clause*] [*into_clause*] ')'

*query_block*:   /* also known as a simple table */
    *query_specification*                     /* SELECT statement */
 |  *table_value_constructor*                 /* VALUES statement */
 |  *explicit_table*                          /* TABLE statement  */
```

您应该知道`INTERSECT`在`UNION`或`EXCEPT`之前进行评估。这意味着，例如，`TABLE x UNION TABLE y INTERSECT TABLE z`总是被评估为`TABLE x UNION (TABLE y INTERSECT TABLE z)`。有关更多信息，请参见第 15.2.8 节，“INTERSECT 子句”。

此外，您应该记住，虽然`UNION`和`INTERSECT`集合运算符是可交换的（顺序不重要），但`EXCEPT`不是（操作数的顺序会影响结果）。换句话说，以下所有语句都是正确的：

+   `TABLE x UNION TABLE y`和`TABLE y UNION TABLE x`产生相同的结果，尽管行的排序可能不同。您可以使用`ORDER BY`强制它们相同；请参见联合中的 ORDER BY 和 LIMIT。

+   `TABLE x INTERSECT TABLE y`和`TABLE y INTERSECT TABLE x`返回相同的结果。

+   `TABLE x EXCEPT TABLE y`和`TABLE y EXCEPT TABLE x`不会产生相同的结果。请参见第 15.2.4 节，“EXCEPT 子句”，以获取示例。

更多信息和示例可以在接下来的章节中找到。

#### 结果集列名和数据类型

集合操作的结果的列名取自第一个查询块的列名。示例：

```sql
mysql> CREATE TABLE t1 (x INT, y INT);
Query OK, 0 rows affected (0.04 sec)

mysql> INSERT INTO t1 VALUES ROW(4,-2), ROW(5,9);
Query OK, 2 rows affected (0.00 sec)
Records: 2  Duplicates: 0  Warnings: 0

mysql> CREATE TABLE t2 (a INT, b INT);
Query OK, 0 rows affected (0.04 sec)

mysql> INSERT INTO t2 VALUES ROW(1,2), ROW(3,4);
Query OK, 2 rows affected (0.01 sec)
Records: 2  Duplicates: 0  Warnings: 0

mysql> TABLE t1 UNION TABLE t2;
+------+------+
| x    | y    |
+------+------+
|    4 |   -2 |
|    5 |    9 |
|    1 |    2 |
|    3 |    4 |
+------+------+
4 rows in set (0.00 sec)

mysql> TABLE t2 UNION TABLE t1;
+------+------+
| a    | b    |
+------+------+
|    1 |    2 |
|    3 |    4 |
|    4 |   -2 |
|    5 |    9 |
+------+------+
4 rows in set (0.00 sec)
```

对于`UNION`、`EXCEPT`和`INTERSECT`查询都是如此。

每个查询块中列的选定位置应具有相同的数据类型。例如，第一个语句选择的第一列应与其他语句选择的第一列具有相同的类型。如果相应结果列的数据类型不匹配，则结果中的列的类型和长度将考虑所有查询块检索的值。例如，结果集中的列长度不受限于第一个语句中的值的长度，如下所示：

```sql
mysql> SELECT REPEAT('a',1) UNION SELECT REPEAT('b',20);
+----------------------+
| REPEAT('a',1)        |
+----------------------+
| a                    |
| bbbbbbbbbbbbbbbbbbbb |
+----------------------+
```

#### 使用`TABLE`和`VALUES`语句进行集合操作

从 MySQL 8.0.19 开始，您还可以在可以使用等效的`SELECT`语句的地方使用`TABLE`语句或`VALUES`语句。假设表`t1`和`t2`如下所示创建和填充：

```sql
CREATE TABLE t1 (x INT, y INT);
INSERT INTO t1 VALUES ROW(4,-2),ROW(5,9);

CREATE TABLE t2 (a INT, b INT);
INSERT INTO t2 VALUES ROW(1,2),ROW(3,4);
```

在忽略以`VALUES`开头的查询输出中的列名的情况下，以下所有`UNION`查询都产生相同的结果：

```sql
SELECT * FROM t1 UNION SELECT * FROM t2;
TABLE t1 UNION SELECT * FROM t2;
VALUES ROW(4,-2), ROW(5,9) UNION SELECT * FROM t2;
SELECT * FROM t1 UNION TABLE t2;
TABLE t1 UNION TABLE t2;
VALUES ROW(4,-2), ROW(5,9) UNION TABLE t2;
SELECT * FROM t1 UNION VALUES ROW(4,-2),ROW(5,9);
TABLE t1 UNION VALUES ROW(4,-2),ROW(5,9);
VALUES ROW(4,-2), ROW(5,9) UNION VALUES ROW(4,-2),ROW(5,9);
```

要强制列名相同，请将左侧的查询块包装在`SELECT`语句中，并使用别名，如下所示：

```sql
mysql> SELECT * FROM (TABLE t2) AS t(x,y) UNION TABLE t1;
+------+------+
| x    | y    |
+------+------+
|    1 |    2 |
|    3 |    4 |
|    4 |   -2 |
|    5 |    9 |
+------+------+
4 rows in set (0.00 sec)
```

#### 使用`DISTINCT`和`ALL`进行集合操作

默认情况下，集合操作的结果中会删除重复行。可选的`DISTINCT`关键字具有相同的效果，但使其显式化。使用可选的`ALL`关键字，不会删除重复行，结果将包含联合中所有查询的所有匹配行。

你可以在同一查询中混合使用`ALL`和`DISTINCT`。混合类型的处理方式是，使用`DISTINCT`的集合操作会覆盖左侧使用`ALL`的任何操作。可以通过在`UNION`、`INTERSECT`或`EXCEPT`后显式地使用`DISTINCT`，或者在没有跟随`DISTINCT`或`ALL`关键字的情况下隐式地使用集合操作来生成`DISTINCT`集合。

在 MySQL 8.0.19 及更高版本中，当一个或多个`TABLE`语句、`VALUES`语句或两者用于生成集合时，集合操作的工作方式相同。

#### 使用`ORDER BY`和`LIMIT`的集合操作

要对作为联合、交集或其他集合操作的一部分使用的单个查询块应用`ORDER BY`或`LIMIT`子句，请将查询块括在括号中，并将子句放在括号内，就像这样：

```sql
(SELECT a FROM t1 WHERE a=10 AND b=1 ORDER BY a LIMIT 10)
UNION
(SELECT a FROM t2 WHERE a=11 AND b=2 ORDER BY a LIMIT 10);

(TABLE t1 ORDER BY x LIMIT 10) 
INTERSECT 
(TABLE t2 ORDER BY a LIMIT 10);
```

对于单个查询块或语句使用`ORDER BY`并不意味着结果中行的顺序，因为默认情况下，集合操作生成的行是无序的。因此，在这种情况下，`ORDER BY`通常与`LIMIT`结合使用，以确定要检索的所选行的子集，即使它并不一定影响这些行在最终结果中的顺序。如果在查询块中没有`LIMIT`出现`ORDER BY`，则会被优化掉，因为在任何情况下都没有影响。

要对整个集合操作的结果进行排序或限制，请将`ORDER BY`或`LIMIT`放在最后一个语句之后：

```sql
SELECT a FROM t1
EXCEPT
SELECT a FROM t2 WHERE a=11 AND b=2
ORDER BY a LIMIT 10;

TABLE t1
UNION 
TABLE t2
ORDER BY a LIMIT 10;
```

如果一个或多个单独的语句使用了`ORDER BY`、`LIMIT`或两者，并且另外，你希望对整个结果应用`ORDER BY`、`LIMIT`或两者，则必须将每个这样的单独语句括在括号中。

```sql
(SELECT a FROM t1 WHERE a=10 AND b=1)
EXCEPT
(SELECT a FROM t2 WHERE a=11 AND b=2)
ORDER BY a LIMIT 10;

(TABLE t1 ORDER BY a LIMIT 10) 
UNION 
TABLE t2 
ORDER BY a LIMIT 10;
```

没有`ORDER BY`或`LIMIT`子句的语句不需要括号；在刚刚显示的两个语句的第二个语句中用`(TABLE t2)`替换`TABLE t2`不会改变`UNION`的结果。

你也可以在集合操作中使用`ORDER BY`和`LIMIT`，就像在这个使用**mysql**客户端的示例中所示的那样：

```sql
mysql> VALUES ROW(4,-2), ROW(5,9), ROW(-1,3) 
 -> UNION 
 -> VALUES ROW(1,2), ROW(3,4), ROW(-1,3) 
 -> ORDER BY column_0 DESC LIMIT 3;
+----------+----------+
| column_0 | column_1 |
+----------+----------+
|        5 |        9 |
|        4 |       -2 |
|        3 |        4 |
+----------+----------+
3 rows in set (0.00 sec)
```

（请记住，`TABLE`语句和`VALUES`语句都不接受`WHERE`子句。）

这种类型的`ORDER BY`不能使用包含表名的列引用（即以*`tbl_name`*.*`col_name`*格式的名称）。相反，在第一个查询块中提供一个列别名，并在`ORDER BY`子句中引用该别名。 （你也可以在`ORDER BY`子句中使用列位置引用该列，但这种列位置的使用已被弃用，因此可能在未来的 MySQL 版本中被移除。）

如果要排序的列被别名，`ORDER BY`子句*必须*引用别名，而不是列名。以下两个语句中第一个是允许的，但第二个会因为`Unknown column 'a' in 'order clause'`错误而失败：

```sql
(SELECT a AS b FROM t) UNION (SELECT ...) ORDER BY b;
(SELECT a AS b FROM t) UNION (SELECT ...) ORDER BY a;
```

为了使`UNION`结果的行由每个查询块检索的行集合依次组成，需要在每个查询块中选择一个额外的列作为排序列，并在最后一个查询块后添加一个按照该列排序的`ORDER BY`子句：

```sql
(SELECT 1 AS sort_col, col1a, col1b, ... FROM t1)
UNION
(SELECT 2, col2a, col2b, ... FROM t2) ORDER BY sort_col;
```

为了在各个结果中保持排序顺序，向`ORDER BY`子句添加一个次要列：

```sql
(SELECT 1 AS sort_col, col1a, col1b, ... FROM t1)
UNION
(SELECT 2, col2a, col2b, ... FROM t2) ORDER BY sort_col, col1a;
```

使用额外的列还可以让你确定每行来自哪个查询块。额外的列还可以提供其他标识信息，比如指示表名的字符串。

#### 集合操作的限制

MySQL 中的集合操作受到一些限制，这些限制在接下来的几段中描述。

包括`SELECT`语句在内的集合操作有以下限制：

+   第一个`SELECT`中的`HIGH_PRIORITY`没有效果。任何后续`SELECT`中的`HIGH_PRIORITY`都会产生语法错误。

+   仅最后一个`SELECT`语句可以使用`INTO`子句。然而，整个`UNION`结果将被写入`INTO`输出目的地。

截至 MySQL 8.0.20，这两个包含`INTO`的`UNION`变体已被弃用；你应该期待它们在未来的 MySQL 版本中被移除的支持：

+   在查询表达式的尾随查询块中，在`FROM`之前使用`INTO`会产生警告。例如：

    ```sql
    ... UNION SELECT * INTO OUTFILE '*file_name*' FROM *table_name*;
    ```

+   在查询表达式的括号尾随块中，使用`INTO`（无论其相对于`FROM`的位置如何）都会产生警告。例如：

    ```sql
    ... UNION (SELECT * INTO OUTFILE '*file_name*' FROM *table_name*);
    ```

    这些变体已经被弃用，因为它们很令人困惑，好像它们收集的信息来自命名表而不是整个查询表达式（`UNION`）。

在`ORDER BY`子句中使用聚合函数的集合操作将被拒绝，并显示`ER_AGGREGATE_ORDER_FOR_UNION`。虽然错误名称可能暗示这仅适用于`UNION`查询，但前述情况也适用于`EXCEPT`和`INTERSECT`查询，如下所示：

```sql
mysql> TABLE t1 INTERSECT TABLE t2 ORDER BY MAX(x);
ERROR 3028 (HY000): Expression #1 of ORDER BY contains aggregate function and applies to a UNION, EXCEPT or INTERSECT
```

锁定子句（比如`FOR UPDATE`或`LOCK IN SHARE MODE`）适用于其后的查询块。这意味着，在与集合操作一起使用的`SELECT`语句中，只有在查询块和锁定子句被括号括起来时才能使用锁定子句。
