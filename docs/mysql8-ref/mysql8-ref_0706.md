# 12.10.1 Unicode 字符集

> 原文：[`dev.mysql.com/doc/refman/8.0/en/charset-unicode-sets.html`](https://dev.mysql.com/doc/refman/8.0/en/charset-unicode-sets.html)

本节描述了 Unicode 字符集可用的排序规则及其区分特性。有关 Unicode 的一般信息，请参见第 12.9 节，“Unicode 支持”。

MySQL 支持多个 Unicode 字符集：

+   `utf8mb4`: 使用每个字符一到四个字节的 Unicode 字符集的 UTF-8 编码。

+   `utf8mb3`: 使用每个字符一到三个字节的 Unicode 字符集的 UTF-8 编码。此字符集在 MySQL 8.0 中已弃用，您应该改用`utf8mb4`。

+   `utf8`: 对`utf8mb3`的别名。在 MySQL 8.0 中，此别名已被弃用；请改用`utf8mb4`。预计在将来的版本中，`utf8`将成为`utf8mb4`的别名。

+   `ucs2`: 使用每个字符两个字节的 Unicode 字符集的 UCS-2 编码。在 MySQL 8.0.28 中已弃用；您应该预期在将来的版本中删除对此字符集的支持。

+   `utf16`: Unicode 字符集的 UTF-16 编码，每个字符使用两个或四个字节。类似于`ucs2`但具有用于补充字符的扩展。

+   `utf16le`: Unicode 字符集的 UTF-16LE 编码。类似于`utf16`但是小端序而不是大端序。

+   `utf32`: 使用每个字符四个字节的 UTF-32 编码的 Unicode 字符集。

注意

`utf8mb3`字符集已被弃用，您应该预期在将来的 MySQL 版本中将其移除。请改用`utf8mb4`。`utf8`目前是`utf8mb3`的别名，但现在已被弃用，`utf8`预计随后将成为`utf8mb4`的引用。从 MySQL 8.0.28 开始，在 Information Schema 表的列和 SQL `SHOW`语句的输出中，`utf8mb3`也会显示为`utf8`的替代项。

为了避免关于`utf8`含义的歧义，考虑在字符集引用中明确指定`utf8mb4`。

`utf8mb4`、`utf16`、`utf16le`和`utf32`支持基本多文种平面（BMP）字符和超出 BMP 范围的补充字符。`utf8mb3`和`ucs2`仅支持 BMP 字符。

大多数 Unicode 字符集都有一般排序规则（名称中带有`_general`或没有语言说明符号），二进制排序规则（名称中带有`_bin`），以及几种特定语言的排序规则（带有语言说明符号）。例如，对于`utf8mb4`，`utf8mb4_general_ci`和`utf8mb4_bin`是其一般和二进制排序规则，而`utf8mb4_danish_ci`是其特定语言之一的排序规则。

大多数字符集具有单一的二进制排序。`utf8mb4` 是一个例外，它有两个：`utf8mb4_bin` 和（从 MySQL 8.0.17 开始）`utf8mb4_0900_bin`。这两个二进制排序具有相同的排序顺序，但通过它们的填充属性和排序权重特性进行区分。参见 排序填充属性 和 字符排序权重。

`utf16le` 的排序支持有限。唯一可用的排序是 `utf16le_general_ci` 和 `utf16le_bin`。这些与 `utf16_general_ci` 和 `utf16_bin` 类似。

+   Unicode Collation Algorithm (UCA) 版本 版本")

+   排序填充属性

+   特定语言排序

+   _general_ci 与 _unicode_ci 排序

+   字符排序权重

+   杂项信息

#### Unicode Collation Algorithm (UCA) 版本

MySQL 根据 [`www.unicode.org/reports/tr10/`](http://www.unicode.org/reports/tr10/) 中描述的 Unicode Collation Algorithm (UCA) 实现 `*`xxx`*_unicode_ci` 排序。该排序使用版本-4.0.0 UCA 权重键：[`www.unicode.org/Public/UCA/4.0.0/allkeys-4.0.0.txt`](http://www.unicode.org/Public/UCA/4.0.0/allkeys-4.0.0.txt)。`*`xxx`*_unicode_ci` 排序对 Unicode Collation Algorithm 仅有部分支持。某些字符不受支持，组合标记也没有完全支持。这影响到越南语、约鲁巴语和纳瓦霍语等语言。在字符串比较中，组合字符被视为与使用单个 Unicode 字符写的相同字符不同，并且这两个字符被认为具有不同的长度（例如，由 `CHAR_LENGTH()` 函数返回或在结果集元数据中返回）。

基于高于 4.0.0 的 UCA 版本的 Unicode 排序在排序名称中包含版本。例如：

+   `utf8mb4_unicode_520_ci` 基于 UCA 5.2.0 权重键 ([`www.unicode.org/Public/UCA/5.2.0/allkeys.txt`](http://www.unicode.org/Public/UCA/5.2.0/allkeys.txt)),

+   `utf8mb4_0900_ai_ci` 基于 UCA 9.0.0 权重键 ([`www.unicode.org/Public/UCA/9.0.0/allkeys.txt`](http://www.unicode.org/Public/UCA/9.0.0/allkeys.txt)).

`LOWER()`和`UPPER()`函数根据其参数的校对执行大小写折叠。如果一个字符只有在 Unicode 版本高于 4.0.0 中才有大写和小写版本，则这些函数只会在参数校对使用足够高的 UCA 版本时转换该字符。

#### 校对填充属性

基于 UCA 9.0.0 及更高版本的校对比基于 UCA 9.0.0 之前版本的校对更快。它们的填充属性也是`NO PAD`，与基于 UCA 9.0.0 之前版本的校对中使用的`PAD SPACE`相反。对于非二进制字符串的比较，`NO PAD`校对将字符串末尾的空格视为任何其他字符（参见比较中的尾随空格处理）。

要确定校对的填充属性，请使用`INFORMATION_SCHEMA`的`COLLATIONS`表，该表具有`PAD_ATTRIBUTE`列。例如：

```sql
mysql> SELECT COLLATION_NAME, PAD_ATTRIBUTE
       FROM INFORMATION_SCHEMA.COLLATIONS
       WHERE CHARACTER_SET_NAME = 'utf8mb4';
+----------------------------+---------------+
| COLLATION_NAME             | PAD_ATTRIBUTE |
+----------------------------+---------------+
| utf8mb4_general_ci         | PAD SPACE     |
| utf8mb4_bin                | PAD SPACE     |
| utf8mb4_unicode_ci         | PAD SPACE     |
| utf8mb4_icelandic_ci       | PAD SPACE     |
...
| utf8mb4_0900_ai_ci         | NO PAD        |
| utf8mb4_de_pb_0900_ai_ci   | NO PAD        |
| utf8mb4_is_0900_ai_ci      | NO PAD        |
...
| utf8mb4_ja_0900_as_cs      | NO PAD        |
| utf8mb4_ja_0900_as_cs_ks   | NO PAD        |
| utf8mb4_0900_as_ci         | NO PAD        |
| utf8mb4_ru_0900_ai_ci      | NO PAD        |
| utf8mb4_ru_0900_as_cs      | NO PAD        |
| utf8mb4_zh_0900_as_cs      | NO PAD        |
| utf8mb4_0900_bin           | NO PAD        |
+----------------------------+---------------+
```

具有`NO PAD`校对的非二进制字符串值（`CHAR`，`VARCHAR`和`TEXT`）与其他校对在尾随空格方面有所不同。例如，`'a'`和`'a '`比较为不同的字符串，而不是相同的字符串。可以使用`utf8mb4`的二进制校对来看到这一点。`utf8mb4_bin`的填充属性是`PAD SPACE`，而`utf8mb4_0900_bin`的填充属性是`NO PAD`。因此，涉及`utf8mb4_0900_bin`的操作不会添加尾随空格，并且涉及具有尾随空格的字符串的比较可能会因为两种校对而有所不同：

```sql
mysql> CREATE TABLE t1 (c CHAR(10) COLLATE utf8mb4_bin);
Query OK, 0 rows affected (0.03 sec)

mysql> INSERT INTO t1 VALUES('a');
Query OK, 1 row affected (0.01 sec)

mysql> SELECT * FROM t1 WHERE c = 'a ';
+------+
| c    |
+------+
| a    |
+------+
1 row in set (0.00 sec)

mysql> ALTER TABLE t1 MODIFY c CHAR(10) COLLATE utf8mb4_0900_bin;
Query OK, 0 rows affected (0.02 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> SELECT * FROM t1 WHERE c = 'a ';
Empty set (0.00 sec)
```

#### 语言特定的校对

如果仅基于 Unicode 校对算法（UCA）的排序对某种语言效果不佳，MySQL 会实现语言特定的 Unicode 校对。语言特定的校对是基于 UCA 的，具有额外的语言定制规则。此类规则的示例稍后在本节中出现。有关特定语言排序的问题，[`unicode.org`](http://unicode.org)提供了通用语言环境数据存储库（CLDR）校对图表，网址为[`www.unicode.org/cldr/charts/30/collation/index.html`](http://www.unicode.org/cldr/charts/30/collation/index.html)。

例如，非语言特定的`utf8mb4_0900_ai_ci`和语言特定的`utf8mb4_*`LOCALE`*_0900_ai_ci` Unicode 校对具有以下特点：

+   校对基于 UCA 9.0.0 和 CLDR v30，是不区分重音和大小写的。这些特征在校对名称中用`_0900`，`_ai`和`_ci`表示。例外：`utf8mb4_la_0900_ai_ci`不是基于 CLDR，因为古典拉丁语在 CLDR 中未定义。

+   校对适用于范围[U+0, U+10FFFF]中的所有字符。

+   如果排序规则不是特定于语言的，它会按照默认顺序（如下所述）对所有字符进行排序，包括补充字符。如果排序规则是特定于语言的，它会根据特定于语言的规则正确对该语言的字符进行排序，并对不属于该语言的字符按默认顺序排序。

+   默认情况下，排序规则根据 DUCET 表（默认 Unicode 排序元素表）中列出的代码点对字符进行排序，根据表中分配的权重值。排序规则根据 UCA 构造的隐式权重值对未在 DUCET 表中列出代码点的字符进行排序。

+   对于非特定于语言的排序规则，缩约序列中的字符被视为单独的字符。对于特定于语言的排序规则，缩约可能会改变字符的排序顺序。

包含在下表中显示的区域代码或语言名称的排序规则名称是特定于语言的排序规则。Unicode 字符集可能包括一个或多个这些语言的排序规则。

**表 12.3 Unicode 排序语言标识符**

| 语言 | 语言标识符 |
| --- | --- |
| 波斯尼亚语 | `bs` |
| 保加利亚语 | `bg` |
| 中文 | `zh` |
| 古典拉丁语 | `la` 或 `roman` |
| 克罗地亚语 | `hr` 或 `croatian` |
| 捷克语 | `cs` 或 `czech` |
| 丹麦语 | `da` 或 `danish` |
| 世界语 | `eo` 或 `esperanto` |
| 爱沙尼亚语 | `et` 或 `estonian` |
| 加利西亚语 | `gl` |
| 德语电话簿排序 | `de_pb` 或 `german2` |
| 匈牙利语 | `hu` 或 `hungarian` |
| 冰岛语 | `is` 或 `icelandic` |
| 日语 | `ja` |
| 拉脱维亚语 | `lv` 或 `latvian` |
| 立陶宛语 | `lt` 或 `lithuanian` |
| 蒙古语 | `mn` |
| 挪威语 / 书面挪威语 | `nb` |
| 挪威语 / 新挪威语 | `nn` |
| 波斯语 | `persian` |
| 波兰语 | `pl` 或 `polish` |
| 罗马尼亚语 | `ro` 或 `romanian` |
| 俄语 | `ru` |
| 塞尔维亚语 | `sr` |
| 僧伽罗语 | `sinhala` |
| 斯洛伐克语 | `sk` 或 `slovak` |
| 斯洛文尼亚语 | `sl` 或 `slovenian` |
| 现代西班牙语 | `es` 或 `spanish` |
| 传统西班牙语 | `es_trad` 或 `spanish2` |
| 瑞典语 | `sv` 或 `swedish` |
| 土耳其语 | `tr` 或 `turkish` |
| 越南语 | `vi` 或 `vietnamese` |
| 语言 | 语言标识符 |

MySQL 8.0.30 及更高版本提供了保加利亚语排序规则 `utf8mb4_bg_0900_ai_ci` 和 `utf8mb4_bg_0900_as_cs`。

克罗地亚排序规则专为以下克罗地亚字母定制：`Č`、`Ć`、`Dž`、`Đ`、`Lj`、`Nj`、`Š`、`Ž`。

MySQL 8.0.30 及更高版本为塞尔维亚语提供了 `utf8mb4_sr_latn_0900_ai_ci` 和 `utf8mb4_sr_latn_0900_as_cs` 排序规则，为波斯尼亚语提供了 `utf8mb4_bs_0900_ai_ci` 和 `utf8mb4_bs_0900_as_cs` 排序规则，当这些语言使用拉丁字母书写时。

从 MySQL 8.0.30 开始，MySQL 为挪���的两种主要变体提供了排序规则：对于书面挪威语，您可以使用 `utf8mb4_nb_0900_ai_ci` 和 `utf8mb4_nb_0900_as_cs`；对于新挪威语，MySQL 现在提供了 `utf8mb4_nn_0900_ai_ci` 和 `utf8mb4_nn_0900_as_cs`。 

对于日语，`utf8mb4`字符集包括`utf8mb4_ja_0900_as_cs`和`utf8mb4_ja_0900_as_cs_ks`校对。这两种校对都是区分重音和区分大小写的。`utf8mb4_ja_0900_as_cs_ks`还区分假名，将片假名字符与平假名字符区分开，而`utf8mb4_ja_0900_as_cs`将片假名和平假名字符视为排序相等。需要日语校对但不需要假名敏感性的应用程序可以使用`utf8mb4_ja_0900_as_cs`以获得更好的排序性能。`utf8mb4_ja_0900_as_cs`使用三个权重级别进行排序；`utf8mb4_ja_0900_as_cs_ks`使用四个。

对于不区分重音的古典拉丁校对，`I`和`J`视为相等，`U`和`V`视为相等。`I`和`J`，以及`U`和`V`在基本字母级别上视为相等。换句话说，`J`被视为带重音的`I`，`U`被视为带重音的`V`。

MySQL 8.0.30 及更高版本提供了蒙古语的校对，使用西里尔字母书写，`utf8mb4_mn_cyrl_0900_ai_ci`和`utf8mb4_mn_cyrl_0900_as_cs`。

西班牙语校对适用于现代和传统西班牙语。对于两者，`ñ`（n-tilde）是`n`和`o`之间的单独字母。此外，对于传统西班牙语，`ch`是`c`和`d`之间的单独字母，`ll`是`l`和`m`之间的单独字母。

传统西班牙语校对也可用于阿斯图里亚斯语和加利西亚语。从 MySQL 8.0.30 开始，MySQL 还为加利西亚语提供了`utf8mb4_gl_0900_ai_ci`和`utf8mb4_gl_0900_as_cs`校对（这些校对与`utf8mb4_es_0900_ai_ci`和`utf8mb4_es_0900_as_cs`相同）。

瑞典校对包括瑞典规则。例如，在瑞典语中，以下关系成立，这是德语或法语说话者所不期望的：

```sql
Ü = Y < Ö
```

#### _general_ci 与 _unicode_ci 校对

对于任何 Unicode 字符集，使用`*`xxx`*_general_ci`校对的操作比`*`xxx`*_unicode_ci`校对的操作更快。例如，`utf8mb4_general_ci`校对的比较速度更快，但略微不够准确，比`utf8mb4_unicode_ci`校对的比较速度更快。原因是`utf8mb4_unicode_ci`支持映射，例如扩展；也就是说，当一个字符与其他字符的组合相等时。例如，在德语和其他一些语言中，`ß`等于`ss`。`utf8mb4_unicode_ci`还支持缩写和可忽略字符。`utf8mb4_general_ci`是一个不支持扩展、缩写或可忽略字符的传统校对。它只能在字符之间进行一对一的比较。

进一步说明，在`utf8mb4_general_ci`和`utf8mb4_unicode_ci`中以下相等性成立（对于比较或搜索的影响，请参阅第 12.8.6 节，“校对效果示例”）：

```sql
Ä = A
Ö = O
Ü = U
```

校对规则之间的一个区别是对于`utf8mb4_general_ci`是真实的：

```sql
ß = s
```

而对于支持德国 DIN-1 排序（也称为字典顺序）的`utf8mb4_unicode_ci`是真实的：

```sql
ß = ss
```

如果使用`utf8mb4_unicode_ci`的排序对某种语言不起作用，MySQL 会实现特定语言的 Unicode 校对规则。例如，`utf8mb4_unicode_ci`适用于德语字典顺序和法语，因此不需要创建特殊的`utf8mb4`校对规则。

`utf8mb4_general_ci`对于德语和法语都是令人满意的，除了`ß`等于`s`，而不等于`ss`。如果这对您的应用程序可接受，应该使用`utf8mb4_general_ci`因为它更快。如果这不可接受（例如，如果需要德语字典顺序），应该使用`utf8mb4_unicode_ci`因为它更准确。

如果需要德国 DIN-2（电话簿）排序，请使用`utf8mb4_german2_ci`校对规则，它将以下字符集视为相等：

```sql
Ä = Æ = AE
Ö = Œ = OE
Ü = UE
ß = ss
```

`utf8mb4_german2_ci`类似于`latin1_german2_ci`，但后者不将`Æ`视为等于`AE`或`Œ`视为等于`OE`。对于德语字典顺序，没有对应于`latin1_german_ci`的`utf8mb4_german_ci`，因为`utf8mb4_general_ci`已经足够。

#### 字符排序权重

字符的排序权重确定如下：

+   对于除`_bin`（二进制）校对规则之外的所有 Unicode 校对规则，MySQL 执行表查找以找到字符的排序权重。

+   对于除`utf8mb4_0900_bin`之外的`_bin`校对规则，权重基于代码点，可能会添加前导零字节。

+   对于`utf8mb4_0900_bin`，权重是`utf8mb4`编码的字节。排序顺序与`utf8mb4_bin`相同，但速度更快。

可以使用`WEIGHT_STRING()`函数显示排序权重。（参见第 14.8 节，“字符串函数和运算符”。）如果校对规则使用权重查找表，但某个字符不在表中（例如，因为它是一个“新”字符），排序权重的确定变得更加复杂：

+   对于一般校对规则中的 BMP 字符（`*`xxx`*_general_ci`），权重是代码点。

+   对于 UCA 校对规则中的 BMP 字符（例如`*`xxx`*_unicode_ci`和特定语言的校对规则），应用以下算法：

    ```sql
    if (code >= 0x3400 && code <= 0x4DB5)
      base= 0xFB80; /* CJK Ideograph Extension */
    else if (code >= 0x4E00 && code <= 0x9FA5)
      base= 0xFB40; /* CJK Ideograph */
    else
      base= 0xFBC0; /* All other characters */
    aaaa= base +  (code >> 15);
    bbbb= (code & 0x7FFF) | 0x8000;
    ```

    结果是两个排序元素的序列，`aaaa`后跟`bbbb`。例如：

    ```sql
    mysql> SELECT HEX(WEIGHT_STRING(_ucs2 0x04CF COLLATE ucs2_unicode_ci));
    +----------------------------------------------------------+
    | HEX(WEIGHT_STRING(_ucs2 0x04CF COLLATE ucs2_unicode_ci)) |
    +----------------------------------------------------------+
    | FBC084CF                                                 |
    +----------------------------------------------------------+
    ```

    因此，`U+04cf CYRILLIC SMALL LETTER PALOCHKA`（`ӏ`）在所有 UCA 4.0.0 校对规则中都大于`U+04c0 CYRILLIC LETTER PALOCHKA`（`Ӏ`）。对于 UCA 5.2.0 校对规则，所有帕洛奇卡字符一起排序。

+   对于一般校对规则中的补充字符，权重是`0xfffd REPLACEMENT CHARACTER`的权重。对于 UCA 4.0.0 校对规则中的补充字符，它们的排序权重是`0xfffd`。也就是说，对于 MySQL 来说，所有补充字符彼此相等，并且大于几乎所有 BMP 字符。

    使用 Deseret 字符和`COUNT(DISTINCT)`的示例：

    ```sql
    CREATE TABLE t (s1 VARCHAR(5) CHARACTER SET utf32 COLLATE utf32_unicode_ci);
    INSERT INTO t VALUES (0xfffd);   /* REPLACEMENT CHARACTER */
    INSERT INTO t VALUES (0x010412); /* DESERET CAPITAL LETTER BEE */
    INSERT INTO t VALUES (0x010413); /* DESERET CAPITAL LETTER TEE */
    SELECT COUNT(DISTINCT s1) FROM t;
    ```

    结果为 2，因为在 MySQL `*`xxx`*_unicode_ci`校对中，替换字符的权重为`0x0dc6`，而 Deseret Bee 和 Deseret Tee 的权重均为`0xfffd`。（如果使用`utf32_general_ci`校对，则结果为 1，因为在该校对中，这三个字符的权重均为`0xfffd`。）

    使用楔形文字字符和`WEIGHT_STRING()`的示例：

    ```sql
    /*
    The four characters in the INSERT string are
    00000041  # LATIN CAPITAL LETTER A
    0001218F  # CUNEIFORM SIGN KAB
    000121A7  # CUNEIFORM SIGN KISH
    00000042  # LATIN CAPITAL LETTER B
    */
    CREATE TABLE t (s1 CHAR(4) CHARACTER SET utf32 COLLATE utf32_unicode_ci);
    INSERT INTO t VALUES (0x000000410001218f000121a700000042);
    SELECT HEX(WEIGHT_STRING(s1)) FROM t;
    ```

    结果为：

    ```sql
    0E33 FFFD FFFD 0E4A
    ```

    `0E33`和`0E4A`是 UCA 4.0.0 中的主要权重。`FFFD`是 KAB 和 KISH 的权重。

    所有补充字符相等的规则并非最佳选择，但不会引起问题。这些字符非常罕见，因此很少出现多字符字符串完全由补充字符组成。在日本，由于补充字符是晦涩的汉字表意文字，典型用户无论如何都不在乎它们的顺序。如果您真的希望按照 MySQL 规则和其次按照代码点值对行进行排序，那很容易实现：

    ```sql
    ORDER BY s1 COLLATE utf32_unicode_ci, s1 COLLATE utf32_bin
    ```

+   对于基于高于 4.0.0 版本的 UCA 的补充字符（例如，`*`xxx`*_unicode_520_ci`），补充字符不一定都具有相同的排序权重。有些具有来自 UCA `allkeys.txt`文件的显式权重。其他根据此算法计算权重：

    ```sql
    aaaa= base +  (code >> 15);
    bbbb= (code & 0x7FFF) | 0x8000;
    ```

“按字符的代码值排序”和“按字符的二进制表示排序”之间存在差异，这种差异仅在`utf16_bin`中出现，因为存在代理。

假设`utf16_bin`（`utf16`的二进制校对）是一种“逐字节”而不是“逐字符”进行二进制比较。如果是这样，那么`utf16_bin`中字符的顺序将与`utf8mb4_bin`中的顺序不同。例如，以下图表显示了两个罕见字符。第一个字符位于`E000`-`FFFF`范围内，因此大于代理但小于补充字符。第二个字符是一个补充字符。

```sql
Code point  Character                    utf8mb4      utf16
----------  ---------                    -------      -----
0FF9D       HALFWIDTH KATAKANA LETTER N  EF BE 9D     FF 9D
10384       UGARITIC LETTER DELTA        F0 90 8E 84  D8 00 DF 84
```

图表中的两个字符按代码点值顺序排列，因为`0xff9d` < `0x10384`。它们按`utf8mb4`值顺序排列，因为`0xef` < `0xf0`。但如果我们使用逐字节比较，它们按`utf16`值顺序排列，因为`0xff` > `0xd8`。

因此，MySQL 的`utf16_bin`校对不是“逐字节”的。它是“按代码点”。当 MySQL 在`utf16`中看到补充字符编码时，它会转换为字符的代码点值，然后进行比较。因此，`utf8mb4_bin`和`utf16_bin`具有相同的排序。这与 SQL:2008 标准对 UCS_BASIC 校对的要求一致：“UCS_BASIC 是一种校对，其排序完全由要排序的字符串中字符的 Unicode 标量值确定。它适用于 UCS 字符库。由于每个字符库都是 UCS 字符库的子集，因此 UCS_BASIC 校对可能适用于每个字符集。注 11：字符的 Unicode 标量值是将其代码点视为无符号整数。”

如果字符集是`ucs2`，比较是逐字节的，但`ucs2`字符串不应包含代理项。

#### 其他信息

`*`xxx`*_general_mysql500_ci`校对保留了原始`*`xxx`*_general_ci`校对的 5.1.24 版本之前的排序，并允许升级用于在 MySQL 5.1.24 之前创建的表（Bug＃27877）。
