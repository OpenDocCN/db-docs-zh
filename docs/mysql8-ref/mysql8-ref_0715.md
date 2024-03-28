# 12.10.8 二进制字符集

> 原文：[`dev.mysql.com/doc/refman/8.0/en/charset-binary-set.html`](https://dev.mysql.com/doc/refman/8.0/en/charset-binary-set.html)

`binary`字符集是二进制字符串的字符集，即字节序列。`binary`字符集有一个排序规则，也称为`binary`。比较和排序基于数字字节值，而不是数字字符代码值（对于多字节字符，它们与数字字节值不同）。有关`binary`字符集的`binary`排序规则与非二进制字符集的`_bin`排序规则之间的区别，请参见第 12.8.5 节，“与 _bin 排序规则相比的 binary 排序规则”。

对于`binary`字符集，大小写和重音等效的概念不适用：

+   对于存储为二进制字符串的单字节字符，字符和字节边界相同，因此在比较中大小写和重音差异很重要。也就是说，`binary`排序是区分大小写和重音的。

    ```sql
    mysql> SET NAMES 'binary';
    mysql> SELECT CHARSET('abc'), COLLATION('abc');
    +----------------+------------------+
    | CHARSET('abc') | COLLATION('abc') |
    +----------------+------------------+
    | binary         | binary           |
    +----------------+------------------+
    mysql> SELECT 'abc' = 'ABC', 'a' = 'ä';
    +---------------+------------+
    | 'abc' = 'ABC' | 'a' = 'ä'  |
    +---------------+------------+
    |             0 |          0 |
    +---------------+------------+
    ```

+   对于存储为二进制字符串的多字节字符，字符和字节边界不同。字符边界丢失，因此依赖于它们的比较是没有意义的。

要对二进制字符串执行大小写转换，首先将其转换为使用适合字符串中存储的数据的字符集的非二进制字符串：

```sql
mysql> SET @str = BINARY 'New York';
mysql> SELECT LOWER(@str), LOWER(CONVERT(@str USING utf8mb4));
+-------------+------------------------------------+
| LOWER(@str) | LOWER(CONVERT(@str USING utf8mb4)) |
+-------------+------------------------------------+
| New York    | new york                           |
+-------------+------------------------------------+
```

要将字符串表达式转换为二进制字符串，以下构造是等效的：

```sql
BINARY *expr*
CAST(*expr* AS BINARY)
CONVERT(*expr* USING BINARY)
```

如果一个值是字符字符串文字，则可以使用`_binary`介绍符将其指定为二进制字符串。例如：

```sql
_binary 'a'
```

`_binary`介绍符也适用于十六进制文字和位值文字，但是不必要；这些文字默认为二进制字符串。

有关介绍符的更多信息，请参见第 12.3.8 节，“字符集介绍符”。

注意

在**mysql**客户端中，二进制字符串使用十六进制表示，具体取决于`--binary-as-hex`的值。有关该选项的更多信息，请参见第 6.5.1 节，“mysql — MySQL 命令行客户端”。
