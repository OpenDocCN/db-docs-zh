# 12.10.2 西欧字符集

> 译文：[`dev.mysql.com/doc/refman/8.0/en/charset-we-sets.html`](https://dev.mysql.com/doc/refman/8.0/en/charset-we-sets.html)

西欧字符集涵盖大多数西欧语言，如法语、西班牙语、加泰罗尼亚语、巴斯克语、葡萄牙语、意大利语、阿尔巴尼亚语、荷兰语、德语、丹麦语、瑞典语、挪威语、芬兰语、法罗语、冰岛语、爱尔兰语、苏格兰语和英语。

+   `ascii`（美国 ASCII）排序规则：

    +   `ascii_bin`

    +   `ascii_general_ci`（默认）

+   `cp850`（DOS 西欧）排序规则：

    +   `cp850_bin`

    +   `cp850_general_ci`（默认）

+   `dec8`（DEC 西欧）排序规则：

    +   `dec8_bin`

    +   `dec8_swedish_ci`（默认）

    MySQL 8.0.28 中已弃用 `dec` 字符集；预计在后续 MySQL 版本中将删除对其的支持。

+   `hp8`（HP 西欧）排序规则：

    +   `hp8_bin`

    +   `hp8_english_ci`（默认）

    MySQL 8.0.28 中已弃用 `hp8` 字符集；预计在后续 MySQL 版本中将删除对其的支持。

+   `latin1`（cp1252 西欧）排序规则：

    +   `latin1_bin`

    +   `latin1_danish_ci`

    +   `latin1_general_ci`

    +   `latin1_general_cs`

    +   `latin1_german1_ci`

    +   `latin1_german2_ci`

    +   `latin1_spanish_ci`

    +   `latin1_swedish_ci`（默认）

    MySQL 的 `latin1` 与 Windows 的 `cp1252` 字符集相同。这意味着它与官方的 `ISO 8859-1` 或 IANA（互联网编号分配机构）`latin1` 相同，只是 IANA `latin1` 将 `0x80` 到 `0x9f` 之间的代码点视为“未定义”，而 `cp1252`，因此 MySQL 的 `latin1`，为这些位置分配了字符。例如，`0x80` 是欧元符号。对于 `cp1252` 中的“未定义”条目，MySQL 将 `0x81` 转换为 Unicode `0x0081`，`0x8d` 转换为 `0x008d`，`0x8f` 转换为 `0x008f`，`0x90` 转换为 `0x0090`，`0x9d` 转换为 `0x009d`。

    `latin1_swedish_ci`排序规则是 MySQL 大多数客户可能使用的默认排序规则。尽管经常说它基于瑞典/芬兰排序规则，但也有瑞典人和芬兰人不同意这种说法。

    `latin1_german1_ci` 和 `latin1_german2_ci` 排序规则基于 DIN-1 和 DIN-2 标准，其中 DIN 代表*德国标准化学会*（ANSI 的德国等效）。DIN-1 称为“字典排序”，DIN-2 称为“电话簿排序”。有关此在比较或进行搜索时的影响的示例，请参见第 12.8.6 节，“排序效果示例”。

    +   `latin1_german1_ci`��字典）规则：

        ```sql
        Ä = A
        Ö = O
        Ü = U
        ß = s
        ```

    +   `latin1_german2_ci`（电话簿）规则：

        ```sql
        Ä = AE
        Ö = OE
        Ü = UE
        ß = ss
        ```

    在 `latin1_spanish_ci` 排序规则中，`ñ`（n-tilde）是 `n` 和 `o` 之间的一个独立字母。

+   `macroman`（Mac 西欧）排序规则：

    +   `macroman_bin`

    +   `macroman_general_ci`（默认）

    `macroroman` 在 MySQL 8.0.28 中已弃用；预计在后续 MySQL 版本中将删除对其的支持。

+   `swe7`（7 位瑞典语）排序规则：

    +   `swe7_bin`

    +   `swe7_swedish_ci`（默认）
