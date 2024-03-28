# 12.9.4 `ucs2`字符集（UCS-2 Unicode 编码）

> 原文：[`dev.mysql.com/doc/refman/8.0/en/charset-unicode-ucs2.html`](https://dev.mysql.com/doc/refman/8.0/en/charset-unicode-ucs2.html)

注意

`ucs2`字符集在 MySQL 8.0.28 中已被弃用；预计将在未来的 MySQL 版本中移除。请改用`utf8mb4`。

在 UCS-2 中，每个字符由一个 2 字节的 Unicode 代码表示，最高有效字节在前。例如：`LATIN CAPITAL LETTER A`的代码为`0x0041`，存储为一个 2 字节序列：`0x00 0x41`。`CYRILLIC SMALL LETTER YERU`（Unicode `0x044B`）存储为一个 2 字节序列：`0x04 0x4B`。有关 Unicode 字符及其代码，请参考[Unicode Consortium website](http://www.unicode.org/)。

`ucs2`字符集具有以下特点：

+   仅支持 BMP 字符（不支持补充字符）

+   使用固定长度的 16 位编码，每个字符需要两个字节。
