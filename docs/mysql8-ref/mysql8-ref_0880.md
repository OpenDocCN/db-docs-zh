# 14.24.4 舍入行为

> 原文：[`dev.mysql.com/doc/refman/8.0/en/precision-math-rounding.html`](https://dev.mysql.com/doc/refman/8.0/en/precision-math-rounding.html)

本节讨论了`ROUND()`函数的精确数学舍入以及对具有精确值类型（`DECIMAL`和整数）的列的插入。

`ROUND()`函数的舍入方式取决于其参数是精确还是近似值：

+   对于精确值数字，`ROUND()`使用“四舍五入到最接近的一半”规则：具有大于或等于.5 的分数部分的值将向上舍入到下一个整数（如果为正数）或向下舍入到下一个整数（如果为负数）。 （换句话说，它远离零四舍五入。）具有小于.5 的分数部分的值将向下舍入到下一个整数（如果为正数）或向上舍入到下一个整数（如果为负数）。 （换句话说，它朝向零四舍五入。）

+   对于近似值数字，结果取决于 C 库。在许多系统上，这意味着`ROUND()`使用“四舍五入到最近的偶数”规则：一个具有恰好处于两个整数之间的分数部分的值将四舍五入为最接近的偶数整数。

以下示例显示了精确值和近似值之间舍入方式的不同：

```sql
mysql> SELECT ROUND(2.5), ROUND(25E-1);
+------------+--------------+
| ROUND(2.5) | ROUND(25E-1) |
+------------+--------------+
| 3          |            2 |
+------------+--------------+
```

对于插入到`DECIMAL`或整数列中，目标是精确数据类型，因此无论要插入的值是精确还是近似值，舍入都使用“远离零的一半”：

```sql
mysql> CREATE TABLE t (d DECIMAL(10,0));
Query OK, 0 rows affected (0.00 sec)

mysql> INSERT INTO t VALUES(2.5),(2.5E0);
Query OK, 2 rows affected, 2 warnings (0.00 sec)
Records: 2  Duplicates: 0  Warnings: 2

mysql> SHOW WARNINGS;
+-------+------+----------------------------------------+
| Level | Code | Message                                |
+-------+------+----------------------------------------+
| Note  | 1265 | Data truncated for column 'd' at row 1 |
| Note  | 1265 | Data truncated for column 'd' at row 2 |
+-------+------+----------------------------------------+
2 rows in set (0.00 sec)

mysql> SELECT d FROM t;
+------+
| d    |
+------+
|    3 |
|    3 |
+------+
2 rows in set (0.00 sec)
```

`SHOW WARNINGS`语句显示由于小数部分的舍入而生成的注释。这种截断不是错误，即使在严格的 SQL 模式下也是如此（请参阅第 14.24.3 节，“表达式处理”）。
