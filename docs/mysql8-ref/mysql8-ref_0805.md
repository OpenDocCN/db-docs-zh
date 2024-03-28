# 14.8.3 函数结果的字符集和排序规则

> 原文：[`dev.mysql.com/doc/refman/8.0/en/string-functions-charset.html`](https://dev.mysql.com/doc/refman/8.0/en/string-functions-charset.html)

MySQL 有许多返回字符串的运算符和函数。本节回答了一个问题：这样的字符串的字符集和排序规则是什么？

对于接受字符串输入并返回字符串结果的简单函数，输出的字符集和排序规则与主要输入值相同。例如，`UPPER(*`X`*)` 返回一个与 *`X`* 相同字符集和排序规则的字符串。对于 `INSTR()`、`LCASE()`、`LOWER()`、`LTRIM()`、`MID()`、`REPEAT()`、`REPLACE()`、`REVERSE()`、`RIGHT()`、`RPAD()`、`RTRIM()`、`SOUNDEX()`、`SUBSTRING()`、`TRIM()`、`UCASE()` 和 `UPPER()` 也适用相同规则。

注意

与所有其他函数不同，`REPLACE()` 函数始终忽略字符串输入的排序规则并执行区分大小写的比较。

如果字符串输入或函数结果是二进制字符串，则该字符串具有 `binary` 字符集和排序规则。可以通过使用 `CHARSET()` 和 `COLLATION()` 函数来检查，两者对于二进制字符串参数都返回 `binary`：

```sql
mysql> SELECT CHARSET(BINARY 'a'), COLLATION(BINARY 'a');
+---------------------+-----------------------+
| CHARSET(BINARY 'a') | COLLATION(BINARY 'a') |
+---------------------+-----------------------+
| binary              | binary                |
+---------------------+-----------------------+
```

对于将多个字符串输入组合并返回单个字符串输出的操作，标准 SQL 的“聚合规则”适用于确定结果的排序规则：

+   如果明确出现 `COLLATE *`Y`*`，则使用 *`Y`*。

+   如果明确出现 `COLLATE *`Y`*` 和 `COLLATE *`Z`*`，则引发错误。

+   否则，如果所有排序规则都是 *`Y`*，则使用 *`Y`*。

+   否则，结果没有排序规则。

例如，使用 `CASE ... WHEN a THEN b WHEN b THEN c COLLATE *`X`* END`，结果的排序规则是 *`X`*。对于 `UNION`、`||`、`CONCAT()`、`ELT()`、`GREATEST()`、`IF()` 和 `LEAST()` 也适用相同规则。

对于转换为字符数据的操作，操作结果字符串的字符集和排序由确定默认连接字符集和排序的`character_set_connection`和`collation_connection`系统变量定义（参见第 12.4 节，“连接字符集和排序”）。这仅适用于`BIN_TO_UUID()`、`CAST()`、`CONV()`、`FORMAT()`、`HEX()`和`SPACE()`。

对于虚拟生成列的表达式，前述原则有一个例外。在这种表达式中，无论连接字符集如何，都使用表字符集来处理`BIN_TO_UUID()`、`CONV()`或`HEX()`的结果。

如果对字符串函数返回的结果的字符集或排序有任何疑问，请使用`CHARSET()`或`COLLATION()`函数来查找：

```sql
mysql> SELECT USER(), CHARSET(USER()), COLLATION(USER());
+----------------+-----------------+--------------------+
| USER()         | CHARSET(USER()) | COLLATION(USER())  |
+----------------+-----------------+--------------------+
| test@localhost | utf8mb3         | utf8mb3_general_ci |
+----------------+-----------------+--------------------+
mysql> SELECT CHARSET(COMPRESS('abc')), COLLATION(COMPRESS('abc'));
+--------------------------+----------------------------+
| CHARSET(COMPRESS('abc')) | COLLATION(COMPRESS('abc')) |
+--------------------------+----------------------------+
| binary                   | binary                     |
+--------------------------+----------------------------+
```
