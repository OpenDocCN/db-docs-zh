# 12.10.7 亚洲字符集

> 原文：[`dev.mysql.com/doc/refman/8.0/en/charset-asian-sets.html`](https://dev.mysql.com/doc/refman/8.0/en/charset-asian-sets.html)

12.10.7.1 cp932 字符集

12.10.7.2 gb18030 字符集

我们支持的亚洲字符集包括中文、日文、韩文和泰文。这些可能会比较复杂。例如，中文字符集必须允许成千上万种不同的字符。有关 `cp932` 和 `sjis` 字符集的更多信息，请参见第 12.10.7.1 节，“cp932 字符集”。有关支持中国国家标准 GB 18030 字符集的更多信息，请参见第 12.10.7.2 节，“gb18030 字符集”。

有关 MySQL 中支持亚洲字符集的一些常见问题和问题的答案，请参见第 A.11 节，“MySQL 8.0 FAQ: MySQL 中文、日文和韩文字符集”。

+   `big5` (Big5 繁体中文) 校对规则:

    +   `big5_bin`

    +   `big5_chinese_ci` (默认)

+   `cp932` (Windows 日文 SJIS) 校对规则:

    +   `cp932_bin`

    +   `cp932_japanese_ci` (默认)

+   `eucjpms` (Windows 日文 UJIS) 校对规则:

    +   `eucjpms_bin`

    +   `eucjpms_japanese_ci` (默认)

+   `euckr` (EUC-KR 韩文) 校对规则:

    +   `euckr_bin`

    +   `euckr_korean_ci` (默认)

+   `gb2312` (GB2312 简体中文) 校对规则:

    +   `gb2312_bin`

    +   `gb2312_chinese_ci` (默认)

+   `gbk` (GBK 简体中文) 校对规则:

    +   `gbk_bin`

    +   `gbk_chinese_ci` (默认)

+   `gb18030` (中国国家标准 GB18030) 校对规则:

    +   `gb18030_bin`

    +   `gb18030_chinese_ci` (默认)

    +   `gb18030_unicode_520_ci`

+   `sjis` (Shift-JIS 日文) 校对规则:

    +   `sjis_bin`

    +   `sjis_japanese_ci` (默认)

+   `tis620` (TIS620 泰文) 校对规则:

    +   `tis620_bin`

    +   `tis620_thai_ci` (默认)

+   `ujis` (EUC-JP 日文) 校对规则:

    +   `ujis_bin`

    +   `ujis_japanese_ci` (默认)

`big5_chinese_ci` 校对规则按笔画数排序。
