# 12.3.1 排序命名约定

> 译文：[`dev.mysql.com/doc/refman/8.0/en/charset-collation-names.html`](https://dev.mysql.com/doc/refman/8.0/en/charset-collation-names.html)

MySQL 排序名称遵循这些约定：

+   排序名称以其关联的字符集名称开头，通常后跟一个或多个指示其他排序特征的后缀。例如，`utf8mb4_0900_ai_ci` 和 `latin1_swedish_ci` 分别是 `utf8mb4` 和 `latin1` 字符集的排序。`binary` 字符集有一个单一的排序，也称为 `binary`，没有后缀。

+   语言特定的排序包括区域代码或语言名称。例如，`utf8mb4_tr_0900_ai_ci` 和 `utf8mb4_hu_0900_ai_ci` 分别使用土耳其和匈牙利的规则对 `utf8mb4` 字符集进行排序。`utf8mb4_turkish_ci` 和 `utf8mb4_hungarian_ci` 类似，但基于较早版本的 Unicode Collation Algorithm。

+   排序后缀指示排序是否区分大小写、重音或假名（或它们的某种组合），或者是二进制的。以下表格显示用于指示这些特征的后缀。

    **Table 12.1 排序后缀含义**

    | 后缀 | 含义 |
    | --- | --- |
    | `_ai` | 重音不敏感 |
    | `_as` | 重音敏感 |
    | `_ci` | 大小写不敏感 |
    | `_cs` | 大小写敏感 |
    | `_ks` | 假名敏感 |
    | `_bin` | 二进制 |

    对于未指定重音敏感性的非二进制排序名称，它由大小写敏感性确定。如果排序名称不包含 `_ai` 或 `_as`，名称中的 `_ci` 暗示 `_ai`，名称中的 `_cs` 暗示 `_as`。例如，`latin1_general_ci` 明确是大小写不敏感的，隐含是重音不敏感的，`latin1_general_cs` 明确是大小写敏感的，隐含是重音敏感的，`utf8mb4_0900_ai_ci` 明确是大小写不敏感和重音不敏感的。

    对于日语排序，`_ks` 后缀表示排序是假名敏感的；也就是说，它区分片假名字符和平假名字符。没有 `_ks` 后缀的日语排序不是假名敏感的，对片假名和平假名字符进行排序时视为相等。

    对于 `binary` 字符集的 `binary` 排序，比较基于数字字节值。对于非二进制字符集的 `_bin` 排序，比较基于数字字符编码值，对于多字节字符，这些值与字节值不同。有关 `binary` 字符集的 `binary` 排序和非二进制字符集的 `_bin` 排序之间的差异，请参阅 Section 12.8.5, “The binary Collation Compared to _bin Collations”。

+   Unicode 字符集的排序名称可能包含版本号，以指示排序所基于的 Unicode 排序算法（UCA）的版本。在名称中没有版本号的基于 UCA 的排序使用版本 4.0.0 的 UCA 权重键。例如：

    +   `utf8mb4_0900_ai_ci`基于 UCA 9.0.0 权重键（[`www.unicode.org/Public/UCA/9.0.0/allkeys.txt`](http://www.unicode.org/Public/UCA/9.0.0/allkeys.txt)）。

    +   `utf8mb4_unicode_520_ci`基于 UCA 5.2.0 权重键（[`www.unicode.org/Public/UCA/5.2.0/allkeys.txt`](http://www.unicode.org/Public/UCA/5.2.0/allkeys.txt)）。

    +   `utf8mb4_unicode_ci`（没有指定版本）基于 UCA 4.0.0 权重键（[`www.unicode.org/Public/UCA/4.0.0/allkeys-4.0.0.txt`](http://www.unicode.org/Public/UCA/4.0.0/allkeys-4.0.0.txt)）。

+   对于 Unicode 字符集，`*`xxx`*_general_mysql500_ci`排序保留了原始`*`xxx`*_general_ci`排序的 5.1.24 之前的顺序，并允许升级用于在 MySQL 5.1.24 之前创建的表（Bug #27877）。
