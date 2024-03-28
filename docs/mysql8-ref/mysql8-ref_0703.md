# 12.9.7 utf32 字符集（UTF-32 Unicode 编码）

> 原文：[`dev.mysql.com/doc/refman/8.0/en/charset-unicode-utf32.html`](https://dev.mysql.com/doc/refman/8.0/en/charset-unicode-utf32.html)

`utf32`字符集是固定长度的（类似于`ucs2`，不同于`utf16`）。`utf32`为每个字符使用 32 位，不同于`ucs2`（每个字符使用 16 位），也不同于`utf16`（某些字符使用 16 位，某些使用 32 位）。

`utf32`占用的空间是`ucs2`的两倍，比`utf16`更多，但`utf32`和`ucs2`一样具有存储的优势：`utf32`所需的字节数等于字符数乘以 4。此外，与`utf16`不同，`utf32`没有编码技巧，因此存储的值等于代码值。

为了展示后一种优势的用处，这里有一个示例，展示如何根据`utf32`的代码值确定一个`utf8mb4`的值：

```sql
/* Assume code value = 100cc LINEAR B WHEELED CHARIOT */
CREATE TABLE tmp (utf32_col CHAR(1) CHARACTER SET utf32,
                  utf8mb4_col CHAR(1) CHARACTER SET utf8mb4);
INSERT INTO tmp VALUES (0x000100cc,NULL);
UPDATE tmp SET utf8mb4_col = utf32_col;
SELECT HEX(utf32_col),HEX(utf8mb4_col) FROM tmp;
```

MySQL 对未分配的 Unicode 字符或专用区域字符的添加非常宽容。实际上，`utf32`只有一个有效性检查：没有代码值可以大于`0x10ffff`。例如，这是不合法的：

```sql
INSERT INTO t (utf32_column) VALUES (0x110000); /* illegal */
```
