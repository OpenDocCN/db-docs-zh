# 12.14.4 向 Unicode 字符集添加 UCA 排序

> 原文：[`dev.mysql.com/doc/refman/8.0/en/adding-collation-unicode-uca.html`](https://dev.mysql.com/doc/refman/8.0/en/adding-collation-unicode-uca.html)

12.14.4.1 使用 LDML 语法定义 UCA 排序

12.14.4.2 MySQL 支持的 LDML 语法

12.14.4.3 在 Index.xml 解析期间的诊断

本节描述了如何通过在 MySQL 的 `Index.xml` 文件中的 `<charset>` 字符集描述中编写 `<collation>` 元素来添加 Unicode 字符集的 UCA 排序。这里描述的过程不需要重新编译 MySQL。它使用了 Locale Data Markup Language (LDML) 规范的一个子集，可在 [`www.unicode.org/reports/tr35/`](http://www.unicode.org/reports/tr35/) 上找到。使用这种方法，您无需定义整个排序。相反，您从现有的“基本”排序开始，并描述新排序与基本排序的不同之处。以下表列出了可以定义 UCA 排序的 Unicode 字符集的基本排序。无法为 `utf16le` 创建用户定义的 UCA 排序；没有 `utf16le_unicode_ci` 排序可作为此类排序的基础。

**表 12.4 可用于用户定义 UCA 排序的 MySQL 字符集**

| 字符集 | 基本排序 |
| --- | --- |
| `utf8mb4` | `utf8mb4_unicode_ci` |
| `ucs2` | `ucs2_unicode_ci` |
| `utf16` | `utf16_unicode_ci` |
| `utf32` | `utf32_unicode_ci` |

以下部分展示了如何添加使用 LDML 语法定义的排序，并提供了 MySQL 支持的 LDML 规则摘要。
