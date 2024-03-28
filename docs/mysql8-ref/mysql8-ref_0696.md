# 12.9 Unicode 支持

> 原文：[`dev.mysql.com/doc/refman/8.0/en/charset-unicode.html`](https://dev.mysql.com/doc/refman/8.0/en/charset-unicode.html)

12.9.1 utf8mb4 字符集（4 字节 UTF-8 Unicode 编码）

12.9.2 utf8mb3 字符集（3 字节 UTF-8 Unicode 编码）

12.9.3 utf8 字符集（utf8mb3 的弃用别名）

12.9.4 ucs2 字符集（UCS-2 Unicode 编码）

12.9.5 utf16 字符集（UTF-16 Unicode 编码）

12.9.6 utf16le 字符集（UTF-16LE Unicode 编码）

12.9.7 utf32 字符集（UTF-32 Unicode 编码）

12.9.8 在 3 字节和 4 字节 Unicode 字符集之间转换

Unicode 标准包括来自基本多语言平面（BMP）和超出 BMP 范围的补充字符。本节描述了 MySQL 中对 Unicode 的支持。有关 Unicode 标准本身的信息，请访问[Unicode Consortium 网站](http://www.unicode.org/)。

BMP 字符具有以下特征：

+   它们的代码点值介于 0 和 65535 之间（或`U+0000`和`U+FFFF`）。

+   它们可以使用 8、16 或 24 位（1 到 3 字节）的可变长度编码进行编码。

+   它们可以使用 16 位（2 字节）的固定长度编码进行编码。

+   它们几乎可以涵盖所有主要语言中的所有字符。

补充字符超出 BMP 范围：

+   它们的代码点值介于`U+10000`和`U+10FFFF`之间。

+   对补充字符的 Unicode 支持需要具有超出 BMP 字符范围的范围的字符集，因此比 BMP 字符占用更多的空间（每个字符最多 4 个字节）。

用于编码 Unicode 数据的 UTF-8（8 位单元的 Unicode 转换格式）方法根据 RFC 3629 实现，描述了采用从一个到四个字节的编码序列的编码序列。UTF-8 的理念是使用不同长度的字节序列对各种 Unicode 字符进行编码：

+   基本拉丁字母、数字和标点符号使用一个字节。

+   大多数欧洲和中东文字母适合于 2 字节序列：扩展拉丁字母（带有颚化音符、长音符、重音符、重音符和其他重音符）、西里尔字母、希腊字母、亚美尼亚字母、希伯来字母、阿拉伯字母、叙利亚字母等。

+   韩文、中文和日文表意文字使用 3 字节或 4 字节序列。

MySQL 支持这些 Unicode 字符集：

+   `utf8mb4`：使用每个字符 1 到 4 个字节的 Unicode 字符集的 UTF-8 编码。

+   `utf8mb3`：使用每个字符 1 到 3 个字节的 Unicode 字符集的 UTF-8 编码。这个字符集在 MySQL 8.0 中已被弃用，你应该使用`utf8mb4`。

+   `utf8`：`utf8mb3`的别名。在 MySQL 8.0 中，这个别名已被弃用；请改用`utf8mb4`。预计在未来的版本中，`utf8`将成为`utf8mb4`的别名。

+   `ucs2`：Unicode 字符集的 UCS-2 编码，每个字符使用两个字节。在 MySQL 8.0.28 中已弃用；您应该预期在将来的版本中删除对该字符集的支持。

+   `utf16`：Unicode 字符集的 UTF-16 编码，每个字符使用两个或四个字节。类似于`ucs2`，但具有补充字符的扩展。

+   `utf16le`：Unicode 字符集的 UTF-16LE 编码。类似于`utf16`，但是小端序而不是大端序。

+   `utf32`：Unicode 字符集的 UTF-32 编码，每个字符使用四个字节。

注意

`utf8mb3`字符集已被弃用，您应该预期在将来的 MySQL 版本中将其删除。请改用`utf8mb4`。`utf8`目前是`utf8mb3`的别名，但现在已被弃用，预计`utf8`随后将成为对`utf8mb4`的引用。从 MySQL 8.0.28 开始，在 Information Schema 表的列和 SQL `SHOW`语句的输出中，`utf8mb3`也会显示为`utf8`的替代项。

另外，在 MySQL 8.0.30 中，所有使用`utf8_`前缀的校对规则都将改名为`utf8mb3_`。

为避免关于`utf8`含义的歧义，考虑明确指定字符集引用为`utf8mb4`。

表 12.2，“Unicode 字符集的一般特性”总结了 MySQL 支持的 Unicode 字符集的一般特性。

**表 12.2 Unicode 字符集的一般特性**

| 字符集 | 支持的字符 | 每个字符所需的存储空间 |
| --- | --- | --- |
| `utf8mb3`，`utf8`（已弃用） | 仅限 BMP | 1、2 或 3 字节 |
| `ucs2` | 仅限 BMP | 2 字节 |
| `utf8mb4` | BMP 和补充 | 1、2、3 或 4 字节 |
| `utf16` | BMP 和补充 | 2 或 4 字节 |
| `utf16le` | BMP 和补充 | 2 或 4 字节 |
| `utf32` | BMP 和补充 | 4 字节 |

超出 BMP 范围的字符在转换为仅支持 BMP 字符的 Unicode 字符集（`utf8mb3`或`ucs2`）时会被视为`REPLACEMENT CHARACTER`并转换为`'?'`。

如果您使用支持补充字符且比仅支持 BMP 的`utf8mb3`和`ucs2`字符集“更宽”的字符集，您的应用程序可能存在潜在的不兼容性问题；请参阅第 12.9.8 节，“在 3 字节和 4 字节 Unicode 字符集之间转换”。该部分还描述了如何将表从（3 字节）`utf8mb3`转换为（4 字节）`utf8mb4`，以及在这样做时可能适用的约束条件。

大多数 Unicode 字符集都有类似的排序规则集。例如，每个字符集都有一个丹麦排序规则，其名称分别为`utf8mb4_danish_ci`、`utf8mb3_danish_ci`（已弃用）、`utf8_danish_ci`（已弃用）、`ucs2_danish_ci`、`utf16_danish_ci`和`utf32_danish_ci`。唯一的例外是`utf16le`，它只有两个排序规则。有关 Unicode 排序规则及其区分特性的信息，包括辅助字符的排序规则属性，请参阅第 12.10.1 节，“Unicode 字符集”。

MySQL 对于 UCS-2、UTF-16 和 UTF-32 的实现以大端字节顺序存储字符，并且在数值开头不使用字节顺序标记（BOM）。其他数据库系统可能使用小端字节顺序或者 BOM。在这种情况下，在这些系统和 MySQL 之间传输数据时需要执行数值转换。UTF-16LE 的实现是小端字节顺序。

MySQL 对于 UTF-8 数值不使用 BOM。

与服务器使用 Unicode 通信的客户端应该相应地设置客户端字符集（例如，通过发出`SET NAMES 'utf8mb4'`语句）。有些字符集不能用作客户端字符集。尝试在`SET NAMES`或`SET CHARACTER SET`中使用它们会产生错误。请参阅不允许的客户端字符集。

以下章节提供了关于 MySQL 中 Unicode 字符集的额外细节。
