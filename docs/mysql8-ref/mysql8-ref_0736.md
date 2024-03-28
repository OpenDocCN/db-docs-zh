# 13.1.3 固定点类型（精确值）- DECIMAL, NUMERIC

> 原文：[`dev.mysql.com/doc/refman/8.0/en/fixed-point-types.html`](https://dev.mysql.com/doc/refman/8.0/en/fixed-point-types.html)

`DECIMAL`和`NUMERIC`类型存储精确的数值数据。当需要保留精确精度时，例如在货币数据中，使用这些类型。在 MySQL 中，`NUMERIC`被实现为`DECIMAL`，因此关于`DECIMAL`的以下说明同样适用于`NUMERIC`。

MySQL 以二进制格式存储`DECIMAL`值。请参阅第 14.24 节，“精度数学”。

在`DECIMAL`列声明中，通常会指定精度和标度。例如：

```sql
salary DECIMAL(5,2)
```

在这个例子中，`5`是精度，`2`是标度。精度表示存储值的有效数字的数量，而标度表示小数点后可以存储的数字的数量。

标准 SQL 要求`DECIMAL(5,2)`能够存储任何具有五位数字和两位小数的值，因此可以存储在`salary`列中的值范围从`-999.99`到`999.99`。

在标准 SQL 中，语法`DECIMAL(*`M`*)`等同于`DECIMAL(*`M`*,0)`。类似地，语法`DECIMAL`等同于`DECIMAL(*`M`*,0)`，其中实现允许决定*`M`*的值。MySQL 支持`DECIMAL`语法的这两种变体形式。*`M`*的默认值为 10。

如果标度为 0，则`DECIMAL`值不包含小数点或小数部分。

`DECIMAL`的最大数字位数为 65，但给定`DECIMAL`列的实际范围可以受到给定列的精度或标度的限制。当为这样的列分配一个具有比指定标度允许的更多小数位数的值时，该值将被转换为该标度。（具体行为取决于操作系统，但通常效果是截断为允许的数字位数。）
