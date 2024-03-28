# 12.9.5 `utf16`字符集（UTF-16 Unicode 编码）

> 原文：[`dev.mysql.com/doc/refman/8.0/en/charset-unicode-utf16.html`](https://dev.mysql.com/doc/refman/8.0/en/charset-unicode-utf16.html)

`utf16`字符集是`ucs2`字符集的扩展，使其能够编码补充字符：

+   对于 BMP 字符，`utf16`和`ucs2`具有相同的存储特性：相同的代码值，相同的编码，相同的长度。

+   对于补充字符，`utf16`有一个特殊的序列来使用 32 位表示字符。这被称为“代理”机制：对于大于`0xffff`的数字，取 10 位并将它们加到`0xd800`，放入第一个 16 位字中，再取 10 位并将它们加到`0xdc00`，放入下一个 16 位字中。因此，所有补充字符需要 32 位，其中前 16 位是介于`0xd800`和`0xdbff`之间的数字，后 16 位是介于`0xdc00`和`0xdfff`之间的数字。示例在 Unicode 4.0 文档的第[15.5 代理区](http://www.unicode.org/versions/Unicode4.0.0/ch15.pdf)中。

因为`utf16`支持代理而`ucs2`不支持，只有在`utf16`中才适用的有效性检查：您不能插入顶部代理而没有底部代理，反之亦然。例如：

```sql
INSERT INTO t (ucs2_column) VALUES (0xd800); /* legal */
INSERT INTO t (utf16_column)VALUES (0xd800); /* illegal */
```

对于技术上有效但不是真正 Unicode 的字符，没有有效性检查（即 Unicode 认为是“未分配代码点”或“私有使用”字符甚至“非法”字符，如`0xffff`）。例如，由于`U+F8FF`是苹果 Logo，这是合法的：

```sql
INSERT INTO t (utf16_column)VALUES (0xf8ff); /* legal */
```

这些字符不能期望对每个人都有相同的含义。

因为 MySQL 必须考虑最坏情况（一个字符需要四个字节），`utf16`列或索引的最大长度仅为`ucs2`列或索引的最大长度的一半。例如，`MEMORY`表索引键的最大长度为 3072 字节，因此这些语句创建具有`ucs2`和`utf16`列的最长允许索引的表：

```sql
CREATE TABLE tf (s1 VARCHAR(1536) CHARACTER SET ucs2) ENGINE=MEMORY;
CREATE INDEX i ON tf (s1);
CREATE TABLE tg (s1 VARCHAR(768) CHARACTER SET utf16) ENGINE=MEMORY;
CREATE INDEX i ON tg (s1);
```
