# 26.2.3 列分区

> 原文：[`dev.mysql.com/doc/refman/8.0/en/partitioning-columns.html`](https://dev.mysql.com/doc/refman/8.0/en/partitioning-columns.html)

26.2.3.1 范围列分区

26.2.3.2 列表列分区

接下来的两节讨论`COLUMNS`分区，这是`RANGE`和`LIST`分区的变体。`COLUMNS`分区允许在分区键中使用多个列。所有这些列都被考虑用于将行放入分区以及确定哪些分区要检查以匹配行进行分区修剪。

此外，`RANGE COLUMNS`分区和`LIST COLUMNS`分区都支持使用非整数列来定义值范围或列表成员。允许的数据类型如下列表所示：

+   所有整数类型：`TINYINT`, `SMALLINT`, `MEDIUMINT`, `INT` (`INTEGER`), 和 `BIGINT`。（这与按`RANGE`和`LIST`进行分区相同。）

    其他数值数据类型（如`DECIMAL`或`FLOAT` 和 `DATETIME`。

    不支持使用其他与日期或时间相关的数据类型作为分区列。

+   以下字符串类型：`CHAR`, `VARCHAR`, `BINARY`, 和 `VARBINARY`。

    `TEXT` 和 `BLOB` 列不支持作为分区列。

下面两节关于`RANGE COLUMNS`和`LIST COLUMNS`分区的讨论假定您已经熟悉基于范围和列表的分区，这在 MySQL 5.1 及更高版本中得到支持；有关更多信息，请参见第 26.2.1 节，“范围分区”和第 26.2.2 节，“列表分区”。
