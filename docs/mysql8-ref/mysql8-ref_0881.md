# 14.24.5 精度数学示例

> 原文：[`dev.mysql.com/doc/refman/8.0/en/precision-math-examples.html`](https://dev.mysql.com/doc/refman/8.0/en/precision-math-examples.html)

本节提供了一些示例，展示了 MySQL 中精度数学查询结果的示例。这些示例演示了第 14.24.3 节，“表达式处理”和第 14.24.4 节，“四舍五入行为”中描述的原则。

**示例 1**。在可能的情况下，数字使用其给定的精确值：

```sql
mysql> SELECT (.1 + .2) = .3;
+----------------+
| (.1 + .2) = .3 |
+----------------+
|              1 |
+----------------+
```

对于浮点值，结果是不精确的：

```sql
mysql> SELECT (.1E0 + .2E0) = .3E0;
+----------------------+
| (.1E0 + .2E0) = .3E0 |
+----------------------+
|                    0 |
+----------------------+
```

另一种看到精确值和近似值处理差异的方法是多次向总和中添加一个小数。考虑以下存储过程，它将`.0001`添加到一个变量 1,000 次。

```sql
CREATE PROCEDURE p ()
BEGIN
  DECLARE i INT DEFAULT 0;
  DECLARE d DECIMAL(10,4) DEFAULT 0;
  DECLARE f FLOAT DEFAULT 0;
  WHILE i < 10000 DO
    SET d = d + .0001;
    SET f = f + .0001E0;
    SET i = i + 1;
  END WHILE;
  SELECT d, f;
END;
```

对于`d`和`f`的总和在逻辑上应该为 1，但这仅适用于十进制计算。浮点计算引入了小误差：

```sql
+--------+------------------+
| d      | f                |
+--------+------------------+
| 1.0000 | 0.99999999999991 |
+--------+------------------+
```

**示例 2**。乘法使用标准 SQL 所需的比例执行。也就是说，对于具有比例*S1*和*S2*的两个数字*X1*和*X2*，结果的比例为*S1* + *S2*：

```sql
mysql> SELECT .01 * .01;
+-----------+
| .01 * .01 |
+-----------+
| 0.0001    |
+-----------+
```

**示例 3**。精确值数字的四舍五入行为是明确定义的：

四舍五入行为（例如，使用`ROUND()`函数）独立于底层 C 库的实现，这意味着结果在不同平台上是一致的。

+   精确值列（`DECIMAL` - DECIMAL, NUMERIC") 和整数）以及精确值数字的四舍五入使用“远离零的方向”规则。具有小数部分为.5 或更大的值将远离零四舍五入到最近的整数，如下所示：

    ```sql
    mysql> SELECT ROUND(2.5), ROUND(-2.5);
    +------------+-------------+
    | ROUND(2.5) | ROUND(-2.5) |
    +------------+-------------+
    | 3          | -3          |
    +------------+-------------+
    ```

+   浮点值的四舍五入使用 C 库，许多系统上使用“四舍五入到最近偶数”的规则。具有恰好处于两个整数之间的小数部分的值将四舍五入为最近的偶数：

    ```sql
    mysql> SELECT ROUND(2.5E0), ROUND(-2.5E0);
    +--------------+---------------+
    | ROUND(2.5E0) | ROUND(-2.5E0) |
    +--------------+---------------+
    |            2 |            -2 |
    +--------------+---------------+
    ```

**示例 4**。在严格模式下，插入超出列范围的值会导致错误，而不是截断为合法值。

当 MySQL 未在严格模式下运行时，会发生截断为合法值的情况：

```sql
mysql> SET sql_mode='';
Query OK, 0 rows affected (0.00 sec)

mysql> CREATE TABLE t (i TINYINT);
Query OK, 0 rows affected (0.01 sec)

mysql> INSERT INTO t SET i = 128;
Query OK, 1 row affected, 1 warning (0.00 sec)

mysql> SELECT i FROM t;
+------+
| i    |
+------+
|  127 |
+------+
1 row in set (0.00 sec)
```

但是，如果启用了严格模式，则会发生错误：

```sql
mysql> SET sql_mode='STRICT_ALL_TABLES';
Query OK, 0 rows affected (0.00 sec)

mysql> CREATE TABLE t (i TINYINT);
Query OK, 0 rows affected (0.00 sec)

mysql> INSERT INTO t SET i = 128;
ERROR 1264 (22003): Out of range value adjusted for column 'i' at row 1 
mysql> SELECT i FROM t;
Empty set (0.00 sec)
```

**示例 5**：在严格模式下，并且设置了`ERROR_FOR_DIVISION_BY_ZERO`，除以零会导致错误，而不是结果为`NULL`。

在非严格模式下，除以零的结果为`NULL`：

```sql
mysql> SET sql_mode='';
Query OK, 0 rows affected (0.01 sec)

mysql> CREATE TABLE t (i TINYINT);
Query OK, 0 rows affected (0.00 sec)

mysql> INSERT INTO t SET i = 1 / 0;
Query OK, 1 row affected (0.00 sec)

mysql> SELECT i FROM t;
+------+
| i    |
+------+
| NULL |
+------+
1 row in set (0.03 sec)
```

但是，如果启用了正确的 SQL 模式，则除以零会导致错误：

```sql
mysql> SET sql_mode='STRICT_ALL_TABLES,ERROR_FOR_DIVISION_BY_ZERO';
Query OK, 0 rows affected (0.00 sec)

mysql> CREATE TABLE t (i TINYINT);
Query OK, 0 rows affected (0.00 sec)

mysql> INSERT INTO t SET i = 1 / 0;
ERROR 1365 (22012): Division by 0 
mysql> SELECT i FROM t;
Empty set (0.01 sec)
```

**示例 6**。精确值文字被评估为精确值。

近似值文字使用浮点数进行评估，但精确值文字被处理为`DECIMAL` - DECIMAL, NUMERIC")：

```sql
mysql> CREATE TABLE t SELECT 2.5 AS a, 25E-1 AS b;
Query OK, 1 row affected (0.01 sec)
Records: 1  Duplicates: 0  Warnings: 0

mysql> DESCRIBE t;
+-------+-----------------------+------+-----+---------+-------+
| Field | Type                  | Null | Key | Default | Extra |
+-------+-----------------------+------+-----+---------+-------+
| a     | decimal(2,1) unsigned | NO   |     | 0.0     |       |
| b     | double                | NO   |     | 0       |       |
+-------+-----------------------+------+-----+---------+-------+
2 rows in set (0.01 sec)
```

**示例 7**。如果聚合函数的参数是精确数值类型，则结果也是精确数值类型，其精度至少与参数相同。

考虑以下语句：

```sql
mysql> CREATE TABLE t (i INT, d DECIMAL, f FLOAT);
mysql> INSERT INTO t VALUES(1,1,1);
mysql> CREATE TABLE y SELECT AVG(i), AVG(d), AVG(f) FROM t;
```

结果仅对浮点参数为双精度。对于精确类型参数，结果也是精确类型：

```sql
mysql> DESCRIBE y;
+--------+---------------+------+-----+---------+-------+
| Field  | Type          | Null | Key | Default | Extra |
+--------+---------------+------+-----+---------+-------+
| AVG(i) | decimal(14,4) | YES  |     | NULL    |       |
| AVG(d) | decimal(14,4) | YES  |     | NULL    |       |
| AVG(f) | double        | YES  |     | NULL    |       |
+--------+---------------+------+-----+---------+-------+
```

结果仅对浮点参数为双精度。对于精确类型参数，结果也是精确类型。
