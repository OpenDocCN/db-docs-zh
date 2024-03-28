# 13.3.3 BINARY 和 VARBINARY 类型

> 原文：[`dev.mysql.com/doc/refman/8.0/en/binary-varbinary.html`](https://dev.mysql.com/doc/refman/8.0/en/binary-varbinary.html)

`BINARY` 和 `VARBINARY` 类型类似于 `CHAR` 和 `VARCHAR`，只是它们存储二进制字符串而不是非二进制字符串。也就是说，它们存储字节字符串而不是字符字符串。这意味着它们具有 `binary` 字符集和校对，比较和排序基于值中字节的数值。

`BINARY` 和 `VARBINARY` 的最大长度与 `CHAR` 和 `VARCHAR` 的最大长度相同，只是 `BINARY` 和 `VARBINARY` 的长度是以字节而不是字符计量的。

`BINARY` 和 `VARBINARY` 数据类型与 `CHAR BINARY` 和 `VARCHAR BINARY` 数据类型不同。对于后者，`BINARY` 属性不会导致列被视为二进制字符串列。相反，它会导致使用列字符集的二进制 (`_bin`) 校对（如果未指定列字符集，则使用表默认字符集），并且列本身存储非二进制字符字符串而不是二进制字节字符串。例如，如果默认字符集是 `utf8mb4`，`CHAR(5) BINARY` 被视为 `CHAR(5) CHARACTER SET utf8mb4 COLLATE utf8mb4_bin`。这与 `BINARY(5)` 不同，后者存储具有 `binary` 字符集和校对的 5 字节二进制字符串。有关 `binary` 字符集的 `binary` 校对与非二进制字符集的 `_bin` 校对之间的差异的信息，请参见 Section 12.8.5, “The binary Collation Compared to _bin Collations”。

如果未启用严格的 SQL 模式，并且将值分配给超出列最大长度的 `BINARY` 或 `VARBINARY` 列，则该值将被截断以适应，并生成警告。对于截断的情况，要导致发生错误（而不是警告）并阻止插入该值，请使用严格的 SQL 模式。请参见 Section 7.1.11, “Server SQL Modes”。

存储 `BINARY` 值时，它们会使用填充值右填充到指定长度。填充值为 `0x00`（零字节）。对于插入，值会使用 `0x00` 右填充，检索时不会删除尾随字节。在比较中，所有字节都是重要的，包括 `ORDER BY` 和 `DISTINCT` 操作。`0x00` 和空格在比较中不同，`0x00` 排在空格之前。

例如：对于 `BINARY(3)` 列，`'a '` 在插入时变为 `'a \0'`。`'a\0'` 在插入时变为 `'a\0\0'`。这两个插入值在检索时保持不变。

对于`VARBINARY`，插入时不进行填充，检索时不剥离任何字节。在比较中，所有字节都是重要的，包括`ORDER BY`和`DISTINCT`操作。在比较中，`0x00`和空格是不同的，`0x00`在排序中排在空格之前。

对于那些剥离尾随填充字节或比较忽略它们的情况，如果列具有需要唯一值的索引，那么在列中插入仅在尾随填充字节数量上不同的值会导致重复键错误。例如，如果表中包含`'a'`，尝试存储`'a\0'`会导致重复键错误。

如果您计划使用`BINARY`数据类型存储二进制数据，并且需要检索的值与存储的值完全相同，那么您应该仔细考虑前导填充和剥离特性。以下示例说明了`BINARY`值的`0x00`填充如何影响列值比较：

```sql
mysql> CREATE TABLE t (c BINARY(3));
Query OK, 0 rows affected (0.01 sec)

mysql> INSERT INTO t SET c = 'a';
Query OK, 1 row affected (0.01 sec)

mysql> SELECT HEX(c), c = 'a', c = 'a\0\0' from t;
+--------+---------+-------------+
| HEX(c) | c = 'a' | c = 'a\0\0' |
+--------+---------+-------------+
| 610000 |       0 |           1 |
+--------+---------+-------------+
1 row in set (0.09 sec)
```

如果检索的值必须与存储时指定的值完全相同且没有填充，则最好使用`VARBINARY`或`BLOB`数据类型之一。

注意

在**mysql**客户端中，二进制字符串使用十六进制表示，取决于`--binary-as-hex`选项的值。有关该选项的更多信息，请参见 Section 6.5.1, “mysql — The MySQL Command-Line Client”。
