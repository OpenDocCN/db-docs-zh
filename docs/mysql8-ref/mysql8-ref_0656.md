# 11.1.7 NULL Values

> 原文：[`dev.mysql.com/doc/refman/8.0/en/null-values.html`](https://dev.mysql.com/doc/refman/8.0/en/null-values.html)

`NULL` 值表示“无数据”。`NULL` 可以以任何大小写形式编写。

请注意，`NULL` 值与数值类型的 `0` 或字符串类型的空字符串等值是不同的。有关更多信息，请参见 Section B.3.4.3, “Problems with NULL Values”。

对于使用 `LOAD DATA` 或 `SELECT ... INTO OUTFILE` 执行的文本文件导入或导出操作，`NULL` 由 `\N` 序列表示。参见 Section 15.2.9, “LOAD DATA Statement”。

对于使用 `ORDER BY` 进行排序，`NULL` 值在升序排序时排在其他值之前，在降序排序时排在其他值之后。
