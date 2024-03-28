> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-schema-auto-increment-columns.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-schema-auto-increment-columns.html)

#### 30.4.3.24 `schema_auto_increment_columns`视图

此视图指示哪些表具有`AUTO_INCREMENT`列，并提供有关这些列的信息，例如当前值和最大列值以及使用率（已使用值与可能值的比率）。默认情况下，按降序使用率和最大列值对行进行排序。

这些模式中的表在查看输出中被排除：`mysql`，`sys`，`INFORMATION_SCHEMA`，`performance_schema`。

`schema_auto_increment_columns`视图具有以下列：

+   `table_schema`

    包含表的模式。

+   `table_name`

    包含`AUTO_INCREMENT`列的表。

+   `column_name`

    `AUTO_INCREMENT`列的名称。

+   `data_type`

    列的数据类型。

+   `column_type`

    列的列类型，即数据类型加上可能的其他信息。例如，对于具有`bigint(20) unsigned`列类型的列，数据类型只是`bigint`。

+   `is_signed`

    列类型是否为有符号。

+   `is_unsigned`

    列类型是否为无符号。

+   `max_value`

    列的最大允许值。

+   `auto_increment`

    列的当前`AUTO_INCREMENT`值。

+   `auto_increment_ratio`

    用于列的已使用值与允许值的比率。这表示数值序列中有多少“已使用”。
