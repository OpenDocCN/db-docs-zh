# 15.2.19 VALUES Statement

> 原文：[`dev.mysql.com/doc/refman/8.0/en/values.html`](https://dev.mysql.com/doc/refman/8.0/en/values.html)

`VALUES`是 MySQL 8.0.19 中引入的 DML 语句，它将一组一个或多个行作为表返回。换句话说，它是一个表值构造函数，也可以作为独立的 SQL 语句。

```sql
VALUES *row_constructor_list* [ORDER BY *column_designator*] [LIMIT *number*]

*row_constructor_list*:
    ROW(*value_list*)[, ROW(*value_list*)][, ...]

*value_list*:
    *value*[, *value*][, ...]

*column_designator*:
    column_*index*
```

`VALUES`语句由`VALUES`关键字后跟一个或多个行构造函数列表组成，用逗号分隔。行构造函数由`ROW()`行构造函数子句组成，其值列表由一个或多个标量值包含在括号中。一个值可以是任何 MySQL 数据类型的文字值或解析为标量值的表达式。

`ROW()`不能是空的（但提供的每个标量值可以是`NULL`）。在同一个`VALUES`语句中，每个`ROW()`在其值列表中必须具有相同数量的值。

`DEFAULT`关键字不受`VALUES`支持，并导致语法错误，除非它用于在`INSERT`语句中提供值。

`VALUES`的输出是一个表：

```sql
mysql> VALUES ROW(1,-2,3), ROW(5,7,9), ROW(4,6,8);
+----------+----------+----------+
| column_0 | column_1 | column_2 |
+----------+----------+----------+
|        1 |       -2 |        3 |
|        5 |        7 |        9 |
|        4 |        6 |        8 |
+----------+----------+----------+
3 rows in set (0.00 sec)
```

从`VALUES`输出的表的列具有隐式命名的列`column_0`，`column_1`，`column_2`等，始终从`0`开始。这个事实可以用来使用可选的`ORDER BY`子句按列对行进行排序，就像这个子句在`SELECT`语句中的工作方式一样，如下所示：

```sql
mysql> VALUES ROW(1,-2,3), ROW(5,7,9), ROW(4,6,8) ORDER BY column_1;
+----------+----------+----------+
| column_0 | column_1 | column_2 |
+----------+----------+----------+
|        1 |       -2 |        3 |
|        4 |        6 |        8 |
|        5 |        7 |        9 |
+----------+----------+----------+
3 rows in set (0.00 sec)
```

在 MySQL 8.0.21 及更高版本中，`VALUES`语句还支持`LIMIT`子句以限制输出中的行数。（以前，`LIMIT`是允许的，但不起作用。）

`VALUES`语句在列值的数据类型方面是宽松的；您可以在同一列中混合类型，如下所示：

```sql
mysql> VALUES ROW("q", 42, '2019-12-18'),
 ->     ROW(23, "abc", 98.6),
 ->     ROW(27.0002, "Mary Smith", '{"a": 10, "b": 25}');
+----------+------------+--------------------+
| column_0 | column_1   | column_2           |
+----------+------------+--------------------+
| q        | 42         | 2019-12-18         |
| 23       | abc        | 98.6               |
| 27.0002  | Mary Smith | {"a": 10, "b": 25} |
+----------+------------+--------------------+
3 rows in set (0.00 sec)
```

重要提示

具有一个或多个`ROW()`实例的`VALUES`充当表值构造函数；尽管它可以用于在`INSERT`或`REPLACE`语句中提供值，但不要将其与也用于此目的的`VALUES`关键字混淆。您也不要将其与指代`INSERT ... ON DUPLICATE KEY UPDATE`中列值的`VALUES()`函数混淆。

您还应该记住`ROW()`是一个行值构造函数（参见 Section 15.2.15.5, “Row Subqueries”)，而`VALUES ROW()`是一个表值构造函数；这两者不能互换使用。

`VALUES` 可以在许多情况下使用 `SELECT`，包括以下情况：

+   使用 `UNION`，如下所示：

    ```sql
    mysql> SELECT 1,2 UNION SELECT 10,15;
    +----+----+
    | 1  | 2  |
    +----+----+
    |  1 |  2 |
    | 10 | 15 |
    +----+----+
    2 rows in set (0.00 sec)

    mysql> VALUES ROW(1,2) UNION VALUES ROW(10,15);
    +----------+----------+
    | column_0 | column_1 |
    +----------+----------+
    |        1 |        2 |
    |       10 |       15 |
    +----------+----------+
    2 rows in set (0.00 sec)
    ```

    您可以将多行构建的表联合在一起，就像这样：

    ```sql
    mysql> VALUES ROW(1,2), ROW(3,4), ROW(5,6)
         >     UNION VALUES ROW(10,15),ROW(20,25);
    +----------+----------+
    | column_0 | column_1 |
    +----------+----------+
    |        1 |        2 |
    |        3 |        4 |
    |        5 |        6 |
    |       10 |       15 |
    |       20 |       25 |
    +----------+----------+
    5 rows in set (0.00 sec)
    ```

    在这种情况下，您也可以（通常更可取地）完全省略 `UNION` 并使用单个 **`VALUES`** 语句，就像这样：

    ```sql
    mysql> VALUES ROW(1,2), ROW(3,4), ROW(5,6), ROW(10,15), ROW(20,25);
    +----------+----------+
    | column_0 | column_1 |
    +----------+----------+
    |        1 |        2 |
    |        3 |        4 |
    |        5 |        6 |
    |       10 |       15 |
    |       20 |       25 |
    +----------+----------+
    ```

    `VALUES` 也可以与 `SELECT` 语句、`TABLE` 语句或两者一起在联合中使用。

    `UNION` 中的构建表必须包含相同数量的列，就像您使用 `SELECT` 一样。有关更多示例，请参见 Section 15.2.18, “UNION Clause”。

    在 MySQL 8.0.31 及更高版本中，您可以像使用 `UNION` 一样使用 `EXCEPT` 和 `INTERSECT` 与 `VALUES`，如下所示：

    ```sql
    mysql> VALUES ROW(1,2), ROW(3,4), ROW(5,6)
     ->   INTERSECT 
     -> VALUES ROW(10,15), ROW(20,25), ROW(3,4);
    +----------+----------+
    | column_0 | column_1 |
    +----------+----------+
    |        3 |        4 |
    +----------+----------+
    1 row in set (0.00 sec)

    mysql> VALUES ROW(1,2), ROW(3,4), ROW(5,6)
     ->   EXCEPT 
     -> VALUES ROW(10,15), ROW(20,25), ROW(3,4);
    +----------+----------+
    | column_0 | column_1 |
    +----------+----------+
    |        1 |        2 |
    |        5 |        6 |
    +----------+----------+
    2 rows in set (0.00 sec)
    ```

    有关更多信息，请参见 Section 15.2.4, “EXCEPT Clause” 和 Section 15.2.8, “INTERSECT Clause”。

+   在连接中。有关更多信息和示例，请参见 Section 15.2.13.2, “JOIN Clause”。

+   在 `INSERT` 或 `REPLACE` 语句中代替 `VALUES()`，在这种情况下，其语义与此处描述的略有不同。有关详细信息，请参见 Section 15.2.7, “INSERT Statement”。

+   在 `CREATE TABLE ... SELECT` 和 `CREATE VIEW ... SELECT` 中代替源表。有关更多信息和示例，请参见这些语句的描述。
