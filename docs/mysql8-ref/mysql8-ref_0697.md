# 12.9.1 utf8mb4 字符集（4 字节 UTF-8 Unicode 编码）

> 原文：[`dev.mysql.com/doc/refman/8.0/en/charset-unicode-utf8mb4.html`](https://dev.mysql.com/doc/refman/8.0/en/charset-unicode-utf8mb4.html)

`utf8mb4`字符集具有以下特点：

+   支持 BMP 和补充字符。

+   每个多字节字符最多需要四个字节。

`utf8mb4`与仅支持 BMP 字符并且每个字符最多使用三个字节的`utf8mb3`字符集形成对比：

+   对于 BMP 字符，`utf8mb4`和`utf8mb3`具有相同的存储特性：相同的代码值，相同的编码，相同的长度。

+   对于补充字符，`utf8mb4`需要四个字节来存储，而`utf8mb3`根本无法存储该字符。当将`utf8mb3`列转换为`utf8mb4`时，您无需担心转换补充字符，因为根本没有。

`utf8mb4`是`utf8mb3`的超集，因此对于诸如以下连接操作，结果具有字符集`utf8mb4`和`utf8mb4_col`的排序规则：

```sql
SELECT CONCAT(utf8mb3_col, utf8mb4_col);
```

同样，`WHERE`子句中的以下比较按照`utf8mb4_col`的排序规则进行：

```sql
SELECT * FROM utf8mb3_tbl, utf8mb4_tbl
WHERE utf8mb3_tbl.utf8mb3_col = utf8mb4_tbl.utf8mb4_col;
```

关于与多字节字符集相关的数据类型存储信息，请参阅字符串类型存储要求。
