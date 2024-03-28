# 12.8.4 表达式中的排序强制性

> 原文：[`dev.mysql.com/doc/refman/8.0/en/charset-collation-coercibility.html`](https://dev.mysql.com/doc/refman/8.0/en/charset-collation-coercibility.html)

在绝大多数语句中，MySQL 使用哪个排序来解决比较操作是显而易见的。例如，在以下情况下，应该清楚 MySQL 使用的排序是列`x`的排序：

```sql
SELECT x FROM T ORDER BY x;
SELECT x FROM T WHERE x = x;
SELECT DISTINCT x FROM T;
```

然而，对于多个操作数，可能存在歧义。例如，此语句执行列`x`和字符串常量`'Y'`之间的比较：

```sql
SELECT x FROM T WHERE x = 'Y';
```

如果`x`和`'Y'`具有相同的排序，则对于比较要使用的排序就没有歧义。但是，如果它们具有不同的排序，比较应该使用`x`的排序还是`'Y'`的排序？`x`和`'Y'`都有排序，那么哪个排序优先？

混合排序也可能发生在除比较之外的上下文中。例如，多参数连接操作`CONCAT(x,'Y')`将其参数组合成一个字符串。结果应该具有什么排序？

为了解决这类问题，MySQL 检查一个项目的排序是否可以强制转换为另一个项目的排序。MySQL 分配强制性值如下：

+   明确的`COLLATE`子句具有强制性为 0（根本不可强制转换）。

+   具有不同排序的两个字符串的连接具有强制性为 1。

+   列或存储过程参数或局部变量的排序具有强制性为 2。

+   “系统常量”（由函数返回的字符串，如`USER()`或`VERSION()`）具有强制性为 3。

+   文本常量的排序具有强制性为 4。

+   数值或时间值的排序具有强制性为 5。

+   `NULL`或从`NULL`派生的表达式具有强制性为 6。

MySQL 使用以下规则的强制性值来解决歧义：

+   使用具有最低强制性值的排序。

+   如果两侧具有相同的强制性，则：

    +   如果两侧都是 Unicode，或两侧都不是 Unicode，则会出现错误。

    +   如果一侧具有 Unicode 字符集，另一侧具有非 Unicode 字符集，则具有 Unicode 字符集的一侧获胜，并且自动进行字符集转换应用于非 Unicode 侧。例如，以下语句不会返回错误：

        ```sql
        SELECT CONCAT(utf8mb4_column, latin1_column) FROM t1;
        ```

        它返回一个具有字符集`utf8mb4`和与`utf8mb4_column`相同排序的结果。在连接之前，`latin1_column`的值会自动转换为`utf8mb4`。

    +   对于使用相同字符集但混合了`_bin`排序和`_ci`或`_cs`排序的操作，将使用`_bin`排序。这类似于将混合非二进制和二进制字符串的操作将操作数评估为二进制字符串，只是应用于排序而不是数据类型。

尽管自动转换不在 SQL 标准中，但标准确实指出每个字符集（在支持的字符方面）都是 Unicode 的“子集”。因为“适用于超集的原则也适用于子集”是一个众所周知的原则，我们认为 Unicode 的排序可以用于与非 Unicode 字符串的比较。更一般地说，MySQL 使用字符集库的概念，有时可以用于确定字符集之间的子集关系，并使操作中的操作数转换为否则会产生错误的操作。参见 Section 12.2.1, “Character Set Repertoire”。

以下表格说明了前述规则的一些应用。

| 比较 | 使用的排序 |
| --- | --- |
| `column1 = 'A'` | 使用`column1`的排序 |
| `column1 = 'A' COLLATE x` | 使用`'A' COLLATE x`的排序 |
| `column1 COLLATE x = 'A' COLLATE y` | 错误 |

要确定字符串表达式的可强制性，使用`COERCIBILITY()`函数（参见 Section 14.15, “Information Functions”）：

```sql
mysql> SELECT COERCIBILITY(_utf8mb4'A' COLLATE utf8mb4_bin);
 -> 0
mysql> SELECT COERCIBILITY(VERSION());
 -> 3
mysql> SELECT COERCIBILITY('A');
 -> 4
mysql> SELECT COERCIBILITY(1000);
 -> 5
mysql> SELECT COERCIBILITY(NULL);
 -> 6
```

对于将数值或时间值隐式转换为字符串，例如在表达式`CONCAT(1, 'abc')`中对参数`1`的情况，结果是一个由`character_set_connection`和`collation_connection`系统变量确定字符集和排序的字符（非二进制）字符串。参见 Section 14.3, “Type Conversion in Expression Evaluation”。
