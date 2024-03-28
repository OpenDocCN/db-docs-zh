# 15.2.16 TABLE 语句

> 原文：[`dev.mysql.com/doc/refman/8.0/en/table.html`](https://dev.mysql.com/doc/refman/8.0/en/table.html)

`TABLE`是 MySQL 8.0.19 中引入的 DML 语句，返回命名表的行和列。

```sql
TABLE *table_name* [ORDER BY *column_name*] [LIMIT *number* [OFFSET *number*]]
```

`TABLE`语句在某些方面类似于`SELECT`。给定名为`t`的表存在，以下两个语句产生相同的输出：

```sql
TABLE t;

SELECT * FROM t;
```

您可以使用`ORDER BY`和`LIMIT`子句对`TABLE`生成的行数进行排序和限制。这些与在`SELECT`中使用相同的子句完全相同（包括与`LIMIT`一起使用的可选`OFFSET`子句），如下所示：

```sql
mysql> TABLE t;
+----+----+
| a  | b  |
+----+----+
|  1 |  2 |
|  6 |  7 |
|  9 |  5 |
| 10 | -4 |
| 11 | -1 |
| 13 |  3 |
| 14 |  6 |
+----+----+
7 rows in set (0.00 sec)

mysql> TABLE t ORDER BY b;
+----+----+
| a  | b  |
+----+----+
| 10 | -4 |
| 11 | -1 |
|  1 |  2 |
| 13 |  3 |
|  9 |  5 |
| 14 |  6 |
|  6 |  7 |
+----+----+
7 rows in set (0.00 sec)

mysql> TABLE t LIMIT 3;
+---+---+
| a | b |
+---+---+
| 1 | 2 |
| 6 | 7 |
| 9 | 5 |
+---+---+
3 rows in set (0.00 sec)

mysql> TABLE t ORDER BY b LIMIT 3;
+----+----+
| a  | b  |
+----+----+
| 10 | -4 |
| 11 | -1 |
|  1 |  2 |
+----+----+
3 rows in set (0.00 sec)

mysql> TABLE t ORDER BY b LIMIT 3 OFFSET 2;
+----+----+
| a  | b  |
+----+----+
|  1 |  2 |
| 13 |  3 |
|  9 |  5 |
+----+----+
3 rows in set (0.00 sec)
```

`TABLE`在两个关键方面与`SELECT`不同：

+   `TABLE`总是显示表的所有列。

    *例外*：`TABLE`的输出*不*包括不可见列。查看第 15.1.20.10 节，“不可见列”。

+   `TABLE`不允许对行进行任意过滤；也就是说，`TABLE`不支持任何`WHERE`子句。

为了限制返回的表列，过滤超出`ORDER BY`和`LIMIT`所能实现的行，或两者都使用，使用`SELECT`。

`TABLE`可以与临时表一起使用。

`TABLE`也可以用于取代`SELECT`在许多其他结构中，包括以下列出的结构：

+   使用诸如`UNION`之类的集合运算符，如下所示：

    ```sql
    mysql> TABLE t1;
    +---+----+
    | a | b  |
    +---+----+
    | 2 | 10 |
    | 5 |  3 |
    | 7 |  8 |
    +---+----+
    3 rows in set (0.00 sec)

    mysql> TABLE t2;
    +---+---+
    | a | b |
    +---+---+
    | 1 | 2 |
    | 3 | 4 |
    | 6 | 7 |
    +---+---+
    3 rows in set (0.00 sec)

    mysql> TABLE t1 UNION TABLE t2;
    +---+----+
    | a | b  |
    +---+----+
    | 2 | 10 |
    | 5 |  3 |
    | 7 |  8 |
    | 1 |  2 |
    | 3 |  4 |
    | 6 |  7 |
    +---+----+
    6 rows in set (0.00 sec)
    ```

    刚刚显示的`UNION`等效于以下语句：

    ```sql
    mysql> SELECT * FROM t1 UNION SELECT * FROM t2;
    +---+----+
    | a | b  |
    +---+----+
    | 2 | 10 |
    | 5 |  3 |
    | 7 |  8 |
    | 1 |  2 |
    | 3 |  4 |
    | 6 |  7 |
    +---+----+
    6 rows in set (0.00 sec)
    ```

    `TABLE`还可以与`SELECT`语句、`VALUES`语句或两者一起在集合操作中使用。查看第 15.2.18 节，“UNION 子句”、第 15.2.4 节，“EXCEPT 子句”和第 15.2.8 节，“INTERSECT 子句”，获取更多信息和示例。另请参阅第 15.2.14 节，“使用 UNION、INTERSECT 和 EXCEPT 的集合操作”。

+   使用`INTO`填充用户变量，并使用`INTO OUTFILE`或`INTO DUMPFILE`将表数据写入文件。查看第 15.2.13.1 节，“SELECT ... INTO 语句”，获取更具体的信息和示例。

+   在许多情况下，您可以使用子查询。给定具有名为`a`的列的任何表`t1`，以及具有单列的第二个表`t2`，以下语句是可能的：

    ```sql
    SELECT * FROM t1 WHERE a IN (TABLE t2);
    ```

    假设表`t1`的单列命名为`x`，前述与以下各语句等效（在任一情况下产生完全相同的结果）：

    ```sql
    SELECT * FROM t1 WHERE a IN (SELECT x FROM t2);

    SELECT * FROM t1 WHERE a IN (SELECT * FROM t2);
    ```

    查看第 15.2.15 节，“子查询”，获取更多信息。

+   在`INSERT`和`REPLACE`语句中，您原本会使用`SELECT *`。有关更多信息和示例，请参阅第 15.2.7.1 节，“INSERT ... SELECT 语句”。

+   `TABLE`在许多情况下也可以代替`SELECT`在`CREATE TABLE ... SELECT`或`CREATE VIEW ... SELECT`中使用。有关这些语句的更多信息和示例，请参阅其描述。
