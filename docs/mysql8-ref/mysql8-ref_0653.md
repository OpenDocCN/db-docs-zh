# 11.1.4 十六进制文字

> 译文：[`dev.mysql.com/doc/refman/8.0/en/hexadecimal-literals.html`](https://dev.mysql.com/doc/refman/8.0/en/hexadecimal-literals.html)

十六进制文字值使用 `X'*`val`*'` 或 `0x*`val`*` 表示法编写，其中 *`val`* 包含十六进制数字（`0..9`，`A..F`）。数字的大小写和任何前导 `X` 的大小写不重要。前导 `0x` 区分大小写，不能写为 `0X`。

合法的十六进制文字：

```sql
X'01AF'
X'01af'
x'01AF'
x'01af'
0x01AF
0x01af
```

非法的十六进制文字：

```sql
X'0G'   (G is not a hexadecimal digit)
0X01AF  (0X must be written as 0x)
```

使用 `X'*`val`*'` 表示法编写的值必须包含偶数个数字，否则会出现语法错误。要纠正问题，请在值前面填充一个前导零：

```sql
mysql> SET @s = X'FFF';
ERROR 1064 (42000): You have an error in your SQL syntax;
check the manual that corresponds to your MySQL server
version for the right syntax to use near 'X'FFF'' 
mysql> SET @s = X'0FFF';
Query OK, 0 rows affected (0.00 sec)
```

使用包含奇数个数字的 `0x*`val`*` 表示法编写的值被视为具有额外的前导 `0`。例如，`0xaaa` 被解释为 `0x0aaa`。

默认情况下，十六进制文字是一个二进制字符串，其中每对十六进制数字表示一个字符：

```sql
mysql> SELECT X'4D7953514C', CHARSET(X'4D7953514C');
+---------------+------------------------+
| X'4D7953514C' | CHARSET(X'4D7953514C') |
+---------------+------------------------+
| MySQL         | binary                 |
+---------------+------------------------+
mysql> SELECT 0x5461626c65, CHARSET(0x5461626c65);
+--------------+-----------------------+
| 0x5461626c65 | CHARSET(0x5461626c65) |
+--------------+-----------------------+
| Table        | binary                |
+--------------+-----------------------+
```

十六进制文字可能具有可选的字符集引导符和 `COLLATE` 子句，以指定其为使用特定字符集和排序规则的字符串：

```sql
[_*charset_name*] X'*val*' [COLLATE *collation_name*]
```

示例：

```sql
SELECT _latin1 X'4D7953514C';
SELECT _utf8mb4 0x4D7953514C COLLATE utf8mb4_danish_ci;
```

示例使用 `X'*`val`*'` 表示法，但 `0x*`val`*` 表示法也允许引导符。有关引导符的信息，请参阅 Section 12.3.8, “Character Set Introducers”。

在数值上下文中，MySQL 将十六进制文字视为 `BIGINT UNSIGNED`（64 位无符号整数）。为确保十六进制文字的数值处理，请在数值上下文中使用它。实现这一目的的方法包括添加 0 或使用 `CAST(... AS UNSIGNED)`。例如，将十六进制文字分配给用户定义的变量时，默认情况下是二进制字符串。要将值分配为数字，请在数值上下文中使用它：

```sql
mysql> SET @v1 = X'41';
mysql> SET @v2 = X'41'+0;
mysql> SET @v3 = CAST(X'41' AS UNSIGNED);
mysql> SELECT @v1, @v2, @v3;
+------+------+------+
| @v1  | @v2  | @v3  |
+------+------+------+
| A    |   65 |   65 |
+------+------+------+
```

空的十六进制值（`X''`）评估为零长度的二进制字符串。转换为数字后，它产生 0：

```sql
mysql> SELECT CHARSET(X''), LENGTH(X'');
+--------------+-------------+
| CHARSET(X'') | LENGTH(X'') |
+--------------+-------------+
| binary       |           0 |
+--------------+-------------+
mysql> SELECT X''+0;
+-------+
| X''+0 |
+-------+
|     0 |
+-------+
```

`X'*`val`*'` 表示法基于标准 SQL。`0x` 表示法基于 ODBC，其中十六进制字符串通常用于为 `BLOB` 列提供值。

要将字符串或数字转换为十六进制格式的字符串，请使用 `HEX()` 函数：

```sql
mysql> SELECT HEX('cat');
+------------+
| HEX('cat') |
+------------+
| 636174     |
+------------+
mysql> SELECT X'636174';
+-----------+
| X'636174' |
+-----------+
| cat       |
+-----------+
```

对于十六进制文字，位操作被视为数值上下文，但在 MySQL 8.0 及更高版本中，位操作允许数值或二进制字符串参数。要明确指定十六进制文字的二进制字符串上下文，请至少在一个参数中使用 `_binary` 引导符：

```sql
mysql> SET @v1 = X'000D' | X'0BC0';
mysql> SET @v2 = _binary X'000D' | X'0BC0';
mysql> SELECT HEX(@v1), HEX(@v2);
+----------+----------+
| HEX(@v1) | HEX(@v2) |
+----------+----------+
| BCD      | 0BCD     |
+----------+----------+
```

显示的结果对于位操作都是相似的，但没有 `_binary` 的结果是一个 `BIGINT` 值，而带有 `_binary` 的结果是一个二进制字符串。由于结果类型的差异，显示的值也不同：数值结果不显示高阶 0 数字。
