# 13.1.7 超出范围和溢出处理

> 原文：[`dev.mysql.com/doc/refman/8.0/en/out-of-range-and-overflow.html`](https://dev.mysql.com/doc/refman/8.0/en/out-of-range-and-overflow.html)

当 MySQL 将一个值存储在数值列中，该值超出了列数据类型的允许范围时，结果取决于当时生效的 SQL 模式：

+   如果启用了严格的 SQL 模式，MySQL 将拒绝超出范围的值并显示错误，插入操作将失败，符合 SQL 标准。

+   如果未启用任何限制模式，MySQL 会将值截断到列数据类型范围的适当端点，并存储结果值。

    当将超出范围的值分配给整数列时，MySQL 会存储代表列数据类型范围相应端点的值。

    当浮点或定点列被分配一个超出指定（或默认）精度和标度所暗示范围的值时，MySQL 会存储代表该范围端点的值。

假设表 `t1` 定义如下：

```sql
CREATE TABLE t1 (i1 TINYINT, i2 TINYINT UNSIGNED);
```

当启用严格的 SQL 模式时，会发生超出范围的错误：

```sql
mysql> SET sql_mode = 'TRADITIONAL';
mysql> INSERT INTO t1 (i1, i2) VALUES(256, 256);
ERROR 1264 (22003): Out of range value for column 'i1' at row 1
mysql> SELECT * FROM t1;
Empty set (0.00 sec)
```

当未启用严格的 SQL 模式时，会发生截断并伴有警告：

```sql
mysql> SET sql_mode = '';
mysql> INSERT INTO t1 (i1, i2) VALUES(256, 256);
mysql> SHOW WARNINGS;
+---------+------+---------------------------------------------+
| Level   | Code | Message                                     |
+---------+------+---------------------------------------------+
| Warning | 1264 | Out of range value for column 'i1' at row 1 |
| Warning | 1264 | Out of range value for column 'i2' at row 1 |
+---------+------+---------------------------------------------+
mysql> SELECT * FROM t1;
+------+------+
| i1   | i2   |
+------+------+
|  127 |  255 |
+------+------+
```

当未启用严格的 SQL 模式时，由于截断而发生的列赋值转换会作为警告报告给`ALTER TABLE`、`LOAD DATA`、`UPDATE` 和多行`INSERT` 语句。在严格模式下，这些语句将失败，并且根据表是否为事务表和其他因素，某些或全部值将不会被插入或更改。有关详细信息，请参见第 7.1.11 节，“服务器 SQL 模式”。

在数值表达式评估过程中发生溢出会导致错误。例如，最大的有符号`BIGINT` - INTEGER, INT, SMALLINT, TINYINT, MEDIUMINT, BIGINT") 值为 9223372036854775807，因此以下表达式会产生错误：

```sql
mysql> SELECT 9223372036854775807 + 1;
ERROR 1690 (22003): BIGINT value is out of range in '(9223372036854775807 + 1)'
```

要使此操作在此情况下成功，将值转换为无符号；

```sql
mysql> SELECT CAST(9223372036854775807 AS UNSIGNED) + 1;
+-------------------------------------------+
| CAST(9223372036854775807 AS UNSIGNED) + 1 |
+-------------------------------------------+
|                       9223372036854775808 |
+-------------------------------------------+
```

溢出是否发生取决于操作数的范围，因此处理前述表达式的另一种方法是使用精确值算术，因为`DECIMAL` - DECIMAL, NUMERIC") 值的范围比整数大：

```sql
mysql> SELECT 9223372036854775807.0 + 1;
+---------------------------+
| 9223372036854775807.0 + 1 |
+---------------------------+
|     9223372036854775808.0 |
+---------------------------+
```

两个整数值相减，其中一个为`UNSIGNED`类型，默认会产生无符号结果。如果结果本应为负数，则会产生错误：

```sql
mysql> SET sql_mode = '';
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT CAST(0 AS UNSIGNED) - 1;
ERROR 1690 (22003): BIGINT UNSIGNED value is out of range in '(cast(0 as unsigned) - 1)'
```

如果启用了`NO_UNSIGNED_SUBTRACTION` SQL 模式，则结果为负数：

```sql
mysql> SET sql_mode = 'NO_UNSIGNED_SUBTRACTION';
mysql> SELECT CAST(0 AS UNSIGNED) - 1;
+-------------------------+
| CAST(0 AS UNSIGNED) - 1 |
+-------------------------+
|                      -1 |
+-------------------------+
```

如果这样的操作结果用于更新一个`UNSIGNED`整数列，结果将被截断为列类型的最大值，或者如果启用了`NO_UNSIGNED_SUBTRACTION`，则被截断为 0。如果启用了严格的 SQL 模式，将会发生错误并且列保持不变。
