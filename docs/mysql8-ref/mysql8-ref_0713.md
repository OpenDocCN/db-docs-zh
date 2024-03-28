> 原文：[`dev.mysql.com/doc/refman/8.0/en/charset-cp932.html`](https://dev.mysql.com/doc/refman/8.0/en/charset-cp932.html)

#### 12.10.7.1 `cp932`字符集

**为什么需要`cp932`？**

在 MySQL 中，`sjis`字符集对应于 IANA 定义的`Shift_JIS`字符集，支持 JIS X0201 和 JIS X0208 字符。（参见[`www.iana.org/assignments/character-sets`](http://www.iana.org/assignments/character-sets)。）

然而，“SHIFT JIS”作为一个描述性术语的含义变得非常模糊，通常包括各种供应商定义的`Shift_JIS`扩展。

例如，在日本 Windows 环境中使用的“SHIFT JIS”是`Shift_JIS`的微软扩展，其确切名称为`Microsoft Windows Codepage : 932`或`cp932`。除了`Shift_JIS`支持的字符外，`cp932`还支持扩展字符，如 NEC 特殊字符、NEC 选定-IBM 扩展字符和 IBM 选定字符。

许多日本用户在使用这些扩展字符时遇到问题。这些问题源于以下因素：

+   MySQL 会自动转换字符集。

+   字符集使用 Unicode（`ucs2`）进行转换。

+   `sjis`字符集不支持这些扩展字符的转换。

+   从所谓的“SHIFT JIS”到 Unicode 有几种转换规则，根据转换规则的不同，一些字符转换为 Unicode 的方式也不同。MySQL 仅支持其中一种规则（稍后描述）。

MySQL 的`cp932`字符集旨在解决这些问题。

因为 MySQL 支持字符集转换，所以将 IANA 的`Shift_JIS`和`cp932`分开成两个不同的字符集非常重要，因为它们提供不同的转换规则。

**`cp932`与`sjis`有何不同？**

`cp932`字符集与`sjis`有以下不同：

+   `cp932`支持 NEC 特殊字符、NEC 选定-IBM 扩展字符和 IBM 选定字符。

+   一些`cp932`字符有两个不同的代码点，两者都转换为相同的 Unicode 代码点。在从 Unicode 转换回`cp932`时，必须选择其中一个代码点。对于这种“往返转换”，使用微软推荐的规则。（参见[`support.microsoft.com/kb/170559/EN-US/`](http://support.microsoft.com/kb/170559/EN-US/)。）

    转换规则如下：

    +   如果字符同时在 JIS X 0208 和 NEC 特殊字符中，则使用 JIS X 0208 的代码点。

    +   如果字符同时在 NEC 特殊字符和 IBM 选定字符中，则使用 NEC 特殊字符的代码点。

    +   如果字符同时在 IBM 选定字符和 NEC 选定-IBM 扩展字符中，则使用 IBM 扩展字符的代码点。

    [`msdn.microsoft.com/en-us/goglobal/cc305152.aspx`](https://msdn.microsoft.com/en-us/goglobal/cc305152.aspx) 上显示的表提供了关于 `cp932` 字符的 Unicode 值的信息。对于 `cp932` 表中带有四位数值下字符的条目，该数字代表相应的 Unicode（`ucs2`）编码。对于带有下划线的两位数值下字符的条目，存在以这两位数值开头的一系列 `cp932` 字符值。点击这样的表条目会带您到一个页面，显示以这些数字开头的每个 `cp932` 字符的 Unicode 值。

    以下链接非常重要。它们对应以下字符集的编码：

    +   NEC 特殊字符（主字节 `0x87`）：

        ```sql
        https://msdn.microsoft.com/en-us/goglobal/gg674964
        ```

    +   NEC 选定—IBM 扩展字符（主字节 `0xED` 和 `0xEE`）：

        ```sql
        https://msdn.microsoft.com/en-us/goglobal/gg671837
        https://msdn.microsoft.com/en-us/goglobal/gg671838
        ```

    +   IBM 选定字符（主字节 `0xFA`、`0xFB`、`0xFC`）：

        ```sql
        https://msdn.microsoft.com/en-us/goglobal/gg671839
        https://msdn.microsoft.com/en-us/goglobal/gg671840
        https://msdn.microsoft.com/en-us/goglobal/gg671841
        ```

+   `cp932` 支持与 `eucjpms` 结合使用的用户定义字符的转换，并解决了 `sjis`/`ujis` 转换的问题。详情请参考 [`www.sljfaq.org/afaq/encodings.html`](http://www.sljfaq.org/afaq/encodings.html)。

对于一些字符，`sjis` 和 `cp932` 的 `ucs2` 转换是不同的。以下表格展示了这些差异。

转换为 `ucs2`：

| `sjis`/`cp932` 值 | `sjis` -> `ucs2` 转换 | `cp932` -> `ucs2` 转换 |
| --- | --- | --- |
| 5C | 005C | 005C |
| 7E | 007E | 007E |
| 815C | 2015 | 2015 |
| 815F | 005C | FF3C |
| 8160 | 301C | FF5E |
| 8161 | 2016 | 2225 |
| 817C | 2212 | FF0D |
| 8191 | 00A2 | FFE0 |
| 8192 | 00A3 | FFE1 |
| 81CA | 00AC | FFE2 |
| `sjis`/`cp932` 值 | `sjis` -> `ucs2` 转换 | `cp932` -> `ucs2` 转换 |

从 `ucs2` 转换：

| `ucs2` 值 | `ucs2` -> `sjis` 转换 | `ucs2` -> `cp932` 转换 |
| --- | --- | --- |
| 005C | 815F | 5C |
| 007E | 7E | 7E |
| 00A2 | 8191 | 3F |
| 00A3 | 8192 | 3F |
| 00AC | 81CA | 3F |
| 2015 | 815C | 815C |
| 2016 | 8161 | 3F |
| 2212 | 817C | 3F |
| 2225 | 3F | 8161 |
| 301C | 8160 | 3F |
| FF0D | 3F | 817C |
| FF3C | 3F | 815F |
| FF5E | 3F | 8160 |
| FFE0 | 3F | 8191 |
| FFE1 | 3F | 8192 |
| FFE2 | 3F | 81CA |
| `ucs2` 值 | `ucs2` -> `sjis` 转换 | `ucs2` -> `cp932` 转换 |

任何日文字符集的用户都应该注意使用 `--character-set-client-handshake`（或 `--skip-character-set-client-handshake`）会产生重要影响。请参阅 7.1.7 “服务器命令选项”。
