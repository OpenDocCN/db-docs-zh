> 原文：[`dev.mysql.com/doc/refman/8.0/en/working-with-null.html`](https://dev.mysql.com/doc/refman/8.0/en/working-with-null.html)

#### 5.3.4.6 处理 NULL 值

直到你习惯了，`NULL`值可能会让人感到惊讶。从概念上讲，`NULL`表示“缺失的未知值”，并且它与其他值的处理方式略有不同。

要测试`NULL`，请使用`IS NULL`和`IS NOT NULL`运算符，如下所示：

```sql
mysql> SELECT 1 IS NULL, 1 IS NOT NULL;
+-----------+---------------+
| 1 IS NULL | 1 IS NOT NULL |
+-----------+---------------+
|         0 |             1 |
+-----------+---------------+
```

您不能使用算术比较运算符如`=`、`<`或`<>`来测试`NULL`。为了自己演示这一点，请尝试以下查询：

```sql
mysql> SELECT 1 = NULL, 1 <> NULL, 1 < NULL, 1 > NULL;
+----------+-----------+----------+----------+
| 1 = NULL | 1 <> NULL | 1 < NULL | 1 > NULL |
+----------+-----------+----------+----------+
|     NULL |      NULL |     NULL |     NULL |
+----------+-----------+----------+----------+
```

因为与`NULL`进行任何算术比较的结果也是`NULL`，所以您无法从这些比较中获得任何有意义的结果。

在 MySQL 中，`0`或`NULL`表示假，而其他任何值表示真。布尔运算的默认真值为`1`。

这种对`NULL`的特殊处理是为什么在前一节中需要使用`death IS NOT NULL`而不是`death <> NULL`来确定哪些动物不再活着。

两个`NULL`值在`GROUP BY`中被视为相等。

在进行`ORDER BY`时，如果使用`ORDER BY ... ASC`，则`NULL`值会首先呈现，如果使用`ORDER BY ... DESC`，则会最后呈现。

处理`NULL`时的一个常见错误是假设无法将零或空字符串插入定义为`NOT NULL`的列，但事实并非如此。这些实际上是值，而`NULL`表示“没有值”。您可以通过使用`IS [NOT] NULL`来轻松测试，如下所示：

```sql
mysql> SELECT 0 IS NULL, 0 IS NOT NULL, '' IS NULL, '' IS NOT NULL;
+-----------+---------------+------------+----------------+
| 0 IS NULL | 0 IS NOT NULL | '' IS NULL | '' IS NOT NULL |
+-----------+---------------+------------+----------------+
|         0 |             1 |          0 |              1 |
+-----------+---------------+------------+----------------+
```

因此，完全可以将零或空字符串插入`NOT NULL`列，因为这些实际上是`NOT NULL`。参见 Section B.3.4.3, “Problems with NULL Values”。
