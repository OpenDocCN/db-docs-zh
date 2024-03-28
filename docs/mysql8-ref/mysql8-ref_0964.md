> 原文：[`dev.mysql.com/doc/refman/8.0/en/derived-tables.html`](https://dev.mysql.com/doc/refman/8.0/en/derived-tables.html)

#### 15.2.15.8 派生表

本节讨论了派生表的一般特性。有关由`LATERAL`关键字引导的横向派生表的信息，请参见 Section 15.2.15.9，“横向派生表”。

派生表是在查询`FROM`子句的范围内生成表的表达式。例如，在`SELECT`语句的`FROM`子句中的子查询是一个派生表：

```sql
SELECT ... FROM (*subquery*) [AS] *tbl_name* ...
```

`JSON_TABLE()`函数生成一个表，并提供了创建派生表的另一种方法：

```sql
SELECT * FROM JSON_TABLE(*arg_list*) [AS] *tbl_name* ...
```

`[AS] *`tbl_name`*`子句是必需的，因为`FROM`子句中的每个表都必须有一个名称。派生表中的任何列必须具有唯一的名称。或者，*`tbl_name`*`后面可以跟着一个括号括起来的列名列表：

```sql
SELECT ... FROM (*subquery*) [AS] *tbl_name* (*col_list*) ...
```

列名的数量必须与表列的数量相同。

为了说明问题，假设您有这个表：

```sql
CREATE TABLE t1 (s1 INT, s2 CHAR(5), s3 FLOAT);
```

下面是如何在`FROM`子句中使用子查询，使用示例表：

```sql
INSERT INTO t1 VALUES (1,'1',1.0);
INSERT INTO t1 VALUES (2,'2',2.0);
SELECT sb1,sb2,sb3
  FROM (SELECT s1 AS sb1, s2 AS sb2, s3*2 AS sb3 FROM t1) AS sb
  WHERE sb1 > 1;
```

结果：

```sql
+------+------+------+
| sb1  | sb2  | sb3  |
+------+------+------+
|    2 | 2    |    4 |
+------+------+------+
```

这里是另一个例子：假设您想知道一个分组表的一组求和的平均值。这不起作用：

```sql
SELECT AVG(SUM(column1)) FROM t1 GROUP BY column1;
```

然而，这个查询提供了所需的信息：

```sql
SELECT AVG(sum_column1)
  FROM (SELECT SUM(column1) AS sum_column1
        FROM t1 GROUP BY column1) AS t1;
```

注意，在子查询中使用的列名（`sum_column1`）在外部查询中被识别。

派生表的列名来自其选择列表：

```sql
mysql> SELECT * FROM (SELECT 1, 2, 3, 4) AS dt;
+---+---+---+---+
| 1 | 2 | 3 | 4 |
+---+---+---+---+
| 1 | 2 | 3 | 4 |
+---+---+---+---+
```

要明确提供列名，请在派生表名称后面跟着一个括号括起来的列名列表：

```sql
mysql> SELECT * FROM (SELECT 1, 2, 3, 4) AS dt (a, b, c, d);
+---+---+---+---+
| a | b | c | d |
+---+---+---+---+
| 1 | 2 | 3 | 4 |
+---+---+---+---+
```

派生表可以返回标量、列、行或表。

派生表受到以下限制：

+   派生表不能包含对同一`SELECT`的其他表的引用（使用`LATERAL`派生表进行处理；请参见 Section 15.2.15.9，“横向派生表”）。

+   在 MySQL 8.0.14 之前，派生表不能包含外部引用。这是 MySQL 在 MySQL 8.0.14 中解除的限制，而不是 SQL 标准的限制。例如，以下查询中的派生表`dt`包含对外部查询中表`t1`的引用`t1.b`：

    ```sql
    SELECT * FROM t1
    WHERE t1.d > (SELECT AVG(dt.a)
                    FROM (SELECT SUM(t2.a) AS a
                          FROM t2
                          WHERE t2.b = t1.b GROUP BY t2.c) dt
                  WHERE dt.a > 10);
    ```

    该查询在 MySQL 8.0.14 及更高版本中有效。在 8.0.14 之前，它会产生一个错误：`Unknown column 't1.b' in 'where clause'`

优化器以一种不需要将派生表实例化的方式确定有关派生表的信息，因此`EXPLAIN`不需要将其实例化。请参见 Section 10.2.2.4，“使用合并或实例化优化派生表、视图引用和通用表达式”。

在某些情况下，使用`EXPLAIN SELECT`可能会修改表数据。如果外部查询访问任何表，并且内部查询调用修改表中一个或多个行的存储函数，则可能会发生这种情况。假设数据库`d1`中有两个表`t1`和`t2`，以及一个修改`t2`的存储函数`f1`，创建如下所示：

```sql
CREATE DATABASE d1;
USE d1;
CREATE TABLE t1 (c1 INT);
CREATE TABLE t2 (c1 INT);
CREATE FUNCTION f1(p1 INT) RETURNS INT
  BEGIN
    INSERT INTO t2 VALUES (p1);
    RETURN p1;
  END;
```

直接在`EXPLAIN SELECT`中引用函数对`t2`没有影响，如下所示：

```sql
mysql> SELECT * FROM t2;
Empty set (0.02 sec)

mysql> EXPLAIN SELECT f1(5)\G
*************************** 1\. row ***************************
           id: 1
  select_type: SIMPLE
        table: NULL
   partitions: NULL
         type: NULL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: NULL
     filtered: NULL
        Extra: No tables used 1 row in set (0.01 sec)

mysql> SELECT * FROM t2;
Empty set (0.01 sec)
```

这是因为`SELECT`语句没有引用任何表，可以在输出的`table`和`Extra`列中看到。这也适用于以下嵌套的`SELECT`：

```sql
mysql> EXPLAIN SELECT NOW() AS a1, (SELECT f1(5)) AS a2\G
*************************** 1\. row ***************************
           id: 1
  select_type: PRIMARY
        table: NULL
         type: NULL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: NULL
     filtered: NULL
        Extra: No tables used 1 row in set, 1 warning (0.00 sec)

mysql> SHOW WARNINGS;
+-------+------+------------------------------------------+
| Level | Code | Message                                  |
+-------+------+------------------------------------------+
| Note  | 1249 | Select 2 was reduced during optimization |
+-------+------+------------------------------------------+
1 row in set (0.00 sec)

mysql> SELECT * FROM t2;
Empty set (0.00 sec)
```

但是，如果外部`SELECT`引用任何表，优化器也会执行子查询中的语句，结果导致`t2`被修改：

```sql
mysql> EXPLAIN SELECT * FROM t1 AS a1, (SELECT f1(5)) AS a2\G
*************************** 1\. row ***************************
           id: 1
  select_type: PRIMARY
        table: <derived2>
   partitions: NULL
         type: system
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 1
     filtered: 100.00
        Extra: NULL
*************************** 2\. row ***************************
           id: 1
  select_type: PRIMARY
        table: a1
   partitions: NULL
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 1
     filtered: 100.00
        Extra: NULL
*************************** 3\. row ***************************
           id: 2
  select_type: DERIVED
        table: NULL
   partitions: NULL
         type: NULL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: NULL
     filtered: NULL
        Extra: No tables used 3 rows in set (0.00 sec)

mysql> SELECT * FROM t2;
+------+
| c1   |
+------+
|    5 |
+------+
1 row in set (0.00 sec)
```

衍生表优化也可以与许多相关的（标量）子查询一起使用（MySQL 8.0.24 及更高版本）。有关更多信息和示例，请参见第 15.2.15.7 节，“相关子查询”。
