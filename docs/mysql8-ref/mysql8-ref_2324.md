> 原文：[`dev.mysql.com/doc/refman/8.0/en/case-sensitivity.html`](https://dev.mysql.com/doc/refman/8.0/en/case-sensitivity.html)

#### B.3.4.1 字符串搜索中的大小写敏感性

对于非二进制字符串（`CHAR`，`VARCHAR`，`TEXT`)，字符串搜索使用比较操作数的排序规则。对于二进制字符串（`BINARY`，`VARBINARY`，`BLOB`)，比较使用操作数中字节的数值；这意味着对于字母字符，比较是区分大小写的。

非二进制字符串与二进制字符串之间的比较被视为二进制字符串的比较。

简单的比较操作（`>=, >, =, <, <=`，排序和分组）基于每个字符的“排序值”。具有相同排序值的字符被视为相同字符。例如，如果在给定排序规则中`e`和`é`具有相同的排序值，则它们被视为相等。

默认字符集和排序规则为`utf8mb4`和`utf8mb4_0900_ai_ci`，因此默认情况下非二进制字符串比较是不区分大小写的。这意味着如果使用`*`col_name`* LIKE 'a%'`进行搜索，您将获得所有以`A`或`a`开头的列值。要使此搜索区分大小写，请确保其中一个操作数具有区分大小写或二进制排序规则。例如，如果要比较具有`utf8mb4`字符集的列和字符串，则可以使用`COLLATE`运算符使其中一个操作数具有`utf8mb4_0900_as_cs`或`utf8mb4_bin`排序规则：

```sql
*col_name* COLLATE utf8mb4_0900_as_cs LIKE 'a%'
*col_name* LIKE 'a%' COLLATE utf8mb4_0900_as_cs
*col_name* COLLATE utf8mb4_bin LIKE 'a%'
*col_name* LIKE 'a%' COLLATE utf8mb4_bin
```

如果要始终以区分大小写的方式处理某一列，请使用区分大小写或二进制排序规则声明该列。参见第 15.1.20 节，“CREATE TABLE 语句”。

要使非二进制字符串的区分大小写比较变为不区分大小写，请使用`COLLATE`命名一个不区分大小写的排序规则。以下示例中的字符串通常是区分大小写的，但`COLLATE`将比较变为不区分大小写：

```sql
mysql> SET NAMES 'utf8mb4';
mysql> SET @s1 = 'MySQL' COLLATE utf8mb4_bin,
           @s2 = 'mysql' COLLATE utf8mb4_bin;
mysql> SELECT @s1 = @s2;
+-----------+
| @s1 = @s2 |
+-----------+
|         0 |
+-----------+
mysql> SELECT @s1 COLLATE utf8mb4_0900_ai_ci = @s2;
+--------------------------------------+
| @s1 COLLATE utf8mb4_0900_ai_ci = @s2 |
+--------------------------------------+
|                                    1 |
+--------------------------------------+
```

二进制字符串在比较时区分大小写。要将字符串作为不区分大小写进行比较，请将其转换为非二进制字符串，并使用`COLLATE`命名一个不区分大小写的排序规则：

```sql
mysql> SET @s = BINARY 'MySQL';
mysql> SELECT @s = 'mysql';
+--------------+
| @s = 'mysql' |
+--------------+
|            0 |
+--------------+
mysql> SELECT CONVERT(@s USING utf8mb4) COLLATE utf8mb4_0900_ai_ci = 'mysql';
+----------------------------------------------------------------+
| CONVERT(@s USING utf8mb4) COLLATE utf8mb4_0900_ai_ci = 'mysql' |
+----------------------------------------------------------------+
|                                                              1 |
+----------------------------------------------------------------+
```

要确定值是作为非二进制还是二进制字符串进行比较，请使用`COLLATION()`函数。此示例显示`VERSION()`返回一个具有不区分大小写排序规则的字符串，因此比较是不区分大小写的：

```sql
mysql> SELECT COLLATION(VERSION());
+----------------------+
| COLLATION(VERSION()) |
+----------------------+
| utf8mb3_general_ci   |
+----------------------+
```

对于二进制字符串，排序值为`binary`，因此比较是区分大小写的。你可以预期在某些情况下会看到`binary`，比如压缩函数，通常返回二进制字符串：字符串：

```sql
mysql> SELECT COLLATION(COMPRESS('x'));
+--------------------------+
| COLLATION(COMPRESS('x')) |
+--------------------------+
| binary                   |
+--------------------------+
```

要检查字符串的排序值，`WEIGHT_STRING()`可能会有所帮助。参见 第 14.8 节，“字符串函数和运算符”。
